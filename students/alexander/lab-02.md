# Lab 02 — Container Runtime: containerd

## Дата

2026-06-01

## Цель

Установить и настроить `containerd` как container runtime для будущего Kubernetes-кластера.

После завершения лабораторной на каждой node должны быть:

- установлен `containerd`;
- установлен `runc`;
- создан `/etc/containerd/config.toml`;
- включён `SystemdCgroup = true`;
- сервис `containerd` запущен и включён в автозагрузку;
- загружен CRI plugin;
- доступен `overlayfs` snapshotter;
- установлен и настроен `crictl`.

---

## Inventory

| Node | Role | Internal IP | OS |
|---|---|---|---|
| control-plane-1 | control-plane | 192.168.1.15/24 | Ubuntu Server 24.04.4 LTS |
| worker-1 | worker | 192.168.1.16/24 | Ubuntu Server 24.04.4 LTS |
| worker-2 | worker | 192.168.1.17/24 | Ubuntu Server 24.04.4 LTS |

---

## Проверка доступных пакетов

Перед установкой проверены доступные версии пакетов:

```text
containerd:
  Installed: (none)
  Candidate: 2.2.1-0ubuntu1~24.04.2
  Version table:
     2.2.1-0ubuntu1~24.04.2 500
        500 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 Packages
     1.7.28-0ubuntu1~24.04.2 500
        500 http://security.ubuntu.com/ubuntu noble-security/main amd64 Packages
     1.7.12-0ubuntu4 500
        500 http://ru.archive.ubuntu.com/ubuntu noble/main amd64 Packages

runc:
  Installed: (none)
  Candidate: 1.3.4-0ubuntu1~24.04.1
  Version table:
     1.3.4-0ubuntu1~24.04.1 500
        500 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 Packages
     1.3.3-0ubuntu1~24.04.3 500
        500 http://security.ubuntu.com/ubuntu noble-security/main amd64 Packages
     1.1.12-0ubuntu3 500
        500 http://ru.archive.ubuntu.com/ubuntu noble/main amd64 Packages
```

По результатам проверки использован основной сценарий для `containerd 2.x`:

```text
containerd 2.2.1
config version = 3
```

---

## Установка containerd и runc

На каждой node выполнены команды:

```bash
apt update
apt install -y containerd runc
```

### control-plane-1

```text
root@control-plane-1:~# containerd --version
containerd github.com/containerd/containerd/v2 2.2.1

root@control-plane-1:~# runc --version
runc version 1.3.4-0ubuntu1~24.04.1
spec: 1.2.1
go: go1.24.4
libseccomp: 2.5.5

root@control-plane-1:~# systemctl is-enabled containerd
enabled

root@control-plane-1:~# systemctl is-active containerd
active
```

### worker-1

```text
root@worker-1:~# containerd --version
containerd github.com/containerd/containerd/v2 2.2.1

root@worker-1:~# runc --version
runc version 1.3.4-0ubuntu1~24.04.1
spec: 1.2.1
go: go1.24.4
libseccomp: 2.5.5

root@worker-1:~# systemctl is-enabled containerd
enabled

root@worker-1:~# systemctl is-active containerd
active
```

### worker-2

```text
root@worker-2:~# containerd --version
containerd github.com/containerd/containerd/v2 2.2.1

root@worker-2:~# runc --version
runc version 1.3.4-0ubuntu1~24.04.1
spec: 1.2.1
go: go1.24.4
libseccomp: 2.5.5

root@worker-2:~# systemctl is-enabled containerd
enabled

root@worker-2:~# systemctl is-active containerd
active
```

---

## Проверка default config containerd

После установки на `worker-1` проверено наличие `/etc/containerd/config.toml`:

```text
root@worker-1:~# test -f /etc/containerd/config.toml \
>   && sed -n '1,220p' /etc/containerd/config.toml \
>   || echo "no /etc/containerd/config.toml"

no /etc/containerd/config.toml
```

До создания файла `containerd` работал с default config.

Ключевые строки default config:

```text
root@worker-1:~# containerd config default \
>   | sed -n '1,220p' \
>   | grep -nE 'version|SystemdCgroup|io.containerd.cri|runc|sandbox_image'

1:version = 3
39:  [plugins.'io.containerd.cri.v1.images']
50:    [plugins.'io.containerd.cri.v1.images'.pinned_images]
53:    [plugins.'io.containerd.cri.v1.images'.registry]
56:    [plugins.'io.containerd.cri.v1.images'.image_decryption]
59:  [plugins.'io.containerd.cri.v1.runtime']
79:    [plugins.'io.containerd.cri.v1.runtime'.containerd]
80:      default_runtime_name = 'runc'
84:      [plugins.'io.containerd.cri.v1.runtime'.containerd.runtimes]
85:        [plugins.'io.containerd.cri.v1.runtime'.containerd.runtimes.runc]
86:          runtime_type = 'io.containerd.runc.v2'
100:          [plugins.'io.containerd.cri.v1.runtime'.containerd.runtimes.runc.options]
109:            SystemdCgroup = false
111:    [plugins.'io.containerd.cri.v1.runtime'.cni]
```

