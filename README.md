# My personal blog

Just some notes and thoughts, most of them about programming probably...

## Run Locally

### Requirements

* [jekyll](https://jekyllrb.com/)
* [bundler](https://bundler.io/)

Install them in the user directory:

```bash
gem install --user-install bundler jekyll
```

Create a local "vendor" directory for *gems*:

```bash
bundle config set --local path 'vendor/bundle'
bundle install
```

### Server

``` bash
bundle exec jekyll serve
```
