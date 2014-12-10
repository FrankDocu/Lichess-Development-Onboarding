The following instructions outline how to set up your development environment for starting development on Lichess. The instructions are aimed to be agnostic of the platform the stack is installed on so a working knowledge the specifics of your GNU/Linux distribution or other such Unix based Operating System is assumed. 

It may be possible to build in Windows but this has not been verified. Some of the scripts that must be ran as part of the process are written as GNU/Linux shell scripts so it would be neccessary to modify these scripts accordingly or verify these scripts work on cygwin or similar.

## Getting Help
If you get stuck during the install process the most suitable place to seek help is the `#lichess` IRC channel on `irc.freenode.org`. The main developer of Lichess (Ornicar) can be found there as well as several people who have successfully installed the stack.

## Prerequisites

Before beginning, please make sure you have the following tools installed, using your favourite package manager to install them where applicable.

### Tools and dependency managers
* `git`
* `sbt`
* `npm`

### Infrastructure
* `mongodb`
* `nginx`

### Compilers
* `gcc` 
* `make` (Required to build stockfish)
* `closure` (JavaScript compiler)

## Installation Steps
### Setting up your Lichess configuration and Compiling the Web App.
1. Fork the lila project on github and clone it from your repository using git.

1. Set up the submodules by executing the following command: `git submodule update --init --recursive`.

1. Add the base application configuration file to the `conf` directory in the checked out project. **This must be retrieved from Ornicar himself.**

1. Add the application override configuration file to the `conf` directory in the checked out project. This can be found at : https://gist.github.com/clarkerubber/e0da7c22500fc6831a17 . **Note:** you should remove the exec_path unless you intend to use your own installed version of stockfish. If this is the case, you should change the path to point there instead.

1. Using your favourite terminal emulator, change your current directory to the top level of the checked out repository. This is important for the successful execution of the Lichess build scripts.

1. Run `npm install gulp`

1. Run `./bin/build-deps.sh`

1. Run `./bin/compile-client`

1. Run `./bin/install-stockfish`

1. Run `./bin/svg-optimise`

### Setting Up Your Web Server

1. Add the following line to your hosts file :
`127.0.0.1 l.org en.l.org de.l.org le.l.org fr.l.org es.l.org l1.org socket.en.l.org socket.le.l.org socket.fr.l.org ru.l.org el.l.org hu.l.org socket.hu.l.org`

1. Add the following 'Server' block to the bottom of your http block in your nginx configuration file: 
`server {
          listen 80;

          server_name l.org ~^\w\w\.l\.org$;

          error_log /var/log/nginx/lila.error.log;
          access_log /var/log/nginx/lila.access.log;

          charset utf-8;

          location / {
            proxy_set_header Host $http_host;
            proxy_set_header X-Forwarded-For $remote_addr
            proxy_read_timeout 90s;
            proxy_pass http://127.0.0.1:9663/;
          }

          location /ai/ {
            proxy_set_header Host $http_host;
            proxy_set_header X-Forwarded-For $remote_addr
            proxy_pass http://127.0.0.1:9663/ai/;
          }

          error_page 500 501 502 503 /oops/servererror.ht
          error_page 504  /oops/timeout.html;
          error_page 429  /oops/toomanyrequests.html;
          location /oops/ {
            root  /home/happy0/projects/lila/public/;
          }
          location = /robots.txt {
            root  /home/happy0/projects/lila/public/;
          }

        }

**Note**: Change the /robots.txt and /oops/ locations to the path of your checked out repository accordingly.

1. Restart (or start) nginx.

### Running the Application

1. Make sure that mongodb and nginx are both running.

1. From the top level of the lichess project, execute `sbt -Dhttp.port=9663`

1. When sbt is finished retrieving dependencies, type `run` and press enter.
 
