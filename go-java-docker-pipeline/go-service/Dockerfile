ARG REGISTRY=docker.artifactory-unified.soleng-us.jfrog.team
FROM $REGISTRY/golang:latest
#FROM golang:latest

RUN mkdir /app
ADD . /app/
WORKDIR /app
RUN go build -o main .
CMD ["/app/main"]

EXPOSE 3000