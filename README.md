# gematik's technology blog


This is a place where we would like to give back to the community by telling our story from the technology front line of the German Healthcare system.

We've seen a lot of technologies come and go in the past years, we also tried a few things in our engineering organization, so maybe people find a few interesting things or even inspiration from that.

## Installation
The blog is based on [github-pages](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/creating-a-github-pages-site-with-jekyll) with [jekyll](https://jekyllrb.com/docs/installation/) and the pages theme ['minimal'](https://github.com/pages-themes/minimal).

Make sure to install the latest version of [Ruby](https://www.ruby-lang.org/en/documentation/installation/) and [Bundler](https://bundler.io/) before running it locally.

## Running locally

'bundle exec jekyll serve'

Alternatively, you can use a [Docker Image](https://github.com/madduci/docker-github-pages) to generate the site, without installing the aforementioned packages (make sure your current directory is the project root):

In Windows using PowerShell:

```sh
docker run --rm -it -p 4000:4000 -v ${PWD}:/site --entrypoint //bin/sh madduci/docker-github-pages -c "bundle install && bundle exec jekyll serve --watch --force_polling --host 0.0.0.0 --incremental"
```

On Linux/Mac:

```sh
docker run --rm -it -p 4000:4000 -v $PWD:/site --entrypoint /bin/sh madduci/docker-github-pages -c "bundle install && bundle exec jekyll serve --watch --force_polling --host 0.0.0.0 --incremental"
```

## Usage
* Clone the repo
* Write your post
* Test the layout and index page locally
* Commit, push, PT


## License

Copyright 2020-2025 gematik GmbH

Apache License, Version 2.0

See the [LICENSE](./LICENSE) for the specific language governing permissions and limitations under the License

## Additional Notes and Disclaimer from gematik GmbH

1. Copyright notice: Each published work result is accompanied by an explicit statement of the license conditions for use. These are regularly typical conditions in connection with open source or free software. Programs described/provided/linked here are free software, unless otherwise stated.
2. Permission notice: Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:
   1. The copyright notice (Item 1) and the permission notice (Item 2) shall be included in all copies or substantial portions of the Software.
   2. The software is provided "as is" without warranty of any kind, either express or implied, including, but not limited to, the warranties of fitness for a particular purpose, merchantability, and/or non-infringement. The authors or copyright holders shall not be liable in any manner whatsoever for any damages or other claims arising from, out of or in connection with the software or the use or other dealings with the software, whether in an action of contract, tort, or otherwise.
   3. The software is the result of research and development activities, therefore not necessarily quality assured and without the character of a liable product. For this reason, gematik does not provide any support or other user assistance (unless otherwise stated in individual cases and without justification of a legal obligation). Furthermore, there is no claim to further development and adaptation of the results to a more current state of the art.
3. Gematik may remove published results temporarily or permanently from the place of publication at any time without prior notice or justification.
4. Please note: Parts of this code may have been generated using AI-supported technology. Please take this into account, especially when troubleshooting, for security analyses and possible adjustments.
