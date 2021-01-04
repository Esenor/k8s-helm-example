# Minikube

----------

**Start**

```bash
minikube start
```

**Enable ingress addon**

```bash
minikube addons enable ingress
```

# Kubernetes / Kubectl

----------

## Namespace

**Get namespace**

```bash
kubectl get namespaces
```

**Add namespace**

```bash
kubectl create namespace {namespace_name}
```

## Deployment

**Standard deployment**

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: standard-nginx-deployment
spec:
  selector:
    matchLabels:
      app: standard-nginx
  replicas: 2
  template:
    metadata:
      labels:
        app: standard-nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

**Apply deployment**

```bash
kubectl apply -f {deployment.yml} -n {namespace}
```

**Describe deployment**

```bash
kubectl describe deployment {deployment_metadata_name} -n {namespace}
```

## Pods

**Get pods**

*all pods form a namespace*

```bash
kubectl get pods -n {namespace}
```

*all pods form a namespace whith more information*

```bash
kubectl get pods -n {namespace} -o wide
```

*all pods form a namespace with a label filter*

```bash
kubectl get pods -n {namespace} -o wide -l app={appNameLabel}
```

**Describe pods**

```bash
kubectl describe pods {pods_metadata_name} -n {namespace}
```

*form a namespace with a label filter*

```bash
kubectl describe pods -n {namespace} -l app={appNameLabel}

## Service

```yml
apiVersion: v1
kind: Service
metadata:
  name: standard-nginx-service
spec:
  selector:
    app: standard-nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

**Apply service**

```bash
kubectl apply -f {service.yml} -n {namespace}
```


# Helm

----------

**Install S3 plugin**

[https://github.com/hypnoglow/helm-s3](https://github.com/hypnoglow/helm-s3)

**Initialize a Helm s3 repository**

```bash
helm s3 init s3://helm-releases-poc/Charts
```

**Add a Helm repository**

```bash
helm repo add olst s3://helm-releases-poc/Charts
```

**List Helm repository**

```bash
helm repo ls
```

**Search charts in repository**

```bash
helm search repo olst
helm search repo olst --versions
helm search repo olst/helm-full-nginx-chart --versions
```

**Create a chart**

Create a new folder with this files strucure (templates folder content are an nginx example of deployment, service and ingress)

```bash
/chartFolder
  | Chart.yaml
  | values.yaml
  | /templates
    | deployment.yml
    | ingress.yml
    | service.yml
```

```yml
# Chart.yaml
apiVersion: v2
name: nginx
description: a nginx application
type: application
version: 0.4.0
appVersion: 1.19.6
```

```yml
# values.yml
environment:
deployment:
  name: helm-nginx-deployment
service:
  name: helm-nginx-service
  port: 80
ingress:
  name: helm-nginx-ingress
  host: helm.efc.olst.io
  port: 80
app:
  name: helm-nginx
  replica: 3
  containers:
    nginx:
      name: nginx
      port: 80
      image: nginx
```

```yml
# deployment.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.deployment.name }}
spec:
  selector:
    matchLabels:
      app: {{ .Values.app.name }}
  replicas: {{ .Values.app.replica }}
  template:
    metadata:
      labels:
        app: {{ .Values.app.name }}
    spec:
      containers:
      - name: {{ .Values.app.containers.nginx.name }}
        image: {{ .Values.app.containers.nginx.image }}
        ports:
        - containerPort: {{ .Values.app.containers.nginx.port }}
```

```yml
# ingress.yml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: {{ .Values.ingress.name }}
spec:
  rules:
  - host: {{ .Values.ingress.host }}
    http:
      paths:
      - path: /
        backend:
          serviceName: {{ .Values.service.name }}
          servicePort: {{ .Values.ingress.port }}
```

```yml
# service.yml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Values.service.name }}
spec:
  selector:
    app: {{ .Values.app.name }}
  ports:
    - protocol: TCP
      port: {{ .Values.service.port }}
      targetPort: {{ .Values.app.containers.nginx.port }}

```

**Lint a chart**

```bash
helm lint ./
```

**Package a chart**

```bash
helm package ./
helm package nginx/helm/chart -d ./dist
```

**Push a chart on repository**

```bash
helm s3 push helm-full-nginx-chart-0.3.0.tgz olst
```

**Install a release from a chart**

```bash
helm install myRelease helm-full-nginx-chart-0.3.0.tgz
helm install myRelease olst/helm-full-nginx-chart
helm install myRelease olst/helm-full-nginx-chart -n staging
helm install myRelease olst/helm-full-nginx-chart --version 0.3.0
```

**List installed release**

```bash
helm ls
helm ls -n staging
helm ls --all-namespaces
```

**Upgrade a release from a  chart**

```bash
helm upgrade myRelease olst/helm-full-nginx-chart --version 0.3.0
helm upgrade myRelease olst/helm-full-nginx-chart -n staging
```

**Rollback a release**

```bash
helm rollback myRelease
helm rollback myRelease -n staging
```

**Release history**

```bash
helm history myRelease
helm history myRelease -n staging
```

**Uninstall a release**

```bash
helm uninstall myRelease
helm uninstall myRelease -n staging
```
