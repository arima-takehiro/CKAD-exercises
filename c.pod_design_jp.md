![](https://gaforgithub.azurewebsites.net/api?repo=CKAD-exercises/pod_design&empty)
# Pod design (20%)
    
## ラベルとアノテーション  
kubernetes.io > Documentation > Concepts > Overview > [Labels and Selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#label-selectors)  

### ３つのPod、nginx1,nginx2,nginx3を作成する。全てのpodにはラベルapp=v1がつく。  
```bash
kubectl run nginx1 --image=nginx --restart=Never --labels=app=v1
kubectl run nginx2 --image=nginx --restart=Never --labels=app=v1
kubectl run nginx3 --image=nginx --restart=Never --labels=app=v1
```  

### podの全てのラベルを見る。  
```bash
kubectl get po --show-labels
```  

### 既存のPodのラベルを変更する  
```bash
kubectl label pod nginx1 app=v2 --overwirte　　
```  

### ラベルのキーを指定してpodを表示する  
```bash
kubectl get po -L app
```  

### ラベルのキーバリューペアに一致するpodを表示する  
```bash
kubectl get po -l app=v2
``` 

### 前に作成したPodからラベルを取り外す  
```bash
kubectl label po nginx1 nginx2 app-
```

### accelerator=nvidia-tesla-p100のラベルを持つノードにpodを配置する  
```yaml
apiVersion: v1
kind: Pod
metadata: 
    name: nginx
    labels:
    - app: v1
      env: prod
spec:
    containers:
    - name: nginx-cont
      image: nginx
      resources:
        limits:
        - cpu: 100m 
          memory: 5Gi
        requests:
        - cpu: 100m
          memory: 4Gi
    nodeSelector:
      accerator: nvia-tesla-p100
```  

### nginx1,nginx2,nginx3に,"description='my description'"と言うアノテーションをつける。  
```bash
kubectl annotate pod nginx1 nginx2 nginx3 description='my description'
# check annotation
kubectl describe po nginx1 | grep -i 'annotations'
```  

### アノテーションを外す  
```bash
kubectl annotate pod nginx1 description-
```  

### nginx1, nginx2, nginx3をまとめて削除する  
```bash
kubectl delete po nginx{1,2,3}
```  

## Deployment

kubernetes.io > Documentation > Concepts > Workloads > Controllers > [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment)  

### nginxデプロイメントを作成する。(image: nginx1.7.1, replicas: 2, port: 80)  
```bash
kubectl run deployment nginx --image=nginx1.7.1 --restart=Always -dry-run -o yaml > mydeployment.yaml
```  

### デプロイメントのロールアウトがどのように行われているのか確認する。  
```bash
kubectl rollout status deploy nginx
```  
### デプロイメントのイメージをnginx1.7.9に変更する。  
```bash
kubectl set image deploy nginx nginx=nginx:1.7.9
```  

### nginx1.91と言うイメージでわざと間違って更新する  
```bash
kubectl set image deploy nginx nginx=nginx:1.91
kubectl get po => ErrImagePull
```  

### デプロイメントをスケールさせる  
```bash
kubectl scale deploy nginx --replicas=3
```  

### デプロイメントをオートスケールさせる  
```bash
kubectl autoscale deploy nginx --min=2 --max=6 --cpu-percent=70
```  

### デプロイメントのロールアウトを中断する  
```bash
kubectl rollout pause deploy nginx
```    

### デプロイメントのロールアウトを再開する  
```bash
kubectl rollout resume deploy nginx
```  

### デプロイメントを前のバージョンに戻す  
```bash
kubectl rollout undo deploy nginx
```  

### デプロイメントを指定のバージョンまで戻す。  
```bash
kubectl rollout undo deploy nginx --to-revision=2
```  

## Jobs  

### perlイメージを使用してjobを作成する。コマンドに、"perl -Mbignum=bpi -wle 'print bpi(2000)'"を指定して実行する。  
```bash
kubectl run pi --image=perl --restart=OnFailure -- perl -Mbignum=bpi -wle 'print bpi(2000)'
or
kubectl create job pi --image=perl -- perl -Mbignum=bpi -wle 'print bpi(2000)'
```  
  
### busyboxイメージを使用してjobを作成する。コマンドに、"echo hello; sleep 30; echo world"を指定して実行する  
```bash
kubectl run busybox --image=busybox --restart=OnFailure -- /bin/sh -c 'echo hello; sleep 30; echo world'
or
kubectl create job busybox --image=busybox -- /bin/sh -c 'echo hello; sleep 30; echo world'
```  

### 実行に30秒以上かかったら自動的に消去されるjobを作成する。  
```
kubectl create job test-job --image=busybox -- bin/sh -c 'echo hello; sleep 30; echo world' > myjob.yaml
# specのところにactiveDeadlineSecondsを設定する
```

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  name: test-job
spec:
  activeDeadlineSeconds: 30
  template:
    metadata:
      creationTimestamp: null
    spec:
      containers:
      - command:
        - /bin/sh
        - -c
        - echo hello; sleep 30; echo world
        image: nginx
        name: test-job
        resources: {}
      restartPolicy: Never
status: {}
```  

### 複数回処理が完了するまでpodを作成し、実行するJobを作成する。  
```yaml
spec:
    completions: 5
```  

### 複数回の処理を並列で実行させるJobを作成する  
parallelismとcompletionsを同時に指定した場合、parallelismが優先される。    
```yaml
spec:
    parallelism: 5
```  

### cronjob  
指定した時間おきにJobを実行してくれる。  
```bash
kubectl create cronjob busybox --image=busybox --schedule="*/1 * * * *" -- /bin/sh -c 'data; echo helloworld'
```  


### 





