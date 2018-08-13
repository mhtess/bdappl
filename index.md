---
layout: default
---

[Note: This course is still being developed. Any feedback would be greatly appreciated and can be sent to `tessler@mit.edu`]


Learning statistics is like learning pottery. With pottery, you can learn how to make different shapes (e.g. a bowl, a vase, a spoon) without understanding general principles. The other way is to learn the basic strokes of forming pottery (e.g. how to mold a curved surface, a flat surface, long pointy things). In this course, we are going to learn the basic strokes of statistics, and compose those strokes to make shapes you've seen before (e.g. a t-test), some shapes you've probably never seen before, and develop ideas how you would make new shapes if you needed to. We won't learn *what tests apply to what data types* but instead foster the ability to reason through data analysis. We will do this through the lens of Bayesian statistics, though the basic ideas will aid your understanding of classical (frequentist) statistics as well.



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

## Appendix

{% assign sorted_pages = site.pages | sort:"name" %}

{% for p in sorted_pages %}
    {% if p.hidden %}
    {% else %}
        {% if p.layout == 'appendix' %}
1. **<a class="chapter-link" href="{{ site.baseurl }}{{ p.url }}">{{ p.title }}</a>**<br>
        <em>{{ p.description }}</em>
        {% endif %}
    {% endif %}
{% endfor %}

## Citation

M. H. Tessler (in prep). *Bayesian data analysis: An introduction using probabilistic programs*. Retrieved <span class="date"></span> from https://mhtess.github.io/bdappl/

## Useful resources

#### WebPPL support and packages

- [webppl.org](http://webppl.org): An online editor for WebPPL
- [WebPPL documentation](http://webppl.readthedocs.io/en/master/)
- [WebPPL dev Google Group](https://groups.google.com/forum/?utm_medium=email&utm_source=footer#!forum/webppl-dev): Public forum for discussing issues with WebPPL
- [WebPPL-viz](http://probmods.github.io/webppl-viz/): A summary of the vizualization options in WebPPL
- [RWebPPL](https://github.com/mhtess/rwebppl): If you would rather use WebPPL within R
- WebPPL [packages](http://webppl.readthedocs.io/en/dev/packages.html) (e.g. csv, json, fs).
- [A WebPPL package with useful BDA helper functions](https://github.com/mhtess/webppl-bda)

#### Basic WebPPL tutorials

- [WebPPL intro from DIPPL](http://dippl.org/chapters/02-webppl.html).
- [WebPPL intro from AgentModels](http://agentmodels.org/chapters/2-webppl.html).

#### Bayesian Data Analysis (using WebPPL)

- [Probabilities and Bayes Rule in WebPPL](http://www.problang.org/chapters/app-01-probability.html) by Michael Franke
- [Comparing methods for computing Bayes Factors](http://michael-franke.github.io/statistics,/modeling/2017/07/07/BF_computation.html) by Michael Franke
- [BDA of Bayesian language models](http://www.problang.org/chapters/app-04-BDA.html)
- [Old BDA course syllabus](http://web.stanford.edu/class/psych201s/) by MH Tessler

#### Other WebPPL applications

- [Probabilistic Models of Cognition](http://probmods.org/): An introduction to computational cognitive science and the probabilistic programming language WebPPL
- [Probabilistic Language Understanding](http://problang.org): An introduction to probabilistic models of language (in particular, the Rational Speech Act theory)
- [Modeling Agents with Probabilistic Programs](http://agentmodels.org): An introduction to formal models of rational agents using WebPPL
- [Forest](http://forestdb.org): A Repository for probabilistic models

### Great textbooks on Bayesian Data Analysis

- [Doing Bayesian Data Analysis](https://sites.google.com/site/doingbayesiandataanalysis/) (Kruschke)
- [Bayesian Data Analysis](http://www.stat.columbia.edu/~gelman/book/) (Gelman)
- [Bayesian Cognitive Modeling](https://bayesmodels.com) (Lee & Wagenmakers)


