<p align="center">
<a href="https://pub.dev/packages/beamer"><img src="https://img.shields.io/pub/v/beamer.svg" alt="pub"></a>
<a href="https://github.com/slovnicki/beamer/blob/master/.github/workflows/test.yml"><img src="https://github.com/slovnicki/beamer/workflows/tests/badge.svg" alt="test"></a>
<a href="https://github.com/google/pedantic"><img src="https://dart-lang.github.io/linter/lints/style-pedantic.svg" alt="style"></a>
</p>

<p align="center">
<a href="https://github.com/slovnicki/beamer/commits/master"><img src="https://img.shields.io/github/commit-activity/m/slovnicki/beamer" alt="GitHub commit activity"></a>
<a href="https://github.com/slovnicki/beamer/issues"><img src="https://img.shields.io/github/issues-raw/slovnicki/beamer" alt="GitHub open issues"></a>
<a href="https://github.com/slovnicki/beamer/issues?q=is%3Aissue+is%3Aclosed"><img src="https://img.shields.io/github/issues-closed-raw/slovnicki/beamer" alt="GitHub closed issues"></a>
<a href="https://github.com/slovnicki/beamer/blob/master/LICENSE"><img src="https://img.shields.io/github/license/slovnicki/beamer" alt="Licence"></a>
</p>

<p align="center">
<a href="https://www.buymeacoffee.com/slovnicki" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png" alt="Buy Me A Coffee" height="30px" width= "108px"></a>
</p>

Handle your application routing, synchronize it with browser URL and more. `Beamer` uses the power of Navigator 2.0 features and implements all the underlying logic for you.

---

