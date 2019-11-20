# python2.7-to-golang
Retire python2.7 scripts and try to migrate to golang.

## Type

### string

* `"aaabbb".startswith("aa")`

```go
strings.HasPrefix("aaabbb", "aa")
```

* `"aaabbb".endswith("bb")`

```go
strings.HasSuffix("aaabbb", "bb")
```

* `"a"*16`

```go
strings.Repeat("a", 16)
```

* `"a" in "aa"`

```go
strings.Contains("aa", "a")
```

* `"abcb".replace("b", "c")`

```go
strings.ReplaceAll("abcb", "b", "c")
```

## Package

### string

* `string.Template("${a}${b}").safe_substitute({"a":1,"b":2})`

This function doesn't has a proper replace go package. We need to implement [PEP0292](https://www.python.org/dev/peps/pep-0292/) ourself. `strings.Replacer` will apply to most of the cases. 

```go
func SafeSubstitute(temp string, vars map[string]string) string {
	replacerList := []string{}
	for varName, varValue := range vars {
		replacerList = append(replacerList, "${"+varName+"}")
		replacerList = append(replacerList, varValue)
		replacerList = append(replacerList, "$"+varName)
		replacerList = append(replacerList, varValue)
	}
	replacerList = append(replacerList, "$$")
	replacerList = append(replacerList, "$")
	rep := strings.NewReplacer(replacerList...)
	return rep.Replace(temp)
}
```

### time

* int(time.time()): Unix second

```go
time.Now().Unix()
```

### urllib

* `urllib.quote(s)`: Quote **path section** of the URL

```go
url.PathEscape(s)
```

* `urllib.quote_plus(s)`: Quote **query section** of the URL

```go
url.QueryEscape(s)
```

### urlparse

* `urlparse.urlparse`
    * `u = urlparse('http://www.aaa.bb:80/c/d.html?a=1')`
        * `u.schema`: `http`
        * `u.netloc`: `www.aaa.bb:80`
        * `u.hostname`: `www.aaa.bb`
        * `u.query`: `a=1`

    * `u, _ := url.Parse("http://www.aaa.bb:80/c/d.html?a=1")`
        * `u.Schema`: `http` 
        * `u.Host`: `www.aaa.bb:80`
            * `host, port, _ := net.SplitHostPort(u.Host)`
                * `host`: `www.aaa.bb`
                * `post`: `80`
                * **Notice that `net.SplitHostPort("www.kimo.com")` return err and empty host and empty port**
        * `u.RawQuery`: `a=1`
