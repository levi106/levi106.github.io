# OpenShift Local のノードに接続したい

`oc debug nodes/<ノード名>` で接続できます。ノードのルートファイルシステムは `/host` にマウントされているので、`chroot /host` でホストのファイルシステムをルートに変更することができます。

```sh
$ oc get node
NAME   STATUS   ROLES                         AGE   VERSION
crc    Ready    control-plane,master,worker   20d   v1.30.7
$ oc debug nodes/crc
Temporary namespace openshift-debug-5zm2f is created for debugging node...
Starting pod/crc-debug-w6vx6 ...
To use host binaries, run `chroot /host`

Pod IP: 192.168.126.11
If you don't see a command prompt, try pressing enter.
sh-5.1# ls /        
afs  boot  etc	 host  lib64	   media  opt	root  sbin  sys  usr
bin  dev   home  lib   lost+found  mnt	  proc	run   srv   tmp  var
sh-5.1# chroot /host
sh-5.1# bash
[root@crc /]# cat /etc/redhat-release 
Red Hat Enterprise Linux CoreOS release 4.17
```

接続すると、`openshift-debug-xxx` という名前の Namespace が作成され、その中に `crc-debug-xxx` という名前の Pod が作成されていました。

```sh
$ oc get ns | grep debug
openshift-debug-2bxfd                              Active   38s
$ oc get po -n openshift-debug-2bxfd
NAME              READY   STATUS    RESTARTS   AGE
crc-debug-8bmqf   1/1     Running   0          47s
```

```sh
$ oc describe po -n openshift-debug-2bxfd crc-debug-8bmqf
Name:                 crc-debug-8bmqf
Namespace:            openshift-debug-2bxfd
Priority:             1000000000
Priority Class Name:  openshift-user-critical
Service Account:      default
Node:                 crc/192.168.126.11
Start Time:           Sun, 02 Mar 2025 13:16:21 +0900
Labels:               <none>
Annotations:          debug.openshift.io/source-container: container-00
                      debug.openshift.io/source-resource: /v1, Resource=nodes/crc
                      openshift.io/required-scc: privileged
                      openshift.io/scc: privileged
Status:               Running
IP:                   192.168.126.11
IPs:
  IP:  192.168.126.11
Containers:
  container-00:
    Container ID:  cri-o://d8e8a0132ad6a1109439089bfc5db76bc00f2fa50b185364247a80921db762f8
    Image:         quay.io/openshift-release-dev/ocp-v4.0-art-dev@sha256:faa1c4113f6c16e5c4553530d3a467d9588c9b2fc0e2769c3803dc5853e02167
    Image ID:      quay.io/openshift-release-dev/ocp-v4.0-art-dev@sha256:faa1c4113f6c16e5c4553530d3a467d9588c9b2fc0e2769c3803dc5853e02167
    Port:          <none>
    Host Port:     <none>
    Command:
      /bin/sh
    State:          Running
      Started:      Sun, 02 Mar 2025 13:16:22 +0900
    Ready:          True
    Restart Count:  0
    Environment:
      TMOUT:  900
    Mounts:
      /host from host (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-d2cp9 (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
Volumes:
  host:
    Type:          HostPath (bare host directory volume)
    Path:          /
    HostPathType:  Directory
  kube-api-access-d2cp9:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
    ConfigMapName:           openshift-service-ca.crt
    ConfigMapOptional:       <nil>
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason   Age    From     Message
  ----    ------   ----   ----     -------
  Normal  Pulled   3m18s  kubelet  Container image "quay.io/openshift-release-dev/ocp-v4.0-art-dev@sha256:faa1c4113f6c16e5c4553530d3a467d9588c9b2fc0e2769c3803dc5853e02167" already present on machine
  Normal  Created  3m17s  kubelet  Created container container-00
  Normal  Started  3m17s  kubelet  Started container container-00
```

`openshift-debug-xxx` という Namespace を作成する oc コマンド内のコードは[ここ](https://github.com/openshift/oc/blob/514a4581b8a6b8b0aef5832e50adf1037ffd9566/pkg/cli/debug/debug.go#L1245)にありました。
現在の Namespace に `pod-security.kubernetes.io/enforce` という Label が無い、あっても値が `privileged` ではない時にこの一時的な Namespace が作成される実装になっています。

- [Troubleshooting CRC - Getting shell access to the OpenShift cluster](https://crc.dev/docs/troubleshooting/#getting-shell-access-to-the-openshift-cluster)
- [How the oc debug command works in OpenShift](https://www.redhat.com/en/blog/how-oc-debug-works)
