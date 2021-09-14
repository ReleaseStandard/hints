
Give you some solution to issues.

## Index
<ul>
  {% for post in site.posts %}
    <li>
      <a href="/hints/{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>

<a class="btn btn-rss" href="https://releasestandard.github.io/hints/feed.xml" target="_blank">RSS</a>
