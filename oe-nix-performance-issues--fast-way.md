```
Totally fair — Nix builds can feel glacial the first time, especially when pulling large Python packages like cython, zstandard, and uvloop. Let’s break down what’s happening and how to speed it up:

🐢 Why It’s Slow Right Now
- Cold cache: Nix is downloading and building everything from scratch — Python 3.14, its dependencies, and all requested packages.
- Large closures: Python packages often pull in compilers, C libraries, and build tools (e.g., gcc, libffi, openssl, etc.).
- No substitutes: If your system isn’t connected to a binary cache (or the cache doesn’t have your exact build), Nix builds locally.
- Amazon Linux 2 quirks: It’s not a native NixOS system, so some builds may be slower or less optimized.

⚡️ How to Speed It Up
1. Use a pinned nixpkgs with binary cache hits
Use a known commit from nixos-unstable that has prebuilt Python 3.14 packages:
inputs.nixpkgs.url = "github:NixOS/nixpkgs/5ae3b07d8d6527c42f17c876e404993199144b6a";


This matches the hash from your earlier flake lock — it’s likely to hit the cache.

2. Strip down buildInputs temporarily
Start with just:
buildInputs = [ pkgs.python314 pkgs.python314Packages.virtualenv ];


Once the shell works, incrementally add cython, zstandard, uvloop, etc.

3. Use nix develop --impure for faster local testing
This skips some purity checks and can reuse host libraries:
nix develop --impure



4. Enable experimental binary cache (if not already)
Make sure you’re using:
nix --extra-experimental-features nix-command --extra-experimental-features flakes develop


And check your /etc/nix/nix.conf or ~/.config/nix/nix.conf for:
substituters = https://cache.nixos.org
trusted-public-keys = cache.nixos.org-1:...



5. Use nix build instead of nix develop for caching
Once your shell works, you can cache it with:
nix build .#devShells.x86_64-linux.default


This will prebuild the shell and speed up future nix develop calls.

🧠 Want a minimal flake just to bootstrap Python 3.14 instantly?
I can scaffold a “barebones” flake that gives you Python 3.14 in <30 seconds, then layer in your packages once the shell is warm. Just say the word and I’ll drop it in.

```
