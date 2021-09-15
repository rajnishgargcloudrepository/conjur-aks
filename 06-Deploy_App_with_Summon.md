# Objectives

We will secure the CityApp by using Summon to inject secrets into environment variables.

The Summon container image is included in the CityApp pod as an init container.

## Load Conjur policy files

We will load the following Conjur policy files to provision the necessary parameters
1. `projects-authn.yaml`
    Creates the `conjur/authn-k8s/aks/apps` policy and creates Summon and Secretless (task 07) authenticators under this policy
2. `app-identity.yaml`
    Creates the `k8s-apps/default` policy and layer
    The authenticators created in `projects-authn.yaml` is added to the `k8s-apps/default` layer
3. `safe-permission.yaml`
    Creates the `world_db` policy and creates variables `username` and `password`, as well as a group `consumers` under this policy
    The `world_db/consumers` group is grated read and execute permissions on the variables
    The `k8s-apps/default` layer created in `app-identity.yaml` is added to the `world_db/consumers` group
- Download the policy files:
```console
wget https://github.com/rajnishgargcloudrepository/conjur-aks/raw/main/task06/projects-authn.yaml
wget https://github.com/rajnishgargcloudrepository/conjur-aks/raw/main/task06/app-identity.yaml
wget https://github.com/rajnishgargcloudrepository/conjur-aks/raw/main/task06/safe-permission.yaml
```
- Load the policy files into Conjur
```console
conjur policy load -f projects-authn.yaml -b root
conjur policy load -f app-identity.yaml -b root
conjur policy load -f safe-permission.yaml -b root
```
## 2 Populate the variables with the username and password of the MySQL database
```console
conjur variable set -i world_db/username -v cityapp
conjur variable set -i world_db/password -v Cyberark1
```
## 3. Create ConfigMap for the follower certificate
The Summon init container validates the follower service using the follower certificate
```console
FOLLOWER_POD_NAME="$(kubectl get pods -n conjur | grep follower | head -n1 | awk '{print $1}')"
kubectl exec -it $FOLLOWER_POD_NAME -c conjur -n conjur -- cat /opt/conjur/etc/ssl/conjur.pem > follower-certificate.pem
kubectl create configmap follower-certificate --from-file=ssl-certificate=<(cat follower-certificate.pem)
```
## 4. Load the secrets yaml file as AKS cluster ConfigMap in the cityapp namespace
```console
wget https://github.com/rajnishgargcloudrepository/conjur-aks/raw/main/task06/secrets.yaml
kubectl create configmap cityapp-summon-init-config --from-file=secrets.yaml -n cityapp
```
## 5. Deploy the CityApp Summon to the AKS Cluster
1. Download cityapp-summon-init.yaml.

Please review the kubernetes manifest file and make necessary changes if your environment is different to this lab before loading it to AKS
```console
wget https://github.com/rajnishgargcloudrepository/conjur-aks/raw/main/task06/cityapp-summon-init.yaml
```
- Update the <ACR-name> in the Kubernetes manifest file before applying.
- Create the CityApp (with Summon) deployment in AKS cluster
```console
kubectl apply -f cityapp-summon-init.yaml -n cityapp
```
## 6. Verify that the CityApp (with Summon) is running successfully
1. Verify CityApp (with Summon) pods
```console
kubectl get pod -n cityapp
```
Sample output:
```console
azureuser@VM-ConjurDemoAKS:~$ kubectl get pods -n cityapp
NAME                                   READY   STATUS    RESTARTS   AGE
cityapp-hardcode-75d696d656-hz7gh      1/1     Running   0          18h
cityapp-summon-init-589578596c-b8fcq   1/1     Running   0          6m5s
follower-6b5d667865-bqggw              1/1     Running   0          54m
```
2. Use curl to verify the random city
```
CITYAPP_SUMMON_POD_NAME="$(kubectl get pods -n cityapp | grep cityapp-summon-init | head -n1 | awk '{print $1}')"
kubectl exec -it $CITYAPP_SUMMON_POD_NAME -n cityapp -- curl http://127.0.0.1:3000
```
Sample output:
```console
azureuser@VM-ConjurDemoAKS:~$ kubectl exec -it $CITYAPP_SUMMON_POD_NAME -c cityapp -n cityapp -- curl -k http://127.0.0.1:3000
Defaulted container "cityapp" out of: cityapp, authenticator (init)
<title> Random World Cities! </title>
<br><br>
<p style="font-size:30px"><b>Campos dos Goytacazes</b> is a city in Rio de Janeiro, Brazil with a population of 398418
<br><br><br><p>
<small>Connected to database world on mysql.conjur.demo:3306 using username: cityapp and password: Cyberark1</small>
```
