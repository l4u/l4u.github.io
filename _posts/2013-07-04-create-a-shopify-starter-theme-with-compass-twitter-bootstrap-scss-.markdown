---
layout: post
title: Create a shopify starter theme
tagline: with Compass, Twitter Bootstrap (SCSS)
tags: [shopify, bootstrap, compass, scss]
category: articles
comments: true
---

This post is a walkthrough of how to create a shopify starter theme. If you want to use this starter theme only, you can skip this tutorial and use the starter theme directly at the [shopify-skeleton-theme-compass-scss-bootstrap Github repo](https://github.com/l4u/shopify-skeleton-theme-compass-scss-bootstrap).

### Why Compass?

Shopify supports using SCSS in the liquid templates. But there are limitations.

-  The assets folder cannot contain sub-folders. Twitter bootstrap SCSS files has to be combined or flattened.
-  @import is not supported.


## Project setup

### Folder Structure
- shopify-skeleton-theme-compass-scss-bootstrap
  - compass-assets
  - skeleton-theme
    - assets
    - config
    - layout
    - snippets
    - templates

### Compass project
~~~ bash
gem install bootstrap-sass
cd shopify-skeleton-theme-compass-scss-bootstrap
compass create compass-assets -r bootstrap-sass --using bootstrap
~~~

### Original skeleton theme
Copy files from the [Shopify skeleton theme](https://github.com/Shopify/skeleton-theme), or add it as a git submodule.

## Integrating Compass project with the skeleton theme
Edit `compass-assets/config.rb`

~~~ ruby
css_dir = "../skeleton-theme/assets/"
images_dir = "../skeleton-theme/assets/"
~~~


### Move files from compass folder to skeleton theme

~~~ bash
mv compass-assets/images/* skeleton-theme/assets/
~~~

### Clean up

You can remove these folder safely.

~~~ bash
rm -rf compass-assets/images
rm -rf compass-assets/stylesheets
~~~

### Appending .liquid extension to the css file

The default filename is style.css. If you will use liquid variables, the filename should be style.css.liquid.

Append `compass-assets/config.rb`

~~~ ruby
on_stylesheet_saved do |filename|
  move_to = filename + ".liquid"
  puts "Moving from #{filename} to #{move_to}"
  FileUtils.mv(filename, move_to)
end
~~~

### Generate the CSS file

~~~ bash
cd compass-assets
compass compile --force
~~~

You should see the following outputs

~~~ bash
create ../skeleton-theme/assets/styles.css
Moving from {path}/styles.css to {path}/styles.css.liquid
~~~

### Rename theme file in the skeleton theme

~~~ bash
rm ./skeleton-theme/assets/style.css.liquid
~~~

Edit `./skeleton-theme/layout/theme.liquid`

rename style.css to styles.css

## The hard work

Edit liquid templates in templates/ , snippets/ and layout/.

## Tools used
-  [bootstrap-sass 2.3](https://github.com/thomas-mcdonald/bootstrap-sass)
-  [Shopify skeleton theme](https://github.com/Shopify/skeleton-theme)
-  [Compass](http://compass-style.org)
