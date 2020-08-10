![](https://gaforgithub.azurewebsites.net/api?repo=CKAD-exercises/configuration&empty)
# Configuration (18%)

## ConfigMaps  

kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Configure a Pod to Use a ConfigMap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/) 
 
### valueがfoo=lala,foo2=loloのconfigmapを作成する  
```bash
kubectl create cm config --from-literal=foo=lala --from-literal=foo2=lolo
```  

### fileからcomfigmapを作成する  
```bash
echo -e "foo3=lili\nfoo4=lele" > config.txt
kubectl create cm config --from-file=config.txt
```  

### .envファイルからconfigmapを作成する  
```bash
echo -e "var1-val1\n# this is a comment\n\nvar2=val2\n#anothercomment" > config2.env
kubectl create cm config2 --from-env-file=config2.env
```  

### ConfigMapからvalueを読み込んで、podの作成時に環境変数としてセットする。  
```bash
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > mypod.yaml
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
  - image: nginx
    name: nginx
    resources: {}
    env:
    - name: VAR1
      valueFrom:
        configMapKeyRef:
          name: config
          key: var1
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```  

### envFromを使用してファイルの環境変数を読み込んだPodを作成する  
```bash 
# create configMap
kubectl create cm anotherconfig --from-literal=var=foo --from-literal=var2=hoge
kubectl run nginx2 --image=nginx --restart=Never --dry-run -o yaml > mypod2.yaml
```  
```yaml
spec:
  containers:
  - name:
    image:
    envFrom:
    - configMapRef:
        name: anotherconfig
```  

### configMapをvolumeとして読み込む  
```yaml
apiVersion: v1
kind: Pod
metadata: 
  name: nginx
  labels: 
    env: test
spec:
  volumes:
  - name: myvolume
    configMap:
      name: config
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: myvolume
      mountPath: /etc/hoge
    
```  

## Security Context
kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Configure a Security Context for a Pod or Container](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)  

### userID 101 で実行されるnginxPodを作成する  
```yaml 
apiVersion: v1
kind: Pod
metadata:
  name: 
  labels: 
spec:
  securityContext:
    runAsUser: 101
  containers:
  - name: nginx
    image: nginx
    resources:
      request:
        cpu: 200m 
        memory: 4Gi
```
### capabilityに"NET_ADMIN"と"SYS_TIME"を追加したPodを作成する  
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels: 
    test : true
spec:
  containers:
  - image: nginx
    name: nginx
    resources:
      requests:
        cpu: 100m 
        memory: 5Gi
      limits: 
        cpu: 100m 
        memory: 5Gi
    securityContext: 
        capabilities:
          add: ["NET_ADMIN","SYS_TIME"]
```  

## Requests and Limits

kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Assign CPU Resources to Containers and Pods](https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-resource/)

### resourceRequestがcpu:100m, memory:512Mi, resourceLimitsがcpu:100m,memory:612Mi のpodを作成する  
```bash
kubectl run nginx --image=ngixn --restart=Never --requests='cpu=100m,memory=256Mi' --limits='cpu=200m,memory=512Mi'
```
or
```yaml
apiVersion: v1
kind: Pod
metadata: 
  name: nginx
  labels:
    env: prod
spec:
  containers:
  - resources:
      requests: 
        cpu: 100m
        memory: 512Mi
      limits:
        cpu: 200m 
        memory: 600Mi
    image: nginx
    name: nginx
    securityContext:
      runAsUser: 100
      capabilities:
        add: ["laj"]
```  

## Secret

kubernetes.io > Documentation > Concepts > Configuration > [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)

kubernetes.io > Documentation > Tasks > Inject Data Into Applications > [Distribute Credentials Securely Using Secrets](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/)

### secretを作成する  
```bash
# from literal
kubectl create secret generic mysecret --from-literal=pass=arima
# from file
kubectl create secret generic mysecret --from-file=myfile=myfilepath
```  

### 値をデコードする  
```bash
echo YWrtaW4k | base64 -d
```  

### 既存のsecretをマウントしたPodを作成する  
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  volumes: 
  - name: mysec
    secret:
      secretName: my-secret
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: mysec
      mountPath: /etc/pass
```  

### secretの値を環境変数として登録したPodを作成する  
```yaml
apiVersion: v1
kind: Pod
metadata: 
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    env:
    - name: PASS
      valueFrom:
        secretKeyRef:
          name: my-secret
          key: pass
```  

## ServiceAccount
### ServiceAccountを作成する  
```bash
kubectl create sa myacc
```

### ServiceAccountを使用したPodを作成する 
```bash
kubectl run nginx --image=nginx --restart=Never --serviceaccount=myacc 
```  

### ServiceAccountの削除  
```bash
kubectl delete sa myacc```


