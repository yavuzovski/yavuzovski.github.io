---
title: problems-with-the-rest
date: 2018-06-19 17:00:12
categories: [backend, graphql, rest]
tags: [backend, graphql, rest]
---

In this post, I will try to show you the problems with the REST architecture.
After that, in another post which will be the second part of this post,
I will try to explain GraphQL architecture which will be a solution to the problems we will see.
Moreover I will create a simple NodeJS app with GraphQL architecture which will be
able to do CRUD operations.

### Prolog

Actually, I tried to explain GraphQL in this post but the writings on
which problems GraphQL was solving the REST had were too long, so I decided to separate this part
as another post. I won't define REST, there are lots of posts on the internet for it already. I will
try to show the parts REST architecture is missing from a front-end developer's point of view on an example
application.

### An Example Application

For our example application, we will use [JSONPlaceholder] REST API.
Let's say that we have a blog. In the main page of this blog, we want to list
all blog posts and the information of who sent those posts.

To achieve this, the endpoints we need are these, and here their responses are:

- [/posts]
``` json
[
    {
      "userId": 1,
      "id": 1,
      "title": "sunt aut facere repellat provident occaecati excepturi optio reprehenderit",
      "body": "quia et suscipit\nsuscipit recusandae consequuntur expedita et cum\nreprehenderit molestiae ut ut quas totam\nnostrum rerum est autem sunt rem eveniet architecto"
    },
    ...
]
```
- [/users/:userId]
``` json
{
    "id": 1,
    "name": "Leanne Graham",
    "username": "Bret",
    "email": "Sincere@april.biz",
    "address": {
      "street": "Kulas Light",
      "suite": "Apt. 556",
      "city": "Gwenborough",
      "zipcode": "92998-3874",
      "geo": {
        "lat": "-37.3159",
        "lng": "81.1496"
      }
    },
    "phone": "1-770-736-8031 x56442",
    "website": "hildegard.org",
    "company": {
      "name": "Romaguera-Crona",
      "catchPhrase": "Multi-layered client-server neural-net",
      "bs": "harness real-time e-markets"
    }
}
```
You can navigate to the links to see all responses.

### RESTful Approaches and Their Problems

Things we have to do to list posts on the main page are: First we have to
gather posts from the `/posts` endpoint.

``` json
[
  {
    "userId": 1,
    "id": 1,
    "title": "sunt aut facere repellat provident occaecati excepturi optio reprehenderit",
    "body": "quia et suscipit\nsuscipit recusandae consequuntur expedita et cum\nreprehenderit molestiae ut ut quas totam\nnostrum rerum est autem sunt rem eveniet architecto"
  },
  ...
]
```

After that, we must gather users by using `userId` value from `/users/:userId` endpoint for
each post. So the response we will get for the first post from [/users/1] endpoint is:

``` json
{
  "id": 1,
  "name": "Leanne Graham",
  "username": "Bret",
  "email": "Sincere@april.biz",
  "address": {
    "street": "Kulas Light",
    "suite": "Apt. 556",
    "city": "Gwenborough",
    "zipcode": "92998-3874",
    "geo": {
      "lat": "-37.3159",
      "lng": "81.1496"
    }
  },
  "phone": "1-770-736-8031 x56442",
  "website": "hildegard.org",
  "company": {
    "name": "Romaguera-Crona",
    "catchPhrase": "Multi-layered client-server neural-net",
    "bs": "harness real-time e-markets"
  }
}
```
We will have all the data we need after gathering users for all posts. Then we can
create the structure we want by putting together `title` and `body` values from posts and `username`
value from users. So the structure of the result we want to have will be like this:

