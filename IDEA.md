# Adonis Brew

Adonis brew is lean and fully featured testing provider for AdoniJs. This file shares the intial idea before writing the single of code.

> NB - This is an idea of what the final implementation should look like and may be the final result is different from what described in this document.


## Types Of Testing

Testing is a very broad concept and has very thin lines to understand the difference between the different types of testing. Let's divide them into seperate categories and then sub-categories.


#### End to End Testing

End to end testing is done for frontend heavy apps or applications which generate HTML and manage state on the frontend using frameworks like **Ember**, **Angular** and **Vue**, rather than generating HTML on server and making round trips for each page load/actions.


End to end testing is out the scope of AdonisJs, since it is a framework for NodeJs. You can try [Night Watch](http://nightwatchjs.org/), [Karma Js](https://karma-runner.github.io/1.0/index.html) or similar for writing End to end tests.

#### Unit Testing

Unit testing is meant to test small pieces of code in isolation. While writing unit tests, you will never deal with the flow of the application, instead you will test individual modules. For example:

1. Testing a service, which reads value from a JSON file and return a strucutured object.
2. Testing a service which tells whether a user is an Admin or not.


#### Mocks

Mocks helps you making use of dummy implementation of actual modules. For example: 

When trying to test whether a user is an Admin or not, you will need a **Lucid model**, and while writing tests, you can mock that model with a plain Javascript class and test your service. Below is an example implementation.

```javascript
'use strict'

class ACL {
  
  static get inject () {
    return ['App/Model/User']
  }

  constructor (User) {
    this.User = User
    this.adminAccessLevel = 4
  }

  * isAdmin (userId) {
    const user = yield this.User.find(UserId)
    return user.level === this.adminAccessLevel
  }

}

module.exports = ACL
```

Above is a simple class which returns whether a user with a given id is an admin or not.

Now in order to test this service, you can mock the **User model** by passing a custom implementation.

```javascript
'use strict'

const UnitTest = use('UnitTest')
const ACL = use('App/Services/ACL')

class DummyUser {
  
  find (id) {
    if (id === 2) {
      return {level: 4}
    } else {
      return {level: 1}
    }
  }  

}

class ACLTest extends UnitTest {

  * testWhenUserIsAdmin (test) {
    test.description('it should return true when user access level is 4')
    const acl = new ACL(new DummyUser())
    const isAdmin = yield acl.isAdmin(2)
    this.expect(isAdmin).to.equal(true);
  }

  * testWhenUserIsNotAdmin (test) {
    test.description('it should return false when user access level is not 4')
    const acl = new ACL(new DummyUser())
    const isAdmin = yield acl.isAdmin(1)
    this.expect(isAdmin).to.equal(false);
  }

}

module.exports = ACLTest
```


This is not the most practical test, but it gives you can idea, how you can make use of `constructor` injection to mock dependencies on runtime. If required you can make use of [Sinon Js](http://sinonjs.org/) or similar to have more control over mocks.

#### Integration Testing

Integration testing is done when we try to test a piece of functionality to make sure that it works fine. These tests do not test the flow of the application but instead test a larger chunk of code.

AdonisJs provides same helpers for Integration testing.

#### Application Testing

Application tests are Zoomed out tests, where you test your application just like a regular customer. Since application testing requires lots of code pieces to be glued together to be tested, AdonisJs provides a handful of tools to make the process easier.

Application testing is again divided into 2 seperate sub-categories called REST API testing and DOM related tests.

#### REST API 

AdonisJs makes use of [https://github.com/visionmedia/supertest](supertest) internally and exposes a nice API to write test for REST API's. For example:

```javascript
const ApplicationTest = use('ApplicationTest')

class UsersTest extends ApplicationTest {

  testSeeUsersList (test) {
    test.description('it should return a list of all the users')

    yield this
      .get('/users')
      .exceptStatus(200)
      .json()
      .isArray()
      .hasLength(10)
      .each((user) => {
        this.expect(user).has.property('username')
        this.expect(user).has.property('email')
      })

  }

}

module.exports = ApplicationTest
```

Also you can login before making an HTTP request.

```javascript
const ApplicationTest = use('ApplicationTest')

class UsersTest extends ApplicationTest {

  testSeeUsersList (test) {
    test.description('it should return a list of all the users')

    const response = yield this
      .get('/users')
      .loginVia(1)
      .end()

  }

}

module.exports = ApplicationTest
```

#### HTML Test

Not only JSON, you can also test and interact with your HTML pages using within your test.

```javascript
const ApplicationTest = use('ApplicationTest')

class UsersTest extends ApplicationTest {

  testSeeUsersList (test) {
    test.description('it should return a list of all the users')

    yield this
      .visit('/login')
      .fill('email', 'foo@bar.com')
      .fill('password', 'secret')
      .press('Login')
      .see('/dashboard')
      .end()
  }

}

module.exports = ApplicationTest
```
