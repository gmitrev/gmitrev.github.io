---
layout: post
title: Yanking Webpacker out of Rails (6)
summary: How to use Rails without webpacker
date: 2019-12-27 23:30
categories: ruby
published: true
---

Call me a grumpy old man buy I find webpack(er) to be a major pain in the ass. It overcomplicates
things and turns (previously) simple tasks such as installing bootstrap into a nightmare. This is
why I prefer not to use it for small personal projects.

Yes, it might be better for js-centric apps but not everyone makes SPAs. This post will explain how
to remove `webpacker` and add javascript back to the sprockets pipeline.

Sadly, we can't simply use `rails new --skip-webpack` and call it a day. There is some manual work
involved. Also, a warning: it will be harder to use newer Rails 6 features such as Action Text.

## Creating a new application

When creating a new application, you can supply two flags that prevent `webpacker` from being
installed. The first is `--skip-webpack-install` which prevents the generator from running `rails
webpacker:install`. The second is `--skip-javascript`, which drops the `webpacker` gem from the
Gemfile. The final command is:

```sh
rails new app-name --skip-webpack-install --skip-javascript
```

## Adding javascript to the sprockets pipeline

You will find that the `app/assets/javascripts` folder no longer exists. Create it by running the
following command in your shell:

```sh
mkdir -p app/assets/javascripts
```

Then, create `app/assets/javascripts/application.js` with the following contents (taken from a
Rails 5 app):

```rb
//= require rails-ujs
//= require turbolinks
//= require_tree .
```

*Note*: If you want to use ActionCable, you can also copy a pristine `cable.js` from a Rails 5 application.

Next, open `app/assets/config/manifest.js` and [add the following
line](https://github.com/rails/sprockets/blob/master/UPGRADING.md#manifestjs):

```rb
...

//= link_directory ../javascripts .js
```

Finally, open you application layout (`app/views/layouts/application.html.erb`) and add the
following line:

```rb
<%= javascript_include_tag 'application', 'data-turbolinks-track': 'reload' %>
```

Also, remove lines with `javascript_pack_tag` if there are any. You're done.

## Disclaimer

Webpacker is the future and being ignorant with regard to how it works is probably not a good idea.
I strongly urge everyone to learn how to use it properly. Still, being quite a new feature
(introduced in Rails 6), the tooling and general knowledge base related to it are still quite
limited.
This is why I generally avoid it for personal projects, where I don't want to spend an
hour trying to setup `bootstrap` when it should not take me more than a couple of minutes.
