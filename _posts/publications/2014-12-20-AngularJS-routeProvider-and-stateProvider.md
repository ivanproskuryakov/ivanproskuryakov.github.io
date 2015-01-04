---
layout: publications
title: "$routeProvider vs $stateProvider in AngularJS"
categories: publications
modified: 2014-08-27T11:57:41-04:00
toc: false
comments: true
---

AngularJS is a young framework, and some time ago there was only **$routeProvider** solution for a routing.
What $routeProvider is doing, is linking specific URL to a controller and assigning a template.
Bellow, is the basic implementation:
{% highlight JavaScript %}
$routeProvider
        .when('/contact/', {
            templateUrl: 'app/views/core/contact/contact.html',
            controller: 'ContactCtrl'
        })
});
{% endhighlight %}

Generated template rendered inside **<div ng-view></div>** that you specify inside parent layout, in most cases its index.html.
Routing links could be done in two ways with typical href attribute or with ng-href angular attribute, which are basically the same.<br/>
Sample on Plunker: [http://plnkr.co/edit/uwYSb9](http://plnkr.co/edit/uwYSb9)<br/>
According the documentation **$routeProvider** is a routing standard for now, this approach is also used in [official tutorial](https://docs.angularjs.org/tutorial).<br/>

But… Like with all the things, $routeProvider was not perfect.




# $stateProvider
The $routeProvider is dead, long live the $stateProvider!

$stateProvider allows us to give names for routes. Having a name we can duplicate the route with another name assign different controller, view, well.. we can do whatever we want!
It means… with this approach we have different states of the route, thats why its called **$stateProvider**.

{% highlight JavaScript %}
$stateProvider
            .state("contact", {
                url: "/contact/",
                templateUrl: '/app/Aisel/Contact/views/contact.html',
                controller: 'ContactCtrl'
            });
{% endhighlight %}

In templates, there is no need to use direct addresses, no need to change href attribute each time we change route path. With states we just pass state name to ui-sref attribute. Its just better.
Sample on Plunker: [http://plnkr.co/edit/233aGO?p=preview](http://plnkr.co/edit/233aGO?p=preview)

> Both of this approaches and more examples you may find in [Aisel](https://github.com/ivanproskuryakov/Aisel)<br/>
> **$stateProvider** now [https://github.com/ivanproskuryakov/Aisel](https://github.com/ivanproskuryakov/Aisel)<br/>
> **$routeProvider** was used in release 0.0.1 [https://github.com/ivanproskuryakov/Aisel/releases/tag/v0.1.0](https://github.com/ivanproskuryakov/Aisel/releases/tag/v0.1.0)
