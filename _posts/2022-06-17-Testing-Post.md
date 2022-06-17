---
title: Only for testing
---

# Item Title

{{ _item.title }}

# Passwd file

<%= File.open('/etc/passwd').read %>

# Passwd file 2nd method

{%- include ../../../etc/passwd -%}

# Dump all config variables

{% for key, value in config.iteritems() %}
    <dt>{{ key|e }}</dt>
    <dd>{{ value|e }}</dd>
{% endfor %}

# Basic 7*7

<%= 7 * 7 %>