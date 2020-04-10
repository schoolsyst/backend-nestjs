# schoolsyst's API

## Authentification
All routes require a JWT token to access (except `/auth/` routes)

To access routes, first [obtain your token](#routes-auth), then, pass it in the `Authorization` header using the format `Bearer <your token>`.

## Common objects
Some objects are reused in a number of routes. Their properties are listed here.

### Subject
Represents a subject of the user.

|Property|Type|Constraints|Description|
|--------|----|-----------|-----------|
| name | string | max length: 100<br>required. | The subject's display name.
| slug | slug | max length: 100<br>must be unique to the user. | Derived from the `name`, contains only alphanumerical characters and the dash "-". Case insensitive.
| 

## Routes

### POST /auth/
> Obtain a JWT token, given a email and a password.

Provide the `email` and `password` in the request's body as such:

```json
{
  "email": "<your email>",
  "password": "<your password>"
}
```

The response will be of the following form:

```json
{
  "access": "<your access token>",
  "refresh": "<your refresh token>"
}
```

### POST /auth/refresh/
> Renew your access token provided the refresh token

Given the refresh token: 
```json
{
  "refresh": "<your refresh token>"
}
```

The response will contain the new access token:

```json
{
  "access": "<your new access token>",
}
```



### GET /users/self/

> Information about the current user (the one authentificated by the access token)

The response will contain: 

- `email` - unique (case insensitive)
- `activated` - Whether or not the user has confirmed his email address.

### POST /users/

> Create an account.

The request must contain:

- email
- password

Upon registration, the API...
1. creates a new unique activation token and saves it on `<the user>.activation_token`
2. sends a confirmation email to `email`, this email contains a button that links to `https://api.schoolsyst.com/users/activate/?token=<the verify token>&email=<the email address>`

### POST /users/activate

Query parameters:

| Name | Type | Constraints | Description
|---|---|---|---|
| email | email address | Exists in the database | 
| token | string | Exists in the database | Stored as a property named `activation_token` on each user. Destroyed 24 hours after creation.

---

Routes beyond this point require the user to have its activation state (`activated` property) set to `true`. Otherwise a `401 Unauthorized` response will be sent.

---

### /settings/

Settings of the user.

| Name | Type | Constraints | Default value | Description
|---|---|---|---|---|
| theme | One of 'light', 'dark' or 'auto' | 'auto'
| 

------------------------------
<p align="center">
  <a href="http://nestjs.com/" target="blank"><img src="https://nestjs.com/img/logo_text.svg" width="320" alt="Nest Logo" /></a>
</p>

[travis-image]: https://api.travis-ci.org/nestjs/nest.svg?branch=master
[travis-url]: https://travis-ci.org/nestjs/nest
[linux-image]: https://img.shields.io/travis/nestjs/nest/master.svg?label=linux
[linux-url]: https://travis-ci.org/nestjs/nest
  
  <p align="center">A progressive <a href="http://nodejs.org" target="blank">Node.js</a> framework for building efficient and scalable server-side applications, heavily inspired by <a href="https://angular.io" target="blank">Angular</a>.</p>
    <p align="center">
<a href="https://www.npmjs.com/~nestjscore"><img src="https://img.shields.io/npm/v/@nestjs/core.svg" alt="NPM Version" /></a>
<a href="https://www.npmjs.com/~nestjscore"><img src="https://img.shields.io/npm/l/@nestjs/core.svg" alt="Package License" /></a>
<a href="https://www.npmjs.com/~nestjscore"><img src="https://img.shields.io/npm/dm/@nestjs/core.svg" alt="NPM Downloads" /></a>
<a href="https://travis-ci.org/nestjs/nest"><img src="https://api.travis-ci.org/nestjs/nest.svg?branch=master" alt="Travis" /></a>
<a href="https://travis-ci.org/nestjs/nest"><img src="https://img.shields.io/travis/nestjs/nest/master.svg?label=linux" alt="Linux" /></a>
<a href="https://coveralls.io/github/nestjs/nest?branch=master"><img src="https://coveralls.io/repos/github/nestjs/nest/badge.svg?branch=master#5" alt="Coverage" /></a>
<a href="https://gitter.im/nestjs/nestjs?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=body_badge"><img src="https://badges.gitter.im/nestjs/nestjs.svg" alt="Gitter" /></a>
<a href="https://opencollective.com/nest#backer"><img src="https://opencollective.com/nest/backers/badge.svg" alt="Backers on Open Collective" /></a>
<a href="https://opencollective.com/nest#sponsor"><img src="https://opencollective.com/nest/sponsors/badge.svg" alt="Sponsors on Open Collective" /></a>
  <a href="https://paypal.me/kamilmysliwiec"><img src="https://img.shields.io/badge/Donate-PayPal-dc3d53.svg"/></a>
  <a href="https://twitter.com/nestframework"><img src="https://img.shields.io/twitter/follow/nestframework.svg?style=social&label=Follow"></a>
</p>
  <!--[![Backers on Open Collective](https://opencollective.com/nest/backers/badge.svg)](https://opencollective.com/nest#backer)
  [![Sponsors on Open Collective](https://opencollective.com/nest/sponsors/badge.svg)](https://opencollective.com/nest#sponsor)-->

## Description

[Nest](https://github.com/nestjs/nest) framework TypeScript starter repository.

## Installation

```bash
$ npm install
```

## Running the app

```bash
# development
$ npm run start

# watch mode
$ npm run start:dev

# production mode
$ npm run start:prod
```

## Test

```bash
# unit tests
$ npm run test

# e2e tests
$ npm run test:e2e

# test coverage
$ npm run test:cov
```

## Support

Nest is an MIT-licensed open source project. It can grow thanks to the sponsors and support by the amazing backers. If you'd like to join them, please [read more here](https://docs.nestjs.com/support).

## Stay in touch

- Author - [Kamil Myśliwiec](https://kamilmysliwiec.com)
- Website - [https://nestjs.com](https://nestjs.com/)
- Twitter - [@nestframework](https://twitter.com/nestframework)

## License

  Nest is [MIT licensed](LICENSE).
