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
>> kubectl get serviceaccount jenkins -o=jsonpath='{.secrets[0].name}'
'jenkins-token-rfbzr'
>> kubectl get secret jenkins-token-rfbzr -o=jsonpath='{.data.token}' |base64 -d 
'eyJhbGciOiJS...'
```
## Application monitoring (jaeger)
## ansible 
inventory
```
leafs:
  hosts:
    leaf01:
      ansible_host: 192.0.2.100
    leaf02:
      ansible_host: 192.0.2.110

spines:
  hosts:
    spine01:
      ansible_host: 192.0.2.120
    spine02:
      ansible_host: 192.0.2.130

network:
  children:
    leafs:
    spines:

webservers:
  hosts:
    webserver01:
      ansible_host: 192.0.2.140
      http_port: 80
      ansible_user: my_server_user
    webserver02:
      ansible_host: 192.0.2.150
  vars:
    ansible_user: my_global_server_user
  
datacenter:
  children:
    network:
    webservers:
