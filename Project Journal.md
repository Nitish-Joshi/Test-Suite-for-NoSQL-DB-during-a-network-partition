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

# Task 1:
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
