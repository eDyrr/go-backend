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