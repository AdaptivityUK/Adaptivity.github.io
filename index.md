--- 

layout: home
---
<h2>Latest posts </h2>
{% for post in site.posts %}
  <article>
    <h3>
      <a href="{{ post.url }}">
        {{ post.title }}
      </a>
    </h3>
    <p class="post-excerpt">
    {% if post.content contains '<!--excerpt.start-->' and post.content contains '<!--excerpt.end-->' %}
    	{{ ((post.content | split:'<!--excerpt.start-->' | last) | split: '<!--excerpt.end-->' | first) | strip_html | truncatewords: 20 }}
    {% else %}
    	{{ post.content | strip_html | truncatewords: 30 }}
        {% endif %}
        </p>
        <time datetime="{{ post.date | date: "%Y-%m-%d" }}">{{ post.date | date_to_long_string }}</time> 
  </article>
  <hr>
{% endfor %}

<h2>About Adaptivity </h2>
Adaptivity helps clients to adapt and thrive in changing situations

We work with clients at all stages of their digital transformation journey to help them drive forward at pace. We help our clients reduce operating costs, innovate faster and derive greater value from their existing systems.
We deliver seamless integration, throughout all your systems, both on-premise and cloud-based.

You can contact us via mail (hello@adaptiivty.uk) or call us on 01923 549012 

<div data-iframe-width="150" data-iframe-height="270" data-share-badge-id="220078a9-703c-4645-a83a-1c3968886f89" data-share-badge-host="https://www.youracclaim.com"></div><script type="text/javascript" async src="//cdn.youracclaim.com/assets/utilities/embed.js"></script>
