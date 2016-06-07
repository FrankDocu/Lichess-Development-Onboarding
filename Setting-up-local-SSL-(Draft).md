Choose any passphrase for the key and remember it briefly. Common name should be `l.org *.l.org *.*.l.org`.

```
# cd /etc/ssl/private
# openssl req -x509 -sha256 -newkey rsa:2048 -keyout l.org.key.orig -out l.org.pem -days 365
Generating a 2048 bit RSA private key
.........+++
.......+++
writing new private key to 'l.org.key'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:DE
State or Province Name (full name) [Some-State]:
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:l.org *.l.org *.*.l.org
Email Address []:
```

Remove the passphrase. Needs to be entered one last time to decrypt the key.
```
# openssl rsa -in l.org.key.orig -out l.org.key
Enter pass phrase for l.org.key.orig:
writing RSA key
```

New nginx config, to be used in parallel with the existing one:

```
server {
  listen 443 ssl;

  server_name l.org ~^\w\w\.l\.org$;

  ssl_certificate /etc/ssl/private/l.org.pem;
  ssl_certificate_key /etc/ssl/private/l.org.key;
  ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
  ssl_session_cache shared:SSL:10m;
  ssl_prefer_server_ciphers on;
  ssl_ciphers AES128+EECDH:AES128+EDH:!aNULL;

  error_log /var/log/nginx/lila.error.log;
  access_log /var/log/nginx/lila.access.log;

  charset utf-8;

  location /assets {
    alias /home/niklas/Projekte/lila/public;
    location ~* \.(eot|otf|ttf|woff|json)$ {
      add_header Access-Control-Allow-Origin "*";
    }
  }

  location / {
    proxy_set_header Host $http_host;
    proxy_set_header X-Forwarded-For $remote_addr;
    proxy_read_timeout 90s;
    proxy_pass http://127.0.0.1:9663/;
  }

  location /ai/ {
    proxy_set_header Host $http_host;
    proxy_set_header X-Forwarded-For $remote_addr;
    proxy_pass http://127.0.0.1:9663/ai/;
  }

  error_page 500 501 502 503 /oops/servererror.html;
  error_page 504  /oops/timeout.html;
  error_page 429  /oops/toomanyrequests.html;
  location /oops/ {
    root  /home/niklas/Projekte/lila/public/;
  }
  location = /robots.txt {
    root  /home/niklas/Projekte/lila/public/;
  }

}

server {
  listen 443 ssl;
  server_name socket.l.org;

  ssl_certificate /etc/ssl/private/l.org.pem;
  ssl_certificate_key /etc/ssl/private/l.org.key;
  ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
  ssl_session_cache shared:SSL:10m;
  ssl_prefer_server_ciphers on;
  ssl_ciphers AES128+EECDH:AES128+EDH:!aNULL;

  charset utf-8;
  location / {
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_pass http://127.0.0.1:9663/;
  }
}
```

The cert is self signed and thus needs to be accepted manually. Cycle through the relevant domains to do this: https://socket.l.org/, https://l.org/, https://en.l.org/. 