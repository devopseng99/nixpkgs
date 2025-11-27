```
🛠 Fixing SSL Support for Python 3.11 on AL2
1. Install a newer OpenSSL (1.1.1 or 3.x)
Amazon Linux 2 doesn’t provide OpenSSL 1.1.1 in its default repos. You’ll need to build it yourself or use a prebuilt package.
Option A: Build OpenSSL 1.1.1 from source
wget https://www.openssl.org/source/openssl-1.1.1w.tar.gz
tar xzf openssl-1.1.1w.tar.gz
cd openssl-1.1.1w
./config --prefix=$HOME/openssl --openssldir=$HOME/openssl
make -j$(nproc)
make install_sw


This installs OpenSSL into $HOME/openssl.

2. Point Python’s configure to the new OpenSSL
When you build Python, tell it where to find the newer OpenSSL:
cd ~/Python-3.11.11
./configure --enable-optimizations \
    --prefix=$HOME/python311 \
    --with-openssl=$HOME/openssl
make -j$(nproc)
make install



3. Verify SSL Support
After installation, check that the _ssl module is present:
$HOME/python311/bin/python3.11 -m ssl


If it runs without error, SSL is working. You can also test HTTPS:
$HOME/python311/bin/python3.11 -c "import urllib.request; print(urllib.request.urlopen('https://www.python.org').read()[:100])"



✅ Key Notes
- Without OpenSSL 1.1.1+, Python 3.11 will disable SSL, meaning pip and HTTPS won’t work.
- Building OpenSSL into a custom prefix keeps it isolated, so you can tar up both python311 and openssl together for portability.
- If you want maximum portability, bundle the OpenSSL build into your tarball alongside Python.

```
