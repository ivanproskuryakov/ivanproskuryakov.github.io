---
layout: publications
title: "Modals as a service in AngularJS"
categories: publications
modified: 2015-02-22T11:57:41-04:00
toc: false
comments: true
image:
  feature: AngularJS.jpg
  teaser: AngularJS.jpg
---
Modals windows could handle files uploading, user login or any other functionality that you may need in your website(application).
Modals are very handy and it common pattern in web development. They could be twice handy if we could launch modal in time and place we would need to.<br/>

In AngularJS this type of functionality could be easily implemented with services, so modal could be shared across the app.
This example deals with user authentication and modal is based on Twitter's bootstrap.

**Service:** authService.js<br/>
To open modal you will need to launch "authService.authenticateWithModal()" function.
{% highlight JavaScript %}
    angular.module('app')
        .service('authService', ['$modal',
            function ($modal) {
                return {
                    authenticateWithModal: function () {
                        var modalAuthInstance = $modal.open({
                            templateUrl: 'path/to/template/login.html',
                            controller: 'ModalAuthCtrl'
                        });
                    }
                }
            }
        ]);
    angular.module('app')
        .controller('ModalAuthCtrl', ['$scope', '$rootScope', '$state', 'userService', 'notify', 'Environment',
            function ($scope, $rootScope, $state, userService, notify, Environment) {
                var locale = Environment.currentLocale();

                $scope.passwordForgot = function () {
                    $scope.$dismiss('close');
                    $state.transitionTo('userPasswordForgot', {locale: locale});
                }

                $scope.register = function () {
                    $scope.$dismiss('close');
                    $state.transitionTo('userRegister', {locale: locale});
                }

                $scope.login = function (username, password) {
                    userService.login(username, password).success(
                        function (data, status) {
                            notify(data.message);

                            if (data.status) {
                                if (data.user.username) {
                                    $rootScope.user = data.user;
                                    $scope.$dismiss('close');
                                }
                            }
                        }
                    );
                };
            }
        ]);
{% endhighlight %}

**Template:** path/to/template/login.html
{% highlight JavaScript %}
<div class="col-xs-12 page-header">
    <h2><span class="glyphicon glyphicon glyphicon-user"></span> Sign In</h2>
</div>
<div class="modal-body">
    <div class="row">
        <!-- Form Fields -->
        <div class="col-md-12" role="main">
            <div class="col-md-12">
                <form class="form-horizontal" name="loginForm" role="form" ng-submit="login(username, password)">
                    <div class="form-group">
                        <label for="signInUsername">Username</label>

                        <div class="input-group">
                            <span class="input-group-addon"><span class="glyphicon glyphicon-envelope"></span></span>
                            <input ng-model="username" type="text" class="form-control" id="signInUsername"
                                   ng-required="true">
                        </div>
                    </div>
                    <div class="form-group">
                        <label for="signInPassword">Your password</label>

                        <div class="input-group">
                            <span class="input-group-addon"><span class="glyphicon glyphicon-asterisk"></span></span>
                            <input ng-model="password" type="password" class="form-control" id="signInPassword"
                                   ng-required="true">
                        </div>
                    </div>
                    <div class="form-group pull-right">
                        <button class="btn btn-primary" type="submit">Log In</button>
                    </div>
                </form>
                <div class="form-group pull-left">
                    <button class="btn btn-success" ng-click="passwordForgot()">Forgot password</button>
                    <button class="btn btn-success" ng-click="register()">Create Account</button>
                </div>

            </div>
        </div>
    </div>
</div>
{% endhighlight %}

**Links**:<br/>
* [https://docs.angularjs.org/guide/services](https://docs.angularjs.org/guide/services)<br/>
* [http://angular-ui.github.io/bootstrap/](http://angular-ui.github.io/bootstrap/)
