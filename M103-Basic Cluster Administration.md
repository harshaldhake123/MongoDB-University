# M103 - Basic Cluster Administration

# Table of Contents:

 [M103 - Basic Cluster Administration](#m103---basic-cluster-administration)
  * [Chapter 0:  Introduction & Setup](#chapter-0---introduction---setup)
  * [Chapter 1: The Mongod](#chapter-1--the-mongod)
    + [Lab - Launching Mongod :](#lab---launching-mongod--)
    + [Lab - Configuration File](#lab---configuration-file)
    + [Lab - Change the Default DB Path](#lab---change-the-default-db-path)
    + [Lab - Logging to a Different Facility](#lab---logging-to-a-different-facility)
    + [Lab - Creating First Application User](#lab---creating-first-application-user)
    + [Lab - Importing a Dataset](#lab---importing-a-dataset)
   
  * [Chapter 2: Replication](#chapter-2--replication)
    + [Lab - Initiate a Replica Set Locally](#lab---initiate-a-replica-set-locally)
    + [Lab - Remove and Re-Add a Node](#lab---remove-and-re-add-a-node)
    + [Lab - Writes with Failovers](#lab---writes-with-failovers)
  
  * [Chapter 3: Sharding](#chapter-3--sharding)
  * [Final Exam](#final-exam)
    + [Final: Question 1](#final--question-1)
    + [Final: Question 2](#final--question-2)
    + [Final: Question 3](#final--question-3)
    + [Final: Question 4](#final--question-4)
    + [Final: Question 5](#final--question-5)

**Working directory:**

    C:\Users\harsh\m103\m103-vagrant-env

## Chapter 0:  Introduction & Setup


    vagrant@m103:~$ vagrant ssh
    vagrant@m103:~$ validate_box

> *OutputValidationKey*


## Chapter 1: The Mongod

    vagrant@m103:~$ mongod --dbpath /data/db/ --port 27000 --bind_ip "127.0.0.1,192.168.103.100" --auth
    
    vagrant@m103:~$ mongo admin --host localhost:27000 --eval '
      db.createUser({
        user: "m103-admin",
        pwd: "m103-pass",
        roles: [
          {role: "root", db: "admin"}
        ]
      })
    '

### Lab - Launching Mongod :

    vagrant@m103:~$ validate_lab_launch_mongod

> *OutputValidationKey*

-----------------------------------------------------------------------------------------------------------

    vagrant@m103:~$ mkdir /data/logs
    vagrant@m103:~$ vi /data/mongod.conf

> storage:   
> &ensp;dbPath: "/data/db" 
> systemLog:   
> &ensp;path: "/data/logs/mongod.log"   
> &ensp;destination: "file" 
> net:   
> &ensp;bindIp :"127.0.0.1,192.168.103.100"   
> &ensp;port: 27000 
> security:   
> &ensp;authorization:enabled 
> processManagement:   
> &ensp;fork : true

  
    vagrant@m103:~$ mongod -f /data/mongod.conf

### Lab - Configuration File


    vagrant@m103:~$ validate_lab_configuration_file

> *OutputValidationKey*
--------------------------------------------------------------------------------------------------------
### Lab - Change the Default DB Path

    vagrant@m103:~$ sudo mkdir -p /var/mongodb/db/
    vagrant@m103:~$ mkdir -p /var/mongodb/db/
    vagrant@m103:~$ sudo kill -9 2400
    vagrant@m103:~$ mongod -f /data/mongod.conf
    vagrant@m103:~$ rm -rf mongodb-27000.sock
    vagrant@m103:~$ sudo chown -R vagrant:vagrant /var/mongodb/db/

> storage:   
> &ensp;dbPath: "/var/mongodb/db/"   
> systemLog:   
> &ensp;path: "/data/logs/mongod.log"   
> &ensp;destination: "file"    
> net:   
> &ensp;bindIp : "127.0.0.1,192.168.103.100"   
> &ensp;port: 27000    
> security:  
> &ensp;authorization: enabled    
> processManagement:   
> &ensp;fork : true

    vagrant@m103:~$ mongo admin --host localhost:27000 --eval '
      db.createUser({
        user: "m103-admin",
        pwd: "m103-pass",
        roles: [
          {role: "root", db: "admin"}
        ]
      })
    '


---------------------------------

    vagrant@m103:~$ validate_lab_change_dbpath

> *OutputValidationKey*
--------------------------------------------------------------
### Lab - Logging to a Different Facility
    vagrant@m103:~$ mongo admin --port 27000 -u m103-admin -p m103-pass --eval 'db.shutdownServer()'

> 
> storage:   
> &ensp;dbPath: "/var/mongodb/db/" 
> systemLog:   
> &ensp;path: "/var/mongodb/db/mongod.log"   
> &ensp;destination: "file"   
> &ensp;logAppend: true
> net:   
> &ensp;bindIp : "127.0.0.1,192.168.103.100"   
> &ensp;port: 27000 
> security:  
> &ensp;authorization: enabled 
> processManagement:   
> &ensp;fork : true
> operationProfiling:
> &ensp;   slowOpThresholdMs: 50



---------------------------------------

    vagrant@m103:~$ validate_lab_different_logpath

> *OutputValidationKey*
--------------------------------------------------------------
### Lab - Creating First Application User
    vagrant@m103:~$ mongo admin --host localhost:27000 -u m103-admin -p m103-pass --eval '
      db.createUser({
        user: "m103-application-user",
        pwd: "m103-application-pass",
        roles: [
          {role: "readWrite", db: "applicationData"}
        ]
      })
    '


--------------------------------------

    vagrant@m103:~$ validate_lab_first_application_user

> *OutputValidationKey*
-------------------------------------------------------------
### Lab - Importing a Dataset
    vagrant@m103:~$ mongoimport --port 27000 -u m103-application-user -p m103-application-pass --authenticationDatabase admin -d applicationData -c products /dataset/products.json

> 2019-01-20T14:59:08.225+0000    connected to: localhost:27000
> 2019-01-20T14:59:11.212+0000    [###.....................]
> applicationData.products     14.6MB/87.9MB (16.6%)
> 2019-01-20T14:59:14.212+0000    [#######.................]
> applicationData.products     29.0MB/87.9MB (32.9%)
> 2019-01-20T14:59:17.209+0000    [###########.............]
> applicationData.products     43.3MB/87.9MB (49.2%)
> 2019-01-20T14:59:20.209+0000    [###############.........]
> applicationData.products     57.3MB/87.9MB (65.2%)
> 2019-01-20T14:59:23.209+0000    [###################.....]
> applicationData.products     71.7MB/87.9MB (81.5%)
> 2019-01-20T14:59:26.209+0000    [#######################.]
> applicationData.products     86.5MB/87.9MB (98.4%)
> 2019-01-20T14:59:26.470+0000    [########################]
> applicationData.products     87.9MB/87.9MB (100.0%)
> 2019-01-20T14:59:26.472+0000    imported 516784 documents



--------------------------

    validate_lab_import_dataset

> *OutputValidationKey*



## Chapter 2: Replication

### Lab - Initiate a Replica Set Locally
----------------------------

    vagrant@m103:~$ sudo -i
    root@m103:~# cd /etc/
    root@m103:/etc# nano mongod-repl-1.conf
    root@m103:/etc# cp mongod-repl-1.conf mongod-repl-2.conf
    root@m103:/etc# cp mongod-repl-1.conf mongod-repl-3.conf
    root@m103:/etc# nano mongod-repl-2.conf
    root@m103:/etc# nano mongod-repl-3.conf
    root@m103:/etc# mkdir -p /var/mongodb/pki
    root@m103:/etc# chown vagrant:vagrant -R /var/mongodb
    root@m103:/etc# openssl rand -base64 741 > /var/mongodb/pki/m103-keyfile
    root@m103:/etc# chmod 600 /var/mongodb/pki/m103-keyfile
    

**mongod-repl-1.conf:-**
> storage:
>  &ensp;dbPath: /var/mongodb/db/1
> systemLog:
>  &ensp;path: /var/mongodb/db/mongod1.log
>  &ensp;destination: file
>  &ensp;logAppend: true
> net:
>  &ensp;port: 27001
>  &ensp;bindIp: "localhost,192.168.103.100"
> security:
>  &ensp;keyFile: /var/mongodb/pki/m103-keyfile
>  &ensp;authorization: enabled
> replication:
>  &ensp;replSetName: m103-repl
> processManagement:
>  &ensp;fork: true

-Similarly, change the *dbpath, port , logpath*  for **mongod-repl-2.conf** and **mongod-repl-3.conf**.

    root@m103:/etc# mkdir /var/mongodb/db/{1,2,3}

    vagrant@m103:~$ mongod -f mongod-repl-1.conf  
    vagrant@m103:~$ mongod -f mongod-repl-2.conf  
    vagrant@m103:~$ mongod -f mongod-repl-3.conf  
      
    vagrant@m103:~$ mongo --port 27001  
    MongoDB Enterprise m103-repl:PRIMARY> rs.initiate()  
    MongoDB Enterprise m103-repl:PRIMARY> use admin  
    MongoDB Enterprise m103-repl:PRIMARY> db.createUser({  
    								user: "m103-admin",  
    								pwd: "m103-pass",  
    								roles: [{role: "root", db: "admin"}]
    								})
    MongoDB Enterprise m103-repl:PRIMARY> exit  
    vagrant@m103:~$ mongo --host "m103-repl/192.168.103.100:27001" -u "m103-admin" -p "m103-pass" --authenticationDatabase "admin"  
    MongoDB Enterprise m103-repl:PRIMARY> rs.status()  
    MongoDB Enterprise m103-repl:PRIMARY> rs.add("m103:27002")  
    MongoDB Enterprise m103-repl:PRIMARY> rs.add("m103:27003")  

  
-------------------------------------  

    vagrant@m103:~$ validate_lab_initialize_local_replica_set  

> *OutputValidationKey*

--------------
### Lab - Remove and Re-Add a Node  


    vagrant@m103:~$mongo --host "m103-repl/192.168.103.100:27001" -u "m103-admin" -p "m103-pass" --authenticationDatabase "admin"  
    MongoDB Enterprise m103-repl:PRIMARY> rs.stepDown()  
    MongoDB Enterprise m103-repl:SECONDARY> rs.status()  
    MongoDB Enterprise m103-repl:PRIMARY>rs.remove("192.168.103.100:27001")  
    MongoDB Enterprise m103-repl:PRIMARY>rs.status()  
    MongoDB Enterprise m103-repl:PRIMARY>rs.add("m103:27001")  
    MongoDB Enterprise m103-repl:PRIMARY>rs.status()  

---------------------------
    vagrant@m103:~$ validate_lab_remove_readd_node  

> *OutputValidationKey*
--------------
### Lab - Writes with Failovers

**Problem: Evaluate the effect of using a write concern with a replica set where one node has failed.
Consider a 3-node replica set with only 2 healthy nodes, that receives the following  insert()  operation:**

    use payroll
    db.employees.insert(
      { "name": "Aditya", "salary_USD": 50000 },
      { "writeConcern": { "w": 3, "wtimeout": 1000 } }
    )


**Which of the following is true about this write operation?**

*Correct:*
1.  The unhealthy node will receive the new document when it is brought back online.

2. If a  writeConcernError  occurs, the document is still written to the healthy nodes.


*Incorrect:*
1. w: "majority"  would also cause this write operation to return with an error.

2. The write operation will always return with an error, even if  wtimeout  is not specified.

--------------
### Lab

## Chapter 3: Sharding

## Final Exam

### Final: Question 1  

**Problem: Which of the following are valid command line instructions to start a mongod? You may assume that all specified files already exist.**  
  
*Correct:*  
 1. mongod --logpath /var/log/mongo/mongod.log --dbpath /data/db --fork  
 2. mongod -f /etc/mongod.conf 
 
*Incorrect:*  
 1. mongod --dbpath /data/db --fork  
 2. mongod --log /var/log/mongo/mongod.log --authentication  
  
----------------------------------------------------------  
  
### Final: Question 2  

**Problem: Given the following config file: How many directories must MongoDB have access to? Disregard the path to the configuration file itself.**
  

    storage:  
      dbPath: /data/db  
    systemLog:   
      destination: file   
      path: /var/log/mongod.log   
    net:   
      bindIp: localhost,192.168.0.100   
    security:   
      keyFile: /var/pki/keyfile   
    processManagement:   
      fork: true

  
*Correct:*  
1. 3 

*Incorrect:*  
1. 1  
2. 2  
3. 4  
  
------------------------------------------------------------  
  
### Final: Question 3  

  
**Problem: Given the following output from rs.status().members:**  
  

    [  
    {  
    "_id": 0,  
    "name": "localhost:27017",  
    "health": 1,  
    "state": 1,  
    "stateStr": "PRIMARY",  
    "uptime": 548,  
    "optime": {  
    "ts": Timestamp(1521038871, 1),  
    "t": NumberLong("1")  
    },  
    "optimeDate": ISODate("2018-03-14T14:47:51Z"),  
    "electionTime": Timestamp(1521038358, 2),  
    "electionDate": ISODate("2018-03-14T14:39:18Z"),  
    "configVersion": 2,  
    "self": true  
    },  
    {  
    "_id": 1,  
    "name": "localhost:27018",  
    "health": 1,  
    "state": 2,  
    "stateStr": "SECONDARY",  
    "uptime": 289,  
    "optime": {  
    "ts": Timestamp(1521038871, 1),  
    "t": NumberLong("1")  
    },  
    "optimeDurable": {  
    "ts": Timestamp(1521038871, 1),  
    "t": NumberLong("1")  
    },  
    "optimeDate": ISODate("2018-03-14T14:47:51Z"),  
    "optimeDurableDate": ISODate("2018-03-14T14:47:51Z"),  
    "lastHeartbeat": ISODate("2018-03-14T14:47:56.558Z"),  
    "lastHeartbeatRecv": ISODate("2018-03-14T14:47:56.517Z"),  
    "pingMs": NumberLong("0"),  
    "syncingTo": "localhost:27022",  
    "configVersion": 2  
    },  
    {  
    "_id": 2,  
    "name": "localhost:27019",  
    "health": 1,  
    "state": 2,  
    "stateStr": "SECONDARY",  
    "uptime": 289,  
    "optime": {  
    "ts": Timestamp(1521038871, 1),  
    "t": NumberLong("1")  
    },  
    "optimeDurable": {  
    "ts": Timestamp(1521038871, 1),  
    "t": NumberLong("1")  
    },  
    "optimeDate": ISODate("2018-03-14T14:47:51Z"),  
    "optimeDurableDate": ISODate("2018-03-14T14:47:51Z"),  
    "lastHeartbeat": ISODate("2018-03-14T14:47:56.558Z"),  
    "lastHeartbeatRecv": ISODate("2018-03-14T14:47:56.654Z"),  
    "pingMs": NumberLong("0"),  
    "syncingTo": "localhost:27022",  
    "configVersion": 2  
    },  
    {  
    "_id": 3,  
    "name": "localhost:27020",  
    "health": 1,  
    "state": 2,  
    "stateStr": "SECONDARY",  
    "uptime": 289,  
    "optime": {  
    "ts": Timestamp(1521038871, 1),  
    "t": NumberLong("1")  
    },  
    "optimeDurable": {  
    "ts": Timestamp(1521038871, 1),  
    "t": NumberLong("1")  
    },  
    "optimeDate": ISODate("2018-03-14T14:47:51Z"),  
    "optimeDurableDate": ISODate("2018-03-14T14:47:51Z"),  
    "lastHeartbeat": ISODate("2018-03-14T14:47:56.558Z"),  
    "lastHeartbeatRecv": ISODate("2018-03-14T14:47:56.726Z"),  
    "pingMs": NumberLong("0"),  
    "syncingTo": "localhost:27022",  
    "configVersion": 2  
    },  
    {  
    "_id": 4,  
    "name": "localhost:27021",  
    "health": 0,  
    "state": 8,  
    "stateStr": "(not reachable/healthy)",  
    "uptime": 0,  
    "optime": {  
    "ts": Timestamp(0, 0),  
    "t": NumberLong("-1")  
    },  
    "optimeDurable": {  
    "ts": Timestamp(0, 0),  
    "t": NumberLong("-1")  
    },  
    "optimeDate": ISODate("1970-01-01T00:00:00Z"),  
    "optimeDurableDate": ISODate("1970-01-01T00:00:00Z"),  
    "lastHeartbeat": ISODate("2018-03-14T14:47:56.656Z"),  
    "lastHeartbeatRecv": ISODate("2018-03-14T14:47:12.668Z"),  
    "pingMs": NumberLong("0"),  
    "lastHeartbeatMessage": "Connection refused",  
    "configVersion": -1  
    },  
    {  
    "_id": 5,  
    "name": "localhost:27022",  
    "health": 1,  
    "state": 2,  
    "stateStr": "SECONDARY",  
    "uptime": 289,  
    "optime": {  
    "ts": Timestamp(1521038871, 1),  
    "t": NumberLong("1")  
    },  
    "optimeDurable": {  
    "ts": Timestamp(1521038871, 1),  
    "t": NumberLong("1")  
    },  
    "optimeDate": ISODate("2018-03-14T14:47:51Z"),  
    "optimeDurableDate": ISODate("2018-03-14T14:47:51Z"),  
    "lastHeartbeat": ISODate("2018-03-14T14:47:56.558Z"),  
    "lastHeartbeatRecv": ISODate("2018-03-14T14:47:55.974Z"),  
    "pingMs": NumberLong("0"),  
    "syncingTo": "localhost:27017",  
    "configVersion": 2  
    },  
    {  
    "_id": 6,  
    "name": "localhost:27023",  
    "health": 1,  
    "state": 2,  
    "stateStr": "SECONDARY",  
    "uptime": 289,  
    "optime": {  
    "ts": Timestamp(1521038871, 1),  
    "t": NumberLong("1")  
    },  
    "optimeDurable": {  
    "ts": Timestamp(1521038871, 1),  
    "t": NumberLong("1")  
    },  
    "optimeDate": ISODate("2018-03-14T14:47:51Z"),  
    "optimeDurableDate": ISODate("2018-03-14T14:47:51Z"),  
    "lastHeartbeat": ISODate("2018-03-14T14:47:56.558Z"),  
    "lastHeartbeatRecv": ISODate("2018-03-14T14:47:56.801Z"),  
    "pingMs": NumberLong("0"),  
    "syncingTo": "localhost:27022",  
    "configVersion": 2  
    }  
    ]  

  
**At this moment, how many replica set members are eligible to become primary in the event of the current Primary crashing or stepping down?**  
  
*Correct:*  
1. 5  

*Incorrect:*  
1. 4  
2. 6  
3. 7  
  
------------------------------------------------------------  
### Final: Question 4  

  
**Problem: Given the following replica set configuration:**  

  

    conf = {  
    "_id": "replset",  
    "version": 1,  
    "protocolVersion": 1,  
    "members": [  
    {  
    "_id": 0,  
    "host": "192.168.103.100:27017",  
    "priority": 2,  
    "votes": 1  
    },  
    {  
    "_id": 0,  
    "host": "192.168.103.100:27018",  
    "priority": 1,  
    "votes": 1  
    },  
    {  
    "_id": 2,  
    "host": "192.168.103.100:27018",  
    "priority": 1,  
    "votes": 1  
    }  
    ]  
    }  

  
**What errors are present in the above replica set configuration?**  
  
*Correct:*  
1. You cannot specify the same host information amoung multiple members.  
2. You cannot specify two members with the same _id  

*Incorrect:*  
1. You can only specify a priority of 0 or 1, member "_id": 0 is incorrectly configured  
2. you cannot have three members in a replica set.  
  
-------------------------------------------------------------------------------------  
### Final: Question 5  
  
  
**Problem: Given the following replica set configuration:**  
  

    conf = {  
    "_id": "replset",  
    "version": 1,  
    "protocolVersion": 1,  
    "members": [  
    {  
    "_id": 0,  
    "host": "localhost:27017",  
    "priority": 1,  
    "votes": 1  
    },  
    {  
    "_id": 1,  
    "host": "localhost:27018",  
    "priority": 1,  
    "votes": 1  
    },  
    {  
    "_id": 2,  
    "host": "localhost:27019",  
    "priority": 1,  
    "votes": 1  
    },  
    {  
    "_id": 3,  
    "host": "localhost:27020",  
    "priority": 0,  
    "votes": 0,  
    "slaveDelay": 3600  
    }  
    ]  
    }

  
  
**What is the most likely role served by the node with "_id": 3?**  
  
*Correct:*  
1. It serves as a "hot" backup of data in case of accidental data loss on the other members, like a DBA accidentally dropping the database.  

*Incorrect:*  
1. It servers to service reads and writes for people in the same geographic region as the host machine  
2. It servers as a reference to perform analytics on how data is changing over time  
3. It serves as a hidden secondary available to use for non-critical analysis operations  
  
-------------------------------------------------------------------------------------------------------------





