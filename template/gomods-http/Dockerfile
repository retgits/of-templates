FROM golang:1.12.5-alpine3.9 as build

RUN apk --no-cache add curl git \
    && echo "Pulling watchdog binary from Github." \
    && curl -sSLf https://github.com/openfaas-incubator/of-watchdog/releases/download/0.5.1/of-watchdog > /usr/bin/fwatchdog \
    && chmod +x /usr/bin/fwatchdog \
    && apk del curl --no-cache

RUN mkdir -p /home/app
WORKDIR /home/app
COPY . .
RUN mv function/go.* . && \
    line=`head -1 go.mod` && \
    export mod="${line:7}" && \
    sed -i -e "s#handler/function#$mod/function#g" main.go

# Run a gofmt and exclude all vendored code.
RUN test -z "$(gofmt -l $(find . -type f -name '*.go' -not -path "./vendor/*" -not -path "./function/vendor/*"))" || { echo "Run \"gofmt -s -w\" on your Golang code"; exit 1; }

RUN CGO_ENABLED=0 GOOS=linux go build --ldflags "-s -w" -a -installsuffix cgo -o handler .

FROM alpine:3.9
# Add non root user and certs
RUN apk --no-cache add ca-certificates \
    && addgroup -S app && adduser -S -g app app \
    && mkdir -p /home/app \
    && chown app /home/app

WORKDIR /home/app

COPY --from=build /home/app/handler          .
COPY --from=build /usr/bin/fwatchdog         .

RUN chown -R app /home/app

USER app

ENV fprocess="./handler"
ENV mode="http"
ENV upstream_url="http://127.0.0.1:8082"

CMD ["./fwatchdog"]
