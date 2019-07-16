# 問題と答え

## 初級
- 全てのネームスペースのPodの情報を一括で表示してください．
<details><summary>answer</summary>
<p>

```
$ kubectl get pods --all-namespaces
```
</p>
</details>

- Podのラベル情報を表示してください．
<details><summary>answer</summary>
<p>

```
$ kubectl get pods --show-labels
```
</p>
</details>

- PodがどのNodeにスケジューリングされているかわかるようにPodの情報を表示してください．
<details><summary>answer</summary>
<p>

```
$ kubectl get pods -o wide
```
</p>
</details>

- 任意のPodのログを表示してください．
<details><summary>answer</summary>
<p>

```
# <pod_name>にPodの名前を入れる
$ kubectl logs <pod_name>
```
</p>
</details>

- `mynamespace`という名前のネームスペースを作成し，そのネームスペース内に`nginx`イメージを用いたPodを作成してください．
  - `mynamespace`を削除し，そもそも削除できるのか，そのネームスペースにあったPodはどうなるのか調べてください．
<details><summary>answer</summary>
<p>

```
# nsはnamespaceの省略形
$ kubectl create ns mynamespace
# --restart=Neverを省くとDeploymentリソースとして作成される
# マニフェストファイルを作成してそれをapplyする形でも大丈夫です
$ kubectl run nginx --image=nginx --restart=Never -n mynamespace

$ kubectl delete ns mynamespace
# ネームスペースを削除するとそのネームスペースに紐づいたリソースは全て消し飛びます（うっかり消してしまわないように注意）
# そのため何か一時的に試したいことがある場合，`hoge-test`のようなネームスペースを作成して最後そのネームスペースを削除すると消し忘れがなく楽です
```
</p>
</details>

- `app=nginx`のラベルが付与された`nginx`イメージを用いたPodを作成してください．
  - ラベルを`app=web`に変更してください．
<details><summary>answer</summary>
<p>

```
# マニフェストファイルを作成してそれをapplyする形でも大丈夫です
$ kubectl run nginx --image=nginx --restart=Never --labels=app=nginx

$ kubectl label po nginx app=web --overwrite 
# `kubectl get po --show-labels`で確認してみるとよいです
```
</p>
</details>

## 中級
- nginxイメージを用いたPodを作成し，Pod内のnginxコンテナに接続して`cat` 等のコマンドを用いて`/usr/share/nginx/html/index.html`の中身を見てください．
<details><summary>answer</summary>
<p>

```
# マニフェストファイルを作成してそれをapplyする形でも大丈夫です
$ kubectl run nginx --image=nginx --restart=Never

$ kubectl exec nginx -it /bin/bash  
# 中でごにょごにょする
```

`/usr/share/nginx/html/index.html`の中身
```html /usr/share/nginx/html/index.html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
</p>
</details>

- 1分ごとに`echo "HelloWorld"`をするJobを作成してください．
<details><summary>answer</summary>
<p>

```
# kubectl run はDeprecated（将来的に消されます）
$ kubectl run hello-cronjob --image=ubuntu --restart=OnFailure --schedule="*/1 * * * *"   -- /bin/sh -c 'echo "HelloWorld"'

# kubectl createを使った場合
$ kubectl create cronjob hello-cronjob --image=ubuntu --schedule="*/1 * * * *" -- /bin/sh -c 'echo "HelloWorld"'
```

マニフェストをapplyする場合のマニフェスト例
```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  creationTimestamp: null
  name: hello-cronjob
spec:
  jobTemplate:
    metadata:
      creationTimestamp: null
      name: hello-cronjob
    spec:
      template:
        metadata:
          creationTimestamp: null
        spec:
          containers:
          - command:
            - /bin/sh
            - -c
            - echo "HelloWorld"
            image: ubuntu
            name: hello-cronjob
            resources: {}
          restartPolicy: OnFailure
  schedule: '*/1 * * * *'
```
</p>
</details>

- 以下のマニフェストに基づくPodはスタートしません（たぶん）．原因を突き止めてください．
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cannot-start-deploy
spec:
  replicas: 1
  selector:
    matchLabels:
      app: sample-app
  template:
    metadata:
      labels:
        app: sample-app
    spec:
      containers:
        - name: nginx-container
          image: nginx:1.12
          resources:
            requests:
              memory: "10Gi"
              cpu: "500m"
            limits:
              memory: "15Gi"
              cpu: "1000m"
```
<details><summary>answer</summary>
<p>

```
# kubectl describeでそれぞれのリソースの詳細を取得することができます
# 下にあるEventsを見てみましょう
$ kubectl describe po <pod_name>
```
原因はメモリ不足です．
</p>
</details>

- レプリカ数3，`nginx`イメージを使った，アップデート時に全てのPodを一度に削除してから新しく作るデプロイメントを作成してください．
<details><summary>answer</summary>
<p>

`Deployment.spec.strategy`で`Recreate`を選択します
マニフェスト例
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-recreate
spec:
  strategy:
    type: Recreate
  replicas: 3
  selector:
    matchLabels:
      app: sample-app
  template:
    metadata:
      labels:
        app: sample-app
    spec:
      containers:
        - name: nginx-container
          image: nginx:latest
