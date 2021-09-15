# Objectives
We will secure the CityApp by using Secretless Broker

To get a better understanding of Secretless Broker, please review [Secretless Broker online doc](https://secretless.io/)

## 1. Prerequisites
The following steps were performed in task 06
- Follower certificate is applied in ConfigMap
- Conjur policies are applied
- AKS cluster role bindings are applied to allow follower to validate pods in cityapp namespace
## 2. Apply secretless configuration
Download secretless.yaml and apply to AKS ConfigMap
```
wget https://github.com/rajnishgargcloudrepository/conjur-aks/raw/main/task07/secretless.yaml
kubectl create configmap cityapp-secretless-config --from-file=secretless.yaml -n cityapp
```
## 3. Deploy CityApp (with Secretless Broker)
Download the cityapp-summon-init.yaml manifest file

Please review the kubernetes manifest file and make necessary changes if your environment is different to this lab before loading it to AKS
```console
wget https://github.com/rajnishgargcloudrepository/conjur-aks/raw/main/task07/cityapp-secretless.yaml
```
- Update the in the Kubernetes manifest file before applying.
- Create the CityApp (with Secretless Broker) deployment in AKS cluster
```console
kubectl apply -f cityapp-secretless.yaml -n cityapp
```
## 4. Verify that the CityApp (with Secretless Broker) is running successfully
1. Verify CityApp (with Summon) pods
```console
kubectl get pod -n cityapp
```
Sample output:
```console
azureuser@VM-ConjurDemoAKS:~$ kubectl get pods -n cityapp
NAME                                   READY   STATUS    RESTARTS   AGE
cityapp-hardcode-75d696d656-hz7gh      1/1     Running   0          19h
cityapp-secretless-69648ffdd6-np8kk    2/2     Running   0          10m
cityapp-summon-init-589578596c-b8fcq   1/1     Running   0          75m
follower-6b5d667865-bqggw              1/1     Running   0          123m
```
2. Use curl to verify the random city
```
CITYAPP_SECRETLESS_POD_NAME="$(kubectl get pods -n cityapp | grep cityapp-secretless | head -n1 | awk '{print $1}')"
kubectl exec -it $CITYAPP_SECRETLESS_POD_NAME -c cityapp -n cityapp -- curl http://127.0.0.1:3000
```
Sample output:
```console
azureuser@VM-ConjurDemoAKS:~$ kubectl exec -it $CITYAPP_SECRETLESS_POD_NAME -c cityapp -- curl http://127.0.0.1:3000
<title> Random World Cities! </title>
<br><br>
<p style="font-size:30px"><b>Giza</b> is a city in Giza, Egypt with a population of 2221868
<br><br><br><p>
<small>Connected to database world on 127.0.0.1:3306 using username:  and password: </small>
```
