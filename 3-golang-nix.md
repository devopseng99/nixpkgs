```
{
  description = "Basic Go dev environment with version pinned";

  inputs = {
    # Pin nixpkgs to a specific commit for reproducibility
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-24.05";
    flake-utils.url = "github:numtide/flake-utils";
  };

  outputs = { self, nixpkgs, flake-utils }:
    flake-utils.lib.eachDefaultSystem (system:
      let
        pkgs = import nixpkgs { inherit system; };
      in {
        # Build a Go project with Go 1.22
        packages.default = pkgs.buildGoModule {
          pname = "go-basic";
          version = "0.1.0";
          src = ./.;
          vendorSha256 = null; # fill in after first nix build
          go = pkgs.go_1_22;   # explicitly use Go 1.22
        };

        # Dev shell with Go 1.22
        devShells.default = pkgs.mkShell {
          buildInputs = [
            pkgs.go_1_22
            pkgs.gopls
            pkgs.git
          ];
        };
      });
}
```

🚀 Usage
- Enter the dev shell:
```
nix develop
```
- → You’ll have Go 1.22, gopls, and git.
- Run your Go program:
```
go run main.go
```
- Build with Nix:
```
nix build
./result/bin/go-basic
```
🔑 Key Notes- pkgs.go_1_22 locks the Go version to 1.22.
- Pinning nixpkgs to nixos-24.05 ensures the toolchain doesn’t drift.
- Replace vendorSha256 = null with the hash Nix suggests after your first build.
Do you want me to scaffold a multi-version flake next, where you can switch between Go 1.21 and 1.22 dev shells for testing compatibility? That would give you a modular lab setup aligned with your iterative style.
