The following instructions outline how to set up your development environment for starting development on Lichess. The instructions are aimed to be agnostic of the platform the stack is installed on, so a working knowledge the specifics of your GNU/Linux distribution or other such Unix based Operating System is assumed. 

If you want to set up the environment on a Windows machine, look here: https://github.com/ornicar/lila/wiki/Lichess-Development-Onboarding-(Windows)

## Getting Help
If you get stuck during the install process the most suitable place to seek help is the `#lichess` IRC channel on `irc.freenode.org`. The main developer of Lichess (Ornicar) can be found there as well as several people who have successfully installed the stack.

## Manual setup

### Prerequisites

Before beginning, please make sure you have the following tools installed, using your favourite package manager to install them where applicable.

#### Tools and dependency managers
* `git`
* `sbt` ([instructions](http://www.scala-sbt.org/release/tutorial/Setup.html)) 
* `npm` (from nodejs, version `2.1.18` should be OK)
* `zsh`

#### Infrastructure
* `mongodb`
* `nginx`

#### Compilers
* `Java 8`
* `closure` (if you wish to compile minified JavaScripts for production)

### Installation Steps

#### Setting up your Lichess configuration and Compiling the Web App

1. Fork the lila project from github on your computer (including submodules): `git clone --recursive https://github.com/ornicar/lila.git`

1. Using your favourite terminal emulator, change your current directory to the top level of the checked out repository. This is important for the successful execution of the Lichess build scripts. `cd lila`

1. Create `conf/application.conf` with the following content:

        include "base"                                                                  

        net {                                                                           
          domain = "l.org"                                                              
          asset.domain = "en.l.org"                                                     
        }
 
1. Run `./bin/build-deps.sh`

1. Compile the scala application with `sbt compile`

1. Run `sudo npm install gulp-cli -g` (or `npm install gulp-cli` and then add the install path of gulp to your PATH variable).

1. Run `./ui/build`

1. Run `npm install && ./bin/svg-optimize --no-svgcleaner` (or also install [svgcleaner](https://github.com/RazrFalcon/svgcleaner))

1. Run `./bin/gen/geoip` and add the following geoip section to application.conf (See section Running the Application how to retrieve the file):

        geoip {
          file = "data/GeoLite2-City.mmdb"
        }
   

#### Setting Up Your Web Server

1. Add the following line to your `/etc/hosts` file:
`127.0.0.1 l.org socket.l.org en.l.org de.l.org le.l.org fr.l.org es.l.org l1.org ru.l.org el.l.org hu.l.org`
(Expand this for any other languages you might want to use)

1. Add the following *server* blocks to the bottom of your http block in your nginx configuration file: 

        server {
          server_name l.org ~^\w\w\.l\.org$;
          listen 80;

          error_log /var/log/nginx/lila.error.log;
          access_log /var/log/nginx/lila.access.log;

          charset utf-8;

          location /assets {
            add_header "Access-Control-Allow-Origin" "*";
            alias /home/happy0/projects/lila/public;
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
            root /home/happy0/projects/lila/public/;
          }
          location = /robots.txt {
            root /home/happy0/projects/lila/public/;
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

   **Note**: Change the `/home/happy0/projects/lila` locations to the path of your checked out repository accordingly.

1. Restart (or start) nginx.

#### Optional: Setup fishnet for server side analysis and play

[fishnet](https://github.com/niklasf/fishnet) is a Python script that manages Stockfish instances and let's them communicate with the server.

1. Install it: `pip install fishnet`

2. Run it and point it to your local installation: `python -m fishnet --endpoint http://en.l.org/fishnet` (will do some interactive configuration when started for the first time)

#### Optional: Setup local SSL with a self signed certificate

1. Generate a self signed certificate: `openssl req -x509 -sha256 -newkey rsa:2048 -keyout /etc/ssl/private/l.org.key.orig -out /etc/ssl/private/l.org.pem -days 365` (Common Name should be `l.org *.l.org *.*.l.org`, all other fields can be empty. Choose a temporary passphrase.)

2. Remove the passphrase: `openssl rsa -in /etc/ssl/private/l.org.key.orig -out /etc/ssl/private/l.org.key` (Needs to be entered one last time)

3. Add the following directives to each of the server blocks in the nginx config.

        listen 443 ssl;

        ssl_certificate /etc/ssl/private/l.org.pem;
        ssl_certificate_key /etc/ssl/private/l.org.key;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_session_cache shared:SSL:10m;
        ssl_prefer_server_ciphers on;
        ssl_ciphers AES128+EECDH:AES128+EDH:!aNULL;

4. Reload nginx.

5. In your browser, accept the certificate on https://l.org, https://socket.l.org, https://en.l.org/, etc.

6. Optionally set `protocol = "https://"` in the `net { }` block of your `conf/application.conf` to use https as the default protocol.

#### Running the Application

1. Make sure that mongodb and nginx are both running.

1. From the top level of the lichess project, execute `sbt -Dhttp.port=9663`

1. When sbt is finished retrieving dependencies, type `run` and press enter.

1. Navigate to http://en.l.org with a browser.

## Import Code into IDE
* Download latest eclipse Mars and extract/install
* Install 'scala ide' (eclipse marketplace; This installs all scala ide plugins needed)
* Install sbteclipse (https://github.com/typesafehub/sbteclipse This helps to import scala project into eclipse with "sbt eclipse" command line)
* Install eclipse-sbt (https://github.com/jibijose/eclipse-sbt-plugin This hooks "sbt" commands into eclipse IDE)
* Create file ~/.sbt/0.13/plugins/plugins.sbt Add line "addSbtPlugin("com.typesafe.sbteclipse" % "sbteclipse-plugin" % "4.0.0")"
* Checkout scala project and run "sbt eclipse". This will create eclipse files (.project, .classpath, .settings)
* Import scala project as 'existing projects into eclipse'
* If you have install plugin mentioned in step 2-4, then you can run sbt commands from eclipse (Like update config, run etc)

## Automatic setup with Vagrant

Rather than manually carrying out the installation steps, you can use Vagrant to manage your development environment. Vagrant is a wrapper around virtual machine tools that helps you set up your development environment.

  * [Download Vagrant](https://www.vagrantup.com/) (or use your package manager). These instructions have been tested on Vagrant 1.7.4; you will probably [need at least Vagrant 1.5](http://stackoverflow.com/q/33644359/344643).
  * [Download Virtualbox](https://www.virtualbox.org/) (or use your package manager).

Clone the repository somewhere on your system and run `vagrant up`:

    $ git clone git@github.com:ornicar/lila
    $ cd lila
    $ vagrant up

This starts the process of downloading the virtual machine image, downloading all dependencies, and initially building the code. It may take a while (1 hour 20 minutes on the author's machine) depending on your network and CPU.

Once the initial build process has finished, it will print instructions to the screen. Follow those instructions to complete setup.

### Vagrant primer

Basic commands:

  * `vagrant up`: Launch the development environment specified by the `Vagrantfile` in the current directory. This creates a virtual machine and runs any provisioning scripts, or if the virtual machine already exists, just boots it up.
  * `vagrant suspend`: Put the virtual machine to sleep. If you don't want to allocate CPU/RAM/battery power to your virtual machine, it makes sense to suspend it. You can relaunch it with `vagrant up` later and resume working where you left off.
  * `vagrant destroy`: Destroys the virtual machine. Frees up disk space used for the virtual machine. To launch the virtual machine later, you would have to restart the long build process with `vagrant up`. You may want to do this if you screwed something up and want to start clean.
  * `vagrant ssh`: Starts an SSH connection to the virtual machine. You can then run commands inside the virtual machine (such as `sbt run`).

The files in the repository are shared with the `/vagrant` directory in the virtual machine. (Note that you start out in the `/home/vagrant` directory after running `vagrant ssh`; if you want to see the repository files, you should run `cd /vagrant` afterward.) If you make changes in your host machine using your favorite text editor, they will immediately show up in the virtual machine, and vice-versa.


## Troubleshooting

### when running vagrant up: ./ui/build: /bin/sh^M: bad interpreter: No such file or directory

Most probably, you are running windows and git has replaced all line breaks in all the files. Configure git correctly before checking out the project, something like this:

`git config --global core.autocrlf input`

Although you might want to set it for this project only if you use git for other stuff, I'll leave this as an exercise for the windows user, read at least everything about it here:

`git help config`

Note that there's probably none that succeded past this issue before, so there is likely a alot more issues up ahead when using windows. Feel free to update this section about your journey, good luck

#### error: git clone git://github.com/../coach   Repository not found.

Check `npm --version` you need at least version 2.
Link node with "sudo ln -s /usr/bin/nodejs /usr/bin/node"
sudo to root, then run "curl -L https://npmjs.org/install.sh | sh" and install latest npm from root shell
Verify it by running "npm --version". latest version is 3.7.2


### ./ui/build errors: git clone git@github.com:github:ornicar/chessground    is not a valid repository name

Same as above, npm at least version 2.

### unresolved dependency com.github.ornicar#scalalib_2.11;5.3
Make sure `./bin/build-deps.sh` runs successfully.

### [PrimaryUnavailableException$: MongoError['No primary node is available!']]
Make sure mongod is running, check /var/log/mongo/mongod.log for errors
It might not start if you have too little free space (might need 3GB), or if there is a previous lock file that hasn't been cleaned up (maybe try removing /var/lib/mongodb/mongod.lock)

### Can't create games
    [ERROR] p.c.s.n.PlayDefaultUpstreamHandler Cannot invoke the action
    java.lang.ArrayIndexOutOfBoundsException: 101
check `mongo --version`, it might be too old. 2.4.14 may not work, while 2.6.11 is reported to work.

### compiling timeouts
If you keep getting timeouts when compiling, you have to create this `SBT_OPTS` environment variable:

    export SBT_OPTS="-Xms64M -Xmx2048M -Xss4M -XX:ReservedCodeCacheSize=64m -XX:+CMSClassUnloadingEnabled -XX:+UseConcMarkSweepGC"

Then, it might be necessary to edit the sbt launcher (`/usr/share/sbt-launcher-packaging/bin/sbt-launch-lib.bash`), if a fix is not already applied, to avoid that the launcher overrides these options. Quoting ddugovic at https://github.com/sbt/sbt-launcher-package/issues/81#issuecomment-112404471

> Workaround (fix?): I added to my `sbt-launch-lib.bash` the following (near the start of `get_mem_opts`):

    elif [[ "${SBT_OPTS}" == *-Xmx* ]] || [[ "${SBT_OPTS}" == *-Xms* ]] || [[ "${SBT_OPTS}" == *-XX:MaxPermSize* ]] || [[ "${SBT_OPTS}" == *-XX:MaxMetaspaceSize* ]] || [[ "${SBT_OPTS}" == *-XX:ReservedCodeCacheSize* ]]; then
         echo ""

If you don't want to edit the launcher file and if it's no problem that the options are used by all other Java applications running by your user, you don't have to edit the launcher file but you can replace `SBT_OPTS` by `JAVA_OPTS`.

### Other unexpected issues with vagrant
File an issue and ping arxanas.

### `DatabaseException['<none>']`
You are probably running mongodb 3.4. Downgrade to 3.2 and it should work.

## Updating the code

If you get compile errors after pulling new code, check if any submodule has updates with `git status` and if so run `git submodule update --recursive`

Run `./ui/build` to update the mithril modules.