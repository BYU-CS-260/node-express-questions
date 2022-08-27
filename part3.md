# Part 3 - Answers

Now we are going to go back to `server.js` and provide functions for getting, adding, and deleting answers to questions.

## Storing answers

First we need a way to store the answers for each question. We are going to do this with an object:

```js
let answers = {};
```

I put this code right under the questions variable:

```js
let questions = [];
let answers = {};
```

In this object, we want to map the question ID to a list of answers for that question. Something like this:

```js
{ 
  questionID1: [ answer1, answer2],
  questionID2: [ answer3, answer4]
}
```

Here is a more detailed example, with real data:

```js
{
  '0e12706f-93c9-48f8-9d6e-df2b3569dfde': [
    {
      id: '9200a3a4-5d52-4dbc-9ee0-a8f283c31932',
      text: 'Can you provide more details?'
    },
    {
      id: '126bf1a0-33fc-4c59-8af8-b81fb8ee619b',
      text: "I'm not sure what you mean"
    }
  ]
}
]
```

## Getting answers

First, let's add an endpoint to get a list of all the answers for a question:

```js
// get a list of all answers for a question
app.get("/api/questions/:questionID/answers", (req, res) => {
  const questionID = req.params.questionID;
  if (questionID in answers) {
    res.send(answers[questionID]);
    return;
  }
  res.send([]);
});
```

We use the `:questionID` parameter to allow the client to tell us _which_ question they want answers for. If that `questionID` is in the `answers` object, we can send back that list. Otherwise we return an empty list.

## Adding an answer

Now we can an endpoint for creating an answer to a question:

```js
// create an answer for a question
app.post("/api/questions/:questionID/answers", (req, res) => {
  const questionID = req.params.questionID;

  answerID = crypto.randomUUID();
  let answer = {
    id: answerID,
    text: req.body.text,
  };
  if (!(questionID in answers)) {
    answers[questionID] = [];
  }
  answers[questionID].push(answer);
  res.send(answer);
});
```

Notice how we use `crypto.randomUUID()` again to create a unique ID for each answer. We also have to initialize an empty array if this is the first time this question has been answered. We know it hasn't been answered yet if `questionID` is not in the `answers` object.

## Deleting an answer

Finally, we can add an endpoint to handle deleting an answer to a question:

```js
// delete an answer for a question
app.delete("/api/questions/:questionID/answers/:answerID", (req, res) => {
  const questionID = req.params.questionID;
  const answerID = req.params.answerID;
  if (!questionID in answers) {
    res.sendStatus(404).send("That answer does not exist.");
    return;
  }
  answers[questionID] = answers[questionID].filter(
    (answer) => answer.id != answerID
  );
  res.sendStatus(200);
});
```

This endpoint uses _two_ parameters, one for the ID of the question and one for the ID of the answer -- `questionID` and `answerID`. They will each be accessible in `req.params`.

We send back a 404 response if the question doesn't exist. We can't possibly delete an answer to the question in that case.

Just like when we deleted a specific question, we can use the `filter()` higher-order function to delete an answer. 

*Note: We do not worry about sending a 404 error if the answer didn't actually exist. Technically, we could check for that case too.*

Join me, adventurers, in [part 4](./part4.md)