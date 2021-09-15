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
1. Download authn-k8s-cluster.yaml. Please review the conjur policy file and make necessary changes if your environment is different to this lab before loading it to conjur
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
1. Download conjur-authenticator-permissions.yaml. Please review the role binding file and make necessary changes if your environment is different to this lab before loading it to AKS
```console
wget https://github.com/rajnishgargcloudrepository/conjur-aks/raw/main/task05/conjur-authenticator-permissions.yaml
```
2. Use kubectl to apply it.
```console
kubectl apply -f conjur-authenticator-permissions.yaml
```
