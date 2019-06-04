# How to deploy docker image on kubernets?

In this tutorials, `kubernetes` will deploy docker-image from my public docker repository [docker-hub](https://cloud.docker.com/repository/registry-1.docker.io/zawthanoo/spring-fat-jar-circleci-k8s).  
In the repository, I have spring-boot application `docker-image` build by `circleci`.
You can reference here [Building spring-boot docker image with CircleCi](https://github.com/zawthanoo/spring-fat-jar-circleci-k8s).

config.yaml
```yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: spring-boot-k8s-config
  labels:
      app: spring-boot-k8s-config
data:
  spring-boot-k8s-config.yml: |
    server:
      servlet:
        contextPath: /spring-k8s
    target:
      application: spring-k8s
      description: Testing kubernetes deployment,service,configMap,secret,ingress.
```

In serect:
`bXlzcWw6CiAgdXNlcm5hbWU6IHJvb3QKICBwYXNzd29yZDogMTIzNDU2` is base64 encoded string of `screct` content of config file.
```
mysql:
  username: root
  password: 123456
```
secret.yaml
```yml
apiVersion: v1
kind: Secret
metadata:
  name: spring-boot-k8s-secret
  labels:
      app: spring-boot-k8s-secret
type: Opaque
data:
  spring-boot-k8s-secret.yml: bXlzcWw6CiAgdXNlcm5hbWU6IHJvb3QKICBwYXNzd29yZDogMTIzNDU2
```

deployment.yaml
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spring-boot-k8s-deployment
  labels:
    app: spring-boot-k8s
spec:
  replicas: 1
  selector:
    matchLabels:
      app: spring-boot-k8s
  template:
    metadata:
      labels:
        app: spring-boot-k8s
    spec:
      containers:
      - name: spring-boot-k8s
        image: zawthanoo/spring-fat-jar-circleci-k8s:0.1.0-9e40b34-15
        ports:
        - containerPort: 8080
        readinessProbe:
          httpGet:
            path: /spring-k8s
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /spring-k8s
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
        volumeMounts:
        - name: spring-boot-k8s-config-volume
          readOnly: true
          mountPath: /usr/local/myapp/config/spring-boot-k8s-config.yml
          subPath: spring-boot-k8s-config.yml
        - name: spring-boot-k8s-secret-volume
          readOnly: true
          mountPath: /usr/local/myapp/config/spring-boot-k8s-secret.yml
          subPath: spring-boot-k8s-secret.yml
      imagePullSecrets:
      - name: muturegistrykey
      volumes:
        - name: spring-boot-k8s-config-volume
          configMap:
            name: spring-boot-k8s-config
            items:
            - key: spring-boot-k8s-config.yml
              path: spring-boot-k8s-config.yml
        - name: spring-boot-k8s-secret-volume
          secret:
            secretName: spring-boot-k8s-secret
```

service.yaml
```yml
apiVersion: v1
kind: Service
metadata:
  name: spring-boot-k8s-svc
  labels:
    name: spring-boot-k8s-svc
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: spring-boot-k8s
```

ingress.yaml
```yml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /spring-k8s
        backend:
          serviceName: spring-boot-k8s-svc
          servicePort: 808
```
Now, you can create or apply the on `kubernetes` by using `kubectl` commend.

Single file run
```
$ kubectl -f apply config.yaml
$ kubectl -f apply secret.yaml
$ kubectl -f apply deployment.yaml
$ kubectl -f apply service.yaml
$ kubectl -f apply ingress.yaml
```

Multiple file run
```
$ kubectl apply -f config.yaml  -f secret.yaml -f deployment.yaml -f service.yaml -f ingress.yaml
```
Check in browser : 
```
http://ip-address/spring-k2
```
If you don't have `kubernetes` enviroment, [kubernetes HA Install + Rancher 2.0](https://github.com/zawthanoo/setup-rke)

If you don't have enought hardware/resource, you can use [minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/)
