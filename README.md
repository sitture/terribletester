# ![TerribleTester Logo](terribletester-logo.png)

[![Build Status](https://travis-ci.org/sitture/terribletester.svg?branch=master&style=flat-square)](https://travis-ci.org/sitture/terribletester)

The source of [terribletester.com](http://terribletester.com) | Learnings of a Terrible Tester. Credits to [holo-alfa](https://github.com/stijnvc/holo-alfa) theme with custom additions.

### Features

* Html compression using http://jch.penibelst.de/

## Running Locally

Run the following command to spin the blog locally at `http://localhost:4000`.

```bash
bundle install --path vendor/bundle
bundle exec jekyll serve
```

## Creating Posts

To create a new post, add a file to your `_posts` directory with the following format:

```sh
YEAR-MONTH-DAY-title.md
```

For example, the following are examples of valid post filenames:

```sh
2018-12-31-new-years-eve-is-awesome.md
2018-09-12-how-to-write-a-blog.md
```

## Compiling Styles

The styles are compiled using `Gulp/PostCSS` to polyfill future CSS spec. You'll need Node and Gulp installed globally.

Edit `/assets/css/` files, which will be compiled to `/assets/built/` automatically.

Run the following from the root directory:

```js
npm install
gulp
```

## Contributions

Please [open an issue here](../../issues) on GitHub if you have a suggestion, or other comments.

If you have something to share, why not contribute as an author and write a blog post about it. Please read [CONTRIBUTING.md](CONTRIBUTING.md) for contribution guidelines.
