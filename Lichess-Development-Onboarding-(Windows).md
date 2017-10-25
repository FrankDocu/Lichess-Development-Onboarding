The following instructions explain how to set up your development environment for Lichess on Windows.

GNU/Linux instructions: https://github.com/ornicar/lila/wiki/Lichess-Development-Onboarding

Note that Windows is not officially supported for lila builds. It works now, but there is no guarantee that it stays like that.

## Prequisites
 - Git
 - [sbt](http://www.scala-sbt.org/0.13/docs/Installing-sbt-on-Windows.html) If 13.6 then open sbt.bat and update to below
set INIT_SBT_VERSION=0.13.16
 - [MongoDB](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-windows/)
 - `npm` from [Node.js](https://nodejs.org/en/).
 - [nginx](http://nginx.org/en/docs/windows.html)
 - [yarn](https://yarnpkg.com/lang/en/docs/install/)
 - `gulp-cli` (`yarn global add gulp-cli`)
 - Powershell (likely already on your machine)

## Installation steps
0. Turn off line feed conversion in windows (all checkins are made from linux).
git config --global core.autocrlf false
1. Fork the lila project from github on your computer (including submodules): `git clone --recursive https://github.com/ornicar/lila.git`
git config --global core.autocrlf true

1. Change your current directory to the top level of the checked out repository. This is important for the successful execution of the Lichess build scripts. `cd lila`.
1. `copy bin\dev.default.windows.bat bin\dev.bat`
Edit dev.bat to have -Dfile.encoding=UTF-8, increase memory
-Xms2048M -Xmx3072M  -Dfile.encoding=UTF-8
1. Create `conf/application.conf` with the following content:

        include "base"

        net {
          domain = "l.org"
        }

        geoip {
           file = "data/GeoLite2-City.mmdb"
        }

        http {
          port = 9663
        }

1. Run `powershell -executionpolicy bypass -File bin\gen\geoip.ps1` to download the GeoIP database.
1. Run `yarn install`
1. Run `.\ui\build.bat`
1. Compile the Scala application with `.\bin\dev.bat compile`


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
            proxy_pass http://127.0.0.1:9663;
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
            proxy_pass http://127.0.0.1:9663;
          }
        }
Don't forget to change `<path\to\>` into an actual path.

3. Restart (or start) nginx.

## Running the application.

1. Make sure nginx is running.
3. Make sure a MongoDB server instance is running.
4. From the top level of the lichess project, execute `.\bin\dev.bat`
5. When sbt is finished retrieving dependencies, type `run 9663` and press enter.
6. In your browser, navigate to `l.org`
