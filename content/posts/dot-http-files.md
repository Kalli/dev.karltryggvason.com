+++
date = '2025-12-07T14:59:17+02:00'
title = 'Your project should have .http files'
+++

It is a common ritual, grepping through http handlers or spelunking in a project's Slack when you just want to know how to hit the auth endpoint. But it doesn't have to be like this! If your service or app exposes or consumes an http api it should have `.http` file(s) that show how to make requests to it. Simple, executable, version controlled request documentation that improves developer experience.

In these files you write out http requests to the api endpoints using a familiar and easy to understand syntax (based on the [RFC9110 - HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html) specification). Here is a very simple example:

```bash
###
# @name getPetById
# Find pet by ID

GET https://petstore.swagger.io/v2/pet/3
```

This is a `GET` request to the url that follows. The three `#` symbols separate different requests, and a single `#` starts a comment. You can use `@name` to name the request to bring structure and documentation to your file. Request headers and bodies and can be added below the request and most tools have support for persisting cookies or auth headers. Here is an example `POST` request to illustrate a more complex request:

```bash
###
# @name addPet
# Add a new pet to the store

POST https://petstore.swagger.io/v2/pet
Content-Type: application/json
Authorization: Bearer {{token}}

{
  "id": 0,
  "name": "Fido",
  "status": "{{name}}"
}

###
# define variables
@status={{available}}
@token=redacted-super-secret-stuff
```

Here we've also included variables (defined with `@status=`, used with double curly braces `{{status}}`). Variables can improve readability and make switching between environments or scenarios a breeze.

<details>
    <summary> Click to expand more examples </summary>

