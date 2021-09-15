# CyberArk Conjur Secrets Manager Azure AKS Integration Lab 2021
This is a tutorial share to you on how to secure secrets of Azure AKS applications by CyberArk Secrets Manager Conjur. We will cover deploying Conjur Master, Conjur follower instances with follower seed fetcher. Conjur Secretless Broker & init container will also be covered in this tutorial.
For more detail about CyberArk Conjur Secrets Manager, please visit the two websites

- [CyberArk Conjur Secrets Manager Enterprise](https://www.cyberark.com/products/secrets-manager-enterprise/)
- [CyberArk Conjur Online Docs](https://docs.cyberark.com/Product-Doc/OnlineHelp/AAM-DAP/Latest/en/Content/Get%20Started/WhatIsConjur.html)

## Lab Guide Version
- Version 1.0
- Release Date 10 October 2021

## Prerequisite
- You have basic understanding Linux installation and administration
- You have basic understanding Docker
- You have basic understanding MySQL Administration
- You have basic understanding Azure configuration and administration
  - For setup the lab, you better knew how to config Azure VM, VNet, AKS, Security Group and Azure ACR
- You have basic understanding Azure AKS Kubernetes setup and administration

## Lab Architecture
- AKS is used as platform to host the [demo app](https://github.com/jeepapichet/cityapp). The application will connect to a MySQL database to retreive data, and during authenication, [secrets](https://docs.cyberark.com/Product-Doc/OnlineHelp/AAM-DAP/Latest/en/Content/Get%20Started/key_concepts/secrets.html) will be used by the application
- The lab was based on Conjur Version 12.2.0

![Architecture](https://github.com/ivanckleecity/CyberArk-DAP-EKS-Lap-2021/blob/main/images/architecture_eks.JPG)

## Lab Guide
### [Task 0: Setup Utilities Host](00-Setup_Utilities_Host.md)
- An Azure VM is required as an utilities host serving the following functions:
  - Jump host to access the AKS environment
  - Docker host for Conjur Master container
  - Docker host for MySQL database container
- We will also configure the Private DNS zone for name resolution here
### [Task 1: Create AKS Cluster](01-Create_AKS_Cluster.md)
- We will provision an AKS cluster that will run the following workloads:
  - Conjur Followers
  - Demo Application Containers (cityapp)
- We will also provision an ACR and integrate with the AKS cluster
### [Task 2: Setup MySQL Database](02-Setup-MySQL-Database.md)
- Our sample application uses a MySQL database.
- In later sections, we will demostrate the hardcoded versus secretless connection methods used by the sample application to connect to this MySQL database,
### [Task 3: Deploy App with Embedded Secret](03-Deploy_App_with_Embedded_Secret.md)
- Get familiar with AKS by deploying a sample application to the AKS cluster.
- The CityApp application connects to the MySQL database setup in task 03 and displays a random city name in the web application.
- The MySQL database credentials specified in task 03 will be embedded in the kubernetes deployment configuration file.
### [Task 4: Setup Conjur Master Server](04-Setup_Conjur_Master_Server.md)
- Setup the CyberArk Conjur Master server in utilities host
### [Task 5: Deploy Follower with Seed Fetcher](05-Deploy_Follower_with_Seed_Fetcher.md)
- Deploy Conjur Followers to your AKS Cluster Node.
- We will deploy Conjur Followers using seed-fetcher, which automatically authenticate and retrieve seed on pod start-up.
- This allows self-healing and auto scaling of follower pods.
- We will also enable k8s authenicator to support the following labs.
### [Task 6: Deploy App with CyberArk Summon Secrets Injection](06-Deploy_App_with_Summon.md)
- We will secure the CityApp by using Summon to inject secrets into environment variables.
- The Summon container image is included in the CityApp pod as an init container.
### [Task 7: Deploy App to EKS Cluster with CyberArk Summon Secrets Injection](07-Deploy_App_with_Summon_Secrets_Injects.md)

### [Task 8: Deploy App to EKS Cluster with CyberArk Secretless Broker](08-Deploy_App_with_Cyberark_Secretless_Broker.md)

### [Congratulation!!! Lab Completed](Task09/readme.md)
