# GolangCRUD

1. Langkah pertama
    Sistem ini berjalan pada macOs, sebelum melanjutkan silahkan cek terlebih dahulu apakah golang sudah diinstall atau belum, 
    jika belum ikuti langkah ini

    buka terminal
    ```
        /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
    ```

    buka dengan perintah nano ~/.zprofile pada terminal seleah itu copy code dibawah

    ```
    eval "$(/opt/homebrew/bin/brew shellenv)"
    ```
    simpan dengan control+x, langakah selanjutnya 
    ```
    source ~/.zprofile
    ```

    langkah selanjutnya install golang
    ```
    brew install golang
    ```
    seleteh selesai cek version
    ```
    go version
    ```

2. langkah kedua 
    
    a. Siapkan dan Impor driver MySQL ke proyek Anda

    Menggunakan Git Bash, pertama-tama instal driver untuk paket database MySQL Go. Jalankan perintah di bawah ini dan instal driver MySQL
    ```
    go get -u github.com/go-sql-driver/mysql
    ```

    b. buat database dimysql dengan sesuai yang diinginkan, jika sudah buat table dengan nama employee atau menggunakan script dibawah
    ```
    CREATE TABLE `employee` (
    `id` int(6) unsigned NOT NULL AUTO_INCREMENT,
    `name` varchar(30) NOT NULL,
    `city` varchar(30) NOT NULL,
    PRIMARY KEY (`id`)
    ) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=latin1;
    ```

3. Membuat Struct, Handler dan Fungsi Handler

    Mari kita buat file bernama main.go dan isi dengan script dibawah
    ```
    package main

    import (
        "database/sql"
        "log"
        "net/http"
        "text/template"

        _ "github.com/go-sql-driver/mysql"
    )

    type Employee struct {
        Id    int
        Name  string
        City string
    }

    func dbConn() (db *sql.DB) {
        dbDriver := "mysql"
        dbUser := "isi_dengan_user_database"
        dbPass := "isi_dengan_password_database"
        dbName := "isi_dengan_nama_database"
        db, err := sql.Open(dbDriver, dbUser+":"+dbPass+"@/"+dbName)
        if err != nil {
            panic(err.Error())
        }
        return db
    }

    var tmpl = template.Must(template.ParseGlob("form/*"))

    func Index(w http.ResponseWriter, r *http.Request) {
        db := dbConn()
        selDB, err := db.Query("SELECT * FROM Employee ORDER BY id DESC")
        if err != nil {
            panic(err.Error())
        }
        emp := Employee{}
        res := []Employee{}
        for selDB.Next() {
            var id int
            var name, city string
            err = selDB.Scan(&id, &name, &city)
            if err != nil {
                panic(err.Error())
            }
            emp.Id = id
            emp.Name = name
            emp.City = city
            res = append(res, emp)
        }
        tmpl.ExecuteTemplate(w, "Index", res)
        defer db.Close()
    }

    func Show(w http.ResponseWriter, r *http.Request) {
        db := dbConn()
        nId := r.URL.Query().Get("id")
        selDB, err := db.Query("SELECT * FROM Employee WHERE id=?", nId)
        if err != nil {
            panic(err.Error())
        }
        emp := Employee{}
        for selDB.Next() {
            var id int
            var name, city string
            err = selDB.Scan(&id, &name, &city)
            if err != nil {
                panic(err.Error())
            }
            emp.Id = id
            emp.Name = name
            emp.City = city
        }
        tmpl.ExecuteTemplate(w, "Show", emp)
        defer db.Close()
    }

    func New(w http.ResponseWriter, r *http.Request) {
        tmpl.ExecuteTemplate(w, "New", nil)
    }

    func Edit(w http.ResponseWriter, r *http.Request) {
        db := dbConn()
        nId := r.URL.Query().Get("id")
        selDB, err := db.Query("SELECT * FROM Employee WHERE id=?", nId)
        if err != nil {
            panic(err.Error())
        }
        emp := Employee{}
        for selDB.Next() {
            var id int
            var name, city string
            err = selDB.Scan(&id, &name, &city)
            if err != nil {
                panic(err.Error())
            }
            emp.Id = id
            emp.Name = name
            emp.City = city
        }
        tmpl.ExecuteTemplate(w, "Edit", emp)
        defer db.Close()
    }

    func Insert(w http.ResponseWriter, r *http.Request) {
        db := dbConn()
        if r.Method == "POST" {
            name := r.FormValue("name")
            city := r.FormValue("city")
            insForm, err := db.Prepare("INSERT INTO Employee(name, city) VALUES(?,?)")
            if err != nil {
                panic(err.Error())
            }
            insForm.Exec(name, city)
            log.Println("INSERT: Name: " + name + " | City: " + city)
        }
        defer db.Close()
        http.Redirect(w, r, "/", 301)
    }

    func Update(w http.ResponseWriter, r *http.Request) {
        db := dbConn()
        if r.Method == "POST" {
            name := r.FormValue("name")
            city := r.FormValue("city")
            id := r.FormValue("uid")
            insForm, err := db.Prepare("UPDATE Employee SET name=?, city=? WHERE id=?")
            if err != nil {
                panic(err.Error())
            }
            insForm.Exec(name, city, id)
            log.Println("UPDATE: Name: " + name + " | City: " + city)
        }
        defer db.Close()
        http.Redirect(w, r, "/", 301)
    }

    func Delete(w http.ResponseWriter, r *http.Request) {
        db := dbConn()
        emp := r.URL.Query().Get("id")
        delForm, err := db.Prepare("DELETE FROM Employee WHERE id=?")
        if err != nil {
            panic(err.Error())
        }
        delForm.Exec(emp)
        log.Println("DELETE")
        defer db.Close()
        http.Redirect(w, r, "/", 301)
    }

    func main() {
        log.Println("Server started on: http://localhost:8080")
        http.HandleFunc("/", Index)
        http.HandleFunc("/show", Show)
        http.HandleFunc("/new", New)
        http.HandleFunc("/edit", Edit)
        http.HandleFunc("/insert", Insert)
        http.HandleFunc("/update", Update)
        http.HandleFunc("/delete", Delete)
        http.ListenAndServe(":8080", nil)
    }
    ```
