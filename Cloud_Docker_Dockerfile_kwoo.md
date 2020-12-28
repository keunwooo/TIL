# Dockerfile



Docker File이란 Docker Image를 만들기 위한 설정 파일



> **FROM** 
>
> 기반이 되는 이미지 레이어입니다.
>
> <이미지 이름>:<태그> 형식으로 작성 
>
> ex) ubuntu:14.04
>
> ```
> FROM openjdk:8-jdk-alpine
> ```



> **COPY**
>
> 파일이나 폴더를 이미지에 복사한다. 
>
> ```
> COPY hello.jar hello.jar
> ```



> **ENV**
>
> 이미지에서 사용할 환경 변수 값을 지정한다.
>
> ```
> ENV PROFILE=local
> ```



> **ENTRYPOINT**
>
> 컨테이너를 구동할 때 실행할 명령어를 지정한다. 
>
> ```
> ENTRYPOINT ["./run.sh"]
> ```



> **MAINTAINER** 
>
> 메인테이너 정보입니다.



> **RUN** 
>
> 도커 이미지가 생성되기 전에 수행할 쉘 명령어



> **VOLUME** 
>
> VOLUME은 디렉터리의 내용을 컨테이너에 저장하지 않고 호스트에 저장하도록 설정합니다. 
>
> 데이터 볼륨을 호스트의 특정 디렉터리와 연결하려면 docker run 명령에서 -v 옵션을 사용해야 합니다. 
>
> ex) -v /root/data:/data



> **CMD**
>
> 컨테이너가 시작되었을 때 실행할 실행 파일 또는 셸 스크립트입니다. 
>
> 해당 명령어는 DockerFile내 1회만 쓸 수 있습니다.



> **WORKDIR**  
>
> CMD에서 설정한 실행 파일이 실행될 디렉터리입니다.



> **EXPOSE** 
>
> 호스트와 연결할 포트 번호입니다.