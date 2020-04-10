# schoolsyst's API

## Authentification

All routes require a JWT token to access (except `/auth/` routes)

To access routes, first [obtain your token](#routes-auth), then, pass it in the `Authorization` header using the format `Bearer <your token>`.

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

| Name  | Type          | Constraints            | Description                                                                                    |
| ----- | ------------- | ---------------------- | ---------------------------------------------------------------------------------------------- |
| email | email address | Exists in the database |
| token | string        | Exists in the database | Stored as a property named `activation_token` on each user. Destroyed 24 hours after creation. |

---

Routes beyond this point require the user to have its activation state (`activated` property) set to `true`. Otherwise a `401 Unauthorized` response will be sent.

From here, all resources are tied to a user, and all have creation and last updated date read-only fields, named respectively `created_at` and `updated_at`

---

### /settings/

Settings of the user.

| Name               | Type                             | Constraints       | Default value                                                          | Description                                                                                                                                                                                                                                      |
| ------------------ | -------------------------------- | ----------------- | ---------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| theme              | One of 'light', 'dark' or 'auto' | 'auto'            |
| year_layout        | daterange[]                      | array.length >= 1 | _required_                                                             | Configures how the year is split. For example, for a student whose school works on semesters, the layout will be `[{start: <start of the year>, end: <end of the first semester>}, {start: <start of the 2nd semester, end: <end of the year>}]` |
| starting_week_type | 'even' or 'odd'                  | 'even'            | Whether the first week of school is an 'odd'-type week of 'even'-type. |
| grades_unit        | float                            | > 0               | _required_                                                             | What unit is used to display grades. Note that grades are stored as floats in [0; 1], no matter what this value is set to.                                                                                                                       |
| holidays           | daterange[]                      |                   | `[]`                                                                   |

NOTE: This endpoint only accepts `GET` and `PATCH` requests, not `POST` (you cannot create _new_ settings)

### /subjects/

| Name   | Type              | Constraints                                    | Default value | Description                                                                                                                     |
| ------ | ----------------- | ---------------------------------------------- | ------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| name   | string            | max length: 100                                | _required_    | The subject's display name.                                                                                                     |
| slug   | slug              | max length: 100<br>must be unique to the user. | _required_    | Derived from the `name`, contains only alphanumerical characters and the dash "-". Case insensitive.                            |
| color  | int (hexadecimal) | must be a valid hex color value                | `0x000000`    |
| weight | float             | >= 0                                           | 1             |                                                                                                                                 |
| goal   | float             | ∈ [0; 1]                                       |               |
| room   | string            | max length: 100                                |               | If the courses of the subject are always in the same room, it's set here and will serve as a default for the "add event" modal. |

### /homework/

| Name         | Type                                                  | Constraints                              | Default value | Description                             |
| ------------ | ----------------------------------------------------- | ---------------------------------------- | ------------- | --------------------------------------- |
| title        | string                                                | max length: 500                          | _required_    |
| subject      | Subject object                                        |                                          | _required_    |
| type         | one of 'test', 'coursework', 'to_bring' or 'exercise' |                                          | _required_    |
| completed_at | datetime                                              | must be in the past (≤ now)<br>read-only | `null`        | The last time `progress` was set to `1` |
| progress     | float                                                 | ∈ [0; 1]                                 | 0             | 0 means not started, 1 means finished.  |
| notes        | Note[]                                                | notes must be owned by the user          | `[]`          | Linked notes                            |
| grades       | Grade[]                                               | grades must be owned by the user         | `[]`          | Linked grades.                          |

### /grades/

| Name        | Type     | Constraints         | Default value | Description                                                             |
| ----------- | -------- | ------------------- | ------------- | ----------------------------------------------------------------------- |
| title       | string   | max length: 500     | _required_    |
| actual      | float    | ∈ [0; 1]            | `null`        | The actual mark                                                         |
| expected    | float    | ∈ [0; 1]            | `null`        | The mark the user thought it would get after doing the test             |
| goal        | float    | ∈ [0; 1]            | `null`        | The goal mark                                                           |
| unit        | float    | > 0                 | _required_    |
| weight      | float    | >= 0                | 1             |
| obtained_at | datetime | must be in the past | `null`        | The datetime where `actual` was first set to a value other than `null`. |

### /notes/

| Name    | Type                                                | Constraints     | Default value | Description                                                        |
| ------- | --------------------------------------------------- | --------------- | ------------- | ------------------------------------------------------------------ |
| name    | string                                              | max length: 500 | ""            |
| content | string                                              |                 | ""            |
| type    | One of 'html', 'markdown', 'asciidoc' or 'external' |                 | 'html'        | For 'external' notes, the content is the URL pointing to the note. |
| quizzes | quiz[] | | `[]` | Linked quizzes.

### /quizzes/

| Name     | Type   | Constraints     | Default value | Description |
| -------- | ------ | --------------- | ------------- | ----------- |
| name | string | max length: 500 | _required_ |  
| questions | Question[] | | `[]` | An array of questions. The definition of those object is still a work in progress.
| tries.test | integer | >= 0 | 0 | Number of trials in test mode |
| tries.train | integer | >= 0 | 0 | Number of trials in train mode |
| tries.total | integer | >= 0 | 0 | Total number of trials |
| modified_at | datetime | must be in the past | `now()` | When was `questions` or `name` last modified
| time_spent | integer |  >= 0 | 0 | (in seconds) the total time spent on this quizz
| sessions | QuizSession[] | | `[]` | An array of quiz sessions. The definition of those object is still a work in progress.

