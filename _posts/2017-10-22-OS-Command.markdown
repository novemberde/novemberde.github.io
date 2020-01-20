---
title: 유용한 OS 명령어 모음
category: os
tags: os, command, 명령어, 사용법, 모음
---
## Summary
---
개발을 하다가 명령어를 모두 기억하고 싶어도 잠깐 다른 업무를 하다가 다시하면 잊게되는 명령어들이 있다.
특히 OS 관련 명령어는 서버에 문제가 생겼을 때 사용하기 때문에 더욱 그렇다.
이번에는 자주 사용하지 않았으면 하는 명령어들을 정리해본다.

---
# OS 명령어
---

sar
- 리소스 사용 내역을 볼 수 있다.
- CPU  사용율
- Idle 상태 상태 확인
- I/O 의 사용율
- 메모리 사용율


```sh
$ sudo apt-get install sysstat
$ sudo vi /etc/default/sysstat
ENABLED="true"

$ sar -u 1 5
Linux 4.4.0-1013-aws (ip-172-31-55-55)         10/22/2017      _x86_64_        (1 CPU)

05:52:57 AM     CPU     %user     %nice   %system   %iowait    %steal     %idle
05:52:58 AM     all      0.00      0.00      0.00      0.00      0.00    100.00
05:52:59 AM     all      0.00      0.00      0.00      0.00      0.00    100.00
05:53:00 AM     all      0.00      0.00      0.00      0.00      0.00    100.00
05:53:01 AM     all      0.00      0.00      0.00      0.00      0.00    100.00
05:53:02 AM     all      0.00      0.00      1.98      0.00      0.00     98.02
Average:        all      0.00      0.00      0.40      0.00      0.00     99.60
```

---

top
- Process의 snapshot 정보
- 어느정도 부하가 있는 명령어

```sh
$ top
19224 aws-kin+  20   0 2480936 169728   8696 S  2.9 16.7  24:48.81 java
    1 root      20   0   37856   4896   2976 S  0.0  0.5   1:00.60 systemd
    2 root      20   0       0      0      0 S  0.0  0.0   0:00.00 kthreadd
    3 root      20   0       0      0      0 S  0.0  0.0   0:08.99 ksoftirqd/0
    5 root       0 -20       0      0      0 S  0.0  0.0   0:00.00 kworker/0:0H
    7 root      20   0       0      0      0 S  0.0  0.0   4:34.75 rcu_sched
    8 root      20   0       0      0      0 S  0.0  0.0   0:00.00 rcu_bh
    9 root      rt   0       0      0      0 S  0.0  0.0   0:00.00 migration/0
```

---

vmstat
- OS의 커널을 통해 취득한 정보
- 메모리 상태
- 프로세스의 개수
- CPU 사용률
- Swap I/O
- I/O
- Context switch 수

```sh
$ vmstat 1 5
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0      0 151836  89728 508492    0    0     1     4    2    3  0  0 100  0  0
 0  0      0 151852  89728 508492    0    0     0     0  121  381  0  0 100  0  0
 0  0      0 151804  89728 508492    0    0     0     0  176  415  1  0 99  0  0
 0  0      0 151804  89728 508492    0    0     0     0  117  372  0  0 100  0  0
 0  0      0 151804  89728 508492    0    0     0     8  121  389  0  0 100  0  0
```


---

ps
- OS의 커널로 프로세스 정보를 얻는다.
- ps 명령어를 실행한 시점에 어떤 프로세스가 동작하는지 상태 등을 확인
- 프로세스의 CPU 사용 누적시간
- 프로세스 번호, 이름, 명령어

