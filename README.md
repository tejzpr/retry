[![License](https://img.shields.io/badge/license-MIT%20License-blue.svg)](https://github.com/s0rg/retry/blob/master/LICENSE)
[![Go Version](https://img.shields.io/github/go-mod/go-version/s0rg/retry)](go.mod)

# retry

 Small, full-featured, 100% test-covered retry package for golang.

# features

 - small (~200 sloc), 100% test-covered codebase
 - fully-customizable, you can specify number of retries, sleep (and sleep-jitter) between them and stdlog verbosity
 - three ways to retry - single function, chain (one-by-one) and parallel execution

# examples

## simple
```go
import (
    "log"

    "github.com/s0rg/retry"
)


    try := retry.New()

    // single
    if err := try.Single("single-func", func() error {
        return initSomeResource()
    }); err != nil {
        log.Fatal("retry:", err)
    }
```

## with config
```go
import (
    "log"
    "time"

    "github.com/s0rg/retry"
)

    try := retry.New(
        retry.Count(5),
		retry.Parallelism(2),
	    retry.Sleep(time.Second*3),
	    retry.Jitter(time.Second/2),
		retry.Verbose(true),
    )

    var (
        dbh *sql.DB
        kaf *kafka.Conn
        red *redis.Conn
    )

    steps := []retry.Step{
        {"database", func() (err error) {
            dbh, err = sql.Open(...)

            return
        }},
        {"kafka", func() (err error) {
            kaf, err = kafka.Connect(...)

            return
        }},
        {"redis", func() (err error) {
            red, err = redis.Connect(...)

            return
        }},
    }

    // parallel execution
    if err := try.Parallel(steps...); err != nil {
        log.Fatal("retry:", err)
    }

    // at this point all tree resources will be avialaible
```
