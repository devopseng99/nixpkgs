```
In our earlier conversation about building Python 3.11, we focused on a very similar theme to what we’re doing now with Python 3.14: keeping the build isolated on Amazon Linux 2 so the system Python wasn’t disturbed.
Here are the main points we touched on for Python 3.11:

🛠 Build Setup
- Installing Development Tools (gcc, make, etc.) and required *-devel libraries (openssl-devel, bzip2-devel, libffi-devel, zlib-devel, readline-devel, sqlite-devel, xz-devel, tk-devel, gdbm-devel, uuid-devel).
- Using yum groupinstall "Development Tools" to bootstrap the environment.
📦 Source Download
- Fetching the tarball from python.org for the 3.11.x release.
- Extracting and preparing the source tree.
⚙️ Configure & Install
- Configuring with an isolated prefix (e.g., /opt/python/3.11) so it wouldn’t overwrite /usr/bin/python.
- Using flags like:
- --enable-optimizations for performance
- --with-lto for link-time optimization
- --enable-shared for shared library support
- Running make -j$(nproc) and make altinstall to avoid clobbering python3.
🔒 Isolation Practices
- Exporting PATH=/opt/python/3.11/bin:$PATH instead of symlinking into /usr/bin.
- Creating a virtual environment with python3.11 -m venv /opt/venvs/py311.
- Pinning scripts with explicit shebangs like #!/opt/venvs/py311/bin/python.
✅ Validation
- Checking python --version and confirming libpython3.11.so was linked correctly.
- Testing modules like ssl, sqlite3, bz2, lzma, _uuid.
🚀 Extras
- Installing modern compression (zstandard) and async libraries (anyio, trio, uvloop) to take advantage of Python 3.11’s performance improvements.
- Discussing reproducibility: caching wheels, recording configure flags, and packaging tarballs for CI/CD.

So in short: we scaffolded Python 3.11 in a prefix-isolated, reproducible fashion on Amazon Linux 2, with emphasis on venv isolation, reproducibility, and performance flags.
👉 Do you want me to diff the build process between Python 3.11 and 3.14 (what’s new, what’s the same, and what flags or libraries you’d tweak)? That would give you a clear upgrade path for your dev lab scaffolding.

```
