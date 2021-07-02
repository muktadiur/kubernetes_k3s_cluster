FROM golang:alpine

WORKDIR /go/src/app

COPY . .

RUN go mod init

RUN go build -o helloworld main.go

EXPOSE 6111

CMD ["./helloworld"]
