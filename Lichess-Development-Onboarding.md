The following instructions outline how to set up your development environment for starting development on Lichess. The instructions are aimed to be agnostic of the platform the stack is installed on, so a working knowledge of the specifics of your GNU/Linux distribution or other such Unix based operating system is assumed. 

You can also [set up the environment on a Windows machine](Lichess-Development-Onboarding-(Windows)).

## Getting Help

If you get stuck during the installation process the most suitable place to seek help is the `#devs` channel on Discord (https://discord.gg/hy5jqSs). The main developer of Lichess (thibault) can be found there as well as several people who have successfully installed the stack. There is also a `#lichess` IRC channel on `irc.freenode.org`.

## Manual setup

### Prerequisites

Before beginning, please make sure you have the following tools installed, using your favourite package manager to install them where applicable.

#### Tools and dependency managers
* `git`
* `sbt` (>= 0.13.14 [instructions](https://www.scala-sbt.org/release/docs/Setup.html), [check if WSL](https://github.com/microsoft/WSL/issues/3286#issuecomment-402594992))
* `node` (>= 10, `nodejs` on Debian, `nodejs-legacy` pre Debian Buster, [instructions](https://github.com/nodesource/distributions/blob/master/README.md#installation-instructions), [check if Ubuntu](https://github.com/yarnpkg/yarn/issues/2821))
* `yarn` (>= 1.0, [instructions](https://yarnpkg.com/lang/en/docs/install/))
* `gulp-cli` (`sudo yarn global add gulp-cli`)
* `wget`

#### Infrastructure
* `mongodb` (>= 3.4.0, [instructions](https://docs.mongodb.com/manual/administration/install-on-linux/))
* `nginx`
* `redis`

#### Compilers
* `Java 8` (:warning: higher version will not work)

### Installation Steps

#### Prepare nginx

Add the following to your nginx configuration (for example replace the default config in `/etc/nginx/sites-enabled/default`). Don't forget to adjust paths and reload.

```
upstream backend {
  server 127.0.0.1:9663;
}

upstream socket_backend {
  server 127.0.0.1:9664;
}

server {
  server_name localhost assets.localhost;
  listen 80;
  listen [::]:80;

  error_log /var/log/nginx/lila.error.log;
  access_log /var/log/nginx/lila.access.log;

  charset utf-8;

  location /assets {
    rewrite "^/assets/_\w{6}/(.*)$" /assets/$1;
    add_header "Access-Control-Allow-Origin" "*";
    add_header "Service-Worker-Allowed" "/";
    alias /home/niklas/Projekte/lila/public;
    break;
  }

  error_page 500 501 502 503 /oops/servererror.html;
  error_page 504 /oops/timeout.html;
  error_page 429 /oops/toomanyrequests.html;
  location /oops/ {
    root /home/niklas/Projekte/lila/public/;
  }
  location = /robots.txt {
    root /home/niklas/Projekte/lila/public/;
  }
  location = /manifest.json {
    root /home/niklas/Projekte/lila/public/;
  }

  location / {
    proxy_http_version 1.1;
    proxy_set_header Host $http_host;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header X-Forwarded-For $remote_addr;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_read_timeout 90s;
    proxy_pass http://backend;
  }
}
```

#### Setup lila

1. Fork the lila project from github on your computer (including submodules): `git clone --recursive https://github.com/ornicar/lila.git`

1. Using your favourite terminal emulator, change your current directory to the top level of the checked out repository. This is important for the successful execution of the Lichess build scripts. `cd lila`

1. `cp bin/dev.default bin/dev`

1. `cp conf/application.conf.default conf/application.conf`

1. Run `./bin/gen/geoip` 

1. Compile the client side modules with `./ui/build`

1. Compile the Scala application with `./bin/dev compile`

#### Setup lila-ws

1. `git clone https://github.com/ornicar/lila-ws.git`

#### Running the Application

1. Make sure that mongodb is running. By default lila will try to connect to `mongodb://127.0.0.1:27017/lichess`.

1. Make sure that redis is running. By default lila will try to connect to `redis://127.0.0.1`

1. From the top level of `lila-ws`, execute `sbt run -Dhttp.port=9664`

1. From the top level of `lila`, execute `./bin/dev run`

1. Navigate to http://localhost/ with a browser. It can take a while to compile some remaining files.

#### Optional: Setup fishnet for server side analysis and play

[fishnet](https://github.com/niklasf/fishnet) is a Python script that manages Stockfish instances and lets them communicate with the server.

1. Install it: `pip install fishnet`

2. Run it and point it to your local installation: `python -m fishnet --endpoint http://localhost/fishnet` (will do some interactive configuration when started for the first time)

## Faster builds

To speed up `./ui/build`, install GNU parallel. The citation warning can be silenced with
```sh
mkdir -p ~/.parallel && touch ~/.parallel/will-cite
```
## Updating the code

Pull new code `git pull`, check if any submodule has updates with `git status` and if so run `git submodule update --recursive`.

Run `./ui/build` to update the client side modules.

For the server, `sbt` (invoked by `./bin/dev`) will automatically recompile any changed files. (In rare cases it does not manage to do this. You can use `sbt clean` as a last resort).

## Working on ...

Here are [some hints](Working-on-...) for working on various parts of the system.

## Using Eclipse IDE

* Download latest eclipse Mars and extract/install
* Install 'scala ide' (eclipse marketplace; This installs all scala ide plugins needed)
* Install sbteclipse (https://github.com/typesafehub/sbteclipse This helps to import scala project into eclipse with "sbt eclipse" command line)
* Install eclipse-sbt (https://github.com/jibijose/eclipse-sbt-plugin This hooks "sbt" commands into eclipse IDE)
* Create file ~/.sbt/0.13/plugins/plugins.sbt Add line "addSbtPlugin("com.typesafe.sbteclipse" % "sbteclipse-plugin" % "4.0.0")"
* Checkout scala project and run "sbt eclipse". This will create eclipse files (.project, .classpath, .settings)
* Import scala project as 'existing projects into eclipse'
* If you have install plugin mentioned in step 2-4, then you can run sbt commands from eclipse (Like update config, run etc)

## Automatic setup with Docker

There is an external project called [lichocker](https://github.com/BrandonE/lichocker), which allows you to run lila in a Docker container. Running in Docker simplifies setup and guarantees that your development environment will perfectly match that of anyone else who uses lichocker, eliminating the "it works on my machine" phenomena. However, it is always more performant to run any project directly on its host-machine, and lichocker is a work-in-progress that might not be reliable for your use.

## Automatic setup with Vagrant

This is [no longer supported](https://github.com/ornicar/lila/commit/75c87849c294d7530111bbb98dc6077a328bcea6). If you create and maintain a vagrant install, please make a GitHub repository for it, and we'll mention it here.

## Troubleshooting

### [PrimaryUnavailableException$: MongoError['No primary node is available!']]
Make sure mongod is running, check /var/log/mongo/mongod.log for errors
It might not start if you have too little free space (might need 3GB), or if there is a previous lock file that hasn't been cleaned up (maybe try removing /var/lib/mongodb/mongod.lock)

### Can't create games
    [ERROR] p.c.s.n.PlayDefaultUpstreamHandler Cannot invoke the action
    java.lang.ArrayIndexOutOfBoundsException: 101
check `mongo --version`, and that is satisfies the requirement at the top of this page.

### compiling timeouts
If you keep getting timeouts when compiling, use the `./bin/dev` wrapper script (instead of running `sbt` directly): `cp bin/dev.default bin/dev && chmod +x bin/dev`. Alternatively you can create this `SBT_OPTS` environment variable:

    export JAVA_OPTS="-Xms2048m -Xmx4g -XX:ReservedCodeCacheSize=64m -XX:+CMSClassUnloadingEnabled -XX:+UseConcMarkSweepGC -XX:+ExitOnOutOfMemoryError -Dkamon.auto-start=true"

If you are using an old sbt and if it's no problem that these options are used by all other Java applications running by your session, you may replace `SBT_OPTS` with `JAVA_OPTS`.

### Couldn't find package "ceval" on the "npm" registry.

Check your `yarn --version`. Requires at least yarn 1.0.

### value toList is not a member of java.util.stream.Stream[String]

Check your Java version. Only Java 8 is supported, no later.