## AROクラスターのアップグレード

### 前準備

kubeadminユーザーとして、AROクラスターにログインして、「管理」の「クラスター設定」画面に移動します。AROクラスターのデフォルトだと、更新チャネルが未設定の状態なので、チャネルを設定します。未設定の横の鉛筆マークをクリックして、チャネル名(この例では、stable-4.10)を入力して、「保存」をクリックします。

![更新チャネルの設定](./images/channel-config1.png)
![更新チャネルの設定](./images/channel-config2.png)
![更新チャネルの設定](./images/channel-config3.png)
<div style="text-align: center;">更新チャネルの設定</div>　　

なお、一部のコンピュートノードを更新対象外にする(カナリアロールアウト更新)ことができます。その場合は、ラベルをつけた特定のノードのプール(Machine Config Pool, MCP)を作成する必要があります。

```
$ : ↓AROクラスターにkubeadminユーザでログイン
$ oc login --token=sha256~XXXXX --server=https://api.testmydomain01.japaneast.aroapp.io:6443


$ : ↓AROクラスターのコンピュートノードのリスト表示
$ oc get -l 'node-role.kubernetes.io/master!=' -o 'jsonpath={range .items[*]}{.metadata.name}{"\n"}{end}' nodes
testmyaro01-jnlll-worker-japaneast1-t676g
testmyaro01-jnlll-worker-japaneast2-8nbm2
testmyaro01-jnlll-worker-japaneast3-6tqgr
$ : ↓「oc label」コマンドによる「node-role.kubernetes.io/workerpool-canary=」ラベルの付与
$ oc label node testmyaro01-jnlll-worker-japaneast3-6tqgr node-role.kubernetes.io/workerpool-canary=
node/testmyaro01-jnlll-worker-japaneast3-6tqgr labeled


$ : ↓「node-role.kubernetes.io/workerpool-canary=」ラベルが付与されたノードを対象とする
$ :   OpenShiftのMachineConfigPoolリソースを作成
$ cat << EOF > worker-canary.yaml
kind: MachineConfigPool
metadata:
  name: workerpool-canary
spec:
  machineConfigSelector:
    matchExpressions:
      - {
         key: machineconfiguration.openshift.io/role,
         operator: In,
         values: [worker,workerpool-canary]
        }
  nodeSelector:
    matchLabels:
      node-role.kubernetes.io/workerpool-canary: ""
EOF
$ oc create -f worker-canary.yaml
machineconfigpool.machineconfiguration.openshift.io/workerpool-canary created


$ : ↓OpenShiftのMachineConfigPoolリソース一覧を表示
$ oc get mcp
NAME                CONFIG                                                        UPDATED   UPDATING   DEGRADED   MACHINECOUNT   READYMACHINECOUNT   UPDATEDMACHINECOUNT   DEGRADEDMACHINECOUNT   AGE
master              rendered-master-b3a0f025835fde2aeb292b6344891769              True      False      False      3              3                   3                     0                      5h8m
worker              rendered-worker-e3b5c7f3534a74ba469358659178d170              True      False      False      2              2                   2                     0                      5h8m
workerpool-canary   rendered-workerpool-canary-e3b5c7f3534a74ba469358659178d170   True      False      False      1              1                   1                     0                      60s
```

ここで作成したMCP「workerpool-canary」を指定して、コンピュートノードのアップグレード対象外にすることが可能です。


### AROクラスターのアップグレード

「クラスター設定」画面から「Select a version」をクリックして、アップグレードするバージョンを選択します。このとき、「Full cluster update」を選択すると、全てのコントローラ/コンピュートノードがアップグレードされます。「Partial cluster update」を選択すると、一部または全てのコンピュートノードをアップグレードから除外できます。ここでは、前述したMCP「workerpool-canary」に含まれるコンピュートノードを選択しています。そして「更新」をクリックして、AROクラスターのアップグレードを開始します。


![AROクラスターのアップグレード](./images/channel-config3.png)
![AROクラスターのアップグレード](./images/aro-upgrade-select1.png)
![AROクラスターのアップグレード](./images/aro-upgrade-select2.png)
<div style="text-align: center;">AROクラスターのアップグレード</div>　　


アップグレードを開始すると、次のような画面が表示されます。ここでは、「Partial cluster update」を選択して、1台のコンピュートノードをアップグレード対象外としています。

![AROクラスターのアップグレード状況](./images/aro-upgrade-status1.png)
![AROクラスターのアップグレード状況](./images/aro-upgrade-status2.png)
![AROクラスターのアップグレード状況](./images/aro-upgrade-status3.png)
<div style="text-align: center;">AROクラスターのアップグレード状況 その1</div>　　


アップグレードしたコンピュートノードでアプリケーションが問題なく実行されることを確認したら、「Resume update」をクリックして、残り1台のコンピュートノードのアップグレードを完了できます。


![AROクラスターのアップグレード状況](./images/aro-upgrade-status4.png)
![AROクラスターのアップグレード状況](./images/aro-upgrade-status5.png)
<div style="text-align: center;">AROクラスターのアップグレード状況 その2</div>　


これで、AROクラスターアップグレードのデモ紹介は終了です。次は、インストラクターによる、[AROクラスター削除](../aro-delete)のデモ紹介です。


#### \[参考情報\]

- [第7章 カナリアロールアウト更新の実行](https://access.redhat.com/documentation/ja-jp/openshift_container_platform/4.10/html/updating_clusters/update-using-custom-machine-config-pools)


[HOME](../../README.md)
