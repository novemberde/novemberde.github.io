---
title: CodeCommit 과 SourceTree 연동
tags: [aws, CodeCommit, code, commit, sourcetree, 연동]
date: "2017-10-20T11:30:03+00:00"
aliases: ["/aws/2017/10/20/CodeCommit.html"]
ShowBreadCrumbs: true
ShowReadingTime: true
ShowPostNavLinks: true
---
## Summary
---
AWS CodeCommit은 AWS 완전 관리형 private git repository로 인데, 콘솔화면이 조금 부실해서
콘솔화면에서 repository 관리가 좋아보이진 않았다.

SourceTree라는 Atlassian의 Git GUI tool로 AWS CodeCommit을 연동해보자.

---
## AWS CodeCommit에서 Git repository 생성하기
---

AWS CodeCommit화면으로 들어가서 Create repository를 하면 아래와 같은 화면이 나오는데 각 항목에 내용을 입력하자.

- Repository name: test
- Description: CodeCommit with SourceTree

![create repository](/images/aws/codecommit/create.png)

---
## Local에 Git repository 설정하기
---

Repository를 생성하면 다음과 같은 화면이 나타난다.

![created](/images/aws/codecommit/created.png)

주목해야 할 부분은 빨간 박스로 표시해놓았다.

현재 root계정으로 작업하고 있는데 이는 권장하지 않고,
별도의 IAM user를 통해 접근을 하는 것을 권장한다는 내용이다.

또한 생성한 유저로 git clone을 하기 위해서는 git config에 credential.helper를 설정한다.

이는 한번 설정한 후에는 반복적으로 자격증명을 할 필요없이 CodeCommit에 반영할 수 있도록 해준다.

먼저 로컬의 Terminal을 열어 아래와 같이 코드를 입력해주자.

```
$ git config --global credential.helper "!aws codecommit credential-helper $@"
$ git config --global credential.UseHttpPath true
```

그 다음 Local에서 CodeCommit과 연동할 IAM user를 생성한다.

만약 이미 AWS cli에 configure가 되어 있다면 해당 유저에게 
AWSCodeCommitPowerUser라는 AWS managed type의 policy를 추가한다.

그렇지 않다면 IAM user를 생성한다. Security, Identity & Compliance 의 IAM 으로 들어가서 Users라는 탭을 클릭한다.
Add user 버튼을 클릭하여 유저를 생성한다. 내용은 아래와 같다.

- User name: CodeCommitUser
- Access type: Programmatic access

![add user](/images/aws/iam/adduser.png)


---

Attach existing policies directly를 선택하여 AWSCodeCommitPowerUser를 선택한다.
이는 CodeCommit에 대해 삭제빼고 모든 권한을 준다.

![codecommit](/images/aws/iam/codecommit.png)


---

생성이 되었다면 아래와 같은 창이 나온다. Download를 해도 되지만 기록되면 좋을게 없으니 이창을 그대로 두고 AWS command line interface를 설치한다.

![created user](/images/aws/iam/created.png)

Local computer에 [aws-cli](http://docs.aws.amazon.com/ko_kr/cli/latest/userguide/installing.html)를 링크된 곳으로 가서 설치한다.

설치 후에 켜둔 창에서 Access key ID와 Secret access key를 복사해서 입력한다. Default region은 ap-northeast-2로 한다.

```
$ aws configure
AWS Access Key ID [****************AAAA]:
AWS Secret Access Key [****************AAAA]:
Default region name [ap-northeast-2]:
```

---

다시 IAM user에서 방금생성한 user를 선택한다.

다음의 화면에서 Generate 버튼을 눌러 AWS CodeCommit에서 사용할 Git credentials을 생성한다.

![git-credentials](/images/aws/iam/git-credentials.png)


---

HTTPS Git credential이 생성이 되면 아래와 같은 화면이 나타난다.

![generated](/images/aws/iam/generated.png)


---

다시 Local로 돌아와서 아래와 repository를 clone하면 Username과 Password를 입력하는데 방금 생성한 값을 넣어주면 된다.

```
$ git clone https://git-codecommit.us-east-1.amazonaws.com/v1/repos/test
Username for 'https://git-codecommit.us-east-1.amazonaws.com/v1/repos/test': 
Password for 'https://home-admin-at-739806549786@git-codecommit.us-east-1.amazonaws.com/v1/repos/test'
```

Clone한 repository로 들어가서 변경사항을 반영해본다.

```
$ > sample.txt
$ git add .
$ git commit -m "add sameple.txt"
$ git push origin master
```

![updated](/images/aws/codecommit/updated.png)


---
## SourceTree에서 Git repository 확인하기
---

SourceTree에서 repository를 add한 후에 사용하면 Local의 GUI환경에서 CodeCommit을 사용할 수 있다.

처음 Push를 하면 아래와 같이 계정정보를 등록하는 화면이 나타난다.

이전과 같이 IAM User의 HTTPS Git credential 값을 입력하면 된다.

![credentials](/images/aws/codecommit/credentials.png)

---

나중에 여러 해당 CodeCommit에 대한 계정 정보를 삭제하고 싶은 경우 [도구>옵션]에서 삭제할 수 있다.

![sourcetreedelete](/images/aws/codecommit/sourcetreedelete.png)

---
### References
---
- [http://docs.aws.amazon.com/ko_kr/codecommit/latest/userguide/welcome.html](http://docs.aws.amazon.com/ko_kr/codecommit/latest/userguide/welcome.html)