# DockerHub



이미지를 생성하면 도커 리포지토리를 이용해서 공유할 수 있다. 다양한 도커 리포지토리가 존재하는데 이 중 대표적인 것이 도커 허브이다. 지금까지 예에서 사용한 nginx나 alpine과 같은 이미지가 위치한 곳이기도 하다. [https://hub.docker.com](https://hub.docker.com/) 사이트에 가입하면 도커 허브에 이미지를 푸시해서 공유할 수 있다.



> 먼저 생성한 이미지에 추가로 태그를 붙인다. 추가 태그에는 리포지토리 계정 이름을 사용한다. 예를 들어 도커 허브 계정이 madvirus라면 madvirus/리포지토리이름:태그 형태를 사용한다. 다음은 simplenode:0.1 이미지에 madvirus/simplenode:0.1 태그를 추가하는 예인데 이때 madvirus라 도커 허브 계정이다.



```
$ docker tag simplenode:0.1 madvirus/simplenode:0.1
```

도커 로그인

```
$ docker login
```

이미지 푸시

```
docker push madvirus/simplenode:0.1
```

이미지 가져와서 사용하기

```
 docker run -d --rm -p 5000:5000 madvirus/simplenode:0.1
```