---

## Настройка SystemdCgroup

На каждой node выполнены команды:

```bash
mkdir -p /etc/containerd

if test -f /etc/containerd/config.toml; then
  cp /etc/containerd/config.toml \
    /etc/containerd/config.toml.bak.$(date +%Y%m%d-%H%M%S)
fi

containerd config default > /etc/containerd/config.toml

sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' \
  /etc/containerd/config.toml

systemctl restart containerd
```

### control-plane-1

```text
root@control-plane-1:~# grep -nE 'version|SystemdCgroup|default_runtime_name|runtime_type' /etc/containerd/config.toml
1:version = 3
80:      default_runtime_name = 'runc'
86:          runtime_type = 'io.containerd.runc.v2'
109:            SystemdCgroup = true

root@control-plane-1:~# systemctl is-enabled containerd
enabled

root@control-plane-1:~# systemctl is-active containerd
active
```

### worker-1

```text
root@worker-1:~# grep -nE 'version|SystemdCgroup|default_runtime_name|runtime_type' /etc/containerd/config.toml
1:version = 3
80:      default_runtime_name = 'runc'
86:          runtime_type = 'io.containerd.runc.v2'
109:            SystemdCgroup = true

root@worker-1:~# systemctl is-enabled containerd
enabled

root@worker-1:~# systemctl is-active containerd
active
```

### worker-2

```text
root@worker-2:~# grep -nE 'version|SystemdCgroup|default_runtime_name|runtime_type' /etc/containerd/config.toml
1:version = 3
80:      default_runtime_name = 'runc'
86:          runtime_type = 'io.containerd.runc.v2'
109:            SystemdCgroup = true

root@worker-2:~# systemctl is-enabled containerd
enabled

root@worker-2:~# systemctl is-active containerd
active
```

---

## Ожидаемый CNI warning

После restart `containerd` на `worker-1` в журнале появилась строка:

```text
level=error msg="failed to load cni during init, please check CRI plugin status before setting up network for pods" error="cni config load failed: no network config found in /etc/cni/net.d: cni plugin not initialized: failed to load cni config"
```

Это ожидаемое состояние на текущем этапе:

```text
CNI plugin ещё не установлен.
Pod network ещё не настроен.
```

Предупреждение не является блокером для Lab 02.

---

## Проверка containerd через ctr

На каждой node выполнены команды:

```bash
ctr namespaces list
ctr plugins list | grep -E 'cri|runtime|snapshotter'
```

### control-plane-1

```text
root@control-plane-1:~# ctr namespaces list
NAME LABELS

root@control-plane-1:~# ctr plugins list | grep -E 'overlayfs|runtime.v2|io.containerd.cri.v1|io.containerd.grpc.v1.*cri'
io.containerd.snapshotter.v1              overlayfs                linux/amd64    ok
io.containerd.runtime.v2                  task                     linux/amd64    ok
io.containerd.cri.v1                      images                   -              ok
io.containerd.cri.v1                      runtime                  linux/amd64    ok
io.containerd.grpc.v1                     cri                      -              ok
```

### worker-1

```text
root@worker-1:~# ctr namespaces list
NAME LABELS

root@worker-1:~# ctr plugins list | grep -E 'cri|runtime|snapshotter'
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

### worker-2

```text
root@worker-2:~# ctr namespaces list
NAME LABELS

root@worker-2:~# ctr plugins list | grep -E 'overlayfs|runtime.v2|io.containerd.cri.v1|io.containerd.grpc.v1.*cri'
io.containerd.snapshotter.v1              overlayfs                linux/amd64    ok
io.containerd.runtime.v2                  task                     linux/amd64    ok
io.containerd.cri.v1                      images                   -              ok
io.containerd.cri.v1                      runtime                  linux/amd64    ok
io.containerd.grpc.v1                     cri                      -              ok
```

Пустой список namespaces на чистой node является ожидаемым состоянием: контейнеры ещё не запускались.

---

## Установка и настройка crictl

На каждой node установлен `crictl v1.34.0`.

Команды установки:

```bash
CRICTL_VERSION="v1.34.0"
TMP_DIR="$(mktemp -d)"

curl -L \
  "https://github.com/kubernetes-sigs/cri-tools/releases/download/${CRICTL_VERSION}/crictl-${CRICTL_VERSION}-linux-amd64.tar.gz" \
  -o "${TMP_DIR}/crictl-${CRICTL_VERSION}-linux-amd64.tar.gz"

tar -tzf "${TMP_DIR}/crictl-${CRICTL_VERSION}-linux-amd64.tar.gz"

tar -C /usr/local/bin \
  -xzf "${TMP_DIR}/crictl-${CRICTL_VERSION}-linux-amd64.tar.gz"

chmod 0755 /usr/local/bin/crictl
chown root:root /usr/local/bin/crictl
```

Проверка на `worker-1`:

```text
root@worker-1:~# command -v crictl
/usr/local/bin/crictl

root@worker-1:~# crictl --version
crictl version v1.34.0

