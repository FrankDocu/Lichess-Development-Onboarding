The following instructions outline how to set up your development environment for starting development on Lichess. The instructions are aimed to be agnostic of the platform the stack is installed on so a working knowledge the specifics of your GNU/Linux distribution or other such Unix based Operating System is assumed. 

It may be possible to build in Windows but this has not been verified. Some of the scripts that must be ran as part of the process are written as GNU/Linux shell scripts so it would be neccessary to modify these scripts accordingly or verify these scripts work on cygwin or similar.

## Getting Help
If you get stuck during the install process the most suitable place to seek help is the `#lichess` IRC channel on `irc.freenode.org`. The main developer of Lichess (Ornicar) can be found there as well as several people who have successfully installed the stack.

## Prerequisites

Before beginning, please make sure you have the following tools installed, using your favourite package manager to install them where applicable.

### Tools and dependency managers
* `git`
* `sbt` ([instructions](http://www.scala-sbt.org/release/tutorial/Setup.html)) 
* `npm` (from nodejs, version `2.1.18` should be OK)
* `zsh`

### Infrastructure
* `mongodb`
* `nginx`

### Compilers
* `gcc` 
* `make` (Required to build stockfish)
* `closure` (JavaScript compiler)
* `Java 8`

## Installation Steps
### Setting up your Lichess configuration and Compiling the Web App.

1. Fork the lila project on github and clone it from your repository using git.

1. Using your favourite terminal emulator, change your current directory to the top level of the checked out repository. This is important for the successful execution of the Lichess build scripts.

1. Set up the submodules by executing the following command: `git submodule update --init --recursive`.

1. Run `./bin/build-deps.sh`

1. Compile the scala application with `sbt compile`

1. Run `sudo npm install gulp -g` (or `npm install gulp` and then add the install path of gulp to your PATH variable.)

1. Run `./ui/build`

1. Run `./bin/install-stockfish` (edit first if you are not running 64bit)

1. Run `./bin/gen/geoip` and add the following geoip section to application.conf:

        geoip {
            file = "data/GeoLite2-City.mmdb"
        }

### Setting Up Your Web Server

1. Add the following line to your hosts file :
`127.0.0.1 l.org en.l.org de.l.org le.l.org fr.l.org es.l.org l1.org socket.en.l.org socket.le.l.org socket.fr.l.org ru.l.org el.l.org hu.l.org socket.hu.l.org`
(Expand this for any other languages you might want to use)

1. Add the following 'Server' blocks to the bottom of your http block in your nginx configuration file: 

        server {
          listen 80;

          server_name l.org ~^\w\w\.l\.org$;

          error_log /var/log/nginx/lila.error.log;
          access_log /var/log/nginx/lila.access.log;

          charset utf-8;

          location /assets {
            alias   /home/happy0/projects/lila/public;
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
            root  /home/happy0/projects/lila/public/;
          }
          location = /robots.txt {
            root  /home/happy0/projects/lila/public/;
          }

        }

        server {
          listen 80;
          server_name ~^socket\.\w\w\.l\.org$;
          charset utf-8;
          location / {
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_pass http://127.0.0.1:9663/;
          }
        }

**Note**: Change the `/home/happy0/projects/lila` locations to the path of your checked out repository accordingly.

1. Restart (or start) nginx.

### Running the Application

1. Add the base application configuration file to the `conf` directory in the checked out project. **This must be retrieved from Ornicar himself.**

1. Add the application override configuration file to the `conf` directory in the checked out project. This can be found at : https://gist.github.com/clarkerubber/e0da7c22500fc6831a17 . **Note:** you should remove the exec_path unless you intend to use your own installed version of stockfish. If this is the case, you should change the path to point there instead.

1. Make sure that mongodb and nginx are both running.

1. From the top level of the lichess project, execute `sbt -Dhttp.port=9663`

1. When sbt is finished retrieving dependencies, type `run` and press enter.

1. Navigate to http://en.l.org with a browser.

### Troubleshooting

#### error: git clone git://github.com/../coach   Repository not found.
Check `npm --version` you need at least version 2.

#### compiling timeouts
If you keep getting timeouts when compiling, you have to create this `SBT_OPTS` environment variable:

    export SBT_OPTS="-Xms64M -Xmx2048M -Xss4M -XX:ReservedCodeCacheSize=64m -XX:+CMSClassUnloadingEnabled -XX:+UseConcMarkSweepGC"

Then, it might be necessary to edit the sbt launcher (`/usr/share/sbt-launcher-packaging/bin/sbt-launch-lib.bash`), if a fix is not already applied, to avoid that the launcher overrides these options. Quoting ddugovic at https://github.com/sbt/sbt-launcher-package/issues/81#issuecomment-112404471

> Workaround (fix?): I added to my `sbt-launch-lib.bash` the following (near the start of `get_mem_opts`):

    elif [[ "${SBT_OPTS}" == *-Xmx* ]] || [[ "${SBT_OPTS}" == *-Xms* ]] || [[ "${SBT_OPTS}" == *-XX:MaxPermSize* ]] || [[ "${SBT_OPTS}" == *-XX:MaxMetaspaceSize* ]] || [[ "${SBT_OPTS}" == *-XX:ReservedCodeCacheSize* ]]; then
         echo ""

If you don't want to edit the launcher file and if it's no problem that the options are used by all other Java applications running by your user, you don't have to edit the launcher file but you can replace `SBT_OPTS` by `JAVA_OPTS`.