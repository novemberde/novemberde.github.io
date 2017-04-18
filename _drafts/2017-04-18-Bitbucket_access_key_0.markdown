---
title: Bitbucket access key 발급하기
---

## Summary
---------------------
 별도의 프로젝트 멤버를 추가하는 방법도 있지만 access key를 발급하여 git repository에 접근하는 방법도 있다. 외부에 프로젝트를 공개할 경우 이러한 방법을 사용하는 것이 좋을 것이다. member로 추가하면 별도의 그룹을 생성하여 각 프로젝트별로 권한을 주기 까다롭기 때문이다.

---------------------

## Key pair 생성하기
---------------------

먼저 access key를 발급하기 전에 putty-gen으로 public key와 private key를 발급하자.

[PuTTY Key Generator](http://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) 는 이 링크에서 다운받으면 된다.

generate 버튼을 누르고 빈공간에서 열심히 마우스를 움직이면 key pair를 생성할 수 있다.

ssh-rsa부분을 따로 복사해두고 private키는 저장하도록 하자.

![PuTTY Key Generator]({{ site.baseurl }}/images/aws/ec2/puttygen.png)

---

## Access Key 발급하기
---

생성된 ssh-rsa으로 시작하는 부분을 복사해서 잘 두고 있어야 한다.

Bitbucket에서 별도의 access key를 발급하고 싶은 repository 에 들어가서 setting 화면으로 넘어가자.

![access_key]({{ site.baseurl }}/images/bitbucket/access_key.png)


---

add key를 누르면 아래와 같은 화면이 나온다. label에는 구분하고 싶은 이름을 넣어주고, key에는 아까 복사한 ssh-rsa로 시작하는 공개키를 붙여넣어주자.

![add_key]({{ site.baseurl }}/images/bitbucket/add_key.png)

---

## Git 설정하기
---



---
### References
---
- [https://confluence.atlassian.com/bitbucket/set-up-ssh-for-git-728138079.html](https://confluence.atlassian.com/bitbucket/set-up-ssh-for-git-728138079.html)