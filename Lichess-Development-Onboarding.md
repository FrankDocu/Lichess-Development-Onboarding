The following instructions outline how to set up your development environment for starting development on Lichess. The instructions are aimed to be agnostic of the platform the stack is installed on, so a working knowledge of the specifics of your GNU/Linux distribution or other such Unix based operating system is assumed. 

You can also [set up the environment on a Windows machine](Lichess-Development-Onboarding-(Windows)) or on [macOS](Lichess-Development-Onboarding-(macOS)).

## Getting Help

If you get stuck during the installation process the most suitable place to seek help is the `#how-to-setup-lila` channel on Discord (https://discord.gg/hy5jqSs). The main developer of Lichess (thibault) can be found there as well as several people who have successfully installed the stack.

## Stream recordings

[![https://www.youtube.com/watch?v=AejdHlL902w](https://img.youtube.com/vi/AejdHlL902w/0.jpg)](https://www.youtube.com/watch?v=AejdHlL902w&list=PLe5ZNOR8Ttm1LlBRmmnccZkQa7XH1K0rK)

## Prerequisites

Before beginning, please make sure you have the following tools installed, using your favourite package manager to install them where applicable.

### Hardware 
* At least 4 GB of RAM
* A CPU with 64-bit architecture.

### Tools and dependency managers
* `git`
* `java` (JDK >= 11)
* `sbt` (>= 1.3, [instructions](https://www.scala-sbt.org/release/docs/Setup.html), [check if WSL](https://github.com/microsoft/WSL/issues/3286#issuecomment-402594992))
* `node` (16 > node >= 10, install via [official install](https://github.com/nodesource/distributions/blob/master/README.md#installation-instructions) or [NVM install](https://github.com/nvm-sh/nvm#installing-and-updating))
* `yarn` (>= 1.0, `apt install yarnpkg` since Debian Buster, [other instructions](https://yarnpkg.com/lang/en/docs/install/))
* `python2` (build dependency for some node packages)

### Running infrastructure
* `mongodb` (>= 4.2, [instructions](https://docs.mongodb.com/manual/administration/install-on-linux/), [WSL2](https://stackoverflow.com/questions/62495999/installing-mongodb-in-wsl))
  * For WSL2, you might want to manually create the default `/data/db` directory. If `sudo service mongod start` does not work, you may want to open a terminal and run `mongod` as super-user.
* `redis` 

Alternatively, if you have setup [docker-compose](https://docs.docker.com/compose/install/) on your machine, write a `docker-compose.yml` file:

    version: "3.3"
    services:
      redis:
        image: redis
        ports:
          - 6379:6379
      mongo:
        image: mongo
        restart: always
        ports:
          - 27017:27017

and spin up a `redis` and `mongodb` instance with:

    docker-compose up

## Installation

### Create database indexes

```
mongo lichess bin/mongodb/indexes.js
```

### Setup lila

```sh
git clone --recursive https://github.com/ornicar/lila.git
cd lila
./ui/build # builds the CSS and JS
./lila # starts the SBT console
```
Once the console has booted, you will see a `lila>` prompt. Type `compile` and sit back. The full compilation takes 5 minutes on GitHub CI servers.

When it's done,  type `run` to start the HTTP server.
Then open http://127.0.0.1:9663 in your browser.

> [Read more about the SBT console commands](https://www.playframework.com/documentation/2.8.x/PlayConsole).

### Setup websockets

If you need websockets (which you probably do):
```sh
git clone https://github.com/ornicar/lila-ws.git
cd lila-ws
sbt run
```

### Optional: Seed database

Parts of the site (like puzzles) require some minimal content to function.
You can use https://github.com/ornicar/lila-db-seed to seed your local database with dummy data.

### Optional: Setup Stockfish analysis

Start a fishnet client for analysis (requires a recent [Rust toolchain](https://rustup.rs/) to build from source, [alternatives](https://github.com/niklasf/fishnet#readme)):

```sh
git clone --recursive https://github.com/niklasf/fishnet.git
cd fishnet
cargo run -- --endpoint http://localhost:9663/fishnet/
```

### Optional: Setup "Play with the computer"

lila-fishnet enables playing vs Stockfish (not needed for analysis):

```sh
git clone https://github.com/ornicar/lila-fishnet.git
cd lila-fishnet
sbt run -Dhttp.port=9665
```

You will also need a client. Start a fishnet client for play against the machine (requires a recent [Rust toolchain](https://rustup.rs/) to build from source, [alternatives](https://github.com/niklasf/fishnet#readme)):

```sh
git clone --recursive https://github.com/niklasf/fishnet.git
cd fishnet
cargo run -- --endpoint http://localhost:9665/fishnet/
```

## Code formatting

These repositories use [scalafmt](https://scalameta.org/scalafmt/) for Scala and [prettier](https://prettier.io/) for everything else.

Please [install scalafmt for your editor](https://scalameta.org/scalafmt/docs/installation.html), or run `scalafmtAll` in the sbt console before submitting code.

Likewise, pick a plugin for prettier ([coc-prettier](https://github.com/neoclide/coc-prettier) is good for nvim), or use `yarn run format`.

## Faster assets builds

To speed up `./ui/build`, install GNU parallel. The citation warning can be silenced with
```sh
mkdir -p ~/.parallel && touch ~/.parallel/will-cite
```

## Recommended setup for hacking lila scala code

[Set up bloop for quick builds and IDE features](Lichess-Development-With-Bloop).

## Development

Here are [some hints](Working-on-...) for working on various parts of the system.

If you want to make a user admin, connect to the lichess db with `mongo lichess` and run
```
db.user4.update({ _id: "your_id" }, {$set: {roles: ["ROLE_SUPER_ADMIN"]}})
```
With `your_id` being the username in lowercase

## Alternatives

### IntelliJ IDE

Here is a guide on how to [set up lila with the IntelliJ IDE](https://github.com/ornicar/lila/wiki/Lichess-Development-Onboarding-(IntelliJ-on-Linux)).

### Docker

There is an external project called [lichocker](https://github.com/BrandonE/lichocker), which allows you to run lila in a Docker container. Running in Docker simplifies setup and guarantees that your development environment will perfectly match that of anyone else who uses lichocker, eliminating the "it works on my machine" phenomena. However, it is always more performant to run any project directly on its host-machine, and lichocker is a work-in-progress that might not be reliable for your use.

### Eclipse IDE (outdated)

* Download latest eclipse Mars and extract/install
* Install 'scala ide' (eclipse marketplace; This installs all scala ide plugins needed)
* Install sbteclipse (https://github.com/typesafehub/sbteclipse This helps to import scala project into eclipse with "sbt eclipse" command line)
* Install eclipse-sbt (https://github.com/jibijose/eclipse-sbt-plugin This hooks "sbt" commands into eclipse IDE)
* Create file ~/.sbt/0.13/plugins/plugins.sbt Add line "addSbtPlugin("com.typesafe.sbteclipse" % "sbteclipse-plugin" % "4.0.0")"
* Checkout scala project and run "sbt eclipse". This will create eclipse files (.project, .classpath, .settings)
* Import scala project as 'existing projects into eclipse'
* If you have install plugin mentioned in step 2-4, then you can run sbt commands from eclipse (Like update config, run etc)

### Vagrant (outdated)

This is [no longer supported](https://github.com/ornicar/lila/commit/75c87849c294d7530111bbb98dc6077a328bcea6). If you create and maintain a vagrant install, please make a GitHub repository for it, and we'll mention it here.

## Troubleshooting

* ```
  [PrimaryUnavailableException$: MongoError['No primary node is available!']]
  ```
  Make sure mongod is running, check `/var/log/mongo/mongod.log` for errors.
  It might not start if you have too little free space (might need 3GB), or if there is a previous lock file that hasn't been cleaned up (maybe try removing `/var/lib/mongodb/mongod.lock`)

* Can't create games
  ```
  [ERROR] p.c.s.n.PlayDefaultUpstreamHandler Cannot invoke the action
  java.lang.ArrayIndexOutOfBoundsException: 101
  ```
  Check `mongo --version`, and that is satisfies the requirement at the top of this page.

* Couldn't find package "ceval" on the "npm" registry.

  Check your `yarn --version`. Requires at least yarn 1.0.

* ```
  java.util.concurrent.TimeoutException: Future timed out after [5 seconds]
  ```
  Check that MongoDB is running. Restart lila, if it was started before MongoDB.
  On OS X, the connection timeout might be needed to be increased initially (5 seconds could be too short for a cold start). See [#6718](https://github.com/ornicar/lila/issues/6718).

* OS X: If you have problems connecting to mongoreactive from lila-ws, change `s"$reactivemongoVersion-linux-x86-64"` to `s"$reactivemongoVersion-osx-x86-64"` in the `reactivemongo-shaded-native` dependency

* Mongo error when Lila running
  ```
  [error] reactivemongo.api.Cursor - fails to send request
  reactivemongo.core.errors.DatabaseException$$anon$1: DatabaseException['error processing query: ns=lichess.challenge limit=50Tree: $and
  ```
  or similar excptions due to missing indexes: Redo the [Create database indexes](#create-database-indexes) step.

* Mongo error when importing games
  ```
  DatabaseException['cannot insert document because it exceeds 180 levels of nesting' (code = 15)]?
  ```
  In `/etc/mongodb.conf`:
  ```
  setParameter:
    maxBSONDepth: 999
  ```

* `sbt` prints `Killed` and exits

  Most likely there was not enough free RAM to compile lila.