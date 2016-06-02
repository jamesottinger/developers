# developers.linode.com

This is the [Linode developer portal](https://developers.linode.com/). It's a jekyll site hosted on GitHub Pages.
Included here are:

* API documentation
* API guides
* API libraries

## Feedback

Feedback on our API, including feature requests and bug reports, may be
submitted as [GitHub issues](https://github.com/linode/developers/issues/new)
on this repository.

## Cloning

This repository uses git submodules. Use a recursive clone:

    git clone --recursive https://github.com/Linode/developers.git

If you already cloned, but missed this step, try this:

    git submodule update --init

## Running

We use GitHub pages. For information about running sites made with GitHub pages,
see [GitHub's docs](https://help.github.com/articles/using-jekyll-as-a-static-site-generator-with-github-pages/).
tl;dr:

    gem install github-pages
    jekyll serve --watch
    # Navigate to http://localhost:4000

## Contributing

Fork this repository and send a pull request, simple as that.
