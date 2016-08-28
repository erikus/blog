+++
date        = "2016-08-27T11:27:27-04:00"
title       = "eg to apply transformations to Go code"
description = "A collection of tools helps you to refactor Go code programmatically"
tags        = [ "tooling" ]
+++

If you are willing to make large scale refactoring in your
Go programs, automating the refactoring tasks is more diserable than
manual editing. `eg` is a program allows you to perform transformations
based on template Go files.

To install the tool, run the following:

```
$ go get golang.org/x/tools/cmd/eg
```

`eg` requires a template file to look for which transformation it should
apply to your source code. What's nice is that template file is a Go file
with little annotations.

Consider the following Go program:

``` go
$ cat $GOPATH/src/hello/hello.go
package hello

import "time"

// ExtendWith50000ns adds 50000ns to t.
func ExtendWith50000ns(t time.Time) time.Time {
	return t.Add(time.Duration(50000))
}
```
Assume you want to eliminate the unnecessary time.Duration casting at ExtendWith50000ns.
And as a good practice, you would also add a unit to the duration rather than just passing 50000.

`eg` requires a template file where you define before and afters that represents the
transformation.

``` go
$ cat T.template
package template

import (
    "time"
)

func before(t time.Time, d time.Duration) time.Time {
    // if already time.Duration, do not cast.
    return t.Add(time.Duration(d))
}

func after(t time.Time, d time.Duration) time.Time  {
    return t.Add(d * time.Nanosecond)
}
```

And run the `eg` command on your hello package to apply it at every occurrence of this pattern.

```
$ eg -w -t T.template hello
=== /Users/jbd/src/hello/hello.go (1 matches)
```

Voila!

The file now contains a duration that is not casted unnecessarily and it has a unit.

``` go
$ cat $GOPATH/src/hello/hello.go
package hello

import "time"

// ExtendWith50000ns adds 50000ns to t.
func ExtendWith50000ns(t time.Time) time.Time {
	return t.Add(50000 * time.Nanosecond)
}
```

Note: There are many [.template files](https://github.com/golang/tools/tree/master/refactor/eg/testdata)
underneath the package for testing purposes but they can also be used as a
reference how to write other transformation templates.