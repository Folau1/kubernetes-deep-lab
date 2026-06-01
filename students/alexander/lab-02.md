\## Lab 02 — Container Runtime: containerd



\### Дата



2026-06-01



\### Цель



Установить и настроить `containerd` как container runtime для будущего Kubernetes-кластера.



Настроить согласованный cgroup driver:



```text

SystemdCgroup = true

```



Установить диагностический инструмент `crictl`, настроить подключение к CRI socket `containerd` и проверить готовность runtime.



\### Что было сделано



\- Проверена доступность пакетов `containerd` и `runc`.

\- Установлены `containerd` и `runc` на всех nodes.

\- Проверены версии установленных компонентов.

\- Проверено состояние systemd-сервиса `containerd`.

\- Создан файл `/etc/containerd/config.toml`.

\- Включен параметр `SystemdCgroup = true`.

\- Выполнен restart сервиса `containerd`.

\- Проверена загрузка `overlayfs` snapshotter.

\- Проверена загрузка runtime и CRI plugins через `ctr`.

\- Установлен `crictl v1.34.0`.

\- Создан файл `/etc/crictl.yaml`.

\- Выполнена проверка CRI runtime через `crictl info`.

\- Подтверждено состояние `RuntimeReady = true`.

\- Подтверждено ожидаемое состояние `NetworkReady = false` до установки CNI plugin.

\- Проверено отсутствие containers, images и pods до установки `kubelet`.



\---



\### Inventory



| Node | Role | Internal IP | OS |

|---|---|---|---|

| control-plane-1 | control-plane | 192.168.1.15/24 | Ubuntu Server 24.04.4 LTS |

| worker-1 | worker | 192.168.1.16/24 | Ubuntu Server 24.04.4 LTS |

| worker-2 | worker | 192.168.1.17/24 | Ubuntu Server 24.04.4 LTS |



\---



\### Проверка доступных пакетов



Перед установкой были проверены доступные версии пакетов:



```text

containerd:

&#x20; Installed: (none)

&#x20; Candidate: 2.2.1-0ubuntu1\~24.04.2

&#x20; Version table:

&#x20;    2.2.1-0ubuntu1\~24.04.2 500

&#x20;       500 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 Packages

&#x20;    1.7.28-0ubuntu1\~24.04.2 500

&#x20;       500 http://security.ubuntu.com/ubuntu noble-security/main amd64 Packages

&#x20;    1.7.12-0ubuntu4 500

&#x20;       500 http://ru.archive.ubuntu.com/ubuntu noble/main amd64 Packages



runc:

&#x20; Installed: (none)

&#x20; Candidate: 1.3.4-0ubuntu1\~24.04.1

&#x20; Version table:

&#x20;    1.3.4-0ubuntu1\~24.04.1 500

&#x20;       500 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 Packages

&#x20;    1.3.3-0ubuntu1\~24.04.3 500

&#x20;       500 http://security.ubuntu.com/ubuntu noble-security/main amd64 Packages

&#x20;    1.1.12-0ubuntu3 500

&#x20;       500 http://ru.archive.ubuntu.com/ubuntu noble/main amd64 Packages

```



По результатам проверки использован основной сценарий для `containerd 2.x`:



```text

containerd 2.2.1

config version = 3

```



\---



\### Установка containerd и runc



На каждой node выполнены команды:



```bash

apt update

apt install -y containerd runc

```



\#### worker-1



```text

root@worker-1:\~# containerd --version

containerd github.com/containerd/containerd/v2 2.2.1



root@worker-1:\~# runc --version

runc version 1.3.4-0ubuntu1\~24.04.1

spec: 1.2.1

go: go1.24.4

libseccomp: 2.5.5



root@worker-1:\~# systemctl is-enabled containerd

enabled



root@worker-1:\~# systemctl is-active containerd

active

```



\#### control-plane-1



```text

root@control-plane-1:\~# containerd --version

containerd github.com/containerd/containerd/v2 2.2.1



root@control-plane-1:\~# runc --version

runc version 1.3.4-0ubuntu1\~24.04.1

spec: 1.2.1

go: go1.24.4

libseccomp: 2.5.5



root@control-plane-1:\~# systemctl is-enabled containerd

enabled



root@control-plane-1:\~# systemctl is-active containerd

active

```



