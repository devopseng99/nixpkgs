```
Isolated Python 3.14 build on Amazon Linux 2
This guide compiles and installs Python 3.14 from source into an isolated prefix (e.g., /opt/python/3.14) on Amazon Linux 2, without touching the system Python. It ends with a self-contained virtual environment ready for CI/CD and automation.

Prerequisites on Amazon Linux 2
- Update packages:
sudo yum update -y
- Build tools:
sudo yum groupinstall -y "Development Tools"
- Libraries for CPython build:
sudo yum install -y \
  openssl-devel bzip2-devel libffi-devel zlib-devel \
  readline-devel sqlite-devel xz-devel ncurses-devel tk-devel \
  gdbm-devel uuid-devel libuuid-devel


Tip: A Python 3 virtual environment on AL2 is supported and recommended; we’ll finish with venv for isolation.


Download and verify Python 3.14 source
- Fetch the release tarball (adjust version if needed):
PYVER=3.14.0
curl -LO https://www.python.org/ftp/python/${PYVER}/Python-${PYVER}.tgz
tar xzf Python-${PYVER}.tgz
cd Python-${PYVER}
- Optionally verify signatures with the Python release keys (skip if you don’t use GPG).

Configure for an isolated prefix
- Choose a dedicated install path:
PREFIX=/opt/python/3.14
sudo mkdir -p $PREFIX
sudo chown $(id -u):$(id -g) $PREFIX
- Configure with performance optimizations, shared lib, and system libs:
./configure \
  --prefix=$PREFIX \
  --enable-optimizations \
  --with-lto \
  --enable-shared \
  CPPFLAGS="-I/usr/include" LDFLAGS="-Wl,-rpath,$PREFIX/lib"
- Build and install without overwriting system Python:
make -j$(nproc)
make altinstall
- Expose binaries on your PATH (shell session only):
export PATH=$PREFIX/bin:$PATH



Create a fully isolated virtual environment
- Bootstrap venv:
$PREFIX/bin/python3.14 -m venv /opt/venvs/py314
source /opt/venvs/py314/bin/activate
- Upgrade pip/setuptools/wheel:
pip install --upgrade pip setuptools wheel
- This mirrors AWS guidance to use a virtual environment for isolation on AL2, keeping system packages untouched.

Validate the build
- Confirm versions and shared library:
python --version
which python
ldconfig -p | grep libpython3.14
python -c "import ssl, sqlite3, bz2, lzma, _uuid; print('OK')"
- Check OpenSSL linkage (for TLS correctness):
python -c "import ssl; print(ssl.OPENSSL_VERSION)"



Optional: Add modern compression and concurrency extras
- Zstandard support:
pip install zstandard
- Async/observability tooling:
pip install anyio trio uvloop rich
- Python 3.14 introduces notable language and stdlib enhancements that are also supported in AWS environments like Lambda, making your 3.14 builds future-friendly for serverless and automation targets.

System-level isolation best practices
- Do not symlink to /usr/bin. Keep the prefix under /opt and use PATH or explicit shebangs.
- Pin interpreter in scripts:
#!/opt/venvs/py314/bin/python
- Use per-project venvs under /opt/venvs or your user home to avoid cross-project contamination.
- Record build manifest (libs installed, configure flags) for reproducibility in CI.

Troubleshooting
- _ssl or _sqlite3 missing: Ensure openssl-devel and sqlite-devel were installed before configure; then reconfigure and rebuild.
- libpython .so not found: Add rpath as shown, or export LD_LIBRARY_PATH:
export LD_LIBRARY_PATH=$PREFIX/lib:$LD_LIBRARY_PATH
- pip cannot build wheels: Install additional -devel headers (e.g., libxml2-devel, libxslt-devel) per your packages’ needs.

Next steps for your lab
- Tarball export: Package $PREFIX and /opt/venvs/py314 for portable deployment.
- CI caching: Cache the built prefix and venv wheels to speed up pipelines.
- Multi-interpreter orchestration: Run separate agents in distinct venvs for clean isolation; 3.14 aligns well with concurrent workflows also reflected in AWS’s runtime support.
Sources: AWS guidance on creating Python virtual environments on Amazon Linux 2; AWS announcement of Python 3.14 runtime support in Lambda


```