Here are a few more example requests from the [Swagger Petstore](https://petstore.swagger.io) spec.

```sh

###
# @name findPetsByStatus
# Finds Pets by status

GET http://petstore.swagger.io/v2/pet/findByStatus?status=available
Authorization: Bearer {{token}}

###
# @name deletePet
# Deletes a pet

DELETE http://petstore.swagger.io/v2/pet/3
api_key: {{api_key}}
Authorization: Bearer {{token}}

###
# @name updatePet
# Update an existing pet

PUT http://petstore.swagger.io/v2/pet
Content-Type: application/json
Authorization: Bearer {{token}}

{
  "category": {
    "id": 0,
    "name": "string"
  },
  "id": 2,
  "name": "doggie",
  "photoUrls": [
    "string"
  ],
  "status": "available",
  "tags": [
    {
      "id": 2,
      "name": "Fido"
    }
  ]
}

###
# @name uploadFile
# uploads an image

POST http://petstore.swagger.io/v2/pet/0/uploadImage
Authorization: Bearer {{token}}
Content-Type: application/octet-stream

< ./image/test.jpg

```
</details>

Typically I would name the file `requests.http`, but you can of course be more descriptive or break things down across different files (for example `internal-api.http` and `graphql-api.http`).


## Clients, tools and extensions

There are many tools and clients for working with these kinds of files. They will let you actually send the http requests and show you the responses (some with with metadata about response times, latency etc). Someone has probably implemented this in your favourite language or editor, but here are a few examples:


| Tool | Best For |
|------|----------|
| JetBrains [http client plugin](https://www.jetbrains.com/help/idea/http-client-in-product-code-editor.html) | Jetbrains ecosystem |
| VS Code  [REST Client](https://marketplace.visualstudio.com/items?itemName=humao.rest-client) extension | VSCode users (there are many other VSCode extensions) |
| [httpyac](https://httpyac.github.io) | Terminal use. Scripting and CI/CD |
| [Resterm](https://github.com/unkn0wn-root/resterm/) | A great TUI version |


I personally use `httpyac` the most, I like that it is scriptable and usable from the command line so I can pipe and chain requests. I often wire up whichever requests I'm using most to a task in [Zed](https://dev.karltryggvason.com/zed/) so that I'm able to fire off requests from my editor when I'm debugging or implementing new endpoints.

## Documentation and code reviews

Testing new features becomes easier with `http` files and they are good to refer to in code reviews. You can both document how a feature or fix works and make it easy for a reviewer to test it. So you could write something like:

> Use the `createTicket` request to create a ticket, and the `getTicket` request to verify it was successfully created

This reads better than inlining curl commands with long urls and unreadable json and makes it easy for your reviewer (and your future self) to understand what functionality you were adding or fixing.

## Environments

Many clients have support for defining different environments, so you can switch between making requests to your local dev env, staging or prod (this also allows you to keep sensitive information, api keys and tokens, out of version controlled files using `.gitignore`). This is achieved by defining your variables in `.env` files and then selecting which environment to run against in your client or tool.

If your service has seed data, you can use variables and values from that in local dev to link things up and make your requests work "out of the box". As soon as you spin up a developer environment you have realistic data in the database and requests to interact with it through your api. This makes it easy for new developers to get up and running and kick the tires. The upfront time spent on setting this up will pay for itself many times over.

## Testing

For clients that have [assertions](https://httpyac.github.io/guide/assert.html#assert-syntax) for responses, you can use the `.http` files to build and run high level integration tests. I have tests like this set up for [*In the Long Run*](http://inthelongrun.app), verifying that features and user flows work as intended, checking integrations and avoiding regressions. Here's what requests in a test scenario might look like: 

```sh
###
# @name getExistingPet200s
GET https://petstore.swagger.io/v2/pet/4
?? status = 200

###
# @name getNonExistingPet404s
GET https://petstore.swagger.io/v2/pet/0
?? status = 404
```

Each request in these tests files will say which status code it expects and/or will assert something about the value of a given attribute of the response. I use simple scenarios (create a thing, retrieve it, delete it etc) and run these locally (in my pre-push hook), in CI/CD pipelines and against production to catch and identify problems. I typically run the requests sequentially in quiet mode bailing on any failed assertions.

## Debugging external apis 

For some projects I also have `.http` files for http apis that my service consumes. This can be helpful if a generated client or sdk is acting in unexpected ways. You can go straight to the source and see responses and headers without having to dive deep into your client code for debugging breakpoints or print statements. For instance I've found this valuable when debugging rate request limits and redirects.

## "Why not?" - Alternatives and objections

Can't people just use `curl` or `httpie` you might ask? And of course they can (some of these tools even offer a "*copy request to curl*" command) but ephemeral snippets or long bash scripts aren't as readable or good for long term documentation.

"*But I have an OpenAPI/graphql spec*" is another refrain. And you should, but an `.http` file is a valuable addition. Specs are written for machines, they are hard to read for people. Opening a spec and clicking through to requests in Swagger UI or the like takes way more work than a few terminal keystrokes or a single click in your editor. Thus having both the spec and a requests file and keeping it up to date is valuable. Admittedly which is the source of truth can become a point of tension, as it is easy for the two to slip out of sync. I've built a [small tool to import OpenAPI example requests into `.http` files](https://github.com/Kalli/openapi-http) to help keep my http files synced up with my specs.

There are popular proprietary tools named after not being able to sleep and people who deliver letters that feature similar functionality which many swear by. But in my opinion you want to go portable, local, open and readable, all of which our humble `http` file achieves.

Colleagues or collaborators might require persuasion. But even if they don't see the benefits, who objects to a simple text file? The fact that you aren't tied to one tool or implementation is a selling point in this respect too, the files can easily accommodate different editors, tools and ways of working.

Some clients extend the "language", add features and functionality with syntax that differs slightly between clients. Though these features can be neat, this is unfortunate, especially if your colleagues or collaborators use different clients with different implementations. I wish we could consolidate this somehow into a standard. But I believe common standards have emerged for variable support and environments which are the basic building blocks you need.

## `GET` started

Add a `requests.http` file to your project, stick the 5 most commonly used requests in there. Is it a silver bullet? No. Complex auth flows are tricky, stateful sequences can strain the format, maybe your colleagues will require hand holding. But it is helpful tooling with a very gentle learning curve that will get you a long way.
