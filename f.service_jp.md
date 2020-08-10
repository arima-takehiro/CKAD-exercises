![](https://gaforgithub.azurewebsites.net/api?repo=CKAD-exercises/services&empty)
# Services and Networking (13%)

### nginxPodを作成してポート80で公開する  
```bash
kubectl run nginx --image=nginx --restart=Never --port=80 --expose
```  

### endpointのチェックをする  
```bash
kubectl get endpoint
```  

### ClusterIPを取得してbusyboxPodからアクセスしてみる  
```bash
kubectl get svc nginx
kubectl run tmpbusybox --image=busybox --restart=Never --rm -it -- sh
wget -O- [ClusterIP]:[Port]
```  

### NodePortに変更してアクセスしてみる  
```bash
kubectl edit svc nginx
```  
```yaml
type: ClusterIP => NodePort # 変更する
```  
```bash
wget -O- clusterIP:port
```  

### fooDeploymentを作成する。image: 'dgkanatsions/simpleapp'  replicas: 3 , label : 'app=foo' , サービスは作らないでport8080を許可する
```bash
kubectl create deploy  foo --image=dgkanatsios/simpleapp 
kubectl run hoge --restart=Never --image=busybox --rm -it -- sh
wget -O- PODIP:PORT
```

### Deploymentを公開するserviceを作成する。サービスに接続するとreplicasに負荷分散されていることがわかる。  
```bash
kubectl expose deployment foo --port=6262 --target-port=8080
kubectl run ggg --restart=Never --image=busybox --rm -it -- sh
wget -O- SERVICE_CLUSTER_IP:PORT
```  

### NetworkPolicyを設定する  
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      run: nginx # 適用するPodを選択する
  ingress:
    - from:
        - podSelector:
            matchLabels:
              access: granted # 許可するPodを選択する

```  

### GKEクラスタでnetworkpolicyを有効にする  
```bash
gcloud container clusters update --update-addons=NetworkPolicy=ENABLED
gcloud container clusters update --enable-network-policy
```  

ingressで指定したラベルのPodからでないとアクセスできなくなる

