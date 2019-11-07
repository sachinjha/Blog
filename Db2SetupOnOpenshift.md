# Installing DB2 developer edition on Openshift (without IBM cloud pak for Data)



Follow the setps to install DB2 using the configuration used in helm charts published by IBM for use in ICP.
The oc new-app --docker-image option to install the db2 docker image from docker hub was tried but there were some issues, 
so the approach taken here is to convert the template files in helm chart into files with all values replaced and then use
the oc create -f  command to generate individual components like secrets, service and statefulset. 

<!--more-->
### Here are the high level steps
1. Clone IBM helm charts repository 
```bash
   git clone https://github.com/IBM/charts
```
2. create a copy of the file : stable/ibm-db2oltp-dev/values.yaml  so that you can update the values as required. 
   Sample values.yaml file is given below
```bash
   cp charts/stable/ibm-db2oltp-dev/values.yaml  .
```
3. Create a folder for template files with replaced values.
```bash
   mkdir kube-resources
```
4. Run the following command to replace the placeholders with values from values.yaml
```bash
helm template charts/stable/ibm-db2oltp-dev --output-dir ./kube-resources/  --values ./values.yaml  --name db2-release
```
5. Delete tests folder under ./kube-resources/
```bash
rm -rf kube-resources/ibm-db2oltp-dev/templates/tests
```
6. Create the required scc and give access to the namespace as described in the helm chart documentation 
    [podsecuritypolicy-requirements](https://github.com/IBM/charts/tree/master/stable/ibm-db2oltp-dev#podsecuritypolicy- requirements).
   "privileged" scc would most likely already exist in the cluster. Just add  the \<namespace\>:\<serviceaccounts\> group to      it using command below. Also run the script ./createSCCandNS.sh --namespace <NAMESPACE>  to add other required scc to    
   namespace.

```bash
 oc adm policy add-scc-to-group privileged system:serviceaccounts:<NAMESPACE>
```
   

7. Switch to the project where db2 is to be installed  and then create the resources using the generated files.

```bash
oc project db2
oc create -f ./kube-resources
```

### Check logs of db2 pod. 

It may take a little while to create the sample db. In my case it took 20+ mins for all the 
initialization activities to complete.

Sample log output 

```bash

Task #1 end 

Task #2 start
Description: Initializing instance list 
Estimated time 5 second(s) 
Task #2 end 

Task #3 start
Description: Configuring DB2 instances 
Estimated time 300 second(s) 
Task #3 end 

Task #4 start
Description: Updating global profile registry 
Estimated time 3 second(s) 
Task #4 end 

The execution completed successfully.

For more information see the DB2 installation log at "/tmp/db2icrt.log.124".
DBI1446I  The db2icrt command is running.


DBI1070I  Program db2icrt completed successfully.


(*) Enabling TEXT_SEARCH for instance ...
DB2 installation is being initialized.

 Total number of tasks to be performed: 4 
Total estimated time for all tasks to be performed: 309 second(s) 

Task #1 start
Description: Setting default global profile registry variables 
Estimated time 1 second(s) 
Task #1 end 

Task #2 start
Description: Initializing instance list 
Estimated time 5 second(s) 
Task #2 end 

Task #3 start
Description: Configuring DB2 instances 
Estimated time 300 second(s) 
Task #3 end 

Task #4 start
Description: Updating global profile registry 
Estimated time 3 second(s) 
Task #4 end 

The execution completed successfully.

For more information see the DB2 installation log at 
"/tmp/db2iupdt.log.15986".
DBI1446I  The db2iupdt command is running.


DBI1070I  Program db2iupdt completed successfully.


11/06/2019 14:05:53     0   0   SQL1032N  No start database manager command was issued.
SQL1032N  No start database manager command was issued.  SQLSTATE=57019
(*) Cataloging existing databases
ls: cannot access /database/data/db2inst1/NODE0000: No such file or directory
(*) Applying Db2 license ...

LIC1402I  License added successfully.


LIC1426I  This product is now licensed for use as outlined in your License Agreement.  USE OF THE PRODUCT CONSTITUTES ACCEPTANCE OF THE TERMS OF THE IBM LICENSE AGREEMENT, LOCATED IN THE FOLLOWING DIRECTORY: "/opt/ibm/db2/V11.1/license/en_US.iso88591"
(*) Saving the checksum of the current nodelock file ...
(*) Updating DBM CFG parameters ... 
DB20000I  The UPDATE DATABASE MANAGER CONFIGURATION command completed 
successfully.
DB20000I  The UPDATE DATABASE MANAGER CONFIGURATION command completed 
successfully.
DB20000I  The UPDATE DATABASE MANAGER CONFIGURATION command completed 
successfully.
(*) k8s environment flagged. Setting Db2 INSTANCE_MEMORY. 
cat: /sys/fs/cgroup/memory/kubepods/burstable/kubepods-burstable-pode03cf5d4_009d_11ea_b806_3a3c96059436.slice/memory.limit_in_bytes: No such file or directory
(standard_in) 1: syntax error
(standard_in) 1: syntax error
(*) Setting INSTANCE_MEMORY to %
SQL0104N  An unexpected token "END-OF-STATEMENT" was found following 
"INSTANCE_MEMORY".  Expected tokens may include:  "<identifier>".  
SQLSTATE=42601

DB2 State : Operable
DB2 has not been started
Starting DB2...

11/06/2019 14:06:21     0   0   SQL1063N  DB2START processing was successful.
SQL1063N  DB2START processing was successful.
(*) Starting TEXT SEARCH service ...
CIE00001 Operation completed successfully. 
(*) User chose to create sampledb database
(*) Creating database sampledb ... 
DB20000I  The CREATE DATABASE command completed successfully.
DB20000I  The ACTIVATE DATABASE command completed successfully.
11/06/2019 14:37:40     0   0   SQL1026N  The database manager is already active.
SQL1026N  The database manager is already active.
### Enabling LOGARCHMETH1

   Database Connection Information

 Database server        = DB2/LINUXX8664 11.1.4.4
 SQL authorization ID   = DB2INST1
 Local database alias   = SAMPLEDB

DB20000I  The UPDATE DATABASE CONFIGURATION command completed successfully.
SQL1363W  One or more of the parameters submitted for immediate modification 
were not changed dynamically. For these configuration parameters, the database 
must be shutdown and reactivated before the configuration parameter changes 
become effective.
### Restarting DB2
11/06/2019 14:38:11     0   0   SQL1064N  DB2STOP processing was successful.
SQL1064N  DB2STOP processing was successful.
11/06/2019 14:38:13     0   0   SQL1063N  DB2START processing was successful.
SQL1063N  DB2START processing was successful.
### Making backup directory and performing backup

Backup successful. The timestamp for this backup image is : 20191106143831

(*) Applying autoconfiguration for instance ... 

   Database Connection Information

 Database server        = DB2/LINUXX8664 11.1.4.4
 SQL authorization ID   = DB2INST1
 Local database alias   = SAMPLEDB

DB20000I  The UPDATE DATABASE CONFIGURATION command completed successfully.
SQL1363W  One or more of the parameters submitted for immediate modification 
were not changed dynamically. For these configuration parameters, the database 
must be shutdown and reactivated before the configuration parameter changes 
become effective.
DB20000I  The UPDATE DATABASE CONFIGURATION command completed successfully.
DB20000I  The UPDATE DATABASE CONFIGURATION command completed successfully.
SQL1363W  One or more of the parameters submitted for immediate modification 
were not changed dynamically. For these configuration parameters, the database 
must be shutdown and reactivated before the configuration parameter changes 
become effective.
DB20000I  The SQL command completed successfully.
11/06/2019 14:42:26     0   0   SQL1064N  DB2STOP processing was successful.
SQL1064N  DB2STOP processing was successful.
11/06/2019 14:42:28     0   0   SQL1063N  DB2START processing was successful.
SQL1063N  DB2START processing was successful.
(*) Enabling TEXT_SEARCH for database sampledb

   Database Connection Information

 Database server        = DB2/LINUXX8664 11.1.4.4
 SQL authorization ID   = DB2INST1
 Local database alias   = SAMPLEDB

DB20000I  The SQL command completed successfully.
DB20000I  The SQL command completed successfully.
CIE00001 Operation completed successfully. 
ssh-keygen: generating new host keys: RSA1 RSA DSA ECDSA ED25519 
(*) All databases are now active. 
(*) Setup has completed.
          from "/database/data/db2inst1/NODE0000/SQL00001/LOGSTREAM0000/".



```


### Identify NodePort and Test Installation using client.

1. In the sample values.yaml, service type has been set to NodePort. You can also use LoadBalancer if available in the environment.
2. Get nodeport details 
```bash
oc get svc -n <namespace>
```
#### Sample output 
   
```bash
db2-ibm-db2oltp-dev-1     ClusterIP     None            <none>        50000/TCP,55000/TCP,60006/TCP,60007/TCP    17h
db2-ibm-db2oltp-dev-db2-1  NodePort    172.21.18.105    <none>        50000:32346/TCP,55000:32508/TCP            18h
```
3. Find the public ip of one of the nodes then use the following URL and creds to connect to the db2 instnace from a client.

   Creds depend on following values in values.yaml
   db2inst:
     instname: "db2inst1"
     password: "password"

    Server:  <node public ip>:<32346>   ( NodePort corresponding to db2 port 50000 ) 
     
    jdbc driver can be downloaded from [here](https://jar-download.com/?search_box=db2jcc). jcc.jar has the DB2Driver class   
    which can be used with Quantum db plugin in Eclipse to test connectivity.


### Sample values.yaml file


```yaml
## Architecture - e.g. amd64, s390x, ppc64le. Specific worker node architecture
## to deploy to.
## You can use kubectl version command to determine the architecture on the
## desired worker node.

global:
  image:
    secretName: "docker-registry-secret-db2"
arch: "x86_64"
image:
  repository: store/ibmcorp/db2_developer_c
  tag: 11.1.4.4
  pullPolicy: IfNotPresent
service:
  name: ibm-db2oltp-dev
  type: NodePort
  port: 50000
  tsport: 55000
db2inst:
  instname: "db2inst1"
  password: "password"
options:
  databaseName: "sampledb"
  oracleCompatibility: false

## global persistence settings
persistence:
  enabled: true
  useDynamicProvisioning: true

## hadr option
hadr:
  enabled: false
  useDynamicProvisioning: false

## Persistence parameters for /database
dataVolume:
  name: "data-stor"

  ## Specify the name of the Existing Claim to be used by your application
  ## empty string means don't use an existClaim
  existingClaimName: ""

  ## Specify the name of the StorageClass
  ## empty string means don't use a StorageClass
  storageClassName: "ibmc-block-bronze"
  size: 20Gi

etcdVolume:
  name: "etcd-stor"

  ## Specify the name of the StorageClass
  ## empty string means don't use a StorageClass
  storageClassName: ""
  size: 1Gi


hadrVolume:
  name: "hadr-stor"

  ## Specify the name of the Existing Claim to be used by your application
  ## empty string means don't use an existClaim
  existingClaimName: ""

  ## Specify the name of the StorageClass
  ## empty string means don't use a StorageClass
  storageClassName: ""
  size: 1Gi

## Configure resource requests and limits
## ref: http://kubernetes.io/docs/user-guide/compute-resources/
##
resources:
  requests:
    memory: 2Gi
    cpu: 2000m
  limits:
    memory: 16Gi
    cpu: 4000m
````
