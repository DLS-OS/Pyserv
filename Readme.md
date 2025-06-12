# PyServ - Simple HTTPS File Server with Optional Authentication

**PyServ** is a lightweight Python-based HTTPS file server that allows you to securely serve a directory over HTTPS, with optional Basic Authentication. It's designed to be simple to deploy for sharing files, hosting CTF challenges, or quick personal use.

---

## 🚀 Features

* Serves files over HTTPS
* Auto-generates SSL certificates (self-signed)
* Optional Basic Authentication (username\:password)
* Logs client access
* Easy command-line interface to start, stop, and check server status
* Runs in the background

---

## 📦 Installation

Ensure you have Python 3 and `openssl` installed.

```bash
chmod +x pyserv
```

---

## 🧪 Usage

### Start the Server

```bash
./pyserv start -d /path/to/serve
```

### Start with Password Authentication

```bash
./pyserv start -d /path/to/serve --passwd admin:1234
```

Or use the default username `admin`:

```bash
./pyserv start --passwd 1234
```

### Start Without Authentication

```bash
./pyserv start -d /path/to/serve --nopass
```

### Stop the Server

```bash
./pyserv stop
```

### Check Server Status

```bash
./pyserv status
```

---

## 🛡️ Authentication

When authentication is enabled, the server uses HTTP Basic Auth. Users must provide a valid username and password to access the files.

---

## 🔐 SSL Certificates

On first run, PyServ automatically generates a self-signed SSL certificate and key located in:

```
~/.pyserv/cert.pem
~/.pyserv/key.pem
```

Certificates are valid for 1 year and stored securely under `~/.pyserv`.

---

## 📁 Default Directory

If no directory is specified, PyServ serves files from:

```
/usr/share/pyserv
```

---

## 📂 Log and PID Files

* Server logs: `~/.pyserv/server.log`
* PID file: `~/.pyserv/server.pid`

---

## ⚠️ Notes

* Use self-signed certificates only for trusted environments or testing purposes.
* Make sure your chosen directory has appropriate read permissions.

