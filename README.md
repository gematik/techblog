# gematik's technology blog


This is a place where we would like to give back to the community by telling our story from the technology front line of the German Healthcare system.

We've seen a lot of technologies come and go in the past years, we also tried a few things in our engineering organization, so maybe people find a few interesting things or even inspiration from that.

## Installation
The blog is based on [github-pages](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/creating-a-github-pages-site-with-jekyll) with [jekyll](https://jekyllrb.com/docs/installation/) and the pages theme ['minimal'](https://github.com/pages-themes/minimal).

Make sure to install the latest version of [Ruby](https://www.ruby-lang.org/en/documentation/installation/) and [Bundler](https://bundler.io/) before running it locally.

## Running locally

'bundle exec jekyll serve'

Alternatively, you can use a [Docker Image](https://github.com/madduci/docker-github-pages) to generate the site, without installing the aforementioned packages:

In Windows (e.g. with git-bash):

```sh
docker run --rm -it -p 4000:4000 -v /$PWD:/site --entrypoint //bin/sh madduci/docker-github-pages -c "bundle install && bundle exec jekyll serve --watch --force_polling --host 0.0.0.0 --incremental"
```

On Linux/Mac:

```sh
docker run --rm -it -p 4000:4000 -v $pwd:/site --entrypoint /bin/sh madduci/docker-github-pages -c "bundle install && bundle exec jekyll serve --watch --force_polling --host 0.0.0.0 --incremental"
```

## Usage
* Clone the repo
* Write your post
* Test the layout and index page locally
* Commit, push, PT
