# Sample App Deployment

This file describes installation of a sample application written using Django and python.

1. File Location and Contents of Dockerfile:

sriz@ % pwd
/Users/sridhar/sriz_labs/Kubernetes_workspace/Sample_App/python-web-app
sriz@ %
 
sriz@ % ls -ltr
total 16
-rw-r--r--  1 sridhar  staff  377  3 May 20:12 Dockerfile
drwxr-xr-x  6 sridhar  staff  192  3 May 20:12 devops
-rw-r--r--  1 sridhar  staff   14  3 May 20:12 requirements.txt
sriz@ % 

sriz@ % cat Dockerfile
FROM ubuntu

WORKDIR /app

COPY requirements.txt /app/
COPY devops /app/

RUN apt-get update && apt-get install -y python3 python3-pip python3-venv

SHELL ["/bin/bash", "-c"]

RUN python3 -m venv venv1 && \
source venv1/bin/activate && \
pip install --no-cache-dir -r requirements.txt

EXPOSE 8000

CMD source venv1/bin/activate && python3 manage.py runserver 0.0.0.0:8000



2. Build a docker image using this Dockerfile.

sriz@ % pwd
/Users/sridhar/sriz_labs/Kubernetes_workspace/Sample_App/python-web-app
sriz@ % 
sriz@ % docker images
REPOSITORY                    TAG       IMAGE ID       CREATED        SIZE
gcr.io/k8s-minikube/kicbase   v0.0.50   f3db27eba481   2 months ago   1.35GB
sriz@ % 
sriz@ % docker build -t python-sample-app-demo:v1 .
[+] Building 7.1s (8/10)                                                                             
 => [internal] load build definition from Dockerfile                                            0.0s
 => => transferring dockerfile: 421B                                                            0.0s
 => [internal] load .dockerignore                                                               0.0s
 => => transferring context: 2B                                                                 0.0s
 => [internal] load metadata for docker.io/library/ubuntu:latest                                3.8s
 => [1/6] FROM docker.io/library/ubuntu@sha256:c4a8d5503dfb2a3eb8ab5f807da5bc69a85730fb49b5cfc  2.7s
 => => resolve docker.io/library/ubuntu@sha256:c4a8d5503dfb2a3eb8ab5

sriz@ % docker images
REPOSITORY                    TAG       IMAGE ID       CREATED              SIZE
python-sample-app-demo        v1        ac0727d5b05a   About a minute ago   622MB
gcr.io/k8s-minikube/kicbase   v0.0.50   f3db27eba481   2 months ago         1.35GB
sriz@ % 

3. Now, we create sample-app-deployment.yaml file using the above docker image

apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-app-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: sample-app
  template:
    metadata:
      labels:
        app: sample-app
    spec:
      containers:
      - name: sample-app
        image: python-sample-app-demo:v1
        ports:
        - containerPort: 8000

4. Run this deployment

sriz@ % kubectl get all                                                
NAME                                        READY   STATUS    RESTARTS   AGE
pod/sample-app-deployment-f96bb5b79-dsk9t   1/1     Running   0          67s
pod/sample-app-deployment-f96bb5b79-thj9z   1/1     Running   0          70s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   115m

NAME                                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/sample-app-deployment   2/2     2            2           9m17s

NAME                                              DESIRED   CURRENT   READY   AGE
replicaset.apps/sample-app-deployment-f96bb5b79   2         2         2       70s
sriz@ % 

