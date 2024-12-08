
In Kubernetes, **Volumes** are used to provide storage to containers in a Pod. They enable data sharing between containers and ensure data persists beyond the container's lifecycle, depending on the volume type.

Volume Types: hostPath and emptyDir

**1. hostPath Volume**
  * A hostPath volume mounts a file or directory from the node's filesystem into a Pod.
  * This allows Pods to access and use files on the node where they are running.
**Key Features:**
  * Provides direct access to the node's filesystem.
  * Can be used for storing logs, accessing host-specific configurations, or running low-level system tools.

**2. emptyDir Volume**
  * An emptyDir volume provides a temporary storage directory that is created when the Pod is assigned to a node and deleted when the Pod is removed from that node.
**Key Features:**
  * Data exists as long as the Pod is running.
  * Completely ephemeral and deleted along with the Pod.

------------------------------------------------------------------------------------------------------
## Task 1: Creating Pod with hostPath

Create a file named mypod-hp.yaml using the content given below
```
vi mypod-hp.yaml
```
Add the given content, by pressing `INSERT`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod-hp
spec:
  containers:
  - image: nginx:latest
    name: con1
    ports:
    - containerPort: 80
    volumeMounts:
    - name: myvol
      mountPath: /data
  volumes:
  - name: myvol
    hostPath:
      path: /data
  ```
save the file using `ESCAPE + :wq!`

Apply the Pod yaml created in the previous step
```
kubectl apply -f mypod-hp.yaml
```
View the objects created by Kubernetes Pod and verify hostPath mounting in POD
```
kubectl get Pods
```
```
kubectl get pods -o wide
```
From the above take a note on which pods are running on which particular node, as this would be required in the below steps.
```
kubectl describe pod <pod-name>
```
Access shell on a container running in your Pod to verify volume
```
kubectl exec -it <pod-name> -- /bin/bash
```
```
cd /data
```
```
echo `Welcome to DevOps Training` > index.html
```
```
exit
```
Now delete your Pod
```
kubectl delete -f mypod-hp.yaml --force
```
Now ssh into the Node on which the particular pod( in which you went inside and created the index.html file) was running.
```
ssh ubuntu@<External-IP>
```
Navigate to the Source location. In this case /data
```
cd /data
```
```
ls -la
```
You would see the index.html file still existing
```
cat index.html
```
```
exit
```


## Task 2 : Creating Pod with emptyDir
Create a file named mydep-empty.yaml using content given below
```
vi mypod-empty.yaml
```

Add the given content, by pressing `INSERT`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod-empty
spec:
  containers:
  - image: nginx:latest
    name: con1
    ports:
    - containerPort: 80
    volumeMounts:
    - name: empty-vol
      mountPath: /data
  - image: tomcat:latest
    name: con2
    ports:
    - containerPort: 80
    volumeMounts:
    - name: empty-vol
      mountPath: /opt/data
  volumes:
  - name: empty-vol
    emptyDir: {}
  ```
Save and exit the editor
Apply the Pod yaml created in the previous step
```
kubectl apply -f mypod-empty.yaml
```
View the objects created by Kubernetes Pod and verify hostPath mounting in POD
```
kubectl get Pods
```
```
kubectl get pods
```
```
kubectl describe pod mypod-empty
```
Access shell on a container running in your Pod to verify volume
```
kubectl exec -it mypod-empty -c con1 -- /bin/bash
```
```
exit
```
```
kubectl exec -it mypod-empty -c con1 -- /bin/bash
```
```
exit
```

## Task 3: Cleanup the resources using the below command
Delete the resources created during the lab:
```
kubectl delete -f mypod-empty.yaml
```
```
kubectl delete -f mypod-hp.yaml
```
Ignore if already deleted
