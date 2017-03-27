# Fake XMLHttpRequest
This is a wrapper around and replacement of the original XMLHttpRequest object. It is meant to be
used in frontend build environment to mock responses to AJAX calls. The urls to catch must be
explicitly defined. All calls, that are not handled by a matching handler go the normal way
using the default XMLHttpRequest.

This project was inspired by the [jquery-mockjax](https://github.com/jakerella/jquery-mockjax) project,
which provides a similar functionality for the jQuery.ajax() method.

# Install
Install this as a node module using 

```bash
npm install fake-xmlhttprequest
```

# Setup
1. Include the `lib/fake-xmlhttprequest.js` in your dummy frontend.
2. Call the setup method `FakeXMLHttpRequest.setup()` to initialize the wrapper and make it
replace the original XMLHttpRequest object.
3. Add response handlers to catch the urls you want return a mocked response.

# Response handlers
To catch a request to a certain url, you need to add a handler for it using
```javascript
FakeXMLHttpRequest.addHandler({
    // handler definition
});
```

A response handler is a simple javascript object that contains the following properties:
* url: A string to be compared with the called url or a regular expression to be matched against
 the request's url
* method [optional]: The http method to only match, e.g. GET or POST. Method is not checked is not set.
* status: The http status code the faked request should return. Can be used to simulate 404 or 50x responses.
* statusText: A matching http status text, e.g. "OK" for status 200.
* responseTime [optional]: An integer or an array with to integers. If not set, the response is return immediately.
* response: A response string or a function that returns a response. The url is given as the argument.
If `url` is a regular expression, the completeMatch is the first argument, followed by all matching groups.
Either `response` or `proxy` must be set.
* proxy: A url to load the response from or a function that returns that url. The url is given as the arguments.
If `url` is a regular expression, the completeMatch is the first argument, followed by all matching groups.
Either `response` or `proxy` must be set

## Simple match and string response
The url must be an exact match. The response is set to the `response` string.
```javascript
FakeXMLHttpRequest.addHandler({
    url: '/some/url/i-want-to-catch/',
    status: 200,
    statusText: 'OK',
    response: 'This is a text response'
});
```

## Regular expression and function response
The request's url will be matched against the given `url` expression. The whole match and the
matching group for the "q" parameter are given as arguments to the `response` function.
```javascript
FakeXMLHttpRequest.addHandler({
    url: '/\/some\/search\/url\?q=(.*)/',
    status: 200,
    statusText: 'OK',
    response: function(completeMatch, q) {
        // do some action here and return a response string or object
    }
});
```

## Regular expression and proxy function
The request's url will be matched against the given `url` expression. The whole match and the
matching group for the "q" parameter are given as arguments to the `proxy` function. The
function must return a url to load the response from.

The proxy response is loaded via another XMLHttpRequest. Make sure the returned proxy url is
not matched by any of your handlers or you could end up in a loop.
```javascript
FakeXMLHttpRequest.addHandler({
    url: '/\/some\/search\/url\?q=(.*)/',
    status: 200,
    statusText: 'OK',
    proxy: function(completeMatch, q) {
        // return the url to the file to fetch.
    }
});
```

## Simulating a response time/slower response
Simulate a server response time that is not 0, e.g. to show and/or test waiting/loading animation.
The response time will be randomly choosen between 500 and 3000 ms.
```javascript
FakeXMLHttpRequest.addHandler({
    url: '/\/some\/search\/url\?q=(.*)/',
    responseTime: [500, 3000],
    status: 200,
    statusText: 'OK',
    proxy: function(completeMatch, q) {
        // return the url to the file to fetch.
    }
});
```
