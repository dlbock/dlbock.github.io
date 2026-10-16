# dlbock.github.io

The source code for [https://dlbock.github.io](https://dlbock.github.io) is powered by [Jekyll](https://github.com/jekyll/jekyll).

## Running locally

This project uses Ruby 2.7.8 (specified in `.ruby-version`) and Bundler 2.2.33. With [rbenv](https://github.com/rbenv/rbenv) installed:

```sh
rbenv install 2.7.8 # if Ruby 2.7.8 is not already installed
gem install bundler -v 2.2.33
bundle install
bundle exec jekyll serve
```

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