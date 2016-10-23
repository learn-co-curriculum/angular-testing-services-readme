# Testing Services

## Overview

Now that we've created our own services, we need to be able to test them too!

## Objectives

- Describe Service testing
- Write a unit test for our custom Service

## Testing Services

Our services are grabbing and manipulating data all over the place, and as we may be using them all over our application, it is important that we test our services to ensure that when we change them, we don't break our application.

## Grabbing our services

Previously, we've been using `$controller` to get our controllers, so logically we should be using `$service` to grab our services. Unfortunately, it's not that simple. We use `$controller` to both grab and instantiate our controllers. We don't need to instantiate our services, so we use a nice service named `$injector`!

Let's take a look at how we'd grab our service:

```js
describe('OurService', function () {
    beforeEach(module('app'));

    var OurService;

    beforeEach(inject(function ($injector) {
        OurService = $injector.get('OurService');
    }));


    it('should test our service', function () {
        // We can use OurService here
    });

});
```

It's a little bit different from before, but not an issue! Still really simple.

We can now access all of our public methods our service gives us, and then test the results.

For instance, for our `MathService` that we used earlier on in the series, we can test it as follows -

```js
describe('MathService', function () {
    beforeEach(module('app'));

    var MathService;

    beforeEach(inject(function ($injector) {
        MathService = $injector.get('MathService');
    }));


    it('should add up correctly', function () {
        expect(MathService.sum([1,23])).toEqual(24);
    });
});
```

Yes, it's that simple!

## Testing $http calls

Now, our services are also going to be calling `$http` a lot, and it would be a lot of effort to mock an entire backend. Luckily, ngMocks provides us with a nice little tool called `$httpBackend`. This allows us to mock our API calls.

There are three parts to `$httpBackend`:

### $httpBackend.when

We use `$httpBackend` to setup the responses to our HTTP calls. It accepts a method and a URL, and allows us to define the response that we'd get back.

If we have a backend endpoint at `/rest/user` that responds with the current user's information, we can mock it as follows:

```js
describe('UserService', function () {
    beforeEach(module('app'));

    var UserService, $httpBackend;

    beforeEach(inject(function ($injector) {
        UserService = $injector.get('UserService');
        $httpBackend = $injector.get('$httpBackend');

        $httpBackend.when('GET', '/rest/user').respond({user: 'Bill Gates', email: 'bill@microsoft.com'});
    }));
});
```

### $httpBackend.expectGET

Now, when we actually test the service's function that calls that endpoint, we need to tell ngMocks that we're expecting the request to occur. We do this by calling `$httpBackend.expectGET` with the endpoint we're expecting to have a request to.

```js
describe('UserService', function () {
    beforeEach(module('app'));

    var UserService, $httpBackend;

    beforeEach(inject(function ($injector) {
        UserService = $injector.get('UserService');
        $httpBackend = $injector.get('$httpBackend');

        $httpBackend.when('GET', '/rest/user').respond({user: 'Bill Gates', email: 'bill@microsoft.com'});
    }));

    it('should get the current users information', function (done) {
        $httpBackend.expectGET('/rest/user');
    });
});
```

We're now setup to receive the mocked backend response. Our response will have several properties, with the `data` property referring to the body of our response. In this case, It'll be an object with `name` equal to `Bill Gates` and his email too.

```js
describe('UserService', function () {
    beforeEach(module('app'));

    var UserService, $httpBackend;

    beforeEach(inject(function ($injector) {
        UserService = $injector.get('UserService');
        $httpBackend = $injector.get('$httpBackend');

        $httpBackend.when('GET', '/rest/user').respond({user: 'Bill Gates', email: 'bill@microsoft.com'});
    }));

    it('should get the current users information', function (done) {
        $httpBackend.expectGET('/rest/user');

        UserService
          .getUserInfo()
          .then(function (res) {
            var data = res.data;
            if (data.email === 'bill@microsoft.com' && data.user === 'Bill Gates') {
              done();
            }
          });
	});
});
```

Here, instead of using `expect().toBe()`, we call a function named `done()` if our response is correct. This means that we can do asynchronous tests in Jasmine. An important item to note here is that we must pass the `done` function in as a argument to this test for this feature to work. 

### $httpBackend.flush()

We then need to call `$httpBackend.flush()` to immediately execute all pending requests (which then fires off our request, calling our callback with the returned data and runs the test).

```js
describe('UserService', function () {
    beforeEach(module('app'));

    var UserService, $httpBackend;

    beforeEach(inject(function ($injector) {
        UserService = $injector.get('UserService');
        $httpBackend = $injector.get('$httpBackend');

        $httpBackend.when('GET', '/rest/user').respond({user: 'Bill Gates', email: 'bill@microsoft.com'});
    }));

    it('should get the current users information', function (done) {
        $httpBackend.expectGET('/rest/user');

        UserService
          .getUserInfo()
          .then(function (res) {
            if (res.email === 'bill@microsoft.com' && res.user === 'Bill Gates') {
              done();
            }
          });

        $httpBackend.flush();
	});
});
```

All done! We've now got a fully tested `UserService`, including mocked HTTP requests.

<p class='util--hide'>View <a href='https://learn.co/lessons/angular-testing-services-readme'>Testing Services</a> on Learn.co and start learning to code for free.</p>
