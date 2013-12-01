httphandlers
============

Simple net/http handlers for easier testing of web apis.

To install:

    go get github.com/jeffh/httphandlers

The handlers are broken into as follows:

    - routers: a simple way to route paths to specific handlers
    - filters: handlers that dispatches to either of two handlers based on a condition on the request
    - decorators: handlers that wraps another handler.
    - fixtures: simplistic handlers

Fixtures
========

The following constructors can be used to return static data:

    - `func NewFixtureAsXML(v interface{}) (*FixtureHandler, error)` returns a handler that returns XML based on the given struct data.
    - `func NewFixtureAsJSON(v interface{}) (*FixtureHandler, error)` returns a handler that returns JSON based on the given struct data.
    - `func NewFixtureAsFile(filepath string) (*FixtureHandler, error)` returns a handler that returns contents from a file.
    - `func NewFixtureAsReader(r io.Reader) (*FixtureHandler, error)` returns a handler that returns contents from the reader.
    - `func NewFixtureAsString(contents string) (*FixtureHandler, error)` returns a handler that returns contents from a string.
    - `func NewFixture(b []byte) (*FixtureHandler, error)` returns a handler that returns contents from raw bytes.

Fixture handlers support the following methods to mutate the response it generates:

    - `func (f *FixtureHandler) WithStatusCode(code int) *FixtureHandler` updates the handler to return a status code. Returns the same handler.
    - `func (f *FixtureHandler) WithHeader(key string, values []string) *FixtureHandler` updates the handler to return a status code. Returns the same handler.


Filters
=======

Filters allow a request to pass through to another handler if the incoming request is valid
to the filter.

RequiredHeadersHandler
----------------------

This handler passes through if only the provided handlers are present in the request.

    - `func NewRequiredHeadersHandler(headers http.Header, handler http.Handler) *RequiredHeadersHandler` returns a handler that calls the passed in handler if the headers are provided.

RequiredHeadersHandler supports one additional method, to attach a custom failure handler:

    - `func (d *RequiredHeadersHandler) WithFailedHandler(h http.Handler) *RequiredHeadersHandler`: sets the handler to call if the request does not contain the required headers and returns the same handler.

Decorators
==========

Decorators provide a simple transformation to the request or response.

RequestRecorderHandler
----------------------

This handler clones the incoming request. This is useful for later inspection by tests.
Recorded requests use `RecordedRequest` which has a slightly different field types

    - `func NewRequestRecorderHandler(h http.Handler) *RequestRecorderHandler` returns a new handler that passes through to the provided handler, but records the incoming request.

It supports querying and reset methods:

    - `func (h *RequestRecorderHandler) ClearRequests()` removes all recored requests.
    - `func (h *RequestRecorderHandler) FilterRequests(fn func(r *RecordedRequest) bool) []*RecordedRequest` returns all the recorded requests that match the given filter function.
    - `func (h *RequestRecorderHandler) RequestsByPath(path string) []*RecordedRequest` returns all the requests that match a given path.
    - `func (h *RequestRecorderHandler) RequestsByMethod(method string) []*RecordedRequest` returns all the requests that match a given HTTP method.

Routers
=======

Provides handlers that can dispatch to handlers.

URLHandler
----------

URLHandler provides a simply mapping to paths.

The internal map the URLHandler uses are strings to handlers:

    {"<METHOD> <PATH>": Handler}

Where `<METHOD>` is an HTTP method verb (etc - GET, POST) and `<PATH>` is path.
Using `ANY` in `<METHOD>` matches any HTTP verb.
Handler is an http.Handler to invoke if the given method and path.

Using the constructor function to create a URLHandler by passing in the mapping style mentioned:

    - `func NewURLHandler(routes map[string]http.Handler) *URLHandler` Creates a new URLHandler

