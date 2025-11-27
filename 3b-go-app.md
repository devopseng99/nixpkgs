```go
package main

import (
	"fmt"
	"log"
	"net/http"
)

// simple handler
func helloHandler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello SkyD! Your Go API is alive 🚀")
}

func main() {
	http.HandleFunc("/hello", helloHandler)

	port := ":8080"
	log.Printf("Starting server on %s\n", port)
	if err := http.ListenAndServe(port, nil); err != nil {
		log.Fatal(err)
	}
}
```

🚀 How to Test
- Enter your dev shell (with Go pinned via Nix):
```
nix develop
```
- Run the server:
```
go run main.go
```
- → You’ll see Starting server on :8080.
- Hit the API:
```
curl http://localhost:8080/hello
```

- → Response:
Hello SkyD! Your Go API is alive 🚀
🔑 Key Notes- This uses only the Go standard library (net/http), so no external dependencies.
- Perfect for testing your Nix flake build pipeline — you’ll get a binary in ./result/bin/go-basic after nix build.
- You can expand later with routes, JSON responses, or external packages (then Nix will require a vendorSha256 lock).
Would you like me to expand this sample into a JSON API (e.g., /status returning { "ok": true }) so you can test structured responses and see how Nix handles vendoring when you add external packages?
