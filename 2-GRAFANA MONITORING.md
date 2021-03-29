# Deploy Grafana and Prometheus to monitor AKS cluster

## Clone the Repository

Get the repository with the base files:
``` bash
$ git clone https://github.com/pablomujica/2-to-k8s-HELM
$ cd 2-to-k8s-HELM
```

## Install Helm
To install Helm, first get the latest version from https://github.com/helm/helm/releases for Linux amd64 and download it:
``` bash
$ wget https://get.helm.sh/helm-v3.5.3-linux-amd64.tar.gz
```

Unpack the downloaded file:
``` bash
$ tar xvf helm-v3.5.3-linux-amd64.tar.gz
```

Move the binary to your bin forlder:
``` bash
$ sudo mv linux-amd64/helm /usr/local/bin
```

Clean the folder:
``` bash
$ rm helm-v3.4.1-linux-amd64.tar.gz
$ rm -rf linux-amd64
```

## Install Prometheus and Grafana

Add the Helm repo for Prometheus and Grafana:
``` bash
$ helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
$ helm repo add grafana https://grafana.github.io/helm-charts
$ helm repo update
```

Create a namespace for the monitoring tools:
``` bash
kubectl create namespace monitoring
```

Install the Prometheus chart in the create namespace, with the default storage providers:
``` bash
$ helm install prometheus prometheus-community/prometheus --namespace monitoring --set alertmanager.persistentVolume.storageClass="default" --set server.persistentVolume.storageClass="default"
```

Get the config file for Datasource `datasource-grafana.yaml`, and verify the variables with the info from the previous command:
``` bash
The Prometheus server can be accessed via port 80 on the following DNS name from within your cluster:
prometheus-server.monitoring.svc.cluster.local
```
``` yaml
datasources:
  datasources.yaml:
    apiVersion: 1
    datasources:
    - name: Prometheus
      type: prometheus
      url: <prometheus-server URL>
      access: proxy
      isDefault: true
```


Install the Grafana Helm with the configuration for the service
``` bash
$ helm install grafana grafana/grafana --namespace monitoring --set persistence.storageClassName="default" --set persistence.enabled=true --set adminPassword='CHANGETHISPASSWORD' --values datasource-grafana.yaml --set service.type=LoadBalancer
```

Get the URL for the service with :
``` bash
$ echo http://$(kubectl get svc --namespace monitoring grafana -o jsonpath='{.status.loadBalancer.ingress[0].ip}'):80
```

Login to the dashboard with the user `admin` and the password created when installing the service

You can import a dashboard as an example.
Go to `Create`/`Import`, in the `Import via grafana.com` add the ID `6417`, and click `Load`. Under the prometheus field, select the data source. At the end click `Import`