---
title: Exercícios de mineração de conhecimento do Azure
permalink: index.html
layout: home
---

# Exercícios de mineração de conhecimento do Azure

Os exercícios a seguir foram projetados para dar suporte aos módulos no Microsoft Learn.

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Exercises'" %} {% for activity in labs  %}
- [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) {% endfor %}
