## Credits

This is my blog, based on [Clean blog jekyll](http://startbootstrap.com/template-overviews/clean-blog-jekyll/)

I find github pages + jekyll + markdown to fit my writing workflow perfectly.

## Installation & Setup

### Using RubyGems:

When installing the theme using RubyGems, demo images, posts, and pages are not included. Follow the instructions below for complete setup.

1. (Optional) Create a new Jekyll site: `jekyll new my-site`
2. Replace the current theme in your `Gemfile` with `gem "jekyll-theme-clean-blog"`.
3. Install the theme: `bundle install`
4. Replace the current theme in your `_config.yml` file with `theme: jekyll-theme-clean-blog`.
5. Build your site: `bundle exec jekyll serve`

Assuming there are no errors and the site is building properly, follow these steps next:

1. Create the following pages if they do not exist already (or change the extension of exisitig markdown files from `.md` to `.html`):
   - `index.html` - set to `layout: home`
   - `about.html` - set to `layout: page`
   - `contact.html` - set to `layout: page`
   - `posts/index.html` - set to `layout: page` (you will also need to create a `posts` directory)
2. Configure the `index.html` front matter. Example:
```
---
layout: home
background: '/PATH_TO_IMAGE'
---
```
3. Configure the `about.html`, `contact.html`, and `posts/index.html` front matter. Example:
```
---
layout: page
title: Page Title
description: This is the page description.
background: '/PATH_TO_IMAGE'
---
```
4. For each post in the `_posts` directory, update the front matter. Example:
```
---
layout: post
title: "Post Title"
subtitle: "This is the post subtitle."
date: YYYY-MM-DD HH:MM:SS
background: '/PATH_TO_IMAGE'
---
```

For reference, look at the [demo repository](https://github.com/BlackrockDigital/startbootstrap-clean-blog-jekyll) to see how the files are set up.

6. Build your site: `bundle exec jekyll serve`

### Using Core Files

When using the core files, the demo images, posts, and pages are all included with the download. After following the instructions below, you can then go and change the content of the pages and posts.

1. Clone the repository.
2. Update the following configuration settings in your `_config.yml` file:
   - `baseurl`
   - `url`
   - `title`
   - `email` (after setting this setting to a working email address, fill out the form on the contact page and send it - then check your email and verify the address and the form will send you messages when used)
   - `description`
   - `author`
   - `twitter_username` (Optional)
   - `facebook_username` (Optional)
   - `github_username` (Optional)
   - `linkedin_username` (Optional)
3. Build your site: `bundle exec jekyll serve`


## Copyright and License

Copyright 2013-2019 Blackrock Digital LLC. Code released under the [MIT](https://github.com/BlackrockDigital/startbootstrap-clean-blog-jekyll/blob/gh-pages/LICENSE) license.
