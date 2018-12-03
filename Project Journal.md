# Objectives:

In this project, you will be testing the Partition Tolerance of a NoSQL DB using the procedures described in the following article: https://www.infoq.com/articles/jepsen. In addition, you will be developing a Data Service API in Go and deploying the API on AWS.

# Requirements:
* Select one CP and one AP NoSQL Database.
* For example: Mongo and Riak (Note: Other NoSQL DBs that can be configured in CP or AP mode are also allowed).
* For each Database:
* Set up your cluster as AWS EC2 Instances. (# of Nodes and Topology is open per your design)
* Set up the Experiments (i.e. Test Cases), run the Experiments and record the results for both AP and CP.
* For one of your NoSQL database, develop the Go API and integrate the API with the Team SaaS App from the Hackathon Project.
* Project must be use in a GitHub Repo (assigned by TA) to maintain source code, documentation and design diagrams.
* Repo will be maintain in: https://github.com/nguyensjsu (Links to an external site.)Links to an external site.
* Keep a Project Journal (as a markdown document) recording weekly progress, challenges, tests and test results.
* All documentation must be written in GitHub Markdown (including diagrams) and committed to GitHub.

## Week 1:
Objective ---> Initial Research on the basic understanding and functionality of the project, studying the requriements, identifying the key aspects and figuring out the necessary technologies to be used.

### Tasks:
* Studying NoSQL Databases
* Understanding CAP Theorem
* Understanding the requirements and architecture of the project through the article - https://www.infoq.com/articles/jepsen
* Researching on the NoSQL DB to be used for both CP and AP.
* Understanding Partition Tolerance and its pitfalls.
* Studying other related terminologies required for the project.

**What is CAP Theorem?**  
ANS: States that it is impossible for a distributed data store to simultaneously provide more than two of the following three guarantees:-  
1.CONSISTENCY - Every read receives the most recent write or an error.  
2.AVAILIBILITY - Every request receives a response that is not an error.  
3.PARTITION TOLERANCE - The system continues to operate despite an arbitrary number of messages being dropped (or delayed) by the network between nodes. 

**What are the requirements of the project that have been understood from the project?**  
ANS: We are supposed to create a cluster of NoSQL DBS wherein if one of the system is not able to communicate with the rest of the systems due to network failure then also our API should be able to read stale data. Thus showing high availability of our cluster (partition tolerant system).

**What is partition tolerance and its pitfalls?**  
Ans: Partition tolerance in CAP means tolerance to a network partition. An example of NP is when two nodes cant talk to each other, but there are clients who are able to talk to either one of both of these nodes. An AP system is able to function during the network split, while being able to provide various forms of eventual consistency.

**What are the benefits of using MongoDB (NoSQL) for this project?**  
Ans: MongoDB is easy to scale. Supports replication and high availability. Use internal memory for storing the working set, enabling faster access of data. Also, with right configuration of replica set, partition tolerance can be tested on MongoDB.

## Week 2:
Objective ---> MongoDB cluster setup with replicaset configuration.

### Task 1 - Create a MongoDB jumpbox:

1) Launch instance
2) Select'Amazon Linux AMI' type instance.
3) Instance Type: t2.micro
4) Number of instances: 1
5) Network: VPC cmpe281 (N.California Region)
6) Subnet: Public
7) Auto-assign Public IP: Enable 
8) Security group: MongoDB (open ports = 22, 27017)
9) Key-value pair: cmpe281

* SSH into the MongoDB jumpbox using the keyValue pair.   
(Use Putty for Windows - a free SSH client for Windows platform) 

### Task 2 - Install MongoDB:

1) sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 9DA31620334BD75D9DCB49F368818C72E52529D4  
2) echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/4.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.0.list
3) sudo apt-get update  
4) sudo apt-get install -y mongodb-org  

### Task 3 - Create a key file to secure authentication between the members of your replica set:

1) Generate your key file:   
$ openssl rand -base64 741 > keyFile  

2) Create and store your key file in the /opt/mongo directory:  
$ sudo mkdir -p /opt/mongodb  

3) Move the keyfile to /opt/mongo, and assign it the correct permissions:  
$ sudo cp keyFile /opt/mongodb  

4) Update the file ownership.  
$ sudo chown mongodb:mongodb /opt/mongodb/keyFile  
$ sudo chmod 0600 /opt/mongodb/keyFile  

### Task 4 - Config mongod.conf:

