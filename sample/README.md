## kubectlコマンドについて

### Context，NameSpaceの切り替え
```
$ kubectl config use-context [context_name]
$ kubectl config set-context [context_name] --namespace=[namespace]

# kubectx,kubensを使うと便利
$ kubectx [context_name]
$ kubens [namespace_name]
```

### リソースを作成する
```
# コマンドから
$ kubectl run --image=nginx:1.12 sample-deployment --replicas 3
# コマンドから(一部のリソースのみ)
$ kubectl create [resource] [name]
# マニフェストファイルから
$ kubectl apply -f [manifestfile_path]
# ディレクトリ内のファイルを適用
# -Rで再帰的に適用
$ kubectl apply -f ./ -R

# Tips: --dry-runでドライランできる マニフェストファイル作りをすぐに終わらせられる
$ kubectl run --image=nginx:1.12 sample-deployment --replicas 3 --dry-run -o yaml
``` 

### リソースの情報を取得する
```
$ kubectl get [resource]
# -o wideでちょっと多めの情報量，-o yamlや-o jsonで任意のファイル形式で出力
$ kubectl get [resource_name] -o wide

# ログを確認
$ kubectl logs [pod_name]

# 詳細を表示
$ kubectl describe [resource] [name]

# マシンリソースの使用量
$ kubectl top [resource]
```

### リソースの情報を編集したい
```
# setを使って一部だけ変更
# 例
$ kubectl set image deployment/nginx nginx=nginx:1.9.1

# エディタによる編集
$ kubectl edit [resource] [name]
```

### リソースを削除する
```
$ kubectl delete [resource] [name]
$ kubectl delete -f [file_path]
```

### 使えるリソースを確認したい
```
$ kubectl api-resources
```