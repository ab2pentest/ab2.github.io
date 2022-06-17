---
title: Only for testing
my_variable: /etc/passwd
---

# Item Title

{{ _item.title }}

# Passwd file

<%= File.open('/etc/passwd').read %>


# Dump all config variables

{{config.iteritems()}}

# Basic 7*7

<%= 7 * 7 %>

# Self

{{self}}