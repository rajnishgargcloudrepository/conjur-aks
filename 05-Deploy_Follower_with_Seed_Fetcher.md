# Objectives

Deploy Conjur Followers to your AKS Cluster Node.

We will deploy Conjur Followers using seed-fetcher, which automatically authenticate and retrieve seed on pod start-up.

This allows self-healing and auto scaling of follower pods.

We will also enable k8s authenicator to support the following labs.

## 1. Push the Conjur container images to ACR

We need to upload the container images to ACR so that it can be used by AKS
```console
az acr login -n <ACR-name>
docker tag registry.tld/conjur-appliance:12.2.0 <ACR-name>.azurecr.io/conjur-appliance:12.2.0
docker tag cyberark/dap-seedfetcher:0.2.0 <ACR-name>.azurecr.io/dap-seedfetcher:0.2.0
docker push <ACR-name>.azurecr.io/conjur-appliance:12.2.0
docker push <ACR-name>.azurecr.io/dap-seedfetcher:0.2.0
```

## 2. Create follower namespace and service account
1. Log in to the utilities host
```console
kubectl create namespace conjur
```
2. Create service account for conjur follower deployment 
```console
kubectl create serviceaccount conjur-cluster -n conjur
```
## 3. Load Conjur policies
1. Download authn-k8s-cluster.yaml
- Please review the conjur policy file and make necessary changes if your environment is different to this lab before loading it to conjur
```console
wget https://github.com/rajnishgargcloudrepository/conjur-aks/raw/main/task05/authn-k8s.yaml
```
2. Load the authn-k8s-cluster.yaml to Conjur Master
```console
conjur policy load -f authn-k8s.yaml -b root
```
Sample output:
```console
azureuser@VM-ConjurDemoAKS:~$ conjur policy load -f authn-k8s.yaml -b root
{
    "created_roles": {
        "cyberark:host:conjur/authn-k8s/aks/apps/default/service_account/conjur-cluster": {
            "id": "cyberark:host:conjur/authn-k8s/aks/apps/default/service_account/conjur-cluster",
            "api_key": "q961ne183q6j637nvqsp3gyk4rwecqtyt10595e8bpf5rypxha8x"
        }
    },
    "version": 1
}
```
## 4. Initialize Conjur internal CA
1. Initialize the Conjur internal CA that will be used for the Kubernetes authenticator
```console
docker exec conjur-appliance chpst -u conjur conjur-plugin-service possum rake authn_k8s:ca_init["conjur/authn-k8s/aks"]
```
Sample output:
```console
Populated CA and Key of service conjur/authn-k8s/aks
To print values:
 conjur variable value conjur/authn-k8s/aks/ca/cert
 conjur variable value conjur/authn-k8s/aks/ca/key
```
2. Access Conjur Master UI and verify that `conjur/authn-k8s/aks/ca/cert` and `conjur/authn-k8s/aks/ca/key` are populated with the certificate and key values
## 5. Enable Kubernetes authenticator on Conjur Master
1. Add `CONJUR_AUTHENTICATORS="authn,authn-k8s/okd"` to `/opt/conjur/etc/conjur.conf` in Conjur container
```console
docker exec conjur-appliance sed -i -e '$aCONJUR_AUTHENTICATORS="authn,authn-k8s/aks"' /opt/conjur/etc/conjur.conf
```
2. Restart conjur service to apply this change.
```console
docker exec conjur-appliance sv restart conjur
```
3. Verify that okd authenticator is now enabled on Master
```console
curl -k https://master.conjur.demo/info
```
Sample output:
```console
    "enabled": [
      "authn",
      "authn-k8s/aks"
    ]
```
## 6. Create cluster role and role binding for conjur-cluster service account
1. Download conjur-authenticator-permissions.yaml
- Please review the role binding file and make necessary changes if your environment is different to this lab before loading it to AKS
```console
wget https://github.com/rajnishgargcloudrepository/conjur-aks/raw/main/task05/conjur-authenticator-permissions.yaml
```
2. Apply the Kubernetes manifest file with kubectl
```console
kubectl apply -f conjur-authenticator-permissions.yaml
```
## 7. Configure AKS cluster API details in Conjur
- Retrieve the AKS cluster API details
```console
TOKEN_SECRET_NAME="$(kubectl get secrets | grep 'conjur.*service-account-token' | head -n1 | awk '{print $1}')"
CA_CERT="$(kubectl get secret $TOKEN_SECRET_NAME -o json | jq -r '.data["ca.crt"]' | base64 --decode)"
SERVICE_ACCOUNT_TOKEN="$(kubectl get secret $TOKEN_SECRET_NAME -o json | jq -r .data.token | base64 --decode)"
API_URL="$(kubectl config view --minify -o json | jq -r '.clusters[0].cluster.server')"
```
- Verify the values of the environment variables
```console
echo $TOKEN_SECRET_NAME
echo $CA_CERT
echo $SERVICE_ACCOUNT_TOKEN
echo $API_URL
```
- Load the values to Conjur variables
```console
conjur variable set -i conjur/authn-k8s/aks/kubernetes/ca-cert -v "$CA_CERT"
conjur variable set -i conjur/authn-k8s/aks/kubernetes/service-account-token -v "$SERVICE_ACCOUNT_TOKEN"
conjur variable set -i conjur/authn-k8s/aks/kubernetes/api-url -v "$API_URL"
```
Sample output:
```console
azureuser@VM-ConjurDemoAKS:~$ conjur variable set -i conjur/authn-k8s/aks/kubernetes/ca-cert -v "$CA_CERT"
Successfully set value for variable 'conjur/authn-k8s/aks/kubernetes/ca-cert'
azureuser@VM-ConjurDemoAKS:~$ conjur variable set -i conjur/authn-k8s/aks/kubernetes/service-account-token -v "$SERVICE_ACCOUNT_TOKEN"
Successfully set value for variable 'conjur/authn-k8s/aks/kubernetes/service-account-token'
azureuser@VM-ConjurDemoAKS:~$ conjur variable set -i conjur/authn-k8s/aks/kubernetes/api-url -v "$API_URL"
Successfully set value for variable 'conjur/authn-k8s/aks/kubernetes/api-url'
```
## 8. Add Conjur Master certificate to config map
The seedfetcher will use ConfigMap value to validate the Conjur Master
```console
openssl s_client -showcerts -connect master.conjur.demo:443 </dev/null 2>/dev/null | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > master-certificate.pem
kubectl create configmap master-certificate --from-file=ssl-certificate=<(cat master-certificate.pem)
```
## 9. Deploy the Follower with seedfetcher
1. Download follower-with-seedfetcher.yaml.
- Please review the kubernetes manifest file and make necessary changes if your environment is different to this lab before loading it to AKS
```console
wget https://github.com/rajnishgargcloudrepository/conjur-aks/raw/main/task05/follower-with-seedfetcher.yaml
```
2. Update the `<ACR-name>` in the Kubernetes manifest file before applying
3. Apply the Kubernetes manifest file with kubectl
```console
kubectl apply -f follower-with-seedfetcher.yaml -n conjur
```
Sample output:
```console
azureuser@VM-ConjurDemoAKS:~$ kubectl apply -f follower-with-seedfetcher.yaml
service/follower created
deployment.apps/follower created
```
4. Verify that the pods are running
```console
azureuser@VM-ConjurDemoAKS:~$ kubectl get pods -n conjur
NAME                                READY   STATUS    RESTARTS   AGE
follower-6b5d667865-bqggw           1/1     Running   0          2m6s
```
5. Verify that follower replication is healthy
```console
curl -k https://master.conjur.demo/health
```
Sample output:
```console
    "replication_status": {
      "pg_stat_replication": [
        {
          "usename": "follower.default.svc.cluster.local",
          "application_name": "follower_follower_default_svc_cluster_local_follower_6b5",
          "client_addr": "10.240.0.4",
          "backend_start": "2021-09-15 05:51:09 +0000",
          "state": "streaming",
          "sent_lsn": "0/3000060",
          "replay_lsn": "0/3000060",
          "sync_priority": 0,
          "sync_state": "async",
          "sent_lsn_bytes": 50331744,
          "replay_lsn_bytes": 50331744,
          "replication_lag_bytes": 0
        }
      ],
      "pg_current_xlog_location": "0/3000060",
      "pg_current_xlog_location_bytes": 50331744
    }
```
