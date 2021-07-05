# nodeapp
test_task

vim nodeapp.yaml  #Создаем Deployment файл
```
apiVersion : v1
kind: Pod
metadata:
  name: node
spec:
  containers:
    - name : nodeapp
      image: freedbka/nodeapp:latest
      ports:
        - containerPort: 3000
```
kubectl create deployment nodeapp --image=freedbka/nodeapp:latest #  Запустили Deployment  
kubectl get deployment,pods -l app=nodeapp # Проверяем запустился ли Deploymetn  
```
kubectl get deployment,pods -l app=nodeapp
NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nodeapp   1/1     1            1           27m

NAME                           READY   STATUS    RESTARTS   AGE
pod/nodeapp-84b899564c-22n2l   1/1     Running   0          27m
```
kubectl scale deployment nodeapp --replicas=2 # Поднимаем 2 реплики  
kubectl expose deployment nodeapp --type=LoadBalancer --port=3000 # Поднимеи сервис  
kubectl get services nodeapp # Проверяем сервис  
```
kubectl get services nodeapp
NAME      TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
nodeapp   LoadBalancer   10.109.102.103   <pending>     3000:31482/TCP   7m24s
```
minikube service nodeapp # запускаем сервис  
```
minikube service nodeapp
|-----------|---------|-------------|-----------------------------|
| NAMESPACE |  NAME   | TARGET PORT |             URL             |
|-----------|---------|-------------|-----------------------------|
| default   | nodeapp |        3000 | http://192.168.99.100:31482 |
|-----------|---------|-------------|-----------------------------|
🎉  Opening service default/nodeapp in default browser...
```
Я Немного поправил код нодовского вебсервера, что-бы убедиться в том, что, балансировка работает  
код:
```
const express = require('express')
const app = express()
const port = 3000
var os = require('os');

var networkInterfaces = os.networkInterfaces();
app.get('/', (req, res) => {
  res.send(networkInterfaces) # Заменил Hello world, На информаци о networkInterfaces
})

app.listen(port, () => {
  console.log(`Example app listening at http://localhost:${port}`)
})
```
На выходе получил 2 разных интерфейса eth0:
```
{"lo":[{"address":"127.0.0.1","netmask":"255.0.0.0","family":"IPv4","mac":"00:00:00:00:00:00","internal":true,"cidr":"127.0.0.1/8"}],"eth0":[{"address":"172.17.0.7","netmask":"255.255.0.0","family":"IPv4","mac":"02:42:ac:11:00:07","internal":false,"cidr":"172.17.0.7/16"}]}
```

```
{"lo":[{"address":"127.0.0.1","netmask":"255.0.0.0","family":"IPv4","mac":"00:00:00:00:00:00","internal":true,"cidr":"127.0.0.1/8"}],"eth0":[{"address":"172.17.0.6","netmask":"255.255.0.0","family":"IPv4","mac":"02:42:ac:11:00:06","internal":false,"cidr":"172.17.0.6/16"}]}
```
minikube addons enable ingress  
pwd /home/freedbka/Documents/test/5.kuber  
vim ingress.yml  
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  rules:
    - host: nodeapp.info
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: nodeapp
                port:
                  number: 3000
```
Добавим строку в vim /etc/hosts  
```
192.168.99.100  nodeapp.info

```
```
kubectl get ingresses
NAME              CLASS    HOSTS          ADDRESS          PORTS   AGE
example-ingress   <none>   nodeapp.info   192.168.99.100   80      2m37s
```
Если ввести в браузер http://nodeapp.info/ и пообновлять страницу, то можем заметить что переменная меняется, значит у нас получилось


