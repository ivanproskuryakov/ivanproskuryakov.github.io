---
layout: publications
title: "Code organisation in AngularJS applications"
categories: publications
modified: 2015-01-13T11:57:41-04:00
toc: false
comments: true
image:
  feature: AngularJS.jpg
---
Architecture and code design are one of the most important things in development process.
Applications without architecture with complicated functionality: users, content management,
categorisation, tagging, reviews, are almost impossible to maintain, work on issues & fix the bugs.

Explanation based on design pattern that are being used in live open-source project [https://github.com/ivanproskuryakov/Aisel](https://github.com/ivanproskuryakov/Aisel)

 {% highlight JavaScript %}
|-- 404.html
|-- app
|   |-- Aisel
|   |   |-- Product // Product module
|   |   |   |-- AiselProduct.js // Bootstrap file for dependencies
|   |   |   |-- config // Module configuration, routing and constants are stored inside config directory
|   |   |   |   |-- product.js
|   |   |   |-- controllers // Controllers directory
|   |   |   |   |-- product.js
|   |   |   |   |-- productCategory.js
|   |   |   |   |-- productCategoryDetails.js
|   |   |   |   |-- productDetails.js
|   |   |   |-- services // Services & factories location
|   |   |   |   |-- product.js
|   |   |   |   |-- productCategory.js
|   |   |   |-- views // Module specific views
|   |   |       |-- category-detail.html
|   |   |       |-- category.html
|   |   |       |-- product-detail.html
|   |   |       |-- product.html
|   |   |       |-- sidebar.html
|   |   |
|   |   |-- Contact // Contact module
|   |   |   |-- AiselContact.js
|   |   |   |-- config
|   |   |   |-- controllers
|   |   |   |-- services
|   |   |   |-- views
|   |   |
|   |   |-- Page // Page module
|   |   |   |-- AiselPage.js
|   |   |   |-- config
|   |   |   |-- controllers
|   |   |   |-- services
|   |   |   |-- views
|   |   |
|   |   |-- User // User module
|   |   |   |-- config
|   |   |   |-- controllers
|   |   |   |-- services
|   |   |   |-- views
|   |   |
|   |-- Kernel // Application specific module, we may call it "Core module"
|   |   |-- Resource
|   |       |-- KernelResource.js
|   |       |-- config // App configuration, routing and constants
|   |       |   |-- resource.js
|   |       |-- filters // Global filters
|   |       |   |-- main.js
|   |       |-- services // Global services
|   |       |   |-- init.js
|   |       |   |-- settings.js
|   |       |-- views // Global templates
|   |           |-- footer.html
|   |           |-- header.html
|   |
|   |-- app.js // Main AngularJS application file
|   |-- bootstrap.js
|   |-- environment.js // Application settings like API URL, locale, etc..
|   |-- main.js // RequireJS loader for development environment
|
|-- bower_components // Libraries installed with bower
|   |-- angular
|   |   |-- ....
|   |-- angular-animate
|   |   |-- ....
|   |-- angular-bootstrap
|   |   |-- ....
|   |-- ....
|
|-- build
|   |-- main.js // Compiled RequireJS loader for production environment
|-- favicon.ico
|-- images
|   |-- ...
|-- index.html
|-- index_dev.html
|-- media
|   |-- product
|-- robots.txt
|-- styles
    |-- main.scss
    |-- styles.css
 {% endhighlight %}

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
    // Load project dependencies
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
    // Kick start the application
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

## Final thoughts
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
  - Adam has Resource, User, Page, Product modules enabled<br/>
  - John use Resource, User, Page<br/>
  - Petr has Resource, Contact<br/>
 In this case if someone breaks the Page module, Petr still will be able to do his things with the Contact module,
 even if all team would commit updates into a Master branch.

This article remains not finished ...