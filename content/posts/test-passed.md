---
title: "Test Passed"
date: 2018-09-19T22:03:20-07:00
---

Eyeballing something and saying it will work is okay in many fields.
Software engineering is at times one of those fields.  Security
engineering however, is not.

One of the great things about Golang's tooling and one of the reasons
that NetAuth is written in Go in the first place is the ease of
developing tests and running them.  Unit tests are an almost trivial
thing to write in Go, and can make it much easier to refactor code
down the road.

Why then, is this the current state of testing in NetAuth?

```
maldridge@theGibson:~/go/src/github.com/NetAuth/NetAuth ((a4903f1...))$ go test -cover ./...
?   github.com/NetAuth/NetAuth/cmd/netauth  [no test files]
?   github.com/NetAuth/NetAuth/cmd/netauthd [no test files]
?   github.com/NetAuth/NetAuth/internal/crypto  [no test files]
?   github.com/NetAuth/NetAuth/internal/crypto/all  [no test files]
    ok  github.com/NetAuth/NetAuth/internal/crypto/bcrypt   (cached)    coverage: 85.7% of statements
    ok  github.com/NetAuth/NetAuth/internal/crypto/nocrypto (cached)    coverage: 100.0% of statements
?   github.com/NetAuth/NetAuth/internal/ctl [no test files]
?   github.com/NetAuth/NetAuth/internal/db  [no test files]
?   github.com/NetAuth/NetAuth/internal/db/all  [no test files]
    ok  github.com/NetAuth/NetAuth/internal/db/memdb    0.021s  coverage: 93.3% of statements
    ok  github.com/NetAuth/NetAuth/internal/db/protodb  0.005s  coverage: 46.8% of statements
    ok  github.com/NetAuth/NetAuth/internal/health  0.013s  coverage: 100.0% of statements
?   github.com/NetAuth/NetAuth/internal/rpc [no test files]
?   github.com/NetAuth/NetAuth/internal/token   [no test files]
?   github.com/NetAuth/NetAuth/internal/token/all   [no test files]
    ok  github.com/NetAuth/NetAuth/internal/token/jwt   0.678s  coverage: 70.4% of statements
    ok  github.com/NetAuth/NetAuth/internal/tree    0.010s  coverage: 85.6% of statements
?   github.com/NetAuth/NetAuth/pkg/client   [no test files]
```

The simple answer is that tests aren't as exciting to write as new
features.

So this led to writing a lot of tests and seeing if it was possible to
get to 100% test coverage.  It is not only possible, but in many cases
it was fairly easy to ensure hat modules behaved exactly as expected.
Perhaps the most important module to increase statement coverage was
in ProtoDB, the default storage engine.  Given that the integrity of
the rest of the system depends on reliably writing to disk, the
ProtoDB code was some of the first to get more complete tests.

After a few days and a lot of time spent thinking about how to make
things fail in exactly the right implausible way, the following test
coverage was obtained:

```
maldridge@theGibson:~/go/src/github.com/NetAuth/NetAuth (master)$ go test -cover ./...
?   github.com/NetAuth/NetAuth/cmd/netauth  [no test files]
?   github.com/NetAuth/NetAuth/cmd/netauthd [no test files]
    ok  github.com/NetAuth/NetAuth/internal/crypto  0.013s  coverage: 100.0% of statements
?   github.com/NetAuth/NetAuth/internal/crypto/all  [no test files]
    ok  github.com/NetAuth/NetAuth/internal/crypto/bcrypt   0.176s  coverage: 100.0% of statements
    ok  github.com/NetAuth/NetAuth/internal/crypto/nocrypto 0.002s  coverage: 100.0% of statements
?   github.com/NetAuth/NetAuth/internal/ctl [no test files]
    ok  github.com/NetAuth/NetAuth/internal/db  0.005s  coverage: 100.0% of statements
?   github.com/NetAuth/NetAuth/internal/db/all  [no test files]
    ok  github.com/NetAuth/NetAuth/internal/db/memdb    0.023s  coverage: 100.0% of statements
    ok  github.com/NetAuth/NetAuth/internal/db/protodb  0.014s  coverage: 100.0% of statements
    ok  github.com/NetAuth/NetAuth/internal/health  0.003s  coverage: 100.0% of statements
?   github.com/NetAuth/NetAuth/internal/rpc [no test files]
    ok  github.com/NetAuth/NetAuth/internal/token   0.011s  coverage: 100.0% of statements
?   github.com/NetAuth/NetAuth/internal/token/all   [no test files]
    ok  github.com/NetAuth/NetAuth/internal/token/jwt   3.566s  coverage: 100.0% of statements
    ok  github.com/NetAuth/NetAuth/internal/tree    0.008s  coverage: 91.7% of statements
?   github.com/NetAuth/NetAuth/pkg/client   [no test files]
```

As you can see, all but one module that have test are now at 100% test
coverage.  Testing the internals of the tree system requires some more
work to make things fail in exactly the right sequence for each test,
but this will happen in time.

Tests for the server itself fall under the range of integration tests
and will require consideration for how to develop, and tests for the
rpc package need to be drafted as well.  The RPC tests are perhaps
some of the most important as they test that the security controls are
functioning properly.

If you'd like to assist with writing tests, feel free to reach out in
#netauth-dev on freenode.