4. Membuat file Template

    buat folder dengan nama form, seletah itu buat file dengan nama Index.tmpl, Header.tmpl, Footer.tmpl, Menu.tmpl, Show.tmpl, New.tmpl dan Edit.tmpl, isi masing file tersebut dengan script dibawah

    Index.tmpl
    ```
    {{ define "Index" }}
    {{ template "Header" }}
        {{ template "Menu"  }}
        <h2> Registered </h2>
        <table border="1">
        <thead>
        <tr>
            <td>ID</td>
            <td>Name</td>
            <td>City</td>
            <td>View</td>
            <td>Edit</td>
            <td>Delete</td>
        </tr>
        </thead>
        <tbody>
        {{ range . }}
        <tr>
            <td>{{ .Id }}</td>
            <td> {{ .Name }} </td>
            <td>{{ .City }} </td> 
            <td><a href="/show?id={{ .Id }}">View</a></td>
            <td><a href="/edit?id={{ .Id }}">Edit</a></td>
            <td><a href="/delete?id={{ .Id }}">Delete</a><td>
        </tr>
        {{ end }}
        </tbody>
        </table>
    {{ template "Footer" }}
    {{ end }}
    ```

    Header.tmpl
    ```
    {{ define "Header" }}
    <!DOCTYPE html>
    <html lang="en-US">
        <head>
            <title>Golang Mysql Curd Example</title>
            <meta charset="UTF-8" />
        </head>
        <body>
            <h1>Golang Mysql Curd Example</h1>   
    {{ end }}
    ```
    Footer.tmpl
    ```
    {{ define "Footer" }}
        </body>
    </html>
    {{ end }}
    ```
    Menu.tmpl
    ```
    {{ define "Menu" }}
        <a href="/">HOME</a> | 
        <a href="/new">NEW</a>
    {{ end }}

    ```
    Show.tmpl
    ```
    {{ define "Show" }}
    {{ template "Header" }}
        {{ template "Menu"  }}
        <h2> Register {{ .Id }} </h2>
        <p>Name: {{ .Name }}</p>
        <p>City:  {{ .City }}</p><br /> <a href="/edit?id={{ .Id }}">Edit</a></p>
    {{ template "Footer" }}
    {{ end }}
    ```
    New.tmpl
    ```
    {{ define "New" }}
    {{ template "Header" }}
        {{ template "Menu" }} 
    <h2>New Name and City</h2>  
        <form method="POST" action="insert">
        <label> Name </label><input type="text" name="name" /><br />
        <label> City </label><input type="text" name="city" /><br />
        <input type="submit" value="Save user" />
        </form>
    {{ template "Footer" }}
    {{ end }}
    ```
    Edit.tmpl
    ```
    {{ define "Edit" }}
    {{ template "Header" }}
        {{ template "Menu" }} 
    <h2>Edit Name and City</h2>  
        <form method="POST" action="update">
        <input type="hidden" name="uid" value="{{ .Id }}" />
        <label> Name </label><input type="text" name="name" value="{{ .Name }}"  /><br />
        <label> City </label><input type="text" name="city" value="{{ .City }}"  /><br />
        <input type="submit" value="Save user" />
        </form><br />    
    {{ template "Footer" }}
    {{ end }}
    ```

5. langkah selanjutnya manjalankan atau bahasa kerennya run donk dengan cara.
    ```
    go run main.go
    ```
    nanti akan tampil di terminal 
    ```
    http://localhost:8080/
    ```
 

    Struktur file

    ![Alt text](/img1.png)

    Hasilnya

    ![Alt text](/img2.png)