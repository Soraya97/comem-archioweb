# COMEM+ Web-Oriented Architecture Course

The goal of this course is to learn about the generic concept of **web
service**, focusing on **REST**ful APIs as one way to expose such a service. You
will:

* Learn the **core principles** of the REST architectural style.
* Learn how to **implement** a REST API in JavaScript with Node.js.
* **Deploy** your REST API on a cloud application platform.
* Add a **real-time** component to your REST API with WebSockets.

This course is a [COMEM+][comem] [web development course][comem-webdev] taught at [HEIG-VD][heig].

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [Plan](#plan)
- [What you will need](#what-you-will-need)
- [Useful links](#useful-links)
- [Evaluation](#evaluation)
  - [Delivery](#delivery)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->



## Plan

* [Introduction](https://mediacomem.github.io/comem-archioweb/2019-2020/subjects/course?home=MediaComem%2Fcomem-archioweb%23readme)

* Basics
  * [JavaScript](https://mediacomem.github.io/comem-archioweb/2019-2020/subjects/js?home=MediaComem%2Fcomem-archioweb%23readme)
  * [Node.js](https://mediacomem.github.io/comem-archioweb/2019-2020/subjects/node?home=MediaComem%2Fcomem-archioweb%23readme) JavaScript runtime
  * [npm](https://mediacomem.github.io/comem-archioweb/2019-2020/subjects/npm?home=MediaComem%2Fcomem-archioweb%23readme) Node.js package manager
  * [REST & HTTP](https://mediacomem.github.io/comem-archioweb/2019-2020/subjects/rest?home=MediaComem%2Fcomem-archioweb%23readme)

* Creating a web service
  * [Express](https://mediacomem.github.io/comem-archioweb/2019-2020/subjects/express?home=MediaComem%2Fcomem-archioweb%23readme) web framework
  * [MongoDB](https://mediacomem.github.io/comem-archioweb/2019-2020/subjects/mongodb?home=MediaComem%2Fcomem-archioweb%23readme) document-oriented database
  * [Mongoose](https://mediacomem.github.io/comem-archioweb/2019-2020/subjects/mongoose?home=MediaComem%2Fcomem-archioweb%23readme) Object-Document Mapper

* Deploying your web service
  * [Heroku](https://mediacomem.github.io/comem-archioweb/2019-2020/subjects/heroku?home=MediaComem%2Fcomem-archioweb%23readme) cloud application platform

* Creating a REST API
  * REST in depth
  * [Express best practices](https://mediacomem.github.io/comem-archioweb/2019-2020/subjects/express-best-practices?home=MediaComem%2Fcomem-archioweb%23readme)
  * [Utilizing Mongoose](https://mediacomem.github.io/comem-archioweb/2019-2020/subjects/express-mongoose?home=MediaComem%2Fcomem-archioweb%23readme) in Express (filtering, pagination, aggregation)
  * [Express Authentication](https://mediacomem.github.io/comem-archioweb/2019-2020/subjects/express-auth?home=MediaComem%2Fcomem-archioweb%23readme)
  * [REST API documentation](https://mediacomem.github.io/comem-archioweb/2019-2020/subjects/apidoc?home=MediaComem%2Fcomem-archioweb%23readme) with apiDoc

* Real-time communication
  * WebSockets
  * Web Application Messaging Protocol (WAMP)



## What you will need

* A Unix CLI (Git Bash is included with Git on Windows)
* [Git][git-downloads]
* A free [GitHub][github] account
* [Google Chrome][chrome] (recommended, any browser with developer tools will do)
* [Node.js][node] 12+
* [Postman][postman] (recommended, any tool that makes raw HTTP requests will do)
* [MongoDB][mongodb]
* A free [Heroku][heroku] account
* The [Heroku CLI][heroku-cli]



## Useful links

* [Architecture & source code management diagrams][diagrams]
* [Demonstration REST API implemented with Express][demo-api] ([documentation][demo-api-doc])
* [Command line cheatsheet][cli-cheatsheet]
* [Git cheatsheet][git-cheatsheet]
* [Project suggestions](PROJECTS.md)



## Evaluation

**API**

Your REST API must be developed with the [Express][express] framework and use a
[MongoDB][mongodb] database. It must provide (at least):

* The API must provide **user management**:
  * New users must be able to **register**.
  * Existing users must be able to **authenticate** (to allow users to log in).
* The API must provide at least **2 other types** of resources:
  * Both types must be linked together (e.g. aggregation or composition).
  * At least one of the types must be linked to users.
  * The API must provide minimal CRUD operations to manage and use those types in a mobile application.
* The API must use the knowledge learned during the course:
  * At least one resource must be a **paginated list**.
  * At least one resource must be a **list with optional filters**.
  * At least one resource must provide **aggregated data** from other resources using a [MongoDB aggregation pipeline][mongodb-aggregation]
    (e.g. the number of items created by a user).
  * The API must be developed as a backend to a mobile application
    using at least 2 [**mobile hardware features**][cordova-plugins], for example:
    * At least one resource must be **geolocated**.
    * At least one resource must have one or multiple **pictures**
      (it is sufficient to store a picture URL or URLs in the database).
  * Sensitive operations must be protected by requiring valid **authentication**.
    * Authentication must be provided in the form of a [JWT token][jwt] or an equivalent mechanism.
* The API must have a real-time pub-sub component:
  * *At least one* of the following must be provided:
    * A `ws://` or `wss://` endpoint to which a plain WebSockets client can connect to receive real-time messages.
    * A WAMP pub-sub topic to which a Subscriber can subscribe to receive real-time messages.
  * The WebSockets endpoint or WAMP topic must send real-time messages containing relevant data for the application.
    (For example, a chat application may notify its clients in real-time of the number of channels, messages, etc, to display activity on the home page.)
  * The WebSockets endpoint or WAMP topic may be unprotected (i.e. implementing authentication or authorization is mandatory).

**Infrastructure**

* The source code of your REST API must be in a repository on GitHub.
* Your REST API must be deployed on Heroku.

**Documentation**

* Your REST API must be documented.
* The real-time component of your API must be documented (not necessarily in the same way).

**Quality of the implementation**

* You must follow REST best practices:
  * Your REST resources must use appropriate HTTP methods, headers and status codes.
  * Your REST resources must have a consistent URL hierarchy and/or naming structure.
* Your asynchronous code must be correct.
* Your Express routes must handle asynchronous operation errors.
* You must avoid excessive code duplication (e.g. using Express middleware).
* Your API must have basic validations on user input (e.g. using Mongoose validations).
* Your API must validate the existence of linked resources (e.g. when creating an item linked to a user).



### Delivery

Send an e-mail *no later than __November 25th 2019__* to Simon Oulevay with:

* The list of group members.
* The link to your source code repository on GitHub.
* The link to your deployed REST API on Heroku.



[chrome]: https://www.google.com/chrome/
[cli-cheatsheet]: https://github.com/MediaComem/comem-webdev/blob/master/CLI-CHEATSHEET.md
[comem]: http://www.heig-vd.ch/comem
[comem-webdev]: https://github.com/MediaComem/comem-webdev
[cordova-plugins]: https://cordova.apache.org/docs/en/latest/#plugin-apis
[demo-api]: https://github.com/MediaComem/comem-webdev-express-rest-demo
[demo-api-doc]: https://mediacomem.github.io/comem-webdev-express-rest-demo/
[diagrams]: diagrams.pdf
[express]: https://expressjs.com
[git-cheatsheet]: https://github.com/MediaComem/comem-webdev/blob/master/GIT-CHEATSHEET.md
[git-downloads]: https://git-scm.com/downloads
[github]: https://github.com
[heroku]: https://www.heroku.com/home
[heroku-cli]: https://devcenter.heroku.com/articles/heroku-cli
[heig]: http://www.heig-vd.ch
[jwt]: https://jwt.io/
[mongodb]: https://www.mongodb.com
[mongodb-aggregation]: https://docs.mongodb.com/manual/core/aggregation-pipeline/
[node]: https://nodejs.org/
[postman]: https://www.getpostman.com