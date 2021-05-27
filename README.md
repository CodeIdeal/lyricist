[![Android API](https://img.shields.io/badge/api-21%2B-brightgreen.svg?style=for-the-badge)](https://android-arsenal.com/api?level=21)
[![kotlin](https://img.shields.io/github/languages/top/adrielcafe/lyricist.svg?style=for-the-badge)](https://kotlinlang.org/)
[![ktlint](https://img.shields.io/badge/code%20style-%E2%9D%A4-FF4081.svg?style=for-the-badge)](https://ktlint.github.io/)
[![License MIT](https://img.shields.io/github/license/adrielcafe/lyricist.svg?style=for-the-badge&color=yellow)](https://opensource.org/licenses/MIT)

# (WIP) Lyricist 🌎🌍🌏 
> The missing [I18N and I10N](https://en.wikipedia.org/wiki/Internationalization_and_localization) library for [Jetpack Compose](https://developer.android.com/jetpack/compose)!

Jetpack Compose revolutionized the way we build UIs on Android, but not how we **interact with strings**. `stringResource()` works well, but don't benefit from the idiomatic Kotlin like Compose does.

Lyricist tries to make working with strings as powerful as building UIs with Compose, *i.e.*, working with parameterized string is now typesafe, use of `when` expression to work with plurals with more flexibility, and even load/update the strings dynamically via an API!

#### Next steps
* Generate the `Strings` class through existing `strings.xml` files

#### Why _Lyricist_?
Inspired by [accompanist](https://github.com/google/accompanist#why-the-name) library: music composing is done by a composer, and since this library is about writing ~~lyrics~~ strings, the role of a [lyricist](https://en.wikipedia.org/wiki/Lyricist) felt like a good name.

## Usage
Take a look at the [sample app](https://github.com/adrielcafe/lyricist/tree/main/sample/src/main/java/cafe/adriel/lyricist/sample) and [sample-multi-module](https://github.com/adrielcafe/lyricist/tree/main/sample-multi-module/src/main/java/cafe/adriel/lyricist/sample/multimodule) for working examples.

First, create a `data class`, `class` or `interface` and declare your strings. The strings can be anything: `Char`, `String`, `AnnotatedString`, `List<String>`, `Set<String>` or even lambdas!
```kotlin
data class Strings(
    val simpleString: String,
    val annotatedString: AnnotatedString,
    val parameterString: (locale: String) -> String,
    val pluralString: (count: Int) -> String,
    val listStrings: List<String>
)
```

Next, create instances for each supported language and annotate with `@Strings`. The `languageTag` must be an [IETF BCP47](https://en.wikipedia.org/wiki/IETF_language_tag) compliant language tag ([docs](https://developer.android.com/guide/topics/resources/providing-resources#LocaleQualifier)).
```kotlin
@Strings(languageTag = Locales.EN, default = true)
val EnStrings = Strings(
    simpleString = "Hello Compose!",

    annotatedString = buildAnnotatedString {
        withStyle(SpanStyle(color = Color.Red, fontFamily = FontFamily.Cursive)) { append("Hello ") }
        withStyle(SpanStyle(fontWeight = FontWeight.Light, fontSize = 16.sp)) { append("Compose!") }
    },

    parameterString = { locale ->
        "Current locale: $locale"
    },

    pluralString = { count ->
        val value = when (count) {
            1 -> "$count wish"
            else -> "$count wishes"
        }
        "You have $value remaining"
    },

    listStrings = listOf("Avocado", "Pineapple", "Plum", "Coconut")
)

@Strings(languageTag = Locales.PT)
val PtStrings = Strings(/* pt strings */)

@Strings(languageTag = Locales.ES)
val EsStrings = Strings(/* es strings */)

@Strings(languageTag = Locales.RU)
val RuStrings = Strings(/* ru strings */)
```

Lyricist will generate the `LocalStrings` property, a [CompositionLocal](https://developer.android.com/reference/kotlin/androidx/compose/runtime/CompositionLocal) that provides the strings of the current locale. It will also generate `rememberStrings()` and `ProvideStrings()`, call them to make `LocalStrings` accessible down the tree.
```kotlin
val lyricist = rememberStrings()

ProvideStrings(lyricist) {
    // Content
}
```

Now you can use `LocalStrings` to retrieve the current strings.
```kotlin
val strings = LocalStrings.current

// Simple simple
Text(text = strings.simpleString)

// Annotated string
Text(text = strings.annotatedString)

// Parameter string
Text(text = strings.parameterString(lyricist.languageTag))

// Plural string
Text(text = strings.pluralString(2))
Text(text = strings.pluralString(1))

// List string
Text(text = strings.listStrings.joinToString())
```

Use the Lyricist instance provided by `rememberStrings()` to change the current locale. This will trigger a [recomposition](https://developer.android.com/jetpack/compose/mental-model#recomposition) that will update the strings wherever they are being used.
```kotlin
lyricist.languageTag = Locales.PT
```

**Important:** Lyricist uses the System locale as default. It won't persist the current locale on storage, is outside its scope.

### Multi module projects

If you are using Lyricist on a multi module project and the generated artifacts (`LocalStrings`, `rememberStrings()`, `ProvideStrings()`) are too generic for you, provide the `lyricist.moduleName` argument to KSP into the module `build.gradle`.
```gradle
ksp {
    arg("lyricist.moduleName", project.name)
}
```

Let's say you have a "dashboard" module, the generated artifacts will be `LocalDashboardStrings`, `rememberDashboardStrings()` and `ProvideDashboardStrings()`.

## Import to your project
TODO

## Developed by
* [Adriel Café](http://github.com/adrielcafe) | [@adrielcafe](https://twitter.com/adrielcafe)
* [Gabriel Souza](https://github.com/DevSrSouza/) | [@devsrsouza](https://twitter.com/devsrsouza)