``` json
[
  {
    "title": "sunt aut facere repellat provident occaecati excepturi optio reprehenderit",
    "body": "quia et suscipit\nsuscipit recusandae consequuntur expedita et cum\nreprehenderit molestiae ut ut quas totam\nnostrum rerum est autem sunt rem eveniet architecto",
    "user": {
      "username": "Bret"
     }
  },
  ...
]
```
In this REST approach, there are two big problems which GraphQL solves.
First of those is [over-fetching] problem. That means we have more data
than we need in our responses. We just need `username` value from `/users/:userId`
endpoint but we have lots of another values in our response. Let's take a look at the
response we have again.
``` json
{
  "id": 1,
  "name": "Leanne Graham",
  "username": "Bret",
  "email": "Sincere@april.biz",
  "address": {
    "street": "Kulas Light",
    "suite": "Apt. 556",
    "city": "Gwenborough",
    "zipcode": "92998-3874",
    "geo": {
      "lat": "-37.3159",
      "lng": "81.1496"
    }
  },
  "phone": "1-770-736-8031 x56442",
  "website": "hildegard.org",
  "company": {
    "name": "Romaguera-Crona",
    "catchPhrase": "Multi-layered client-server neural-net",
    "bs": "harness real-time e-markets"
  }
}
```

Another problem is [under-fetching] problem. That means we have to call another endpoint to
gather the data we need. In our example, we sent `1` request to `/posts` endpoint to gather posts.
After that, we had to send `n` requests to `/users/:userId` endpoint for `n` posts we got.
This problem is also known as `n + 1` problem.

### Trying to Solve the Problems Again With REST

So can't we solve those problems with RESTful approach? We can, but those solutions also have
another costs. Some of those solutions are:

1. Modifying the current endpoint responses.

   - Let's say we asked to the back-end developer to modify `/users/:userId` endpoint to
   return just the `username` value for getting rid of the over-fetching problem.
      ``` json
      {
        "username": "Bret"
      }
      ```
    With this solution, now we won't have unnecessary data in our responses. Let's go back to our example app.
    Some time has been passed and there is a change on the main page. Now we want to display `email` values
    instead of `username` under the posts but `/users/:userId` endpoint doesn't contain `email` value anymore.
    Now we have to ring the bell of our beloved back-end developer to access `email` value again.

    - To solve the under-fetching problem, let's ask to the back-end developer to modify `/posts` endpoint to
    return `username` value of the users which have the `userId` instead of the `userId` value. So the exact
    structure we need will return:
      ``` json
      [
        {
          "title": "sunt aut facere repellat provident occaecati excepturi optio reprehenderit",
          "body": "quia et suscipit\nsuscipit recusandae consequuntur expedita et cum\nreprehenderit molestiae ut ut quas totam\nnostrum rerum est autem sunt rem eveniet architecto",
          "user": {
            "username": "Bret"
          }
        },
        ...
      ]
      ```
    With this solution, we will gather `username` value too with just one request and under-fetching problem will be solved.
    Actually, this solution also solves the over-fetching problem too. But again, this solution also has costs it brings.
      
      Let's go back to our app again. Another change also made and now we want to list the amount of posts sent based on cities.
      To achieve this, the roadmap based on our old endpoints is this:
      Gather posts from `/posts` endpoint. Then gather `city` value for each post from `/users/:userId` endpoint
      by using `userId` value of the posts. But after the changes we made, `/posts` endpoint returns `username` instead of `userId`
      value! Because of that, we must ask back-end developer to modify `/posts` endpoint again to return `city` value too
      ``` json
      [
        {
          "title": "sunt aut facere repellat provident occaecati excepturi optio reprehenderit",
          "body": "quia et suscipit\nsuscipit recusandae consequuntur expedita et cum\nreprehenderit molestiae ut ut quas totam\nnostrum rerum est autem sunt rem eveniet architecto",
          "user": {
            "username": "Bret",
            "address": {
              "city": "Gwenborough" 
            }
          }
        },
        ...
      ]
      ```
      or return `userId` value again.
      ``` json
      [
        {
          "userId": 1,
          "title": "sunt aut facere repellat provident occaecati excepturi optio reprehenderit",
          "body": "quia et suscipit\nsuscipit recusandae consequuntur expedita et cum\nreprehenderit molestiae ut ut quas totam\nnostrum rerum est autem sunt rem eveniet architecto",
          "user": {
            "username": "Bret"
          }
        },
        ...
      ]
      ```

