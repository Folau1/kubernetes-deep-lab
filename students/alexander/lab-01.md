# Lab Journal

Lab journal — рабочий журнал участника Kubernetes Deep Lab.

Он нужен, чтобы фиксировать прогресс, команды, выводы, ошибки, вопросы и результаты по каждой лабораторной.

## Участник

| Поле | Значение |
|---|---|
| Имя | Александр |
| GitHub | [Folau1](https://github.com/Folau1) |

## Общий прогресс

| Lab | Status | PR | Notes |
|---|---|---|---|
| Lab 00 — Environment Validation | done |  | Локальный стенд подготовлен |
| Lab 01 — Node Baseline | done |  | Ноды готовы к установке container runtime |

## Lab 01 — Node Baseline

### Дата

2026-06-01

### Цель

Привести каждую VM к предсказуемому базовому состоянию перед установкой container runtime и Kubernetes-компонентов.

### Что было сделано

- Проверены OS, hostname, IP, маршруты и ресурсы всех VM.
- Hostname приведены к единому lowercase-формату.
- Настроено локальное разрешение имён через `/etc/hosts`.
- Проверена node-to-node L3 connectivity.
- Проверены DNS и internet access на `control-plane-1`.
- Проверена доступность package repositories.
- Выполнен `apt update`.
- Проверена доступность пакетов `containerd` и `runc`.
- Проверена синхронизация времени.
- Подтверждено отсутствие swap.
- Загружены модули ядра `overlay` и `br_netfilter`.
- Настроены persistent kernel modules и sysctl-параметры.
- Проверена сохранность настроек после reboot.
- Подтверждено чистое состояние VM перед следующей лабораторной.

### Inventory

| Node | Role | Internal IP | OS | CPU | RAM | Root disk |
|---|---|---|---|---:|---:|---:|
| control-plane-1 | control-plane | 192.168.1.15/24 | Ubuntu Server 24.04.4 LTS | 2 vCPU | 3.8 GiB | 40G |
| worker-1 | worker | 192.168.1.16/24 | Ubuntu Server 24.04.4 LTS | 2 vCPU | 3.8 GiB | 30G |
| worker-2 | worker | 192.168.1.17/24 | Ubuntu Server 24.04.4 LTS | 2 vCPU | 3.8 GiB | 30G |

Default gateway:

```text
192.168.1.1
```

### Команды и полные выводы

#### control-plane-1

```text
=== NODE: control-plane-1 ===

=== INVENTORY ===
 Static hostname: control-plane-1
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: 9ad1bce9da55425cbfa3bac436d603b8
         Boot ID: dac778dff053439a9f17f008f6df61b1
  Virtualization: oracle
Operating System: Ubuntu 24.04.4 LTS
          Kernel: Linux 6.8.0-117-generic
    Architecture: x86-64
 Hardware Vendor: innotek GmbH
  Hardware Model: VirtualBox
Firmware Version: VirtualBox
   Firmware Date: Fri 2006-12-01
    Firmware Age: 19y 6month
lo               UNKNOWN        127.0.0.1/8 ::1/128
enp0s3           UP             192.168.1.15/24 fdc1:30bd:dce:0:a00:27ff:fe67:6b11/64 fe80::a00:27ff:fe67:6b11/64
default via 192.168.1.1 dev enp0s3 proto static
192.168.1.0/24 dev enp0s3 proto kernel scope link src 192.168.1.15
PRETTY_NAME="Ubuntu 24.04.4 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04.4 LTS (Noble Numbat)"
VERSION_CODENAME=noble
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=noble
LOGO=ubuntu-logo
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       414Mi       3.4Gi       1.0Mi       202Mi       3.4Gi
Swap:             0B          0B          0B
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda2        40G  2.9G   35G   8% /
2

=== HOSTNAME RESOLUTION ===
control-plane-1
control-plane-1
192.168.1.15    STREAM control-plane-1
192.168.1.15    DGRAM
192.168.1.15    RAW
192.168.1.15    control-plane-1
127.0.0.1 localhost

192.168.1.15 control-plane-1
192.168.1.16 worker-1
192.168.1.17 worker-2

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
ff02::3 ip6-allhosts

=== NODE-TO-NODE PING ===
PING 192.168.1.16 (192.168.1.16) 56(84) bytes of data.
64 bytes from 192.168.1.16: icmp_seq=1 ttl=64 time=1.25 ms
64 bytes from 192.168.1.16: icmp_seq=2 ttl=64 time=0.742 ms
64 bytes from 192.168.1.16: icmp_seq=3 ttl=64 time=0.755 ms

--- 192.168.1.16 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2052ms
rtt min/avg/max/mdev = 0.742/0.915/1.248/0.235 ms
PING 192.168.1.17 (192.168.1.17) 56(84) bytes of data.
64 bytes from 192.168.1.17: icmp_seq=1 ttl=64 time=1.78 ms
64 bytes from 192.168.1.17: icmp_seq=2 ttl=64 time=0.841 ms
64 bytes from 192.168.1.17: icmp_seq=3 ttl=64 time=1.22 ms

--- 192.168.1.17 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2009ms
rtt min/avg/max/mdev = 0.841/1.280/1.784/0.387 ms

=== SWAP ===
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       416Mi       3.4Gi       1.0Mi       202Mi       3.4Gi
Swap:             0B          0B          0B

=== MODULES AND SYSCTL ===
overlay
br_netfilter
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
br_netfilter           32768  0
bridge                425984  1 br_netfilter
overlay               212992  0
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1

=== DNS AND INTERNET ===
2606:4700:10::6814:1cf6 archive.ubuntu.com.cdn.cloudflare.net archive.ubuntu.com
2606:4700:10::ac42:98b0 archive.ubuntu.com.cdn.cloudflare.net archive.ubuntu.com
PING 192.168.1.1 (192.168.1.1) 56(84) bytes of data.
64 bytes from 192.168.1.1: icmp_seq=1 ttl=64 time=1.23 ms
64 bytes from 192.168.1.1: icmp_seq=2 ttl=64 time=1.03 ms
64 bytes from 192.168.1.1: icmp_seq=3 ttl=64 time=1.02 ms

--- 192.168.1.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 1.016/1.091/1.234/0.100 ms
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=1 ttl=57 time=45.2 ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=57 time=44.4 ms
64 bytes from 1.1.1.1: icmp_seq=3 ttl=57 time=45.0 ms

--- 1.1.1.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 44.365/44.845/45.203/0.352 ms
PING archive.ubuntu.com.cdn.cloudflare.net (172.66.152.176) 56(84) bytes of data.
64 bytes from 172.66.152.176: icmp_seq=1 ttl=57 time=45.2 ms
64 bytes from 172.66.152.176: icmp_seq=2 ttl=57 time=44.8 ms
64 bytes from 172.66.152.176: icmp_seq=3 ttl=57 time=45.0 ms

--- archive.ubuntu.com.cdn.cloudflare.net ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2038ms
rtt min/avg/max/mdev = 44.807/44.990/45.203/0.162 ms

=== PACKAGE REPOSITORIES ===
2606:4700:10::6814:1cf6 archive.ubuntu.com.cdn.cloudflare.net archive.ubuntu.com
2606:4700:10::ac42:98b0 archive.ubuntu.com.cdn.cloudflare.net archive.ubuntu.com
2606:4700:10::ac42:98b0 security.ubuntu.com.cdn.cloudflare.net security.ubuntu.com
2606:4700:10::6814:1cf6 security.ubuntu.com.cdn.cloudflare.net security.ubuntu.com
2600:1901:0:26f3:: redirect.k8s.io pkgs.k8s.io
140.82.121.3    github.com
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
HTTP/2 200
date: Mon, 01 Jun 2026 14:50:41 GMT
content-type: text/html;charset=UTF-8
server: cloudflare
set-cookie: <redacted>
cf-cache-status: DYNAMIC
cf-ray: a04f0839daa4ef91-WAW

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0   138    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
HTTP/2 302
server: nginx
date: Mon, 01 Jun 2026 14:50:41 GMT
content-type: text/html
content-length: 138
location: https://kubernetes.io/blog/2023/08/15/pkgs-k8s-io-introduction/
via: 1.1 google
alt-svc: h3=":443"; ma=2592000,h3-29=":443"; ma=2592000

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0HTTP/2 200
date: Mon, 01 Jun 2026 14:50:32 GMT
content-type: text/html; charset=utf-8
vary: X-PJAX, X-PJAX-Container, Turbo-Visit, Turbo-Frame, X-Requested-With, Accept-Language, Sec-Fetch-Site,Accept-Encoding, Accept, X-Requested-With
content-language: en-US
etag: W/"054dd5f899749b586a7d33c77e384e24"
cache-control: max-age=0, private, must-revalidate
strict-transport-security: max-age=31536000; includeSubdomains; preload
x-frame-options: deny
x-content-type-options: nosniff
x-xss-protection: 0
referrer-policy: origin-when-cross-origin, strict-origin-when-cross-origin
content-security-policy: <omitted>
le-src 'unsafe-inline' github.githubassets.com; upgrade-insecure-requests; worker-src github.githubassets.com github.com/assets-cdn/worker/ github.com/assets/ gist.github.com/assets-cdn/worker/
server: github.com
accept-ranges: bytes
set-cookie: <redacted>
set-cookie: <redacted>
set-cookie: <redacted>
x-github-request-id: AB34:1E8470:3A91D30:2E64096:6A1D9C42


WARNING: apt does not have a stable CLI interface. Use with caution in scripts.

Hit:1 http://ru.archive.ubuntu.com/ubuntu noble InRelease
Get:2 http://ru.archive.ubuntu.com/ubuntu noble-updates InRelease [126 kB]
Get:3 http://security.ubuntu.com/ubuntu noble-security InRelease [126 kB]
Get:4 http://ru.archive.ubuntu.com/ubuntu noble-backports InRelease [126 kB]
Get:5 http://security.ubuntu.com/ubuntu noble-security/main amd64 Packages [1,704 kB]
Get:6 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 Packages [2,050 kB]
Get:7 http://security.ubuntu.com/ubuntu noble-security/main Translation-en [268 kB]
Get:8 http://security.ubuntu.com/ubuntu noble-security/main amd64 Components [42.4 kB]
Get:9 http://security.ubuntu.com/ubuntu noble-security/restricted amd64 Packages [3,006 kB]
Get:10 http://ru.archive.ubuntu.com/ubuntu noble-updates/main Translation-en [360 kB]
Get:11 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 Components [177 kB]
Get:12 http://ru.archive.ubuntu.com/ubuntu noble-updates/restricted amd64 Packages [3,278 kB]
Get:13 http://security.ubuntu.com/ubuntu noble-security/restricted Translation-en [698 kB]
Get:14 http://security.ubuntu.com/ubuntu noble-security/universe amd64 Packages [1,192 kB]
Get:15 http://security.ubuntu.com/ubuntu noble-security/universe Translation-en [230 kB]
Get:16 http://security.ubuntu.com/ubuntu noble-security/universe amd64 Components [74.2 kB]
Get:17 http://security.ubuntu.com/ubuntu noble-security/multiverse Translation-en [9,000 B]
Get:18 http://ru.archive.ubuntu.com/ubuntu noble-updates/restricted Translation-en [760 kB]
Get:19 http://ru.archive.ubuntu.com/ubuntu noble-updates/universe amd64 Packages [1,694 kB]
Get:20 http://ru.archive.ubuntu.com/ubuntu noble-updates/universe Translation-en [330 kB]
Get:21 http://ru.archive.ubuntu.com/ubuntu noble-updates/universe amd64 Components [386 kB]
Get:22 http://ru.archive.ubuntu.com/ubuntu noble-updates/multiverse Translation-en [11.3 kB]
Get:23 http://ru.archive.ubuntu.com/ubuntu noble-updates/multiverse amd64 Components [940 B]
Get:24 http://ru.archive.ubuntu.com/ubuntu noble-backports/main amd64 Components [5,772 B]
Get:25 http://ru.archive.ubuntu.com/ubuntu noble-backports/universe amd64 Components [10.5 kB]
Fetched 16.7 MB in 4s (4,133 kB/s)
Reading package lists...
Building dependency tree...
Reading state information...
49 packages can be upgraded. Run 'apt list --upgradable' to see them.
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

=== TIME SYNC ===
               Local time: Mon 2026-06-01 14:51:02 UTC
           Universal time: Mon 2026-06-01 14:51:02 UTC
                 RTC time: Mon 2026-06-01 14:51:02
                Time zone: Etc/UTC (UTC, +0000)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
active
Timezone=Etc/UTC
NTPSynchronized=yes
TimeUSec=Mon 2026-06-01 14:51:02 UTC

=== CLEAN NODE CHECK ===
```

#### worker-1

```text
=== NODE: worker-1 ===

=== INVENTORY ===
 Static hostname: worker-1
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: 073f7868b1964915ae98ba90571f0bf4
         Boot ID: 596da33ef5634e49b757e35fdd87e425
  Virtualization: oracle
Operating System: Ubuntu 24.04.4 LTS
          Kernel: Linux 6.8.0-117-generic
    Architecture: x86-64
 Hardware Vendor: innotek GmbH
  Hardware Model: VirtualBox
Firmware Version: VirtualBox
   Firmware Date: Fri 2006-12-01
    Firmware Age: 19y 6month
lo               UNKNOWN        127.0.0.1/8 ::1/128
enp0s3           UP             192.168.1.16/24 fdc1:30bd:dce:0:a00:27ff:fedc:9b5/64 fe80::a00:27ff:fedc:9b5/64
default via 192.168.1.1 dev enp0s3 proto static
192.168.1.0/24 dev enp0s3 proto kernel scope link src 192.168.1.16
PRETTY_NAME="Ubuntu 24.04.4 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04.4 LTS (Noble Numbat)"
VERSION_CODENAME=noble
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=noble
LOGO=ubuntu-logo
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       428Mi       3.3Gi       1.0Mi       274Mi       3.4Gi
Swap:             0B          0B          0B
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda2        30G  2.9G   26G  11% /
2

=== HOSTNAME RESOLUTION ===
worker-1
worker-1
192.168.1.16    STREAM worker-1
192.168.1.16    DGRAM
192.168.1.16    RAW
192.168.1.16    worker-1
127.0.0.1 localhost

192.168.1.15 control-plane-1
192.168.1.16 worker-1
192.168.1.17 worker-2

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
ff02::3 ip6-allhosts

=== NODE-TO-NODE PING ===
PING 192.168.1.15 (192.168.1.15) 56(84) bytes of data.
64 bytes from 192.168.1.15: icmp_seq=1 ttl=64 time=0.750 ms
64 bytes from 192.168.1.15: icmp_seq=2 ttl=64 time=0.745 ms
64 bytes from 192.168.1.15: icmp_seq=3 ttl=64 time=0.792 ms

--- 192.168.1.15 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2029ms
rtt min/avg/max/mdev = 0.745/0.762/0.792/0.021 ms
PING 192.168.1.17 (192.168.1.17) 56(84) bytes of data.
64 bytes from 192.168.1.17: icmp_seq=1 ttl=64 time=1.90 ms
64 bytes from 192.168.1.17: icmp_seq=2 ttl=64 time=1.36 ms
64 bytes from 192.168.1.17: icmp_seq=3 ttl=64 time=0.877 ms

--- 192.168.1.17 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 0.877/1.379/1.897/0.416 ms

=== SWAP ===
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       428Mi       3.3Gi       1.0Mi       275Mi       3.4Gi
Swap:             0B          0B          0B

=== MODULES AND SYSCTL ===
overlay
br_netfilter
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
br_netfilter           32768  0
bridge                425984  1 br_netfilter
overlay               212992  0
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1

=== CLEAN NODE CHECK ===
```

#### worker-2

```text
=== NODE: worker-2 ===

=== INVENTORY ===
 Static hostname: worker-2
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: 37dbcca896764718866c8f1e799af019
         Boot ID: ed0c4f4a40ef406a8833199be45b97ba
  Virtualization: oracle
Operating System: Ubuntu 24.04.4 LTS
          Kernel: Linux 6.8.0-117-generic
    Architecture: x86-64
 Hardware Vendor: innotek GmbH
  Hardware Model: VirtualBox
Firmware Version: VirtualBox
   Firmware Date: Fri 2006-12-01
    Firmware Age: 19y 6month
lo               UNKNOWN        127.0.0.1/8 ::1/128
enp0s3           UP             192.168.1.17/24 fdc1:30bd:dce:0:a00:27ff:fe57:3beb/64 fe80::a00:27ff:fe57:3beb/64
default via 192.168.1.1 dev enp0s3 proto static
192.168.1.0/24 dev enp0s3 proto kernel scope link src 192.168.1.17
PRETTY_NAME="Ubuntu 24.04.4 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04.4 LTS (Noble Numbat)"
VERSION_CODENAME=noble
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=noble
LOGO=ubuntu-logo
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       419Mi       3.4Gi       1.0Mi       273Mi       3.4Gi
Swap:             0B          0B          0B
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda2        30G  2.9G   26G  11% /
2

=== HOSTNAME RESOLUTION ===
worker-2
worker-2
192.168.1.17    STREAM worker-2
192.168.1.17    DGRAM
192.168.1.17    RAW
192.168.1.17    worker-2
127.0.0.1 localhost

192.168.1.15 control-plane-1
192.168.1.16 worker-1
192.168.1.17 worker-2

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
ff02::3 ip6-allhosts

=== NODE-TO-NODE PING ===
PING 192.168.1.15 (192.168.1.15) 56(84) bytes of data.
64 bytes from 192.168.1.15: icmp_seq=1 ttl=64 time=0.660 ms
64 bytes from 192.168.1.15: icmp_seq=2 ttl=64 time=1.46 ms
64 bytes from 192.168.1.15: icmp_seq=3 ttl=64 time=2.78 ms

--- 192.168.1.15 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2438ms
rtt min/avg/max/mdev = 0.660/1.632/2.775/0.871 ms
PING 192.168.1.16 (192.168.1.16) 56(84) bytes of data.
64 bytes from 192.168.1.16: icmp_seq=1 ttl=64 time=0.617 ms
64 bytes from 192.168.1.16: icmp_seq=2 ttl=64 time=3.37 ms
64 bytes from 192.168.1.16: icmp_seq=3 ttl=64 time=0.495 ms

--- 192.168.1.16 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2355ms
rtt min/avg/max/mdev = 0.495/1.493/3.369/1.326 ms

=== SWAP ===
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       419Mi       3.4Gi       1.0Mi       273Mi       3.4Gi
Swap:             0B          0B          0B

=== MODULES AND SYSCTL ===
overlay
br_netfilter
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
br_netfilter           32768  0
bridge                425984  1 br_netfilter
overlay               212992  0
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1

=== CLEAN NODE CHECK ===
```

### Ключевые выводы

```text
VM inventory: OK
OS/resources: OK
DNS/internet: OK
outbound HTTPS: OK
time sync: OK
hostname resolution: OK
node-to-node L3 connectivity: OK
swap disabled: OK
kernel/network prerequisites: OK
reboot persistence: OK
package manager readiness: OK
clean nodes, no Kubernetes/runtime: OK
```

Серверы готовы к следующей лабораторной — установке `containerd`.

На всех VM подтверждено:

```text
overlay
br_netfilter
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
Swap: 0B
```

Проверка чистого состояния завершилась пустым выводом для:

```text
containerd
runc
crictl
kubeadm
kubelet
kubectl
```

### Ошибки и диагностика

| Симптом | Слой | Что проверил | Решение |
|---|---|---|---|
| Hostname части VM использовали смешанный регистр | OS / hostname | `hostname`, `hostnamectl` | Hostname изменены на `control-plane-1`, `worker-1`, `worker-2` |
| Во время первичной проверки временно не проходил `ping` до одной из VM | Network / L3 | Проверил состояние VM и повторил `ping` при включённых нодах | После включения всех VM связность подтверждена: `0% packet loss` |
| `getent hosts "$(hostname)"` мог возвращать IPv6 раньше IPv4 | Name resolution | Дополнительно выполнил `getent ahostsv4 "$(hostname)"` | Подтверждены ожидаемые internal IPv4 всех nodes |

### Что стало понятнее

- Для Kubernetes недостаточно просто создать VM и проверить доступ по SSH. Перед установкой компонентов нужно привести каждую node к одинаковому и предсказуемому состоянию.
- Hostname лучше сразу задавать в едином lowercase-формате.
- Для отдельной проверки внутреннего IPv4 удобно использовать:

```bash
getent ahostsv4 "$(hostname)"
```

- Записи всех nodes в `/etc/hosts` упрощают локальное разрешение имён и диагностику сетевой связности до появления cluster DNS.
- Для Kubernetes networking нужны модули ядра:

```text
overlay
br_netfilter
```

- Параметры в `/etc/sysctl.d/k8s.conf` и модули в `/etc/modules-load.d/k8s.conf` важно проверять не только после настройки, но и после reboot.
- Перед установкой `containerd`, `kubeadm` и `kubelet` полезно зафиксировать чистое состояние VM.
- При проверке `ping` важно убедиться, что все VM одновременно включены.

### Вопросы и ответы

#### Почему `getent hosts` иногда показывает IPv6-адрес раньше IPv4?

`getent hosts` использует системный механизм разрешения имён и может вернуть доступные адреса в порядке, определённом настройками resolver. Наличие IPv6-адреса в выводе не означает, что IPv4 отсутствует.

Для отдельной проверки internal IPv4 использовалась команда:

```bash
getent ahostsv4 "$(hostname)"
```

Результат:

```text
control-plane-1 -> 192.168.1.15
worker-1        -> 192.168.1.16
worker-2        -> 192.168.1.17
```

#### Нужен ли IPv6 для дальнейших лабораторных работ?

Текущий стенд строится вокруг внутренних IPv4-адресов. При этом полностью удалять IPv6-настройки без необходимости не стал. В baseline оставлен параметр:

```text
net.bridge.bridge-nf-call-ip6tables = 1
```

Он не мешает IPv4-стенду и пригодится, если в дальнейшем будет рассматриваться dual-stack.

#### Почему проверку time sync в этой лабораторной достаточно выполнить на control-plane node?

В README это минимальная контрольная точка: control-plane особенно чувствителен к проблемам со временем из-за TLS-сертификатов, bootstrap-процедур и будущей работы etcd.

При этом корректное время важно на всех nodes: для kubelet, журналов событий и диагностики. Перед вводом кластера в эксплуатацию time sync стоит проверить на каждой VM.

#### Правильно ли прописывать все nodes в `/etc/hosts` на каждой VM?

Да. Для небольшого учебного стенда это удобный и предсказуемый вариант:

```text
192.168.1.15 control-plane-1
192.168.1.16 worker-1
192.168.1.17 worker-2
```

Так каждая VM может разрешить имена остальных nodes ещё до установки Kubernetes и cluster DNS.

### Вопросы для дальнейшего изучения

- В каких случаях для production-стенда лучше использовать внутренний DNS вместо статических записей в `/etc/hosts`?
- Какие дополнительные проверки времени стоит выполнять перед добавлением worker nodes в кластер?
- Как меняется подготовка nodes при настройке Kubernetes dual-stack?
