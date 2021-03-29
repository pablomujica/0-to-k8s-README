# Deploy ACR and asociate to AKS

## Clone the Repository

Get the repository with the base files:
``` bash
$ git clone https://github.com/pablomujica/1-to-k8s-ACR
$ cd 1-to-k8s-ACR
```

## Create the ACR
In the `terraform.tfvars` file, define your variables to use:
```
cluster    = "nameofthecluster"
```

Then apply the file to create the resources:
``` bash
$ terraform apply
```

## Use the Registry:
Associate to the AKS cluster
``` bash
$ az aks update -n <Cluster Name> -g <Cluster RG Name> --attach-acr $(terraform output -raw acr_name)
```

Login in you local Docker
``` bash
$ az acr login --name $(terraform output -raw acr_name)
```