\#### worker-2



```text

root@worker-2:\~# containerd --version

containerd github.com/containerd/containerd/v2 2.2.1



root@worker-2:\~# runc --version

runc version 1.3.4-0ubuntu1\~24.04.1

spec: 1.2.1

go: go1.24.4

libseccomp: 2.5.5



root@worker-2:\~# systemctl is-enabled containerd

enabled



root@worker-2:\~# systemctl is-active containerd

active

```



\---



\### Проверка default config containerd



После установки на `worker-1` проверено наличие файла `/etc/containerd/config.toml`.



```text

root@worker-1:\~# echo "===== EXISTING CONFIG ====="

root@worker-1:\~# test -f /etc/containerd/config.toml \\

>   \&\& sed -n '1,220p' /etc/containerd/config.toml \\

>   || echo "no /etc/containerd/config.toml"



===== EXISTING CONFIG =====

no /etc/containerd/config.toml

```



До создания файла `containerd` работал с default config.



Проверка ключевых строк default config:



```text

root@worker-1:\~# containerd config default \\

>   | sed -n '1,220p' \\

>   | grep -nE 'version|SystemdCgroup|io.containerd.cri|runc|sandbox\_image'



1:version = 3

39:  \[plugins.'io.containerd.cri.v1.images']

50:    \[plugins.'io.containerd.cri.v1.images'.pinned\_images]

53:    \[plugins.'io.containerd.cri.v1.images'.registry]

56:    \[plugins.'io.containerd.cri.v1.images'.image\_decryption]

59:  \[plugins.'io.containerd.cri.v1.runtime']

79:    \[plugins.'io.containerd.cri.v1.runtime'.containerd]

80:      default\_runtime\_name = 'runc'

84:      \[plugins.'io.containerd.cri.v1.runtime'.containerd.runtimes]

85:        \[plugins.'io.containerd.cri.v1.runtime'.containerd.runtimes.runc]

86:          runtime\_type = 'io.containerd.runc.v2'

100:          \[plugins.'io.containerd.cri.v1.runtime'.containerd.runtimes.runc.options]

109:            SystemdCgroup = false

111:    \[plugins.'io.containerd.cri.v1.runtime'.cni]

```



До изменения конфигурации использовалось значение:



```text

SystemdCgroup = false

```



\---



\### Настройка SystemdCgroup



На каждой node выполнены команды:



```bash

mkdir -p /etc/containerd



if test -f /etc/containerd/config.toml; then

&#x20; cp /etc/containerd/config.toml \\

&#x20;   /etc/containerd/config.toml.bak.$(date +%Y%m%d-%H%M%S)

fi



containerd config default > /etc/containerd/config.toml



sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' \\

&#x20; /etc/containerd/config.toml



systemctl restart containerd

```



\#### worker-1



```text

root@worker-1:\~# grep -nE 'version|SystemdCgroup|default\_runtime\_name|runtime\_type' \\

>   /etc/containerd/config.toml



1:version = 3

80:      default\_runtime\_name = 'runc'

86:          runtime\_type = 'io.containerd.runc.v2'

109:            SystemdCgroup = true



root@worker-1:\~# systemctl is-enabled containerd

enabled



root@worker-1:\~# systemctl is-active containerd

active

```



\#### control-plane-1



```text

root@control-plane-1:\~# grep -nE 'version|SystemdCgroup|default\_runtime\_name|runtime\_type' \\

>   /etc/containerd/config.toml



1:version = 3

80:      default\_runtime\_name = 'runc'

86:          runtime\_type = 'io.containerd.runc.v2'

109:            SystemdCgroup = true



root@control-plane-1:\~# systemctl is-enabled containerd

enabled



root@control-plane-1:\~# systemctl is-active containerd

active

```



\#### worker-2



