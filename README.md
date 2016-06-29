#Kubernetes Demo 


##Links
- [Kubernetes Github](https://github.com/kubernetes/kubernetes)
- [Kubernetes Home Page](http://kubernetes.io/)

##Kubectl

CLI which functions as the primary mmethod of interacting with a kubernetes cluster

Is configured using a `kubeconfig` file

Example:

```
apiVersion: v1
kind: Config
users:
- name: admin
  user:
    client-certificate-data: <removed>
    client-key-data: <removed>
clusters:
- name: aws_kubernetes
  cluster:
    certificate-authority-data: <removed> 
    server: https://kubemasterext.e2e.apigee.net:443
contexts:
- context:
    cluster: aws_kubernetes
    namespace: default
    user: admin
  name: aws-kubernetes-context
current-context: aws-kubernetes-context
```


##Namespaces

Create a namespace

```
kubectl create ns jbowen
```

Set context to use `jbowen` namespace.

```
kubectl config set contexts.aws-kubernetes-context.namespace jbowen
```

##Pods  

Go over the yaml spec

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80

```

##Deployments

Go over the yaml spec

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

Create a deployment  

```
kubectl create -f nginx-deploy.yaml
```

Create the Alpine Curl Pod in a different shell and exec into it

```
kubectl create -f alpine-curl-deploy.yaml
kubectl exec <pod-name> -i -t -- ash
```

Get the nginx pod IP

```
kubectl describe pod nginx | grep IP
```

Curl the nginx pod from it's IP from within the Alpine Curl Pod

```
curl above IP
```

##Services

Expose deployed pod as a service

```
kubectl create -f nginx-svc.yaml
```

Go over yaml spec

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  ports:
  - port: 8000
    targetPort: 80
    protocol: TCP
  selector:
    app: nginx
```

Get the service IP

```
kubectl describe svc nginx 
```

##Volumes

Create Persistent Volume

```
kubectl create -f ebs-2a-pv.yaml
```

Go over the yaml spec

```
  apiVersion: v1
  kind: PersistentVolume
  metadata:
    name: pv001
  spec:
    capacity:
      storage: 5Gi
    accessModes:
      - ReadWriteOnce
    persistentVolumeReclaimPolicy: Recycle
    awsElasticBlockStore:
        volumeID: vol-5e38f3eb
```

Create Persistent Volume Claim

```
kubectl create -f redis-pvc.yaml
```

Go over yaml spec

```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: redis-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```

Create Redis Server Service

```
kubectl create -f redis-server-svc.yaml
```

Create a Redis Server deployment with an persistent volume claim

```
kubectl create -f redis-server-deploy.yaml
```

Go over the yaml spec

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
    name: redis-server
spec:
    replicas: 1
    template:
        metadata:
            labels:
                app: redis-server
        spec:
            nodeSelector:
                availabilityZone: us-west-2a
            containers:
            - name: redis
              image: redis
              volumeMounts:
              - name: redis-persistent-storage
                mountPath: /data/redis
              ports:
              - containerPort: 6379
                name: "redis-server"
            volumes:
            - name: redis-persistent-storage
              persistentVolumeClaim:
                claimName: redis-claim
```

Create the Redis Client Deployment

```
kubectl create -f redis-client-deploy.yaml
```

Create the Redis Client Service

```
kubectl create -f redis-client-svc.yaml
```

Get the Redis Client Service IP

```
kubectl describe svc redis-client | grep IP
```

Show that we can put a key-value pair into the redis cache using the Alpine Curl Pod

```
kubectl exec <pod-name> -i -t -- ash
curl -X POST -d 'world' 10.0.92.12:3000/hello
curl 10.0.92.12:3000/hello
```

##Scaling

Demonstrate scaling a deployment

```
kubectl scale deployment nginx-deployment --replicas=3
```

Scale back down

```
kubectl scale deployment nginx-deployment --replicas=1
```

##Rolling Updates

Update to a newer version of nginx

```
kubectl replace -f nginx-deploy-new.yaml
```