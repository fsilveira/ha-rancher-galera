# HA Rancher Galera

__Consider this a POC and not a production ready system!__ 

Built for use with Docker __17.05.0-ce__+ in __Swarm Mode__

## Setup Hosts
### Init Swarm Nodes/Cluster

Swarm Master:
		
	docker swarm init
		
Additional Swarm Node(s):

	docker swarm join <MasterNodeIP>:2377 + join-tokens shown at swarm init

To get the tokens at a later time, run `docker swarm join-token (manager|worker)`

### Create Rancher network
#### Create custom ingress if not exist (for docker >= 17.05.0-ce)

	docker network create -d overlay --ingress rancher-ingress

#### Create custom network 

        docker network create -d overlay --opt encrypted rancherntw
	
## Deploy app with Docker-compose

Docker Compose isn't required if you use swarm, if you need version 3 please show [docker-compose documentation](https://docs.docker.com/compose/install/)

```bash
docker stack deploy --compose-file docker-compose.yml ha
docker service scale ha_dbcluster=3
docker service scale ha_rancher=2
```

### Verify cluster nodes
#### Show services  
```bash
docker stack services ha
ID                  NAME                MODE                REPLICAS            IMAGE                           PORTS
2rnkf52mmd7x        ha_rancher          replicated          2/2                 rancher/server:latest           *:8080->8080/tcp
ism0wiqtgrls        ha_dbcluster        replicated          3/3                 sysc0d/mariadb-cluster:latest
```

#### Verify cluster DB
```bash
docker exec -it dbcluster.<RN>.<ID> mysql -u root -p -e "SHOW STATUS LIKE 'wsrep_cluster_size'"
+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| wsrep_cluster_size | 3     |
+--------------------+-------+
```
