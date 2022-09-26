# ingress-nginx

入门：https://kubernetes.github.io/ingress-nginx/deploy/

## helm

```sh
[root@master01 ~]# helm version
version.BuildInfo{Version:"v3.7.1", GitCommit:"1d11fcb5d3f3bf00dbe6fe31b8412839a96b3dc4", GitTreeState:"clean", GoVersion:"go1.16.9"}
```

```sh
Kubernetes                                                  Workstation
+---------------------------------------------------+     +------------------+
|                                                   |     |                  |
|  +-----------+   apiserver        +------------+  |     |  +------------+  |
|  |           |   proxy            |            |  |     |  |            |  |
|  | apiserver |                    |  ingress   |  |     |  |  ingress   |  |
|  |           |                    | controller |  |     |  | controller |  |
|  |           |                    |            |  |     |  |            |  |
|  |           |                    |            |  |     |  |            |  |
|  |           |  service account/  |            |  |     |  |            |  |
|  |           |  kubeconfig        |            |  |     |  |            |  |
|  |           +<-------------------+            |  |     |  |            |  |
|  |           |                    |            |  |     |  |            |  |
|  +------+----+      kubeconfig    +------+-----+  |     |  +------+-----+  |
|         |<--------------------------------------------------------|        |
|                                                   |     |                  |
+---------------------------------------------------+     +------------------+
```

日志格式

```sh
log_format upstreaminfo
    '$remote_addr - $remote_user [$time_local] "$request" '
    '$status $body_bytes_sent "$http_referer" "$http_user_agent" '
    '$request_length $request_time [$proxy_upstream_name] [$proxy_alternative_upstream_name] $upstream_addr '
    '$upstream_response_length $upstream_response_time $upstream_status $req_id';
```

## pod 与宿主机时间不一致

```sh
[root@master01 /opt/yaml]# kubectl exec -ti $POD_NAME -- date
Sun Nov 21 15:35:21 UTC 2021
[root@master01 /opt/yaml]# date
Sun Nov 21 23:35:29 CST 2021

```

```yaml
 。。。
  volumeMounts:
     - name: timezone
       mountPath: /etc/localtime
volumes:
  - name: timezone
    hostPath:
      path: /usr/share/zoneinfo/Asia/Shanghai
```

修改对应的pod磁盘挂载   calico   coredns

其实这个ingress-nginx已经安装成功了，我们通过访问映射的端口已经出现400的错误，说明已经成功，现在问题是服务没有成功映射出来。

## krew

kubectl 插件

https://krew.sigs.k8s.io/docs/user-guide/setup/install/

```sh
[root@master01 ~]# vim krew.sh 
#!/bin/bash
set -x; cd "$(mktemp -d)" && OS="$(uname | tr '[:upper:]' '[:lower:]')" && ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')" && KREW="krew-${OS}_${ARCH}" && curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/${KREW}.tar.gz" && tar zxvf "${KREW}.tar.gz" && ./"${KREW}" install krew     

[root@master01 ~]# . krew.sh 
[root@master01 /tmp/tmp.jkxBTGAHPd]# ls
+ ls --color=auto
krew-linux_amd64  krew-linux_amd64.tar.gz  LICENSE
++ printf '\033]0;%s@%s:%s\007' root master01 /tmp/tmp.jkxBTGAHPd
[root@master01 /tmp/tmp.jkxBTGAHPd]# ./krew-linux_amd64 install krew
+ ./krew-linux_amd64 install krew
Adding "default" plugin index from https://github.com/kubernetes-sigs/krew-index.git.
Updated the local copy of plugin index.
Installing plugin: krew
Installed plugin: krew
\
 | Use this plugin:
 | 	kubectl krew
 | Documentation:
 | 	https://krew.sigs.k8s.io/
 | Caveats:
 | \
 |  | krew is now installed! To start using kubectl plugins, you need to add
 |  | krew's installation directory to your PATH:
 |  | 
 |  |   * macOS/Linux:
 |  |     - Add the following to your ~/.bashrc or ~/.zshrc:
 |  |         export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"
 |  |     - Restart your shell.
 |  | 
 |  |   * Windows: Add %USERPROFILE%\.krew\bin to your PATH environment variable
 |  | 
 |  | To list krew commands and to get help, run:
 |  |   $ kubectl krew
 |  | For a full list of available plugins, run:
 |  |   $ kubectl krew search
 |  | 
 |  | You can find documentation at
 |  |   https://krew.sigs.k8s.io/docs/user-guide/quickstart/.
 | /
/
++ printf '\033]0;%s@%s:%s\007' root master01 /tmp/tmp.jkxBTGAHPd

[root@master01 ~]# cat .bashrc 
# .bashrc

# User specific aliases and functions

alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'

# Source global definitions
if [ -f /etc/bashrc ]; then
	. /etc/bashrc
fi
source /usr/share/bash-completion/bash_completion
source <(kubectl completion bash)
export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"

[root@master01 ~]# reboot
```

备份一下

```sh
[root@master01 ~]# tar -czvf krew.tar.gz .krew/   #备份
[root@master01 ~]# tar -tzvf krew.tar.gz   #查看
```

```sh
[root@master01 ~]# kubectl krew version
OPTION            VALUE
GitTag            v0.4.2
GitCommit         6fcdb79
IndexURI          https://github.com/kubernetes-sigs/krew-index.git
BasePath          /root/.krew
IndexPath         /root/.krew/index/default
InstallPath       /root/.krew/store
BinPath           /root/.krew/bin
DetectedPlatform  linux/amd64
```

```sh
# This is a basic workflow to help you get started with Actions

name: image-sync
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
  schedule:
    - cron: '0 23 * * *'

env:
  IMAGE_SYNCER_VERSION: v1.3.0
  DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
  DOCKERHUB_PASSWORD: ${{ secrets.DOCKERHUB_PASSWORD }}

jobs:
  image-sync:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout
      uses: actions/checkout@v2

    - name: install image-sync tool
      run: |
        wget https://github.com/AliyunContainerService/image-syncer/releases/download/${IMAGE_SYNCER_VERSION}/image-syncer-${IMAGE_SYNCER_VERSION}-linux-amd64.tar.gz
        tar -zxf image-syncer-${IMAGE_SYNCER_VERSION}-linux-amd64.tar.gz
        
    - name: init config.yaml with env
      run: |
        sed -i "s#USERNAME#${DOCKERHUB_USERNAME}#g" config.yaml
        sed -i "s#PASSWORD#${DOCKERHUB_PASSWORD}#g" config.yaml
        
    - name: sync images
      run: |
        ./image-syncer --proc=20 --config=config.yaml --retries=1
```

