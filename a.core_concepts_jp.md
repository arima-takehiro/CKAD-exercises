# Core Concept(13％)

kubernetes.io > Documentation > Reference > kubectl CLI > [kubectl チートシート](https://kubernetes.io/docs/reference/kuberctl/cheatsheet/)  

kubernetes.io > Documentation > Tasks > Monitoring, Logging, and Debugging > [実行中のコンテナのシェルにアクセスする](https://kubernetes.io/docks/tasks/debug-application-cluster/get-shell-running-container/)

kubernetes.io > Documentation > Tasks > Access Applications in a Cluster > [複数のクラスタへのアクセス（構成の切り替え）](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)

kubernetes.io > Documentation > Tasks > Access Applications in a Cluster > [クラスタへのアクセス](https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/) using API

kubernetes.io > Documentation > Tasks > Access Applications in a Cluster > [クラスタ内のアプリにアクセスするためにPortForwardingを使用する](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/)  

### NameSpaceとnginxイメージを使用したPodを作成する  
```bash
kubectl create ns mynamespace
kubectl run nginx --image=nginx --restart=Never -n mynamespace
```

### YAMLを使って記述されたPodを作る  

YAMLを簡単に作る方法  
```yaml
kubectl run nginx --image=nginx --restart=Never --dry-run=client -o yaml > pod.yaml
```

```bash
cat pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
    - name: nginx
      image: nginx
      imagePullPolicy: IfNotPresent
      name: nginx
      resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```
```bash
kubectl create -f pod.yaml -n mynamespace  
```

代わりに一行で書くと。。。     
```bash
kubectl run nginx --image=nginx --restart=Never --dry-run=client -o yaml | kubectl create -n mynamespace -f -
```  
`--dry-run=client`：オブジェクトを作ることなく、マニフェストが正しいか調べられる。  
`--restart=Never`：NeverはPodに障害が発生しても新しいPodを作成しないので、通常のPodが作成される。  
`--restart=Always`：AlwaysはPodが常に動き続けるように新しいPodを生成するので、Deploymentが作成される。  
`--restart=OnFailure`：OnFailureはPodに障害が起きた時に新しいPodを生成するので、jobが生成される。  


### envコマンドを実行するbusyboxPodを作成する。実行し、出力を見る。  
```bash
kubectl run busybox --restart=Never --image=busybox -it -- env 
or 
kubectl run busybox --image=busybox --restart=Never --command -- env
kubectl logs busybox
```  

### YAMLを使用してenvコマンドを実行するbusyboxPodを作成する。実行し、出力を見る。  
```bash
kubectl run busybox --restart=Never --image=busybox --dry-run -o yaml --command -- env > pod.yaml  
vi pod.yaml
```  
```yaml
apiVersion: v1
kind: Pod
metadata: 
    creationTimestamp: null
    labels:
      run: busybox
    name: busybox
spec:
    restartPolicy: Never
    containers:
    - name: busybox
      image: busybox
      resources:{}
      command: 
      - env
```  

### mynsと言う新しいNameSpaceを作成しないでYamlで出力する。  
```bash
kubectl create namespace myns -o yaml --dry-run
```

### myrqと言うResourceQuotaを作成しないでYamlで出力する。(limit cpu:1000m, memory: 1Gi, )  
```bash
kubectl create quota myrq --hard=cpu=1,memory=1G,pods=2 -o yaml --dry-run
```  

### 全てのnamespaceのpodを取得する。  
```bash
kubectl get po --all-namespaces
```  

### nginxというnginxPodを作成する。port=80を許可する。  
```bash
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml --port=80
```  

### podのイメージをnginx:1.7.1に変更する。イメージが取得できたら既存のpodを削除して、新しいpodを即座に再作成する。    
```bash
# kubectl set image RESOURCE RESOURCENAME CONTAINERNAME=NEWIMAGE
kubectl set image pod nginx nginx=nginx1.7.1
kubectl describe pod nginx
kubectl get po nginx -w
```
### 前回のステップで使用したbusyboxPodを使用して、Podの/に接続する。　　
```bash
# getでipAddressを出力させる。  
kubectl get po -o wide
kubectl run busybox --image=busybox --restart=Never -it --rm -- wget IPAddress
```  

### 現在実行中のPodをYamlで出力させる  
```bash
kubectl get po PODNAME -o yaml
```  

### Podのログを出力する  
```bash
kubectl logs nginx
```  

### Podが壊れたか再起動した時、前回のインスタンスのログを出力させる。 
```bash
kubectl logs nginx -p
```  

### Podのシンプルなシェルを実行する  
```bash
kubectl exec nginx -it -- /bin/sh
```  

### busyboxPodを作成し、hello,worldを出力して終了するPodを作成する。  
```bash
kubectl run busybox --restart=Never --image=busybox -it -- /bin/sh -c 'echo hello world'
```

### Podを作成する時に環境変数として "var1=val1" を登録して、podの中にセットされているか確認する。　　
```bash
kubectl run nginx --image=nginx --restart=Never --env=var1=val1
kubectl exec -it nginx -- env
or
kubectl exec -it nginx -- /bin/sh```
