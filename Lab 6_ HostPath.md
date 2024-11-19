## Task 1: Creating Deployment with hostPath
Create a file named mydep-hp.yaml using the content given below
```
vi mydep-hp.yaml
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mydep-hp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: mydep
  template:
    metadata:
      labels:
        app: mydep
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
Save and exit the editor and Apply the Deployment yaml created in the previous step
```
kubectl apply -f mydep-hp.yaml
```
View the objects created by Kubernetes Deployment and verify hostPath mounting in POD
```
kubectl get deployments
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
Now delete your deployment
```
kubectl delete -f mydep-hp.yaml --force
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

