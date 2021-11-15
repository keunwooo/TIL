```
docker build -t fems:0.2  .
```

```
docker run --rm -it -d --name fems -p 23523:8100 fems:0.2
```

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

# FROM gcr.io/distroless/base-debian10

# WORKDIR /

# COPY --from=build /fems /fems

# EXPOSE 8080
# USER nonroot:nonroot

# ENTRYPOINT ["/fems"]

```

