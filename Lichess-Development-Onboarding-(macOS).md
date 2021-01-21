Note that development on macOS is not officially supported by the core team, but several community members have done it successfully.

Follow the [Linux setup guide](Lichess-Development-Onboarding). Please note the following differences:

It is recommended to use [homebrew](https://brew.sh/) as the package manager. Most dependencies can be installed from there.

# lila-ws

The following change needs to be made for lila-ws to run correctly on macOS.

Edit [build.sbt](https://github.com/ornicar/lila-ws/blob/master/build.sbt) to make the following change (replace the red line with the green contents, without the leading +/- symbol):
```diff
-libraryDependencies += "org.reactivemongo"           % "reactivemongo-shaded-native"  % s"$reactivemongoVersion-linux-x86-64"
+libraryDependencies += "org.reactivemongo"           % "reactivemongo-shaded-native"  % s"$reactivemongoVersion-osx-x86-64"
```

Note that not all features of lila-ws have been thoroughly tested on mac, but it seems to work well enough for testing basic pvp gameplay.