```text

root@worker-2:\~# grep -nE 'version|SystemdCgroup|default\_runtime\_name|runtime\_type' \\

>   /etc/containerd/config.toml



1:version = 3

80:      default\_runtime\_name = 'runc'

86:          runtime\_type = 'io.containerd.runc.v2'

109:            SystemdCgroup = true



root@worker-2:\~# systemctl is-enabled containerd

enabled



root@worker-2:\~# systemctl is-active containerd

active

```



\---



\### Проверка журнала containerd



После restart сервиса на `worker-1` выполнена проверка:



```bash

journalctl -u containerd --no-pager -n 20

```



Вывод:



```text

Jun 01 15:35:58 worker-1 containerd\[4071]: time="2026-06-01T15:35:58.000240475Z" level=info msg="skip loading plugin" error="skip plugin: tracing endpoint not configured" id=io.containerd.internal.v1.tracing type=io.containerd.internal.v1

Jun 01 15:35:58 worker-1 containerd\[4071]: time="2026-06-01T15:35:58.000248235Z" level=info msg="loading plugin" id=io.containerd.ttrpc.v1.otelttrpc type=io.containerd.ttrpc.v1

Jun 01 15:35:58 worker-1 containerd\[4071]: time="2026-06-01T15:35:58.000258789Z" level=info msg="loading plugin" id=io.containerd.grpc.v1.healthcheck type=io.containerd.grpc.v1

Jun 01 15:35:58 worker-1 containerd\[4071]: time="2026-06-01T15:35:58.000269729Z" level=info msg="loading plugin" id=io.containerd.grpc.v1.cri type=io.containerd.grpc.v1

Jun 01 15:35:58 worker-1 containerd\[4071]: time="2026-06-01T15:35:58.000345764Z" level=info msg="Connect containerd service"

Jun 01 15:35:58 worker-1 containerd\[4071]: time="2026-06-01T15:35:58.000374673Z" level=info msg="using experimental NRI integration - disable nri plugin to prevent this"

Jun 01 15:35:58 worker-1 containerd\[4071]: time="2026-06-01T15:35:58.000851872Z" level=error msg="failed to load cni during init, please check CRI plugin status before setting up network for pods" error="cni config load failed: no network config found in /etc/cni/net.d: cni plugin not initialized: failed to load cni config"

Jun 01 15:35:58 worker-1 containerd\[4071]: time="2026-06-01T15:35:58.027157390Z" level=info msg="Start subscribing containerd event"

Jun 01 15:35:58 worker-1 containerd\[4071]: time="2026-06-01T15:35:58.027263877Z" level=info msg="Start recovering state"

Jun 01 15:35:58 worker-1 containerd\[4071]: time="2026-06-01T15:35:58.027436297Z" level=info msg="Start event monitor"

Jun 01 15:35:58 worker-1 containerd\[4071]: time="2026-06-01T15:35:58.027448897Z" level=info msg="Start cni network conf syncer for default"

Jun 01 15:35:58 worker-1 containerd\[4071]: time="2026-06-01T15:35:58.027456123Z" level=info msg="Start streaming server"

Jun 01 15:35:58 worker-1 containerd\[4071]: time="2026-06-01T15:35:58.027587195Z" level=info msg="Registered namespace \\"k8s.io\\" with NRI"

Jun 01 15:35:58 worker-1 containerd\[4071]: time="2026-06-01T15:35:58.027603044Z" level=info msg="runtime interface starting up..."

Jun 01 15:35:58 worker-1 containerd\[4071]: time="2026-06-01T15:35:58.027608654Z" level=info msg="starting plugins..."

Jun 01 15:35:58 worker-1 containerd\[4071]: time="2026-06-01T15:35:58.027757656Z" level=info msg="Synchronizing NRI (plugin) with current runtime state"

Jun 01 15:35:58 worker-1 containerd\[4071]: time="2026-06-01T15:35:58.029271430Z" level=info msg=serving... address=/run/containerd/containerd.sock.ttrpc

Jun 01 15:35:58 worker-1 containerd\[4071]: time="2026-06-01T15:35:58.029430116Z" level=info msg=serving... address=/run/containerd/containerd.sock

Jun 01 15:35:58 worker-1 containerd\[4071]: time="2026-06-01T15:35:58.029917482Z" level=info msg="containerd successfully booted in 0.055484s"

Jun 01 15:35:58 worker-1 systemd\[1]: Started containerd.service - containerd container runtime.

```



