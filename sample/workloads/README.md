## Tips

### Pod
- pod内のコンテナにログインしたい

```
$ kubectl exec -it [pod_name] /bin/bash
```

### Deployment
- Deploymentの変更履歴を見たい

```
 $ kubectl rollout history deployment [deployment_name]
 # apply時に--recordオプションを渡した場合はCHANGE-CAUSEにも記載されている
 ```

- Deploymentのロールバックをしたい
```
# 一つ前に戻る
$ kubectl rollout undo deploy [deployment_name]

# revisonを指定して戻る
$ kubectl rollout undo deploy [deployment_name] --to-revision 1
```

- Deploymentの更新を一時的に中断してほしい
```
# 更新の一時停止
$ kubectl rollout pause deployment [deployment_name]

# 更新の再開
$ kubectl rollout resume deployment [deployment_name]
```