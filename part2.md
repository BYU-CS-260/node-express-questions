# Part 2 - Testing

We are going to use [Jest](https://jestjs.io/) for testing.  The [documentation for Jest]() shows a great example:

If you have a file called `sum.test` with the following code:

```js
function sum(a, b) {
  return a + b;
}
module.exports = sum;
```

Then you can create a file named `sum.test.js` with this basic test:

```js
const sum = require('./sum');

test('adds 1 + 2 to equal 3', () => {
  expect(sum(1, 2)).toBe(3);
});
```

The `test` function takes as its second argument a function. This second function has one line:

```js
expect(sum(1, 2)).toBe(3);
```

It expects the sum of 1 and 2 to be equal to 3.

Then all you have to do is add the following section to your `package.json`:

```js
{
  "scripts": {
    "test": "jest"
  }
}
```

Finally, if you run `npm test`, then Jest will print this message:

```
PASS  ./sum.test.js
✓ adds 1 + 2 to equal 3 (5ms)
```


## Writing your first test

Begin by installing jest:

```sh
npm install jest
```

Now create a file called `server.test.js`:

```js
const axios = require("axios");

const baseURL = "http://localhost:3030";

describe("get empty questions", () => {
  test("should return 200", async () => {
    // Request all questions. This should be empty if we started the dev server from scratch.
    const response = await axios.get(`${baseURL}/api/questions`);
    expect(response.status).toBe(200);
    expect(response.data).toEqual([]);
  });
});
```

This simple test checks the `/api/questions` endpoint. Specifically, it checks whether the server returns the 200 OK response and whether the body of the response contains an empty array. 

There are a few new things here:

* We are using `describe()` to _scope_ our tests. This will be useful with later tests we write.

* We use `toBe` in the first expect and `toEqual` in the second one. [This is a good article explaining the difference.](https://dev.to/thejaredwilcurt/why-you-should-never-use-tobe-in-jest-48ca#:~:text=toEqual%20and%20the%20.,equality%20fails%20on%20non%2Dprimatives.) You are safe using `toBe` for number comparisons (like a status code), but should use `toEqual` when you are comparing objects.

Now modify `package.json` so it includes:

```js
  "scripts": {
    "test": "jest"
  }
```

Mine looks like this:

```js
{
  "dependencies": {
    "axios": "^0.27.2",
    "body-parser": "^1.20.0",
    "express": "^4.18.1",
    "jest": "^28.1.3"
  },
  "scripts": {
    "test": "jest"
  }
}
```

To run the tests you need to have the server running, so you will need two terminals. In one terminal, 
run the server with `node server.js`. In the other terminal, in the same directory, run `npm run test`:

```sh
% npm run test

> test
> jest

 PASS  ./server.test.js
  get empty questions
    ✓ should return 200 (13 ms)

Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        0.193 s
Ran all test suites.
```

## Testing a POST endpoint

Now we want to test the POST endpoint to see if we can add a question.

Here is an example of how to do that:

```js
describe("post a question", () => {
  // Create a new question
  const newQuestion = {
    text: "Can you help me?",
  };

  test("posts a question", async () => {
    // post the question
    const response = await axios.post(`${baseURL}/api/questions`, newQuestion);
    expect(response.status).toBe(200);
    expect(response.data.text).toEqual("Can you help me?");
  });
});
```

Notice how we call `axios.post()` to call the `/api/questions` endpoint with a POST request. We then check whether the status code is 200 and whether the body of the response includes the text of the question we asked.

BUT!! After we do this, our server is now "polluted" with a test question. We should clean up after ourselves and delete the question we added, since we are just testing. We can do this with `afterAll()`, which runs after all tests in the `describe()` function. Here is our revised test:

```js
describe("post a question", () => {
  // Create a new question
  const newQuestion = {
    text: "Can you help me?",
  };
  // store the added question's ID here, eventually
  let addedQuestionID = 0;
  afterAll(async () => {
    // after we are done with all tests, delete the added question
    await axios.delete(`${baseURL}/api/questions/${addedQuestionID}`);
  });

  test("posts a question", async () => {
    // post the question
    const response = await axios.post(`${baseURL}/api/questions`, newQuestion);
    // keep track of the added question's ID
    addedQuestionID = response.data.id;
    expect(response.status).toBe(200);
    expect(response.data.text).toEqual("Can you help me?");
  });
});
```

Notice that we keep track of the ID of the question we added using `addedQuestionID`. Then in `afterAll()`, we can use this variable to send a DELETE request to the server.

This is what the `describe()` function is useful for. We wouldn't want to call this delete endpoint after every test. Just this test.

*Note: Yes, technically, we are using delete before it has been tested. There is not a good way around this.*

If you save `server.test.js` then you can re-run `npm run test`:

```sh
% npm run test

> test
> jest

 PASS  ./server.test.js
  get empty questions
    ✓ should return 200 (14 ms)
  post a question
    ✓ posts a question (5 ms)

Test Suites: 1 passed, 1 total
Tests:       2 passed, 2 total
Snapshots:   0 total
Time:        0.207 s, estimated 1 s
Ran all test suites.
```

Hopefully you are feeling great right now. We have *two* tests passing. We can re-run the tests as often as we want because we are cleaning up after ourselves. We can be *sure* our code is working.

![young kid, a hockey fan, saying I feel amazing](images/i-feel-amazing.gif)

### Testing a DELETE endpoint

We want to test DELETE. But we can't do that unless we add something to the database first. Say welcome to `beforeAll()`:

```js
describe("delete a question", () => {
  let addedQuestionID = 0;
  beforeAll(async () => {
    // before we test deleting, we need to add a question and keep track of its ID
    const newQuestion = {
      text: "Can you help me?",
    };
    const response = await axios.post(`${baseURL}/api/questions`, newQuestion);
    addedQuestionID = response.data.id;
  });
  test("deletes a question", async () => {
    // delete the added question
    const deleteResponse = await axios.delete(
      `${baseURL}/api/questions/${addedQuestionID}`
    );
    expect(deleteResponse.status).toBe(200);
  });
});
```

Notice how we add a question in `beforeAll()` and then delete it and check for a 200 status code in the `test()`. We use the variable `addedQuestionID` to keep track of the question we added, so we can delete that exact question in our test.

We can run `npm run test` again:

```sh
 npm run test

> test
> jest

 PASS  ./server.test.js
  get empty questions
    ✓ should return 200 (12 ms)
  post a question
    ✓ posts a question (5 ms)
  delete a question
    ✓ deletes a question (1 ms)

Test Suites: 1 passed, 1 total
Tests:       3 passed, 3 total
Snapshots:   0 total
Time:        0.18 s, estimated 1 s
Ran all test suites.
```

### Testing GET again

We only tested GET with an empty list of questions. What if it's not working with multiple questions? We better test it:

```js
describe("get multiple questions", () => {
  let addedQuestionIDs = [];
  beforeAll(async () => {
    // before we test getting multiple questions, we have to add multiple questions and keep track of their IDS
    const newQuestion = {
      text: "Can you help me?",
    };
    let response = await axios.post(`${baseURL}/api/questions`, newQuestion);
    addedQuestionIDs.push(response.data.id);
    response = await axios.post(`${baseURL}/api/questions`, newQuestion);
    addedQuestionIDs.push(response.data.id);
  });
  afterAll(() => {
    // after we test getting, we need to delete all those questions
    // forEach is a higher order function that works like a for loop
    addedQuestionIDs.forEach(
      async (addedQuestionID) =>
        await axios.delete(`${baseURL}/api/questions/${addedQuestionID}`)
    );
  });
  test("should return 2 questions", async () => {
    // get all questions
    const response = await axios.get(`${baseURL}/api/questions`);
    expect(response.status).toBe(200);
    expect(response.data.length).toEqual(2);
  });
});
```

Whew, we are using both `beforeAll()` and `afterAll()`. We keep track of a _list_ of question IDs in `addedQuestionIDs`, so that we can be sure to delete them all later on. When we delete questions in `afterAll()` we use the higher-order function `forEach` to loop through all the questions.

In our test, we use `data.length` to check that we got a total of 2 questions in our response.

Once more, we can run `npm run test`:

npm run test

```sh
> test
> jest

 PASS  ./server.test.js
  get empty questions
    ✓ should return 200 (14 ms)
  post a question
    ✓ posts a question (5 ms)
  delete a question
    ✓ deletes a question (2 ms)
  get multiple questions
    ✓ should return 2 questions (1 ms)

Test Suites: 1 passed, 1 total
Tests:       4 passed, 4 total
Snapshots:   0 total
Time:        0.219 s, estimated 1 s
Ran all test suites.
```

![a woman giving herself a high five](./images/high-five-self.gif)

You're ready to move on to [part 3](./part3.md).