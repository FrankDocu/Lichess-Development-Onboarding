## Basics

Everything starts with the various `.xml` files in [`translation/source`](https://github.com/ornicar/lila/tree/master/translation/source). They define all the individual phrases that can be translated. Each file contains phrases for a certain concept or part of Lichess, e.g. `swiss.xml` for swiss tournaments, `faq.xml` for the FAQ, etc. `site.xml` is the main file that contains all site-wide phrases and everything that doesn't fit anywhere else.

Each phrase has a key/name and the original text in British English, e.g.

```xml
<string name="playWithAFriend">Play with a friend</string>
```

They can have placeholders marked by `%s`. These can be replaced by something, i.e. the number of games or a separate translated string that needs to be formatted differently or be a clickable link. `%1$s`, `%2$s`, etc. can be used for multiple placeholders.

```xml
<string name="xStartedStreaming">%s started streaming</string>
<string name="xStartedFollowingY">%1$s started following %2$s</string>
```

There are also `plurals` elements for phrases that need to change depending on the value of a placeholder:

```xml
<plurals name="nbBlunders">
  <item quantity="one">%s blunder</item>
  <item quantity="other">%s blunders</item>
</plurals>
```

The content of those `.xml` files are automatically uploaded to <http://crowdin.com/project/lichess> where volunteers translate them. The resulting translations are again automatically downloaded and regularly merged in PRs named "New Crowdin updates", resulting in another batch of `.xml` files in [`translation/dest/<area>/<lang-code>.xml`](https://github.com/ornicar/lila/tree/master/translation/dest) where `<area>` is the `faq`, `site`, etc. from above.

## Using translations in Scala

There's a Scala equivalent for every translation key defined in [`modules/i18n/src/main/I18nKeys.scala`](https://github.com/ornicar/lila/tree/master/modules/i18n/src/main/I18nKeys.scala). This file can be generated automatically from the source `.xml` files by running [`node bin/trans-dump.js`](https://github.com/ornicar/lila/tree/master/bin/trans-dump.js).

It defines an `I18nKeys` object with values for each translation key:

```scala
object I18nKeys {
    val `playWithAFriend` = new I18nKey("playWithAFriend")
    // ...

    object swiss {
        val `swissTournaments` = new I18nKey("swiss:swissTournaments")
        // ...
    }
}
```

That object is generally imported with `import lila.i18n.{ I18nKeys => trans }` so that keys can be accessed as `trans.theKey` for values from `site.xml`. Other areas need to be specified, e.g. `trans.swiss.theKey` for `swiss.xml`, etc.

Each key is an instance of `I18nKey` defined in [`modules/i18n/src/main/I18nKey.scala`](https://github.com/ornicar/lila/tree/master/modules/i18n/src/main/I18nKey.scala) which defines everything that can be done with them:

```scala
final class I18nKey(val key: String) {
  def apply(args: Any*)(implicit lang: Lang): RawFrag = // ...
  def plural(count: Count, args: Any*)(implicit lang: Lang): RawFrag = // ...
  def pluralSame(count: Int)(implicit lang: Lang): RawFrag = plural(count, count)

  def txt(args: Any*)(implicit lang: Lang): String = // ...
  def pluralTxt(count: Count, args: Any*)(implicit lang: Lang): String = // ...
  def pluralSameTxt(count: Int)(implicit lang: Lang): String = pluralTxt(count, count)
}
```

There are three different functions, each in two versions, one producing a fragment, i.e. for use in scalatags when generating HTML, and another producing a `String`.

* Applying a key directly as `trans.theKey()` directly gives a translated fragment. Values to replace placeholders can be passed directly as arguments: `trans.theKey("abc", 42)`.
* To use plural keys you need to use `trans.theKey.plural(42, "abc", 42)`. The first number determines the plural version to use  (i.e. "game" vs "games") and everything after that is replacing the placeholders. Note that the initial value has to be repeated in the correct position.
* For the common case when there is only one value that both determines the plural and needs to be inserted, there is `trans.theKey.pluralSame(42)`.

And then there are the three `String` version equivalents `trans.theKey.txt`, `trans.theKey.pluralTxt` and `trans.theKey.pluralSameTxt`.

### `implicit Lang`

All these functions have an implicit `Lang` parameter that specifies the target language to translate to when they are called. This parameter can be automatically extracted from an implicit `Context` in scope. Generally, that means adding additional `(implicit ctx: Context)` or `(implicit lang: Lang)` argument lists to all functions calling one of the translation functions, all the way up the call stack until the controller endpoints which will have an explicit `Context` available.

## Using translations in JavaScript

For JS, the Scala code generally passes a JSON object with `{ transKey: value }` down to the JS controller for each page. Often, a list of keys to send down is defined in `app/views/<area>/bits.scala` which is then converted to JSON using the `i18nJsObject` function brought into scope by an `import lila.app.templating.Environment._` (and originating in [`app\templating\I18hHelper.scala`](https://github.com/ornicar/lila/blob/master/app/templating/I18hHelper.scala)).

Here's an example from the swiss tournament pages: [`app/views/swiss.bits.scala`](https://github.com/ornicar/lila/blob/df603a006982f3cdbc0f655f12834922b6ec78dd/app/views/swiss/bits.scala#L84)

```scala
  def jsI18n(implicit ctx: Context) = i18nJsObject(i18nKeys)
  private val i18nKeys = List(
    trans.join,
    trans.withdraw,
    trans.youArePlaying,
    // ...
  ).map(_.key)
```

This object is passed to the call initializing the JS controller: [`app/views/swiss/show.scala`](https://github.com/ornicar/lila/blob/df603a006982f3cdbc0f655f12834922b6ec78dd/app/views/swiss/show.scala#L39)

```scala
embedJsUnsafeLoadThen(s"""LichessSwiss.start(${safeJsonValue(
  Json.obj(
    "data"   -> data,
    "i18n"   -> bits.jsI18n,
    // ...
)})""")
```

The JS controller then uses this JSON object to construct a `Trans` object: [`ui/swiss/src/ctrl.ts`](https://github.com/ornicar/lila/blob/df603a006982f3cdbc0f655f12834922b6ec78dd/ui/swiss/src/ctrl.ts#L24)

```js
this.trans = lichess.trans(opts.i18n);
```

The `lichess.trans` function is defined in [`ui/site/src/component/trans.ts`](https://github.com/ornicar/lila/blob/df603a006982f3cdbc0f655f12834922b6ec78dd/ui/site/src/component/trans.ts#L23).

The resulting `Trans` object provides similar functions as `I18nKey` in Scala:

* `trans("theKey", 42, "abc")` will give the translation for "theKey" as a `string`, replacing placeholders with `42` and `"abc"`.
* `trans.noarg("theKey")` is a fast-path for keys without placeholders
* `trans.plural("theKey", 42)` is for plural forms and the same as `pluralSameTxt` in Scala
* `trans.vdom("theKey", h("a", ...))` can be used when the placeholders are vdom nodes and returns a list of strings and nodes.
* `trans.vdomPlural("theKey", 42, h("a", 42))` is the same for plural forms