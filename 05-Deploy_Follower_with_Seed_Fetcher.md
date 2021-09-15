# Objectives

Deploy Conjur Followers to your AKS Cluster Node.

We will deploy Conjur Followers using seed-fetcher, which automatically authenticate and retrieve seed on pod start-up.

This allows self-healing and auto scaling of follower pods.

We will also enable k8s authenicator to support the following labs.

## 1 Push the Conjur container images to ACR

We need to upload the container images to ACR so that it can be used by AKS
```console
az acr login -n <ACR-name>
docker tag registry.tld/conjur-appliance:12.2.0 <ACR-name>.azurecr.io/conjur-appliance:12.2.0
docker tag cyberark/dap-seedfetcher:0.2.0 <ACR-name>.azurecr.io/dap-seedfetcher:0.2.0
docker push <ACR-name>.azurecr.io/conjur-appliance:12.2.0
docker push <ACR-name>.azurecr.io/dap-seedfetcher:0.2.0
```

## 2 Create follower namespace and service account
1. Log in to the utilities host
```console
kubectl create namespace conjur
```
2. Create service account for conjur follower deployment 
```console
kubectl create serviceaccount conjur-cluster -n conjur
```
## 3 Load Conjur policies
1. Download authn-k8s-cluster.yaml. Please review the conjur policy file authn-k8s-cluster.yaml and make necessary changes if your environment is different to this lab before loading it to conjur
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