2. Creating new endpoints according to the need.

    - Let's go back to our main page example. Instead of modifying `/posts` and `/users/:userId` endpoints, let's
    ask for a new endpoint from the back-end developer. For example: `/posts/usernames`. And response of this endpoint
    is like this:
      ``` json
      [
        {
          "title": "sunt aut facere repellat provident occaecati excepturi optio reprehenderit",
          "body": "quia et suscipit\nsuscipit recusandae consequuntur expedita et cum\nreprehenderit molestiae ut ut quas totam\nnostrum rerum est autem sunt rem eveniet architecto",
          "user": {
            "username": "Bret"
          }
        },
        ...
      ]
      ```
    
    So the exact structure we want to achieve. With this solution, both we didn't break any components that dependent to `/posts` or
    `/users/:userId` endpoints and we overcome over-fetching and under-fetching problems. Sounds like a miracle, right?

      Well, not exactly. Our beloved back-end developer had to create this endpoint for us. And if we had to develop something
      dependent to that endpoint, we would be blocked until our back-end developer creates it. So, the problem here is the solution
      itself actually. A new endpoint means new code to develop and maintain.

3. Giving arguments to the endpoints as `query strings` and describing the response values.

   - Let's go back to our main page example and to our old endpoints again. We can ask back-end developer to modify `/posts`
   endpoint to expect an argument named as `fields`. Also, this argument can be optional and if they are empty, endpoint can work
   as usual. This way, we won't create a new point and also won't break other components that depend on `/posts` endpoint.
   Let's say this is our endpoint and it's response:

      `/posts?fields=title,body,username`
     ``` json
     [
       {
         "title": "sunt aut facere repellat provident occaecati excepturi optio reprehenderit",
         "body": "quia et suscipit\nsuscipit recusandae consequuntur expedita et cum\nreprehenderit molestiae ut ut quas totam\nnostrum rerum est autem sunt rem eveniet architecto",
         "user": {
           "username": "Bret"
         }
       },
       ...
     ]
     ```
      `/posts`
     ``` json
     [
       {
         "userId": 1,
         "id": 1,
         "title": "sunt aut facere repellat provident occaecati excepturi optio reprehenderit",
         "body": "quia et suscipit\nsuscipit recusandae consequuntur expedita et cum\nreprehenderit molestiae ut ut quas totam\nnostrum rerum est autem sunt rem eveniet architecto"
       },
       ...
     ]
     ```
  So which problems we can encounter with this approach? Well, same with the creating new endpoints. Also, the code that will be written
  will be more complex to check query strings.

### Conclusion

As you can see we can solve over-fetching and under-fetching problems with the RESTful approach. But all those solutions
are **driven by the emerging needs at front-end and requires back-end developers to hardcode the solution at back-end.**
**Actually, we have all the data we need, but when we have to derive a new structure from those datas, there isn't a system
that will create those structures dynamically.**Therefore, we will create those derived structures from the resources we
have and ignore over-fetching and under-fetching problems or back-end developers will create hard-coded solutions 
as we need, which prevents front-end and back-end to work separately and therefore it is an inflexible and inefficient solution.
Probably the best solution we can come up with is balancing all those solutions as we need. For specific structures; creating new
endpoints. For less specific ones; enabling optional arguments with query strings. For common structures; ignoring some amount
of over-fetching and under-fetching.

In [part 2], I tried to show how GraphQL solves these problems we talked about.

[JSONPlaceholder]: https://jsonplaceholder.typicode.com/
[/posts]: https://jsonplaceholder.typicode.com/posts
[/users/:userId]: https://jsonplaceholder.typicode.com/users/1
[/users/1]: https://jsonplaceholder.typicode.com/users/1 
[over-fetching]: https://stackoverflow.com/a/44568365
[under-fetching]: https://stackoverflow.com/a/44568365
[part 2]: https://yavuzovski.github.io