---
layout: post
title:  "JSON and GO"
date:   2018-07-11 23:00:00 +0700
categories: [JSON, Golang]
comment: true
---

To decode JSON data in Golang we use the [Unmarshal](https://golang.org/pkg/encoding/json/#Unmarshal) function or [NewDecoder](https://golang.org/pkg/encoding/json/#NewDecoder).

These functions are intentionally similar. Using Marshal/Unmarshal when you need to work with byte slices (in-memory), use Encode/Decode when you need to work with a reader/writer (file or other stream). The added advantage with Unmarshal is since the byte slice is already constructed and available, it will not copy the bytes, so struct accesses will operate on the byte slice in-place

```golang
func Unmarshal(data []byte, v interface{}) error
```

As an example, we make a POST request to create a user

```
curl -X POST -d '{"username": "xyz"}' http://localhost:8000/users
```

We need to create a place where the decoded data will be stored

```golang
type User struct {
    Username string `string:"username"`
}
```

```golang

var user User

// Unmarshal Way
func CreateUser(w http.ResponseWriter, r *http.Request) {
    body, _ := ioutil.ReadAll(r.Body)
    err = json.Unmarshal(body, &user)
}

// NewDecoder Way
func CreateUser(w http.ResponseWriter, r *http.Request) {
    err := json.NewDecoder(r.Body).Decode(&user)
}
```

**References:** https://blog.golang.org/json-and-go