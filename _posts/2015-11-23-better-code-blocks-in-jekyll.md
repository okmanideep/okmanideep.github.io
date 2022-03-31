---
layout: post
cover: false
title: Better code blocks in Jekyll
description: 'Your code can look a lot better in your blog powered by Jekyll if you do these simple things. Add line numbers, line highlights in your plain github flavored markdown in all your blog posts.'
date:   2015-11-23 20:10:00
tags: jekyll gh-pages
class: 'post-template'
subclass: 'post tag-jekyll tag-gh-pages'
categories: 'okmanideep'
navigation: True
disqus: 20151123201000
logo: 'logo-white.png'
---

If you are blogging a lot of code, it makes a huge difference if you style your code blocks well. If you are using some Jekyll theme, chances are the default output is not so pleasing. If you are not much familiar with `Jekyll` or `Pygments`, trust me it would take a lot of time to get what you want by just searching the web. I have gathered  all the scattered suggestions I found and arranged them into this post. Let's make your code look better

## Setting up pygments ##
In `_config.yml`

```ruby
markdown: redcarpet
redcarpet:
    extension:
        - fenced_code_blocks
highlighter: pygments
...
...
```

This sets `redcarpet` as our markdown parser and `pygments` as our code highlighter.  
And `fenced_code_blocks` extension enables you to write code blocks like in Github flavoured markdown using {{ "```" }}  

{% highlight text lineanchors %}
{{ "```css" }}
.left {float: left;}

.right {float: right;}
{{ "```" }}
{% endhighlight %}


## Syntax highlighting ##
By now your code probably looks like this

{% highlight text %}
  .left {float: left;}
  
  .right {float: right;}
{% endhighlight %}

Just like plain text and no syntax highlighting based on the language you specified. But if you look at source you can see that `pygments` has analysed the code as css and added some classes to different parts of the code. All we need to do is add some css to those classes.

I have written a [github like color scheme](https://gist.github.com/okmanideep/aa0890c6da9104e16a7a). You can find other styles online for [pygments syntax highlighting css](https://www.google.co.in/search?q=pygments+syntax+highlighting+css).

After adding the sytax highlighting css your code should look like this

{% highlight css %}
  .left {float: left;}
  
  .right {float: right;}
{% endhighlight %}

## Line Numbers ##
As mentioned by [Drew Silcock](https://drewsilcock.co.uk/proper-linenumbers/), [CSS Counters](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Lists_and_Counters/Using_CSS_counters) is the best way to add line numbers to your code blocks. Add css counters to your `pre` blocks using [jekyll-pygments-line-numbers.css](https://gist.github.com/okmanideep/55419a959b7fd39a1c89).

To use this technique we should tell pygments to generate `lineanchors` for the code blocks. We can do this by using liquid template tags for our code blocks

{% highlight text lineanchors %}
{{ "{% highlight css " }}%}
.left {float: left;}

.right {float: right;}
{{ "{% endhighlight " }}%}
{% endhighlight %}

But we want to write our code block using `{{"```"}}css`. We can do this by adding a custom redcarpet parser. Add this `redcarpet-custom.rb` file to your `_plugins` folder

```ruby
module Jekyll
  module Converters
    class Markdown
      class RedcarpetParser
        module WithPygments
          include CommonMethods
          def block_code(code, lang)
            require 'pygments'
            lang = lang && lang.split.first || "text"
            output = add_code_tags(
              Pygments.highlight(code, :lexer => lang,
                                 :options => { :encoding => 'utf-8', :lineanchors =>'line' }),
              lang
            )
          end
        end
      end
    end
  end
end
```

You can add as many [Pygment options](http://pygments.org/docs/formatters/) to the `:options => { ... }` hash. The above ovverrides the [WithPygments module](https://github.com/jekyll/jekyll/blob/master/lib/jekyll/converters/markdown/redcarpet_parser.rb) of the redcarpet parser.

After this your code blocks should look like

```css
.left {float: left;}

.right {float: right;}
```

## Line highlighting ##
...
