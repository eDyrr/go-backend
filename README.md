# go-backend

this will contain everything that I learned about backend in golang

well as far as I am in backend, I've learned a couple of technologies such as Caching, Databases, Authentification, APIs ... etc

the main algorithm for a backend application is:

1. turn the app on
2. connect to the server
3. turn the db on (if there is a db)
4. recieve requests
5. send requests
6. manage resources

thats basically it, now if however there's other components included in the app like a database, we need to first connect to it first then turn the server on

now for the db to work with golang we first need to know the db that we're going to work with, then download its drivers.
for the sake of this tutorial ive chose sqlite (never really worked with it before), to get the drivers on your local machine, just run the following command `go get modernc.org/sqlite`.
after downloading the drivers, what we need is to write the code that allows us to interact with the DB.
```
func main() {
    db, err := sql.Open("sqlite", "file:mydb.sqlite?cache=shared")
    if err != nil {
        log.Fatal(err)
    }

    defer db.Close()

    _, err = db.Exec(`CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY, name TEXT)`)
    if err != nil {
        log.Fatal(err)
    }
}
```

now that we have the DB's essential part out the way, we can pass to the routing part, for the routing I am used to using `gorilla/mux`, to get this module on the machine, we run `go get -u github.com/gorilla/mux`, basically for the people that dont know, `gorilla/mux` matches incoming http request to resources (handlers), create URLs ...etc.

now that we have `gorilla/mux` installed, we need to create a router, to do so we just have one line of code to write:
`r := mux.NewRouter()`
after creating the router, we can attach handler functions to it, where we can have a bunch of endpoints and there's a bunch of functions that get called once a user requests some kind of http request to one of endpoints (resources).
for example 
`r.HandleFunc("/", func(w http.ResponseWriter, r* http.Request) {`
`w.Write([]byte("hello there"))`
`}).Methods("GET")`

at the end of the main function we listen and serve, passing the router to it by:
`http.ListenAndServe(":8080", r)`
the main function should look like this:
```
func main() {
    db, err := sql.Open("sqlite", "file:mydb.sqlite?cache=shared")
    if err != nil {
        log.Fatal(err)
    }

    defer db.Close()

   r := mux.NewRouter()
   r.HandleFunc("/", Index)

   http.ListenAndServe(":8080", r)
}
```

now I think its a fitting moment to talk about the file structure that any of my own standard golang backend project follows:

```
bookstore-api/
├── cmd/
│   └── server/
│       └── main.go      # Entry point for the application
├── pkg/
│   ├── book/
│   │   ├── handler.go   # HTTP handlers for book-related routes
│   │   ├── model.go     # Book data model
│   │   └── service.go   # Business logic for books
│   ├── database/
│   │   └── database.go  # Database connection and setup
│   ├── user/
│   │   ├── handler.go   # HTTP handlers for user-related routes
│   │   ├── model.go     # User data model
│   │   └── service.go   # Business logic for users
│   └── utils/
│       └── utils.go     # Utility functions and common helpers
├── go.mod               # Go module file
└── go.sum               # Go checksum file
```

here's a [link](https://www.codingexplorations.com/blog/managing-files-in-a-go-api-folder-structure-best-practices-for-organizing-your-project) to the original article that I got this structure from

I like this structure because its simple and holds related code together.

Now its time to talk about the frontend side of our application (even though it is not our focus but still), I prefer this more detailed/optimized file structure:
```
.
├── cmd
│   └── server
│       ├── internal
│       │   └── web
│       │       ├── static
│       │       │   └── css
│       │       │       └── styles.css
│       │       └── templates
│       │           ├── index.html
│       │           └── users
│       │               └── login.html
│       └── main.go
├── go.mod
├── go.sum
├── pkg
│   ├── database
│   │   ├── database.go
│   │   └── model.go
│   └── user
│       └── model.go
└── README.md
```

To serve the html files using golang we should use the `html/template` package, now after accessing an endpoint and we wanted to serve an html page, what we do is we first parse the file, then simply execute the template/parsed-file, the following code will accomplish what we need:

```
r.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
    tmpl, err := template.ParseFiles("path/to/file.html")
    if err != {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }

    tmpl.Execute(w, nil)
})
```
now like most webpages they come styled and unless we add the css styles in the `<style></style>` tags, but if you want to write a `.css` file and call it using the `<link>` tag, it wont work, yet there is a way where we can serve css files, and thats by writing the following code:

```
staticPath := filepath.Join("internal", "web", "static")
fs := http.FileServer(http.Dir(staticPath))
r.PathPrefix("/static/").Handler(http.StripPrefix("/static/", fs))
```

now that the code above is added in the `main()` function the `<link>` tag will work as it should.
