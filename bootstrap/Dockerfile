#**********************************************************************
# Builder
# 
# Create a go runtime suitable for building and testing kfctl
ARG GOLANG_VERSION=1.12.7
FROM golang:$GOLANG_VERSION as builder

RUN apt-get update
RUN apt-get install -y git unzip

# junit report is used to conver go test output to junit for reporting
RUN go get -u github.com/jstemmer/go-junit-report

ENV PATH /go/bin:/usr/local/go/bin:/opt/google-cloud-sdk/bin:${PATH}

# use go modules
ENV GO111MODULE=on
ENV GOPATH=/go

# Create bootstrap folder
RUN mkdir -p ${GOPATH}/src/github.com/kubeflow/kubeflow/bootstrap
WORKDIR ${GOPATH}/src/github.com/kubeflow/kubeflow/bootstrap

# Download dependencies first to optimize Docker caching.
COPY go.mod .
COPY go.sum .
RUN go mod download
# Copy in the source
COPY . .
RUN make build-bootstrap

#**********************************************************************
#
# bootstrap
#
FROM golang:$GOLANG_VERSION as bootstrap

RUN mkdir -p /opt/kubeflow
WORKDIR /opt/kubeflow

COPY --from=builder /go/src/github.com/kubeflow/kubeflow/bootstrap/bin/bootstrapper /opt/kubeflow/

RUN mkdir -p /opt/kfapps
# directory for cert file
RUN mkdir -p /opt/ca
RUN mkdir -p /opt/versioned_registries
RUN chmod a+rx /opt/kubeflow/bootstrapper

ENV PATH /opt/kubeflow:/opt/bootstrap:${PATH}


EXPOSE 8080

# Set default values for USER, USER_ID, GROUP_ID
# The startup script will create the user and su to that user.
# We delay the user creation until runtime so that user can specify
# the user info at runtime.

# Work around for https://github.com/ksonnet/ksonnet/issues/298
ENV USER kubeflow
ENV USER_ID 1000
ENV GROUP_ID 1000
ENV GROUP kubeflow
