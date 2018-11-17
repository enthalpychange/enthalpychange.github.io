---
layout: post
title:  "Basic tagging with Jekyll"
tags:
- jekyll
---
As you can probably tell already, this blog is hosted on GitHub Pages, which means I'm using Jekyll. A base installation
doesn't provide much functionality. Tagging is pretty essential to a blog, and it can be done in a few lines of code. In this post, I will show the simple method I'm currently using.

Each post can have multiple tags in its [Front Matter][frontmatter]:

{% highlight markdown %}
---
layout: post
title:  "Basic tagging with Jekyll"
tags:
- jekyll
- blog
---
{% endhighlight %}

Each post's tags are stored in the <i>page.tags</i> variable, and the tags across all posts are stored in <i>site.tags</i>.

In the post layout, I added this snippet of code below the content:

{% highlight html %}
{% raw %}
<div class="tags">
  Tags:
  {% for tag in page.tags %}
    <a href="/tags/#{{ tag }}">{{ tag }}</a>
  {% endfor %}
  {% if page.tags.size == 0 %}
    None
  {% endif %}
</div>
{% endraw %}
{% endhighlight %}

It iterates over <i>page.tags</i> and places a link to the specific tag on the [tags][tags] page.
If there are no tags, it will just say "None."

The tags page contains a nested for-loop to iterate through the posts for each tag:

{% highlight html linenos %}
{% raw %}
---
layout: page
title: Tags
permalink: /tags/
---

{% assign sorted_tags = site.tags | sort %}
{% for tag in sorted_tags %}
  <h4 id="{{ tag[0] }}"><b>{{ tag[0] }}</b></h4>
  <ul>
  {% for page in tag[1] %}
    <li><a href="{{ page.url }}">{{ page.title }}</a></li>
  {% endfor %}
  </ul>
{% endfor %}
{% endraw %}
{% endhighlight %}

<b>Line 4:</b> permalink allows the page to be accessible at the /tags URL.<br/>
<b>Line 7:</b> <i>site.tags</i> are sorted alphabetically and assigned to <i>sorted_tags</i>.<br/>
<b>Line 9:</b> <i>tag[0]</i> contains the tag's name<br/>
<b>Line 11:</b> <i>tag[1]</i> contains the pages with the tag<br/>
<b>Line 12:</b> put a link for each page

With just the few lines of code shown above, there is now a /tags page that lists all the tags and posts.
The tags in each post link back to the /tags page, allowing the reader to find related content.
There are many ways to improve upon this design, but this covers the basic requirements.

[frontmatter]: https://jekyllrb.com/docs/frontmatter/
[tags]: /tags
