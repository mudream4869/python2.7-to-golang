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

* "1|2|3".split("|", 1)

Notice that `1` means split the string by at **most 1 `|`**, however split function in golang use the number of **parts** as a parameter. 

```go
strings.SplitN("1|2|3", "|", 2)
```

### float(?)

Python's float function tranforms many things to float. According to [this](https://stackoverflow.com/a/20929983/4411336), "infinity" can also be tranformed to float. 

* [Stackoverflow: Converting unknown interface to float64 in Golang
](https://stackoverflow.com/questions/20767724/converting-unknown-interface-to-float64-in-golang)


### Encode to utf-8???

No need.

* Ref: [Stackoverflow: Equivalent of python's encode('utf8') in golang](https://stackoverflow.com/questions/42541297/equivalent-of-pythons-encodeutf8-in-golang)

## Package

### threading

* `threading.Lock`: Golang's sync.Mutex only provides **Lock** and **Unlock** operation.
    * No **TryLock**?
        * [This issue](https://github.com/golang/go/issues/6123) has been discussed in 2013, and golang seems not to implement this function.
        * [github.com/LK4D4/trylock](https://github.com/LK4D4/trylock)
        * Or implement by channel:
```go
// A Lock
ch := make(chan bool, 1)

// Lock
ch <- true

// Trylock?
select {
case ch <- true:
    // Trylock successfully
default:
    // Fail to lock 
}

// Islock?
len(ch) == 1

// Unlock
<-ch
```

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

### os

* `os.path.join(a, b, c)`

```go
path.Join(a, b, c)
```

### base64

#### URL Encoding

* `base64.urlsafe_b64encode(s)`

Example: base64.urlsafe_b64encode("a") = "YQ=="

```go
base64.RawURLEncoding.EncodeToString([]byte("a"))
```

EncodeToString("a") return "YQ" which has no "=" padding.

#### URL Decoding

* `base64.urlsafe_b64decode(s)`

Example: base64.urlsafe_b64decode("YQ==") = "a"
Throw exception if no "=" padding


```go
base64.RawURLEncoding.DecodeToString([]byte("YQ"))
```

DecodeToString("YQ") return "a".
DecodeToString("YQ==") return error: "=" is an illegal character.

### pytricia

Support ipv4, ipv6

* [github.com/yl2chen/cidranger](https://github.com/yl2chen/cidranger)
* Check/Get operation need about 100ns~300ns (DB records $\approx$ 1200)
