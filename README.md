# Frontend Task for Frontend Position (Paid)

This test is a part of our hiring process for the Frontend Engineer. It should take you between 5-7 hours, depending on your experience, to implement the minimal version. But we thought about a few bonuses, so feel free to spend some time on them if you want.

First time freelancers looking to build their profiles are encouraged to apply. You need to submit this within 48 hours of receiving this, and you'll be paid for 7 hours expected time for this task. If you're not able to submit it in 48 hours and your task doesn't do the required functionality then we'll cancel the contract.

## Exercise

The application needs be built on Next.js using styled components, TypeScript.

For the purpose of this test, you can use Bootstrap, Material or Ant Design for the base design library. Sorry mate but you can't use Tailwind CSS. Copy Styling of different components such as buttons, lists, fields etc. from the [Vercel's Design System](https://vercel.com/design/introduction) , if you've a better and faster way to do this then we'll give you additional 25% bonus over the total amount. At the end of this task, your design should look something like this -

 <img width="1452" alt="Screenshot 2023-06-11 at 3 09 35 AM" src="https://github.com/Decibel-io/nextjs-task/assets/93420471/8e9bf88e-8ea9-4da1-bd0c-9c1b394eca6c"> 


This application must:
- Display a paginated list of calls that you’ll retrieve from the API.
- Display the call details view if the user clicks on a call. the view should display all the data related to the call itself.
- Be able to archive one or several calls
- Group calls by date
- Handle real-time events (Whenever a call is archived or a note is being added to a call, these changes should be reflected on the UI immediately)

Bonus (extra 50% bonus on top of base amount):
- Provide filtering feature, to filter calls by type (archived, missed …)
- Use GraphQL to fetch data
- Deploy your application to Vercel. or GitHub pages.

**Important Note**: We want you to build this small app as you'd have done it for your current job. (UI, UX, tests, documentation matters).

## APIs

There are 2 versions of the APIs for this test, so you can choose between:
- REST API or 
- GraphQL API.

Both expose the same data, so it’s really about which one you prefer.

### Model

Both APIs use the same models.

Call Model

```
type Call {
  id: ID! // "unique ID of call"
  direction: String! // "inbound" or "outbound" call
  from: String! // Caller's number
  to: String! // Callee's number
  duration: Float! // Duration of a call (in seconds)
  is_archived: Boolean! // Boolean that indicates if the call is archived or not
  call_type: String! // The type of the call, it can be a missed, answered or voicemail.
  via: String! // Aircall number used for the call.
  created_at: String! // When the call has been made.
  notes: Note[]! // Notes related to a given call
}
```

Note Model

```
type Note {
  id: ID!
  content: String!
}
```

### GraphQL API (For REST API, scroll down)

Base URL: https://frontend-test-api.aircall.io/graphql

#### Authentication

You must first authenticate yourself before requesting the API. You can do so by executing the Login mutation. See below.

#### Queries

All the queries are protected by a middleware that checks if the user is authenticated with a valid JWT.

`paginatedCalls` returns a list of paginated calls. You can fetch the next page of calls by changing the values of `offset` and `limit` arguments.

```
paginatedCalls(
  offset: Float = 0
  limit: Float = 10
): PaginatedCalls!

type PaginatedCalls {
  nodes: [Call!]
  totalCount: Int!
  hasNextPage: Boolean!
}
```

`activitiy` returns a single call if any, otherwise it returns null.

```
call(id: Float!): Call
```

`me` returns the currently authenticated user.

```
me: UserType!
```

```
type UserType {
  id: String!
  username: String!
}
```

#### Mutations

To be able to grab a valid JWT token, you need to execute the `login` mutation.

`login` receives the username and password as 1st parameter and return the access_token and the user identity.

```graphql
login(input: LoginInput!): AuthResponseType!

input LoginInput {
  username: String!
  password: String!
}

interface AuthResponseType {
  access_token: String!
  user: UserType
}
```

Once you are correctly authenticated you need to pass the Authorization header for all the next calls to the GraphQL API.

```JSON
{
  "Authorization": "Bearer <YOUR_ACCESS_TOKEN>"
}
```

Note that the access_token is only available for 10 minutes. You need to ask for another fresh token by calling the `refreshToken` mutation before the token gets expired.

`refreshToken` allows you to ask for a new fresh token based on your existing access_token

```graphql
refreshToken: AuthResponseType!
```

This will send you the same response as the `login` mutation.

You must use the new token for the new requests made to the API.

`archiveCall` as the name implies it either archive or unarchive a given call.If the call doesn't exist, it'll throw an error.

```
archiveCall(id: ID!): Call!
```

`addNote` create a note and add it prepend it to the call's notes list.

```
addNote(input: AddNoteInput!): Call!

input AddNoteInput {
  activityId: ID!
  content: String!
}
```

#### Subscriptions

To be able to listen for the mutations/changes done on a given call, you can call the `onUpdateCall` using an actibity ID.

`onUpdateCall` receives the call ID as the 1st parameter and returns a call instance.

```
onUpdateCall(id: ID): Call!
```

Now, whenever a call data changed either via the `addNote` or `archiveCall` mutations, you will receive a subscription event informing you of this change.

_Don't forget to pass the Authorization header with the right access token in order to be able to listen for these changes_

### REST API

Base URL: https://frontend-test-api.aircall.io

#### Authentication

You must first authenticate yourself before requesting the API. You can do so by sending a POST request to `/auth/login`. See below.

#### GET endpoints

All the endpoints are protected by a middleware that checks if the user is authenticated with a valid JWT.

`GET` `/calls` returns a list of paginated calls. You can fetch the next page of calls by changing the values of `offset` and `limit` arguments.

```
/calls?offset=<number>&limit=<number>
```

Response:
```
{
  nodes: [Call!]
  totalCount: Int!
  hasNextPage: Boolean!
}
```

`GET` `/calls/:id` return a single call if any, otherwise it returns null.

```
/calls/:id<uuid>
```

`GET` `/me` return the currently authenticated user.

```
/me
```

Response
```
{
  id: String!
  username: String!
}
```

#### POST endpoints

To be able to grab a valid JWT token, you need to call the following endpoint:

`POST` `/auth/login` receives the username and password in the body and returns the access_token and the user identity.

```
/auth/login

// body
{
  username: String!
  password: String!
}
```

Once you are correctly authenticated you need to pass the Authorization header for all the next calls to the REST API.

```JSON
{
  "Authorization": "Bearer <YOUR_ACCESS_TOKEN>"
}
```

Note that the access_token is only available for 10 minutes. You need to ask for another fresh token by calling the `/auth/refresh-token` endpoint before the token gets expired.

`POST` `/auth/refresh-token` allows you to ask for a new fresh token based on your existing access_token

This will return the same response as the `/auth/login` resource.

You must use the new token for the new requests made to the API.

`POST` `/calls/:id/note` create a note and add it prepend it to the call's notes list.

```
`/calls/:id/note`

Body
{
  content: String!
}
```

It returns the `Call` as a response or an error if the note doesn't exist.

#### PUT endpoints

`PUT` `/calls/:id/archive` as the name implies it either archive or unarchive a given call. If the call doesn't exist, it'll throw an error.

```
PUT /calls/:id/archive
```

#### Real-time

In order to be aware of the changes done on a call, you need to subscribe to this private channel: `private-aircall` and listen for the following event: `update-call` which will return the call payload.

This event will be called each time you add a note or archive a call.

Note that, you need to use Pusher SDK in order to listen for this event.

Because this channel is private you need to authenticate first, to do that, you need to make 
- `APP_AUTH_ENDPOINT` point to: `https://frontend-test-api.aircall.io/pusher/auth`
- set `APP_KEY` to `d44e3d910d38a928e0be`
- and set `APP_CLUSTER` to `eu`

#### Errors

The REST API can return a different type of errors:

`400` `BAD_REQUEST` error, happens when you provide some data which doesn't respect a given shape.

Example
```
{
  "statusCode": 400,
  "message": [
    "content must be a string",
    "content should not be empty"
  ],
  "error": "Bad Request"
}
```

`401` `UNAUTHORIZED` error, happens when the user is not authorized to perform an action or if his token is no longer valid

Example
```
{
  "statusCode": 401,
  "message": "Unauthorized"
}
```

`404` `NOT_FOUND` error, happens when the user requests a resource that no longer exists.

Example
```
{
  "statusCode": 404,
  "message": "The call does not exist!",
  "error": "Not Found"
}
```

## Code Submit

Please organize, design, test and document your code as if it were going into production, create a loom video with PR and send us a pull request. 

We will review it and get back to you in order to talk about your code! If we don't get back to you in 72 hours then let us know.

All the best and happy coding.
