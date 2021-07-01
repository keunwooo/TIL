**이미지 관련 **

이미지 목록 보기

```null
$ sudo docker images
```

이미지 보기

```null
$ sudo docker search [이미지 이름]
```

이미지 받기

```null
$ sudo docker pull [이미지 이름]:[버전]
```

버전: **latest** 를 쓰면 최신 버전으로 받을수 있다.

이미지 삭제

```null
$ sudo docker rmi [이미지 id]
```

컨테이너를 삭제하기 전에 이미지를 삭제 할때, **-f** 옵션을 붙어면 컨테이너도 강제 삭제가 가능하다.

```null
$ sudo docker rmi -f [이미지 id]
```



**컨테이너 관련**

다양한 프로그램(nginx, database, WAS 등)을 컨테이너 라는 격리된 환경을 이용하여 실행시킬수 있습니다.

#### 컨테이너 목록 보기

```null
$ sudo docker ps
```

옵션

- -a : 모든 컨테이너 목록 출력

#### 컨테이너 실행

```null
$ sudo docker run [options] image[:TAG|@DIGEST] [COMMAND] [ARG...]
```

| 옵션   | 설명                                                         |
| ------ | ------------------------------------------------------------ |
| -d     | detached mode 흔히 말하는 백그라운드 모드                    |
| -p     | 호스트와 컨테이너의 포트를 연결 (포워딩)                     |
| -v     | 호스트와 컨테이너의 디렉토리를 연결 (마운트)                 |
| -e     | 컨테이너 내에서 사용할 환경변수 설정                         |
| --name | 컨테이너 이름 설정                                           |
| --it   | -i와 -t를 동시에 사용한 것으로 터미널 입력을 위한 옵션 (컨테이너의 표준 입력과 로컬 컴퓨터의 키보드 입력을 연결) |
| --rm   | 프로세스 종료시 컨테이너 자동 제거                           |
| --link | 컨테이너 연결 [컨테이너 명:별칭]                             |

- ex) $ sudo docker run -i -t --name server ubuntu:latest /bin/bash



컨테이너 시작

```null
$ sudo docker start [컨테이너 id 또는 name]
```

컨테이너 재시작

```null
$ sudo docker restart [컨테이너 id 또는 name]
```

컨테이너 접속

```null
$ sudo docker attach [컨테이너 id 또는 name]
```

컨테이너 정지

```null
$ sudo docker stop [컨테이너 id 또는 name]
```

- Bash Shell에서 `exit` 또는 `Ctrl + D`를 입력하면 컨테이너가 정지된다.
- `Ctrl + P`, `Ctrl + Q`를 차례대로 입력하여 컨테이너를 정지하지 않고, 컨테이너에서 빠져나온다.

컨테이너 삭제

```null
$ sudo docker rm [컨테이너 id 또는 name]
// 모든 컨테이너 삭제
$ sudo docker rm `docker ps -a -q`
```

