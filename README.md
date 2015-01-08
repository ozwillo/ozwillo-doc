## Ozwillo documentation contributions

### Installation

The documentation website is processed by Jekyll and served thanks to Github Pages. Please follow the installation instructions [here](https://help.github.com/articles/using-jekyll-with-pages/).

When installing gems, you might prefer to install them locally thanks to:
```
bundle install --path vendor/bundle
```

### Launch Jekyll

To launch Jekyll and process assets:
```
bundle exec jekyll serve
```

### Specifics

##### Processing markdown

It seems including markdown files from index.html with `{% include my-file.md %}` just does a raw paste without processing the markdown. That's why we use (see in index.html) this syntax:
```
{% capture inc0 %}{% include 0-welcome.md %}{% endcapture %}{{ inc0 | markdownify }}
```

See more about this issue [here](https://github.com/jekyll/jekyll/issues/1303).

##### Provide DOM class and id

Kramdown is used so that we can add class and id on created DOM elements.