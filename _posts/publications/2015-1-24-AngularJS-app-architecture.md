---
layout: publications
title: "Code organisation in AngularJS applications"
categories: publications
modified: 2015-01-24T11:57:41-04:00
toc: false
comments: true
image:
  feature: AngularJS.jpg
  teaser: AngularJS.jpg
---
Architecture and code design are two of the most important things of development process.
An application with complicated functionality, such as users, content management,
categorisation, tagging, reviews, etc.. and no design pattern is extremely hard to maintain
as well as to work on issues & to fix bugs.

This publication is based on design pattern that is being used in [Aisel](https://github.com/ivanproskuryakov/Aisel), the project I'm working on. [https://github.com/ivanproskuryakov/Aisel](https://github.com/ivanproskuryakov/Aisel)

 {% highlight JavaScript %}
|-- 404.html
|-- app
|   |-- Aisel
|   |   |-- Product // Product module
|   |   |   |-- AiselProduct.js // Module bootstrap file
|   |   |   |-- config // Module configuration, routing and constants are stored inside config directory
|   |   |   |   |-- product.js
|   |   |   |
|   |   |   |-- controllers // Controllers directory
|   |   |   |   |-- product.js
|   |   |   |   |-- ...
|   |   |   |
|   |   |   |-- services // Services & factories location
|   |   |   |   |-- product.js
|   |   |   |   |-- ...
|   |   |   |
|   |   |   |-- views // Module specific views
|   |   |   |   |-- category.html
|   |   |   |   |-- ...
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
|   |-- environment.js // Application settings like API URL, locale etc..
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

**A. App environments** <br/>
Development and production environments are separated by different directory index files.
Virtual host points to index_dev.html file on the local machine and to the index.html on production.

This design pattern based on [RequireJS](http://requirejs.org/) file and module loader.
During development requireJS points to **/app/main.js**
 {% highlight html %}
 <script data-main="/app/main" src="/bower_components/requirejs/require.js"></script>
 {% endhighlight %}
while on production environment it is bound by **/build/main**
 {% highlight html %}
 <script data-main="/build/main" src="/bower_components/requirejs/require.js"></script>
 {% endhighlight %}

All of application JavaScript files, dependencies and vendors are defined in **main.js**
with ADM(Asynchronous Module Definition) pattern.

> ... Advantages of the AMD pattern are described on [requirejs.org](requirejs.org) -> [Why AMD?](http://requirejs.org/docs/whyamd.html) page.

**B. RequireJS configuration**<br/>
Main.js file is the main configuration file with paths to the vendors and dependencies that are used in the app.
The most important part of is the **deps** section. This section gives a better flexibility and allows to have fully demountable modules.
In this case we may have different modules enabled during the development process or deploy angular app with different modules on board.<br/>
 - Website app consists of AiselSearcha and AiselPage modules<br/>
 - Blog app consists of AiselSearch, AiselPage and AiselUser modules<br/>
 - E-commerce could have: AiselSearch, AiselHomepage, AiselProduct, AiselUser, AiselCart etc..<br/>
 {% highlight JavaScript %}
require.config({
    // Load project dependencies
    paths: {
        'angular': '../bower_components/angular/angular',    // Javascript library name
        'jQuery': '../bower_components/jquery/jquery.min',   // .js extension is not used
        'domReady': '../bower_components/domReady/domReady',
        'twitter-bootstrap': '../bower_components/sass-bootstrap/dist/js/bootstrap',
        'angular-resource': '../bower_components/angular-resource/angular-resource',
        'angular-route': '../bower_components/angular-route/angular-route',
        'ui-bootstrap-tpls': '../bower_components/angular-bootstrap/ui-bootstrap-tpls',
        'angular-ui-router': '../bower_components/angular-ui-router/release/angular-ui-router',
        'angular-notify': '../bower_components/angular-notify/dist/angular-notify.min',
        "... other bower dependencies"
    },
    // Add angular modules that do not support AMD out of the box, put them in a shim
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
        './Kernel/Resource/KernelResource', // Kernel module
        './Aisel/Homepage/AiselHomepage', // Homepage module
        './Aisel/Contact/AiselContact', // Contact module
        './Aisel/Search/AiselSearch', // Search module
        './Aisel/Page/AiselPage', // Page module
        'bootstrap', // manually start up angular application
        "... other modules and definitions"
    ],
    priority: [
        "angular"
    ]
});
 {% endhighlight %}

In **deps** section requireJS loads modules and manually starts angular application with boostrap.js
{% highlight JavaScript %}
define([
    'require',
    'angular',
    'app',
], function (require, angular) {
    'use strict';
    require(['domReady!'], function (document) {
        angular.bootstrap(document, ['app']); // file that holds the root module of our application. (app.js)
    });
});
{% endhighlight %}

**C. AngularJS root**<br/>
With RequireJS module loader, app.js and other files used in application become modules,
in this case they must be wrapped with **"define([]"** structure.
Code of app.js loader has to be also wrapped with **define** as shown below:
{% highlight JavaScript %}
 define([
         'angular', 'jQuery', 'underscore', 'angular-resource',
         'angular-cookies', 'angular-sanitize', 'textAngular',
         'ui-bootstrap-tpls', 'ui-utils', 'angular-gravatar',
         'md5', 'angular-disqus', 'angular-notify', 'twitter-bootstrap',
         'angular-ui-router', 'angular-route', 'angular-animate',
         'angular-loading-bar'], // plug modules defined in require.js
     function (angular) {
         'use strict';

         var app = angular.module('app', [
             'ngCookies', 'ngResource', 'ngSanitize', 'ngRoute', 'ui.bootstrap', 'ui.router',
             'ui.utils', 'ui.validate', 'ui.gravatar', 'textAngular', 'ngDisqus', 'cgNotify',
             'ngAnimate', 'angular-loading-bar',
             'environment' // plug AngularJS modules
         ])

         app.value('appSettings', [])
         .run(['$http', '$rootScope', 'settingsService', 'initService',
             function ($http, $rootScope, settingsService, initService) {
                 initService.launch();

                 // run section ...
             }])
         .config(function ($provide, $locationProvider, $httpProvider) {
             $locationProvider.html5Mode(true);
             document.getElementById("page-is-loading").style.visibility = "hidden";

             // config section ...
         });
         return app;
     });
{% endhighlight %}
> ... More about module loader at [http://requirejs.org/docs/api.html#funcmodule](http://requirejs.org/docs/api.html#funcmodule)

**D. Module structure**<br/>
Each logical unit of functionality must be implemented as a single module.
Full module name should consist of namespace and module name ex: AiselProduct.
Basic module structure displayed bellow:
{% highlight JavaScript %}
|-- app
|   |-- Aisel
|   |   |-- Product // Product module
|   |   |   |-- AiselProduct.js // Module bootstrap file
|   |   |   |-- config // Module configuration, routing and constants are stored inside config directory
|   |   |   |   |-- product.js
|   |   |   |-- controllers // Controllers directory
|   |   |   |   |-- product.js
|   |   |   |   |-- productCategory.js
|   |   |   |   |-- productCategoryDetails.js
|   |   |   |   |-- productDetails.js
|   |   |   |-- services // Services & factories location
|   |   |   |   |-- product.js
|   |   |   |-- views // Module specific views
|   |   |       |-- category-detail.html
|   |   |       |-- category.html
|   |   |       |-- product-detail.html
|   |   |       |-- product.html
|   |   |       |-- sidebar.html
{% endhighlight %}
**Define** section of AiselProduct.js file loads module dependencies and bootstraps Product module
{% highlight JavaScript %}
define(['app',
    './config/product',
    './controllers/product',
    './controllers/productDetails',
    './controllers/productCategory',
    './controllers/productCategoryDetails',
    './services/product',
    './services/productCategory',
], function (app) {
    console.log('Product module loaded ...');
});
{% endhighlight %}

<br/>


## Final thoughts
An app with no organisation consumes more time, and a chance that you forgot something and will
have an issue later on, dramatically increases. Whether we want it or not, we will start thinking
about the code re-organisation.

Otherwise, an app designed with modular architecture gives us a possibility to decompose functionality,
refactor stand alone units any time and group them into modules. It gives us a possibility to delegate different
development stages to different teams, e.g. team A will work on User functionality, team B will be
responsible for content management etc..<br/>

In short an application should have the following principles:<br/>

 **A. Code simplicity**<br/>
 This will increase productivity and code understanding for newcomers. Even if the project has no documentation,
 it will be easy to understand design pattern and start contribution.

 **A. Logical Units** <br/>
 It means we need to decompose the code into small logical units,
 if something goes wrong in future we will need to work only with a single unit.
 Defined strict application structure will also increase understanding for all contributors or team members.
 It means less questions/time, better code quality.

 **C. Independent functionality** <br/>
 Possibility to enable/disable a different part of the functionality.<br/>
 Example:<br/>
  - Adam has Resource, User, Page and Product modules enabled<br/>
  - John uses Resource, User and Page<br/>
  - Peter has Resource and Contact<br/>
 In this case if Adam breaks the Page module, Peter will still be able to do his tasks with the Contact module,
 even if the whole team commits updates into the Master branch.