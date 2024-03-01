# Microservice architecture example

For example we deploy simple app

### Deployment scheme

![image-20240228131857627](https://github.com/Mardukay/kubernetes-notes/blob/main/images/microservice-app)

### Deployment plan

1. Deploy PODs
2. Create Services(ClusterIP)
   - redis
   - db
3. Create Services(NodePort)
   - voting-app
   - result-app

### Deploy voting app with kind:Pod

`voting-app-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: voting-app-pod
  labels:
    name: voting-app-pod
    app: demo-voting-app
    
spec:
  containers:
    - name: voting-app
      image: kodekloud/examplevotingapp_vote:v1
      ports:
        - containerPort: 80
```

`result-app-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: result-app-pod
  labels:
    name: result-app-pod
    app: demo-voting-app
    
spec:
  containers:
    - name: result-app
      image: kodekloud/examplevotingapp_result:v1
      ports:
        - containerPort: 80
```

`redis-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis-pod
  labels:
    name: redis-pod
    app: demo-voting-app
    
spec:
  containers:
    - name: redis
      image: redis
      ports:
        - containerPort: 6379
```

`postgres-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: postgres-pod
  labels:
    name: postgres-pod
    app: demo-voting-app
    
spec:
  containers:
    - name: postgres
      image: postgres
      ports:
        - containerPort: 5432
      env:                         #better practice is using secrets
        - name: POSTGRES_USER      #but for example we use env:
          value: "postgres"
        - name: POSTGRES_PASSWORD
          value: "postgres"
        - name: POSTGRES_HOST_AUTH_METHOD
          value: trust
```

`worker-app-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: worker-app-pod
  labels:
    name: worker-app-pod
    app: demo-voting-app
    
spec:
  containers:
    - name: worker-app
      image: kodekloud/examplevotingapp_worker:v1
      
```

### Create services

`redis-service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis
  labels:
    name: redis-service
    app: demo-voting-app
spec:
  ports:
    - port: 6379
      targetPort: 6379
  selector:
    name: redis-pod
    app: demo-voting-app
```

`postgres-service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: db
  labels:
    name: postgres-service
    app: demo-voting-app
spec:
  ports:
    - port: 5432
      targetPort: 5432
  selector:
    name: postgres-pod
    app: demo-voting-app
```

`voting-app-service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: voting-service
  labels:
    name: voting-service
    app: demo-voting-app
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30004
  selector:
    name: voting-app-pod
    app: demo-voting-app
```

`result-app-service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: result-service
  labels:
    name: result-service
    app: demo-voting-app
spec:
  type: NodePort
  ports:
    - port: 8080
      targetPort: 80
      nodePort: 30005
  selector:
    name: result-app-pod
    app: demo-voting-app
```

### Deploy voting app with kind:Deployment

`voting-app-deploy.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: voting-app-deploy
  app: demo-voting-app
  
spec:
  replicas: 1
  selector:
    matchLabels:
      name: voting-app-pod
      app: demo-voting-app
  template:
    metadata:
      name: voting-app-pod
      labels:
        name: voting-app-pod
        app: demo-voting-app
    spec:
      containers:
        - name: voting-app
          image: kodekloud/examplevotingapp_vote:v1
          ports:
            - containerPort: 80
```

`redis-deploy.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-deploy
  app: demo-voting-app
  
spec:
  replicas: 1
  selector:
    matchLabels:
      name: redis-pod
      app: demo-voting-app
  template:
    metadata:
      name: redis-pod
      labels:
        name: redis-pod
        app: demo-voting-app
    spec:
      containers:
        - name: redis
          image: redis
          ports:
            - containerPort: 6379
```

`postgres-deploy.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres-deploy
  app: demo-voting-app
  
spec:
  replicas: 1
  selector:
    matchLabels:
      name: postgres-pod
      app: demo-voting-app
  template:
    metadata:
      name: postgres-pod
      labels:
        name: postgres-pod
        app: demo-voting-app
    spec:
      containers:
        - name: postgres
          image: postgres
          ports:
            - containerPort: 5432
          env:                         #better practice is using secrets
            - name: POSTGRES_USER      #but for example we use env:
              value: "postgres"
            - name: POSTGRES_PASSWORD
              value: "postgres"
            - name: POSTGRES_HOST_AUTH_METHOD
              value: trust
```

`result-app-deploy.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: result-app-deploy
  app: demo-voting-app
  
spec:
  replicas: 1
  selector:
    matchLabels:
      name: result-app-pod
      app: demo-voting-app
  template:
    metadata:
      name: result-app-pod
      labels:
        name: result-app-pod
        app: demo-voting-app
    spec:
      containers:
        - name: result-app
          image: kodekloud/examplevotingapp_result:v1
          ports:
            - containerPort: 80
```

`worker-app-deploy.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: worker-app-deploy
  app: demo-voting-app
  
spec:
  replicas: 1
  selector:
    matchLabels:
      name: worker-app-pod
      app: demo-voting-app
  template:
    metadata:
      name: worker-app-pod
      labels:
        name: worker-app-pod
        app: demo-voting-app
    spec:
      containers:
        - name: worker-app
          image: kodekloud/examplevotingapp_worker:v1

```

### Create pods/deloyment and services from files

```bash
kubectl create -f <path_to_file>
```

Expose NodePort services with PortForward:

```bash
kubectl port-forward svc/voting-service 9000:80
```

```bash
kubectl port-forward svc/result-service 9001:8080 
```

### 
