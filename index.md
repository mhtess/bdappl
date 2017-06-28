---
layout: default
---

Learning statistics is like learning pottery. With pottery, you can learn how to make different shapes (e.g. a bowl, a vase, a spoon) without understanding general principles. The other way is to learn the basic strokes of forming pottery (e.g. how to mold a curved surface, a flat surface, long pointy things). In this course, we are going to learn the basic strokes of statistics, and compose those strokes to make shapes you've seen before (e.g. a t-test), some shapes you've probably never seen before, and develop ideas how you would make new shapes if you needed to. We won't learn *what tests apply to what data types* but instead foster the ability to reason through data analysis. We will do this through the lens of Bayesian statistics, though the basic ideas will aid your understanding of classical (frequentist) statistics as well.

[Note: This course is still being developed.]


## Chapters

{% assign sorted_pages = site.pages | sort:"name" %}

{% for p in sorted_pages %}
    {% if p.hidden %}
    {% else %}
        {% if p.layout == 'chapter' %}
1. **<a class="chapter-link" href="{{ site.baseurl }}{{ p.url }}">{{ p.title }}</a>**<br>
        <em>{{ p.description }}</em>
        {% endif %}
    {% endif %}
{% endfor %}

## Citation

M. H. Tessler (2017). *Bayesian data analysis: An introduction using probabilistic programs*. Retrieved <span class="date"></span> from https://mhtess.github.io/bdappl/

## Useful resources

- [Probabilistic Models of Cognition](https://probmods.org): An introduction to computational cognitive science and the probabilistic programming language WebPPL
- [The Design and Implementation of Probabilistic Programming Languages](http://dippl.org): An introduction to probabilistic programming languages, WebPPL in particular
- [Modeling Agents with Probabilistic Programs](http://agentmodels.org): An introduction to formal models of rational agents using WebPPL
- [Pragmatic language interpretation as probabilistic inference](http://langcog.stanford.edu/papers_new/goodman-2016-underrev.pdf): A recent review of the language understanding as probabilistic inference
- [webppl.org](http://webppl.org): An online editor for WebPPL
- [WebPPL documentation](http://webppl.readthedocs.io/en/master/)
- [WebPPL-viz](http://probmods.github.io/webppl-viz/): A summary of the visualization options in WebPPL
- [Forest](http://forestdb.org): A Repository for probabilistic models
- [RWebPPL](https://github.com/mhtess/rwebppl): If you would rather use WebPPL within R