```sh
$ ps -elf
F S UID        PID  PPID  C PRI  NI ADDR SZ WCHAN  STIME TTY          TIME CMD
4 S root         1     0  0  80   0 -  9464 -      Jun11 ?        00:01:00 /sbin/init
1 S root         2     0  0  80   0 -     0 -      Jun11 ?        00:00:00 [kthreadd]
1 S root         3     2  0  80   0 -     0 -      Jun11 ?        00:00:09 [ksoftirqd/0]
1 S root         5     2  0  60 -20 -     0 -      Jun11 ?        00:00:00 [kworker/0:0H]
1 S root         7     2  0  80   0 -     0 -      Jun11 ?        00:04:35 [rcu_sched]
1 S root         8     2  0  80   0 -     0 -      Jun11 ?        00:00:00 [rcu_bh]
1 S root         9     2  0 -40   - -     0 -      Jun11 ?        00:00:00 [migration/0]
5 S root        10     2  0 -40   - -     0 -      Jun11 ?        00:00:50 [watchdog/0]
5 S root        11     2  0  80   0 -     0 -      Jun11 ?        00:00:00 [kdevtmpfs]
...

$ ps -elf | grep init
4 S root         1     0  0  80   0 -  9464 -      Jun11 ?        00:01:00 /sbin/init
0 S ubuntu   26767 23875  0  80   0 -  3237 pipe_w 06:36 pts/0    00:00:00 grep --color=auto
--exclude-dir=.bzr --exclude-dir=CVS --exclude-dir=.git --exclude-dir=.hg --exclude-dir=.svn init
```

---

netstat
- 드라이버 수준에서 네트워크 정보를 출력
- option
  - -a: 소켓 정보
  - -r: 라우팅 정보
  - -i: 인터페이스 단위 정보

```sh
$ netstat -i
Kernel Interface table
Iface   MTU Met   RX-OK RX-ERR RX-DRP RX-OVR    TX-OK TX-ERR TX-DRP TX-OVR Flg
docker0    1500 0     26198      0      0 0         25454      0      0      0 BMU
docker_gwbridge  1500 0         8      0      0 0             8      0      0      0 BMRU
eth0       9001 0   6761565      0      0 0       3220652      0      0      0 BMRU
lo        65536 0   3854159      0      0 0       3854159      0      0      0 LRU
veth2ea2a18  1500 0         0      0      0 0             8      0      0      0 BMRU

$ netstat -a
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 *:ssh                   *:*                     LISTEN
...

Active UNIX domain sockets (servers and established)
Proto RefCnt Flags       Type       State         I-Node   Path
unix  3      [ ]         DGRAM                    8768     /run/systemd/notify
...

$ netstat -r
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
default         ip-172-31-32-1. 0.0.0.0         UG        0 0          0 eth0
172.17.0.0      *               255.255.0.0     U         0 0          0 docker0
172.18.0.0      *               255.255.0.0     U         0 0          0 docker_gwbridge
172.31.32.0     *               255.255.240.0   U         0 0          0 eth0

$ netstat -tnlp
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -
tcp6       0      0 :::2377                 :::*                    LISTEN      -
tcp6       0      0 :::7946                 :::*                    LISTEN      -
tcp6       0      0 :::22                   :::*                    LISTEN      -

$ netstat -ant | grep 5432 | wc -l
```

---

iostat
- 블록 장비 수준으로 OS 커널 내부에서 측정한다.
- 디스크 사용률
- option
  - -x: Response time, queue length

```sh
$ iostat
Linux 4.4.0-1013-aws         10/22/2017      _x86_64_        (1 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
          0.06    0.00    0.02    0.01    0.01   99.91

Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
xvda              0.18         0.90         4.01   10298246   45935300

$ iostat -x
Linux 4.4.0-1013-aws       10/22/2017      _x86_64_        (1 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.06    0.00    0.02    0.01    0.01   99.91

Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
xvda              0.00     0.10    0.07    0.11     0.90     4.01    54.75     0.00    2.98    1.08    4.26   0.49   0.01
```


---

lsof
- list open files
- COMMAND : 실행한 명령어
- PID : process id
- USER : 실행한 사용자
- FD: File Descriptor, 파일의 종류.
  - cwd: current working directory
  - rtd: root directory
  - mem : memory-mapped file
  - txt: program text (code and data);
- TYPE: 파일 종류
  - DIR: 디렉터리
  - CHR:  character special file
  - REG: regular file
  - unix: 유닉스 도메인 소켓 (MySQL 등이 사용하는 소켓으로 로컬 프로세스에서만 사용 가능하며 TCP/UDP 보다 속도가 매우 빠름)
- DEVICE : 장치 번호
- SIZE/OFF: 파일의 크기나 오프셋
- NODE: 노드 번호
- NAME:  파일명

