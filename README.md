# kubernetes-vs-swarm

Simple examples for launching an app on Kubernetes and Docker Swarm to see the differences

## Prerequisites

You should have Docker for Mac/Windows installed with Kubernetes enabled. DO NOT
enable the "Deploy Docker Stacks to Kubernetes by Default" option

## Example

### Lets create some NGINX document root volumes with an `index.html`.
```
mkdir -p /tmp/k8-nginxdocroot
echo "k8 nginx" > /tmp/k8-nginxdocroot/index.html

mkdir -p /tmp/swarm-nginxdocroot
echo "swarm nginx" > /tmp/swarm-nginxdocroot/index.html
```

### Create our "secret" values:

**Docker Secret**
```
docker secret create nginx1_secret ./secret.txt
docker secret ls
```

**Kubernetes**
```
kubectl create secret generic mynginx-secret --from-file=./secret.txt
kubectl get secret mynginx-secret -o yaml
kubectl describe secret mynginx-secret
```

### Launch our app on Docker Swarm as a Service, view and check logs

Docker Service (via stack)
```
docker stack deploy -c swarm-nginx1.yaml nginx1
docker service ls
docker service inspect nginx1_app
docker service logs --tail 100 --follow (docker service ps nginx1_app | grep nginx1 | awk '{print $1; exit}')
```

Kubernetes
```
kubectl create -f kubernetes-nginx1.yaml
kubectl get all
kubectl describe pod nginx1
kubectl logs --tail 100 --follow (kubectl get pods | grep nginx1 | awk '{print $1; exit}')
```

### Shell into one of the replicas:

Docker Swarm
```
docker exec -it (docker ps | grep nginx1_app | awk '{print $1; exit}') /bin/bash
```

Kubernetes
```
docker exec -it (docker ps | grep k8s_nginx1 | awk '{print $1; exit}') /bin/bash
```

Once inside a container
From within see ENV vars + secret value
```
env | grep SOME_ENV_VAR
env | grep SECRET_PATH
cat $SECRET_PATH
```

### Curl it!

Docker Swarm
```
curl http://localhost:30080
```

Kubernetes
```
curl http://localhost:30081
```

### Scale it

Docker Swarm
```
docker service scale nginx1_app=3
docker service ps nginx1_app
```

Kubernetes
```
kubectl scale deployment nginx1 --replicas=3
kubectl get pods
kubectl describe deployment nginx1
```

### Remove

Docker Swarm
```
docker service rm nginx1_app
docker secret rm nginx1_secret
docker service ls
```

Kubernetes
```
kubectl delete -f kubernetes-nginx1.yaml
kubectl delete secret mynginx-secret
kubectl get all
```
