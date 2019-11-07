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
3. Run the following command to replace the placeholders with values from values.yaml
```bash
helm template charts/stable/ibm-db2oltp-dev --output-dir ./kube-resources/  --values ./values.yaml  --name db2-release
```
4. Delete tests folder under ./kube-resources/
```bash
rm -rf kube-resources/ibm-db2oltp-dev/templates/tests
```
5. Create the required scc and give access to the namespace as described in the helm chart documentation 
    [podsecuritypolicy-requirements](https://github.com/IBM/charts/tree/master/stable/ibm-db2oltp-dev#podsecuritypolicy- requirements).
   "privileged" scc would most likely already exist in the cluster. Just add  the \<namespace\>:\<serviceaccounts\> group to      it using command below. Also run the script ./createSCCandNS.sh --namespace <NAMESPACE>  to add other required scc to    
   namespace.

```bash
 oc adm policy add-scc-to-group privileged system:serviceaccounts:<NAMESPACE>
```
   

6. Switch to the project where db2 is to be installed  and then create the resources using the generated files.

```bash
oc project db2
oc create -f ./kube-resources
```

7. Check logs of db2 pod. It may take a little while to create the sample db. In my case it took 20+ mins for all the 
initialization activities to complete.



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