В журнале присутствует предупреждение:



```text

failed to load cni during init

cni config load failed: no network config found in /etc/cni/net.d

```



Это ожидаемое состояние на текущем этапе.



Причина:



```text

CNI plugin еще не установлен.

Pod network еще не настроен.

```



Предупреждение не является блокером для текущей лабораторной.



\---



\### Проверка containerd через ctr



Команды проверки:



```bash

ctr namespaces list

ctr plugins list | grep -E 'cri|runtime|snapshotter'

```



\#### worker-1



```text

root@worker-1:\~# ctr namespaces list

NAME LABELS



root@worker-1:\~# ctr plugins list | grep -E 'cri|runtime|snapshotter'

io.containerd.snapshotter.v1              blockfile                linux/amd64    skip

io.containerd.snapshotter.v1              btrfs                    linux/amd64    skip

io.containerd.snapshotter.v1              devmapper                linux/amd64    skip

io.containerd.snapshotter.v1              erofs                    linux/amd64    skip

io.containerd.snapshotter.v1              native                   linux/amd64    ok

io.containerd.snapshotter.v1              overlayfs                linux/amd64    ok

io.containerd.snapshotter.v1              zfs                      linux/amd64    skip

io.containerd.runtime.v2                  task                     linux/amd64    ok

io.containerd.cri.v1                      images                   -              ok

io.containerd.cri.v1                      runtime                  linux/amd64    ok

io.containerd.grpc.v1                     cri                      -              ok

```



\#### control-plane-1



```text

root@control-plane-1:\~# ctr plugins list | grep -E 'overlayfs|runtime.v2|io.containerd.cri.v1|io.containerd.grpc.v1.\*cri'

io.containerd.snapshotter.v1              overlayfs                linux/amd64    ok

io.containerd.runtime.v2                  task                     linux/amd64    ok

io.containerd.cri.v1                      images                   -              ok

io.containerd.cri.v1                      runtime                  linux/amd64    ok

io.containerd.grpc.v1                     cri                      -              ok

```



\#### worker-2



```text

root@worker-2:\~# ctr plugins list | grep -E 'overlayfs|runtime.v2|io.containerd.cri.v1|io.containerd.grpc.v1.\*cri'

io.containerd.snapshotter.v1              overlayfs                linux/amd64    ok

io.containerd.runtime.v2                  task                     linux/amd64    ok

io.containerd.cri.v1                      images                   -              ok

io.containerd.cri.v1                      runtime                  linux/amd64    ok

io.containerd.grpc.v1                     cri                      -              ok

```



Пустой список namespaces на чистой node является ожидаемым состоянием: контейнеры пока не запускались.



\---



\### Установка и настройка crictl



Перед установкой на `worker-1` проверено отсутствие `crictl`:



```text

root@worker-1:\~# command -v crictl || true

```



Вывод отсутствовал.



Использована версия:



```text

crictl v1.34.0

```



Проверка доступности release archive:



```bash

CRICTL\_VERSION="v1.34.0"



curl -I --connect-timeout 10 -L \\

&#x20; "https://github.com/kubernetes-sigs/cri-tools/releases/download/${CRICTL\_VERSION}/crictl-${CRICTL\_VERSION}-linux-amd64.tar.gz"

```



Результат:



```text

HTTP/2 302

location: https://release-assets.githubusercontent.com/...



HTTP/2 200

content-disposition: attachment; filename=crictl-v1.34.0-linux-amd64.tar.gz

content-type: application/octet-stream

content-length: 19610258

```



Архив доступен для скачивания.



На каждой node выполнены команды:



