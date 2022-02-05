## API Design - choosing status code for a GET route

# Context

You need to design some routes with the following properties:

- you need to do some computation in the backend
- they are idempotent: every time you call the route with the same params it should give you the same result and nothing should change in the state o the application. Eg. no records are created, updated, deleted ... 
- they return a collection of data as response

# Examples

An endpoint that will for a book returns similar books according to a predefined formula. 

```
/books/:id/similar
``` 

An endpoint that will first select some resources and filter them based on some criteria

```
/books/drafts?language=en
```

Or an endpoint that accepts various filtering parameters and returns a collection of resources

```
/books?status=published&language=en&author=12
```

# Question

In all these cases the main question is: 

> What should the response be in case there are no resources found?

or more general 

> How to choose what status to return?

# Solution

## Choosing the HTTP verb

First, I should start by saying that the HTTP verb to be used for such routes is **GET**.  
For me, it feels like the normal choice, here is why:

- The other possible verb could be POST, but POST is mostly used for situations where the request will change the application state (eg: will create a new resource ...)
- JSON:API [spec](https://jsonapi.org/format/#fetching) also says: 

> Data, including resources and relationships, can be fetched by sending a GET request to an endpoint.

## Status to return

Choosing a status is dependent on what operation one is doing there. 
I think it is better to explain this by taking one of the examples above: 

```
/books/:id/similar
```

In this case, response status should be:

- `200` (ok) if any similar book is found
- `200` (ok) with an empty array if no similar book is found
- `401` (not authorized) is the endpoint that requires authorization it was not provided or invalid
- `422` (unprocessable entity) if the route accepts some query params and they are not valid
- `404` (not_found) if book id does not exist or is not allowed to be queried

I think from all the status responses, one that is worth discussing a little bit is why return `200` (ok) when no results are found and why not return for example `404` (not found)?

The reasons for choosing 200 are:

First, the JSON:API spec defines that in case of fetching a collection of resources the response should be 200, and in case of empty should just include an empty array/data:

> A server MUST respond to a successful request to fetch a resource collection with an array of resource objects or an empty array ([]) as the response document’s primary data.

Second, thinking about this the request is valid (give me similar books with this book), and also the empty response is a valid response (there are no similar books) and not an error response. An error response will be like saying "before querying this endpoint you should know if a book has or has not similar books" but this is exactly the purpose of this endpoint to answer this question. So asking the question and answering with the empty array is a valid response for this question. 

This, let's try to analyze this status response through the difference between this route and one like `books/:id`. In the case of `books/:id` if there is no book with the provided id it is expected to return 404 not only because the JSON:API [spec is describing](https://jsonapi.org/format/#fetching-resources-responses-404) very clearly this situation but also because if the queries id does not exist it is like the URL `books/:id` does not exist so it is normal to respond with 404. But if the case of `books/:id/similar` if there are no similar books then the URL is still valid in the sense that it is still valid to ask the question *"Are there any similar books for this specific book id?"*

## Security considerations

When choosing a status you should also make sure that by choosing that status you will not reveal a piece of information that you did not intend.

Here are is an example of what NOT to do:

Responding with `409` (conflict) when someone tries to GET /books/:id/similar for a book that exists but is not yet published or active.  It could be tempting to respond with `409` as somehow the query is valid but the state of the book is not allowing this request to be processed by doing so you will communicate also that that book id exists in your system even if the API user cannot access it right now. This is a piece of extra information that could allow an attacker to understand what books are not yet published for example. 













