FROM golang:1.13-alpine3.10

# Add build tools.
RUN apk --no-cache --virtual build-dependencies add \
  build-base \
  git

# Grab and install the latest version of lndconnect.
WORKDIR $GOPATH/src/github.com/LN-Zap/lndconnect
RUN git clone https://github.com/LN-Zap/lndconnect . \
  && make \
  && make install \
  && cp /go/bin/lndconnect /bin/

