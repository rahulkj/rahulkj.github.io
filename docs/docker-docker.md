Docker Docker
---

- To create a docker image, first create a `Dockerfile` in a director and then start editing the file.

- I created a docker image that I use with concourse and the contents of the file are as follows:
```
FROM ubuntu:17.04
MAINTAINER rahulkj@gmail.com
RUN apt-get update && apt-get install -y curl wget default-jdk maven gradle golang git
RUN wget -O cf-cli.deb "https://cli.run.pivotal.io/stable?release=debian64&source=github-rel"
RUN dpkg -i cf-cli.deb
RUN cf --version
RUN java -version
RUN mvn -v
RUN gradle -v
RUN go version
RUN ls -al $HOME
RUN mkdir $HOME/go
ENV GOPATH $HOME/go/
ENV GOBIN $GOPATH/bin
ENV PATH $PATH:$GOBIN
RUN go get github.com/pivotal-cf/om
RUN go get github.com/pivotal-cf/pivnet-cli
RUN go get github.com/vmware/govmomi/govc
```

- In the steps above, I'm bundling Java JDK, Maven, Gradle, Pivotal CLI's like cf, om and pivnet-cli, and finaly govc to interact with vsphere. Look at the way you set the `GOPATH` and `PATH` variable by using `ENV`

- The next step is to create a image from this file, so fire the command:
  - `docker build -t rjain/buildbox .` where `rjain/buildbox` is my image name

- Finally push the image up to your docker hub, by firing the command
  - `docker login`
  - `docker push rjain/buildbox`

- And verify that your image is showing up on https://hub.docker.com/
