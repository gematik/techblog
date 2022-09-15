<div>
{% if site.data.navigation.staticnav[0] %}
  {% for item in site.data.navigation.staticnav %}
    <h3>{{ item.title }}</h3>
      {% if item.subfolderitems[0] %}
        <ul>
          {% for entry in item.subfolderitems %}
              <li><a href="{{ entry.url }}">{{ entry.page }}</a><br />
              {{ entry.summary }}
                {% if entry.subsubfolderitems[0] %}
                  <ul>
                  {% for subentry in entry.subsubfolderitems %}
                      <li>
                        <a href="{{ subentry.url }}">{{ subentry.page }}</a>
                        {{ subentry.summary }}
                     </li>
                  {% endfor %}
                  </ul>
                {% endif %}
              </li>
          {% endfor %}
        </ul>
      {% endif %}
    {% endfor %}
{% endif %}
</div>
<h3>Our blog articles</h3>
{% for category in site.categories reversed %}
  <h4>{{ category[0] }}</h4>
<ul>
  {% for post in category[1] %}
    <li>
      <a href="{{ post.url }}">{{ post.title }} - {{ post.date | date_to_string }}</a>
      {{ post.excerpt }}
    </li>
  {% endfor %}
</ul>
{% endfor %}
