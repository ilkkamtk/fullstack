# How to fix the mongodb certificate issue

These options might or might not work.

Remove .venv and recreate it
```
rm -rf .venv
python3 -m venv .venv
source .venv/bin/activate
```

If this does not work, try manually installing a certificate bundle inside your venv:

```
pip install certifi
```

Then force Python to use it:

```
export SSL_CERT_FILE=$(python3 -m certifi)
export SSL_CERT_DIR=$(python3 -m certifi)
```

To make this persistent, add to your venv’s activation script:

```
.venv/bin/activate
```

Add near the bottom:
```
export SSL_CERT_FILE=$(python3 -m certifi)
export SSL_CERT_DIR=$(python3 -m certifi)
```
