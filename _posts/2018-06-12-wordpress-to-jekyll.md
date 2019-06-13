---
layout: post
title: 'Migrating a blog from Wordpress to Jekyll'
---
# Migrating a blog from Wordpress to Jekyll
So I've decided it's time to start writing again.  As a result since taking this decision I've done everything I can to procrastinate and avoid writing.  The bulk of this time has been to move my blog from self hosted Wordpress over to Jekyll on GitHub pages.

## A quick word on the why
Wordpress is a very capable platform with huge support and tons of plugins and themes.  I would absolutely recommend it to anyone starting out who just wants to get a blog up and going.

It is however big, slow and cumbersome.  An absurd amount of the web is powered by Wordpress and so it has to be all things to all people and it's gotten bulky.  My blog is a collection of maybe 30 pages that aren't going to change once they're up.  

Google has a nice tool which assesses the speed of your site and a bunch of other metrics. My site was not scoring well.

![](/assets/images/slow-mobile.png)
![](/assets/images/slow-desktop.png)

I'd also been hearing a lot about people using a tool called Gatsby to make their blog, which if I'm honest is what started me down this rabbit hole.

### So why not Gatsby?
I get that a static site generator is going to require some technical know how and a bit more work, but after even a light look into the world of Gatsby it strikes me as huge overkill for my little blog.  I just wanted to run some commands and have a static blog locally, then tweak it a bit and push.  Every "Getting started with Gatsby" article seemed to involve a small novel to get going.  I'm sure it's great, but I just wasn't willing to put the effort in.

The other major advantage of Jekyll is that it is natively supported (encouraged even) for GitHub Pages.  This means now all I need to do is commit my posts to GitHub and it's auto built and deployed to my site.  Added bonus: it comes with HTTPS out the box.

## Ok, so what is Jekyll?
It's a static site generator.  You write your posts/pages in markdown (actual .md files!), and it smushes them up with some templates and spits out a super fast static website.  You can configure the site and templates as you wish (change the templates, content, css, add javascript, whatever you like).  It's written in Ruby (boo) but you don't need to know any to use it so that's a plus.

