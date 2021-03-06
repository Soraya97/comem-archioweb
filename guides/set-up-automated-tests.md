# Set up automated testing for an Express.js REST API

This guide explains how to set up [automated tests][automated-testing] for an
HTTP API implemented with [Express.js][express] and [Mongoose][mongoose], using
the following tools:

* [Mocha][mocha] (test framework)
* [Chai][chai] (assertion library)
* [SuperTest] (HTTP test library)

> This is just a particular selection of popular tools. There are many other
> tools that can do the same. For example, read [this
> article][top-js-test-frameworks-2019] for a list of the most popular
> JavaScript test frameworks in 2019.
>
> Note that the code in this tutorial uses [Promises][promise] and [async
> functions][async-await]. Read [this guide](./async-js.md) if you are not
> familiar with the subject.

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [Requirements](#requirements)
- [Install the test framework and run your first test](#install-the-test-framework-and-run-your-first-test)
- [Your domain model & API](#your-domain-model--api)
- [Set up your test suite](#set-up-your-test-suite)
  - [Switch databases when running tests](#switch-databases-when-running-tests)
- [Write your first test](#write-your-first-test)
  - [Disconnect from the database once the tests are done](#disconnect-from-the-database-once-the-tests-are-done)
  - [Fix Mongoose `collection.ensureIndex is deprecated` warning](#fix-mongoose-collectionensureindex-is-deprecated-warning)
  - [Get rid of request logs while testing](#get-rid-of-request-logs-while-testing)
- [Add a unicity constraint to your model](#add-a-unicity-constraint-to-your-model)
- [Reproducible tests](#reproducible-tests)
- [Write more detailed assertions](#write-more-detailed-assertions)
- [Write a second test](#write-a-second-test)
- [Optional: check your test coverage](#optional-check-your-test-coverage)
- [Tips](#tips)
  - [Chai](#chai)
- [Documentation](#documentation)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->



## Requirements

* [Node.js][node] 12.x
* You are using [npm][npm] to manage your dependencies.
* A working [Express.js][express] application providing the HTTP API you wish to
  test.
* You are using [Mongoose][mongoose] to store your data in a [MongoDB][mongo]
  database.



## Install the test framework and run your first test

Start by installing Mocha, which will run your tests, and Chai, which you will
use to make assertions:

```bash
$> npm install --save-dev mocha chai
```

Create a `spec` folder in your project.

> `spec` is short for **specification**. When you write tests, each test will be
> identified by a short string which describes what the test is testing. For
> example, a test could be called `POST /users should create a user`. This
> **specifies** what the test is doing (`POST /users`) and what the expected
> result is `should create a user` in a brief sentence.
>
> This practice is part of [Behavior-Driven Development][bdd].

You will now create a sample test to make sure that both Mocha and Chai work.
Create a `spec/example.spec.js` file with the following contents:

```js
const { expect } = require('chai');

// Define a test with Mocha.
it('should work', function() {
  // Make assertions with Chai.
  expect(true).to.equal(true);
});
```

To run these tests, you can use the `mocha` command which comes with the npm
package. The most convenient way to do it is to define a new `test` script in
the `scripts` section of your `package.json`:

```json
"scripts": {
  "...": "<PREVIOUS SCRIPTS HERE...>",
  "test": "mocha spec/**/*.spec.js"
}
```

> Running `mocha spec/**/*.spec.js` instructs Mocha to run all the files
> matching the pattern `spec/**/*.spec.js`, which means any file in the `spec`
> directory (recursively) with the extension `.spec.js`. The file you just
> created, `spec/example.spec.js`, matches this pattern.

You can now simply run `npm test` to execute your test(s):

```bash
$> npm test

> my-app@0.0.0 test /path/to/my-app
> mocha spec/**/*.spec.js

  ✓ should work
```



## Your domain model & API

The rest of this tutorial assumes that you have the following Mongoose schema:

```js
const userSchema = new Schema({
  name: String,
  password: String
});
```

And the following routes:

* `POST /users` creates a user.
* `GET /users` lists users by ascending name.

If that is not the case, adapt the tests to your domain model and API.



## Set up your test suite

You can delete the `spec/example.spec.js` file you created earlier. Create a
`spec/users.spec.js` file instead, since we want to test the 2 user-related
routes:

```js
const { expect } = require('chai');

describe('POST /users', function() {
  it('should create a user');
});

describe('GET /users', function() {
  it('should retrieve the list of users');
});
```

> Note the nested `describe/it` structure provided by Mocha. It helps you
> structure your tests when writing them. Everything that is in the
> `describe('POST /users')` block describes how that route should work. The
> `it('should create a user')` block insides describes one thing that route
> should do. Combining the two gives you the whole test specification: `POST
> /users should create a user`. You can use multiple `it` blocks inside a
> `describe` block if you want.

Mocha allows you to write `it` calls without a test function: this creates a
**pending test**, a test that is not yet implemented. If you run your tests with
`npm test` now, you should see the tests marked as pending:

```bash
$> npm test
> my-app@0.0.0 test /path/to/my-app
> mocha spec/**/*.spec.js

  POST /users
    - should create a user

  GET /users
    - should retrieve the list of users

  0 passing (3ms)
  2 pending
```

> You could use this functionality to prepare your test suite before writing the
> tests themselves. If you write your tests first using [Test-Driven
> Development][tdd], you could even write your whole test suite before
> implementing your API!

### Switch databases when running tests

When working on an existing application, you probably have a development
database containing some data. You don't want your tests to mess with that data.
It's good practice to **switch to a different database for testing**.

Presumably, your MongoDB connection URL is configurable through an environment
variable. For example, your `app.js` file may contain a line that looks like
this:

```js
mongoose.connect(process.env.MONGODB_URI || 'mongodb://localhost/my-app', {
  // Options...
});
```

This means that you can easily switch the database URL by setting the
`$MONGODB_URI` environment variable. This is useful not only when deploying in a
production environment, but also for the test environment.

To avoid setting this variable by hand every time, you can use the [`cross-env`
package][cross-env] which sets environment variables in a
cross-platform-compatible way. Install it:

```bash
$> npm install --save-dev cross-env
```

You can now **update** the `test` script in your `package.json` file to look
like this:

```json
"scripts": {
  "test": "cross-env MONGODB_URI=mongodb://127.0.0.1/my-app-test mocha spec/**/*.spec.js"
}
```

This switches the `$MONGODB_URI` variable to another value before running your
tests. In this example, it connects to the `my-app-test` database on
`127.0.0.1` (localhost) instead of the `my-app` database. That way, your tests
will modify a separate database without touching your development data.

> You could normally use `mongodb://localhost/my-app-test` for the database
> URL, but it does not seem to work on Windows for some reason.



## Write your first test

Let's write the test to create a user (`POST /users should create a user`).
You will need to make an HTTP POST request. [SuperTest][supertest] is an HTTP
test library which is designed to test Node.js HTTP servers. You can install it
with npm:

```bash
$> npm install --save-dev supertest
```

Import it into the `spec/users.spec.js` file by adding the following line at the
top:

```js
const supertest = require('supertest');
```

You will also your Express.js application, which presumably is exported from the
`app.js` file at the root of your repository. Add the following line to import
it:

```js
const app = require('../app');
```

Now you can implement your test. **Modify the `it('should create a user')`
call** to add the test function. It should look like this:

```js
describe('POST /users', function() {
  it('should create a user', async function() {

  });
});
```

Add the following contents inside the `async function`:

```js
const res = await supertest(app)
  .post('/users')
  .send({
    name: 'John Doe',
    password: '1234'
  })
  .expect(200)
  .expect('Content-Type', /json/);
```

SuperTest uses chained calls to let you build your HTTP call step by step. You
first call `supertest(app)` to give your app to the SuperTest client, then use
its various chainable methods:

* `.post('/path')` makes a POST request (or `.get` for a GET request, `.patch`
  for a PATCH request, etc).
* `.send(body)` lets you set the request body. It is serialized as JSON by
  default.
* `.expect(number)` is an **assertion**. It lets you specify which status code
  you expect the response to have. If your API responds with a different status
  code, an error will be thrown indicating that there was a problem.
* `.expect('header', value)` lets you specify the value you expect a specific
  header to have in the response. In this case, you expect your API to send back
  JSON in the response (the regular expression `/json/` matches any string that
  contains the word `json`, like `application/json`).

> Check the [SuperTest documentation][supertest-examples] for more examples and
> information on the various methods.

If you run `npm test` now, you should see something like this:

```bash
$> npm test

> my-app@0.0.0 test /path/to/my-app
> mocha spec/**/*.spec.js

  POST /users
POST /users 200 93.114 ms - 52
    ✓ should create a user (109ms)

  GET /users
    - should retrieve the list of users

  1 passing (115ms)
  1 pending
```

### Disconnect from the database once the tests are done

You may notice that the `npm test` command does not exit and that you have to
stop it manually with `Ctrl-C`. This is because Mongoose is not designed to work
with tests: it is designed to stay connected to the database.

You have to tell Mongoose to disconnect after your tests are done. Import
Mongoose at the top of the file:

```js
const mongoose = require('mongoose');
```

At this at the bottom of the file:

```js
after(mongoose.disconnect);
```

> Here you are using [Mocha hooks][mocha-hooks]. The `before` and `after` global
> functions provided by Mocha allow you to run code before or after your test
> suite.

### Fix Mongoose `collection.ensureIndex is deprecated` warning

If you use version 5.x of Mongoose, you may see the following warning:

```
(node:21235) DeprecationWarning: collection.ensureIndex is deprecated. Use createIndexes instead.
```

To get rid of it, set the `useCreateIndex` option to true in your
`mongoose.connect` call (presumably in `app.js`):

```js
mongoose.connect(process.env.MONGODB_URI || 'mongodb://localhost/my-app', {
  // <PREVIOUS OPTIONS HERE...>
  useCreateIndex: true
});
```

### Get rid of request logs while testing

You may have a request logger in your Express.js application. If you used
`express-generator`, you might see this line in the `npm test` command's output:

```
POST /users 200 93.114 ms - 52
```

It is because of this line in `app.js` (for `express-generator` apps):

```js
app.use(logger('dev'));
```

This middleware logger will log all HTTP requests made to the application. You
do not need this information while testing (you already know that you are
calling `POST /users` in this test).

You can conditionally omit this middleware in test mode:

```js
// Log requests (except in test mode).
if (process.env.NODE_ENV !== 'test') {
  app.use(logger('dev'));
}
```

Setting this environment variable when running your tests now gets rid of the
request logs:

```bash
$> NODE_ENV=test npm test

> my-app@0.0.0 test /path/to/my-app
> mocha spec/**/*.spec.js

  POST /users
    ✓ should create a user (109ms)

  GET /users
    - should retrieve the list of users

  1 passing (115ms)
  1 pending
```

To avoid setting this variable every time, you can use `cross-env` again.
**Update** the `test` script in your `package.json` file to look like this:

```json
"scripts": {
  "...": "<PREVIOUS SCRIPTS HERE...>",
  "test": "cross-env MONGODB_URI=mongodb://127.0.0.1/my-app-test NODE_ENV=test mocha spec/**/*.spec.js"
}
```



## Add a unicity constraint to your model

To illustrate a problem that is often encountered with testing, you will now add
a unicity constraint to the `name` property of users.

First, you should probably remove existing users from the test database if you
have run the test more than once. Otherwise, MongoDB will not be able to create
the unique index (since there are already duplicates stored in the collection):

```bash
$> mongo my-app-test
> db.users.remove({})
WriteResult({ "nRemoved" : 3 })
```

Now **update** the schema to look like this (add a `unique` constraint to the
`name` property):

```js
const userSchema = new Schema({
  name: {
    type: String,
    unique: true
  },
  password: String
});
```

Run `npm test` once. It should work, since there are no users in the database.
Run it a second time, and it will fail because it is trying to insert the same
user again.

Your test changes behavior depending on the state of the database.



## Reproducible tests

Your automated tests should always behave the same way. In other words, they
should be **reproducible**. There are generally 2 solutions to achieve this:

* Make sure that the **initial state is always the same** when the test runs
  (e.g. the state of the database).
* Or, make sure that the test **uses different (probably random) data every
  time** to avoid collisions, especially when it comes to unicity constraints.

Which solution is best is debatable. In this tutorial, we will use the first
one: you will make sure that when each test runs, the state of the database is
always the same.

The simplest way to do that is to **wipe the database clean before each test**.
This ensures that any test will alway starts with the same state: nothing.

Create a new `spec/utils.js` file with the following function:

```js
const User = require('../models/user');

exports.cleanUpDatabase = async function() {
  await Promise.all([
    User.deleteMany()
  ]);
};
```

> This new `cleanUpDatabase` function uses your Mongoose model to delete all
> entries. You could add more `deleteMany` calls for other models to the
> `Promise.all([])` array to delete other collections in parallel.

Import this new function at the top of the `spec/users.spec.js` file:

```js
const { cleanUpDatabase } = require('./utils');
```

You can now call it before each test by adding this line before your `describe`
calls:

```js
beforeEach(cleanUpDatabase);
```

> You are using [Mocha hooks][mocha-hooks] again. The `beforeEach` and
> `afterEach` global functions provided by Mocha allow you to run code before or
> after each test runs.

You should now be able to run `npm test` several times in a row without error.
Since the database is wiped every time before your test runs, it can keep
creating the same user with the same name every time.



## Write more detailed assertions

So far, your test makes a POST request on `/users` and checks that the response
has the status code 200 OK with a `Content-Type` header indicating that the
response is JSON.

That's nice, but you are not checking what is in the response body yet. With
SuperTest, the response object (`res`) has a `body` property which contains the
parsed JSON body. You can make assertions on it too.

Add the following assertions to your test after the SuperTest call chain:

```js
// Check that the response body is a JSON object with exactly the properties we expect.
expect(res.body).to.be.an('object');
expect(res.body._id).to.be.a('string');
expect(res.body.name).to.equal('John Doe');
expect(res.body).to.have.all.keys('_id', 'name');
```

Chai has [many assertions][chai-bdd] you can use to verify the shape of objects
or any other kind of JavaScript value (e.g. arrays).

When testing a particular API route, **you should make assertions on everything
your route does that is expected to be used by the end user**. You want to make
sure that your API works as advertised. For example: check the status code,
check important headers, check the response body.

> Note the `expect(res.body._id).to.be.a('string')` assertion. You cannot check
> the exact value of the user's ID because you cannot know it until the user has
> been created. So you just check that it's a string. If you wanted to go
> further, you could check the exact format of that ID with
> `expect(res.body._id).to.match(/^[0-9a-f]{24}$/)` (for MongoDB IDs).
>
> Also note the `expect(res.body).to.have.all.keys('_id', 'name')` assertion.
> You have an assertion to check the ID and another to check the name, but it's
> also important to check that the body does not contain other properties you
> were not expecting. That way, when you add more properties to your schema, the
> test will remind you that you should add new assertions.
>
> If you wanted to go further, you could also check that the created user has
> actually been saved to the database. There could conceivable be a bug where
> the API gives you the correct answer even though it saved something slightly
> different to the database (or did not save it at all).



## Write a second test

Let's test the application's other route. **Modify the `it('should retrieve the
list of users')` call** to add the test function. It should look like this:

```js
describe('GET /users', function() {
  it('should retrieve the list of users', async function() {

  });
});
```

Add the following contents inside the `async function`:

```js
const res = await supertest(app)
  .get('/users')
  .expect(200)
  .expect('Content-Type', /json/);
```

This is very similar to the previous test. Note that you are not using `.send`
this time. Since this is a GET request, no request body can be sent.

Make some assertions on the request body:

```js
expect(res.body).to.be.an('array');
expect(res.body).to.have.lengthOf(0);
```



## Test fixtures

In the case of the `POST /users` test, it was necessary to have an empty
database to avoid issues with the unicity constraint. But it's a bit of a
problem for the `GET /users` test: with the database empty, the API will always
respond with an empty list. That's not a really good test of the list
functionality.

You need specific data to already be in the database before you run the test, so
that you will know what the expected result is. This is what's called a [test
fixture][fixture]: something you use to consistently test a piece of code.

Because each test is different, **each should set up its own fixtures** so that
the initial state is exactly as expected.

In the case of `GET /users`, you need some users in the database before you
attempt to retrieve the list. To create them, you will need the `User` model,
which you can import by adding this to the top of the test file:

```js
const User = require('../models/user');
```

You now need to make sure that some users are created **before the test runs**.
You can use Mocha's `beforeEach` hook. Just make sure to put it in the right
place. You want these fixtures to be created for the `GET /users` test, but not
for the `POST /users` test. You can achieve this by putting it inside the
`describe('GET /users', ...)` block: it will only apply to tests in that block.

Here's how it should look like:

```js
describe('GET /users', function() {
  beforeEach(async function() {
    // Create 2 users before retrieving the list.
    await Promise.all([
      User.create({ name: 'John Doe' }),
      User.create({ name: 'Jane Doe' })
    ]);
  });

  it('should retrieve the list of users', async function() {
    // ...
  });
});
```

If you run `npm test` now, your test will fail because you made an assertion
that the list should be empty. But thanks to the fixtures you created, it now
has 2 users:

```bash
$> npm test
> my-app@0.0.0 test /path/to/my-app
> cross-env MONGODB_URI=mongodb://127.0.0.1/my-app-test NODE_ENV=test mocha spec/**/*.spec.js

  POST /users
    ✓ should create a user (104ms)

  GET /users
    1) should retrieve the list of users

  1 passing (194ms)
  1 failing

  1) GET /users
       should retrieve the list of users:

      AssertionError: expected [ Array(2) ] to have a length of 0 but got 2
      + expected - actual

      -2
      +0
```

Update the assertion to fit the new data:

```js
expect(res.body).to.have.lengthOf(2);
```

Now add some more assertions to check that the array contains exactly what you
expect:

```js
expect(res.body[0]).to.be.an('object');
expect(res.body[0]._id).to.be.a('string');
expect(res.body[0].name).to.equal('Jane Doe');
expect(res.body[0]).to.have.all.keys('_id', 'name');

expect(res.body[1]).to.be.an('object');
expect(res.body[1]._id).to.be.a('string');
expect(res.body[1].name).to.equal('John Doe');
expect(res.body[1]).to.have.all.keys('_id', 'name');
```

You are mainly testing 2 things here:

* As before, you are checking that the records in the database (the users in
  this case) have been correctly sent in the response with the expected
  properties.
* In addition, you are checking that the list is sorted in the correct order (by
  ascending name): `Jane Doe` must be first and `John Doe` second.



## Am I testing every possible scenario?

You now have partial coverage on these two `POST /users` and `GET /users`
routes. But the two tests you have written only cover the best case scenario:
what happens when everything goes as planned and everyone is happy.

When writing automated tests, you should attempt to cover all likely scenarios.
Here's a few examples of some tests you could add to this small project:

* If you have validations, write tests that make sure that you cannot create a
  user with invalid data. For example:
  * Attempt to create a user with an empty name, or a name that's too long.
    Check that it fails.
  * Attempt to create a user with a name that already exists (by using a test
    fixture).
* Keep the early test your wrote that checks what happens when the user list is
  empty. Once your system goes to production, it may never produce an empty list
  of users again. But it would be nice to know that your system still works in
  its initial state, especially if you might need to deploy another instance for
  another customer in the future.
* If your list supports various filters, sorts or pagination, you should write
  tests for these as well.




## Optional: check your test coverage

Install [nyc][nyc], the command line interface for [Istanbul][istanbul], a test
coverage analysis tool:

```bash
$> npm install --save-dev nyc
```

**Update** the `test` script in your `package.json` to add `nyc --reporter=html`
right before the `mocha` command:

```json
"scripts": {
  "...": "<PREVIOUS SCRIPTS HERE...>",
  "test": "cross-env MONGODB_URI=mongodb://127.0.0.1/my-app-test NODE_ENV=test nyc --reporter=html mocha spec/**/*.spec.js"
}
```

Add the following directories to your `.gitignore` file:

```
/.nyc_output
/coverage
```

Run `npm test` again. You should see a new `coverage` directory appear. If you
open the `index.html` file within, you will see a report indicating which lines
of your code are covered by your automated tests, and which are not.

It is not always possible to achieve 100% coverage, but generally the higher
your coverage, the better chance you have of catching bugs or breaking changes.



## Tips

### Chai

* You can make negative assertions with `.not`:

  ```js
  expect(numberVariable).to.not.be.an.('object');
  ```
* Pay attention to the subtle difference between
  `expect(object).to.equal(anotherObject)` and
  `expect(object).to.eql(anotherObject)`. The `equal` assertion checks for
  strict equality, while the `eql` assertion performs a deep comparison. For
  example, take the following two objects:

  ```js
  const object = { foo: 'bar' };
  const anotherObject = { foo: 'bar' };
  ```

  These are **not the same object**: they are two separate object instances, so
  an `equal` assertion would not pass. However, **the structure of both objects
  is the same**, so an `eql` assertion would pass.



## Documentation

* [Mocha][mocha] (test framework)
  * [Hooks][mocha-hooks]
* [Chai][chai] (assertion library)
  * [Assertions][chai-bdd] ([BDD][bdd] style)
* [SuperTest][supertest] (HTTP test library)
* [nyc][nyc] (test coverage analysis)
* [Express.js][express] (Node.js web framework)
* [Mongoose][mongoose] (Node.js object-document mapper)

**Further reading**

* [Behavior-Driven Development][bdd]
* [Test fixtures][fixture]
* [Test-Driven Development][tdd]
* [Top JavaScript Testing Frameworks in Demand for 2019][top-js-test-frameworks-2019]





[async-await]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function
[automated-testing]: https://mediacomem.github.io/comem-archidep/2019-2020/subjects/automated-testing/?home=MediaComem%2Fcomem-archidep%23readme#1
[bdd]: https://en.wikipedia.org/wiki/Behavior-driven_development
[chai]: https://www.chaijs.com
[chai-bdd]: https://www.chaijs.com/api/bdd/
[cross-env]: https://www.npmjs.com/package/cross-env
[express]: https://expressjs.com
[fixture]: https://en.wikipedia.org/wiki/Test_fixture
[istanbul]: https://istanbul.js.org
[mocha]: https://mochajs.org
[mocha-hooks]: https://mochajs.org/index.html#hooks
[mongo]: https://www.mongodb.com
[mongoose]: https://mongoosejs.com
[node]: https://nodejs.org
[npm]: https://www.npmjs.com
[nyc]: https://github.com/istanbuljs/nyc
[promise]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise
[supertest]: https://github.com/visionmedia/supertest
[supertest-examples]: https://github.com/visionmedia/supertest#example
[tdd]: https://en.wikipedia.org/wiki/Test-driven_development
[top-js-test-frameworks-2019]: https://blog.bitsrc.io/top-javascript-testing-frameworks-in-demand-for-2019-90c76e7777e9
