---
layout: publications
title: "Passing errors from Symfony to AngularJS"
categories: publications
modified: 2015-06-16T11:57:41-04:00
toc: false
comments: true
image:
  feature: AngularJS.jpg
  teaser: AngularJS.jpg
---
All frameworks have error handling: Symfony, Rails, Spring and approach always the same: display all exceptions in development environment, and required minimum when app in production.

So we need to pass all errors via API from backend to frontend, for these we will need:<br/>
 * Add exception listener on global level for backend(Symfony2), and throw unified error Response with API.<br/>
 * Add HTTP error interceptor(AngularJS), it will consume response error data directly from API and display it on separate templates.<br/>

**Implementation:**<br/>
In AngularJS add new route with name "exception" and new interceptor which will redirect to this "exception" route, if an HTTP error comes up.
{% highlight javascript %}
'use strict';

/**
 * This file is part of the Aisel package.
 *
 * (c) Ivan Proskuryakov
 *
 * For the full copyright and license information, please view the LICENSE
 * file that was distributed with this source code.
 *
 * @name            AiselException
 * @description     Module configuration
 */

define(['app'], function (app) {
    app.config(['$stateProvider', function ($stateProvider) {
        $stateProvider
            .state("exception", {
                url: "/:locale/exception/:code",
                templateUrl: '/app/Aisel/Exception/views/exception.html',
                controller: 'ExceptionCtrl'
            });
    }]);

    app.config(function ($httpProvider) {

        var exceptionInterceptor = [
            '$q', '$injector', 'Environment','$rootScope',
            function ($q, $injector, Environment, $rootScope) {

                function success(response) {
                    return response;
                }

                function error(response) {
                    console.log(response);
                    $rootScope.exception = response;
                    var locale = Environment.currentLocale();

                    $injector.get('$state').transitionTo(
                        'exception',
                        {
                            locale: locale,
                            code: response.data.error.code
                        }
                    );

                    return $q.reject(response);
                }

                return function (promise) {
                    return promise.then(success, error);
                }
            }];

        $httpProvider.responseInterceptors.push(exceptionInterceptor);
    });
});
});

{% endhighlight %}

Controller for "exception" route
{% highlight javascript %}
'use strict';

/**
 * This file is part of the Aisel package.
 *
 * (c) Ivan Proskuryakov
 *
 * For the full copyright and license information, please view the LICENSE
 * file that was distributed with this source code.
 *
 * @name            AiselException
 * @description     ...
 */

define(['app'], function (app) {
    app.controller('ExceptionCtrl', ['$state', '$scope', 'notify', 'Environment', '$rootScope',
        function ($state, $scope, notify, Environment, $rootScope) {

            $scope.exception = undefined;

            if ($rootScope.exception) {
                $scope.exception = $rootScope.exception;
                $rootScope.exception = undefined;

                var errorTrace = $scope.exception.data.error;
                var message = errorTrace.exception[0].message;

                if (!angular.isDefined(message)) {
                    message = errorTrace.message;
                }
                notify(message);

            } else {
                var locale = Environment.currentLocale();
                $state.transitionTo('homepage', {locale: locale});
            }
        }
    ]);
});
{% endhighlight %}

and template:
{% highlight html %}
<div class="col-md-12 page-header">
    <span class="pull-left">
        <h2>Oops, something went wrong... </h2>
    </span>
</div>
<div class="col-md-12" ng-if="exception">
    <strong>Trace</strong>
    <pre>
        <small>
            exception.data.error.exception
        </small>
    </pre>
    <strong>Config</strong>
    <pre>
        <small>
            exception.config | json
        </small>
    </pre>
</div>
{% endhighlight %}

For backend, which in my case is Symfony2, add new service as shown bellow:
{% highlight javascript %}
    aisel_exception_listener:
        class: %aisel_exception_listener.class%
        arguments:
        tags:
            - { name: "kernel.event_listener", event: "kernel.exception", method: "onKernelException" }
{% endhighlight %}