```bash

CRICTL\_VERSION="v1.34.0"

TMP\_DIR="$(mktemp -d)"



curl -L \\

&#x20; "https://github.com/kubernetes-sigs/cri-tools/releases/download/${CRICTL\_VERSION}/crictl-${CRICTL\_VERSION}-linux-amd64.tar.gz" \\

&#x20; -o "${TMP\_DIR}/crictl-${CRICTL\_VERSION}-linux-amd64.tar.gz"



tar -tzf "${TMP\_DIR}/crictl-${CRICTL\_VERSION}-linux-amd64.tar.gz"



tar -C /usr/local/bin \\

&#x20; -xzf "${TMP\_DIR}/crictl-${CRICTL\_VERSION}-linux-amd64.tar.gz"



chmod 0755 /usr/local/bin/crictl

chown root:root /usr/local/bin/crictl

```



\#### worker-1



```text

root@worker-1:\~# ls -lh "${TMP\_DIR}"

total 19M

\-rw-r--r-- 1 root root 19M Jun  1 15:36 crictl-v1.34.0-linux-amd64.tar.gz



root@worker-1:\~# tar -tzf "${TMP\_DIR}/crictl-v1.34.0-linux-amd64.tar.gz"

crictl



root@worker-1:\~# command -v crictl

/usr/local/bin/crictl



root@worker-1:\~# crictl --version

crictl version v1.34.0



root@worker-1:\~# ls -l /usr/local/bin/crictl

\-rwxr-xr-x 1 root root 40548006 Aug 21  2025 /usr/local/bin/crictl

```



\#### control-plane-1



```text

root@control-plane-1:\~# crictl --version

crictl version v1.34.0

```



\#### worker-2



```text

root@worker-2:\~# crictl --version

crictl version v1.34.0

```



\---



\### Настройка /etc/crictl.yaml



На каждой node создан файл:



```bash

cat <<'EOF' > /etc/crictl.yaml

runtime-endpoint: unix:///run/containerd/containerd.sock

image-endpoint: unix:///run/containerd/containerd.sock

timeout: 10

debug: false

EOF

```



\#### control-plane-1



```text

root@control-plane-1:\~# cat /etc/crictl.yaml

runtime-endpoint: unix:///run/containerd/containerd.sock

image-endpoint: unix:///run/containerd/containerd.sock

timeout: 10

debug: false

```



\#### worker-1



```text

root@worker-1:\~# cat /etc/crictl.yaml

runtime-endpoint: unix:///run/containerd/containerd.sock

image-endpoint: unix:///run/containerd/containerd.sock

timeout: 10

debug: false

```



\#### worker-2



```text

root@worker-2:\~# cat /etc/crictl.yaml

runtime-endpoint: unix:///run/containerd/containerd.sock

image-endpoint: unix:///run/containerd/containerd.sock

timeout: 10

debug: false

```



\---



\### Проверка CRI runtime через crictl



На `worker-1` выполнена полная проверка:



```bash

crictl info

```



Ключевые фрагменты вывода:



```text

{

&#x20; "config": {

&#x20;   "containerd": {

&#x20;     "defaultRuntimeName": "runc",

&#x20;     "runtimes": {

&#x20;       "runc": {

&#x20;         "options": {

&#x20;           "SystemdCgroup": true

&#x20;         },

&#x20;         "runtimeType": "io.containerd.runc.v2"

&#x20;       }

&#x20;     }

&#x20;   }

&#x20; },

&#x20; "lastCNILoadStatus": "cni config load failed: no network config found in /etc/cni/net.d: cni plugin not initialized: failed to load cni config",

&#x20; "status": {

&#x20;   "conditions": \[

&#x20;     {

&#x20;       "message": "",

&#x20;       "reason": "",

&#x20;       "status": true,

&#x20;       "type": "RuntimeReady"

&#x20;     },

&#x20;     {

&#x20;       "message": "Network plugin returns error: cni plugin not initialized",

&#x20;       "reason": "NetworkPluginNotReady",

&#x20;       "status": false,

&#x20;       "type": "NetworkReady"

&#x20;     },

&#x20;     {

&#x20;       "message": "",

&#x20;       "reason": "",

&#x20;       "status": true,

&#x20;       "type": "ContainerdHasNoDeprecationWarnings"

&#x20;     }

&#x20;   ]

&#x20; }

}

```



На всех nodes выполнена краткая проверка ключевых параметров.



