# Springboot demo app

SpringBoot Demo with MySQL running on Kubernetes

## Deploy to Kubernetes

### 1 Create namespace

```shell
kubectl create namespace demoapp
```

### 2 Default namespace to demoapp

```shell
kubectl config set-context --current --namespace=demoapp
```

### 3 Create MySQL Secrets to have encrypted password 

```shell
kubectl create secret generic mysql-secrets \
  --from-literal=rootpassword=r00tDefaultPassword1! \
  --from-literal=username=demo \
  --from-literal=password=defaultPassword1! \
  --from-literal=database=DB


### Clone the repo if using OCI CloudShell or local

```shell
cd springboot-demo-k8s-mysql/kubernetes
```

### Deploy MySQL 8

#### 4 Create PVC for MySQl using CSI for Block Volume

```shell
kubectl apply -f mysql-pvc-manual.yaml
```

#### 5 Create Service for MySQL (for load balancing)

```shell
kubectl apply -f mysql-svc.yaml
```

#### 6 Create Deployment for MySQL

```shell
kubectl apply -f mysql-dep.yaml
kubectl get pods
```

### Deploy the Spring Boot Demo App

#### 7 Create Service for Demo App

Note: This step will create a new LoadBalancer on the infrastructure

```shell
kubectl apply -f app-svc.yaml
```

#### 8 Create Deployment for Demo App

Note: The app will create the necessary tables on the MySQL on the first run

```shell
kubectl apply -f app-dep.yaml
```

#### 9 Optional: Check logs

```shell
kubectl logs -l app=demoapp --follow
```

#### Optional: Insert Data to MySQL

##### 10 Connect to mysql

```shell
kubectl run -it --rm --image=mysql:8 --restart=Never mysql-client -- mysql DB -h mysql -pr00tDefaultPassword1!
```

Press enter

```shell
If you don't see a command prompt, try pressing enter.

mysql>
```

```sql

insert into users (first_name, last_name) values ('joe', 'doe');
insert into users (first_name, last_name) values ('joe', 'doe');
insert into users (first_name, last_name) values ('joe', 'doe');
insert into users (first_name, last_name) values ('joe', 'doe');
insert into users (first_name, last_name) values ('joe', 'doe');
insert into users (first_name, last_name) values ('joe', 'doe');
insert into users (first_name, last_name) values ('joe', 'doe');
insert into users (first_name, last_name) values ('joe', 'doe');
insert into users (first_name, last_name) values ('joe', 'doe');
insert into users (first_name, last_name) values ('joe', 'doe');
insert into users (first_name, last_name) values ('joe', 'doe');
insert into users (first_name, last_name) values ('joe', 'doe');
insert into users (first_name, last_name) values ('joe', 'doe');
insert into users (first_name, last_name) values ('joe', 'doe');
insert into users (first_name, last_name) values ('joe', 'doe');
insert into users (first_name, last_name) values ('joe', 'doe');
insert into users (first_name, last_name) values ('joe', 'doe');
insert into users (first_name, last_name) values ('joe', 'doe');
insert into users (first_name, last_name) values ('joe', 'doe');
insert into users (first_name, last_name) values ('joe', 'doe');
insert into users (first_name, last_name) values ('joe', 'doe');
```

Expected results:

```shell
If you don't see a command prompt, try pressing enter.

mysql> insert into users (first_name, last_name) values ('joe', 'doe');
Query OK, 1 row affected (0.00 sec)

mysql> quit
Bye
pod "mysql-client" deleted
```

#### 11 Optional: Test with port-forward

```shell
kubectl port-forward deploy/demoapp 8081:8081
```

Navigate to http://localhost:8081/users

#### 12 Test with LoadBalancer IP Address

```shell
kubectl get svc
```

Navigate to http://<demoapp_EXTERNAL_IP_ADDRESS>/users

## Create Horizontal Pod Autoscaler for Demo App

### Install metrics server

```shell
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

### Create autoscale for Demo App
### when cpu utilization goes above 5%, start new pods
```shell
kubectl autoscale deployment demoapp --cpu-percent=5 --min=1 --max=3
```

### Check HPA (Horizontal Pod Autoscaling)

```shell
kubectl get hpa
```

### Increase load

```shell
kubectl run -i --tty load-generator --rm --image=busybox --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://demoapp/users; done"
```

Within a minute or so, we should see the higher CPU load by executing:

```shell
kubectl get hpa
```
All commands
---------------
kubectl create namespace demoapp
kubectl config set-context --current --namespace=demoapp

kubectl create secret generic mysql-secrets \
  --from-literal=rootpassword=r00tDefaultPassword1! \
  --from-literal=username=demo \
  --from-literal=password=defaultPassword1! \
  --from-literal=database=DB


kubectl apply -f mysql-pvc-manual.yaml

kubectl apply -f mysql-svc.yaml
kubectl apply -f mysql-dep.yaml
kubectl apply -f app-svc.yaml

kubectl apply -f app-dep.yaml
kubectl logs -l app=demoapp --follow

in windows cmd
kubectl run -it --rm --image=mysql:8 --restart=Never mysql-client -- mysql DB -h mysql -pr00tDefaultPassword1!

kubectl port-forward deploy/demoapp 8081:8081










































## Prometheus and Grafana

### Install the grafana-prometheus stack

```shell
  choco install kubernetes-helm
  helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
   helm repo add stable https://charts.helm.sh/stable
     helm repo update
helm install prometheus prometheus-community/kube-prometheus-stack
 kubectl port-forward deployment/prometheus-grafana 3000
```

### get the grafana admin password

```shell
kubectl get secret prometheus-grafana \
 -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
 ```

### Test Grafana with port-forward

 ```shell
kubectl port-forward svc/prometheus-grafana 8085:80
 ```

 Navigate to http://localhost:8085/

## Build Demo App image

Skip this step if you just want to test the app on Kubernetes

```shell
docker build --pull --no-cache --squash --rm --progress plain -f Dockerfile -t sbdemo .
```
