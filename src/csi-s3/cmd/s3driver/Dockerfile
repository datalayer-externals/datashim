ARG TARGETARCH
FROM golang:1.23-alpine3.20 as base

ARG S3BACKER_VERSION=1.5.0

RUN apk add --update --no-cache \
  fuse-dev \
  fuse \
  git curl 

WORKDIR /go/src

# Build csi-s3
COPY . csi-s3
WORKDIR /go/src/csi-s3
RUN ls -l && \ 
    go mod tidy && \
    CGO_ENABLED=0 GOOS=linux go build -a -ldflags '-extldflags "-static"' -o /go/bin/s3driver /go/src/csi-s3/cmd/s3driver

# Installing and compiling various S3 filesystems and mounters
# Datashim's CSI-S3 is only tested with goofys at present so the others are redundant for now
# Build goofys 
# Add additional line to go.mod to deal with CVE-2021-38561

# AP: NOTE: git checkout is required because as of 27/01/2023
# goofys cannot be built.
# This instruction can be safely removed once
# https://github.com/kahing/goofys/issues/740 has been fixed
# Update: Aug '23 - the fix is merged https://github.com/kahing/goofys/pull/744
WORKDIR /go/src
RUN git clone https://github.com/kahing/goofys.git && \
    cd goofys && \
    go mod tidy && \
    go build -o /go/bin/goofys

# Compile & install s3backer
# WORKDIR /
# RUN git clone https://github.com/archiecobbs/s3backer.git /s3backer && \
#     cd s3backer && \
#     git checkout tags/${S3BACKER_VERSION} && \
#     ./autogen.sh && \
#     ./configure && \
#     make && \
#     make install

# Compile and install s3fs-fuse
# WORKDIR /
# RUN git clone https://github.com/s3fs-fuse/s3fs-fuse.git && cd s3fs-fuse && \
# #git checkout v1.86 && \
# ./autogen.sh && ./configure && make && make install

# install rclone
# ARG RCLONE_VERSION=v1.47.0
# RUN cd /tmp \
#   && curl -O https://downloads.rclone.org/${RCLONE_VERSION}/rclone-${RCLONE_VERSION}-linux-amd64.zip \
#   && unzip /tmp/rclone-${RCLONE_VERSION}-linux-amd64.zip \
#   && mv /tmp/rclone-*-linux-amd64/rclone /usr/bin \
#   && rm -r /tmp/rclone*

FROM alpine:3.20

RUN apk add --update --no-cache fuse fuse-dev
COPY --from=base /go/bin/s3driver /s3driver
COPY --from=base /go/bin/goofys /bin/goofys
# COPY --from=base /usr/bin/s3backer /usr/bin/s3backer
# COPY --from=base /usr/bin/s3fs-fuse /usr/bin/s3fs-fuse
# COPY --from=base /usr/bin/rclone /usr/bin/rclone

ENTRYPOINT ["/s3driver"]
