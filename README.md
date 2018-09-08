# Flask Bookshelf

Flask + Vue + Postgres + Docker + Kubernetes

## Docker

```sh
$ docker-compose up -d --build
```

[http://localhost:8080/](http://localhost:8080/)

## Kubernetes

### Minikube

Install and run [Minikube](https://kubernetes.io/docs/setup/minikube/).

#### [Hyperkit](https://github.com/moby/hyperkit) Example

Install:

```sh
$ curl -Lo docker-machine-driver-hyperkit \
https://storage.googleapis.com/minikube/releases/latest/docker-machine-driver-hyperkit \
&& chmod +x docker-machine-driver-hyperkit \
&& sudo cp docker-machine-driver-hyperkit /usr/local/bin/ \
&& rm docker-machine-driver-hyperkit \
&& sudo chown root:wheel /usr/local/bin/docker-machine-driver-hyperkit \
&& sudo chmod u+s /usr/local/bin/docker-machine-driver-hyperkit
```

Run:

```sh
$ minikube start --vm-driver=hyperkit
$ eval $(minikube docker-env)
$ minikube dashboard
```

### Postgres

Create deployment:

```sh
$ kubectl create -f ./kubernetes/postgres-deployment.yml
```

Create the service:

```sh
$ kubectl create -f ./kubernetes/postgres-service.yml
```

Create the database:

```sh
$ kubectl get pods
$ kubectl exec postgres-<POD_IDENTIFIER> \
    --stdin --tty -- createdb -U postgres books
```

### Flask

Build and push the image to Docker Hub:

```sh
$ docker build -t mjhea0/flask-kubernetes ./services/server
$ docker push mjhea0/flask-kubernetes
```

> Make sure to replace `mjhea0` with your Docker Hub namespace in the above commands as well as in *kubernetes/flask-deployment.yml*

Create the deployment:

```sh
$ kubectl create -f ./kubernetes/flask-deployment.yml
```

Create (and expose) the service:

```sh
$ kubectl expose deployment flask --type=NodePort --port 5000
```

Try it out:

```sh
$ open $(minikube service flask --url)/books/ping
$ open $(minikube service flask --url)/books
```

### Vue

Build and push the image to Docker Hub:

```sh
$ docker build -t mjhea0/vue-kubernetes ./services/client \
    -f ./services/client/Dockerfile-minikube
$ docker push mjhea0/vue-kubernetes
```

> Again, replace `mjhea0` with your Docker Hub namespace in the above commands as well as in *kubernetes/vue-deployment.yml*

Create the deployment:

```sh
$ kubectl create -f ./kubernetes/vue-deployment.yml
```

Create (and expose) the service:

```sh
$ kubectl expose deployment vue --type=NodePort --port 8080
```

Try it out:

```sh
$ open $(minikube service vue --url)
```

### Ingress

Enable and apply:

```sh
$ minikube addons enable ingress
$ kubectl apply -f ./kubernetes/minikube-ingress.yml
```

Add entry to */etc/hosts* file:

```
<MINIKUBE_IP> hello.world
```

Try it out:

1. http://hello.world
1. http://hello.world/books
