# kubernetes-volumes

On-disk files in a Container are ephemeral, which presents some problems for non-trivial applications when running in Containers. First, when a Container crashes, kubelet will restart it, but the files will be lost - the Container starts with a clean state. Second, when running Containers together in a Pod it is often necessary to share files between those Containers. The Kubernetes Volume abstraction solves both of these problems.

https://kubernetes.io/docs/concepts/storage/volumes/


## The application

Consider the given node application: https://github.com/charroux/kubernetes-volumes/tree/master/hello

Containing the main program : https://github.com/charroux/kubernetes-volumes/blob/master/hello/server.js

Where 

```javascript
app.use(express.static(__dirname+'/public'));
```

indicates that there in a public directory containing the html page.

Nevetheless the public directory is not present in the app: https://github.com/charroux/kubernetes-volumes/tree/master/hello

The goal of this example is to use a Kubernetes volumue containing such html code.

An example of a public directory is provided: https://github.com/charroux/kubernetes-volumes/tree/master/public

So the question is: how to mount an existing directory inside a Kuberntes application that doesn't contain such a directory?

## Create a PersistentVolume

Here is a volume definition: https://github.com/charroux/kubernetes-volumes/blob/master/pv-volume.yaml

Check the host path: /root/Documents/volumes/public

This should be an existing directory in the local machine, and IT MUST contain the content of: https://github.com/charroux/kubernetes-volumes/tree/master/public

Then, create the local volume using:

```java
kubectl apply -f pv-volume.yaml 
```

And check if it has been created:

```java
kubectl get pv
```

## Create a PersistentVolumeClaim

There are many kinds of persistent volumes:
- Amazon -> AWSElasticBlockStore
- Google -> GCEPersistentDisk
- NFS
- local
- ...

This example uses a local volume.

However, how to deal with those different persistent volumes? The soution is to use a persistent volume claim (PVC).

Here is an example of a Persistent Volume Claim: https://github.com/charroux/kubernetes-volumes/blob/master/pv-claim.yaml

As you can see there is no reference to the underlying Persistent Volume. Hence, a Persistent Volume Claim is just a request to an existing Persistent Volume, whatever this kind is.

Create a Persistent Volume Claim:

```java
kubectl apply -f pv-claim.yaml 
```

Then check is state: 

```java
kubectl get pvc
```

## Create a Pod

https://github.com/charroux/kubernetes-volumes/blob/master/pv-pod.yaml

```java
kubectl apply -f pv-pod.yaml 
```

```java
kubectl get pod angular-pv-pod
```

util the pod is running.

Enter the pod in an interactive way:

```java
kubectl exec -it angular-pv-pod -- /bin/bash
```

```java
ls /usr/share/images
```

Check if the directory has been monted: the image should be present.

kubectl describe pods

read the IP address: IP:           10.42.0.126

Test with: http://10.42.0.126:8080/

## delete resource

Delete pods first.

Persistant volumes are protected against removal. So before removal the protection must be udpated.

Edit the pv:

kubectl edit pvc YOUR_PVC -n NAME_SPACE

Manually edit and put # before this line: -kubernetes.io/pvc-protection

i to insert
ESC then :w to save
ESC then :q to quit



