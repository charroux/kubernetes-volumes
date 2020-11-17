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

indicates that there is a public directory containing the html page.

Nevetheless the public directory is not present in the app: https://github.com/charroux/kubernetes-volumes/tree/master/hello

The goal of this example is to use a Kubernetes volumue containing the public part of a web site (html, images...).

An example of a public directory is provided: https://github.com/charroux/kubernetes-volumes/tree/master/public

So the question is: how to mount an existing directory inside a Kuberntes application that doesn't contain such a directory?

But first of all, this app has been alredy dockerised and the docker image published in the Docker Hub: https://hub.docker.com/r/efrei/hellonode

Endeed, by default Kubernetes retreives the docker images from the Docker Hub.

Look at https://github.com/charroux/kubernetes-volumes to discover how to dockerise an application.
 
## Create a PersistentVolume

Here is a volume definition: https://github.com/charroux/kubernetes-volumes/blob/master/pv-volume.yaml

Where the host path to the persistent volume is set to: /host

### Kubernetes Volumes on minikube

If you are using minikube you must do extra configuration. 
Indeed, minikube is running at the top of a virtualization system (virtual box as an example) 
and you must mount a local directory toward a munikube directory using:

mimikube mount .:/host

where . is the local directory (this one where you are typing the command) and /host is a directory inside the minikune virtual machine. 

The current directory may contain the html page be displayed by the application.
So, copy inside the current directory the content of: https://github.com/charroux/kubernetes-volumes/tree/master/public
or any other HTML pages.

### Kubernetes Volumes on other implementation than minikube

The host path in the persistent volume yaml file should be an existing directory in your local machine, and IT MUST contain the content of: https://github.com/charroux/kubernetes-volumes/tree/master/public or any other HTML pages.

## Creating the local volume

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

This example uses a local volume (which is not a good solution for production but a persistent claim allows to change for a more appropriate one).

However, how to deal with those different persistent volumes, and how to adapte the volume to different choices? The soution is to use a persistent volume claim (PVC).

Here is an example of a Persistent Volume Claim: https://github.com/charroux/kubernetes-volumes/blob/master/pv-claim.yaml

Oddly, there is no reference to the underlying Persistent Volume. Hence, a Persistent Volume Claim is just a request to an existing Persistent Volume, whatever this kind is. 

It's a way to adapt the persistence to an existing one.

Create a Persistent Volume Claim:

```java
kubectl apply -f pv-claim.yaml 
```

Then check is state: 

```java
kubectl get pvc
```

## Create a deployment

Now the last part: how to use a persistent volume for a deployment (see https://github.com/charroux/kubernetes to discover what a deployment is)?

https://github.com/charroux/kubernetes-volumes/blob/master/pv-deployment.yml

This deployment deploys the image of the node app coming from the Docker hub (https://hub.docker.com/r/efrei/hellonode):

```java
      containers:
      - name: node-pv-container
        image: efrei/hellonode:1
```

where the volume has just a reference to the persistent volume claim:

```java
      volumes:
      - name: node-pv-storage
        persistentVolumeClaim:
          claimName: website-pv-claim
```

And the mount path is an abstract directory pointing to the persistent volume: 

```java
        volumeMounts:
        - mountPath: /usr/src/app/public
          name: node-pv-storage
```

Now let's have a look again in the node app deployed by this deployment to check where is the reference to this abstract path: 

```javascript
app.use(express.static(__dirname+'/public'));
```

where __dirname is given by the Docker command WORKDIR in the docker file: https://github.com/charroux/kubernetes-volumes/blob/master/hello/Dockerfile

Deploy the app with 
```java
kubectl apply -f pv-deployment.yml 
```

Then check the pod deployed until its state is running: 

```java
kubectl get pods
```

You can interact with the container to check is the directory has been mounted:

```java
kubectl exec -it podsname -- /bin/bash
```

where podsname must be the name of the pod.

Simply list the content of the abstract directory:

```java
ls /usr/src/app/public
```

That's it! You should see the HTML content of the real directory on your machine.

Finaly, how to check the app in a web browser?

Get first the IP addresse using:

```java
kubectl describe pods
```

read the IP address: IP: 10.42.0.126 as an example

Test it with: http://10.42.0.126:8080/

Notice that this is the IP address of the pod and remerber that this addresse is ephemeral since a pod can be destroyed at any moment and a new one can be created on a new IP address. You must use a service in front of a pod to keep the same IP address (see https://github.com/charroux/kubernetes).

## delete resource

Volume are protected against removal and deleting a volume is a little bit tricky.

So before removal the protection must be udpated.

Edit the pv:

```java
kubectl edit pvc YOUR_PVC -n NAME_SPACE
```

Retreive all nameespaces with:

```java
kubectl get namespace
```

By default, the default namespace is used.

Manually edit and put # before this line: -kubernetes.io/pvc-protection

You enter inside a VI editor and you can edit it using the VI commands:

i to insert
ESC then :w to save
ESC then :q to quit

You should now be abble to detele the volume:

```java
kubectl delete pv --all
```

Then the volume claim:

```java
kubectl delete pvc --all
```

