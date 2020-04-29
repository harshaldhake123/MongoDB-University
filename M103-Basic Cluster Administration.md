
# M103 - Mongo University - Basic Cluster Administration

    C:\Users\harsh\m103\m103-vagrant-env


## Chapter 0: Introduction & Setup


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
--------------------------

    vagrant@m103:~$ validate_lab_configuration_file

> *OutputValidationKey*
--------------------------------------------------------------------------------------------------------

    vagrant@m103:~$ sudo mkdir -p /var/mongodb/db/
    vagrant@m103:~$ mkdir -p /var/mongodb/db/
    vagrant@m103:~$ sudo kill -9 2400
    vagrant@m103:~$ mongod -f /data/mongod.conf
    vagrant@m103:~$ rm -rf mongodb-27000.sock
    vagrant@m103:~$ sudo chown -R vagrant:vagrant /var/mongodb/db/

>storage:   
>&ensp;dbPath: "/var/mongodb/db/"   
>systemLog:   
>&ensp;path: "/data/logs/mongod.log"   
>&ensp;destination: "file"    
> net:   
> &ensp;bindIp : "127.0.0.1,192.168.103.100"   
> &ensp;port: 27000    
> security:  
>&ensp;authorization: enabled    
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

### Lab - Change the Default DB Path
---------------------------------

    vagrant@m103:~$ validate_lab_change_dbpath

> *OutputValidationKey*
--------------------------------------------------------------

    vagrant@m103:~$ mongo admin --port 27000 -u m103-admin -p m103-pass --eval 'db.shutdownServer()'

> 
> storage:   
> &ensp;dbPath: "/var/mongodb/db/" 
> systemLog:   
> &ensp;path: "/var/mongodb/db/mongod.log"   
> &ensp;destination: "file"   
> &ensp;logAppend: true
> net:   
>     &ensp;bindIp : "127.0.0.1,192.168.103.100"   
>     &ensp;port: 27000 
>  security:  
> &ensp;authorization: enabled 
> processManagement:   
> &ensp;fork : true
> operationProfiling:
> &ensp;   slowOpThresholdMs: 50


### Lab - Logging to a Different Facility
---------------------------------------

    vagrant@m103:~$ validate_lab_different_logpath

> *OutputValidationKey*
--------------------------------------------------------------

    vagrant@m103:~$ mongo admin --host localhost:27000 -u m103-admin -p m103-pass --eval '
      db.createUser({
        user: "m103-application-user",
        pwd: "m103-application-pass",
        roles: [
          {role: "readWrite", db: "applicationData"}
        ]
      })
    '

### Lab - Creating First Application User
--------------------------------------

    vagrant@m103:~$ validate_lab_first_application_user

> *OutputValidationKey*
-------------------------------------------------------------

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


### Lab - Importing a Dataset
--------------------------

    validate_lab_import_dataset

> *OutputValidationKey*



## Chapter 2: Replication










