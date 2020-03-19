The following instructions outline how to set up your development environment for starting development on Lichess. The instructions are aimed to be agnostic of the platform the stack is installed on, so a working knowledge of the specifics of your GNU/Linux distribution or other such Unix based operating system is assumed. 

You can also [set up the environment on a Windows machine](Lichess-Development-Onboarding-(Windows)).

## Getting Help

If you get stuck during the installation process the most suitable place to seek help is the `#lila-development` channel on Discord (https://discord.gg/hy5jqSs). The main developer of Lichess (thibault) can be found there as well as several people who have successfully installed the stack. There is also a `#lichess` IRC channel on `irc.freenode.org`.

## Prerequisites

Before beginning, please make sure you have the following tools installed, using your favourite package manager to install them where applicable.

### Tools and dependency managers
* `git`
* `sbt` (>= 1.3 [instructions](https://www.scala-sbt.org/release/docs/Setup.html), [check if WSL](https://github.com/microsoft/WSL/issues/3286#issuecomment-402594992))
* `node` (>= 10, `nodejs` on Debian, `nodejs-legacy` pre Debian Buster, [instructions](https://github.com/nodesource/distributions/blob/master/README.md#installation-instructions), [check if Ubuntu](https://github.com/yarnpkg/yarn/issues/2821))
* `yarn` (>= 1.0, [instructions](https://yarnpkg.com/lang/en/docs/install/))
* `gulp-cli` (`sudo yarn global add gulp-cli`)
* `java` (JDK 13)

### Infrastructure
* `mongodb` (>= 3.6.0, [instructions](https://docs.mongodb.com/manual/administration/install-on-linux/))
* `redis` 

## Installation

### Setup lila

```sh
git clone --recursive https://github.com/ornicar/lila.git
cd lila
./ui/build # builds the CSS and JS
./lila # starts the SBT console
```
Once the console has booted, you will see a `lila>` prompt. Type `compile` and sit back.

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

## Code formatting

These repositories use [scalafmt](https://scalameta.org/scalafmt/).

Please [install it for your code editor](https://scalameta.org/scalafmt/docs/installation.html)
if you're going to contribute to this project. We recommend scalafmt-native which is simpler to setup than the nailgun solution.

If you don't install it, please run `scalafmtAll` in the sbt console before committing.

## Faster builds

To speed up `./ui/build`, install GNU parallel. The citation warning can be silenced with
```sh
mkdir -p ~/.parallel && touch ~/.parallel/will-cite
```

## Development

Here are [some hints](Working-on-...) for working on various parts of the system.

## Alternatives

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
  java.util.concurrent.TimeoutException: Futures timed out after [5 seconds]
  ```
  Check that MongoDB is running. Restart lila, if it was started before MongoDB.