And its implementation:
{% highlight javascript %}
    <?php

    /*
     * This file is part of the Aisel package.
     *
     * (c) Ivan Proskuryakov
     *
     * For the full copyright and license information, please view the LICENSE
     * file that was distributed with this source code.
     */

    namespace Aisel\ResourceBundle\Request;

    use Symfony\Component\HttpFoundation\Response;
    use Symfony\Component\HttpFoundation\JsonResponse;
    use Symfony\Component\HttpKernel\Event\GetResponseForExceptionEvent;
    use Symfony\Component\HttpKernel\Exception\HttpExceptionInterface;
    use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;
    use Exception;
    use Aisel\ResourceBundle\Exception\ValidationFailedException;
    use Symfony\Component\Validator\ConstraintViolation;
    /**
     * Class ExceptionListener.
     *
     * @author Ivan Proskuryakov <volgodark@gmail.com>
     */
    class ExceptionListener
    {

        /**
         * @param GetResponseForExceptionEvent $event
         *
         * @return string
         */
        private function getRequestType(GetResponseForExceptionEvent $event)
        {
            $contentType = $event
                ->getRequest()
                ->headers
                ->get('Content-Type');

            if (preg_match('/application\/json/', $contentType)) {
                return 'api';
            }

            return 'web';
        }

        /**
         * Set response vars and generate response.
         *
         * @param string $code
         * @param string $message
         * @param array  $headers
         *
         * @return JsonResponse $response
         */
        private function createResponse($code, $message, $headers = [])
        {
            return new JsonResponse(
                [
                    'code' => $code,
                    'message' => $message,
                ],
                $code,
                $headers
            );
        }

        /**
         * @param GetResponseForExceptionEvent $event
         *
         * @return JsonResponse $response
         */
        private function exceptionEventProcessor(GetResponseForExceptionEvent $event)
        {
            $exception = $event->getException();
            $response = null;

            switch (true) {
                case $exception instanceof HttpExceptionInterface:
                    $response = $this->responseHttpException($exception);
                    break;

                case $exception instanceof Exception:
                    $response = $this->responseException($exception);
                    break;

                // others ...
            }

            return $response;
        }

        /**
         * Response for NotFoundHttpException (Route was not found).
         *
         * @param NotFoundHttpException $exception
         *
         * @return JsonResponse
         */
        private function responseHttpException(HttpExceptionInterface $exception)
        {
            return $this->createResponse(
                $exception->getStatusCode(),
                $exception->getMessage(),
                $exception->getHeaders()
            );
        }

        /**
         * Response for Exception.
         *
         * @param Exception $exception
         *
         * @return JsonResponse
         */
        private function responseException(Exception $exception)
        {
            if (method_exists($exception, 'getStatusCode')) {
                $code = $exception->getStatusCode();
            } else {
                $code = $exception->getCode();
            }

            // Any type of exception that we are not expecting gives a 500
            if (!array_key_exists($code, Response::$statusTexts)) {
                $code = 500;
            }

            /*
             * Status codes < 100 and > 600 comes from exceptions not related with http exceptions.
             * You should not mask these codes. You should display $exception->getMessage() and debug
             * exception.
             */

            return $this->createResponse(
                $code,
                $exception->getMessage()
            );
        }

    }

{% endhighlight %}

**P.S.**
When we work with RESTful application or a website development process is more complicated. With client-server approach backend team provide API endpoints the frontend, which consumes the results.<br/>
If HTTP error comes up, developers usually open console tab in favourite browser or use something else trying reproduce the request and get error message.

Why do we need to do this redundant work just to get an error message?<br/>
Why not to show custom templates for 404 or 500 HTTP errors instead of showing blank or broken window?<br/>
I guess that we always forgot about error templates and error handling in RESTful apps. <br/>


