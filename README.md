# Custom OpenFaaS Templates

This repository contains templates for [OpenFaaS](https://openfaas.com) which I use for my OpenFaaS functions.

```
$ faas-cli template pull https://github.com/retgits/of-templates
$ faas-cli new --list

Languages available as templates:
- gomods-http
- go-arti-http
```

## gomods-http

The template _gomods-http_ is equivalent to [`golang-http`](https://github.com/openfaas-incubator/golang-http-template/tree/master/template/golang-http) with the exception being this template uses [Go modules](https://github.com/golang/go/wiki/Modules) and the Go [1.12.5](https://golang.org/doc/go1.12)

## go-arti-http

The template _go-arti-http_ is equivalent to `gomods-http` with the exception being this template uses [JFrog Artifactory](https://jfrog.com/artifactory) to resolve Go modules. If you need an Artifactory instance yo try it out, you can use [JFrog's testdrive](https://try.jfrog.com) and get a four-day triel of both Artifactory and Xray.

## Status of the templates

The template is customized to suit my needs, though if you have any thoughts or suggestions, I'd love to hear them.

Just like the `golang-http` template, this template makes use of the OpenFaaS incubator project [of-watchdog](https://github.com/openfaas-incubator/of-watchdog).

### Trying the templates

```
$ faas-cli template pull https://github.com/retgits/of-templatesgolang-http-template
$ faas-cli new --lang <template> <fn-name>
```

The above commands will create a folder with the name of your function and a set of files:

```text
.
├── go.mod
├── go.sum
└── handler.go
```

You can change the name of your package by updating the name in the `go.mod` file. The build template will automatically pick up the correct name when building an OpenFaaS function. It is important that you keep the `package function` declaration at the top of `handler.go`.

### Example usage

The below samples are from the [OpenFaaS](https://github.com/openfaas-incubator/golang-http-template) repository.

Example writing a successful message:

```go
package function

import (
	"fmt"
	"net/http"

	"github.com/openfaas-incubator/go-function-sdk"
)

// Handle a function invocation
func Handle(req handler.Request) (handler.Response, error) {
	var err error

	message := fmt.Sprintf("Hello world, input was: %s", string(req.Body))

	return handler.Response{
		Body:       []byte(message),
    }, err
}
```

Example writing a custom status code

```go
package function

import (
	"fmt"
	"net/http"

	"github.com/openfaas-incubator/go-function-sdk"
)

// Handle a function invocation
func Handle(req handler.Request) (handler.Response, error) {
	var err error

	return handler.Response{
		Body:       []byte("Your workload was accepted"),
		StatusCode: http.StatusAccepted,
	}, err
}
```

Example writing an error / failure.

```go
package function

import (
	"fmt"
	"net/http"

	"github.com/openfaas-incubator/go-function-sdk"
)

// Handle a function invocation
func Handle(req handler.Request) (handler.Response, error) {
	var err error

	return handler.Response{
        Body: []byte("the input was invalid")
	}, fmt.Errorf("invalid input")
}
```

The error will be logged to `stderr` and the `body` will be written to the client along with a HTTP 500 status code.

Example reading a header.

```go
package function

import (
	"log"

	"github.com/openfaas-incubator/go-function-sdk"
)

// Handle a function invocation
func Handle(req handler.Request) (handler.Response, error) {
	var err error

	log.Println(req.Header) // Check function logs for the request headers

	return handler.Response{
		Body: []byte("This is the response"),
		Header: map[string][]string{
			"X-Served-By": []string{"My Awesome Function"},
		},
	}, err
}
```
