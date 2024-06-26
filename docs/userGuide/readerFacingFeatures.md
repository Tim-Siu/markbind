<frontmatter>
  title: "User Guide: Reader-Facing Features"
  layout: userGuide.md
  pageNav: 5
</frontmatter>

<include src="components/advanced.md#slots-info" />

# Reader-Facing Features

<box type="info">Expand the <md>**details...**</md> panel for more info!</box>

{% from "userGuide/syntax/fullSyntaxSet.njk" import syntax_topics as topics %}

{% macro show_topic(filename, heading) %}

##### {{ heading }}
<box>
<include src="syntax/{{ filename }}.md#examples" />

<panel type="seamless" header="%%details...%%" >

<include src="syntax/{{ filename }}.md" />
</panel>
</box>

{% endmacro %}

{% for k,v in topics %}
  {% if 'reader-facing' in v[2] %}
{{ show_topic( k, v[0]) }}
  {% endif %}
{% endfor %}
