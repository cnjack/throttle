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
## Explain
copy from [martini-contrib/throttle](https://github.com/martini-contrib/throttle)

## Authors

* [Jack](https://github.com/cnjack)
