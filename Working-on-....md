# User interface
The mithril or snabbdom UI modules are in `ui/`. Say we work on `ui/round`, the playing UI.

Enable auto-recompile:

```
cd ui/round
yarn run dev --watch
```

To enable auto-recompile for the .scss files:

```
cd ui
yarn run gulp css
```

# Translations

1. New translation keys are added in `translation/source/site.xml` in British English.
1. Then regenerate translation keys for Scala: `node bin/trans-dump.js`
1. Do not touch `translations/dest/`. New translations from crowdin will automatically be applied here.

If you need to add a new file to `translation/source` (for example to translate a completely new page), you also need to add the name of the file (without the extension):

In `bin/trans-dump`:
[here](https://github.com/ornicar/lila/blob/3b38ffdbbedc2730746becbb4a4f076ddfb5274c/bin/trans-dump.js#L5)

In `/build.sbt`:
[here](https://github.com/ornicar/lila/blob/3b38ffdbbedc2730746becbb4a4f076ddfb5274c/build.sbt#L79)

# Puzzles

Import some puzzle data: https://github.com/ornicar/lila-db-seed

# Forum

Create the default categories: https://gist.github.com/kraktus/7a60362b6af362bb1bea0c1b6a212d15

# Insights

1. Download insights sample data: https://github.com/ornicar/lila/files/6807098/insight.bson.zip
2. run
```
mongorestore --db lichess-insight --collection insight insight.bson
```
3. Register an account named `thibault`.
4. Either play one rated game with it or hack [this function](https://github.com/ornicar/lila/blob/6048f3c4e223357ea99ed84ea4f0a82f251eb2fe/modules/insight/src/main/InsightApi.scala#L44).
5. Access to `http://localhost:9663/insights/thibault/`

# Accessibility

To activate Accessibility mode on lichess, press `tab` then `enter` on the homepage.

Most of the code is located in `ui/nvui`, and it is then loaded as a plugin.