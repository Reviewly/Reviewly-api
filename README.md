The Reviewly API
==================

Welcome to the Reviewly API! If you're looking to integrate your application with Reviewly, you're in the right place. We're happy to have you!

Making a request
----------------

All URLs start with **`https://api.reviewly.dev/00000000-0000-0000-0000-000000000000/`**. URLs are HTTPS only. The path is prefixed with the organization UUID, but no `/api/v1` API prefix. Also, note the different domain!

To make a request for all the projects on your account, append the `projects` index path to the base URL to form something like `https://api.reviewly.dev/00000000-0000-0000-0000-000000000000/projects.json`. In cURL, it looks like this:

``` shell
curl -H "Authorization: Bearer $API_KEY" -A 'MyApp (yourname@example.com)' https://api.reviewly.dev/00000000-0000-0000-0000-000000000000/projects.json
```

To create something, it's the same idea, but you also have to include the `Content-Type` header and the JSON data:

``` shell
curl -H "Authorization: Bearer $API_KEY" \
  -H 'Content-Type: application/json' \
  -A 'User-Agent: MyApp (yourname@example.com)' \
  -d '{ "project_uuid": "b5f4b3ea-28df-4475-9ada-dcdb917023d4", "email": "candidate@example.com" }' \
  https://api.reviewly.dev/00000000-0000-0000-0000-000000000000/candidates.json
```

Authentication
--------------

All Reviewly API requests require to be authenticated by passing along the API key. You can find this key in the organization's settings page.

Identifying your application
----------------------------

You must include a `User-Agent` header with **both**:

* The name of your application
* A link to your application or your email address

We use this information to get in touch if you're doing something wrong (so we can warn you before you're blacklisted) or something awesome (so we can congratulate you). Here are examples of acceptable `User-Agent` headers:

* `User-Agent: Freshbooks (http://freshbooks.com/contact.php)`
* `User-Agent: Fabian's Ingenious Integration (fabian@example.com)`

If you don't include a `User-Agent` header, you'll get a `400 Bad Request` response.

JSON only
---------

We use JSON for all API data. The style is no root element and snake_case for object keys. This means that you have to send the `Content-Type` header `Content-Type: application/json; charset=utf-8` when you're POSTing or PUTing data into Reviewly. All API URLs end in `.json` to indicate that they return JSON. Alternatively you can send `Accept: application/json`.

Pagination
----------

Most collection APIs paginate their results. The Reviewly API follows the [RFC8288 convention](https://www.rfc-editor.org/rfc/rfc8288) of using the `Link` header to provide URLs for the `next` page. Follow this convention to retrieve the next page of data—please don't build the pagination URLs yourself!

Here's an example response header from requesting the third page of projects:

```
Link: <https://api.reviewly.dev/00000000-0000-0000-0000-000000000000/projects.json?page=4>; rel="next"
```

The Reviewly API also provides the `Total-Count` header, which displays the total number of resources in the collection you are fetching. And also the `Current-Page`, `Page-Items`, and `Total-Pages` headers.

Handling errors
---------------

API clients must expect and gracefully handle transient errors, such as rate limiting or server errors. We recommend baking 5xx and 429 response handling into your low-level HTTP client so your integration can handle most errors automatically.

### [5xx server errors](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes#5xx_server_errors)

If Reviewly is having trouble, you will get a response with a 5xx status code indicating a server error. 500 (Internal Server Error), 502 (Bad Gateway), 503 (Service Unavailable), and 504 (Gateway Timeout) may be retried with [exponential backoff](https://en.wikipedia.org/wiki/Exponential_backoff).

API endpoints
-------------

Get an organization
--------------

* `GET /organizations/:uuid.json` will return the organization with the given UUID, granted they have access to it.

###### Example JSON Response
```json
{
  "uuid": "3f9dd65e-5b85-4dff-80ce-28156b15dde0",
  "name": "Pied Piper",
  "created_at": "2014-04-06T00:00:00.000Z"
}
```

Get all projects
----------------

* `GET /organizations/:uuid/projects.json` will return a paginated list of active projects in the organization sorted by most recently created project first.

###### Example JSON Response
```json
[
  {
    "uuid": "b5f4b3ea-28df-4475-9ada-dcdb917023d4",
    "name": "Senior Backend Exercise",
    "created_at": "2016-04-24T00:00:00.000Z"
  },
  {
    "uuid": "d03c6ada-bc56-40af-b4de-6f22e8b2f425",
    "name": "Junior Backend Exercise",
    "created_at": "2015-04-12T00:00:00.000Z"
  }
]
```

Invite a candidate
-----------------

* `POST /organizations/:uuid/candidates.json` with the `project_uuid` and `email` to invite a candidate to a project.

###### Example JSON Request

``` json
{
  "project_uuid": "b5f4b3ea-28df-4475-9ada-dcdb917023d4",
  "email": "candidate@example.com"
}
```

This will return `201 Created` with the current JSON representation of the candidate if the creation was a success.

###### Example JSON Response

``` json
{
  "uuid": "a18ab4a2-0983-45d5-b6ea-a4be74e2cd1d",
  "email": "candidate@example.com",
  "name": "John Doe",
  "username": "johndoe",
  "status": "replicating",
  "created_at": "2017-04-23T00:00:00.000Z",
  "project": {
    "uuid": "b5f4b3ea-28df-4475-9ada-dcdb917023d4",
    "name": "Senior Backend Exercise"
  }
}
```

Get all reviews
----------------

* `GET /organizations/:uuid/reviews.json` will return a paginated list of submitted reviews (accepted/rejected) in the organization sorted by most recently submitted first.

###### Example JSON Response
```json
[
  {
    "uuid": "9ad376a2-7dcb-4a56-bcd3-0f9e682fa037",
    "internal_notes": "They will be a great addition to Pied Piper\nThe solution was very clean",
    "feedback": "I liked how you improved the data compression algorithm!",
    "status": "approved",
    "invited_at": "2017-04-30T00:00:00.000Z",
    "started_at": "2017-05-07T00:00:00.000Z",
    "reviewed_at": "2017-05-14T00:00:00.000Z",
    "project": {
      "uuid": "b5f4b3ea-28df-4475-9ada-dcdb917023d4",
      "name": "Senior Backend Exercise"
    },
    "candidate": {
      "uuid": "a18ab4a2-0983-45d5-b6ea-a4be74e2cd1d",
      "email": "candidate@example.com",
      "name": "John Doe",
      "username": "johndoe",
      "reviews": {
        "approved_count": 1,
        "rejected_count": 0,
        "pending_count": 2
      }
    },
    "reviewer": {
      "uuid": "2138116b-ffe4-4cbc-bc09-c18b907bb08c",
      "name": "Richard Hendricks",
      "username": "richard"
    }
  }
]
```

Getting Help
------------

If you have a question about the API, please [contact us](https://reviewly.dev).

License
-------

These API docs are licensed under [Creative Commons (CC BY-SA 4.0)](http://creativecommons.org/licenses/by-sa/4.0/). Please share, remix, and distribute as you see fit.
