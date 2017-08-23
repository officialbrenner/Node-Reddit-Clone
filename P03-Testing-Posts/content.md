---
title: "Testing Posts"
slug: testing-posts
---

So now we can create and show posts, but we are never done with code until we've written automated tests for it. So we'll have to setup our test environment and make a few controller- or route-level tests.

# Installing Mocha and Chai & Hello World Test

Mocha.js is a test framework for Node.js, and Chai.js is a helper library that makes tests run and look cleaner. `chai-http` makes it easy for us to make test http requests to our server. Let's add them to our project and run our first test.

We're going to add these to our `devDependencies` in our `package.json` file because we don't need the testing libraries in production.

```bash
$ npm install mocha chai chai-http --save-dev
```

Now create a folder called `test` in the root of your project.

Add a file called `index.js` and let's require our testing libraries and then create our first hello world style test.

```js
var chai = require('chai')
var chaiHttp = require('chai-http');
var should = chai.should();

chai.use(chaiHttp);

describe('Site', function() {
  it('should have a live home page', function (done) {
    chai.request('localhost:3000')
      .get('/')
      .end(function (err, res){
        res.status.should.be.equal(200);
        done();
      });
  });
});
```

This test tests that the reponses status should be equal to 200 - which if you recall your HTTP status codes, means the response is successful.

Now let's run the test. First update your `package.js` file to have a test command:

```json
"scripts": {
  "test": "mocha --timeout 10000"
},
```

We'll use the `--timeout 10000` option in case requests to our database are slower than the 2 second default mocha timeout.

Now we can run our tests one of two ways:
```bash
$ mocha
$ npm test
```

What was the result? Can you make the test fail?

# Testing Posts#Create

Next let's make a test for the posts#create route we made. We can make a new file in `test` called `posts.js`.

```js
var chai = require('chai')
var chaiHttp = require('chai-http');
var should = chai.should();

chai.use(chaiHttp);

describe('Posts', function() {
  it('should create with valid attributes at POST /posts', function (done) {
    // test code
  });
});
```

The order of pseudocode we want to see is as follows

```js
// How many tours are there now?
// Make a request to create another
// Check that the database has one more tour in it
// Check that the response is a successful
```

So if we write that in:

```js
// How many tours are there now?
Tour.find(function(err, tours) {
  var tourCount = tours.count;

  var tour = { title: "post title", url: "https://www.google.com", summary: "post summary" }
  chai.request('localhost:3000')
    .post('/tours', tour)
    .end(function (err, res){

      // Check that the database has one more tour in it
      Tour.find(function(err, tours) {
        tourCount.should.be.equal(tours + 1);

        // Check that the response is a successful
        res.should.have.status(200);
      done();
    });
  });
});
```

This is a good test, except remember that each time we run our test suite we will be creating this post. We need to make sure we delete this post before we run the test. So let's wrap that in a mongoose model `.remove()` method.


```js
var tour = { title: "post title", url: "https://www.google.com", summary: "post summary" }

Tour.findOneAndRemove(tour, function() {
  Tour.find(function(err, tours) {
    var tourCount = tours.count;
    chai.request('localhost:3000')
      .post('/tours', tour)
      .end(function (err, res){
        Tour.find(function(err, tours) {
          tourCount.should.be.equal(tours + 1);
          res.should.have.status(200);
        done();
      });
    });
  });
});
```

Now we have a test for the posts#create route that should be green. Can you make it fail? How about if our `post` object doesn't have a title, url, or summary? Those are all required fields. What do you see if you change that and run the test? Does it fail? How do you know what made it fail?

When a controller or route test runs, it runs itself and it hits your server endpoint code locally. That means you can put `console.log` or `debugger` statements in either one to check various values.