# SE Team training Calm 201 Lab

## Prerequisites
- Kubernetes cluster from Karbon1.0.3  
Metallb configured

- Calm2.9  
Project(default) and Provider(Karbon) configured  

- Workspace VM  
kubectl  
kubeconfig  
watch  
git  
sshkey file  

- Variables  
YOURNAME: Your Unique Name without spaces (ex: shuchida)

## 0.[WSVM] Create namespace
```shell
kubectl create ns [YOURNAME]
```

## 0.[CALM] Login to Calm and upload "cicd-mongo.json" blueprint
```shell
Blueprint Name: [YOURNAME]-cicd-mongo
Project: default
Password: nutanix/4u
```

## 1.[CALM] Launch mongodb
```shell
Push Launch button
Name of the Application: [YOURNAME]-cicd-mongo
yourname: [YOURNAME]
Push Create button
```

## 2.[WSVM] Check mongodb deployment, pod and service are created
```shell
kubectl get deploy,po,svc -n [YOURNAME]
```

## 3.[CALM] Upload "cicd-app.json" blueprint
```shell
Blueprint Name: [YOURNAME]-cicd-app
Project: default
Password: nutanix/4u
```

## 4.[CALM] Upload "cicd-base.json" blueprint
```shell
Blueprint Name: [YOURNAME]-cicd-base
Project: default
Password: nutanix/4u

Services --> Developer Workstation --> VM --> NETWORK ADAPTERS (NICS) (1) --> NIC1 --> Select XXXXX
Services --> Jenkins Slave --> VM --> NETWORK ADAPTERS (NICS) (1) --> NIC1 --> Select XXXXX
Services --> Jenkins Master --> VM --> NETWORK ADAPTERS (NICS) (1) --> NIC1 --> Select XXXXX
Services --> Docker Registry --> VM --> NETWORK ADAPTERS (NICS) (1) --> NIC1 --> Select XXXXX
Services --> Artifactory --> VM --> NETWORK ADAPTERS (NICS) (1) --> NIC1 --> Select XXXXX
Services --> Gitolite --> VM --> NETWORK ADAPTERS (NICS) (1) --> NIC1 --> Select XXXXX
```

## 5.[CALM] Launch cicd-base application
```shell
Push Launch Button
Name of the Application: [YOURNAME]-cicd-base
Blueprint Name: [YOURNAME]-cicd-app
Karbon Cluster Name: XXXXXX
Prism Central IP Address: Your PC address
yourname: [YOURNAME]
Push Create Button
```
takes 20-30mins to boot up

## 6.[CALM] Check CICD environment
```shell
Application --> [YOURNAME]-cicd-base --> Audit --> Create --> OneTimeWorkFlows Start --> Output Connection Details

Jenkins Master Details: memo the output
Docker Registry Details: memo the output
Artifactory Details: memo the output
Developer Workstation Details: memo the output
```

## 7.[Browser] Access Jenkins
```shell
Access to Jenkins Master URL as indicated at "Jenkins Master URL"
Username: admin
Password: Shown at "Initial Authorization Password"

You see 2 pipelines, one is devops, the other is devops_deploy
devops: main pipeline to build container images, unit/integration test, call devops_deploy pipeline to deploy and cleanup
devops_deploy: call calm plugin to deploy the container images using [YOURNAME]-cicd-app blueprint
```

## 8.[Browser] Access Docker Registry
```shell
Access to Docker Registry URL as indicated at "Docker Registry URLs"
Username: nutanix
Password: nutanix/4u
```

## 9.[Browser] Access Artifactory
```shell
Access to Artifactory URL as indicated at "Artifactory URL"
Username: admin
Password: password
```

## 10.[WSVM-->DevWSVM] Access Developer Workstation
```shell
Access to developer workstation, ssh to "Developer Workstaion IP Address" using ssh key
ssh -i calmkey nutanix@IP
```

## 11.[WSVM] Create services for nginx and nodejs
```shell
kubectl apply -f nginx-calm-lb-service.yaml -n [YOURNAME]
kubectl apply -f nodejs-calm-lb-service.yaml -n [YOURNAME]
kubectl get services -n [YOURNAME]
kubectl get services nodejs-calm-lb-service -n shuchida -o jsonpath='{.status.loadBalancer.ingress[0].ip}' --> memo the ip address as NODEJS address
kubectl get services nginx-calm-lb-service -n shuchida -o jsonpath='{.status.loadBalancer.ingress[0].ip}' --> memo the ip address as NGINX address
```

## 12.[CALM] Modify [YOURNAME]-cicd-app blueprint
```shell
Open [YOURNAME]-cicd-app blueprint --> Application Profile: Default --> Variables
mgmtvm_address: "Developer Workstation IP Address" shown at #10
pc_instance_ip: Prism Central IP Address
nodejs_ip: ip address of NODEJS address taken at #11
yourname: [YOURNAME]
Push Save Button
```

## 13.[DevWSVM] Push application code to SCM server to trigger CICD pipeline
```shell
cd ~/devops
git status
git add .
git commit -m "My first commit"
git push origin master
```

## 14.[Jenkins] Check CI/CD pipelines
```shell
Click devops pipeline, you should see whole pipeline is loaded and executed.
Also click devops_deploy pipeline, you should see deployment task is completed.
```

## 15.[WSVM] Check kubernetes cluster
```shell
kubectl get deploy,po,svc,ep -n [YOURNAME]

The output should be someting like this.

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

## 15.[Browser] Access the application from browser
```shell
Access the NGINX ip address from web browser.
Push Search button to display the rank.
```

## 16.[CALM] Check an application is instanciated
```shell
Check Application field and you'll see something like devopsXXXXJenkinsY.
This app is created from [YOURNAME]-cicd-app blueprint kicked from Jenkins Calm Plugin.
```

## 17.[DevWSVM] Modify application code and push to SCM server to trigger CICD pipeline
```shell
cd ~/devops
sed -i "s/4C4C4E/800080/g" web/src/css/style.css
git status
git diff
git commit -am "Changed the color to #800080"
git push origin master
```

## 18.[Jenkins] Check CI/CD pipelines
```shell
Click devops pipeline, you should see whole pipeline is loaded and executed.
Also click devops_deploy pipeline, you should see deployment task is completed.
```

## 19.[WSVM] Check kubernetes cluster
```shell
kubectl get deploy,po,svc,ep -n [YOURNAME]

The output should be someting like this.

$ kubectl get deploy,po,svc,ep -n shuchida

```

## 20.[Browser] Access the application from browser
```shell
Access the NGINX ip address from web browser, the server info pane changed the color to purple
Push Search button to display the rank.
```

## 21.[CALM] Check an application is instanciated
```shell
Check Application field and you'll see another app named like devopsXXXXJenkinsY.
This app is created from [YOURNAME]-cicd-app blueprint kicked from Jenkins Calm Plugin.
```
