# Musings of a Coder

This is my personal blog. I plan to write small tidbits in it whenver I feel like. The blog is intended to be used in github pages. It's built with Jekyll.

## Theme
Using the `jekyll-swiss` theme. It seems to be a nice theme for writing heavy blogs.

## Theme Customization
### Syntax Highlighting
Jekyll supports syntax highlighting out of the box now. But a stylesheet needs to be imported. I had to override `style.md` of the gem based `jekyll-swiss` theme. I copied the file from the gem and placed it under `assets`.
The syntax highlighting stylesheet is `assets/tango.css`. Imported in the overridded `style.md`.

### Favicon
Added a favicon by ovverriding `_includes/head.html`.

### Footer Copy
Overrode `_includes/footer.html` to modify the footer text a little.
