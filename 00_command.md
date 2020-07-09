# SE Team training Calm/Karbon 201 Lab

## Objectives

- Create CI/CD environment(SCM, Container registry, Artifact repository, pipeline management) using Nutanix Calm  
- Deploy containerized application on k8s(Karbon) cluster from Nutanix Calm  
- Create Blue-Green deployment using Nutanix Calm  

![Architecture Diagram](./images/CICDEnvironment.png)

## Prerequisites

- Kubernetes cluster (v1.16.10) built by Karbon 2.1.0
  - Install Metallb 0.9.3 [reference doc](https://next.nutanix.com/architectural-best-practices-74/utilizing-metallb-to-provide-loadbalancer-services-for-nutanix-karbon-32966) & [offical doc](https://metallb.universe.tf/installation/#installation-by-manifest)

```shell
# Install
kubectl apply --validate=false -f https://raw.githubusercontent.com/metallb/metallb/v0.9.3/manifests/namespace.yaml
kubectl apply --validate=false -f https://raw.githubusercontent.com/metallb/metallb/v0.9.3/manifests/metallb.yaml
# On first install only
kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)"
# Layer2 config prepare at least 2 available IPs
availip_begin="10.55.7.240"
availip_end="10.55.7.244"
echo "apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - ${availip_begin}-${availip_end}" |tee metallb-config.yaml
kubectl apply -f metallb-config.yaml
```

- Calm 3.0
  - add kubernetes provider to project

- Centos Image [download](http://download.nutanix.com/calm/CentOS-7-x86_64-GenericCloud-1801-01.qcow2)

- kubectl VM(Assigned from trainer, your laptop can also be used if it has the tools below.)  
kubectl  
kubeconfig  
watch  
git  

- run this script to get private/public keys used in this lab, or use yourself's

```shell
KEY_NAME=key-`date +%Y%m%d%H%M%S`
echo 'LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFb3dJQkFBS0NBUUVBeXp3UWRWaW5O
ZWhVMFZscEZqSjBXZktFOW9QejNpK1hyRWVrSUJDMnlEWFFpRjNvCjBPQVBKSTd2Q1NSYlhXaWV5
U05nbEIxWTE4RXJTVXZzSmJnNlArbFZzWEFMTU11NytZZkZNSDl3dWsrZ01Mb3cKRXIwZVExMFc1
cGtPYzhKTG8wOEt2K09Ma013UGZvSGhaQ05HQVNTRTk3R29iTTNOdkp3Z2I2ZU1WcVlJQTRDdQph
VHpPTDBQazJSYUpMN3VESHlWTDdNbFB1YjNQazR2MjU4WXJnUVZFL0tGdjVEZHV0ZUk4cDFLZG10
MS8xa2psCjhPYVVLQ0lVeFRnZlJyTW5aOHJsN2Qxd2FrcHJkYkREMkdCakw2WDN6YTNJTWtSb0tC
Ry9WVGNwaTJwWm9FSVYKVjFzOEplUTY4aHQ1SFhLMkV6NzNRd3NSZGxneGFBU09UUVppUHdJREFR
QUJBb0lCQUNEenNpOTBtbGZ4UFo5YwpBZC9wZWxnRExHL1Q0VHlMTnVwbUMwUnZzVk13OW9qRndU
UHN3RWZMTFJLc1pHdjJ4SmdVTmxxTmo1UWI2K3JNCmRwTi9GV0s3Y0h0K2tUSHVOcUkwMDFJVHQ3
RmRxN2w2MXNTVENVOGV6dEhBeTBrVm5iSHk1T2xMK1pzV213U0cKQm9kMytLZkN4aFJySjhYTWNL
aytIenc3VEZQWCtjU202TDFTNnNrZWN2TCtOcEsyY2sxa3pSeHRQMFdvOXR1awpwMXNPV040cWVw
cHJ3akZ6eTlGY0dORFhiUmM5T2N0U290T1NBMUhDTmxPN2szWjdDRFZ5b0RoRUdnMHIyYXNECjVX
Q2pva3ZiYU11VVAzSXVpVENJS1JoRXlPY3l0VUJBOU5KT3pzQ3VzZlE5dzBOcCtwTXp6cFhKcmtr
ZGdpNUsKVGlsVzNCRUNnWUVBNlJRYXhMeGxsN3BPdGZab0MzcG9mYmt5UnBVbjIyMWNFeEZzUTdX
RzZHaWRTVGUyVnliVQp6RWtxVnFYVWJBbWowOVZIcnk5eHg3aU9ZYnlIYnQzdHlvckloRmtNd0w2
a0R2Wjkza2dqM2pJMVhSbDMyVWhTCnRUQTlGY0JnQjZ2TmhacFBEMHM5TGhsR3dKUUtXeXRuZ0dp
YnZnbkhlajFLQ3dDRWhLdVJyMlVDZ1lFQTN6aWUKK2htbDBtRGlNbmJxTTN1c0NrVEZ0MDJsNVBw
cVpaNVNzOVJrZGpTMHJHT1VIRHBmd3RNUUJiQW5MT3oxbkpKcApyQ3lsRHFWUUpNd0hmTUZGd1hX
V1E5U2FSM2Q0SWdkNUN2RHFybGdJUlZqSDlBOGtjNkVFODVUTGlqMWpNSSthCjZ2eGV5Y2tHMXlX
TjRHTDAya2drRW1XN0xrN0toRjRMWDc1WGF0TUNnWUFmQzA5NWJzSjFVQnBWd0NraGhqOU8KcDVF
YnQ1Z2tjTTkzdHJWZDIxckpyODFyU0NGZ1FFck5udDBseDFhZ3dUUHg3aEZyTXYyTHBYaEpha3Y3
cDFHWgp6WnhoakY3WGVVenV0VVFVakRWQ3poc3ZlRW52UmVQR1FGT1NoeG9EUzQzSFFzTk8vUzE3
NjEveUhJVzRtUEpqCk53dXZaeXdMaXdoR3FOMzlzY0JveFFLQmdHcXZDdndKS2dGa0lIZHg4Zmp0
RUlwUDJCY0ZXMVlXZ0tkVWhiNUEKc2tyMFlhMUtZazBqY1FVU0RzUkpMbEFBYUZxT2QwYThoNTU2
VDlWbVZlWEFFWENtbkV5SzNibHVWZElkdERJUQpkbmtneGNNdTdWR3Rzd1E0WHI5enk5dWovellF
VEV3bWxiN20vd2Z3Z09CUEt5cnY4NTEvVlhpS1c2dnJyby94CnpxSFZBb0dCQU9KOVRwVVNXSEhI
MXdQR0hldmFFaTRJbWNvdTZ6eUlyWUdObE81RE1MMHhVWW15ejdrWW0wbW8KVjVqcEU2MU4yNTJq
VTBMNit1VkViUnpqaGdtZ281K2NGZEsxRHN5YlNIbFVZa3FFZHRhM3lOM25xMDNGTm9tawpwbjNr
SUZzUmxzcUxDQm5PWU1kbHJiRy90WEM1VFh4cVV4WWF0QS9KSDBWRW1oVEYzK29LCi0tLS0tRU5E
IFJTQSBQUklWQVRFIEtFWS0tLS0tCg==' |base64 --decode >${KEY_NAME}
chmod 600 ${KEY_NAME}
ssh-keygen -y -f ${KEY_NAME} > ${KEY_NAME}.pub
```

- Variables  
YOURNAME: Your Unique Name without spaces (ex: shuchida)

- Instruction  
[KubeVM]: Operate from your kubectl VM  
[CALM]: Operate from Calm GUI  
[Browser]: Operate from your local browser  
[DevWSVM]: Operate from the Developer's workspace VM created from 03_cicd-base.json blueprint  
[Jenkins]: Operate from your Jenkins instance created from 03_cicd-base.json blueprint  

## 1.Create MongoDB Pod from Calm

![Mongo](./images/Mongo.png)

## 1-1.[KubeVM] Create namespace

```shell
kubectl create ns [YOURNAME]
kubectl get ns
watch kubectl get deploy,po,svc -n [YOURNAME]
```

## 1-2.[CALM] Login to Calm and upload "01_cicd-mongo.json" blueprint

```shell
Blueprint Name: [YOURNAME]-cicd-mongo
Project: default
Password: nutanix/4u
```

![MongoUpload](./images/MongoUpload.png)

```text
Pod --> MongoDBPod --> Deployment
Account: [Karbon provider name]

Push Save button
```

![Mongok8s](./images/Mongok8s.png)

## 1-3.[CALM] Launch mongodb

```shell
Push Launch button
Name of the Application: [YOURNAME]-cicd-mongo
yourname: [YOURNAME]
Push Create button
```

![MongoLaunch](./images/MongoLaunch.png)

```shell
Note: You can ignore the 3 Warnings.
```

![MongoWarnings](./images/MongoWarnings.png)

## 1-4.[KubeVM] Check mongodb deployment, pod and service are created

```shell
kubectl get deploy,po,svc -n [YOURNAME]
```

```text
The output should be something like this.

$ kubectl get deploy,po,svc -n shuchida
NAME                                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/k8s-mongodb-deployment    1/1     1            1           9d

NAME                                           READY   STATUS    RESTARTS   AGE
pod/k8s-mongodb-deployment-7b4df64bb7-n9wp7    1/1     Running   0          9d

NAME                              TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)          AGE
service/mongodb-calm-lb-service   ClusterIP      172.19.163.5     <none>          27017/TCP        9d
```

## 2.Create CICD toolkit from Calm

![CICD](./images/CICD.png)

### 2-1.[CALM] Upload "02_cicd-app.json" blueprint

```text
Blueprint Name: [YOURNAME]-cicd-app
Project: default
Password: nutanix/4u
```

![AppUpload](./images/AppUpload.png)

```shell
Note: Don't launch the app for now, this app is deployed by Jenkins at later step.
```

### 2-2.[CALM] Upload "03_cicd-base.json" blueprint

```text
Blueprint Name: [YOURNAME]-cicd-base
Project: default
Password: nutanix/4u

Services:

(Option, you could choose image you have download in Chapter Prerequisites)
Developer Workstation --> VM --> DISKS (1) --> Image --> Select image you created
Jenkins Slave --> VM --> DISKS (1) --> Image --> Select image you created
Jenkins Master --> VM --> DISKS (1) --> Image --> Select image you created
Docker Registry --> VM --> DISKS (1) --> Image --> Select image you created
Artifactory --> VM --> DISKS (1) --> Image --> Select image you created
Gitolite --> VM --> DISKS (1) --> Image --> Select image you created

(Select Network for VMs)
Developer Workstation --> VM --> NETWORK ADAPTERS (NICS) (1) --> NIC1 --> Select Karbon-Network
Jenkins Slave --> VM --> NETWORK ADAPTERS (NICS) (1) --> NIC1 --> Select Karbon-Network
Jenkins Master --> VM --> NETWORK ADAPTERS (NICS) (1) --> NIC1 --> Select Karbon-Network
Docker Registry --> VM --> NETWORK ADAPTERS (NICS) (1) --> NIC1 --> Select Karbon-Network
Artifactory --> VM --> NETWORK ADAPTERS (NICS) (1) --> NIC1 --> Select Karbon-Network
Gitolite --> VM --> NETWORK ADAPTERS (NICS) (1) --> NIC1 --> Select Karbon-Network

Credentials:

- Prism Central User --> Fill out your Prism Central admin user password
- Docker Registry Credential --> password --> nutanix/4u
- Jenkins Key --> using private key in Chapter Prerequisites
- Nutanix (Default) --> using private key in Chapter Prerequisites

If you use private key yourself, remeber to update public_key in Application Profile Default
```

![BaseUpload](./images/BaseUpload.png)
![BaseServices](./images/BaseServices.png)
![BaseCredentials](./images/BaseCredentials2.png)

### 2-3.[CALM] Launch cicd-base application

```text
Push Launch Button
Name of the Application: [YOURNAME]-cicd-base

Profile configuration:
- Blueprint Name: [YOURNAME]-cicd-app
- Karbon Cluster Name: Your Karbon cluster name
- Prism Central IP Address: Your PC address
- yourname: [YOURNAME]
```

![BaseProfile](./images/BaseProfile.png)


```text
Note: 20-30mins to boot up
```

### 2-4.[CALM] Check CICD environment

```text
Application --> [YOURNAME]-cicd-base --> Audit --> Create --> OneTimeWorkFlows Start --> Output Connection Details

Jenkins Master Details: memo the output
Docker Registry Details: memo the output
Artifactory Details: memo the output
Developer Workstation Details: memo the output
```

![BaseConnection](./images/BaseConnection.png)
![BaseJenkins](./images/BaseJenkins.png)
![BaseDocker](./images/BaseDocker.png)
![BaseArtifactory](./images/BaseArtifactory.png)
![BaseDeveloperWS](./images/BaseDeveloperWS.png)

### 2-5.[Browser] Access Jenkins

```text
Access to Jenkins Master URL as indicated at "Jenkins Master URL"
Username: admin
Password: Shown at "Initial Authorization Password"
```

![BaseJenkinsUI](./images/BaseJenkinsUI.png)

```text
You see 2 pipelines, one is devops, the other is devops_deploy

devops: main pipeline to build container images, unit/integration test, call devops_deploy pipeline to deploy and cleanup
devops_deploy: call calm plugin to deploy the container images using [YOURNAME]-cicd-app blueprint
```

![BaseJenkinsUI2](./images/BaseJenkinsUI2.png)

### 2-6.[Browser] Access Docker Registry

```text
Access to Docker Registry URL as indicated at "Docker Registry URLs"
Username: nutanix
Password: nutanix/4u
```

![BaseDockerUI](./images/BaseDockerUI.png)

### 2-7.[Browser] Access Artifactory

```text
Access to Artifactory URL as indicated at "Artifactory URL"
Username: admin
Password: password
```

![BaseArtifactoryUI](./images/BaseArtifactoryUI.png)
![BaseArtifactoryUI2](./images/BaseArtifactoryUI2.png)

### 2-8.[DevWSVM] Access Developer Workstation

Access to developer workstation, ssh to "Developer Workstaion IP Address" using ssh key, got ssh private key from **Chapter Prerequisites**

```shell
ssh -i key nutanix@"Developer Workstation IP Address"
```

Initial application code has been placed on this workstation. Move to devops directory and look around 

```shell
cd ~/devops
git status
git remote -v
```

The output should be something like this.

```text
[nutanix@dev-workstation-2193 devops]$ git status
# On branch master
#
# Initial commit
#
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#
#	Jenkinsfile
#	Makefile
#	app/
#	db/
#	docker-ansible/
#	docker/
#	target/
#	web/
nothing added to commit but untracked files present (use "git add" to track)
[nutanix@dev-workstation-2193 devops]$ 
[nutanix@dev-workstation-2193 devops]$ 
[nutanix@dev-workstation-2193 devops]$ git remote -v
origin	git@git-server-2598.apjsme.local:devops.git (fetch)
origin	git@git-server-2598.apjsme.local:devops.git (push)
[nutanix@dev-workstation-2193 devops]$ 
```

## 3.Application deployment

![App](./images/App.png)

### 3-1.[KubeVM] Create services for nginx and nodejs pods

```shell
kubectl apply -f 04_nginx-calm-lb-service.yaml -n [YOURNAME]
kubectl apply -f 05_nodejs-calm-lb-service.yaml -n [YOURNAME]
kubectl get services -n [YOURNAME]
kubectl get services nodejs-calm-lb-service -n [YOURNAME] -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
--> memo the ip address as "NODEJS IP address"
kubectl get services nginx-calm-lb-service -n [YOURNAME] -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
--> memo the ip address as "NGINX IP address"
```

## 3-2.[CALM] Modify [YOURNAME]-cicd-app blueprint

```shell
Open [YOURNAME]-cicd-app blueprint

Pod --> NginxPod --> Deployment
Account: [Karbon provider name]
Pod --> NodeJSPod --> Deployment
Account: [Karbon provider name]

Credentials
- Nutanix (Default) - using same private key (got from Chapter Prerequisites or yourself)
- Prism Central User --> Fill out your Prism Central admin user password
```

![AppNginxk8s](./images/AppNginxk8s.png)
![AppNodejsk8s](./images/AppNodejsk8s.png)
![AppCredentials](./images/AppCredentials.png)

```text
Application Profile: Default --> Variables:
mgmtvm_address: "Developer Workstation IP Address" fetched at 2-4
pc_instance_ip: Prism Central IP Address
karbon_cluster_name: Your Karbon cluster name
nodejs_ip: IP address of NODEJS address fetched at 3-1
yourname: [YOURNAME]

Push Save button

```

![AppVariables](./images/AppVariables.png)

```shell
Note: You can ignore the 15 Warnings.
```

![AppWarnings](./images/AppWarnings.png)

## 3-3.[DevWSVM] Push application code to SCM server to trigger CICD pipeline

```shell
cd ~/devops
git status
git add .
git commit -m "My first commit"
git push origin master
```

The output should be like this.

```text
[nutanix@dev-workstation-2193 devops]$ git status
# On branch master
#
# Initial commit
#
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#
#	Jenkinsfile
#	Makefile
#	app/
#	db/
#	docker-ansible/
#	docker/
#	target/
#	web/
nothing added to commit but untracked files present (use "git add" to track)
[nutanix@dev-workstation-2193 devops]$ 
[nutanix@dev-workstation-2193 devops]$ 
[nutanix@dev-workstation-2193 devops]$ 
[nutanix@dev-workstation-2193 devops]$ git add .
[nutanix@dev-workstation-2193 devops]$ 
[nutanix@dev-workstation-2193 devops]$ 
[nutanix@dev-workstation-2193 devops]$ git commit -m "My first commit"
[master (root-commit) 2721a8f] My first commit
 35 files changed, 11798 insertions(+)
 create mode 100644 Jenkinsfile
 create mode 100644 Makefile
 create mode 100644 app/docker/dev/Dockerfile
 create mode 100644 app/docker/dev/docker-compose.yml
 create mode 100755 app/src/app.js
 create mode 100644 app/src/models/peakModel.js
 create mode 100644 app/src/package.json
 create mode 100644 app/src/routes/infoRoutes.js
 create mode 100644 app/src/routes/peakRoutes.js
 create mode 100644 app/tests/package.json
 create mode 100644 app/tests/test.js
 create mode 100644 db/mongo/data/collections.json
 create mode 100644 db/mongo/docker/dev/Dockerfile
 create mode 100644 db/mongo/docker/dev/docker-compose.yml
 create mode 100644 docker-ansible/src/Dockerfile
 create mode 100644 docker-ansible/src/ansible/probe.yml
 create mode 100644 docker/dev/Dockerfile
 create mode 100644 docker/dev/docker-compose.yml
 create mode 100644 docker/release/docker-compose.yml
 create mode 100644 target/seed
 create mode 100644 web/bin/nginx-startup.sh
 create mode 100644 web/docker/dev/Dockerfile
 create mode 100644 web/docker/dev/docker-compose.yml
 create mode 100644 web/src/css/bootstrap-grid.min.css
 create mode 100644 web/src/css/bootstrap.min.css
 create mode 100644 web/src/css/style.css
 create mode 100644 web/src/images/Nutanix-neg.png
 create mode 100644 web/src/images/Nutanix_X.png
 create mode 100644 web/src/images/nutanix-x-2.png
 create mode 100644 web/src/images/nutanix_logo.png
 create mode 100644 web/src/images/search.png
 create mode 100644 web/src/index.html
 create mode 100644 web/src/js/bootstrap.bundle.min.js
 create mode 100644 web/src/js/data.js
 create mode 100644 web/src/js/jquery-3.2.1.js
[nutanix@dev-workstation-2193 devops]$ 
[nutanix@dev-workstation-2193 devops]$ 
[nutanix@dev-workstation-2193 devops]$ 
[nutanix@dev-workstation-2193 devops]$ git push origin master
Counting objects: 64, done.
Compressing objects: 100% (52/52), done.
Writing objects: 100% (64/64), 261.85 KiB | 0 bytes/s, done.
Total 64 (delta 4), reused 0 (delta 0)
To git@git-server-2598.apjsme.local:devops.git
 * [new branch]      master -> master
```

## 3-4.[Jenkins] Check CI/CD pipelines

Click devops pipeline, you should see whole pipeline is loaded and executed.

![AppPipeline](./images/AppPipeline.png)

Click devops_deploy pipeline, you should see deployment task is completed.

![AppPipeline2](./images/AppPipeline2.png)

## 3-5.[KubeVM] Check kubernetes cluster

```shell
kubectl get deploy,po,svc,ep -n [YOURNAME]
```

The output should be someting like this.

```text
$ kubectl get deploy,po,svc,ep -n shuchida
NAME                                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/k8s-mongodb-deployment    1/1     1            1           53m
deployment.extensions/k8s-nginx-deployment-1    2/2     2            2           4m46s
deployment.extensions/k8s-nodejs-deployment-1   1/1     1            1           4m47s

NAME                                           READY   STATUS    RESTARTS   AGE
pod/k8s-mongodb-deployment-7b4df64bb7-n9wp7    1/1     Running   0          53m
pod/k8s-nginx-deployment-1-5988ccbcb8-9dqsg    1/1     Running   0          4m46s
pod/k8s-nginx-deployment-1-5988ccbcb8-t7k95    1/1     Running   0          4m46s
pod/k8s-nodejs-deployment-1-58c98f7655-zb8mf   1/1     Running   0          4m47s

NAME                              TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)          AGE
service/mongodb-calm-lb-service   ClusterIP      172.19.163.5     <none>          27017/TCP        52m
service/nginx-calm-lb-service     LoadBalancer   172.19.141.183   10.139.81.162   80:30436/TCP     59m
service/nodejs-calm-lb-service    LoadBalancer   172.19.117.109   10.139.81.163   3000:30983/TCP   59m

NAME                                ENDPOINTS                       AGE
endpoints/mongodb-calm-lb-service   172.20.1.100:27017              52m
endpoints/nginx-calm-lb-service     172.20.1.14:80,172.20.1.15:80   59m
endpoints/nodejs-calm-lb-service    172.20.1.13:3000                59m
```

## 3-6.[Browser] Access the application from browser

Access the NGINX ip address from web browser fetched at 3-1. Push Search button to display the peak rank.

![AppUI](./images/AppUI.png)

## 3-7.[CALM] Check an application is instanciated

Check Application field and you'll see something like devopsXXXXJenkins1. This app is created from [YOURNAME]-cicd-app blueprint launched by Jenkins Calm Plugin.

![AppApplication](./images/AppApplication.png)

## 3-8.[DevWSVM] Modify application code and push to SCM server to trigger CICD pipeline

```shell
cd ~/devops
sed -i "s/4C4C4E/800080/g" web/src/css/style.css
git status
git diff
git commit -am "Changed the color to #800080"
git push origin master
```

The output should be something like this.

```text
[nutanix@dev-workstation-2193 devops]$ 
[nutanix@dev-workstation-2193 devops]$ sed -i "s/4C4C4E/800080/g" web/src/css/style.css
[nutanix@dev-workstation-2193 devops]$ 
[nutanix@dev-workstation-2193 devops]$ 
[nutanix@dev-workstation-2193 devops]$ 
[nutanix@dev-workstation-2193 devops]$ git status
# On branch master
# Changes not staged for commit:
#   (use "git add <file>..." to update what will be committed)
#   (use "git checkout -- <file>..." to discard changes in working directory)
#
#	modified:   web/src/css/style.css
#
no changes added to commit (use "git add" and/or "git commit -a")
[nutanix@dev-workstation-2193 devops]$ 
[nutanix@dev-workstation-2193 devops]$ 
[nutanix@dev-workstation-2193 devops]$ git diff
diff --git a/web/src/css/style.css b/web/src/css/style.css
index 925bb70..35f6132 100644
--- a/web/src/css/style.css
+++ b/web/src/css/style.css
@@ -20,7 +20,7 @@
 }
 
 .server-info {
-       background: #4C4C4E;
+       background: #800080;
        color: #fff;
        line-height 28px;
 }
[nutanix@dev-workstation-2193 devops]$ 
[nutanix@dev-workstation-2193 devops]$ 
[nutanix@dev-workstation-2193 devops]$ 
[nutanix@dev-workstation-2193 devops]$ git commit -am "Changed the color to #800080"
[master cfa2ed6] Changed the color to #800080
 1 file changed, 1 insertion(+), 1 deletion(-)
[nutanix@dev-workstation-2193 devops]$ 
[nutanix@dev-workstation-2193 devops]$ 
[nutanix@dev-workstation-2193 devops]$ git push origin master
Counting objects: 11, done.
Compressing objects: 100% (6/6), done.
Writing objects: 100% (6/6), 570 bytes | 0 bytes/s, done.
Total 6 (delta 4), reused 0 (delta 0)
To git@git-server-2598.apjsme.local:devops.git
   2721a8f..cfa2ed6  master -> master
```

## 3-9.[Jenkins] Check CI/CD pipelines

Click devops pipeline, you should see whole pipeline is loaded and executed.

![AppPipeline3](./images/AppPipeline3.png)

Click devops_deploy pipeline, you should see deployment task is completed.

![AppPipeline4](./images/AppPipeline4.png)

## 3-10.[KubeVM] Check kubernetes cluster

```shell
kubectl get deploy,po,svc,ep -n [YOURNAME]

The output should be someting like this.
Note new pods are deployed, named XXXXXX-2.

$ kubectl get deploy,po,svc,ep -n shuchida
NAME                                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/k8s-mongodb-deployment    1/1     1            1           9d
deployment.extensions/k8s-nginx-deployment-1    2/2     2            2           12m
deployment.extensions/k8s-nginx-deployment-2    2/2     2            2           3m58s
deployment.extensions/k8s-nodejs-deployment-1   1/1     1            1           12m
deployment.extensions/k8s-nodejs-deployment-2   1/1     1            1           3m59s

NAME                                          READY   STATUS    RESTARTS   AGE
pod/k8s-mongodb-deployment-7b4df64bb7-n9wp7   1/1     Running   0          9d
pod/k8s-nginx-deployment-1-7fcdf85ddf-brwps   1/1     Running   0          12m
pod/k8s-nginx-deployment-1-7fcdf85ddf-sp2l6   1/1     Running   0          12m
pod/k8s-nginx-deployment-2-56f477b5b7-szjk2   1/1     Running   0          3m58s
pod/k8s-nginx-deployment-2-56f477b5b7-tc6sr   1/1     Running   0          3m58s
pod/k8s-nodejs-deployment-1-6b7ddc6c4-nfjfv   1/1     Running   0          12m
pod/k8s-nodejs-deployment-2-7c88b7456-snvrc   1/1     Running   0          3m59s

NAME                              TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)          AGE
service/mongodb-calm-lb-service   ClusterIP      172.19.163.5     <none>          27017/TCP        9d
service/nginx-calm-lb-service     LoadBalancer   172.19.141.183   10.139.81.162   80:30436/TCP     9d
service/nodejs-calm-lb-service    LoadBalancer   172.19.117.109   10.139.81.163   3000:30983/TCP   9d

NAME                                ENDPOINTS                       AGE
endpoints/mongodb-calm-lb-service   172.20.1.100:27017              9d
endpoints/nginx-calm-lb-service     172.20.1.47:80,172.20.1.48:80   9d
endpoints/nodejs-calm-lb-service    172.20.1.46:3000                9d
```

## 3-11.[Browser] Access the application from browser

Access the NGINX ip address from web browser fetched at 3-1, the color of server info pane changed to purple. Push Search button to display the rank.

![AppUI2](./images/AppUI2.png)

## 3-12.[CALM] Check an application is instanciated

Check Application field and you'll see another app named like devopsXXXXJenkins2. This app is created from [YOURNAME]-cicd-app blueprint kicked from Jenkins Calm Plugin.

![AppApplication2](./images/AppApplication2.png)

