# throttle

Simple throttling for gin

[API Reference](http://godoc.org/github.com/cnjack/throttle)

## How to use
```go
package main

import (
	"github.com/cnjack/throttle"
	"github.com/gin-gonic/gin"
	"time"
)

func main() {
	r := gin.New()
	r.Use(throttle.Policy(&throttle.Quota{
		Limit:  1,
		Within: time.Hour,
	}))
	r.Any("/", func(c *gin.Context) {
		c.Writer.Write([]byte("hello world"))
	})
	r.Run(":8088")
}

```
#### Policy

`throttle.Policy` is a middleware that allows you to throttle by the given quota.

#### Quota

`throttle.Quota` is a quota type with a limit and a duration

## Options
You can configure the options for throttling by passing in ``throttle.Options`` as the second argument to ``throttle.Policy``. Use it to configure the following options (defaults are used here):

```go
&throttle.Options{
	// The Status Code returned when the client exceeds the quota. Defaults to 429 Too Many Requests
	StatusCode int

	// The response body returned when the client exceeds the quota
	Message string

	// A function to identify a request, must satisfy the interface func(*http.Request)string
	// Defaults to a function identifying the request by IP or X-Forwarded-For Header if provided
	// So if you want to identify by an API key given in request headers or something else, configure this option
	IdentificationFunction func(*http.Request) string

	// The key prefix to use in any key value store
	KeyPrefix string

	// The store to use. The key value store has to satisfy the throttle.KeyValueStorer interface
	// For further explanation, see below
	Store KeyValueStorer

	// If the throttle is disabled or not
	// defaults to false
	Disabled bool
}
```

## State Storage
Throttling relies on storage of one key per Policy and user in a (KeyValue) Storage. The interface the store has to satisfy is ``throttle.KeyValueStorer``, or, more explicit:

```go
type KeyValueStorer interface {
	Get(key string) ([]byte, error)
	Set(key string, value []byte) error
}
```

This allows for drop in replacement of the store with most common go libraries for key value stores like [redis.go](https://github.com/hoisie/redis)

```go
var client redis.Client

m.Use(throttle.Policy(&throttle.Quota{
	Limit: 10,
	Within: time.Minute,
}, &throttle.Options{
	Store: &client,
}))
```

Adapters are also very easy to write. ``throttle`` prefixes every key, your adapter does not have to care about it, and the stored value is stringified JSON.

The default state storage is in memory via a concurrent-safe `map[string][]byte` cleaning up every 15 minutes. While this works fine for clients running one instance of a martini server, for all other uses you should obviously opt for a proper key value store.

## Headers & Status Codes
``throttle`` adds the following ``X-RateLimit-*``-Headers to every response it controls:

- X-RateLimit-Limit: The maximum number of requests that the consumer is permitted to make within the given time window
- X-RateLimit-Remaining: The number of requests remaining in the current rate limit window
- X-RateLimit-Reset: The time at which the current rate limit window resets in [UTC epoch seconds](http://en.wikipedia.org/wiki/Unix_time)

No ``Retry-After`` Header is added to the response, since the ``X-RateLimit-Reset`` makes it redundant. Also it is not recommended to use a 503 Service Unavailable Status Code when Limiting the rate of requests, since the 5xx Status Code Family indicates an error on the servers side.

## Explain
copy from [martini-contrib/throttle](https://github.com/martini-contrib/throttle)

## Authors

* [Jack](https://github.com/cnjack)