```
</p>
</details>

- レプリカ数3，`nginx`イメージを使った，アップデート時に最低3つのPodは動いており最大でも4つのPodが動いているようなデプロイメントを作成してください．
<details><summary>answer</summary>
<p>

`Deployment.spec.strategy`で`RollingUpdate`を選択します
`maxUnavailable`は稼働状態のPodがレプリカ数から最大何個マイナスになってもよいか，
`maxSurge`は稼働状態のPodがレプリカ数から最大何個プラスになってもよいかを定義します
マニフェスト例
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-rollingupdate
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1
  replicas: 3
  selector:
    matchLabels:
      app: sample-app
  template:
    metadata:
      labels:
        app: sample-app
    spec:
      containers:
        - name: nginx-container
          image: nginx:latest
```
</p>
</details>

- 一つのノードに`development=true`というラベルを付与してください
  - `nginx`イメージを使ったレプリカ数1のデプロイメントを作成し，その際必ず`development=true`というラベルがついたノードにスケジューリングされるようにしてください（ラベルの値は文字列であることに注意）
  - ノードのラベル`development=true`を`development=false`に変更してください．その時の上で作成したPodの挙動を確認してください．
  - 上で作成したPodを削除してください．その時のPodの挙動を確認してください．
<details><summary>answer</summary>
<p>

```
# noはnodeの省略形です
$ kubectl label no <node_name> development=true

# マニフェストをapply
$ kubectl apply -f <file_path>

# ラベルを変更する
$ kubectl label no <node_name> development=false --overwrite
# このときPodはNodeSelectorの条件に当てはまらなくなりますがRunningのままです
# NodeSelectorの条件が参照されるのはあくまでスケジューリング時なので，その後nodeのラベルが変更されても改めてスケジューリングが行われることはありません

$ kubectl delete po <pod_name>
# 様子をみる
$ kubectl get po -w
# PodがPendingのままになります
# より詳細を見たい場合は`kubectl describe`してみるとよいです
```

マニフェスト例
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dev-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: sample-app
  template:
    metadata:
      labels:
        app: sample-app
    spec:
      containers:
        - name: nginx-container
          image: nginx:latest
      nodeSelector:
        development: "true"
```
</p>
</details>

## 上級
- 80番ポートを開いた`nginx`イメージを用いたPodを作成してください．
  - `ubuntu`イメージを用いたPodを作成しそのPod内にログインし，上で作成したPodのnginxコンテナの80番ポートに対してcurl等を用いてGETリクエストを送ってください．
  - Service/NodePortを作成してください．このときServiceの8080番ポートをPodの80番ポートに紐づけるようにしてください．
  - Cluster外部(NodeにSSHもしない)から適切なポート番号を用いてnginx PodにGETリクエストを送ってください．
<details><summary>answer</summary>
<p>

```
# Podを作成
$ kubectl run nginx --image=nginx:latest --restart=Never --port=80
$ kubectl run ubuntu --image=ubuntu:latest --restart=Never

# PodのIPを確認する
$ kubectl get po -o wide

# コンテナにログイン
$ kubectl exec ubuntu -it /bin/bash
root@ubuntu $ apt update 
root@ubuntu $ apt install curl
root@ubuntu $ curl {Pod_IP}:80

# NodePortを作成
$ kubectl create service nodeport my-ns --tcp=8080:80
# selectorに該当するようにPodにラベルを追加する
$ kubectl edit po nginx

# nodeのIP調べる
$ kubectl get no -o wide

# NodePortの開いているノードのポート番号を調べる
$ kubectl get svc

# GETする
$ curl {nodeのIP}:{nodeのPort} 
```
</p>
</details>

- レプリカ数3の`nginx:1.12`イメージを用いたデプロイメントを作成してください．
  - イメージを`nginx:latest`に変更してください．
  - デプロイメントの変更履歴を表示してください．
  - `nginx:1.12`にロールアウトしてください．
  - 再度ロールアウトしてください．その後`nginx`イメージのバージョンを確認してください．
<details><summary>answer</summary>
<p>

```
# deploymentの作成
$ kubectl apply -f <file_path> --record

# deploy.spec.template.spec.containers.imageを書き換える
$ kubectl edit deployment <deployment_name>

# デプロイメントの変更履歴を見る
$ kubectl rollout history deployment <deployment_name>

# ロールアウトする
$ kubectl rollout undo deploy <deployment_name>

# もう一度
$ kubectl rollout undo deploy <deployment_name>
# これを行うと最初にロールアウトを行う前の状態になる(kubectl get deploymentなどで確認してみるとよい)
# 二つ以上前に戻りたい場合は --to-revision でリビジョンを指定してあげる必要がある
```

マニフェスト例
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-demo
spec:
  replicas: 3
  selector:
    matchLabels:
      app: sample-app
  template:
    metadata:
      labels:
        app: sample-app
    spec:
      containers:
        - name: nginx-container
          image: nginx:1.12
```
</p>
</details>
