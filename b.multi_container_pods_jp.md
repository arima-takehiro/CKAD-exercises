![](https://gaforgithub.azurewebsites.net/api?repo=CKAD-exercises/multi_container&empty)
# Multi-container Pods (10%)   

### 二つのコンテナを含むPodを作成する。両方ともイメージはbusyboxでコマンド"echo "hello"; sleep 3600"を実行する。二つ目のコンテナに接続し、lsコマンドを実行する。  
最も簡単な方法で一つ目のコンテナを作成し、yamlファイルに保存しておく。  
```bash
kubectl run firstpod --image=busybox --restart=Never -o yaml --dry-run -- /bin/sh -c 'echo hello; sleep 3600' > mypod.yaml
kubectl apply -f mypod.yaml
kubectl exec -it secondpod -- /bin/sh
```

