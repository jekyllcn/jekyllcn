---
layout: docs
title: Continuous Integration 持续集成
permalink: /docs/continuous-integration/
---

You can easily test your website build against one or more versions of Ruby.
对应于 Ruby 的一个或多个版本，你很容易就可以测试你的网站。
The following guide will show you how to set up a free build environment on
[Travis][0], with [GitHub][1] integration for pull requests. Paid
alternatives exist for private repositories.

以下指引将展示怎样在 [Travis][0] 上建立一个免费的，集成了处理 pull 请求的 [GitHub][1] 的构建环境。使用私有代码库的话，也有对应的付费选择。

[0]: https://travis-ci.org/
[1]: https://github.com/

## 1. Enabling Travis and GitHub
## 1. 启用 Travis 以及 Github

Enabling Travis builds for your GitHub repository is pretty simple:
启用 Travis 来构建你的 Github 代码库非常简单：

1. Go to your profile on travis-ci.org: https://travis-ci.org/profile/username
2. Find the repository for which you're interested in enabling builds.
3. Click the slider on the right so it says "ON" and is a dark grey.
4. Optionally configure the build by clicking on the wrench icon. Further
   configuration happens in your `.travis.yml` file. More details on that
   below.

## 2. The Test Script

The simplest test script simply runs `jekyll build` and ensures that Jekyll
doesn't fail to build the site. It doesn't check the resulting site, but it
does ensure things are built properly.

When testing Jekyll output, there is no better tool than [html-proofer][2].
This tool checks your resulting site to ensure all links and images exist.
Utilize it either with the convenient `htmlproof` command-line executable,
or write a Ruby script which utilizes the gem.

### The HTML Proofer Executable

{% highlight bash %}
#!/usr/bin/env bash
set -e # halt script on error

bundle exec jekyll build
bundle exec htmlproof ./_site
{% endhighlight %}

Some options can be specified via command-line switches. Check out the
`html-proofer` README for more information about these switches, or run
`htmlproof --help` locally.

### The HTML Proofer Library

You can also invoke `html-proofer` in Ruby scripts (e.g. in a Rakefile):

{% highlight ruby %}
#!/usr/bin/env ruby

require 'html/proofer'
HTML::Proofer.new("./_site").run
{% endhighlight %}

Options are given as a second argument to `.new`, and are encoded in a
symbol-keyed Ruby Hash. For more information about the configuration options,
check out `html-proofer`'s README file.

[2]: https://github.com/gjtorikian/html-proofer

## 3. Configuring Your Travis Builds

This file is used to configure your Travis builds. Because Jekyll is built
with Ruby and requires RubyGems to install, we use the Ruby language build
environment. Below is a sample `.travis.yml` file, followed by
an explanation of each line.

**Note:** You will need a Gemfile as well, [Travis will automatically install](http://docs.travis-ci.com/user/languages/ruby/#Dependency-Management) the dependencies based on the referenced gems:

{% highlight ruby %}
source "https://rubygems.org"

gem "jekyll"
gem "html-proofer"
{% endhighlight %}


{% highlight yaml %}
language: ruby
rvm:
- 2.1
# Assume bundler is being used, install step will run `bundle install`.
script: ./script/cibuild

# branch whitelist
branches:
  only:
  - gh-pages     # test the gh-pages branch
  - /pages-(.*)/ # test every branch which starts with "pages-"

env:
  global:
  - NOKOGIRI_USE_SYSTEM_LIBRARIES=true # speeds up installation of html-proofer
{% endhighlight %}

Ok, now for an explanation of each line:

{% highlight yaml %}
language: ruby
{% endhighlight %}

This line tells Travis to use a Ruby build container. It gives your script
access to Bundler, RubyGems, and a Ruby runtime.

{% highlight yaml %}
rvm:
- 2.1
{% endhighlight %}

RVM is a popular Ruby Version Manager (like rbenv, chruby, etc). This
directive tells Travis the Ruby version to use when running your test
script.

{% highlight yaml %}
script: ./script/cibuild
{% endhighlight %}

Travis allows you to run any arbitrary shell script to test your site. One
convention is to put all scripts for your project in the `script`
directory, and to call your test script `cibuild`. This line is completely
customizable. If your script won't change much, you can write your test
incantation here directly:

{% highlight yaml %}
install: gem install jekyll html-proofer
script: jekyll build && htmlproof ./_site
{% endhighlight %}

The `script` directive can be absolutely any valid shell command.

{% highlight yaml %}
# branch whitelist
branches:
  only:
  - gh-pages     # test the gh-pages branch
  - /pages-(.*)/ # test every branch which starts with "pages-"
{% endhighlight %}

You want to ensure the Travis builds for your site are being run only on
the branch or branches which contain your site. One means of ensuring this
isolation is including a branch whitelist in your Travis configuration
file. By specifying the `gh-pages` branch, you will ensure the associated
test script (discussed above) is only executed on site branches. If you use
a pull request flow for proposing changes, you may wish to enforce a
convention for your builds such that all branches containing edits are
prefixed, exemplified above with the `/pages-(.*)/` regular expression.

The `branches` directive is completely optional.

{% highlight yaml %}
env:
  global:
  - NOKOGIRI_USE_SYSTEM_LIBRARIES=true # speeds up installation of html-proofer
{% endhighlight %}

Using `html-proofer`? You'll want this environment variable. Nokogiri, used
to parse HTML files in your compiled site, comes bundled with libraries
which it must compile each time it is installed. Luckily, you can
dramatically decrease the install time of Nokogiri by setting the
environment variable `NOKOGIRI_USE_SYSTEM_LIBRARIES` to `true`.

<div class="note warning">
  <h5>Be sure to exclude <code>vendor</code> from your
   <code>_config.yml</code></h5>
  <p>Travis bundles all gems in the <code>vendor</code> directory on its build
   servers, which Jekyll will mistakenly read and explode on.</p>
</div>

{% highlight yaml %}
exclude: [vendor]
{% endhighlight %}

### Questions?

This entire guide is open-source. Go ahead and [edit it][3] if you have a
fix or [ask for help][4] if you run into trouble and need some help.

[3]: https://github.com/jekyll/jekyll/edit/master/site/_docs/continuous-integration.md
[4]: https://github.com/jekyll/jekyll-help#how-do-i-ask-a-question