---
title: Bitbucket access key 발급하기
---

## Summary
---------------------
 서버에 프로젝트를 배포할 경우 ftp로 파일을 옮기는 방식으로 사용할 수도 있다.
 하지만 git repository를 사용하면 git 명령어만으로 간단히 서버코드를 갱신할 수 있다.

 로그인이 아닌 access key를 통해 git repository에 접근하는 방법에 대해 알아보자.

---------------------

## Key pair 생성하기
---------------------

먼저 배포하고 싶은 서버에 ssh로 접속한 후 key pair를 생성하자.


```bash
# 이 커맨드는 현재 접속한 유저 directory에 .ssh폴더가 없으면 생성할 것이고, 있다면 .ssh directory에 key pair만 생성할 것이다.
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/USERNAME/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/USERNAME/.ssh/id_rsa.
Your public key has been saved in /home/USERNAME/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:L68PHdXfH2yfSUKraajtNLnm9ltO2HXtzk0y8Z76XUI USERNAME@localhost
The key\'s randomart image is:
+---[RSA 2048]----+
|                 |
|      . .    .   |
|     . =    ...  |
|      + + ... o.o|
|       =So.  +E*+|
|      o  o+.o.=+*|
|       oo.o=  +oB|
|      . .++    O=|
|        .=+. .oo*|
+----[SHA256]-----+

# ~/.ssh의 file을 확인해보자.
$ ls -a ~/.ssh
.  ..  authorized_keys  id_rsa  id_rsa.pub

# 파일을 확인했다면 권한설정을 해주자.
$ chmod 700 .ssh && chmod 600 .ssh/id_rsa && chmod 600 .ssh/id_rsa.pub
```

개인키와 공개키를 발급하였다. 이제 ssh-agent를 시작하고 keys를 load하자.

```bash
# ssh-agent가 실행중인지 확인해보자
$ ps -e | grep [s]sh-agent 
 9060 ?? 0:00.28 /usr/bin/ssh-agent -l

# 만일 아무 반응이 없거나 실행 중이지 않다면 아래 명령어를 실행하자.
$ ssh-agent /bin/bash

# 새로운 키를 load  하자.
$ ssh-add ~/.ssh/id_rsa 
Enter passphrase for /home/USERNAME/.ssh/id_rsa:
Identity added: /home/USERNAME/.ssh/id_rsa (/Users/emmpa1/.ssh/id_rsa)

# ssh-agent가 관리할 수 있도록 리스트에 등록해주자.
$ ssh-add -l
2048 7a:9c:b2:9c:8e:4e:f4:af:de:70:77:b9:52:fd:44:97 /home/USERNAME/.ssh/id_rsa (RSA)

```

다음의 명령어로 공개키를 따로 복사해두자. bitbucket에서 사용할 것이다.
```bash
# 이 명령어 뒤에 나온 ssh-rsa로 시작하는 부분을 전부 복사하자.
$ cat ~/.ssh/id_rsa.pub
ssh-rsa ~~~
~~~
USERNAME@ip-172-31-14-07
```

---

## Access Key 발급하기
---

Bitbucket에서 별도의 access key를 발급하고 싶은 repository 에 들어가서 setting 화면으로 넘어가자.

![access_key]({{ site.baseurl }}/images/bitbucket/access_key.png)


---

add key를 누르면 아래와 같은 화면이 나온다. label에는 구분하고 싶은 이름을 넣어주고, key에는 아까 복사한 ssh-rsa로 시작하는 공개키를 붙여넣어주자.

![add_key]({{ site.baseurl }}/images/bitbucket/add_key.png)

---

다시 ssh 접속화면으로 돌아와서 가져오고 싶은 repository를 clone하자.

주의할 점은 https가 아닌 ssh를 선택한 값으로 clone해야 한다.

```
$ git clone git@bitbucket.org:KH_Own_Team/test.git
```
---
### References
---
- [https://confluence.atlassian.com/bitbucket/set-up-ssh-for-git-728138079.html](https://confluence.atlassian.com/bitbucket/set-up-ssh-for-git-728138079.html)