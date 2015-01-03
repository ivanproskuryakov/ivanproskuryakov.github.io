---
layout: publications
title: "AngularJS $routeProvider vs $stateProvider"
categories: publications
modified: 2014-08-27T11:57:41-04:00
toc: false
comments: true
---

## $routeProvider

AngularJS very young framework, but some time ago there was only $routeProvider solution for a routing for AngularJS app.
$routeProvider is linking specific URL to controller and assigning a template. Pretty simple.

{% highlight JavaScript %}
$routeProvider
        .when('/contact/', {
            templateUrl: 'app/views/core/contact/contact.html',
            controller: 'ContactCtrl'
        })
});
{% endhighlight %}

After that generated template rendered with **<div ng-view></div>** directive
Inside the templates, links could be done in two ways with typical href attribute or with ng-href angular attribute, which is basically the same.
Code sample: [http://plnkr.co/edit/uwYSb9](http://plnkr.co/edit/uwYSb9)

According documentation **$routeProvider** is a routing standard for now, this approach is also used in [official tutorial](https://docs.angularjs.org/tutorial).
But… Like with all the things, $routeProvider was not perfect.

## $stateProvider
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
Code sample on plunker: [http://plnkr.co/edit/233aGO?p=preview](http://plnkr.co/edit/233aGO?p=preview)

Both of this approaches and more coding examples you will find in Aisel project that I’m working.
$stateProvider used current branch [https://github.com/ivanproskuryakov/Aisel](https://github.com/ivanproskuryakov/Aisel)
$routeProvider was used in release 0.0.1 [https://github.com/ivanproskuryakov/Aisel/releases/tag/v0.1.0](https://github.com/ivanproskuryakov/Aisel/releases/tag/v0.1.0)
