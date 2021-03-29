# Publish a container to run in the Cluster

## Clone the Repository

Get the repository with the base files:
``` bash
$ git clone https://github.com/pablomujica/4-to-k8s-Docker
$ cd 4-to-k8s-Docker
```


## Build the Dockerfile

To directly build the container in the ACR: 

``` bash
$ az acr build --registry <ACR Name> --image app/app .
```

## Create a deployment plan

Get the repository with the deployment file:
``` bash
$ cd ..
$ git clone https://github.com/pablomujica/5-to-k8s-Deployment
$ cd https://github.com/pablomujica/5-to-k8s-Deployment
```

In the deployment `flask-app.yaml`, you define the spec for the app deployment:
``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: flask-app
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  minReadySeconds: 5 
  template:
    metadata:
      labels:
        app: flask-app
    spec:
      nodeSelector:
        "beta.kubernetes.io/os": linux
      containers:
      - name: flask-app
        image: <ACR Name>.azurecr.io/app/app:latest
        ports:
        - containerPort: 8000
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
```

After that, you can define more resource separating with a `---` between the resources.

Next, we define the service to manage the incomming traffic:
``` yaml
apiVersion: v1
kind: Service
metadata:
  name: flask-app
spec:
  type: LoadBalancer
  ports:
  - port: 8000
  selector:
    app: flask-app
```

## Deploy to the cluster
Next, you deploy this plan to the cluster:
```
$ kubectl apply -f flask-app.yaml
```

## Test the Container and the LoadBalancer
To get the IP for the service, behind wicht the Deployment runs:
``` bash
$ echo http://$(kubectl get svc flask-app  -o jsonpath='{.status.loadBalancer.ingress[0].ip}'):8000
```

To check if your loadbalancer is using the two replicas, just use cUrl to check it:
```
$ curl http://20.62.141.200:8000
```

Try it really quick with a small bash loop:
``` bash
$ for i in {1..10}; do curl $(echo http://$(kubectl get svc flask-app  -o jsonpath='{.status.loadBalancer.ingress[0].ip}'):8000) -w "\n"; done
```