```sh
$ lsof -c docker
$ lsof +D /tmp
$ sudo lsof -i TCP:22
```

---

grep
```sh
Usage: grep [OPTION]... PATTERN [FILE]...
Search for PATTERN in each FILE or standard input.
PATTERN is, by default, a basic regular expression (BRE).
Example: grep -i 'hello world' menu.h main.c

$ grep -nir "test"
```

---

ifconfig
- 네트워크 정보 출력

```sh
eth0      Link encap:Ethernet  HWaddr 12:99:72:e5:aa:aa
          inet addr:172.31.58.121  Bcast:172.31.63.255  Mask:255.255.240.0
          inet6 addr: fe80::1099:72ff:fee5:aaaa/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:9001  Metric:1
          RX packets:173257 errors:0 dropped:0 overruns:0 frame:0
          TX packets:46138 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:213551810 (213.5 MB)  TX bytes:20486641 (20.4 MB)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:160 errors:0 dropped:0 overruns:0 frame:0
          TX packets:160 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1
          RX bytes:11840 (11.8 KB)  TX bytes:11840 (11.8 KB)
```

---

free
- 메모리 사용량 출력

```sh
              total        used        free      shared  buff/cache   available
Mem:        1014660       50304      109820        5704      854536      750084
Swap:             0           0           0
```

---

nslookup
- domain name server 정보 출력

```sh
$ nslookup naver.com
Server:         172.31.0.2
Address:        172.31.0.2#53

Non-authoritative answer:
Name:   naver.com
Address: 125.209.222.141
Name:   naver.com
Address: 125.209.222.142
Name:   naver.com
Address: 202.179.177.21
Name:   naver.com
Address: 202.179.177.22
```


---

watch
- 파일 상태 실시간 확인

```sh
$ watch --help
Usage:
 watch [options] command

Options:
  -b, --beep             beep if command has a non-zero exit
  -c, --color            interpret ANSI color and style sequences
  -d, --differences[=<permanent>]
                         highlight changes between updates
  -e, --errexit          exit if command has a non-zero exit
  -g, --chgexit          exit when output from command changes
  -n, --interval <secs>  seconds to wait between updates
  -p, --precise          attempt run command in precise intervals
  -t, --no-title         turn off header
  -x, --exec             pass command to exec instead of "sh -c"

 -h, --help     display this help and exit
 -v, --version  output version information and exit

For more details see watch(1).

$ watch -n 2 ls -al
```

---

adduser
- 유저 생성

```sh
# 비밀번호 없이 유저 생성
$ sudo adduser myUser --disabled-password --ingroup ubuntu
```


---

who
- 현재 접속하고 있는 유저 리스트 출력
```sh
$ who
ubuntu   pts/0        2017-10-30 02:08 (232.232.232.121)
```

---

whoami
- 현재 사용하는 유저 출력
```sh
$ whoami
ubuntu
```

---

pwd
- 현재 작업하는 디렉터리 출력
```sh
$ pwd
/home/ubuntu
```


---

du
- Disk Usage
- 디스크의 사용 정보를 출력한다.

df
- Disk Free
- 디스크 정보를 출력한다.

```sh
$ du -c
4       ./.git/objects/info
59944   ./.git/objects/pack
59952   ./.git/objects

$ df
Filesystem     1K-blocks    Used Available Use% Mounted on
udev              499480       0    499480   0% /dev
tmpfs             101468    5676     95792   6% /run
/dev/xvda1       8065444 1761212   6287848  22% /
tmpfs             507328       0    507328   0% /dev/shm
tmpfs               5120       0      5120   0% /run/lock
tmpfs             507328       0    507328   0% /sys/fs/cgroup
tmpfs             101468       0    101468   0% /run/user/1000
```

---
## Referensces
---

- 오다 케이지/ 쿠레마츠 타니히토/ 오카다 노리마사/ 히라야마 츠요시 지음, 김완섭 옮김,『그림으로 공부하는 시스템 성능 구조』, 제이펍(1999), p50-p90
- [https://www.lesstif.com/pages/viewpage.action?pageId=20776078](https://www.lesstif.com/pages/viewpage.action?pageId=20776078)