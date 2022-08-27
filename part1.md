# Part 1 - Questions

We are going to build the back end for a question-and-answer website. We'll start by building a server that allows clients to add, post, and delete comments.

## Initialization

The first step is to initialize a new project. You do
this with `npm init`.

```sh
mkdir lesson2
cd lesson2
mkdir back-end
cd back-end
npm init
```

Like the first lesson, we'll answer these questions:

```sh
package name: (lesson2)
version: (1.0.0)
description: help ticket server
entry point: (index.js)
test command:
git repository:
keywords:
author:
license: (ISC)
```

Now we need to install Express and an npm library for parsing the body of POST requests.

```sh
npm install express body-parser
```

## Basic code

Create a file called `server.js` with the following code:

```js
const express = require('express');
const bodyParser = require("body-parser");
const crypto = require("crypto");


const app = express();
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: false }));
```

This includes the modules we're using and initializes them.

Now add this line:

```js
app.listen(3030, () => console.log("Server listening on port 3030!"));
```

This starts the server on port 3000.


You're welcome to run the server at this point:

```sh
node server.js
```

However, since we haven't added any endpoints, it won't do very much!

## Getting questions

We want to be able to get all the questions in the system. This is pretty easy!

> **Note**
> Be sure to add all the rest of your code _above_ the `app.listen()` line. Once this line is executed, the server begins listening for requests and doesn't run any lines below that.


We want to be able to keep track of questions, so we will just store them in an array:

```js
let questions = [];
```

Now we can write an endpoint to get all questions:

```js
app.get("/api/questions", (req, res) => {
  res.send(questions);
});
```

## Creating a question

We should be able to create a new question. You can do this with the following endpoint:

```js
app.post("/api/questions", (req, res) => {
  const id = crypto.randomUUID();
  let question = {
    id: id,
    text: req.body.text,
  };
  questions.push(question);
  res.send(question);
});
```

In the first line, we use `app.post()` because we want to handle POST requests from clients. We can use the same URL as the `app.get()` above, and Express knows to send these to the different functions we define.

Notice that in the second line we are using `crypto.randomUUID()`. This generates a long, random identifier so we have a unique `id` field for each question. The random identifiers look like this:

```js
'0e12706f-93c9-48f8-9d6e-df2b3569dfde'
```

Having a unique identifier is really helpful when we want to either edit or delete a specific question.

After this we just create a question object, with two fields, `id` and `text`, and push it into the array. We send back a `200 OK` response with the question object that we created. This is standard practice for a POST request.

Here is what the list of questions is going to look like after we add a few:

```js
[
  {
    id: '12c8d0a9-2115-44ea-9aaa-0e81a5e126db',
    text: 'Can you help me?'
  },
  {
    id: '422ce04e-acc1-4862-9a98-a31839e410f0',
    text: 'What time is it?'
  }
]
```

## Deleting questions

We need to be able to delete questions, too. Normally this would be restricted to the person who created it, because they are the "owner" of the question. We aren't going to create users quite yet, so we will just let anyone delete it.

```js
app.delete("/api/questions/:id", (req, res) => {
  const id = req.params.id;
  questions = questions.filter((question) => question.id != id);
  res.sendStatus(200);
});
```

Here, we use `app.delete()` to handle a DELETE request. The URL uses `:id` to indicate that the client, when they call this endpoint, will supply an identifier of one of the questions. These are the identifiers created `crypto.randomUUID()`.

On the second line, we can get this ID from the URL using `req.params.id`. You can use any variable name you want here. If you used `:questionID` in the URL, you would use `req.params.questionID`.

On the third line, we are using a higher-order function, called `filter()` to delete the question that has a matching ID. 

*Note: We do not worry about sending a 404 error if the question didn't actually exist. Technically, we could check for that case.*

## Testing with curl

As we've done previously, you can test this with curl. For example:

```sh
curl localhost:3030/api/questions
```

However, this gets tedious to keep typing things in the terminal. 

Go to [part2](./part2.md) to learn how to test this with Jest.

