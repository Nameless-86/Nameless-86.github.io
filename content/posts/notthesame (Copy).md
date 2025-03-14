+++
date = '2024-12-13'
title = 'Little Web Server'
+++

## In my journey of learning Go, I did a small web server using only the standard library.

```golang
import (
	"encoding/json"
	"fmt"
	"net/http"
	"strconv"
	"sync"
)
```

### For the enpoints I used ServeMux, whats ServeMux?

ServeMux is an HTTP request multiplexer.
It matches the URL of each incoming request against a list of registered patterns and calls the handler for the pattern that most closely matches the URL.
In simpler terms it handles the logic for every endpoint, following this pattern [[METHOD ][HOST]/[PATH]]

```golang
mux := http.NewServeMux()
	mux.HandleFunc("/", handleRoot)

	mux.HandleFunc("POST /users", createUser)
	mux.HandleFunc(
		"GET /users/{id}",
		getUser,
	)
	mux.HandleFunc(
		"DELETE /users/{id}",
		deleteUser,
	)
```

## What does each handler do?

all handlers are created for processing HTTP requests

### handleRoot

In this case when a request is done to the root path of the server "/" (mux takes care of the routing) and it just gives a response as a string that says "Hello Friend"

```golang
func handleRoot(
	w http.ResponseWriter,
	r *http.Request,
) {
	fmt.Fprintf(w, "Hello Friend")
}
```

### User Struct, data structure for the cache and mutex

for the handlers that interact with the server with more depth, like creating an user, deleting or selecting an user by its ID we need
something for the user, like a struct that holds its name, for storing more than one user we could use a hash map that maps an id (int) to the user that has been created

```golang
type User struct {
	Name string `json:"name"`
}

var userCache = make(map[int]User)

var cacheMutex sync.RWMutex
```

From the sync package we take RWMutex, this is very usefull for sync and share resources.
A Mutex is a mutual exclusion lock. The zero value for a Mutex is an unlocked mutex.

In the majority of apps you have more reads than writes, the RWMutex is useful for perfomance and readers don't block each other
Also writers block the readers until the Unlock() method is called

A Mutex is a mutual exclusion lock. The zero value for a Mutex is an unlocked mutex.

When you send an HTTP request with a JSON body (the name of the user) it gets decoded into the user variable, if there is an error (invalid json or doesnt match the user struct, you will get a BadRequest error)

If you provide an empty user you will also get an error, finally we add the user to the cache (hashmap) ensuring safety by locking and unlocking the resource

### createUser

```golang
func createUser(
	w http.ResponseWriter,
	r *http.Request,
) {
	var user User
	err := json.NewDecoder(r.Body).Decode(&user)
	if err != nil {
		http.Error(
			w,
			err.Error(),
			http.StatusBadRequest,
		)
		return
	}
	if user.Name == "" {
		http.Error(
			w,
			"name is required",
			http.StatusBadRequest,
		)
		return
	}

	cacheMutex.Lock()
	userCache[len(userCache)+1] = user
	cacheMutex.Unlock()

	w.WriteHeader(http.StatusNoContent)
	//sucessfully processed but no content to return
}
```

### Get User

```golang
func getUser(
	w http.ResponseWriter,
	r *http.Request,
) {
	id, err := strconv.Atoi(r.PathValue("id"))
	if err != nil {
		http.Error(
			w,
			err.Error(),
			http.StatusBadRequest,
		)
		return
	}
	cacheMutex.RLock()
	user, ok := userCache[id]
	cacheMutex.RUnlock()
	if !ok {
		http.Error(
			w,
			"user not found",
			http.StatusNotFound,
		)
		return
	}
	w.Header().Set("Content-Type", "application")
	j, err := json.Marshal(user)
	if err != nil {
		http.Error(
			w,
			err.Error(),
			http.StatusInternalServerError,
		)
		return
	}
	w.WriteHeader(http.StatusOK)
	w.Write(j)
}
```

We have already created a user, but how do i get an user from the cache?
For the user that we want we need its id, so the function will extract the user id from the provided url (it would look like this using curl `curl localhost:1234/1`) the server will send a response `r`and we extract the id with the `PathValue` method, before that the id gets converted into a integer, if it fails it will return an error (first error handler of the function)

After that we access the cache, it supports multiple reads because of the Mutex, we access retrieving the user by its id if the user does not exists we use the second error handler

Then we set the response as a json file, converting the user struct that we made at the begining into a JSON byte slice with the Marshal method, if something fails we have the third error handler

Finally we write a HTTP 200 response as a header and the encoded JSON user to the body of the response

```golang
func deleteUser(
	w http.ResponseWriter,
	r *http.Request,
) {
	id, err := strconv.Atoi(r.PathValue("id"))
	if err != nil {
		http.Error(
			w,
			err.Error(),
			http.StatusBadRequest,
		)
		return

	}
	if _, ok := userCache[id]; !ok {
		http.Error(
			w,
			"user not found",
			http.StatusBadRequest,
		)
		return
	}
	cacheMutex.Lock()
	delete(userCache, id)
	cacheMutex.Unlock()

	w.WriteHeader(http.StatusNoContent)
}
```
