# Lerning CaddyServer

Caddy is a powerful, enterprise-ready, open-source web server with automatic HTTPS written in Go. It's known for its ease of use, especially when it comes to setting up SSL certificates.

### 1\. Installation

**1.1. Download Caddy**

Visit the official Caddy website ([https://caddyserver.com/download](https://caddyserver.com/download)) to download the appropriate version for your operating system.

**1.2. Install Caddy**

  * **Linux/macOS:**
    1.  Extract the downloaded archive (e.g., `tar -xzf caddy_*.tar.gz`).
    2.  Move the Caddy executable to a directory in your system's PATH (e.g., `/usr/local/bin` or `/usr/bin`):
        ```bash
        sudo mv caddy /usr/local/bin
        ```
    3.  Verify the installation:
        ```bash
        caddy version
        ```
  * **Windows:**
    1.  Extract the downloaded ZIP file.
    2.  Move the `caddy.exe` file to a directory of your choice (e.g., `C:\Caddy`).
    3.  Add the directory to your system's PATH environment variable so you can run Caddy from any location in the command prompt.

**1.3. Install as a System Service (Recommended for Production)**

For production environments, it's highly recommended to run Caddy as a system service (e.g., using `systemd` on Linux) to ensure it starts automatically on boot and runs reliably in the background.

  * **Linux (systemd):**
    Caddy provides official systemd unit files.
    1.  Create a user and group for Caddy:
        ```bash
        sudo groupadd --system caddy
        sudo useradd --system \
            --gid caddy \
            --create-home \
            --home-dir /var/lib/caddy \
            --shell /usr/sbin/nologin \
            --comment "Caddy web server" \
            caddy
        ```
    2.  Download the systemd service file:
        ```bash
        sudo curl -o /etc/systemd/system/caddy.service https://raw.githubusercontent.com/caddyserver/dist/master/init/caddy.service
        ```
    3.  Create the Caddy configuration directory and file:
        ```bash
        sudo mkdir /etc/caddy
        sudo touch /etc/caddy/Caddyfile
        sudo chown -R caddy:caddy /etc/caddy
        ```
    4.  Create a directory for Caddy's data (where certificates are stored):
        ```bash
        sudo mkdir -p /var/lib/caddy
        sudo chown caddy:caddy /var/lib/caddy
        ```
    5.  Reload systemd and start Caddy:
        ```bash
        sudo systemctl daemon-reload
        sudo systemctl enable caddy
        sudo systemctl start caddy
        sudo systemctl status caddy
        ```

### 2\. Usage

Caddy's configuration is primarily done through a file called `Caddyfile`. This file is known for its simplicity and readability.

**2.1. Basic `Caddyfile` Structure**

A basic `Caddyfile` looks like this:

```caddy
<your-domain.com> {
    root * /path/to/your/website
    file_server
}
```

  * `<your-domain.com>`: This is the address Caddy will serve. If you omit this, Caddy will bind to `localhost:80` (or `localhost:443` if HTTPS is involved).
  * `root * /path/to/your/website`: Specifies the document root for your website. The `*` means it applies to all requests.
  * `file_server`: Enables Caddy's static file server.

**2.2. Examples of `Caddyfile` Configurations**

**2.2.1. Serving a Static Website**

Create a directory for your website (e.g., `/var/www/mywebsite`) and put some HTML files in it.

```caddy
# Caddyfile for a static website
mywebsite.com {
    root * /var/www/mywebsite
    file_server
}
```

Save this as `Caddyfile` in `/etc/caddy/` (if using systemd) or in the same directory where your `caddy` executable is.

To start Caddy (if not using systemd):

```bash
caddy run --config /etc/caddy/Caddyfile
```

If using systemd, just restart the service:

```bash
sudo systemctl reload caddy
```

**2.2.2. Automatic HTTPS**

Caddy automatically obtains and renews SSL/TLS certificates from Let's Encrypt for your domains. Just specify your domain name.

```caddy
example.com {
    root * /var/www/example.com
    file_server
    # Caddy will automatically get an SSL certificate for example.com
}
```

**Important:** For Caddy to obtain an SSL certificate, your domain must be pointing to the server's public IP address where Caddy is running, and ports 80 and 443 must be accessible from the internet.

**2.2.3. Reverse Proxy**

Caddy can easily act as a reverse proxy, forwarding requests to another server (e.g., a Node.js application, an API).

```caddy
api.example.com {
    reverse_proxy localhost:3000
    # This will proxy requests from api.example.com to a service running on localhost:3000
}
```

**2.2.4. Multiple Sites**

You can serve multiple sites from a single `Caddyfile`:

```caddy
example.com {
    root * /var/www/example.com
    file_server
}

blog.example.com {
    reverse_proxy localhost:8000
}

another-site.com {
    root * /var/www/another-site
    file_server
}
```

**2.2.5. Basic Authentication**

```caddy
secure.example.com {
    root * /var/www/secure-site
    file_server
    basicauth {
        username JdufWsdw3d3dwd
        password $2a$14$W5b1PqH3l.U/tqWkK.v2O.O4u.s.t.g.f.i.j.k.l.m.n.o.p.q.r.s.t.u.v.w.x.y.z.A.B.C.D.E.F.G.H.I.J.K.L.M.N.O.P.Q.R.S.T.U.V.W.X.Y.Z.0.1.2.3.4.5.6.7.8.9.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.q.r.s.t.u.v.w.x.y.z.0.1.2.3.4.5.6.7.8.9.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.q.r.s.t.u.v.w.x.y.z
    }
}
```

**Note:** The password in `basicauth` should be a bcrypt hash. You can generate one using online tools or a command-line utility.

**2.2.6. Redirects**

```caddy
old-domain.com {
    redir https://new-domain.com{uri} permanent
}
```

**2.3. Running Caddy**

  * **From the command line (for testing/development):**
    Navigate to the directory containing your `Caddyfile` and run:
    ```bash
    caddy run
    ```
    Or specify the config file:
    ```bash
    caddy run --config /path/to/Caddyfile
    ```
  * **As a system service (for production):**
    If you've installed Caddy as a `systemd` service, you'll manage it using `systemctl`:
    ```bash
    sudo systemctl start caddy      # Start Caddy
    sudo systemctl stop caddy       # Stop Caddy
    sudo systemctl restart caddy    # Restart Caddy
    sudo systemctl reload caddy     # Reload Caddyfile without downtime
    sudo systemctl status caddy     # Check Caddy's status
    ```
    Whenever you make changes to your `Caddyfile`, remember to run `sudo systemctl reload caddy` for the changes to take effect.

### 3\. Key Concepts and Best Practices

  * **`Caddyfile` Location:**
      * For systemd installations, the default `Caddyfile` location is `/etc/caddy/Caddyfile`.
      * When running from the command line, Caddy will look for a `Caddyfile` in the current directory by default.
  * **Ports:**
      * Caddy typically listens on port 80 for HTTP and port 443 for HTTPS. Ensure these ports are open in your firewall.
  * **Logging:**
    By default, Caddy logs to `stdout`/`stderr`. When running as a system service, these logs are usually captured by `journald`. You can view them with:
    ```bash
    journalctl -u caddy --no-pager
    ```
    You can also configure custom log files in your `Caddyfile`.
  * **Permissions:**
    Ensure that the `caddy` user (or the user Caddy runs as) has read access to your website files and write access to its data directory (e.g., `/var/lib/caddy`) for storing certificates.
  * **Security:**
      * Always keep Caddy updated to the latest stable version.
      * Follow best practices for securing your server (e.g., strong passwords, firewall rules, regular updates).
  * **Testing `Caddyfile`:**
    You can validate your `Caddyfile` syntax without starting the server:
    ```bash
    caddy validate --config /path/to/Caddyfile
    ```

This detailed guide should help you get started with CaddyServer. Its simplicity and automatic HTTPS make it an excellent choice for a wide range of web serving needs.
