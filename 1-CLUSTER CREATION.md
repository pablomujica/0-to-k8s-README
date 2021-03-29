# Deploy AKS with Terraform from WSL

## Clone the Repository

Get the repository with the base files:
``` bash
$ git clone https://github.com/pablomujica/2-to-k8s-HELM
$ cd 2-to-k8s-HELM
```


## Install Terraform

Install unzip, this will be used unpack the client :
``` bash
$ sudo apt-get install unzip
```

Download the client, change the link to the latest x64 linux link from https://www.terraform.io/downloads.html :
``` bash
$ wget <copied link>
```

Unzip the file:
``` bash
$ unzip terraform_0.12.7_linux_amd64.zip
```

Move the file to your bin directory:
``` bash
$ sudo mv terraform /usr/local/bin
```

Check that the installation was succesuful:
``` bash
$ terraform version
```

## Install the Azure CLI:

Install:
``` bash
$ curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

Login to your account:
``` bash
$ az login
```

## Deploy the cluster

Clone the repo with the terraform files for the deploy and enter that directory:
``` bash
$ git clone https://github.com/hashicorp/learn-terraform-provision-aks-cluster
$ cd learn-terraform-provision-aks-cluster
```

Create the ServicePricipal to be used to deploy the resource:
``` bash
$ az ad sp create-for-rbac --skip-assignment
{
  "appId": "aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa",
  "displayName": "azure-cli-2021-03-27-22-23-21",
  "name": "http://azure-cli-2021-03-27-22-23-21",
  "password": "********-****-****-****-************",
  "tenant": "cccccccc-ccccc-ccccc-cccc-cccccccc"
}
```

Copy the variables `appId` and `password` to the `terraform.tfvars` file:
```
appId    = "aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa"
password = "********-****-****-****-************"
```

Init the Terraform directory:
``` bash
$ terraform init
```

Create the resource specified in the file:
``` bash
$ terraform apply
```

Confirm the creation of the resources:
```
Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes
```

Wait for the process to end, in my case took 15 minutes
```
(...)

Apply complete! Resources: 3 added, 0 changed, 0 destroyed.

Outputs:

kubernetes_cluster_name = "<Cluster Name>"
resource_group_name = "<Cluster RG Name>"
```

## Connect to the cluster

### Install the kubectl client:

Get the binary:
``` bash
$ curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
```

Give execute permisions
``` bash
$ chmod +x ./kubectl
```

Move the file to your bin directory:
``` bash
$ sudo mv ./kubectl /usr/local/bin/kubectl
```

Check the installation
``` bash
$ kubectl version --client
```

### Get the credentials

Export the credentials from the az-cli:
``` bash
$ az aks get-credentials --resource-group $(terraform output -raw resource_group_name) --name $(terraform output -raw kubernetes_cluster_name)
Merged "<Cluster Name>" as current context in /home/snow/.kube/config
```

### Connect to the Dashboard
Create the role and associate it to the cluster-admin user so it can connect:
``` bash
$ kubectl create clusterrolebinding kubernetes-dashboard --clusterrole=cluster-admin --serviceaccount=kube-system:kubernetes-dashboard --user=clusterUser
```

Create the tunnel to access the dashboard and access it from http://127.0.0.1:8001/
``` bash
$ az aks browse --resource-group $(terraform output -raw resource_group_name) --name $(terraform output -raw kubernetes_cluster_name)
Merged "<Cluster Name>" as current context in /tmp/tmpnv4i1a8q
Proxy running on http://127.0.0.1:8001/
Press CTRL+C to close the tunnel...
```

In the Dashboard you will be asked for a token to login, to create it, in another shell, without closing the one with the tunnel:
``` bash
$ kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep service-controller-token | awk '{print $1}')

Name:         service-controller-token-pqgsq
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: service-controller
              kubernetes.io/service-account.uid: ******-****-****-****-**********

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1761 bytes
namespace:  11 bytes
token:      eyJhbGciO...

```
Enter the token value to the dashboard, and you should be able to enter.

## Delete the cluster
``` bash
$ terraform destroy
```