\#### control-plane-1



```text

root@control-plane-1:\~# crictl info | grep -E '"type": "RuntimeReady"|"type": "NetworkReady"|"status": true|"status": false|SystemdCgroup|runtimeType'

&#x20;           "SystemdCgroup": true

&#x20;         "runtimeType": "io.containerd.runc.v2",

&#x20;       "status": true,

&#x20;       "type": "RuntimeReady"

&#x20;       "status": false,

&#x20;       "type": "NetworkReady"

&#x20;       "status": true,

```



\#### worker-1



```text

root@worker-1:\~# crictl info

&#x20;           "SystemdCgroup": true

&#x20;         "runtimeType": "io.containerd.runc.v2",

&#x20;       "status": true,

&#x20;       "type": "RuntimeReady"

&#x20;       "status": false,

&#x20;       "type": "NetworkReady"

&#x20;       "status": true,

```



\#### worker-2



```text

root@worker-2:\~# crictl info | grep -E '"type": "RuntimeReady"|"type": "NetworkReady"|"status": true|"status": false|SystemdCgroup|runtimeType'

&#x20;           "SystemdCgroup": true

&#x20;         "runtimeType": "io.containerd.runc.v2",

&#x20;       "status": true,

&#x20;       "type": "RuntimeReady"

&#x20;       "status": false,

&#x20;       "type": "NetworkReady"

&#x20;       "status": true,

```



На всех nodes подтверждено:



```text

runtimeType: io.containerd.runc.v2

SystemdCgroup: true

RuntimeReady: true

NetworkReady: false

```



`NetworkReady: false` является ожидаемым состоянием.



Причина:



```text

CNI plugin еще не установлен.

Pod network еще не настроен.

```



Основной успешный критерий текущей части лабораторной:



```text

RuntimeReady: true

```



\---



\### Проверка пустого runtime



До установки `kubelet` и Kubernetes components выполнены команды:



```bash

crictl ps

crictl ps -a

crictl images

crictl pods

```



\#### worker-1



```text

root@worker-1:\~# crictl ps

CONTAINER           IMAGE               CREATED             STATE               NAME                ATTEMPT             POD ID              POD                 NAMESPACE



root@worker-1:\~# crictl ps -a

CONTAINER           IMAGE               CREATED             STATE               NAME                ATTEMPT             POD ID              POD                 NAMESPACE



root@worker-1:\~# crictl images

IMAGE               TAG                 IMAGE ID            SIZE



root@worker-1:\~# crictl pods

POD ID              CREATED             STATE               NAME                NAMESPACE           ATTEMPT             RUNTIME

```



\#### control-plane-1



```text

root@control-plane-1:\~# crictl ps

CONTAINER           IMAGE               CREATED             STATE               NAME                ATTEMPT             POD ID              POD                 NAMESPACE



root@control-plane-1:\~# crictl ps -a

CONTAINER           IMAGE               CREATED             STATE               NAME                ATTEMPT             POD ID              POD                 NAMESPACE



root@control-plane-1:\~# crictl images

IMAGE               TAG                 IMAGE ID            SIZE



root@control-plane-1:\~# crictl pods

POD ID              CREATED             STATE               NAME                NAMESPACE           ATTEMPT             RUNTIME

```



\#### worker-2



```text

root@worker-2:\~# crictl ps

CONTAINER           IMAGE               CREATED             STATE               NAME                ATTEMPT             POD ID              POD                 NAMESPACE



root@worker-2:\~# crictl ps -a

CONTAINER           IMAGE               CREATED             STATE               NAME                ATTEMPT             POD ID              POD                 NAMESPACE



root@worker-2:\~# crictl images

IMAGE               TAG                 IMAGE ID            SIZE



root@worker-2:\~# crictl pods

POD ID              CREATED             STATE               NAME                NAMESPACE           ATTEMPT             RUNTIME

```



Пустые списки являются ожидаемым состоянием: `kubelet` еще не создавал containers и pods через CRI.



\---



\### Ключевые выводы команд



На всех nodes получен аналогичный результат:



