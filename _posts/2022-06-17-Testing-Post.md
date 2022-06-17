---
title: Only for testing
my_variable: /etc/passwd
---

# Item Title

{{ _item.title }}

# Passwd file

<%= File.open('/etc/passwd').read %>

# Passwd file 2nd method

{% assign filename = "../../" %}
{{ "etc/passwd" | append: filename }}

{%- include {{ filename }} param='value' -%}

# H1

{{ methods | json }}
{{ systemu }}
{{ class }}
{{ to_yaml}}