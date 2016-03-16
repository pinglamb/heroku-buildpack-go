# Heroku Buildpack: Go

This is the [Heroku buildpack][buildpack] for [Go][go].

## Getting Started


Follow the guide at
<https://devcenter.heroku.com/articles/getting-started-with-go>

There's also a hello world sample app at
<https://github.com/heroku/go-getting-started>

## Example

```
$ ls -A1
.git
Godeps
Procfile
web.go

$ heroku create
Creating polar-waters-4785...
...

$ git push heroku master
...
-----> Fetching custom git buildpack... done
-----> Go app detected
-----> Installing go1.4.1... done
-----> Running: godep go install -tags heroku ./...
-----> Discovering process types
       Procfile declares types -> web

-----> Compressing... done, 1.6MB
-----> Launching... done, v4
       https://polar-waters-4785.herokuapp.com/ deployed to Heroku
```

This buildpack will detect your repository as Go if you are using either:

- [Godep][godep]
- [GB][gb]
- [govendor][govendor]

This buildpack adds a `heroku` [build constraint][build-constraint], to enable
heroku-specific code. See the [App Engine build constraints
article][app-engine-build-constraints] for more.

## govendor specifics

The [vendor.json][vendor.json] spec that govendor follows for it's meta-data
file allows for arbitrary, tool specific fields. This buildpack uses this
feature to track build specific bits. These bits are encoded in the following
top level json keys:

- `"rootPath"` (String): the root package name of the packages your are pushing
to Heroku. You can find this locally with `go list -e .`. There is no default for
this and it must be specified.
- `"goVersion"` (String): the major version of go you would like to use. If not specified
the buildpack defaults to the most recent supported version of Go.
- `"install"` (Array of strings): a list of the packages you wish the go command to
install. This defaults to "." if not specified. "./cmd/..." or "./..." are other popular
choices, but the exact choice depens on the layour of your repository.

Example with everything, for a project using go 1.6, located at
`$GOPATH/src/github.com/heroku/go-getting-started` and requiring a single  package
spec of `./...` to install.

```json
{
    ...
    "rootPath": "github.com/heroku/go-getting-started",
    "install" : [ "./..." ],
    "goVersion": "go1.6",
    ...
}
```

A tool like jq or a text editor can be used to inject these variables into
`vendor/vendor.json` and the buildpack suggests some quick one-liners if it
detects these fields are not set.

## Hacking on this Buildpack

To change this buildpack, fork it on GitHub. Push changes to your fork, then
create a test app with `--buildpack YOUR_GITHUB_GIT_URL` and push to it. If you
already have an existing app you may use `heroku config:add
BUILDPACK_URL=YOUR_GITHUB_GIT_URL` instead of `--buildpack`.

## Using with cgo

This buildpack supports building with C dependencies via
[cgo][cgo]. You can set config vars to specify CGO flags
to, e.g., specify paths for vendored dependencies. E.g., to build
[gopgsqldriver][gopgsqldriver], add the config var
`CGO_CFLAGS` with the value `-I/app/code/vendor/include/postgresql` and include
the relevant Postgres header files in `vendor/include/postgresql/` in your app.

## Passing a symbol (and optional string) to the linker

This buildpack supports the go [linker's][go-linker] ability (`-X symbol
value`) to set the value of a string at link time. This can be done by setting
`GO_LINKER_SYMBOL` and `GO_LINKER_VALUE` in the application's config before
pushing code. If `GO_LINKER_SYMBOL` is set, but `GO_LINKER_VALUE` isn't set
then `GO_LINKER_VALUE` defaults to [`$SOURCE_VERSION`][source-version].

This can be used to embed the commit sha, or other build specific data directly
into the compiled executable.

[go]: http://golang.org/
[buildpack]: http://devcenter.heroku.com/articles/buildpacks
[go-linker]: https://golang.org/cmd/ld/
[godep]: https://github.com/tools/godep
[govendor]: https://github.com/kardianos/govendor
[gb]: https://getgb.io/
[quickstart]: http://mmcgrana.github.com/2012/09/getting-started-with-go-on-heroku.html
[build-constraint]: http://golang.org/pkg/go/build/
[app-engine-build-constraints]: http://blog.golang.org/2013/01/the-app-engine-sdk-and-workspaces-gopath.html
[source-version]: https://devcenter.heroku.com/articles/buildpack-api#bin-compile
[cgo]: http://golang.org/cmd/cgo/
[vendor.json]: https://github.com/kardianos/vendor-spec
[gopgsqldriver]: https://github.com/jbarham/gopgsqldriver
