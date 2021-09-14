# Objectives

Get familiar with AKS by deploying a sample application to the AKS cluster.
The CityApp application connects to the MySQL database setup in task 03 and displays a random city name in the web application.
The MySQL database credentials specified in task 03 will be embedded in the kubernetes deployment configuration file.

# Use Docker to build the images in the utilities host
1. Login to the utilities host (setup in task 00)
2. Load the CityApp container image to Docker
```console
wget https://github.com/rajnishgargcloudrepository/conjur-aks/raw/main/task03/build.tar.gz
tar xvf build.tar.gz
cd build
./build.sh
```
3. Verify that the CityApp container image is successfully loaded
```console
docker images
```
Output Example
```console
REPOSITORY                         TAG       IMAGE ID       CREATED         SIZE
cityapp                            1.0       f157c356faf4   2 minutes ago   294MB
```
# Push the CityApp container image to ACR
We need to upload the container image to ACR so that it can be used by AKS
```console
az acr login -n <ACR-name>
docker tag cityapp:1.0 <ACR-name>.azurecr.io/cityapp:1.0
docker push <ACR-name>.azurecr.io/cityapp:1.0
```

# Create CityApp namespace in AKS cluster
Create a namespace to run all the CityApp deployments
```console
kubectl create namespace cityapp
```
# Deploy CityApp (embedded credentials) to AKS cluster
Download the CityApp Kubernetes configuration file
```console
wget https://raw.githubusercontent.com/rajnishgargcloudrepository/conjur-aks/main/task03/cityapp-harcode.yaml
```
Create the CityApp (embedded credentials) deployment in AKS cluster
```console
kubectl apply -f cityapp-hardcode.yaml
```
Sample output:
```console
$ kubectl apply -f cityapp-hardcode.yaml -n cityapp
serviceaccount/cityapp-hardcode created
service/cityapp-hardcode created
deployment.apps/cityapp-hardcode created
```
# Verify that the CityApp (embedded credentials) deployment is running successfully
1. Verify CityApp pod status
```console
kubectl get pods
```
Sample output:
```console
NAME                                READY   STATUS    RESTARTS   AGE
cityapp-hardcode-75d696d656-hz7gh   1/1     Running   0          13s
```
2. Verify that pod access to MySQL database
Get a shell in the CityApp pod and curl to itself
```
kubectl exec -it cityapp-hardcode-75d696d656-hz7gh -- /bin/sh
curl http://127.0.0.1:3000
```
Sample output:
```console
/usr/src # curl http://127.0.0.1:3000
<title> Random World Cities! </title>
<br><br>
<p style="font-size:30px"><b>Klin</b> is a city in Moskova, Russian Federation with a population of 90000
<br><br><br><p>
<small>Connected to database world on mysql.conjur.demo:3306 using username: cityapp and password: Cyberark1</small>
```
# Task 03 Completed
