# dlbock.github.io

The source code for [https://dlbock.github.io](https://dlbock.github.io) is powered by [Jekyll](https://github.com/jekyll/jekyll).

## Running locally

This project uses Ruby 3.3.6 (specified in `.ruby-version`) and Bundler 4.0.23. With [rbenv](https://github.com/rbenv/rbenv) installed:

```sh
rbenv install 3.3.6 # if Ruby 3.3.6 is not already installed
gem install bundler -v 4.0.23
CFLAGS=-Wno-default-const-init-field-unsafe bundle install
bundle exec jekyll serve
```

The `CFLAGS` setting works around an Apple Clang warning treated as an error while building the locked Nokogiri version. The project pins Logger 1.5 and includes WEBrick for compatibility with its GitHub Pages Jekyll version on newer Ruby releases.

Then open <http://127.0.0.1:4000> in your browser.

## License

The following directories and their contents are **&copy; Dahlia Bock**. You may not reuse anything therein without my permission:

```
about.md
_posts/
assets/*
```

All other directories and files are MIT Licensed. Feel free to use the HTML and CSS as you please.

## GitHub Pages

To update `github-pages`

```
$ bundle install
$ bundle update github-pages
```