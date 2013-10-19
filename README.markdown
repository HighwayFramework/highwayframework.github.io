# Highway Framework Website Setup

The following commands clone the `source` branch, which contains our source code in a local directory, 
and then clones the `master` into the `_deploy` directory beneath that.  This will ensure you don't mark
a post as updated just because you've never generated it before.

```
git clone <repo_url> <local_dir_name> -b source
cd <local_dir_name>
git clone <repo_url> _deploy -b master
```

Once you've done this, there are three basic commands you'll want to know:

* `rake generate` - will compile the website on your box to the `public` directory
* `rake preview` - will start a web server pointed at `public` hosted at [http://localhost:4000](http://localhost:4000)
* `rake deploy` - will update `_deploy` from github, copy `public` to `_deploy`, and push changes to GitHub.

## Setup on Windows

If you're using Windows, you will want to make sure you've got Python installed (`cinst python`) as it is required for rendering code blocks.

Also you might want to install Pik, a ruby version picker for Windows.  Apparently they have a Chocolatey package at `cinst pik` but I installed it manually from their website at [https://github.com/vertiginous/pik](https://github.com/vertiginous/pik).

## General Setup

I recommend running Ruby 1.9.3, it is the most stable choice.

Like most Ruby projects, after you're done cloning, you'll want to `bundle install`

You will need the Ruby developer kit for 1.9.3 to successfully run the bundler, on windows PIK can help with that.

# BELOW THIS LINE IS THE ORIGINAL OCTOPRESS README

## What is Octopress?

Octopress is [Jekyll](https://github.com/mojombo/jekyll) blogging at its finest.

1. **Octopress sports a clean responsive theme** written in semantic HTML5, focused on readability and friendliness toward mobile devices.
2. **Code blogging is easy and beautiful.** Embed code (with [Solarized](http://ethanschoonover.com/solarized) styling) in your posts from gists, jsFiddle or from your filesystem.
3. **Third party integration is simple** with built-in support for Pinboard, Delicious, GitHub Repositories, Disqus Comments and Google Analytics.
4. **It's easy to use.** A collection of rake tasks simplifies development and makes deploying a cinch.
5. **Ships with great plug-ins** some original and others from the Jekyll community &mdash; tested and improved.

**Note**: Octopress requires a minimum Ruby version of `1.9.3-p0`.

## Documentation

Check out [Octopress.org](http://octopress.org/docs) for guides and documentation.


## Contributing

[![Build Status](https://travis-ci.org/imathis/octopress.png?branch=master)](https://travis-ci.org/imathis/octopress)

We love to see people contributing to Octopress, whether it's a bug report, feature suggestion or a pull request. At the moment, we try to keep the core slick and lean, focusing on basic blogging needs, so some of your suggestions might not find their way into Octopress. For those ideas, we started a [list of 3rd party plug-ins](https://github.com/imathis/octopress/wiki/3rd-party-plugins), where you can link your own Octopress plug-in repositories. For the future, we're thinking about ways to easier add them them into our main releases.


## License
(The MIT License)

Copyright © 2009-2013 Brandon Mathis

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the ‘Software’), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED ‘AS IS’, WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.


#### If you want to be awesome.
- Proudly display the 'Powered by Octopress' credit in the footer.
- Add your site to the Wiki so we can watch the community grow.
