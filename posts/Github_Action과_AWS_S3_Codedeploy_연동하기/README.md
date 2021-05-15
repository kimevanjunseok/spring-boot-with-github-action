> [Repository](https://github.com/kimevanjunseok/spring-boot-with-github-action)

이번에는 Github Action과 AWS S3, CodeDeploy를 연동해보자.

## EC2에 IAM 역할 추가

먼저 IAM에 들어가서 좌측 `역할`을 클릭한다. 그럼 다음과 같은 화면이 나오는데 `역할 만들기`를 클릭한다.

![image](https://user-images.githubusercontent.com/45934117/117002645-c9ba8380-ad1e-11eb-8f0f-18f9abdceb83.png)

`AWS 서비스`에 `EC2`를 클릭한다.

![image](https://user-images.githubusercontent.com/45934117/117002744-e8207f00-ad1e-11eb-9782-a47261b4d847.png)

권한 설정은 `AmazonEC2RoleforAWSCodeDeploy`를 선택한다.

![image](https://user-images.githubusercontent.com/45934117/117002839-0ab29800-ad1f-11eb-9343-5b50a05096e4.png)

태그는 원하는 이름으로 짓는다.

![image](https://user-images.githubusercontent.com/45934117/117002913-21f18580-ad1f-11eb-9329-5e262cde5031.png)

최종적으로 확인 후 만들어준다.

![image](https://user-images.githubusercontent.com/45934117/117002974-3d5c9080-ad1f-11eb-89ba-6a12fd465ed9.png)

이제 만든 역할을 EC2에 등록해보자. 인스턴스에 들어가 해당 인스턴스에 마우스 오른쪽 버튼을 클릭해서 `보안` → `IAM 역할 수정`을 클릭한다. (Windows 기준)

![image](https://user-images.githubusercontent.com/45934117/117003861-5a459380-ad20-11eb-88ae-12fc2db4d01a.png)

방금 생성한 역할을 선택하고 저장한다.

![image](https://user-images.githubusercontent.com/45934117/117004035-90831300-ad20-11eb-81d9-67bffe4a00b4.png)

그리고 `인스턴스 재부팅`을 한다. 재부팅을 해야 역할이 정상적으로 동작한다고 한다.

![image](https://user-images.githubusercontent.com/45934117/117004163-bb6d6700-ad20-11eb-8a16-dc183db6144f.png)

## EC2 서버 Setting 및 CodeDeploy agent 설치

### 자바 설치 및 타임존 변경

EC2 서버에 접속한다. 처음 접속하는 것이기에 기본 Setting부터 한다.

먼저 자바8를 설치한다.

- `sudo apt install openjdk-8-jdk`

다음은 타임존을 변경할 것이다. 기본 시간은 UTC로 되어있다. 다음 명령어를 실행한다.

- `sudo rm /etc/localtime`
- `sudo ln -s /usr/share/zoneinfo/Asia/Seoul /etc/localtime`

만약 정상적으로 수행되었다면 `date` 명령어를 입력하면 타임존이 `KST`로 변경되있을 것이다.

### CodeDeploy agent 설치

이제 CodeDeploy agent를 설치해보자. [이곳](https://docs.aws.amazon.com/codedeploy/latest/userguide/codedeploy-agent-operations-install-ubuntu.html)을 참고했다.

다음 3개의 명령어를 입력한다. Ubuntu Server 버전에 따라 다르다. (아래 명령어는 16.04 이상 기준) 

- `sudo apt update`

- `sudo apt install ruby-full`

- `sudo apt install wget`

다음 명령어를 입력한다.

- `cd /home/ubuntu`

다음 명령어는 `wget https://bucket-name.s3.region-identifier.amazonaws.com/latest/install`이다. `bucket-name`과 `region-identifier`는 [여기](https://docs.aws.amazon.com/ko_kr/codedeploy/latest/userguide/resource-kit.html#resource-kit-bucket-names)를 참고했다. 본인이 Amazon S3에 만든 버킷 기준이다.

- `wget https://aws-codedeploy-ap-northeast-2.s3.ap-northeast-2.amazonaws.com/latest/install`

install 파일에 해당 권한이 없으니 추가한다.

- `chmod +x ./install`

이제 CodeDeploy agent를 설치하자. (Ubuntu 14.04, 16.04, 18.04 기준)

- `sudo ./install auto`

정상적으로 실행되고 있는지 다음 명령어로 확인할 수 있다. 동작하고 있다면 `running`이란 단어를 볼 수 있다.

- `sudo service codedeploy-agent status`

## CodeDeploy를 위한 권한 생성

우선 `IAM`으로 가서 역할을 만든다. 그리고 `AWS 서비스` → `CodeDeploy`를 선택하면 아래 `사용 사례 선택`이 나온다. 그림처럼 `CodeDeploy`를 클릭한다.

![image](https://user-images.githubusercontent.com/45934117/117011437-ce843500-ad28-11eb-8260-c8117fddb96c.png)

CodeDeploy는 권한이 하나이다. 선택 없이 넘어가자.

![image](https://user-images.githubusercontent.com/45934117/117011509-de037e00-ad28-11eb-9e59-d7ab1ff9fdef.png)

태그는 매번 똑같다. 원하는 아름으로 하자.

![image](https://user-images.githubusercontent.com/45934117/117011577-eb206d00-ad28-11eb-988b-a0c74cac813c.png)

이제 역할 이름을 만들고 역할을 만들어주자.

![image](https://user-images.githubusercontent.com/45934117/117011637-fc697980-ad28-11eb-995b-4585abb002c2.png)

## CodeDeploy 생성

`CodeDeploy` → `애플리케이션`에 들어가서 `애플리케이션 생성`을 하자.

![image](https://user-images.githubusercontent.com/45934117/117009673-f8d4f300-ad26-11eb-9524-524b47e5236c.png)

애플리케이션 구성은 다음과 같이 한다.

![image](https://user-images.githubusercontent.com/45934117/117009775-130ed100-ad27-11eb-824f-e30aea876ef8.png)

애플리케이션을 생성하면 배포 그룹을 생성하라고 할 것이다. 한 번 생성해보자. 

![image](https://user-images.githubusercontent.com/45934117/117009850-2457dd80-ad27-11eb-95ee-950db1a85b73.png)

`배포 그룹 이름`과 `서비스 역할`을 입력한다. `서비스 역할`은 위의 `CodeDeploy를 위한 권한 생성`에서 만든 `codedeploy-role`을 선택한다. (역할 이름이 다르면 다를 수 있다.)

![image](https://user-images.githubusercontent.com/45934117/117011829-3aff3400-ad29-11eb-9252-82478c3070fb.png)

배포 유형은 `현재 위치`이다. (만약 배포할 서비스가 2대 이상이면 `블루/그린` 선택)

![image](https://user-images.githubusercontent.com/45934117/117010093-6a14a600-ad27-11eb-87bf-0d4943a176d1.png)

환경 구성은 `Amazon EC2 인스턴스`를 선택한다.

![image](https://user-images.githubusercontent.com/45934117/117010162-7b5db280-ad27-11eb-9e31-754580474624.png)

마지막으로 배포 구성은 다음 그림과 같이 선택하고 로드 밸런서는 체크하지 않는다. (배포 구성은 한 번 배포할 때 몇 대의 서버에 배포할지를 결정한다.)

![image](https://user-images.githubusercontent.com/45934117/117010289-9f20f880-ad27-11eb-8faf-07660133be2a.png)

## Github Action, S3, CodeDeploy 연동

### appspec.yml 파일

EC2 서버에 폴더(`app`) 하나 만들어 준다.

- `/home/ubuntu/app` 

`appspec.yml` 파일을 만들어 준다. 위치는 다음 그림과 같다.

![image](https://user-images.githubusercontent.com/45934117/117165999-f3e47200-ae00-11eb-8076-a628846c572a.png)

`appspec.yml` 파일을 작성하자.

```yaml

version: 0.0
os: linux
files:
  - source: /
    destination: /home/ubuntu/app/
    overwrite: yes
```

- `source`
  - CodeDeploy에 전달해 준 파일 중 destination에 이동시킬 대상 지정
  - `/`를 지정하면 전체 파일
- `destination`
  - source에서 보낸 파일을 받을 위치
- `overwrite`
  - 기존 파일이 있다면 덮어쓸지 말지 결정

### Github Action 설정

```yaml
    # appspec.yml 파일 복사
    - name: Copy appspec.yml
      run: cp appspec.yml ./deploy

	# ...
	
    - name: CodeDeploy
      run: aws deploy create-deployment --application-name springboot-with-githubaction --deployment-group-name springboot-with-githubaction-group --s3-location bucket=springboot-with-githubaction,key=springboor-with-githubaction.zip,bundleType=zip
```

- `appspec.yml`도 S3에 보내야 하기 때문에 `./deploy` 위치에 복사해준다.
- 다음은 `Amazon S3`로부터 `deploy`하기 위해 작성된 명령어다. [여기](https://docs.aws.amazon.com/codedeploy/latest/userguide/application-revisions-push.html)를 참고했다.
  - `--application-name`: CodeDeploy에서 만든 애플리케이션 이름이다.
  - `--deployment-group-name`: 애플리케이션 만든 후 생성한 배포 그룹 이름이다.
  - `--s3-location`: `Amazon S3`에 업로드 된 정보를 입력
    - 현재 정보
    - `bucket`: springboot-with-githubaction
    - `key`: springboor-with-githubaction.zip
    - `bundleType`: zip

## 배포 확인

정상적으로 동작했다면 `CodeDeploy` → `배포`에서 `성공`을 확인할 수 있다.

![image](https://user-images.githubusercontent.com/45934117/117027806-03e44f00-ad38-11eb-8bf8-b209fd8a26a1.png)

## 하면서 알게된 것

처음에 `CodeDeploy`에 실패했다. "왜 실패했을까?" 생각했지만, 아래 문구만 나오고 이유를 정확히 알 수 없었다.

![image](https://user-images.githubusercontent.com/45934117/117173258-8ee04a80-ae07-11eb-829e-bb4b75103c9d.png)

그러나 EC2 서버에서 `CodeDeploy Agent Log`를 확인할 수 있다고 한다. Log 위치는 `/var/log/aws/codedeploy-agent/`이다. 여기서 Log를 확인할 수 있다. ~~(Github Action yml 파일에 오타가 있었다.)~~

## 참고자료

[Github Action을 사용한 Spring boot & gradle CI/CD 구축 - 3](https://stalker5217.github.io/devops/github_action_ci_cd_3/)

스프링 부트와 AWS로 혼자 구현하는 웹 서비스(이동욱 님)

