```
Awesome — here’s your starter repo layout for a reproducible Go API project using Nix flakes. This is the clean launchpad you can clone, build, and expand.

📁 Directory Structure
go-basic/
 ├─ flake.nix
 ├─ go.mod
 └─ main.go



📝 flake.nix
{
  description = "Go API with pinned Go version and reproducible build";

  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-24.05";
    flake-utils.url = "github:numtide/flake-utils";
  };

  outputs = { self, nixpkgs, flake-utils }:
    flake-utils.lib.eachDefaultSystem (system:
      let
        pkgs = import nixpkgs { inherit system; };
      in {
        packages.default = pkgs.buildGoModule {
          pname = "go-basic";
          version = "0.1.0";
          src = ./.;
          vendorSha256 = "sha256-0000000000000000000000000000000000000000000000000000"; # replace after first build
          go = pkgs.go_1_22;
        };

        devShells.default = pkgs.mkShell {
          buildInputs = [ pkgs.go_1_22 pkgs.gopls pkgs.git ];
        };
      });
}



🧑‍💻 go.mod
module go-basic

go 1.22



🚀 main.go
package main

import (
	"fmt"
	"log"
	"net/http"
)

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



🧪 How to Use
- Enter the dev shell:
nix develop
- Run the server:
go run main.go
- Test the endpoint:
curl http://localhost:8080/hello
- Build with Nix:
nix build


- → Copy the suggested vendorSha256 and replace the placeholder in flake.nix.
Would you like me to scaffold a second endpoint (e.g., /status returning JSON) and show how to add external packages like github.com/gin-gonic/gin so you can test vendoring and expand your API? That would be a great next layer for your lab.
```