## Where to start
So first things first; this has already been done before and blogged.  I found [this post](https://girliemac.com/blog/2013/12/27/wordpress-to-jekyll/) and [this post](https://blog.webjeda.com/wordpress-to-jekyll-migration/) particularly useful.  However I hit issues which neither covered, and I figured it can't hurt to have another post.

Caveat emptor: I'm on a mac.

First things first, you need to upgrade your Ruby version to something about 2.6.X.  On a Mac to do this you need to install RVM. 
{% highlight bash %}

\curl -sSL https://get.rvm.io | bash -s stable

{% endhighlight %}

Now you have RVM you can upgrade Ruby.

{% highlight bash %}
rvm install ruby-2.6.3

{% endhighlight %}

Hopefully your Ruby version is now up to scratch.

{% highlight bash %}
(base) Sams-MacBook-Pro:sam sam$ ruby -v
ruby 2.6.3p62 (2019-04-16 revision 67580) [x86_64-darwin18]
{% endhighlight %}


Now we can install Jekyll itself.  

{% highlight bash %}
gem install jekyll-importer
gem install jekyll-import
gem install jekyll bundler

{% endhighlight %}

Navigate to wherever you store your code, and we'll create a new jekyll project.  Use your site name; if you've got your own domain, name it that.

{% highlight bash %}
jekyll new yourdomain.io
{% endhighlight %}

You'll have a new folder called yourname.io with your site in. cd into it.  We can now run this.

{% highlight bash %}
bundle exec jekyll serve
{% endhighlight %}

This magic command hosts your site locally at localhost:4000 and allows you to see your wonderful site.  Obviously it's a little raw, but let's get it up onto GitHub then we'll start customising it.

### To GitHub!
GitHub allows you one pages site per user account (you can have them for all your subprojects etc., but only one for your top level GitHub user).  You need to create a repo which is <yrusername>.github.io.  For me, that was samberic.github.io.

On your local project, set it up as a git project and push to your new repo

{% highlight bash %}
git init
git add *
git commit -m "initial commit"
-- follow instructions on github to add remote and push.

{% endhighlight %}
Hopefully you can now got to <yrusername.github.io and see your site in all it's glory. If not, go to your repo on GitHub > settings > GitHub Pages, and you'll hopefully see some sort of useful error message.  

### Customising your theme
What really isn't obvious when you install Jekyll is how to customise it.

![](/assets/images/initial-structure.png)

There are no HTML files? How do you change this? Such confuse!

Turns out that themes are bundled in gem files.  You can read the full doc-o's on the [jekyll site](https://jekyllrb.com/docs/themes/), but basically you need to find where the files are on your file system and copy them in if you want to edit them.  I think this is a valiant attempt to allow you to keep up to date with theme changes upstream, but in reality most people are going to want to customise the heck out of their themes so seems a bit pointless not exploding it locally by default.

Find the name of your theme in config.yml; at the time of writing the default is "minima". On a mac, run

{% highlight bash %}
open $(bundle show minima)
{% endhighlight %}

And all the juicy files will be revealed.  _layouts are the layouts of the various page types (the most pertinent to blogs are "post.html") and _includes are reusable html fragments across the layouts.  copy the same folder structure and files into your project and they will overwrite by default.  At a very minimum you're going to need to copy post.html across so we can add comments later on.

You may want to go with a whole different theme. These can be found at RubyGems at [this link](https://rubygems.org/search?utf8=%E2%9C%93&query=jekyll-theme), but has the downside of no top level previews so it's a bit of a fuss trying to get one you like, although there are more direct demo links from [this GitHub](https://github.com/planetjekyll/awesome-jekyll-themes).  If you like a theme, just change the theme name in your config.yml and jekyll will handle the rest.

Whenever you change your theme you need to do the whole customisation thing again though.

### Exporting old content from Wordpress
I was expecting this to be a nightmare but it turned out to be simple.  There's a great plugin called [wordpress-to-jekyll-exporter](https://ben.balter.com/wordpress-to-jekyll-exporter/) which does just that- it takes all your posts and spits them out into jekyll suitable files.  It will put them in a Zip.  Simply unzip this, and copy all your posts out into the _posts directory of your theme.

If you refresh your page locally you should now have your entire Wordpress blog locally in a jekyll site.

Small note: I got hit with errors because my old posts contained curly braces from code.  Where this is the case you need to wrap the whole thing in a raw tag

{% highlight bash %}


 {{ "{% raw " }}%}
{ your code here }
 {{ "{% endraw " }}%} 


{% endhighlight %}

### Setting up comments
I'm going to assume you're already on the disqus comments system.  If not, sign up for Disqus (it's a really great comments engine) and follow [their guide](https://help.disqus.com/articles/1717131-importing-comments-from-wordpress) on how to do it.

Assuming you're already on disqus, you just need to paste the [Disqus universal code](http://docs.disqus.com/developers/universal/) into your post.html wherever you want your comments to show.  Make sure you've selected in Disqus the correct site if you've got multiple sites! I got thrown for a loop because Disqus got confused by my account and gave me a busted universal code.  Ensure that whatever is under the src part resembles your blog name in disqus!


{% highlight javascript %}
s.src = 'https://sambastuff.disqus.com/embed.js'; # Make sure this line looks like your site!

{% endhighlight %}
_Important point to note_: Locally you won't see any comments. I think that's because Disqus checks its being served up from the correct site it was set up as.  This means you're going to need to push up to GitHub to see it work.  Also, presuming like me you have a custom domain, you need to point your custom domain at GitHub pages and put your url in the GitHub pages settings for them to show up.

I think that's everything; if you hit issues drop a comment and I'll try and help!