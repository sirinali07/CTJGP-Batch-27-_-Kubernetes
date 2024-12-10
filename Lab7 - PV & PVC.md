## Static Provisioning

### Task 1: Get Node Label and Create Custom Index.html on Node
add the label to worker node
```
kubectl label node node-1 role=node
```
View worker nodes and their labels
```
kubectl get nodes --show-labels | grep role=node
```
Make a note of the kubernetes.io/hostname label of one of the nodes and ssh to one of the nodes using below command
```
ssh -i <.pem key> ubuntu@<node_public_IP> 
```
Note: Make sure you `.pem` key with you


Switch to root and run the following commands. A directory with custom index.html is created for PersistentVolume mount 
```
sudo su
```
```
mkdir /pvdir
```
```
echo Hello World! > /pvdir/index.html
```

### Task 2: Create a Local Persistent Volume
```
vi pv-volume.yaml
```
```yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: pv-volume
spec:
  storageClassName: manual
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: "/pvdir"
```
```
kubectl apply -f pv-volume.yaml
```
```
kubectl get pv
```
```
kubectl describe pv pv-volume
```

### Task 3: Create a PV Claim
```
vi pv-claim.yaml
```
```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pv-claim
spec:
  storageClassName: manual
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
```
```
kubectl apply -f pv-claim.yaml
```

### Task 4: Create nginx Pod with NodeSelector
```
vi pv-pod.yaml
```
```yaml
kind: Pod
apiVersion: v1
metadata:
  name: pv-pod
spec:
  volumes:
    - name: pv-storage
      persistentVolumeClaim:
        claimName: pv-claim
  containers:
     - name: pv-container
       image: nginx
       ports:
          - containerPort: 80
            name: "http-server"
       volumeMounts:
          - mountPath: "/usr/share/nginx/html"
            name: pv-storage
  nodeSelector:
    role: node
```
Apply the Pod yaml created in the previous step
```
kubectl apply -f pv-pod.yaml
```
View Pod details and see that is created on the required node
```
kubectl get pods -o wide
```
Access shell on a container running in your Pod
```
kubectl exec -it pv-pod -- /bin/bash
```
```
exit
```
### Task 5: Cleanup the resources
delete the resources created in this lab.
```

kubectl delete -f pv-pod.yaml
```
```
kubectl delete -f pv-claim.yaml
```
```
kubectl delete -f pv-volume.yaml
```
## Dynamic Provisioning

The **EBS CSI** driver enables Kubernetes to interact with AWS EBS for provisioning, attaching, and managing persistent storage dynamically.

A **StorageClass** in Kubernetes is a powerful abstraction layer that defines how storage resources (like AWS EBS, Google Persistent Disks, Azure Disks, or others) should be provisioned and managed dynamically. It simplifies and automates the creation and management of persistent storage in a Kubernetes cluster by providing predefined policies for storage provisioning.

### Task 1: Check CSI and Storage Class

Verify Installed EBS CSI
```bash
kubectl get pods -A | grep ebs
```
Ensure the pods related to aws-ebs-csi-driver are running successfully.

Verify the available Storage Class
```bash
kubectl get sc
```
### Task 2: Create a PersistentVolumeClaim (PVC) using below yaml

```
vi pvc.yaml
```

Add the given content, by pressing `INSERT`

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ebs-dynamic-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: gp2
```
save the file using `ESCAPE + :wq!`

Apply the definition yaml
```
kubectl apply -f pvc.yaml
```

Verify the PVC
```bash
kubectl get pvc
```
Ensure the PVC is Bound to a dynamically provisioned PV.


### Task 3: Create a Pod that Uses the PVC

```
vi pod.yaml
```

Add the given content, by pressing `INSERT`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ebs-dynamic-app
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - mountPath: "/usr/share/nginx/html"
      name: ebs-volume
  volumes:
  - name: ebs-volume
    persistentVolumeClaim:
      claimName: ebs-dynamic-pvc
```
save the file using `ESCAPE + :wq!`

Apply the definition yaml
```
kubectl apply -f pod.yaml
```

Verify the POD
```bash
kubectl get po
```
Ensure the Pod is running and the volume is mounted successfully.

### Task 4: Test the Persistent Volume

Access the Pod
```bash
kubectl exec -it ebs-dynamic-app -- bash
```

Navigate to the Mounted Directory
```bash
cd /usr/share/nginx/html
```

Create a Test File
```bash
echo "Hello from EBS-backed storage!" > index.html
```

Exit the Pod
```bash
exit
```

### Task 5: Cleanup the resources
Delete the pod

```bash
kubectl delete -f pod.yaml
```
Delete the pvc

```bash
kubectl delete -f pvc.yaml
```
