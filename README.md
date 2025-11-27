# Steps: NIX
Install Nix:
```
curl -L https://nixos.org/nix/install | sh
```
Create a flake.nix:
```
{
  description = "Python 3.14 dev shell";

  inputs.nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";

  outputs = { self, nixpkgs }: let
    system = "x86_64-linux";
    pkgs = import nixpkgs { inherit system; };
  in {
    devShell.${system} = pkgs.mkShell {
      buildInputs = [
        pkgs.python314
        pkgs.python314Packages.virtualenv
        pkgs.zlib
        pkgs.openssl
        pkgs.libffi
      ];
    };
  };
}

```

Fix
```
sudo localedef -i en_US -f UTF-8 en_US.UTF-8
export LC_ALL=en_US.UTF-8

# You can also add this to your .bashrc or .profile:
export LANG=en_US.UTF-8
export LC_ALL=en_US.UTF-8
```



🧱 Next Steps for Your Python 3.14 Nix Shell
Now that your shell is building, here’s how to scaffold it into a branded dev lab overlay:
1. Pin Python + Cython
Your current build includes python3.14-cython-3.1.4. You can extend this with:
```
buildInputs = [
  pkgs.python314
  pkgs.python314Packages.cython
  pkgs.python314Packages.virtualenv
  pkgs.zlib pkgs.openssl pkgs.libffi
];
```

2. Add Overlay for Custom Packages
If you want to patch or override packages (e.g., a custom uvloop or zstandard), use:

```
overlays = [
  (final: prev: {
    python314 = prev.python314.override {
      packageOverrides = pyFinal: pyPrev: {
        zstandard = pyPrev.zstandard.overridePythonAttrs (old: {
          version = "0.22.0";
        });
      };
    };
  })
];
```

3. Expose Scripts or Entry Points
You can add a shellHook to run setup commands:
```
shellHook = ''
  echo "Welcome to py314 dev shell"
  source ./venv314/bin/activate || true
'';
```

4. Brand Your Flake
Use expressive naming:
```
- flake.nix → flake-py314.nix
- description = "VORTEX Python 3.14 shell with Cython and async overlays"
```

🌀 Optional: Multi-language Expansion
Want to scaffold a polyglot dev shell with Python 3.14, Node.js, and Go?
```
buildInputs = [
  pkgs.python314
  pkgs.nodejs_20
  pkgs.go_1_21
];
```


```mermaid
flowchart TD
    A[flake.nix]
    A --> B[inputs]
    B --> B1[nixpkgs → github:NixOS/nixpkgs]
    B --> B2[system → x86_64-linux]

    A --> C[outputs]
    C --> D[devShell.x86_64-linux]
    D --> E[pkgs.mkShell]
    E --> F[buildInputs]
    F --> F1[python314]
    F --> F2[cython]
    F --> F3[virtualenv]
    F --> F4[zlib, openssl, libffi]

    E --> G[shellHook]
    G --> H["echo Welcome to py314 shell"]
    G --> I["source ./venv314/bin/activate"]

    A --> J[overlays (optional)]
    J --> J1[override zstandard version]

```




 Test it out
 ```
nix --extra-experimental-features nix-command \
    --extra-experimental-features flakes \
    develop
```

You’ll drop into a shell with Python 3.14 and virtualenv ready to go. From there:
```
python3.14 -m venv venv314
source venv314/bin/activate
```



Enter the shell:
```
nix develop
```

==========

# Steps: CONDA
Install Miniconda:
```
curl -LO https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
```

Create a Python 3.14 environment:
```
conda create -n py314 python=3.14
conda activate py314
```

Install extras:
```
conda install zstandard uvloop rich
```


## does nix-python behave different from regular python
🧊 How Nix Python Differs
- Reproducibility
- Nix pins exact versions of Python and all its dependencies (OpenSSL, zlib, libffi, etc.).
- Regular Python builds (system packages, source, Conda) can drift depending on what’s installed on the host.
- With Nix, if you declare python314 in a flake, every developer and CI runner gets the same binary and linked libraries.
- Isolation
- Nix installs Python into its own store path (/nix/store/...) rather than /usr/bin or /opt.
- This avoids conflicts with system Python and makes multiple versions coexist cleanly.
- Regular Python often requires manual prefixing (/opt/python/3.14) or virtualenvs to avoid collisions.
- Dependency Management
- Nix treats Python packages as derivations: each wheel or source build is reproducible and cached.
- Regular Python relies on pip/venv, which can vary depending on network state or PyPI changes.
- Environment Activation
- Nix shells (nix develop) inject Python into your PATH with all declared dependencies.
- Regular Python uses venv or Conda activate scripts to adjust PATH and environment variables.

🚀 Runtime Behavior
- Interpreter: Once inside a Nix shell, python3.14 behaves identically to a source‑built Python 3.14 — same syntax, same stdlib, same REPL.
- Modules: The difference is in where modules are loaded from (Nix store vs site‑packages).
- Performance: No runtime penalty — Nix doesn’t change CPython internals, just how it’s built and distributed.
- Compatibility: Scripts and apps don’t care whether Python came from Nix or a tarball, as long as the version matches.

🔑 When to Use Nix Python
- You want reproducibility across machines (CI/CD, dev labs, onboarding).
- You need multiple Python versions side by side without conflicts.
- You want declarative builds that can be version‑controlled and cached.
If you just need a quick isolated interpreter, a source build or Conda is fine. But for your modular lab scaffolding, Nix Python fits beautifully because it aligns with your blueprint‑driven, reproducible style.


```mermaid
flowchart LR
    subgraph PY311[Python 3.11 Build Flow]
        A1[Download Python-3.11.x.tgz]
        B1[Configure --enable-optimizations --with-lto --enable-shared]
        C1[make altinstall → /opt/python/3.11]
        D1[Create venv → /opt/venvs/py311]
        E1[Validate ssl, sqlite3, bz2, lzma, uuid]
    end

    subgraph PY314[Python 3.14 Build Flow]
        A2[Download Python-3.14.0.tgz]
        B2[Configure --enable-optimizations --with-lto --enable-shared + rpath flags]
        C2[make altinstall → /opt/python/3.14]
        D2[Create venv → /opt/venvs/py314]
        E2[Validate ssl, sqlite3, bz2, lzma, uuid, zstd]
        F2[Test multi-interpreter + free-threaded mode]
    end

    A1 --> B1 --> C1 --> D1 --> E1
    A2 --> B2 --> C2 --> D2 --> E2 --> F2
```