$ sudo nano /etc/mongod.conf  

1)  remove or comment out bindIp: 127.0.0.1  
    replace with bindIp: 0.0.0.0 (binds on all ips)  

    (network interfaces)  
    net:  
        port: 27017  
        bindIp: 0.0.0.0  

2) Uncomment security section & add key file  

    security:  
      keyFile: /opt/mongodb/keyFile  

3) Uncomment Replication section. Name Replica Set = cmpe281

    replication:  
        replSetName: cmpe281  
        
### Task 5 - Create mongod.service:

$ sudo nano /etc/systemd/system/mongod.service  

[Unit]  
    Description=High-performance, schema-free document-oriented database  
    After=network.target  

[Service]  
    User=mongodb  
    ExecStart=/usr/bin/mongod --quiet --config /etc/mongod.conf  

[Install]  
    WantedBy=multi-user.target  
    
### Task 6 - Enable Mongo Service:  

$ sudo systemctl enable mongod.service  

### Task 7 - Restart MongoDB to apply our changes:  

$ sudo service mongod restart  
$ sudo service mongod status  

### Task 8 - Create AMI of the instance with MongoDB installed:

1) Select the primary instance -> Actions -> Image -> Create Image  
2) Name = MongoDB AMI  

* We will be setting up the MongoDB cluster in two regions, N. California and Oregon.
* The MongoDB instances will communicate with each other using the [VPC Peering](https://github.com/nguyensjsu/cmpe281-Nitish-Joshi/blob/master/VPC%20peering.md) technique.

### Task 9 - Create the MongoDB instances:

#### Subtask 1 - Instances in N. California Region.

1) Launch Instance 'Ubuntu Server 16.04 LTS (HVM)' type.
2) My AMIs: 'MongoDB AMI'
3) Instance Type: t2.micro
4) Number of instances: 3
5) Network: VPC cmpe281
6) Subnet: Private
7) Auto-assign Public IP: Disable 
8) Security Group: MongoDB (22, 27017)
9) Key-value pair: cmpe281

#### Subtask 2 - Instances in Oregon Region.

1) Launch Instance 'Ubuntu Server 16.04 LTS (HVM)' type.
2) My AMIs: 'MongoDB AMI'
3) Instance Type: t2.micro
4) Number of instances: 2
5) Network: VPC cmpe281-oregon
6) Subnet: Private
7) Auto-assign Public IP: Disable 
8) Security Group: MongoDB (22, 27017)
9) Key-value pair: cmpe281-oregon

** To begin with, one primary and two secondary nodes will be in N. California Region and the other two secondary nodes will be in the Oregon region.
** SSH into the primary instance via the MongoDB jumpbox.

* Connecting to private instance from a public instance:  
--> SSH into the Public instance (jumpbox in this case).  
--> Get .pem file into the root folder from local. (Use Winscp for windows).  
--> Make sure private instance has port 22 opened.  
--> sudo ssh -i cmpe281.pem ec2-user@<private-ip of the instance>  

### Task 10 - Initialize the replica set:

$ mongo  
$ rs.initiate()  
$ rs.status()  

### Task 11 - Create administrator account on the primary instance:

$ mongo  
$ use admin  
$ db.createUser( {   
  user: "admin",  
  pwd: "cmpe281",  
  roles: [{ role: "root", db: "admin" }]   
});  

### Task 12 - Initiate replication and add secondary nodes to the replica set:

$ mongo  
$ use admin  
$ db.auth("admin","cmpe281")    
$ rs.initiate()  
$ rs.add("10.0.1.9") // secondary1  
$ rs.add("10.0.1.84") // secondary2  
$ rs.add("11.0.1.237") // secondary3  
$ rs.add("11.0.1.54") // secondary4  

$ rs.status()  

## Week 3:
Objective ---> Riak cluster setup.

### PART I: CONFIGURING A RIAK CLUSTER ON AWS:

### Task 1 - Launching Riak Marketplace AMI (5 Nodes):
1. AMI:             Riak KV 2.2 Series
2. Instance Type:   t2.micro
3. VPC:             cmpe281
4. Network:         private subnet
5. Auto Public IP:  no
6. Security Group:  riak-cluster 
7. SG Open Ports:   (see below)
8. Key Pair:        cmpe281-us-west-1

