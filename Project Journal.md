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

# Week 1:
Objective ---> Initial Research on the basic understanding and functionality of the project, studying the requriements, identifying the key aspects and figuring out the necessary technologies to be used.

# Tasks:
* Studying NoSQL Databases
* Understanding CAP Theorem
* Understanding the requirements and architecture of the project through the article - https://www.infoq.com/articles/jepsen
* Researching on the NoSQL DB to be used for both CP and AP.
* Understanding Partition Tolerance and its pitfalls.
* Studying other related terminologies required for the project.

**What is CAP Theorem?**  
ANS: States that it is impossible for a distributed data store to simultaneously provide more than two of the following three guarantees:-
1. CONSISTENCY - Every read receives the most recent write or an error.
2. AVAILIBILITY - Every request receives a response that is not an error.
3. PARTITION TOLERANCE - The system continues to operate despite an arbitrary number of messages being dropped (or delayed) by the network between nodes. 

**What are the requirements of the project that have been understood from the project?**  
ANS: We are supposed to create a cluster of NoSQL DBS wherein if one of the system is not able to communicate with the rest of the systems due to network failure then also our API should be able to read stale data. Thus showing high availability of our cluster (partition tolerant system).

**What is partition tolerance and its pitfalls?**  
Ans: Partition tolerance in CAP means tolerance to a network partition. An example of NP is when two nodes cant talk to each other, but there are clients who are able to talk to either one of both of these nodes. An AP system is able to function during the network split, while being able to provide various forms of eventual consistency.

**What are the benefits of using MongoDB (NoSQL) for this project?**  
Ans: MongoDB is easy to scale. Supports replication and high availability. Use internal memory for storing the working set, enabling faster access of data. Also, with right configuration of replica set, partition tolerance can be tested on MongoDB.

# Week 2:
Objective ---> MongoDB cluster setup with replicaset configuration.

## Task 1 - Configure VPC:

### STEP 1:
1) Launch VPC Wizard
2) VPC with Public and Private Subnets
3) VPC Name = cmpe281
4) Use a NAT instance instead.
5) Instance type = t2.micro
6) Key pair name = cmpe281
7) Create VPC

## Task 2 - Configure and setup the primary instance of the MongoDB cluster:

### STEP 1: Creating a primary node.
1) Launch instance
2) Select'Ubuntu Server 16.04 LTS (HVM)' type instance.
3) Instance Type: t2.micro
4) Number of instances: 1
5) Network: VPC cmpe281
6) Subnet: Public
7) Auto-assign Public IP: Disable 
8) Security group: MongoDB (open ports = 22, 27017)
9) Key-value pair: cmpe281

** Name the primary instance as MongoDB primary

### STEP 2: Allocate a new elastic IP to the primary instance.
1) Allocate new address
2) Scope: VPC -> Allocate
3) Select the address
4) Associate address
5) Select the primary instance
6) Allocate

### STEP 3: SSH into the primary instance.
1) SSH into the above primary node using the keyValue pair.
(Use Putty for Windows - a free SSH client for Windows platform)

### STEP 4: Installing MongoDB on the primary node.
1) sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 9DA31620334BD75D9DCB49F368818C72E52529D4
2) echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/4.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.0.list
3) sudo apt-get update
4) sudo apt-get install -y mongodb-org

### STEP 5: Creating a key file that will be used to secure authentication between the members of the replica set.
1) Give this command to generate the key file:
  $ openssl rand -base64 741 > keyFile
2) Create the /opt/mongo directory to store the key file:
  $ sudo mkdir -p /opt/mongodb
3) Move the keyfile to /opt/mongo, and assign it the correct permissions:
  $ sudo cp keyFile /opt/mongodb
4) Update the ownership of the file.
  $ sudo chown mongodb:mongodb /opt/mongodb/keyFile
  $ sudo chmod 0600 /opt/mongodb/keyFile
  
### STEP :6 Config mongod.conf
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
        
### STEP 7: Creating mongod.service   
$ sudo nano /etc/systemd/system/mongod.service

[Unit]
    Description=High-performance, schema-free document-oriented database
    After=network.target

[Service]
    User=mongodb
    ExecStart=/usr/bin/mongod --quiet --config /etc/mongod.conf

[Install]
    WantedBy=multi-user.target
    
### STEP 8: Enabling Mongo Service
$ sudo systemctl enable mongod.service

### STEP 9: Restart MongoDB to apply the changes
$ sudo service mongod restart
$ sudo service mongod status

### STEP 10: Initialize the replica set
$ mongo
$ rs.initiate()
$ rs.status()

## Task 3 - Create AMI of the primary node

### STEPS:
1) Select the primary instance -> Actions -> Image -> Create Image
2) Name = MongoDB AMI

## Task 4 - Configure the replicaset (secondary and arbiter nodes) from the AMI

### STEP 1: Creating the secondary nodes.
1) Launch Instance 
2) My AMIs: 'MongoDB AMI'
3) Instance Type: t2.micro
4) Number of instances: 3
5) Network: VPC cmpe281
6) Subnet: Public subnet
7) Auto-assign Public IP: Disable 
8) Security Group: MongoDB (22, 27017)
9) Key-value pair: cmpe281

** Name the instances as MongoDB secondary1, MongoDB secondary2, MongoDB secondary3

### STEP 2: Assigning elastic IP to secondary nodes.
1) Allocate new address
2) Scope: VPC -> Allocate
3) Select the address
4) Associate address
5) Select the secondary instance

** Repeat the above steps for other 2 secondary nodes.

NOTE: We can only allocate a maximum of 5 elastic IPs. Hence, for the arbiter node, we need to auto-assign the public IP as we have allocated the 5 elastic IPs to the primary node, 3 secondary nodes and the NAT instance.

### STEP 3: Creating the aribiter node.
1) Launch instance
2) My AMIs: 'MongoDB AMI'
3) Instance Type: t2.micro
4) Number of instances: 1
5) Network: VPC cmpe281
6) Subnet: Public subnet
7) Auto-assign Public IP: Enable 
8) Security Group: MongoDB (22, 27017)
9) Key-value pair: cmpe281

** Name the instance as MongoDB arbiter.




