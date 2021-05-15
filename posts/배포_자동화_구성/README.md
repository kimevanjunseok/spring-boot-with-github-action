> [Repository](https://github.com/kimevanjunseok/spring-boot-with-github-action)

이제 Jar를 배포하여 실행해보자.

## deploy.sh 생성

`scripts` 디렉토리를 생성 후 안에 `deploy.sh` 파일을 만들어 준다. 위치는 다음 그림과 같다.

![image](https://user-images.githubusercontent.com/45934117/117677422-667e9480-b1e9-11eb-8f88-45afb82e75c7.png)

`deploy.sh` 파일을 작성하자.

```sh
#!/usr/bin/env bash

REPOSITORY=/home/ubuntu/app

echo "> 현재 구동 중인 애플리케이션 pid 확인"

CURRENT_PID=$(pgrep -fl action | grep java | awk '{print $1}')

echo "현재 구동 중인 애플리케이션 pid: $CURRENT_PID"

if [ -z "$CURRENT_PID" ]; then
  echo "현재 구동 중인 애플리케이션이 없으므로 종료하지 않습니다."
else
  echo "> kill -15 $CURRENT_PID"
  kill -15 $CURRENT_PID
  sleep 5
fi

echo "> 새 애플리케이션 배포"

JAR_NAME=$(ls -tr $REPOSITORY/*.jar | tail -n 1)

echo "> JAR NAME: $JAR_NAME"

echo "> $JAR_NAME 에 실행권한 추가"

chmod +x $JAR_NAME

echo "> $JAR_NAME 실행"

nohup java -jar $JAR_NAME > $REPOSITORY/nohup.out 2>&1 &
```

마지막 줄의 명령어를 보자. nohup 실행 시 CodeDeploy는 무한 대기한다. 이를 해결하기 위해 `nohup.out` 파일을 표준 입출력용으로 별도로 사용하는데, 사용하지 않으면 `nohup.out` 파일이 생기지 않고, CodeDeploy 로그에 표준 입출력이 출력된다. nohup이 끝나기 전까지 CodeDeploy도 끝나지 않으니 꼭 해야 한다.

## Github Action 설정 추가

`script`의 파일도 S3에 보내주자. 다음 설정을 추가한다.

```yaml
    # script files 복사
    - name: Copy script
      run: cp ./scripts/*.sh ./deploy
```

## appspec.yml 설정 추가

```yaml
permissions:
  - object: /
    pattern: "**"
    owner: ubuntu
    group: ubuntu

hooks:
  ApplicationStart:
    - location: deploy.sh
      timeout: 60
      runas: ubuntu
```

- `permissions`
  - CodeDeploy에서 EC2 서버로 넘겨준 파일들을 모두 ubuntu 권한을 갖도록 한다.
- `hooks`
  - Codedeploy 배포 단계에서 실행할 명령어를 지정
  - timeout을 60으로 하여 스크립트 실행 시간이 60초가 넘어가면 실패하게 한다. (무한정 기다릴 수 없으니...)
  - ApplicationStart 단계에서 `deploy.sh`를 ubuntu 권한으로 실행하게 한다.

## 배포 확인

정상적으로 배포된 것을 확인할 수 있다.

![image](https://user-images.githubusercontent.com/45934117/117670619-0b49a380-b1e3-11eb-850a-3b62e32efbed.png)

그리고 웹 브라우저에서 EC2 도메인을 입력해서 확인해보자. 

![image](https://user-images.githubusercontent.com/45934117/117670769-261c1800-b1e3-11eb-8e18-7e8d9e11f168.png)

## CodeDeploy 로그 확인

전 포스팅에서도 말했지만 다른 곳에서도 로그를 확인할 방법이 있었다. 여기서 확인하는 것이 보기 편하고 필요한 로그들을 볼 수 있는 것 같다.

Log 위치: `/opt/codedeploy-agent/deployment-root`

디렉토리의 내용을 확인하면 `deployment-logs`라는 디렉토리를 확인할 수 있다. 이곳에서 CodeDeploy 로그를 확인할 수 있다. 물론 `deploy.sh`에서 작성한 `echo`도 확인할 수 있다.

![image](https://user-images.githubusercontent.com/45934117/117804878-45727e00-b293-11eb-9242-a7cb5921f5ff.png)

그리고 영문과 숫자, 대시(-)가 있는 디렉토리명을 볼 수 있다. 그 디렉토리명은 `CodeDeploy ID`이다. 그리고 디렉토리에 들어가서 내용을 확인하면 배포한 단위별로 `배포 ID`로 된 디렉토리명들을 볼 수 있다. 이곳에서는 배포 파일이 정상적으로 왔는지 확인해 볼 수 있다.

## 참고자료

스프링 부트와 AWS로 혼자 구현하는 웹 서비스(이동욱 님)