### Go를 빌드해보자





### Go 를 컨테이너화 해보자

https://blog.chann.kr/posts/scratch-container-with-go/ 출처

```
FROM golang:1.16-alpine

ENV CGO_ENABLED=0

WORKDIR /app

COPY go.mod ./
COPY go.sum ./
RUN go mod download

COPY . .
RUN go build -o ./fems .

EXPOSE 8080
CMD ["./fems"]
```

```
docker run --rm -it -d --name test -p 8080:8080 fems:0.1
```

```
docker build -t fems:0.1  .
```