root@worker-1:~# ls -l /usr/local/bin/crictl
-rwxr-xr-x 1 root root 40548006 Aug 21  2025 /usr/local/bin/crictl
```

На `control-plane-1` и `worker-2` получен тот же результат:

```text
crictl version v1.34.0
```

---

## Настройка /etc/crictl.yaml

На каждой node создан файл:

```bash
cat <<'EOF' > /etc/crictl.yaml
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 10
debug: false
EOF
```

Проверка:

```text
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 10
debug: false
```

---

## Проверка CRI runtime через crictl

На каждой node выполнена команда:

```bash
crictl info
```

Ключевые значения:

```text
runtimeType: io.containerd.runc.v2
SystemdCgroup: true
RuntimeReady: true
NetworkReady: false
```

Фрагмент вывода `crictl info` на `worker-1`:

```text
{
  "config": {
    "containerd": {
      "defaultRuntimeName": "runc",
      "runtimes": {
        "runc": {
          "options": {
            "SystemdCgroup": true
          },
          "runtimeType": "io.containerd.runc.v2"
        }
      }
    }
  },
  "lastCNILoadStatus": "cni config load failed: no network config found in /etc/cni/net.d: cni plugin not initialized: failed to load cni config",
  "status": {
    "conditions": [
      {
        "message": "",
        "reason": "",
        "status": true,
        "type": "RuntimeReady"
      },
      {
        "message": "Network plugin returns error: cni plugin not initialized",
        "reason": "NetworkPluginNotReady",
        "status": false,
        "type": "NetworkReady"
      }
    ]
  }
}
```

`NetworkReady: false` является ожидаемым состоянием до установки CNI plugin.

Главный критерий готовности runtime на текущем этапе:

```text
RuntimeReady: true
```

---

## Проверка пустого runtime

До установки `kubelet` и Kubernetes components на каждой node выполнены команды:

```bash
crictl ps
crictl ps -a
crictl images
crictl pods
```

На всех nodes получен аналогичный результат:

```text
root@worker-1:~# crictl ps
CONTAINER           IMAGE               CREATED             STATE               NAME                ATTEMPT             POD ID              POD                 NAMESPACE

root@worker-1:~# crictl ps -a
CONTAINER           IMAGE               CREATED             STATE               NAME                ATTEMPT             POD ID              POD                 NAMESPACE

root@worker-1:~# crictl images
IMAGE               TAG                 IMAGE ID            SIZE

root@worker-1:~# crictl pods
POD ID              CREATED             STATE               NAME                NAMESPACE           ATTEMPT             RUNTIME
```

Пустые списки являются ожидаемым состоянием: `kubelet` ещё не создавал containers и pods через CRI.

---

## Итоговый checkpoint

```text
control-plane-1   containerd 2.2.1   runc 1.3.4   crictl v1.34.0   SystemdCgroup=true   active/enabled   CRI OK
worker-1          containerd 2.2.1   runc 1.3.4   crictl v1.34.0   SystemdCgroup=true   active/enabled   CRI OK
worker-2          containerd 2.2.1   runc 1.3.4   crictl v1.34.0   SystemdCgroup=true   active/enabled   CRI OK
```

---

## Ключевые выводы

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

---

## Ошибки и диагностика

| Симптом | Слой | Что проверил | Решение |
|---|---|---|---|
| После restart `containerd` в журнале появилась ошибка `failed to load cni during init` | Container runtime / CNI | Проверил `journalctl -u containerd`, `ctr plugins list`, `crictl info` | Ошибка ожидаема до установки CNI plugin. `RuntimeReady = true`, поэтому блокера для Lab 02 нет |
| `NetworkReady = false` в выводе `crictl info` | CRI / CNI | Проверил поле `NetworkReady` и текст ошибки | Ожидаемое состояние: pod network ещё не настроен |
| Списки `crictl ps`, `crictl images`, `crictl pods` пустые | CRI runtime | Проверил runtime через `crictl` | Ожидаемое состояние до установки `kubelet` |

---

## Что стало понятнее

- `containerd` — основной container runtime на наших нодах. Kubernetes будет взаимодействовать с ним через CRI.
- `runc` находится уровнем ниже и непосредственно запускает контейнеры по OCI-спецификации.
- `ctr` полезен для диагностики самого `containerd`: plugins, snapshotter и namespaces.
- `crictl` показывает состояние runtime через CRI, то есть ближе к тому, как его будет видеть `kubelet`.
- Параметр `SystemdCgroup = true` нужен для согласованной работы `containerd` и будущего `kubelet`.
- Пустые списки containers, images и pods до установки Kubernetes components являются нормальным состоянием.
- `NetworkReady = false` не является проблемой до установки CNI plugin.
- Главный критерий готовности runtime на текущем этапе — `RuntimeReady = true`.

---

## Вопросы для дальнейшего изучения

- Как изменится состояние `NetworkReady` после установки CNI plugin?
- В чём практическая разница между диагностикой через `ctr` и через `crictl`?
- Сейчас на стенде используется `crictl v1.34.0`. Будем ли мы обновлять его в следующих лабораторных работах или продолжим использовать эту версию для совместимости с выбранной версией Kubernetes?

---

## Статус

Done
