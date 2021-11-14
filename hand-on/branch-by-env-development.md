# Branch by Environments Development

1. `Production Environment`: `main branch`
2. `UAT Environment`: `uat branch`
3. `Local Machine` and `DEV Environment`: `dev branch`

---

## Steps

1. Create git repository call `hello-api`

   ```sh
   git init hello-api
   cd hello-api
   ```

2. Create `dev` branch
3. Checkout to `dev`
4. `On dev`: Create `Hello api/v1`

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

5. `On dev`: Build artifacts[windows, linux, mac]

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

6. `On dev`: Run and Test

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

   2. Test hello-api

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

7. Commit
8. Create remote repository
9. Add team members to remote repository
10. Add remote repository
11. `On dev`: Push dev branch to remote repository
12. `On main`: Team members pull source code, then change to dev branch
13. `On dev`: Create and Checkout to `uat` branch
14. `On uat`: Merge `dev` to `uat` branch
15. `On uat`: Push `uat` branch to remote repository
16. Team members pull source code, then change to `uat` branch
17. Choose a branch to work with by responsibility
18. `On dev`: Refactor inline func

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

      ```sh
      sh test-v1
      ```

    - Commit
    - Push to remote repository

19. Found a bug on uat
20. `On dev`: Create Hello api/v2

    1. Add following lines at the end of `main.go`

       ```go
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
       import (
           ...
           "encoding/json"
       )

       func main() {
           router.Get("/api/v1/greeting", HelloText)

           router.Post("/api/v2/greeting", HelloJSON)
           http.ListenAndServe(":3000", router)
       }
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

      ```sh
      sh test-v1
      sh test-v2
      ```

    - Commit
    - Push to remote repository

21. `On uat`: Fixed a bug "append `!` to message" on uat

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

22. Update test script

    ```sh
    echo "echo expected: \n" > test-v1
    echo "echo Hello boyone\!\n" >> test-v1
    echo "echo got:" >> test-v1
    echo "curl http://localhost:3000/api/v1/greeting?name=boyone" >> test-v1
    ```

    - Build artifacts
    - Regression tests

      ```sh
      sh test-v1
      ```

    - Commit
    - Push to remote repository

23. `On dev`: `Cherry Pick` the fixed bug commit to dev

    ```sh
    git cherry-pick <Commit ID>
    ```

    - Solve conflicts
    - Update test-v2 script

      ```sh
      echo "echo expected: \n" > test-v2
      echo 'echo {\"message\":\"Hello boyone!\"}\n' >> test-v2
      echo "echo got:" >> test-v2
      echo "curl http://localhost:3000/api/v2/greeting -X POST --data-raw '{ \"name\": \"boyone\" }'" >> test-v2
      ```

    - Build artifacts
    - Regression tests

      ```sh
      sh test-v1
      sh test-v2
      ```

    - Commit
    - Push to remote repository

24. `On main`: Merge `uat` to `main`

    - Regression tests

      ```sh
      curl http://localhost:3000/api/v1/greeting?name=boyone
      ```

    - Push to remote repository

25. `On dev`: Move func HelloText and func HelloJSON to greeting package

    ```sh
    mkdir greeting
    touch greeting.go
    echo "package greeting" > greeting/greeting.go
    ```

    - Copy func HelloText and func HelloJSON to greeting/greeting.go
    - Add import greeting and change HelloText and HelloJSON func in func main

      ```go
      import (
          "hello-api/greeting"
          ...
      )

      func main() {
        router.Get("/api/v1/greeting", greeting.HelloText)

        router.Post("/api/v2/greeting", greeting.HelloJSON)
      }
      ```

    - Delete func HelloText and func HelloJSON in main.go
    - Build artifacts
    - Regression tests

      ```sh
      sh test-v1
      sh test-v2
      ```

    - Commit
    - Push to remote repository

26. Found a bug on main
27. `On main`: Fixed a bug "add `,` after Hello" on main

    ```go
    func HelloText(w http.ResponseWriter, r *http.Request) {
         name := r.FormValue("name")
         fmt.Fprintf(w, "Hello, %s!", name)
    }
    ```

    - Update test script

      ```sh
      echo "echo expected: \n" > test-v1
      echo "echo Hello, boyone\!\n" >> test-v1
      echo "echo got:" >> test-v1
      echo "curl http://localhost:3000/api/v1/greeting?name=boyone" >> test-v1
      ```

    - Build artifacts
    - Regression tests

      ```sh
      sh test-v1
      ```

    - Commit
    - Push to remote repository

28. `On uat`: `Cherry Pick` the fixed bug commit fixed bug to `uat`

    - Solve conflicts
    - Build artifacts
    - Regression tests

      ```sh
      sh test-v1
      ```

    - Commit
    - Push to remote repository

29. `On dev`: `Cherry Pick` the fixed bug commit fixed bug to `dev`

    - Solve conflicts
    - Update test-v2 script

      ```sh
      echo "echo expected: \n" > test-v2
      echo 'echo {\"message\":\"Hello, boyone!\"}\n' >> test-v2
      echo "echo got:" >> test-v2
      echo "curl http://localhost:3000/api/v2/greeting -X POST --data-raw '{ \"name\": \"boyone\" }'" >> test-v2
      ```

    - Build artifacts
    - Regression tests

      ```sh
      sh test-v1
      sh test-v2
      ```

    - Commit
    - Push to remote repository

30. `On uat`: Merge `dev` to `uat`

    - Solve conflicts
    - Build artifacts
    - Regression tests

      ```sh
      sh test-v1
      sh test-v2
      ```

    - Commit
    - Push to remote repository
