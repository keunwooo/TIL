# Dockerfile



Docker File이란 Docker Image를 만들기 위한 설정 파일

```
FROM ubuntu:14.04
MAINTAINER alicek106
LABEL "purpose"="practice"
RUN apt-get update
RUN apt-get install apache2 -y
ADD test.html /var/www/html
WORKDIR /var/www/html
RUN ["/bin/bash","-c","echo hello >> test2.html"]
EXPOSE 80
CMD apachectl -DFOREGROUND

```



> **FROM** 
>
> 기반이 되는 이미지 레이어입니다.
>
> 반드시 한번이상 입력해야한다. 이미지의 이름의 포맷은 docker run 명령어에서 이미지 이름을 사용했을 때와 같다. 사용하려는 이미지가 도커에 없다면 자동으로 pull한다.
>
> <이미지 이름>:<태그> 형식으로 작성 
>
> ex) ubuntu:14.04
>
> ```
> FROM openjdk:8-jdk-alpine
> ```





> **MAINTAINER** 
>
> 이미지를 생성한 개발자의 정보를 나타낸다. Dockerfile을 작성한 사람과 연락할 수 있는 이메일 등을 입력한다. 
>
> MAINTATINER는 도커 1.13.0 버전 이후로 사용하지 않는다. 대신 LABEL로 사용이 가능하다.



> **LABEL**
>
> 이미지에 메타데이터를 추가한다. 메타데이터는 키:값 형태로 저장되며, 여러개의 메타데이터가 저장될 수 있다. 
>
> 추가된 메타데이터 정보는 docker inspect 명령어로 이미지의 정보를 구해서 확인 가능하다.
>
> ```
> LABEL maintainer "alicek106 <alicek106@naver.com>"
> ```



> **RUN** 
>
> 이미지를 만들기 위해 컨테이너 내부에서 명령어를 실행한다.
>
> ```
> RUN apt-get update
> RUN apt-get install apache2 -y
> ```
>
> 아파치 웹 서버가 설치된 이미지가 생성된다.
>
> 이미지를 빌드할 때 별도의 입력을 받아야 하는 RUN이 있다면 build 명령어는 이를 오류로 간주하고 빌드를 종료한다.



> **ADD**
>
> 파일을 이미지에 추가한다.
>
> 추가하는 파일은 Dockerfile이 위치한 디렉터리인 Context에서 가져온다. (도커파일이 위치한 디렉터리에서 파일을 가져온다.)
>
> ```
> ADD test.html /var/www/html
> ```
>
> Dockerfile이 위치한 디렉터리에서 test.html 파일을 이미지의 /var/www/html 디렉터리에 추가.



>**WORKDIR**  
>
>명령어를 실행할 디렉터리를 나타낸다.
>
>배시 셀에서 cd 명령어를 입력하는 것과 같은 기능을 한다.
>
>
>
>```
>WORKDIR /var/www/html
>```
>
>후
>
>```
>RUN touch test
>```
>
>를 실행하면 /var/www/html/ 디렉터리에 test파일이 생성된다.



> **EXPOSE** 
>
> 호스트와 연결할 포트 번호입니다.
>
> Dockerfile의 빌드로 생성된 이미지에서 노출할 포트를 설정한다.
>
> EXPOSE를 설정한 이미지로 컨테이너를 생성했다고 해서 반드시 이 포트가 호스트의 포트와 바인딩되는 것이 아니라 단지 80번 포트를 사용할 것을 나타내는 것뿐임.



> **CMD**
>
> 컨테이너가 시작되었을 때 실행할 명령어(커맨드)입니다. 
>
> 해당 명령어는 DockerFile내 1회만 쓸 수 있습니다.
>
> ex) 
>
> ```
> CMD apachectl -DFOREGROUND
> ```
>
> 컨테이너를 생성할 때 별도의 커맨드를 입력하지 않아도 이미지에 내장된 커맨드가 적용되어 컨테이너가 시작될때 자동으로 아파치 웹 서버가 작동한다.
>
> 아파치 웹서버는 하나의 터미널을 차지하는 포그라운드 모드로 실행되기 때문에 -d 옵션을 사용하여 detached 모드로 컨테이너를 생성한다.



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



> **VOLUME** 
>
> VOLUME은 디렉터리의 내용을 컨테이너에 저장하지 않고 호스트에 저장하도록 설정합니다. 
>
> 데이터 볼륨을 호스트의 특정 디렉터리와 연결하려면 docker run 명령에서 -v 옵션을 사용해야 합니다. 
>
> ex) -v /root/data:/data



# Dockerfile 빌드

```
docker build -t mybuild:0.0 ./
```

> -t 옵션은 생성될 이미지의 이름
>
> -t 옵션을 지정하지 않으면 16진수의 이름으로 이미지가 생성된다. 가급적 -t 옵션 사용
>
> build 명령어 끝에는 Dockerfile이 저장된 경로를 입력한다. (./ 현재디렉터리)



```
docker run -d -P --name myserver mybuild:0.0
```

> -P 옵션은 이미지에 설정된 EXPOSE의 모든 호스트에 연결하도록 설정한다.
>
> EXPOSE로 노출된 포트를 호스트에서 사용 가능한 포트에 차례로 연결하므로 이 컨테이너가 호스트의 어떤 포트와 연결됐는지 확인할 필요가 있다.



```
docker ps
docker port [컨테이너이름]
```