- [Key Concepts](#key-concepts)
- [Examples](#examples)
    - [Books](#books)
    - [Advanced Books](#advanced-books)
    - [Deep Location](#deep-location)
    - [Guards](#guards)
    - [BottomNavigationBar](#bottomnavigationbar)
- [Usage](#usage)
  - [With App.router](#with-app.router)
  - [As a Widget](#as-a-widget)
  - [General Notes](#general-notes)
- [Migrating from 0.4.x to >=0.5.x](#migrating-from-0.4.x-to->=0.5.x)
- [Contributing](#contributing)

# Key Concepts

The key concept of Beamer is a `BeamLocation` which represents a stack of one or more pages. You will be extending `BeamLocation` to define your app's locations to which you can then _beam to_ using

```dart
Beamer.of(context).beamTo(MyLocation())
// or context.beamTo(MyLocation())
```

You can also `beamTo` a specific configuration of some location;

```dart
context.beamTo(
  BooksLocation(
    pathBlueprint: '/books/:bookId',
    pathParameters: {'bookId': '2'},
  ),
),
```

You can think of it as _teleporting_ / _beaming_ to another place in your app. Similar to `Navigator.of(context).pushReplacementNamed('/my-route')`, but Beamer is not limited to a single page, nor to a push _per se_. You can create an arbitrary stack of pages that gets build when you beam there.

Using Beamer _can_ feel like using many of `Navigator`'s `push/pop` methods at once.

# Examples

## Books

Here is a recreation of books example from [this article](https://medium.com/flutter/learning-flutters-new-navigation-and-routing-system-7c9068155ade) where you can learn a lot about Navigator 2.0. See [Example](https://pub.dev/packages/beamer/example) for full application code of this example.

<p align="center">
<img src="https://raw.githubusercontent.com/slovnicki/beamer/master/res/example-books-v0.5.0.gif" alt="example-url-sync" style="margin-right:16px;margin-left:16px">

## Advanced Books

For a step further, we add more flows to demonstrate the power of Beamer.

<p align="center">
<img src="https://raw.githubusercontent.com/slovnicki/beamer/master/res/example-books-v0.5.0.gif" alt="example-url-sync" style="margin-right:16px;margin-left:16px">

## Deep Location

You can instantly beam to a location in your app that has many pages stacked and then pop them one by one or simply `beamBack` to where you came from.

<p align="center">
<img src="https://raw.githubusercontent.com/slovnicki/beamer/master/res/example-deep-location.gif" alt="example-url-sync" style="margin-right:32px;margin-left:32px">

## Guards

<p align="center">
<img src="https://raw.githubusercontent.com/slovnicki/beamer/master/res/example-books-v0.5.0.gif" alt="example-url-sync" style="margin-right:16px;margin-left:16px">

## BottomNavigationBar

<p align="center">
<img src="https://raw.githubusercontent.com/slovnicki/beamer/master/res/example-deep-location.gif" alt="example-url-sync" style="margin-right:32px;margin-left:32px">

# Usage

## With App.router

In order to use Beamer on your entire app, you must (as per [official documentation](https://api.flutter.dev/flutter/widgets/Router-class.html)) construct your `*App` widget with `.router` constructor to which (along with all your regular `*App` attributes) you provide

- `routerDelegate` that controls (re)building of `Navigator` pages and
- `routeInformationParser` that decides which URI corresponds to which `Router` state/configuration, in our case - `BeamLocation`.

Here you can just use the Beamer implementation of those - `BeamerRouterDelegate` and `BeamerRouteInformationParser`, to which you pass your `BeamLocation`s. Optional `notFoundPage` will be shown when URI coming from browser is not among the ones that are defined in your `BeamLocation`s.

```dart
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      routerDelegate: BeamerRouterDelegate(
        initialLocation: HomeLocation(),
        notFoundPage: Scaffold(body: Center(child: Text('Not found'))),
      ),
      routeInformationParser: BeamerRouteInformationParser(
        beamLocations: [
          HomeLocation(),
          BooksLocation(),
          ArticlesLocation(),
        ],
      ),
    );
  }
}
```

An example of above book example's `BeamLocation`s would be:

```dart
class HomeLocation extends BeamLocation {
  @override
  List<String> get pathBlueprints => ['/'];

  @override
  List<BeamPage> get pages => [
        BeamPage(
          key: ValueKey('home'),
          child: HomeScreen(),
        ),
      ];
}

class BooksLocation extends BeamLocation {
  BooksLocation({
    String pathBlueprint,
    Map<String, String> pathParameters,
    Map<String, String> queryParameters,
    Map<String, dynamic> data,
  }) : super(
          pathBlueprint: pathBlueprint,
          pathParameters: pathParameters,
          queryParameters: queryParameters,
          data: data,
        );

  @override
  List<String> get pathBlueprints => [
        '/books/:bookId/genres/:genreId',
        '/books/:bookId/buy',
      ];

  @override
  List<BeamPage> get pages => [
        ...HomeLocation().pages,
        if (pathSegments.contains('books'))
          BeamPage(
            key: ValueKey('books-${queryParameters['title'] ?? ''}'),
            child: BooksScreen(
              titleQuery: queryParameters['title'] ?? '',
            ),
          ),
        if (pathParameters.containsKey('bookId'))
          BeamPage(
            key: ValueKey('book-${pathParameters['bookId']}'),
            child: BookDetailsScreen(
              bookId: pathParameters['bookId'],
            ),
          ),
        if (pathSegments.contains('buy'))
          BeamPage(
            key: ValueKey('book-${pathParameters['bookId']}-buy'),
            child: BuyScreen(
              book: data['book'],
            ),
          ),
        if (pathSegments.contains('genres'))
          BeamPage(
            key: ValueKey('book-${pathParameters['bookId']}-genres'),
            child: GenresScreen(
              book: data['book'],
            ),
          ),
        if (pathParameters.containsKey('genreId'))
          BeamPage(
            key: ValueKey('genres-${pathParameters['genreId']}'),
            child: GenreDetailsScreen(
              genre: data['genre'],
            ),
          ),
      ];
}

class ArticlesLocation extends BeamLocation {
  ArticlesLocation({
    String pathBlueprint,
    Map<String, String> pathParameters,
    Map<String, String> queryParameters,
    Map<String, dynamic> data,
  }) : super(
          pathBlueprint: pathBlueprint,
          pathParameters: pathParameters,
          queryParameters: queryParameters,
          data: data,
        );

  @override
  List<String> get pathBlueprints => ['/articles/:articleId'];

  @override
  List<BeamPage> get pages => [
        ...HomeLocation().pages,
        if (pathSegments.contains('articles'))
          BeamPage(
            key: ValueKey('articles'),
            child: ArticlesScreen(),
          ),
        if (pathParameters.containsKey('articleId'))
          BeamPage(
            key: ValueKey('articles-${pathParameters['articleId']}'),
            child: ArticleDetailsScreen(
              articleId: pathParameters['articleId'],
            ),
          ),
      ];
}
```

## As a Widget


## General Notes

- When extending `BeamLocation`, two getters need to be implemented; `pathBlueprints` and `pages`.
  - `pages` represent a stack that will be built by `Navigator` when you beam there, and `pathBlueprints` is there for Beamer to decide which `BeamLocation` corresponds to an URL coming from browser.
  - `BeamLocation` takes query and path parameters from URI. The `:` is necessary in `pathBlueprints` if you _might_ get path parameter from browser.

- `BeamPage`'s child is an arbitrary `Widgets` that represent your app screen / page.
  - `key` is important for `Navigator` to optimize rebuilds. This should be an unique value for "page state".
  - `BeamPage` creates `MaterialPageRoute`, but you can extends `BeamPage` and override `createRoute` to make your own implementation instead. The `key` is important for `Navigator` to optimize its rebuilds.

**Note** that "Navigator 1.0" can be used alongside Beamer. You can easily `push` or `pop` pages with `Navigator.of(context)`, but those will not be contributing to the URI. This is often needed when some info/helper page needs to be shown that doesn't influence the browser's URL. And of course, when using Beamer on mobile, this is a non-issue as there is no URL.

# Migrating from 0.4.x to >=0.5.x

- instead of wrapping `MaterialApp` with `Beamer`, use `*App.router()`
- `String BeamLocation.pathBlueprint` is now `List<String> BeamLocation.pathBlueprints`
- `BeamLocation.withParameters` constructor is removed and all parameters are handled with 1 constructor. See example if you need `super`.
- `BeamPage.page` is now called `BeamPage.child`

# Contributing

This package is still in early stages. To see the upcoming features, check the [Issue board](https://github.com/slovnicki/beamer/issues).

If you notice any bugs not present in issues, please file a new issue. If you are willing to fix or enhance things yourself, you are very welcome to make a pull request. Before making a pull request;

- if you wish to solve an existing issue, please let us know in issue comments first
- if you have another enhancement in mind, create an issue for it first so we can discuss your idea

Also, you can <a href="https://www.buymeacoffee.com/slovnicki" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png" alt="Buy Me A Coffee" height="20px" width= "72px"></a> to speed up the development.

