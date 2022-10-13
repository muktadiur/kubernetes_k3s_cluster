FROM golang:alpine as build
WORKDIR /go/src/app
COPY . .
RUN go mod init
RUN go build -o helloworld main.go

FROM alpine
EXPOSE 6111
WORKDIR /app
COPY --from=build /go/src/app/ /app/
CMD ["./helloworld"]
