---
title: Only for testing
my_variable: /etc/passwd
---

# Item Title

{{ _item.title }}

# Passwd file

<%= File.open('/etc/passwd').read %>

# Passwd file 2nd method

{%- include {{ page.my_variable }} param='value' -%}

# Dump all config variables

{{config}}

# Basic 7*7

{%- 7 * 7 -%}

# Self

{{self}}