> [Repository](https://github.com/kimevanjunseok/spring-boot-with-github-action)

## Github Action

Github Action은 Github에서 바로 소프트웨어 Workflow를 자동화할 수 있는 도구이다. 간단하게 말하면 Github에서 CI/CD를 할 수 있다. 자세한 내용은 공식문서를 참고하자.

> Automate, customize, and execute your software development workflows right in your repository with GitHub Actions. You can discover, create, and share actions to perform any job you'd like, including CI/CD, and combine actions in a completely customized workflow. - [Github Action 공식문서](https://docs.github.com/en/actions)

## Github Action 만들기

먼저 Repository 상단의 Action을 클릭하자.

![image](https://user-images.githubusercontent.com/45934117/116655055-571b7200-a9c5-11eb-8222-b0ad8268889b.png)

그럼 다음과 같은 화면이 나온다.

![image](https://user-images.githubusercontent.com/45934117/116655243-baa59f80-a9c5-11eb-9a35-3a383dc1f1ad.png)

Java with Gradle의 Set up this workflow를 클릭한다.

![image](https://user-images.githubusercontent.com/45934117/116655543-41f31300-a9c6-11eb-8904-574c513b345c.png)

Start commit를 누르고 Commit new file을 누르면 yml파일이 생성된다. 생성하는 동시에 Github Action이 동작한다. 

![image](https://user-images.githubusercontent.com/45934117/116659038-eaf03c80-a9cb-11eb-8ed1-8b34f360c973.png)

![image](https://user-images.githubusercontent.com/45934117/116662366-a74c0180-a9d0-11eb-8ff0-9c84cab8210c.png)

생성된 `gradle.yml`파일을 보자.

```yml
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
      uses: actions/setup-java@v2 # 만약 v1이라면 with의 distribution는 생략해도 된다.
      with:
        java-version: '8'
        distribution: 'adopt'
    - name: Grant execute permission for gradlew
      run: chmod +x gradlew
    - name: Build with Gradle
      run: ./gradlew build
```

- [name](https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions#name)
  - Workflow의 이름이다. 위 사진의 좌측을 보면 `All workflows` 밑에 `Java CI with Gradle`의 workflow가 있는 것을 볼 수 있다.
- [on](https://docs.github.com/en/actions/reference/events-that-trigger-workflows)
  - Events that trigger workflows
  - 특정 활동을 할 때 동작시킬 수 있다.
  - 위 코드로 예를 들면 `master` 브렌치에 `push`하거나 `pull_request`할 때 동작한다.
- [jobs](https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions#jobs)
  - 이벤트가 발생하는 경우 jobs가 동작한다.
  - 먼저 [runs-on](https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions#jobsjob_idruns-on)로 The type of machine을 정한다. (예: Windows, Ubuntu, macOS...)
  - 그리고 [steps](https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions#jobsjob_idsteps)이라는 일련의 작업을 가지고 있다. 각 단계별로 동작한다. 위 예시로 JDK, Gradlew 권한 설정 후 빌드한다.
  - `set-java`의 V1과 V2가 궁금하다면 [여기](https://github.com/actions/setup-java)를 보자.

## 참고 자료

[cheese10yun/github-action](https://github.com/cheese10yun/github-action)