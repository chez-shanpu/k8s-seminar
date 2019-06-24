# 問題

## 初級
- 全てのネームスペースのPodの情報を一括で表示してください．
- Podのラベル情報を表示してください．
- PodがどのNodeにスケジューリングされているかわかるようにPodの情報を表示してください．
- 任意のPodのログを表示してください．
- `mynamespace`という名前のネームスペースを作成し，そのネームスペース内に`nginx`イメージを用いたPodを作成してください．
  - `mynamespace`を削除し，そもそも削除できるのか，そのネームスペースにあったPodはどうなるのか調べてください．
- `app=nginx`のラベルが付与された`nginx`イメージを用いたPodを作成してください．
  - ラベルを`app=web`に変更してください．

## 中級
- nginxイメージを用いたPodを作成し，Pod内のnginxコンテナに接続して`/usr/share/nginx/html/index.html`を書き変えてください．
- 1分ごとに`echo "HelloWorld"`をするJobを作成してください．
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
- レプリカ数3，`nginx`イメージを使った，アップデート時に全てのPodを一度に削除してから新しく作るデプロイメントを作成してください．
- レプリカ数3，`nginx`イメージを使った，アップデート時に最低3つのPodは動いており最大でも4つのPodが動いているようなデプロイメントを作成してください．
- 一つのノードに`development=true`というラベルを付与してください
  - `nginx`イメージを使ったレプリカ数1のデプロイメントを作成し，その際必ず`development=true`というラベルがついたノードにスケジューリングされるようにしてください（ラベルの値は文字列であることに注意）
  - ノードのラベル`development=true`を`development=false`に変更してください．その時の上で作成したPodの挙動を確認してください．
  - 上で作成したPodを削除してください．その時のPodの挙動を確認してください．

## 上級
- 80番ポートを開いた`nginx`イメージを用いたPodを作成してください．
  - そのPodに対してClusterIPを割り当ててください．
  - `ubuntu`イメージを用いたPodを作成し，上で作成したPodのnginxコンテナの80番ポートに対してcurl等を用いてGETリクエストを送ってください．
- レプリカ数3の`nginx:1.12`イメージを用いたデプロイメントを作成してください．
  - イメージを`nginx:latest`に変更してください．
  - デプロイメントのローリングアップデートの情報を表示してください．
  - `nginx:1.12`にロールアウトしてください．
  - 再度ロールアウトしてください．その後`nginx`イメージのバージョンを確認してください．