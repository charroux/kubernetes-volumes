# kubernetes-volumes

## Create a PersistentVolume

https://github.com/charroux/kubernetes-volumes/blob/master/pv-volume.yaml

Check the host path in the file: /root/Documents/volumes/static/images

copy an image inside it

```java
kubectl apply -f pv-volume.yaml 
```

```java
kubectl get pv
```

## Create a PersistentVolumeClaim

https://github.com/charroux/kubernetes-volumes/blob/master/pv-claim.yaml

```java
kubectl apply -f pv-claim.yaml 
```

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



