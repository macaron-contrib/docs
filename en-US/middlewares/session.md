---
root: false
name: Session
sort: 7
---

# Session

Middleware session provides session management for Macaron [Instances](../intro/core_concepts#instances).

- [GitHub](https://github.com/macaron-contrib/session)
- [API Reference](https://gowalker.org/github.com/macaron-contrib/session)

## Installation

    go get github.com/macaron-contrib/session

## Usage

```go
import (
    "github.com/Unknwon/macaron"
    "github.com/macaron-contrib/session"
)

func main() {
    m := macaron.Classic()
    m.Use(session.Sessioner())

    m.Get("/", func(sess session.Store) string {
        sess.Set("session", "session middleware")
        return sess.Get("session").(string)
    })

    m.Get("/signup", func(ctx *macaron.Context, f *session.Flash) {
        f.Success("yes!!!")
        f.Error("opps...")
        f.Info("aha?!")
        f.Warning("Just be careful.")
        ctx.HTML(200, "signup")
    })

    m.Run()
}
```

```html
<!-- templates/signup.tmpl -->
<h2>{{.Flash.SuccessMsg}}</h2>
<h2>{{.Flash.ErrorMsg}}</h2>
<h2>{{.Flash.InfoMsg}}</h2>
<h2>{{.Flash.WarningMsg}}</h2>
```

## Options

`session.Sessioner` comes with a variety of configuration options([`session.Options`](https://gowalker.org/github.com/macaron-contrib/session#Options)):

```go
//...
m.Use(session.Sessioner(session.Options{
    // Name of provider. Default is "memory".
    Provider:       "memory",
    // Provider configuration, it's corresponding to provider.
    ProviderConfig: "",
    // Cookie name to save session ID. Default is "MacaronSession".
    CookieName:     "MacaronSession",
    // Cookie path to store. Default is "/".
    CookiePath:     "/",
    // GC interval time in seconds. Default is 3600.
    Gclifetime:     3600,
    // Max life time in seconds. Default is whatever GC interval time is.
    Maxlifetime:    3600,
    // Use HTTPS only. Default is false.
    Secure:         false,
    // Cookie life time. Default is 0.
    CookieLifeTime: 0,
    // Cookie domain name. Default is empty.
    Domain:         "",
    // Session ID length. Default is 16.
    IdLength:       16,
    // Configuration section name. Default is "session".
    Section:        "session",
}))
//...
```

## Providers

There are 8 built-in implementations of session provider, you have to import provider driver explicitly except for **memory** and **file** providers.

Following are some basic usage examples for providers.

### Memory

```go
//...
m.Use(session.Sessioner())
//...
```

### File

```go
//...
m.Use(session.Sessioner(session.Options{
    Provider:       "file",
    ProviderConfig: "data/sessions",
}))
//...
```

### Redis

```go
import _ "github.com/macaron-contrib/session/redis"

//...
m.Use(session.Sessioner(session.Options{
    Provider:       "redis",
    ProviderConfig: "127.0.0.1:6379,100,macaron",
}))
//...
```

### Memcache

```go
import _ "github.com/macaron-contrib/session/memcache"

//...
m.Use(session.Sessioner(session.Options{
    Provider:       "memcache",
    ProviderConfig: "127.0.0.1:9090",
}))
//...
```

### PostgreSQL

Use following SQL to create database:

```sql
CREATE TABLE `session` (
    `session_key` char(64) NOT NULL,
    `session_data` blob,
    `session_expiry` int(11) unsigned NOT NULL,
    PRIMARY KEY (`session_key`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8;
```

```go
import _ "github.com/macaron-contrib/session/postgres"

//...
m.Use(session.Sessioner(session.Options{
    Provider:       "postgres",
    ProviderConfig: "user=a password=b dbname=c sslmode=disable",
}))
//...
```

### MySQL

Use following SQL to create database:

```sql
CREATE TABLE `session` (
    `session_key` char(64) NOT NULL,
    `session_data` blob,
    `session_expiry` int(11) unsigned NOT NULL,
    PRIMARY KEY (`session_key`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8;
```

```go
import _ "github.com/macaron-contrib/session/mysql"

//...
m.Use(session.Sessioner(session.Options{
    Provider:       "mysql",
    ProviderConfig: "username:password@protocol(address)/dbname?param=value",
}))
//...
```

### Couchbase

```go
import _ "github.com/macaron-contrib/session/couchbase"

//...
m.Use(session.Sessioner(session.Options{
    Provider:       "couchbase",
    ProviderConfig: "username:password@protocol(address)/dbname?param=value",
}))
//...
```

### Ledis

```go
import _ "github.com/macaron-contrib/session/ledis"

//...
m.Use(session.Sessioner(session.Options{
    Provider:       "ledis",
    ProviderConfig: "data/sessions",
}))
//...
```

## Implement Provider Interface

In case you need to have your own implementation of session storage and provider, you can implement following two interfaces and take **memory** provider as a study example.

```go
// RawStore is the interface that operates the session data.
type RawStore interface {
	// Set sets value to given key in session.
	Set(key, value interface{}) error
	// Get gets value by given key in session.
	Get(key interface{}) interface{}
	// Delete delete a key from session.
	Delete(key interface{}) error
	// ID returns current session ID.
	ID() string
	// Release releases session resource and save data to provider.
	Release() error
	// Flush deletes all session data.
	Flush() error
}

// Provider is the interface that provides session manipulations.
type Provider interface {
	// Init initializes session provider.
	Init(gclifetime int64, config string) error
	// Read returns raw session store by session ID.
	Read(sid string) (RawStore, error)
	// Exist returns true if session with given ID exists.
	Exist(sid string) bool
	// Destory deletes a session by session ID.
	Destory(sid string) error
	// Regenerate regenerates a session store from old session ID to new one.
	Regenerate(oldsid, sid string) (RawStore, error)
	// Count counts and returns number of sessions.
	Count() int
	// GC calls GC to clean expired sessions.
	GC()
}
```