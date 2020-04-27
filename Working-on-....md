# User interface
The mithril or snabbdom UI modules are in `ui/`. Say we work on `ui/round`, the playing UI.

Enable auto-recompile:

```
cd ui/round
gulp
```

To enable auto-recompile for the .scss files:

```
cd ui
gulp css
```

# Translations

1. New translation keys are added in `translation/source/site.xml` in British English.
1. Then regenerate translation keys for Scala: `node bin/trans-dump.js`
1. Do not touch `translations/dest/`. New translations from crowdin will automatically be applied here.

If you need to add a new file to `translation/source` (for example for translating a completely new page), you also need to add the name of the file (without the extension):

In `bin/trans-dump`:
[here](https://github.com/ornicar/lila/blob/3b38ffdbbedc2730746becbb4a4f076ddfb5274c/bin/trans-dump.js#L5)

In `/build.sbt`:
[here](https://github.com/ornicar/lila/blob/3b38ffdbbedc2730746becbb4a4f076ddfb5274c/build.sbt#L79)

# Puzzles

Add some puzzles to your local DB: https://github.com/ornicar/lichess-puzzle-kit