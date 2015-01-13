---
layout: publications
title: "Architecture in AngularJS apps"
categories: publications
modified: 2014-08-27T11:57:41-04:00
toc: false
comments: true
---

Architecture and code design are the most important parts of development process.
Applications without architecture with complicated functionality: users, content management,
categorisation, tagging, reviews, are almost impossible to maintain, work on issues & fix the bugs.

#### Ideology and theory, why...
An app with no organisation consumes more time, and a chance that you forgot something and will
have an issue later on, dramatically increases. Whether we want it or not, we will start thinking
about the code re-organisation.

Otherwise, an app designed with modular architecture give us a possibility to decompose functionality,
or semantically group the same parts into modules. This gives a possibility to delegate different
development stages to different teams, e.g. team A will work on User Functionality, team B will be
responsible for content management etc..

In short application should have following principles:<br/>
 **A.** Code must be flexible.<br/>
 It means we need to decompose code into small logical parts,
 if something went wrong in future it will be easy to refactor the code

 **B.** Increase has to be simple<br/>
 This will increase productivity, code understanding for newcomers. Even if the project has documentation
 it will be easy to understand design pattern and start contribution.

 **C.** Detachable functionality<br/>
 Possibility to enable/disable a different part of the functionality.<br/>
 Like example bellow:<br/>
  - Adam has Core, User, Page, Product modules enabled<br/>
  - John use Core, User, Page<br/>
  - Petr has Core, Contact<br/>
 In this case if someone breaks the Page module, Petr still will be able to do his things with the Contact module,
 even if all team would commit updates into a Master branch.

These are the most important things to focus on..

#### Live example
Explanation based on design pattern that are being used in live open-source project [https://github.com/ivanproskuryakov/Aisel](https://github.com/ivanproskuryakov/Aisel)


**A**. Development and production environments restricted by having different directory indexes.
On local machine our virtual host points to index_dev.html, on production - index.html.


**B**. [RequireJS](http://requirejs.org/) for dynamic(lazy) javascript module loading.
For development app uses app/main.js as main [RequireJS](http://requirejs.org/) file:
 {% highlight html %}
 <script data-main="/app/main" src="/bower_components/requirejs/require.js"></script>
 {% endhighlight %}
and minified build/main.js for running on production environments:
 {% highlight html %}
 <script data-main="/build/main" src="/bower_components/requirejs/require.js"></script>
 {% endhighlight %}

All of application JavaScript files, dependencies and vendors are defined in **main.js**
using [RequireJS](http://requirejs.org/) library and ADM(Asynchronous Module Definition) pattern.

> Why AMD pattern is so nice, described on [requirejs.org](requirejs.org) -> [Why AMD?](http://requirejs.org/docs/whyamd.html) page.

**C.**
Main.js file is the main require.js and application configuration file, in it we define vendors dependencies and modules of AngularJS application

 {% highlight JavaScript %}
require.config({
    // Load paths from global variable
    paths: {
        'jQuery': '../bower_components/jquery/jquery.min',
        'domReady': '../bower_components/domReady/domReady',
        'angular': '../bower_components/angular/angular',
        'twitter-bootstrap': '../bower_components/sass-bootstrap/dist/js/bootstrap',
        'angular-resource': '../bower_components/angular-resource/angular-resource',
        'angular-route': '../bower_components/angular-route/angular-route',
        'ui-bootstrap-tpls': '../bower_components/angular-bootstrap/ui-bootstrap-tpls',
        'angular-ui-router': '../bower_components/angular-ui-router/release/angular-ui-router',
        'angular-notify': '../bower_components/angular-notify/dist/angular-notify.min',
        "... other bower dependencies"
    },
    // Add angular modules that does not support AMD out of the box, put it in a shim
    shim: {
        'angular-route': ['angular'],
        'angular-ui-router': ['angular'],
        'angular' : {'exports' : 'angular', deps: ['jQuery']},
        'jQuery': {'exports' : 'jQuery'},
        "domReady": ["angular"],
        "angular-resource": ["angular"],
        "angular-cookies": ["angular"],
        "ui-bootstrap-tpls": ["angular"],
        "twitter-bootstrap": ["angular"],
        "... other JS definitions"
    },
    // Kick start application
    deps: [
        './environment',
        './Kernel/Resource/KernelResource',
        './Aisel/Homepage/AiselHomepage',
        './Aisel/Contact/AiselContact',
        './Aisel/Search/AiselSearch',
        './Aisel/Page/AiselPage',
        'bootstrap',
        "... other modules and definitions"
    ],
    priority: [
        "angular"
    ]
});
 {% endhighlight %}

This article remains not finished ...