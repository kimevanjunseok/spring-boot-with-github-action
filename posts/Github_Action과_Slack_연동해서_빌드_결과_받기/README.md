> [Repository](https://github.com/kimevanjunseok/spring-boot-with-github-action)

이제 빌드 결과를 Slack으로 받아보자.

## Action-Slack

[action-slack](https://github.com/marketplace/actions/action-slack)을 사용해서 빌드 결과를 받아올 수 있다.

> You can notify slack of GitHub Actions.

한 번 사용해보자. 먼저 [여기](https://api.slack.com/apps)에 들어가서 `Create New App`를 누르고, 본인이 원하는 `App Name`과 `Development Slack Workspace`를 선택해서 만들어준다. 그럼 다음과 같은 화면이 나온다.

![image](https://user-images.githubusercontent.com/45934117/116698303-2eb06980-a9ff-11eb-8731-1f65a8111d69.png)

만약 길을 잃었다면 우측에 `Settings`에 `Basic Information`을 클릭하고 `Add features and functionality`를 클릭하면 다음과 같이 나온다.

먼저 `Bots`를 클릭해서 `App Display Name`을 만들자. (안 만드니까 `Add New Webhook to Workspace`를 눌렀을 때 만들라고 나옴...)

이제 `Incoming WebHooks`를 누르고 활성화를 시키자. 그리고 `Add New Webhook to Workspace`를 클릭하면 다음 화면이 나올 것이다.

![image](https://user-images.githubusercontent.com/45934117/116701172-8ef4da80-aa02-11eb-84b8-ccac50925f24.png)

`Allow`를 클릭하면 `Webhook URL`에 URL이 생긴다. 이제 URL을 `Copy`해서 Github으로 가자.

![image](https://user-images.githubusercontent.com/45934117/116702321-e8a9d480-aa03-11eb-9112-5c2bb1ab8048.png)

`Settings` → `Secrets`에 가서 `New repository secret`을 누른다. 

![image](https://user-images.githubusercontent.com/45934117/116702509-227adb00-aa04-11eb-8886-8c0af43cb2e3.png)

`Name`에 `SLACK_WEBHOOK_URL`을 넣고, `Value`에 아까 `Copy`한 URL을 넣으면 된다.

이제 `yml`파일을 작성해보자.

```yaml
name: Java CI with Gradle

on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
    - name: Set up JDK 8
      uses: actions/setup-java@v2
      with:
        java-version: '8'
        distribution: 'adopt'
    - name: Grant execute permission for gradlew
      run: chmod +x gradlew
    - name: Build with Gradle
      run: ./gradlew build

    - name: action-slack
      uses: 8398a7/action-slack@v3
      with:
        status: ${{ job.status }}
        author_name: Github Action Test # default: 8398a7@action-slack
        fields: repo,message,commit,author,action,eventName,ref,workflow,job,took
      env:
        SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }} # required
      if: always() # Pick up events even if the job fails or is canceled.
```

이제 기본 세팅이 끝났다. 한 번 확인해보자.

![image](https://user-images.githubusercontent.com/45934117/116703260-09265e80-aa05-11eb-994c-cbecaea89180.png)

잘 동작하는 것을 확인할 수 있다. `author_name`과 `fields` 설정은 위 그림으로 어떤 설정인지 유추할 수 있으니... 설명은 생략한다.

## 참고자료

[action-slack](https://github.com/marketplace/actions/action-slack)

[노아론블로그](https://blog.aaronroh.org/112)