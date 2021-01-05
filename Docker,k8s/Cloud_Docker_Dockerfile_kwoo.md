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
> 로컬 디렉터리에서 읽어 들인 컨텍스트로부터 이미지에 파일을 복사하는 역할을 한다. 사용하는 형식은 ADD와 동일하다.
>
> COPY는 로컬의 파일만 이미지에 추가가 가능하지만 ADD는 외부 URL 및 tar 파일에서도 파일을 추가할 수 있다.
>
> COPY의 기능이 ADD에 포함되어 있다.
>
> 파일이나 폴더를 이미지에 복사한다. 
>
> 
>
> ```
> COPY hello.jar hello.jar
> COPY test.html /home/
> COPY ["test.html","/home/"]
> ```

>```
>ADD https://raw.githubusercontent.com/alicek106/mydockerrepo/master/test.html /home
>```
>
>ADD에 추가할 파일을 깃과 같은 외부 URL로 지정이 가능하다.
>
>```
>ADD test.tar /home
>```
>
>tar 파일을 추가하는 것이 가능하다. tar 파일은 자동으로 해제되서 추가되어진다.
>
>**ADD는 잘 권장하지 않음 ADD로 추가하면 정확하게 어떤 파일이 추가되는지 모르기 때문**
>
> 





> **ENV**
>
> 이미지에서 사용할 환경 변수 값을 지정한다.
>
> 설정한 환경변수는 ${ENV_NAME} 또는 $ENV_NAME의 형태로 사용 가능
>
> ```
> FROM ubuntu:14.04
> ENV test /home
> WORKDIR $test
> RUN touch $test/mytouchfile
> ```
>
> 이미지 빌드 후 컨테이너 생성한 뒤 환경변수를 확인하면 /home 값이 적용된 것을 확인이 가능.
>
> run 명령어에서 -e 옵션을 사용해 같은 이름의 환경변수를 사용하면 기존의 값은 덮어 쓰여진다.
>
> ```
> docker build -t myenv:0.0
> 
> docker run -i -t --name env_test myenv:0.0 /bin/bash
> 
> 배시쉘 접속 후
> echo $test 로 /home 의 결과 확인
> ```
>
> ```
> docker run -i -t --name env_test_override \-e test=myvalue \
> myenv:0.0 /bin/bash
> 
> 배시쉘 접속 후 
> echo $test 로 myvalue 의 결과 확인
> ```
>
> 
>
> ```
> ENV PROFILE=local
> ```





> **VOLUME** 
>
> 빌드된 이미지로 컨테이너를 생성했을 때 호스트와 공유할 컨테이너 내부의 디렉터리를 설정합니다.
>
> VOLUME ["/home/dir", "home/dir2"] 처럼 JSON 배열 형식으로 여러 개를 사용하거나 VOLUME /home/dir /home/dir2로도 사용이 가능
>
> 
>
> 컨테이너 내부의 /home/volume 디렉터리를 호스트와 공유하도록 설정
>
> ```
> FROM ubuntu:14.04
> RUN mkdir /home/volume
> RUN echo test >> /home/volume/testfile
> VOLUME /home/volume
> ```
>
> ```
> docker build -t myvolume:0.0 .
> docker run -i -t -d --name volume_teset myvolume:0.0
> 
> docker volume ls
> ```
>
> 



> **ARG**
>
> build 명령어를 실행할 때 추가로 입력을 받아 Dockerfile 내에서 사용 될 변수의 값을 설정한다.
>
> ARG의 값은 기본적으로 build 명령어에서 입력받아야 하지만 다음과 같이 직접 값을 지정할 수 있다.
>
> ```
> FROM ubuntu:14.04
> ARG my_arg
> ARG my_arg_2=value2
> RUN touch ${my_arg}/mytouch
> ```
>
> 
>
> ```
> docker build --build-arg my_arg=/home -t myarg:0.0 ./
> ```
>
> ENV 와 ARG의 값을 사용하는 방법은 같기 ${} 로 같기 때문에 ARG로 설정한 변수를 ENV에서 같은 이름으로 다시 정의하면 --build-arg 옵션에서 설정하는 값은 ENV에 의해 덮여쓰여진다.



