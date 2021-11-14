# Trunk Based Development

1. `Production`, `UAT`, `DEV` Environment, and `Local Machine` working on `main branch`

## Steps

1. Create git repository call `hello-api`

   ```sh
   git init hello-api
   cd hello-api
   ```

2. Create `Hello api/v1`

   1. Create file call go.mod

      ```sh
      touch go.mod
      ```

   2. Add following content to `go.mod`

      ```txt
      module hello-api

      go 1.17

      require github.com/go-chi/chi v1.5.4
      ```

   3. Create file call `go.sum`

      ```sh
      touch go.mod
      ```

   4. Add following content to `go.sum`

      ```txt
      github.com/go-chi/chi v1.5.4 h1:QHdzF2szwjqVV4wmByUnTcsbIg7UGaQ0tPF2t5GcAIs=
      github.com/go-chi/chi v1.5.4/go.mod h1:uaf8YgoFazUOkPBG7fxPftUylNumIev9awIWOENIuEg=
      ```

   5. Create file call `main.go`

      ```sh
      touch main.go
      ```

   6. Add following lines to `main.go`

      ```go
      package main

      import (
          "fmt"
          "net/http"

          "github.com/go-chi/chi"
      )

      func main() {
          router := chi.NewRouter()

          router.Get("/api/v1/greeting", func(w http.ResponseWriter, r *http.Request) {
              name := r.FormValue("name")
              fmt.Fprintf(w, "Hello %s", name)
      })

          http.ListenAndServe(":3000", router)
      }
      ```

3. Build artifacts[windows, linux, mac]

   1. windows

      ```sh
      GOARCH=amd64 GOOS=windows CGO_ENABLE=0 go build -o bin/windows/app.exe
      ```

   2. linux

      ```sh
      GOARCH=amd64 GOOS=linux CGO_ENABLE=0 go build -o bin/linux/app
      ```

   3. mac

      ```sh
      GOARCH=amd64 GOOS=darwin CGO_ENABLE=0 go build -o bin/mac/app
      ```

4. Run and Test

   1. Run hello-api

      ```sh
      # windows
      bin/windows/app.exe
      ```

      ```sh
      # linux
      bin/linux/app
      ```

      ```sh
      # mac
      bin/mac/app
      ```

   2. Test hello-api.v1

      ```sh
      curl http://localhost:3000/api/v1/greeting?name=boyone
      ```

      - Create file call `test-v1`

        ```sh
        touch test-v1
        ```

      - Add following content `test-v1`

        ```sh
        echo "echo expected: \n" > test-v1
        echo "echo Hello boyone\n" >> test-v1
        echo "echo got:" >> test-v1
        echo "curl http://localhost:3000/api/v1/greeting?name=boyone" >> test-v1
        ```

      - Run test

        ```sh
        sh test-v1
        ```

5. Create remote repository
6. Add team members to remote repository
7. Add remote repository
8. Push to remote repository
9. Team members pull source code
10. Choose an environment to work with by responsibility
11. Refactor inline func

    1. Add following lines at the end of `main.go`

       ```go
       func main() {
           ...
       }

       func HelloText(w http.ResponseWriter, r *http.Request) {
            name := r.FormValue("name")
            fmt.Fprintf(w, "Hello %s", name)
       }
       ```

    2. Change `inline` func to `HelloText`

       ```go
       func main() {
           router := chi.NewRouter()

           router.Get("/api/v1/greeting", HelloText)
       }
       ```

    - Build artifacts
    - Regression tests
    - Push to remote repository

12. Release and Found a bug on `UAT`
13. Create Hello api/v2

    1. Add following lines at the end of `main.go`

       ```go
       import (
           "encoding/json"
           ...
       )

       func main() {
           ...
       }

       func HelloText(w http.ResponseWriter, r *http.Request) {
           ...
       }

       func HelloJSON(w http.ResponseWriter, r *http.Request) {
            name := struct {
                Name string `json:"name,omitempty"`
            }{}
            json.NewDecoder(r.Body).Decode(&name)

            greeting := struct {
                Message string `json:"message,omitempty"`
            }{fmt.Sprintf("Hello %s", name.Name)}

            w.Header().Set("Content-Type", "application/json")
            w.WriteHeader(http.StatusOK)
            json.NewEncoder(w).Encode(&greeting)
       }
       ```

    2. Add following lines to `main.go`

       ```go
       // flags()
       router.Get("/api/v1/greeting", HelloText)
       // add check flags
       router.Post("/api/v2/greeting", HelloJSON)
       http.ListenAndServe(":3000", router)
       ```

    3. Add test hello-api.v2

    ```sh
    curl http://localhost:3000/api/v1/greeting?name=boyone
    ```

    - Create file call `test-v2`

      ```sh
      touch test-v2
      ```

    - Add following content `test-v2`

      ```sh
      echo "echo expected: \n" > test-v2
      echo 'echo {\"message\":\"Hello boyone\"}\n' >> test-v2
      echo "echo got:" >> test-v2
      echo "curl http://localhost:3000/api/v2/greeting -X POST --data-raw '{ \"name\": \"boyone\" }'" >> test-v2
      ```

    - Run test

      ```sh
      sh test-v2
      ```

    - Build artifacts
    - Regression tests
    - Push to remote repository

