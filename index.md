---
layout: default
---

The present course serves an a practical introduction to the Rational Speech Act modeling framework. Little is presupposed beyond a willingness to explore recent progress in formal, implementable models of language understanding.


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
- [Pragmatic language interpretation as probabilistic inference](http://langcog.stanford.edu/papers_new/goodman-2016-underrev.pdf): A recent review of the RSA framework
- [webppl.org](http://webppl.org): An online editor for WebPPL
- [WebPPL documentation](http://webppl.readthedocs.io/en/master/)
- [WebPPL-viz](http://probmods.github.io/webppl-viz/): A summary of the vizualization options in WebPPL
- [Forest](http://forestdb.org): A Repository for probabilistic models
- [RWebPPL](https://github.com/mhtess/rwebppl): If you would rather use WebPPL within R
- [WebPPL Tutorials](https://github.com/mhtess/webppl-tutorials): Basic tutorials for WebPPL

## Acknowledgments