> **ENTRYPOINT**
>
> 컨테이너가 시작될 때 수행할 명령을 지정한다는 점에서 CMD와 비슷하지만,ENTRYPOINT는 컨테이너를 구동할 때 실행할 명령어를 지정한다. 
>
> 엔트리포인트는 커맨드를 인자로 받아서 사용할 수 있는 스크립트의 역할을 할 수 있다는 점에서 다르다.
>
> ```
> docker run -i -t --name no_entrypoint ubuntu:14.04 /bin/bash
> 
> docker run -i -t --entrypoint="echo" --name yes_entrypoint ubuntu:14.04 /bin/bash
> ```
>
> **entrypoint 가 설정되면 run 명령어의 맨 마지막에 입력된 cmd를 인자로 삼아 명령어를 출력한다.**
>
> 
>
> entrypoint 없음, cmd: /bin/bash  --> 컨테이너 내부에서 /bin/bash 실행
>
> entrypoint:ehco,cmd: /bin/bash --> 컨테이너 내부에서 /bin/bash 실행
>
> 
>
> ```
> ENTRYPOINT ["./run.sh"]
> ```



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



> .dockerignore
>
> ```
> test2.html
> *.html
> */*.html
> test.htm?
> ```

> ignore 파일은 컨텍스트의 최상위 경로, 즉 build 명령어에서 맨 마지막에 오는 경로인 Dockerfile이 위치한 경로와 같은 곳에 위치해야 한다.
>
> dockerignore의 제외목록에는 해당되지만, 특수한 파일을 포함하고자 하면 ! 를 사용한다
>
> ```
> *.html
> !test*.html
> ```
>
> 



> 도커파일을 빌드하고 다시 같은 빌드를 진행하면 이전의 이미지 빌드에서 사용했던 캐시를 사용한다.



> **Dockerfile 명 다르게 하기 **
>
> ```
> docker build -f Dockerfile2 -t mycache:0.0 ./
> ```
>
> 



> **캐시 기능 사용하지 않기 **
>
> build 명령어에 --no -cache 옵션을 추가한다.
>
> ```
> docker build 
> ```
>
> 



# 멀티 스테이지를 이용한 Dockerfile 빌드하기



> 기존 **Dockerfile을 이용한 빌드 방법**
>
> ```
> package main
> import "fmt"
> func main(){
> 	fmt.Println("hello world")
> }
> ```
>
> 
>
> ```
> FROM golang
> ADD main.go /root
> WORKDIR /root
> RUN go build -o /root/mainApp /root/main.go
> CMD ["./mainApp"]
> ```
>
> 
>
> 단순히 Hello world 를 출력함에도 불구하고 이미지의 크기에 800MB 이다.
>
> 실제 실행파일크기는 작지만 불필요한 라이브러리와 패키지가 크기를 차지하고 있다. 
>
> 17.05 이상은 멀티스테이지 빌드방법을 통해 크기를 줄일수 있다.



> 멀티 스테이지 빌드는 하나의 Dockerfile 안에 여러개의 FROM 이미지를 정의함으로써 빌드완료 시 최종적으로 생성될 이미지의 크기를 줄이는 역할을 한다.
>
> 
>
> 위 Dockerfile에서 빌드된 이미지와 동일하지만 멀티 스테이지 빌드 사용
>
> ```
> FROM golang
> ADD main.go /root
> WORKDIR /root
> RUN go build -o /root/mainApp /root/main.go
> 
> FROM alpine:latest
> WORKDIR /root
> COPY --from=0 /root/mainApp .
> CMD ["./mainApp"]
> ```
>
>  2개의 FROM 사용
>
> - 첫 번째 FROM
>
> 이전과 동일하게  main.go 파일을 /root/mainApp으로 빌드함
>
> - 두번째 FROM
>
> COPY를 통해 첫번째 FROM에서 사용된 이미지의 최종 상태에 존재하는 /root/myApp 파일을 두번째 이미지인 alpine:latest 에 복사한다.
>
> -- from=0 은 첫 번째 FROM 에서 빌드된 이미지의 최종 상태
>
> 
>
> **즉, 첫번째 FROM 이미지에서 빌드한 /root/mainApp 파일을 두번째 FROM에 명시된 이미지인 alpine:latest에 복사한다.**
>
> 

