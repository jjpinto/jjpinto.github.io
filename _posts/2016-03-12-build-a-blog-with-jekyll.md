---
layout: post
title: Building a blog with Jekyll
categories: [Blog]
tags: [Jekyll, Node.js, Ruby]
comments: true

---
When I launched the new version of my blog, I wanted things to be as simple as possible. No CMS, no admin UI, no rich-text editor, no databases, etc. Something I could edit with any generic text editor, add some simple markup language and deliver. That's when I found **Jekyll**.

### What Jekyll is
**Jekyll** is a static site generator written in `Ruby`. It generates static html pages. The page is presented through several templates and then fires the whole site, were articles are written in a text markup language like Textile or Markdown through the liquid converters to generate fully generated compiled website

On a side note, [GitHubPages](http://pages.github.com/) are powered by **Jekyll**. Therefore I have a simple workflow with fewer dependencies, no server-side language nor database and, best of all, it's hosted on *GitHub*. Also, I can direct my blogsite domain by creating a *CNAME* file.


### Running Jekyll on Windows

While Windows is not officially supported, it is possible to get **Jekyll** running on Windows. Thanks are offered to Julian Thilo, who has written up [instructions](http://jekyll-windows.juthilo.com/) to get **Jekyll** to work. Installation was simple and I had no issues whatsoever.

I dug a bit further after the installation and decided to install __`Ruby-based Rouge`__, which is faster and easier to install than *Pigments*, which requires Python installation. They are both syntax highlighters.


### Jekyll site

Here’s how to get a boilerplate **Jekyll** site up and running. 

{% highlight csharp %}
~ $ gem install jekyll
~ $ jekyll new myblog
~ $ cd myblog
~/myblog $ jekyll serve

Now browse to http://localhost:4000
{% endhighlight %}

An overview of the initial can be seen through [directory structure](http://jekyllrb.com/docs/structure/). 

### Deploying Jekyll to GitHub Pages

Setting up a [GitHubPages](http://pages.github.com/) website couldn’t be simpler. It consists only of creating a repo with a name in the format username.github.com. By the way, a *GitHub* account is required.

* You can push raw **Jekyll** source to your repo and GitHub Pages will automagically compile it through **Jekyll**. 

Customize your blog with your name, avatar and social links by editing the `_config.yml` file and then you should be able to publish your first blog post by editing an markdown (.md) file. Edit `/_posts/XX-hello-world.md` to publish your first blog post. 

### Conclusion

That’s my introduction to **Jekyll**, the simple, blog aware, static-site generator. If you are trying to build a blog and are having a problem getting something to work or want to know why I set up something in a certain way, please ask a question in the comments below.


- - -
<a class="btn btn-default" href="https://github.com/jjpinto/jjpinto.github.io">You can find my blog project at GitHub</a>