```text

containerd github.com/containerd/containerd/v2 2.2.1



runc version 1.3.4-0ubuntu1\~24.04.1

spec: 1.2.1

go: go1.24.4

libseccomp: 2.5.5



crictl version v1.34.0



systemctl is-enabled containerd

enabled



systemctl is-active containerd

active

```



Конфигурация:



```text

1:version = 3

80:      default\_runtime\_name = 'runc'

86:          runtime\_type = 'io.containerd.runc.v2'

109:            SystemdCgroup = true

```



Plugins:



```text

io.containerd.snapshotter.v1              overlayfs                linux/amd64    ok

io.containerd.runtime.v2                  task                     linux/amd64    ok

io.containerd.cri.v1                      images                   -              ok

io.containerd.cri.v1                      runtime                  linux/amd64    ok

io.containerd.grpc.v1                     cri                      -              ok

```



CRI status:



```text

runtimeType: io.containerd.runc.v2

SystemdCgroup: true

RuntimeReady: true

NetworkReady: false

```



\---



\### Итоговый checkpoint



```text

control-plane-1   containerd 2.2.1   runc 1.3.4   crictl v1.34.0   SystemdCgroup=true   active/enabled   CRI OK

worker-1          containerd 2.2.1   runc 1.3.4   crictl v1.34.0   SystemdCgroup=true   active/enabled   CRI OK

worker-2          containerd 2.2.1   runc 1.3.4   crictl v1.34.0   SystemdCgroup=true   active/enabled   CRI OK

```



\---



\### Ключевые выводы



```text

containerd installed: OK

runc installed: OK

/etc/containerd/config.toml exists: OK

config version = 3: OK

SystemdCgroup = true: OK

containerd enabled: OK

containerd active: OK

overlayfs snapshotter: OK

runtime.v2 task plugin: OK

CRI images plugin: OK

CRI runtime plugin: OK

gRPC CRI plugin: OK

crictl v1.34.0 installed: OK

/etc/crictl.yaml configured: OK

RuntimeReady: true

NetworkReady: false — expected before CNI installation

containers: empty — expected

images: empty — expected

pods: empty — expected

```



\---



\### Ошибки и диагностика



| Симптом | Слой | Что проверил | Решение |

|---|---|---|---|

| После restart `containerd` в журнале появилась ошибка `failed to load cni during init` | Container runtime / CNI | Проверил `journalctl -u containerd`, `ctr plugins list`, `crictl info` | Ошибка ожидаема до установки CNI plugin. `RuntimeReady = true`, поэтому блокера для Lab 02 нет |

| `NetworkReady = false` в выводе `crictl info` | CRI / CNI | Проверил поле `NetworkReady` и текст ошибки | Ожидаемое состояние: pod network еще не настроен |

| Списки `crictl ps`, `crictl images`, `crictl pods` пустые | CRI runtime | Проверил runtime через `crictl` | Ожидаемое состояние до установки `kubelet` |



\---



\### Что стало понятнее



\- `containerd` — это основной container runtime на наших нодах. Он отвечает за работу с контейнерами и будет использоваться Kubernetes через CRI.

\- `runc` находится уровнем ниже: именно он непосредственно запускает контейнеры по OCI-спецификации.

\- Команда `ctr` полезна, когда нужно проверить сам `containerd`: плагины, snapshotter, namespaces и состояние runtime.

\- `crictl` удобнее для проверки Kubernetes-сценария, потому что он работает через CRI и показывает состояние runtime так, как его будет видеть `kubelet`.

\- Параметр:



```text

SystemdCgroup = true
```



\- Пустые списки containers, images и pods до установки Kubernetes components являются нормальным состоянием.

\- Значение:



```text

NetworkReady = false

```



не является проблемой до установки CNI plugin.

\- Главный критерий готовности runtime на текущем этапе:



```text

RuntimeReady = true

```



\---



\### Вопросы для дальнейшего изучения



\- Как изменится состояние `NetworkReady` после установки CNI plugin?

\- В чем практическая разница между диагностикой через `ctr` и через `crictl`?

\- Есть более свежая версия, мы будем обновлять или будем на `1.34`?

\---



\### Статус



Done

