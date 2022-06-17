---
title: Only for testing
---

# Item Title

{{ _item.title }}

# Passwd file

<%= File.open('/etc/passwd').read %>

# Passwd file 2nd method

{%- include "../../../etc/passwd" param='value' -%}

# Dump all config variables

{{config.iteritems()}}

# Basic 7*7

<%= 7 * 7 %>