---
title: Install kind
---

[kind](https://kind.sigs.k8s.io/) は、Docker コンテナを使用してローカルに Kubernetes クラスターを実行するためのツールです。インストール手順は [Quick Start](https://kind.sigs.k8s.io/docs/user/quick-start/) に書いてある通りなのですが、手元で Kubernetes のテスト環境を作成するために行なった内容をまとめておきます。

## インストール環境

- ホスト: Ubuntu 22.04.4 LTS (Jammy)
- コンテナ ランタイム: podman (ルートレス)

## podman のインストール

podman はパッケージ マネージャーを使用してインストールすることができます。

```sh
$ sudo apt install -y podman
```

バージョン 3.4.4 の podman がインストールされました。

```sh
$ podman -v
podman version 3.4.4
```

## cgroup

kind を実行するホストは cgroup v2 が必要があります。念の為確認しておきます:

```sh
$ stat -fc %T /sys/fs/cgroup/
cgroup2fs
```

`cgroup2fs` なので OK、cgroup v1 ならば `tmpfs` と出力されます。

kind をルートレス、つまり管理者権限でなく実行するには、一般ユーザーで cgroup コントローラーを使えるようにする必要があります。

例えば普通に Hello コンテナを起動することはできますが、

```sh
$ podman run quay.io/podman/hello
!... Hello Podman World ...!

         .--"--.
       / -     - \
      / (O)   (O) \
   ~~~| -=(,Y,)=- |
    .---. /`  \   |~~
 ~/  o  o \~~~~.----. ~~
  | =(X)= |~  / (O (O) \
   ~~~~~~~  ~| =(Y_)=-  |
  ~~~~    ~~~|   U      |~~

Project:   https://github.com/containers/podman
Website:   https://podman.io
Desktop:   https://podman-desktop.io
Documents: https://docs.podman.io
YouTube:   https://youtube.com/@Podman
X/Twitter: @Podman_io
Mastodon:  @Podman_io@fosstodon.org
```

`--cpus 0.5` をつけて実行しようとすると cgroup コントローラーが利用できないというエラーが出力されます。これがエラーとならないようにする必要があります。

```sh
$ podman run --cpus 0.5 quay.io/podman/hello
Error: OCI runtime error: the requested cgroup controller `cpu` is not available
```

そのためには、`/etc/systemd/system/user@.service.d/delegate.conf` というファイルを作成して、デリゲートを有効にします。

```sh
$ sudo mkdir /etc/systemd/system/user@.service.d
$ cat << EOF | sudo tee /etc/systemd/system/user@.service.d/delegate.conf > /dev/null
[Service]
Delegate=yes
EOF
$ sudo systemctl daemon-reload
```

cpu cgroup コントローラーが使えるようになりました。

```sh
$ podman run --cpus 0.5  quay.io/podman/hello
!... Hello Podman World ...!

         .--"--.
       / -     - \
      / (O)   (O) \
   ~~~| -=(,Y,)=- |
    .---. /`  \   |~~
 ~/  o  o \~~~~.----. ~~
  | =(X)= |~  / (O (O) \
   ~~~~~~~  ~| =(Y_)=-  |
  ~~~~    ~~~|   U      |~~

Project:   https://github.com/containers/podman
Website:   https://podman.io
Desktop:   https://podman-desktop.io
Documents: https://docs.podman.io
YouTube:   https://youtube.com/@Podman
X/Twitter: @Podman_io
Mastodon:  @Podman_io@fosstodon.org
```

## iptables

iptables モジュールが必要なので、以下の設定ファイルを作成して、カーネル モジュールをリロードします。

```sh
$ cat << EOF | sudo tee /etc/modules-load.d/iptables.conf > /dev/null
ip6_tables
ip6table_nat
ip_tables
iptable_nat
EOF
$ sudo systemctl restart systemd-modules-load.service
```

## kind のバイナリのインストール

2024年7月時点では v0.23.0 が最新なので、ビルド済みのバイナリをダウンロードして `/usr/local/bin` の下に展開します。

```sh
$ curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.23.0/kind-linux-amd64
$ sudo install -o root -g root -m 0755 kind /usr/local/bin/kind
```

## kubectl のバイナリのインストール

kubectl のビルド済みバイナリをダウンロードして `/usr/local/bin` の下に展開します。

```sh
$ curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
$ sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
$ kubectl version --client
Client Version: v1.30.2
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
```

## CNI plugins に関するワークアラウンド

通常は以上の手順でクラスターの作成ができるはずなのですが、次のようなエラー メッセージが出て `kind create cluster` の実行に失敗しました。

```
[plugin bridge does not support config version "1.0.0" plugin portmap does not support config version "1.0.0" plugin firewall does not support config version "1.0.0" plugin tuning does not support config version "1.0.0"]
```

これは[こちら](https://bugs.launchpad.net/ubuntu/+source/libpod/+bug/2024394)で報告されているバグのようで、次のワークアラウンドを実行しました。

```sh
$ curl -O http://archive.ubuntu.com/ubuntu/pool/universe/g/golang-github-containernetworking-plugins/containernetworking-plugins_1.1.1+ds1-3build1_amd64.deb
$ sudo dpkg -i containernetworking-plugins_1.1.1+ds1-3build1_amd64.deb
```

## クラスターの作成

`kind create cluster` コマンドを実行するとクラスターを作成することができます。

```sh
$ kind create cluster
enabling experimental podman provider
Cgroup controller detection is not implemented for Podman. If you see cgroup-related errors, you might need to set systemd property "Delegate=yes", see https://kind.sigs.k8s.io/docs/user/rootless/
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.30.0) 🖼
 ✓ Preparing nodes 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Have a question, bug, or feature request? Let us know! https://kind.sigs.k8s.io/#community 🙂
```

kind-kind という名前のコンテキストが `~/.kube/config` に登録されます。

```sh
$ kubectl config get-contexts
CURRENT   NAME        CLUSTER     AUTHINFO    NAMESPACE
*         kind-kind   kind-kind   kind-kind
```

既定では 1 つのみノードを持つクラスターが作成されます。

```sh
$ kubectl config use-context kind-kind
Switched to context "kind-kind".
$ kubectl get node
NAME                 STATUS   ROLES           AGE     VERSION
kind-control-plane   Ready    control-plane   3m34s   v1.30.0
```

## クラスターの削除

作成したクラスターは `kind delete cluster` コマンドで削除できます。

```sh
$ kind delete cluster
enabling experimental podman provider
Deleting cluster "kind" ...
Deleted nodes: ["kind-control-plane"]
```
