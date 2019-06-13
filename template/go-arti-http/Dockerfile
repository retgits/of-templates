FROM golang:1.12.5-alpine3.9 as build

ENV JFROG_CLI_OFFER_CONFIG=false
ARG ARTIFACTORY_URL
ARG ARTIFACTORY_USER
ARG ARTIFACTORY_PASSWD
ARG APP_VERSION
ARG APP_NAME

RUN apk --no-cache add curl git \
    && echo "Pulling watchdog binary from Github." \
    && curl -sSLf https://github.com/openfaas-incubator/of-watchdog/releases/download/0.5.1/of-watchdog > /usr/bin/fwatchdog \
    && chmod +x /usr/bin/fwatchdog \
    && curl -Lk https://api.bintray.com/content/jfrog/jfrog-cli-go/\$latest/jfrog-cli-linux-386/jfrog?bt_package=jfrog-cli-linux-386 > /usr/bin/jfrog \
    && chmod +x /usr/bin/jfrog \
    && apk del curl --no-cache

RUN mkdir -p /home/app
WORKDIR /home/app
COPY . .
RUN mv function/go.* . && \
    line=`head -1 go.mod` && \
    export mod="${line:7}" && \
    sed -i -e "s#handler/function#$mod/function#g" main.go

RUN jfrog rt config --interactive=false --enc-password=true --url=${ARTIFACTORY_URL} --user=${ARTIFACTORY_USER} --password=${ARTIFACTORY_PASSWD}
RUN CGO_ENABLED=0 GOOS=linux jfrog rt go --build-name=${APP_NAME} --build-number=${APP_VERSION} "build --ldflags \"-s -w\" -a -installsuffix cgo -o handler ." go
RUN jfrog rt go-publish --deps=ALL --build-name=${APP_NAME} --build-number=${APP_VERSION} --module=${APP_NAME} go ${APP_VERSION}
RUN jfrog rt bce ${APP_NAME} ${APP_VERSION}
RUN jfrog rt bp ${APP_NAME} ${APP_VERSION}

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
