![](https://gaforgithub.azurewebsites.net/api?repo=CKAD-exercises/observability&empty)
# Observability (18%)  

## LivenessProbe and ReadinessProbe  

kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Configure Liveness and Readiness Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/)

### 'ls'コマンドを実行するLivenessProbeを持つPodを作成する。  
```yaml
apiVersion: v1
kind: Pod
metadata:
  name:
  labels:
spec:
  containers:
    - name: nginx
      image: nginx
      livenessProbe:
        exec:
          command: 
          - ls
```  
```bash
# check livehessProbe
kubectl describe pod nginx | grep -i liveness
```  

### mypod.yamlを修正して開始して5秒後にprobeがスタートして5秒間隔でprobeするようにする  
```yaml
spec:
  containers:
  - name: nginx
    image: nginx
    livenessProbe:
      initialDelaySeconds: 5
      periodSeconds: 5
      exec:
        command:
        - ls
```  

### HTTPを使ってreadinessProbeを実装する（path '/', port 80)  
```yaml
spec:
  containers: 
  - name: nginx
    image: nginx
    readinessProbe:
      httpGet:
        path: /
        port: 80
    ports:
      containerPort: 80
```  

## Logging  

```bash
kubectl run busybox --image=busybox --restart=Never -- /bin/sh -c Q. 
kubectl logs busybox -f
```   




