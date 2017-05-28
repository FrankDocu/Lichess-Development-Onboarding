The following instructions explain how to set up your development environment for Lichess on Windows.

GNU/Linux instructions: https://github.com/ornicar/lila/wiki/Lichess-Development-Onboarding

Note that Windows is not officially supported for lila builds. You can get it to work now, but there is no guarantee that it stays like that.

## Prequisites
 - Git
 - [sbt](http://www.scala-sbt.org/0.13/docs/Installing-sbt-on-Windows.html)
 - [MongoDB](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-windows/)
 - `npm` from [Node.js](https://nodejs.org/en/).
 - [nginx](http://nginx.org/en/docs/windows.html)

## Installation steps

1. Clone the lila project to your computer: `git clone https://github.com/ornicar/lila.git`
2. Using the command prompt, navigate to the directory where you cloned it into: `cd lila`
3. Set up the submodules: `git submodule update --init --recursive`
4. Before we compile, we have to change some options related to the memory management of Java, otherwise you'll get an OutOfMemoryException.

        set JAVA_OPTS=-Xms64M -Xmx2048M -Xss4M -XX:ReservedCodeCacheSize=64m -XX:+CMSClassUnloadingEnabled -XX:+UseConcMarkSweepGC
5. Compile the application using `sbt compile`.
6. Run `npm install -g gulp-cli` to be able to build the lichess UI.
7. Run this `build-ui.bat` script to build the UI (based on `ui/build`): https://gist.github.com/ProgramFOX/5608e07f81daa6a4aa5b9501c684bdf4 (make sure to run it from the lila root directory)
8. Download http://geolite.maxmind.com/download/geoip/database/GeoLite2-City.mmdb.gz and unpack it. Then create a `data` folder in your lila application root and put GeoLite2-City.mmdb in there.


## Setting up your web server

1. Add the following line to your hosts file (`C:\Windows\System32\drivers\etc\hosts`): `127.0.0.1 l.org socket.l.org en.l.org`
2. Open your `nginx.conf` file, and add this 'Server' block to the `http` block:

        server {
          server_name l.org ~^\w\w\.l\.org$;
          listen 80;
        
          error_log logs\lila.access.log;
          access_log logs\lila.error.log;
        
          charset utf-8;
        
          location /assets {
            add_header "Access-Control-Allow-Origin" "*";
            alias <path\to\>\lila\public;
          }
        
          location / {
            proxy_set_header Host $http_host;
            proxy_set_header X-Forwarded-For $remote_addr;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_read_timeout 90s;
            proxy_http_version 1.1;
            proxy_pass http://127.0.0.1:9663/;
          }
        
          error_page 500 501 502 503 /oops/servererror.html;
          error_page 504 /oops/timeout.html;
          error_page 429 /oops/toomanyrequests.html;
          location /oops/ {
            root <path\to\>\lila\public;
          }
          location = /robots.txt {
             root <path\to\>\lila\public;
          }
        }
        
        server {
          server_name socket.l.org;
          listen 80;
        
          charset utf-8;
        
          location / {
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_set_header X-Forwarded-For $remote_addr;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_pass http://127.0.0.1:9663/;
          }
        }
Don't forget to change `<path\to\>` into an actual path.

## Running the application.

1. Navigate into the `conf` directory of your lila application root. Create a file `application.conf` with these contents:

        include "base"
         
        net {
          domain = "l.org"
          asset.domain = "en.l.org"
          extra_ports = []
        }
         
        geoip {
          file = "data/GeoLite2-City.mmdb"
        }
        
        http {
          port = 9663
        }
2. Make sure nginx is running.
3. Make sure a MongoDB server instance is running.
4. Run `sbt`.
5. When you can input a command, type `run 9663` and press enter.
6. In your browser, navigate to `l.org`
