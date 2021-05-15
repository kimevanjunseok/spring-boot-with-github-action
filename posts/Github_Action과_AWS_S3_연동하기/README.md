> [Repository](https://github.com/kimevanjunseok/spring-boot-with-github-action)

이번에는 Github Action과 AWS S3를 연동하는 방법을 알아보자. 구조는 다음과 같다.

![image](https://user-images.githubusercontent.com/45934117/118360320-078e9600-b5c2-11eb-88bb-fc58ade638fa.png)

## EC2 생성

EC2는 다음 스펙으로 생성했다.

- Ubuntu Server 18.04
- t2.micro

## IAM 설정

> AWS 리소스에 대한 액세스를 안전하게 제어할 수 있는 웹 서비스입니다. IAM을 사용하여 리소스를 사용하도록 인증(로그인) 및 권한 부여(권한 있음)된 대상을 제어합니다. - AWS

먼저 IAM을 검색하면 다음과 같은 화면으로 올 수 있을 것이다. 사용자로 들어와서 `사용자 추가`를 해보자.

![image](https://user-images.githubusercontent.com/45934117/116873170-2ef47300-ac52-11eb-8dd3-293caeb1687c.png)

먼저 사용자 이름을 적고, 엑세스 유형은 `프로그래밍 방식 엑세스`를 한다.

![image](https://user-images.githubusercontent.com/45934117/116873213-416eac80-ac52-11eb-9ed9-82dc23232aeb.png)

권한 설정은 `기존 정책 직접 연결`을 선택한다. 그러면 정책 검색을 할 수 있는 화면에서 다음과 같이 검색해서 선택한다.

![image](https://user-images.githubusercontent.com/45934117/116873425-a1655300-ac52-11eb-9814-e521db9db418.png)

다음 태그 추가는 본인이 인지 가능한 정도의 이름으로 만들자.

![image](https://user-images.githubusercontent.com/45934117/116873517-c8bc2000-ac52-11eb-9f4d-0b83e3f353b1.png)

이제 내가 생성할 권한을 최종적으로 확인할 수 있다.

![image](https://user-images.githubusercontent.com/45934117/116873551-da9dc300-ac52-11eb-9f36-54ad38315e56.png)

최종 생성 완료를 하면 `엑세스 키 ID`와 `비밀 엑세스 키`가 생성된다. 이 키들을 가지고 `Github Action`과 연동할 것이다.

![image](https://user-images.githubusercontent.com/45934117/116873645-ff923600-ac52-11eb-84c7-d44e14d5316f.png)

## Github Action Key 등록

해당 `Repository`의 `Settings` → `Secrets`에서 만들어주면 된다. 생성한 `엑세스 키 ID`와 `비밀 엑세스 키`를 저장하자.

![image](https://user-images.githubusercontent.com/45934117/116873965-7f200500-ac53-11eb-8011-bb2eebb118d8.png)

## S3 버킷 설정

> AWS의 S2 서비스는 일종의 파일 서버입니다. 순수하게 파일을 저장하고 접근 권한을 관리, 검색 등을 지원하는 파일 서버의 역할을 합니다. - 스프링 부트와 AWS로 혼자 구현하는 웹 서비스(이동욱 님)

Github Action에서 생성된 파일을 저장하도록 구성할 것이다. 이번엔 `S3`를 검색해서 이동하고, 버킷을 만들어보자.
![image](https://user-images.githubusercontent.com/45934117/116874140-c7d7be00-ac53-11eb-863b-2948137d5b25.png)

원하는 버킷 이름을 만들어준다.

![image](https://user-images.githubusercontent.com/45934117/116874221-ed64c780-ac53-11eb-8e93-2ce1fda505dc.png)

그다음으로 권한 설정에서 `모든 퍼블릭 엑세스 차단`을 선택한다. 왜 그럴까?

> 실제 서비스에서 할 때는 Jar 파일이 퍼블릭일 경우 누구나 내려받을 수 있어 코드나 설정값, 주요 키값들이 다 탈취될 수 있습니다. - 스프링 부트와 AWS로 혼자 구현하는 웹 서비스(이동욱 님)

![image](https://user-images.githubusercontent.com/45934117/116874307-12593a80-ac54-11eb-9801-c54183b28543.png)

이제 버킷을 만들어 주면 된다.

## Github yml 파일 설정

```yaml
    # 디렉토리 생성
    - name: Make Directory
      run: mkdir -p deploy

    # Jar 파일 복사
    - name: Copy Jar
      run: cp ./build/libs/*.jar ./deploy

    # 파일 압축
    - name: Make zip file
      run: zip -r ./springboor-with-githubaction.zip ./deploy

    # Deploy
    - name: Deploy
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_KEY }}
        aws-region: ap-northeast-2

    - name: Upload to S3
      run: aws s3 cp --region ap-northeast-2 --acl private ./springboor-with-githubaction.zip s3://springboot-with-githubaction/

```

순서는 이러하다.

- 디렉토리 생성: 먼저 폴더를 만든다. 이 폴더에는 `build` 후 생성된 `jar 파일`을 넣을 것이다.
- Jar 파일 복사: `jar 파일`을 복사해서 `deploy` 폴더에 담는다.
- 파일 압축: [CodeDeploy](https://docs.aws.amazon.com/ko_kr/codedeploy/latest/userguide/welcome.html)는 Jar 파일을 인식 못 하기 때문에 압축한다.
- Deploy: S3로 파일 업로드 혹은 CodeDeploy로 배포 등 외부 서비스와 연동될 행위들을 선언한다. 적성 방법은 [여기](https://github.com/aws-actions/amazon-ecs-deploy-task-definition)와 [여기](https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/cli-services-s3-commands.html#using-s3-commands-managing-objects-copy)를 참고했다.

완성했다면 Github에 Push 해보자.

![image](https://user-images.githubusercontent.com/45934117/116884684-f9f01c80-ac61-11eb-8d77-5e58a4211bd4.png)

만약 빌드에 성공했다면 S3 버킷에 업로드가 된 것을 확인할 수 있다.

![image](https://user-images.githubusercontent.com/45934117/116884789-1c823580-ac62-11eb-9dda-496ab1b48285.png)

## 참고자료

[Github Action을 사용한 Spring boot & gradle CI/CD 구축 - 2](https://stalker5217.github.io/devops/github_action_ci_cd_2/)

스프링 부트와 AWS로 혼자 구현하는 웹 서비스(이동욱 님)