#### Security group setup:
* Riak Cluster Security Group (Open Ports): 22(SSH), 8087 (Riak Protocol Buffers Interface), 8098 (Riak HTTP Interface).
* In order to allow communication between the Riak instances, need to add additional rules within this security group.
* The additional rules are - 4369, 6000-7999, 8099, 9080 (Set the source to the current security group).

### Task 2 - Launching "Jump Box" AWS Linux AMI:
1. AMI:             Amazon Linux AMI 2018.03.0 (HVM)
2. Instance Type:   t2.micro
3. VPC:             cmpe281
4. Network:         public subnet
5. Auto Public IP:  yes
6. Security Group:  cmpe281-dmz 
7. SG Open Ports:   22, 80, 443
8. Key Pair:        cmpe281-us-west-1

### Task 3 - SSH into Riak Instances (via Jump Box):
1. SSH into the Jump Box.
2. SSH into the Riak instance via the Jump Box. (Connecting to private instance from a public instance).

### Task 4 - Setup Riak Cluster:

#### Start Riak on all Nodes:
1. sudo riak start (Run the command on all instances)

#### Join Riak from Other Nodes to the First Node:
1. sudo riak-admin cluster join riak@<ip.of.first.node> (Run the command in all the other instances)

#### Plan and Commit Changes:
1. sudo riak-admin cluster plan 
2. sudo riak-admin cluster commit 

#### Check Status of the Cluster:
1. sudo riak-admin member_status 
2. sudo riak-admin cluster status 

** The Riak cluster is now running on the AWS.

### Task 5 - Create bucket-type in Riak:
1. sudo riak-admin bucket-type create subjects 
2. sudo riak-admin bucket-type activate subjects

** Since all our 5 Riak Nodes are in the private subnet, we will need Kong in order to access those instances.

### PART II: CONFIGURING KONG AND POSTMAN SETUP TO TEST RIAK AP PROPERTIES.

### Task 6 - Installing Docker on Riak Jumpbox (AWS EC2 Instance) :
1. sudo yum update -y
2. sudo yum install -y docker
3. sudo service docker start
4. sudo usermod -a -G docker ec2-user

### Task 7 -Setting up Kong on Docker:

#### Create Network:
$ sudo docker network create --driver bridge kong

#### List of Networks:
$ sudo docker network ls

#### Run Kong Database:
$ sudo docker run -d --name kong-database --network kong -p 9402:9402 cassandra:2.2

#### Run kong:
$ sudo docker run -d --name kong1 --network kong -e "KONG_DATABASE=cassandra" -e "KONG_CASSANDRA_CONTACT_POINTS=kong-db" -e "KONG_PG_HOST=kong-db" -p 8000:8000 -p 8443:8443 -p 8001:8001 -p 7946:7946 -p 7946:7946/udp kong:0.9.9

#### Check running docker containers:
$ sudo docker ps --all --format "table {{.ID}}\t{{.Names}}\t{{.Image}}\t{{.Status}}\t"

### Task 8 - Configure Kong setup via Postman:

** Add a security group named "Kong" with open ports 8000 and 8001 to configure Kong (via Postman)

#### Configure Kong Upstream URL for all the 5 Nodes.

POST http://13.56.164.9:8001/apis   //IP address of Riak Jumpbox

Body:  

{
"name":"node1",  
"request_path": "/node1",  
"strip_request_path":"true",  
"preserve_host":"true",  
"upstream_url":"http://10.0.1.112:8098/"   // Private IP of the Instance  
}

** Repeat the above steps for the remaining 4 nodes as well. (Replace the Private IP in the upstream URL)

#### Ping all the 5 Nodes.
GET http://13.56.164.9:8000/node1/ping   
GET http://13.56.164.9:8000/node2/ping   
GET http://13.56.164.9:8000/node3/ping   
GET http://13.56.164.9:8000/node4/ping   
GET http://13.56.164.9:8000/node5/ping   

#### Check Bucket Properties:
GET http://13.56.164.9:8000/node1/types/subjects/buckets/cmpe281/props

#### Set Bucket Properties:
PUT http://13.56.164.9:8000/node1/types/subjects/buckets/cmpe281/props

Body: 

{"props":{"allow_mult":false}}


## Week 4:
Objective ---> CRUD operations and GO API setup.

### Task:

1) Understand the GO API functionality.
2) Initial setup to run goapi 
3) Method for connecting mongodb using mgo
5) Implementing basic CRUD operations for Mongodb in GO

Kindly refer to the source code for CRUD API implementaion. 
