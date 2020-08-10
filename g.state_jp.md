![](https://gaforgithub.azurewebsites.net/api?repo=CKAD-exercises/state&empty)
# State Persistence(8%)

kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Configure a Pod to Use a Volume for Storage](https://kubernetes.io/docs/tasks/configure-pod-container/configure-volume-storage/)

kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Configure a Pod to Use a PersistentVolume for Storage](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/)

## Volumesを定義する  
### 二つのコンテナを作成する(image:busybox, command: sleep 3600), 二つのコンテナの"/etc/foo"にemptyDirをマウントし、共有しているかをみる。  
```bash
kubectl run busybox --image=busybox --restart=Never --dry-run -o yaml -- sleep 3600 > mypod.yaml
```  

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  dnsPolicy: ClusterFirst
  restartPolicy: Never
  containers:
  - args:
    - /bin/sh
    - -c
    - sleep 3600
    image: busybox
    imagePullPolicy: IfNotPresent
    name: busybox
    resources: {}
    volumeMounts: #
    - name: myvolume #
      mountPath: /etc/foo #
  - args:
    - /bin/sh
    - -c
    - sleep 3600
    image: busybox
    name: busybox2 # don't forget to change the name during copy paste, must be different from the first container's name!
    volumeMounts: #
    - name: myvolume #
      mountPath: /etc/foo #
  volumes: #
  - name: myvolume #
    emptyDir: {} #
```  
emptydirが共有されている。
```bash
kubectl exec -it busybox -c busybox2 -- sh
```  

### 10MBのpersistentVolumeを作成する、アクセスモードは"ReadWriteOnce"と"ReadWriteMany"で、storageClassはnormal, hostPathのmountPathは/etc/foo  
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: myvolume
spec:
  capacity:
    storage: 10Mi
  accessModes:
    - ReadWriteOnce
    - ReadWriteMany
  storageClassName: normal
```
