# Yaml in kubernetes

### Kubernetes api version for komponents

| Kind       | apiVersion |
| ---------- | ---------- |
| Pod        | v1         |
| Service    | v1         |
| ReplicaSet | apps/v1    |
| Deployment | apps/v1    |

### Pod definition

definition file always contain 4 top level fields:

`pod-definition.yml`

```yaml
#4 root level fields
#always required in configuration files
apiVersion: v1        #version of the kubernetes api we using to create object
kind: Pod             #type of object
metadata:             #data about the object
  name: myapp-pod
  labels:
    app: myapp
    type: front-end

spec:                #object specification, different for different objects
  containers:
    - name: nginx-container
      image: nginx
```

Then we can create pod from this file:

```bash
kubectl create -f pod-definition.yml
```

----

### Replication controller

`rc-definition.yml`

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: myapp-rc
  labels:
    app: myapp
    type: front-end

spec:						#replication controller spec
  template:                 #template for pod
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    
    spec:                   #pod spec
      containers:
        - name: nginx-container
          image: nginx
  replicas: 3               #replicas count
  
```

Then we can create replication controller from this file:

```bash
kubectl create -f rc-definition.yml
```

Check replication controller:

```bash
kubectl get replicationcontroller
```

----

### Replicaset

`replicaset-definition.yml`

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
    app: myapp
    type: front-end

spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx
          
  replicas: 3
  selector:                  #define which pods fall under this replicaset
    matchLabels:
      type: front-end
      
```

Then we can create replicaset from this file:

```bash
kubectl create -f replicaset-definition.yml
```

Check replicaset:

```bash
kubectl get replicaset
```

How to scale replicaset:

- change replicas count
- update replicaset config

```bash
kubectl replace -f replicaset-definition.yml
```

Second way to do this is:

```bash
kubectl scale --replicas=6 -f replicaset-definition.yml
```

Or update resource:

```bash
kubectl scale --replicas=6 replicaset myapp-replicaset
```

Check replicaset api version:

```bash
kubectl api-resources | grep replicaset
```

```bash
kubectl explain replicaset
```

Edit replicaset config:

```bash
kubectl edit replicaset <name>
```

---------------

### Deployment

Rolling updates - update containers one after another

The deployment provides us with the capability upgrade underlying instances seamlessly with the rolling update and provide rollback changes, paus and unpause changes.

**Creating deployment**:

`deployment-definition.yml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-replicaset
  labels:
    app: myapp
    type: front-end

spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx
          
  replicas: 3
  selector:                  #define which pods fall under this replicaset
    matchLabels:
      type: front-end
```

Create deployment from file:

```bash
kubectl create -f deployment-definition.yml
```

Show deployment:

```bash
kubectl get deployment
```

How this work:

1. We create deployment
2. Deployment create replicaset
3. Replicaset create pods

#### Updates and rollback

Rollout and versioning:

- when we first create a deployment it triggers a rollout, a new rollout create a new deployment revision
- when application is upgraded(container version is updating) a new rollout is triggered and a new deployment revision is created

Rollout command:

```bash
kubectl rollout status deployment/myapp-deployment
```

```bash
kubectl rollout history deployment/myapp-deployment
```

Deployment strategy:

1. Recreate - destroy all old pods and recreate with new version
2. Rolling update - destroy and recreate pods one by one

How to update deployment after changes:

```bash
kubectl apply -f deployment-definition.yml
```

Rollback changes:

```bash
kubectl rollout undo deployment/myapp-deployment
```

------

### Network

IP address is assigned to a pod. Each pod in kubernetes gets its own internal IP address.

When kubernetes is initially configured, we create an interanl private network and all the are attached to it

When we deploy multiple pods they all get separate IP assigned from this network

The pods can communicate with each other through this IP

-------

### Services

Service Types

- NodePort
- ClusterIP
- LoadBalancer

#### NodePort

NodePort service is an object that listen to the port on the node and forward requests on that port to a port on the pod running the web application.

![image-20240227163801962](https://github.com/Mardukay/kubernetes-notes/blob/main/images/nodeport)

`service-definition.yml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service

spec:
  type: NodePort      # service type
  ports:
    - targetPort: 80  # port on pod
      port: 80        # port on service object
      nodePort: 30008 # node port
      # if we don't provide a target port then it the same as port
      # if we don't provide a nodePort then it allocated automatically from range 30000-32767
      # we can have multiple port mapping in one service
  # define pod
  selector:
    app: myapp
    type: front-end
      
```

Create service from file:

```bash
kubectl create -f service-definition.yml
```

