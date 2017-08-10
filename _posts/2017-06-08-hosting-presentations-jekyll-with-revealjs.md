---
layout: post
title: Hosting Presentations on Jekyll Powered Blog with Reveal.js
tags: [jekyll, revealjs, presentations, web development]
categories: jekyll
teaser: /public/assets/hello-revealjs.png
published: false
---

Jekyll always fascinated me with it’s simplicity. Besides, I tend to write my articles in markdown and Jekyll gracefully handles markdown. So just few months ago I decided to move my blog from Weebly to Jekyll. 

Turns out setting up blog on Jekyll was not so simple for me. I first used minimal_mistakes theme, and after spending 2 weeks figuring out what goes where and how to customize what, realized that I first need to understand Jekyll. So after spending another week understanding how jekyll works, I picked one simple theme Lanyon by Mark Otto and modified it slightly to fit my needs. It’s working great! Besides Leonides is also great theme.

Coming to presentations part, I recently shared my learning on DynamoDB with my team and chose RevealJS to create minimalistic and simple presentation. I am writing  this article to summarize steps taken to setup RevealJS in my blog.

Luu Gia Thuy created one beautiful presentation from which I took inspiration and basic layout to setup RevealJS in my blog. Not being front-end guy, I didn’t change much in layout and just modified few RevealJS options.

## How:

### Setup RevealJS in Blog Repo (Similar as in Luu Gia Thuy’s presentation)

- Clone or Download from RevealJS repo, and copy into site’s root folder.
- Create a layout file named `slide.html` specific for showing presentation slides. Here is the gist which you can use for presentation. The gist finds
- Modify the gist, especially head section if you want to make any seo optimization or apply site wide meta tags. Place this file in `_layouts` directory of site. 

At this point, you have reveal js added in site’s root directory and have a simple layout file that links with reveal.js. 

Now, to actually add presentations to site, you can create normal post as you would with `front-matter` and `markdown` but with `slide` layout. If you go this route, one big problem you might face would be to filter out separate presentations with other posts as posts and presentations are different things. 

<p align="center">
  <img src="https://media.giphy.com/media/14aUO0Mf7dWDXW/giphy.gif" alt="Panda Furstrated with filtering presentations from post categories, tags and archives">
  <p align="center">
  <em>
  Filtering presentations from post categories, tags and archives be like :(
    </em>
    </p>
</p>

And, filtering out presentations from other normal posts with either tags or categories is painful and error-prone process. Besides, presentations are different  kind of collections; very different from posts in nature. Obviously, in this case Jekyll `collections` is perfect for listing presentation slides.

### Jekyll Collections for listing Presentation Slides

In `_config.yml`, add presentations as collection.

{% highlight liquid %}
collections:
  presentations:
    output: true
    permalink: /presentations/:name
{% endhighlight %}    

Specifying presentations as collections this way tells jekyll that any files that you add in `_presentations` directory is a item of presentations collection. This will make it easier to identify and list presentations.

The option `output: true` is important as it will produce file for each presentation in collection. Read in detail about this option at Jekyll Collection documentation.
 
The option `permalink` simply specifies common url pattern where presentations will be published.

- Create `presentations.html` with `page` as layout at root level of site, and iterate over all presentations of site. Making collection gives us simple variable `site.presentations` to iterate over presentations.

{% highlight liquid %}
---
layout: page
title: Presentations
---
<div class="posts">
  {% for presentation in site.presentations %}
    <li>
      <a href="{{ site.baseurl }}{{ presentation.url }}">
        {{ post.title }}
      </a>
      <span class="entry-date">
        <time datetime="{{ presentation.date | date_to_xmlschema }}" itemprop="datePublished">{{ presentation.date | date: site.date_format }}
        </time>
      </span>
    </li>
  {% endfor %}
</div>
{% endhighlight %}

- Now create actual presentation by specifying layout as slide and theme in front matter. 

{% highlight liquid %}
---
layout: slide
title: "My Jekyll Presentation"
description: "Some detailed description of presentation"
published: true
---
{% endhighlight %}

Enclose all content of slide within <section> tag as shown below:

{% highlight liquid %}
<section data-markdown data-separator="^\n----\n" data-separator-vertical="^\n---\n">
<script type="text/template">
# Welcome
----
# slide 2
### Some content
- bullet points
- another list item
---
### vertical slide of slide 2
##### some more content
----
{% endhighlight %}

RevealJS supports markdown and also accepts some options such as data-separator , data-separator-markdown , data-separator-notes and data-charset along with data-markdown. You can read more about it from reveal.js readme.

![Thanks for Reading!]({{site.url}}{{site.baseurl}}/public/assets/gracias.png)
<p align="center"><em>Thanks for Reading!</em></p>
## Further Customizations

Apart from this, there are number of other useful configuration options you can specify in Reveal.initialize section of slide.html prepared earlier. You can read more from reveal.js#configuration.

Hopefully, this small guide might help you to host beautiful presentations on Jekyll powered blogs or sites.

Share and Tweet :yellow_heart:!
