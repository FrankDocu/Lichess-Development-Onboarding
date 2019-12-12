The following instructions outline how to set up your development environment for starting development on Lichess. The instructions are aimed to be agnostic of the platform the stack is installed on, so a working knowledge of the specifics of your GNU/Linux distribution or other such Unix based operating system is assumed. 

You can also [set up the environment on a Windows machine](Lichess-Development-Onboarding-(Windows)).

## Getting Help

If you get stuck during the installation process the most suitable place to seek help is the `#devs` channel on Discord (https://discord.gg/hy5jqSs). The main developer of Lichess (thibault) can be found there as well as several people who have successfully installed the stack. There is also a `#lichess` IRC channel on `irc.freenode.org`.

## Manual setup

### Prerequisites

Before beginning, please make sure you have the following tools installed, using your favourite package manager to install them where applicable.

#### Tools and dependency managers
* `git`
* `sbt` (>= 1.3 [instructions](https://www.scala-sbt.org/release/docs/Setup.html), [check if WSL](https://github.com/microsoft/WSL/issues/3286#issuecomment-402594992))
* `node` (>= 10, `nodejs` on Debian, `nodejs-legacy` pre Debian Buster, [instructions](https://github.com/nodesource/distributions/blob/master/README.md#installation-instructions), [check if Ubuntu](https://github.com/yarnpkg/yarn/issues/2821))
* `yarn` (>= 1.0, [instructions](https://yarnpkg.com/lang/en/docs/install/))
* `gulp-cli` (`sudo yarn global add gulp-cli`)

#### Infrastructure
* `mongodb` (>= 3.6.0, [instructions](https://docs.mongodb.com/manual/administration/install-on-linux/))
* `redis`

#### Compilers
* `Java` Any version from 8 to 13.

### Installation Steps

#### Setup lila

1. Fork the lila project from github on your computer: `git clone https://github.com/ornicar/lila.git`

1. Using your favourite terminal emulator, change your current directory to the top level of the checked out repository. This is important for the successful execution of the Lichess build scripts. `cd lila`

1. Compile the client side modules with `./ui/build`

1. Start the SBT console with `./lila`

Type `~run` to start the HTTP server. It will recompile and restart when needed.

> You could also type `~compile` to just compile and recompile when needed.

1. Open http://127.0.0.1:9000 in your browser. 

#### Setup websockets

```sh
git clone https://github.com/ornicar/lila-ws.git
cd lila-ws
sbt run
```

## Faster builds

To speed up `./ui/build`, install GNU parallel. The citation warning can be silenced with
```sh
mkdir -p ~/.parallel && touch ~/.parallel/will-cite
```

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

### java.util.concurrent.TimeoutException: Futures timed out after [5 seconds]

Check that MongoDB is running. Restart lila, if it was started before MongoDB.