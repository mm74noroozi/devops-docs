# devops-docs
## k8s update strategies
- canary : some new version pods with keeping old versions
- recreate : depricate all old pods while deploying new pods (used when system is not compatible with two versions at once)
- rolling : partially update pods and after health check depricate preivious ones.
## deployments vs statefulsets
in statefulset pods have states and can not easily be replacible. they have unique netvork ID and Storage ID
example:
```YAML
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql-set
spec:
  selector:
    matchLabels:
      app: mysql
  serviceName: "mysql"
  replicas: 3
  template:
    metadata:
      labels:
        app: mysql
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: mysql
        image: mysql:5.7
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: mysql-store
          mountPath: /var/lib/mysql
        env:
          - name: MYSQL_ROOT_PASSWORD
            valueFrom:
              secretKeyRef:
                name: mysql-password
                key: MYSQL_ROOT_PASSWORD
  volumeClaimTemplates:
  - metadata:
      name: mysql-store
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: "linode-block-storage-retain"
      resources:
        requests:
          storage: 5Gi
```
result:
```
kubectl get pods

NAME          READY   STATUS      RESTARTS   AGE
mysql-set-0   1/1         Running        0                 142s
mysql-set-1   1/1         Running        0                 132s
mysql-set-2   1/1         Running        0                 120s
```
## create service account
```
>> kubectl create serviceaccount jenkins
>> kubectl describe serviceaccount jenkins

Name:                jenkins
Namespace:           default
Labels:              <none>
Annotations:         <none>
Image pull secrets:  <none>
Mountable secrets:   jenkins-token-rfbzr
Tokens:              jenkins-token-rfbzr
Events:              <none>

>> kubectl describe secret jenkins-token-rfbzr
Name:         jenkins-token-rfbzr
Namespace:    default
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: jenkins
              kubernetes.io/service-account.uid: ac331ed8-bde2-4507-b896-73ead8d40d65

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1029 bytes
namespace:  7 bytes
token:      eyJhbGciOiJS...
```
