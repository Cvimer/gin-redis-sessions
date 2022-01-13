# sessions

[![GoDoc](https://godoc.org/github.com/Calidity/sessions?status.svg)](https://godoc.org/github.com/Calidity/sessions)
[![Go Report Card](https://goreportcard.com/badge/github.com/Calidity/gin-sessions)](https://goreportcard.com/report/github.com/Calidity/gin-sessions)

Gin middleware for session management with multi-backend support:

- [cookie-based](#cookie-based)
- [Redis](#redis) using [go-redis/redis/v8](https://github.com/go-redis/redis)

This Redis client allows for using an existing client with support for Redis Sentinel and cluster.

Forked from https://github.com/gin-contrib/sessions

## Usage

### Start using it

Download and install it:

```bash
$ go get github.com/Cvimer/gin-redis-sessions
```

Import it in your code:

```go
import "github.com/Cvimer/gin-redis-sessions"
```

## Basic Examples

### single session

```go
package main

import (
    "github.com/Cvimer/gin-redis-sessions"
    "github.com/Cvimer/gin-redis-sessions/cookie"
    "github.com/gin-gonic/gin"
)

func main() {
  r := gin.Default()
  store := cookie.NewStore([]byte("secret"))
  r.Use(sessions.Sessions("mysession", store))

  r.GET("/hello", func(c *gin.Context) {
    session := sessions.Default(c)

    if session.Get("hello") != "world" {
      session.Set("hello", "world")
      session.Save()
    }

    c.JSON(200, gin.H{"hello": session.Get("hello")})
  })
  r.Run(":8000")
}
```

### multiple sessions

```go
package main

import (
    "github.com/Cvimer/gin-redis-sessions"
    "github.com/Cvimer/gin-redis-sessions/cookie"
    "github.com/gin-gonic/gin"
)

func main() {
  r := gin.Default()
  store := cookie.NewStore([]byte("secret"))
  sessionNames := []string{"a", "b"}
  r.Use(sessions.SessionsMany(sessionNames, store))

  r.GET("/hello", func(c *gin.Context) {
    sessionA := sessions.DefaultMany(c, "a")
    sessionB := sessions.DefaultMany(c, "b")

    if sessionA.Get("hello") != "world!" {
      sessionA.Set("hello", "world!")
      sessionA.Save()
    }

    if sessionB.Get("hello") != "world?" {
      sessionB.Set("hello", "world?")
      sessionB.Save()
    }

    c.JSON(200, gin.H{
      "a": sessionA.Get("hello"),
      "b": sessionB.Get("hello"),
    })
  })
  r.Run(":8000")
}
```

## Backend Examples

### cookie-based

```go
package main

import (
    "github.com/Cvimer/gin-redis-sessions"
    "github.com/Cvimer/gin-redis-sessions/cookie"
    "github.com/gin-gonic/gin"
)

func main() {
  r := gin.Default()
  store := cookie.NewStore([]byte("secret"))
  r.Use(sessions.Sessions("mysession", store))

  r.GET("/incr", func(c *gin.Context) {
    session := sessions.Default(c)
    var count int
    v := session.Get("count")
    if v == nil {
      count = 0
    } else {
      count = v.(int)
      count++
    }
    session.Set("count", count)
    session.Save()
    c.JSON(200, gin.H{"count": count})
  })
  r.Run(":8000")
}
```

### Redis

```go
package main

import (
    "github.com/Cvimer/gin-redis-sessions"
    sRedis "github.com/Cvimer/gin-redis-sessions/redis"
    "github.com/gin-gonic/gin"
    "github.com/go-redis/redis/v8"
)

func main() {
    r := gin.Default()
    store, _ := sRedis.NewRedisStore(redis.NewClient(&redis.Options{Addr: "localhost:6379"}), []byte("secret"))
    r.Use(sessions.Sessions("mysession", store))

    r.GET("/incr", func(c *gin.Context) {
        session := sessions.Default(c)
        var count int
        v := session.Get("count")
        if v == nil {
            count = 0
        } else {
            count = v.(int)
            count++
        }
        session.Set("count", count)
        session.Save()
        c.JSON(200, gin.H{"count": count})
    })
    r.Run(":8000")
}
```