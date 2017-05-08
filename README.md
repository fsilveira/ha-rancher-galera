# docker-mariadb-cluster

__Consider this a POC and not a production ready system!__ 

Built for use with Docker __17.04.0-ce__+ in __Swarm Mode__

# WORK in Progress!

## Setup
### Init Swarm Nodes/Cluster

Swarm Master:
		
	docker swarm init
		
Additional Swarm Node(s):

	docker swarm join <MasterNodeIP>:2377 + join-tokens shown at swarm init

To get the tokens at a later time, run `docker swarm join-token (manager|worker)`

### Create DB network

	docker network create -d overlay mysubnet

### Init/Bootstrap DB Cluster 

At first we start with a new service, which is set to `--replicas=1` to turn this instance into a bootstrapping node.
If there is just one service task running within the cluster, this instance automatically starts with `bootstrapping` enabled. 

	docker service create --name dbcluster \
	--network mydbnet \
	--replicas=1 \
	--env DB_SERVICE_NAME=dbcluster \
	sysc0d/mariadb-cluster

Note: the service name provided by `--name` has to match the environment variable __DB_SERVICE_NAME__ set with `--env DB_SERVICE_NAME`.
	
Of course there are the default MariaDB options to define a root password, create a database, create a user and set a password for this user.
Example:

	docker service create --name dbcluster \
	--network mydbnet \
	--replicas=1 \
	--env DB_SERVICE_NAME=dbcluster \
	--env MYSQL_ROOT_PASSWORD=rootpass \
	--env MYSQL_DATABASE=mydb \
	--env MYSQL_USER=mydbuser \
	--env MYSQL_PASSWORD=mydbpass \
	sysc0d/mariadb-cluster

### Scale out additional cluster members
Just after the first service instance/task is running with we are good to scale out.
Check service with `docker service ps dbcluster`. The result should look like this, with __CURRENT STATE__ telling something like __Running__.

	ID                         NAME         IMAGE                    NODE    DESIRED STATE  CURRENT STATE           ERROR
	7c81muy053eoc28p5wrap2uzn  dbcluster.1  sysc0d/mariadb-cluster  node01  Running        Running 41 seconds ago  

Lets scale out now:

	docker service scale dbcluster=3

This additional 2 nodes start will come up in "cluster join"-mode. Lets check again: `docker service ps dbcluster`

	ID                         NAME         IMAGE                    NODE    DESIRED STATE  CURRENT STATE               ERROR
	7c81muy053eoc28p5wrap2uzn  dbcluster.1  sysc0d/mariadb-cluster  node01  Running        Running 6 minutes ago       
	8ht037ka0j4g6lnhc194pxqfn  dbcluster.2  sysc0d/mariadb-cluster  node02  Running        Running about a minute ago  
	bgk07betq9pwgkgpd3eoozu6u  dbcluster.3  sysc0d/mariadb-cluster  node03  Running        Running about a minute ago 

### Verify cluster nodes
```bash
docker exec -it dbcluster.<RN>.<ID> mysql -u root -p -e "SHOW STATUS LIKE 'wsrep_cluster_size'"
+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| wsrep_cluster_size | 3     |
+--------------------+-------+
```
