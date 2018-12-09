# MongoDB Sharding: Research, Setup and Configurations.

## Research & Architecture:  
* <b>Mongod</b> - A daemon program which runs quietly in background on your machine. Responsible for handling data requests, managing data access, and other background operations.  
* <b>Mongo</b> - A default command line client or interface for MongoDB.  
* <b>Mongos</b> - Mongos runs on the application server and play a role of query router for sharded clusters.  

* Main objective of Sharding in MongoDB is scalibility.   
* MongoDB supports horizontal scalability which can be achieved with the help of Sharding.   

### What is Sharding?  
* Sharding is a method for distributing data across multiple machines.   
* MongoDB uses Sharding to support deployments with very large data sets and high throughput operations.   

## Sharded Cluster:

### Components of MongoDB sharded cluster:

1. <b>CONFIG SERVERS:</b> 
* Configuration details and metadata are stored in Config servers. 
* Metadata stores the information about the locations of data chunks, which is important since the data will be distributed across multiple shards. 

2. <b>MONGOS:</b> 
- Mongos runs on the application server and plays the role of query router. 
- The job of query router is to process the queries from the application layer and communicate with the config servers to figure out where a given piece of information is stored into the sharded cluster. 
- It then accesses and returns the data from the appropriate shard(s).

3. <b>SHARD:</b>
- A shard is simply a database server that contains a subset of the sharded data.
- In production environments, a single shard is usually composed of a replica set instead of a single machine to ensure data availability.
- Items in the database are divided among shards either by range or hashing.

## Architecture:

![MongoDB Sharding Architecture](https://github.com/nguyensjsu/cmpe281-Nitish-Joshi/blob/master/MongoDB%20Sharding/MongoDB%20Sharding%20Architecture.png)

## SHARDING STRATEGIES:

1) Hash Based Strategy: Colelction data is partitioned into chunks and then based on the hashed shard key values, each chunk is then assigned a range.

2) Range Sharding: Collection data is divided into ranges [min,max] determined by the shard key. 

### MongoDB default ports:

|Server/Service|Port Number|
|--------------|-----------|
|Mongod, Mongos|   27017   |
|Shard Server  |   27018   |
|Config Server |   27019   |

## SETUP:

![Configurations](https://github.com/nguyensjsu/cmpe281-Nitish-Joshi/blob/master/Screenshots/Sharding%20SS/Sharding%20Setup%20on%20AWS.png)  

![SG1](https://github.com/nguyensjsu/cmpe281-Nitish-Joshi/blob/master/Screenshots/Sharding%20SS/Sharding%20security%20group%20configuration.png)  

![SG2](https://github.com/nguyensjsu/cmpe281-Nitish-Joshi/blob/master/Screenshots/Sharding%20SS/Sharding%20security%20group%20configuration%20-%20part2.png)  

### Security Groups:

1. mongodb-internal-access   
Port Range = 27017-27019  
Source = Custom - sg  

2. mongodb-external-access  
Port = 27017  
Source = 0.0.0.0/0  

### Create config-server-1:

Launch Instance       :'Amazon Linux AMI' type.  
Instance Type         : t2.micro  
Number of instances   : 1  
Network               : VPC cmpe281-oregon  
Subnet                : Public  
Auto-assign Public IP : Enable  
Security Group        : mongodb-internal-access  
Key-value pair        : cmpe281-oregon  

* For the installation of MongoDB on the AWS server, we first need to configure the package management system (yum).  

$ sudo nano /etc/yum.repos.d/mongodb-org-3.4.repo

[mongodb-org-3.4]  
name=MongoDB Repository  
baseurl=https://repo.mongodb.org/yum/amazon/2013.03/mongodb-org/3.4/x86_64/  
gpgcheck=1  
enabled=1  
gpgkey=https://www.mongodb.org/static/pgp/server-3.4.asc  

$ sudo yum update -y  
$ sudo yum install -y mongodb-org-3.4.10

#### To start mongodb at boot, issue the following command:
$ sudo chkconfig mongod on

* Create an Image of the above config-server-1.

### Setting up the config-server-1

#### Creating data directory path and set the ownership of that folder.
$ sudo mkdir -p /data/db
$ sudo chown -R mongod:mongod /data/db

* Create the remaining servers, config-server-2, mongos and the 4 shard servers using the image created earlier.

#### Configure the mongod.conf 
$ sudo nano /etc/mongod.conf
- change dbpath to /data/db
- port to 27019
- comment out bindIp // so that config-server can be accessible from anywhere
- uncomment replication and add -> replSetName:crs
- uncomment sharding and add -> clusterRole:configsvr

#### Start the mongod daemon by specifying the configuration file path and log path.
$ sudo mongod --config /etc/mongod.conf --logpath /var/log/mongodb/mongod.log

#### Check the running process
$ ps -aux | grep mongod

#### To check port of the mongod process
$ sudo lsof -iTCP -sTCP:LISTEN | grep mongo

### Setting up the config-server-2

* Repeat the same steps used for config-server-1.

#### Now connect the mongo shell to one of the config-servers by specifying the port numbers.

$ mongo -port 27019

#### To deploy our config-servers as a replica set, we have to initiate the replica set here, using the rs.initiate()

rs.initiate(  
{  
	_id:"crs",  
	configsvr: true,  
	members:[
        {_id : 0, host : "private-ip:27019"},
        {_id : 1, host : "private-ip:27019"}
	]  
}  
)  

### Setup 4 Shard Servers:

#### Creating data directory path and set the ownership of that folder.
$ sudo mkdir -p /data/db
$ sudo chown -R mongod:mongod /data/db

#### Configure the mongod.conf file
$ sudo nano /etc/mongod.conf
- change dbpath to /data/db
- port to 27018
- comment out bindIp // so that config-server can be accessible from anywhere
- uncomment replication and add -> replSetName:rs0/rs1
- uncomment sharding and add -> clusterRole:shardsvr

#### Start the mongod daemon by specifying the configuration file path and log path.
$ sudo mongod --config /etc/mongod.conf --logpath /var/log/mongodb/mongod.log

#### Now connect the mongo shell to one of the shard-servers by specifying the port numbers.

$ mongo -port 27018

#### To deploy our shard-servers as a replica set, we have to initiate the replica set here, using the rs.initiate()

rs.initiate(  
{  
	_id:"rs0",  
	shardsvr: true,  
	members:[
        {_id : 0, host : "private-ip:27019"},
        {_id : 1, host : "private-ip:27019"}
	]  
}  
)  

rs.initiate(  
{  
	_id:"rs1",  
	shardsvr: true,  
	members:[
        {_id : 0, host : "private-ip:27019"},
        {_id : 1, host : "private-ip:27019"}
	]  
}  
)  

### Setup Mongos:

$ sudo yum install -y mongodb-org-mongos-3.4.17

$ sudo nano /etc/mongod.conf
configDB: crs/pvt-ip:27019,pvt-ip:27019

$ mongo -port 27017

$ sh.addShard("rs0/11.0.0.197:27018,11.0.0.158:27018")
$ sh.addShard("rs1/11.0.0.84:27018,11.0.0.65:27018")

$ db.adminCommand({listShards:1})

$ use admin
$ db.runCommand({enablesharding:"laliga"})
$ db.bios.ensureIndex( { _id : "hashed" } )
$ sh.shardCollection( "laliga.bios", { "_id" : "hashed" } )

$ use laliga 

* Insert bios data for sharding 

$ db.bios.getShardDistribution()

#### Check for sharded data on shard servers:
$ db.bios.count()  
$ db.bios.find().pretty()