14. Fixed a bug "append `!` to message"

    ```go
    func HelloText(w http.ResponseWriter, r *http.Request) {
         name := r.FormValue("name")
         fmt.Fprintf(w, "Hello %s!", name)
    }

    func HelloJSON(w http.ResponseWriter, r *http.Request) {
         ...
         greeting := struct {
             Message string `json:"message,omitempty"`
         }{fmt.Sprintf("Hello %s!", name.Name)}
         ...
     }
    ```

15. Add flag for specific run

    ```go
    import (
        "encoding/json"
        "flag"
        ...
    )

    func main() {
        var dev = flag.Bool("dev", false, "flag for dev")
        flag.Parse()
        ...

        router.Get("/api/v1/greeting", HelloText)

        if *dev {
            router.Post("/api/v2/greeting", HelloJSON)
        }
        ...
     }
    ```

16. Update test script

    ```sh
    echo "echo expected: \n" > test-v1
    echo "echo Hello boyone\!\n" >> test-v1
    echo "echo got:" >> test-v1
    echo "curl http://localhost:3000/api/v1/greeting?name=boyone" >> test-v1

    echo "echo expected: \n" > test-v2
    echo 'echo {\"message\":\"Hello boyone!\"}\n' >> test-v2
    echo "echo got:" >> test-v2
    echo "curl http://localhost:3000/api/v2/greeting -X POST --data-raw '{ \"name\": \"boyone\" }'" >> test-v2
    ```

    - Build artifacts
    - Regression tests

      ```sh
      #run with flags `dev`
      app -dev=true
      ```

    - Push to remote repository

17. Release on `UAT`
    - Regression tests on `UAT` // run without flags
18. Release on `Production`
    - Regression tests on `Production` // run without flags
19. Move HelloText and HelloJSON func to greeting package

    ```sh
    mkdir greeting
    touch greeting.go
    echo "package greeting" > greeting.go
    ```

    - Copy func HelloText and func HelloJSON to greeting/greeting.go
    - Add import greeting and change HelloText and HelloJSON func in func main

      ```go
      import (
          ...
          greeting
      )

      func main() {
        router.Get("/api/v1/greeting", greeting.HelloText)

        router.Post("/api/v2/greeting", greeting.HelloJSON)
      }
      ```

    - Delete func HelloText and func HelloJSON in main.go
    - Build artifacts
    - Regression tests for `DEV` // run with flags `dev`
    - Push to remote repository
    - Regression tests for `UAT` // run without flags
    - Regression tests `Production` // run without flags

20. Found a bug on `Production`

21. Fixed a bug "add `,` after Hello"

    ```go
    func HelloText(w http.ResponseWriter, r *http.Request) {
         name := r.FormValue("name")
         fmt.Fprintf(w, "Hello, %s!", name)
    }

    func HelloJSON(w http.ResponseWriter, r *http.Request) {
         ...
         greeting := struct {
             Message string `json:"message,omitempty"`
         }{fmt.Sprintf("Hello, %s!", name.Name)}
         ...
     }
    ```

22. Update test script

    ```sh
    echo "echo expected: \n" > test-v1
    echo "echo Hello, boyone\!\n" >> test-v1
    echo "echo got:" >> test-v1
    echo "curl http://localhost:3000/api/v1/greeting?name=boyone" >> test-v1

    echo "echo expected: \n" > test-v2
    echo 'echo {\"message\":\"Hello, boyone!\"}\n' >> test-v2
    echo "echo got:" >> test-v2
    echo "curl http://localhost:3000/api/v2/greeting -X POST --data-raw '{ \"name\": \"boyone\" }'" >> test-v2
    ```

    - Build artifacts
    - Regression tests for `DEV` // run with flags `dev`
    - Push to remote repository
    - Regression tests for `UAT` // run without flags
    - Regression tests `Production` // run without flags

23. Release /api/v2 on `UAT`

    1. Add flag `uat` for specific run

       ```go
       import (
           "encoding/json"
           "flag"
           ...
       )

       func main() {
           var dev = flag.Bool("dev", false, "flag for dev")
           var uat = flag.Bool("uat", false, "flag for dev")
           flag.Parse()
           ...

           router.Get("/api/v1/greeting", HelloText)

           if *dev || *uat {
               router.Post("/api/v2/greeting", HelloJSON)
           }
           ...
       }
       ```

    - Build artifacts
    - Regression tests on `DEV` // run with flags `dev`
    - Push to remote repository
    - Regression tests on `UAT` // run with flags `uat`
    - Regression tests on `Production` // run without flags
