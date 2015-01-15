## Ozwillo documentation contributions

The doc is served on <a href="http://doc.ozwillo.com" target="_blank">doc.ozwillo.com</a>

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

##### Processing less files

jekyll-less is not supported by gh-pages, that's why processing less files is done independently from jekyll.

If you edit the css/main.less file, you should then follow these additional installation steps (after having install node and npm first):
```
sudo npm install -g less
sudo npm install -g onchange
```

And then automatically track changes with:
```
onchange "css/main.less" -- lessc css/main.less > css/main.css
```

##### Provide DOM class and id

Kramdown is used so that we can add class and id on created DOM elements. For example to generate a `<p class="focus">paragraph</p>`:
```
paragraph
{: .focus}
```

More info [here](http://kramdown.gettalong.org/quickref.html#block-attributes).

### Credits

The styling and behavior of the documentation is deeply inspired by [meteor docs](http://docs.meteor.com/). The CSS relies on [normalize](git.io/normalize) and [typeplate](http://typeplate.com).