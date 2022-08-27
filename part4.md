# Part 4 - More Testing

We are going to add just one function, to show you that we can do testing of GET, POST, and DELETE all at once:

```js
describe("answers", () => {
  let addedQuestionID = 0;
  let addedAnswerID = 0;
  beforeAll(async () => {
    // before testing posting an answer, we need to post a question and keep track of its ID
    const newQuestion = {
      text: "Can you help me?",
    };
    const response = await axios.post(`${baseURL}/api/questions`, newQuestion);
    addedQuestionID = response.data.id;
  });
  afterAll(async () => {
    // after we are done testing, we have to delete the question
    await axios.delete(
      `${baseURL}/api/questions/${addedQuestionID}/answers/${addedAnswerID}`
    );
  });
  test("adds an answer", async () => {
    // create a new answer and post it
    const newAnswer = {
      text: "Can you provide more details?",
    };
    const response = await axios.post(
      `${baseURL}/api/questions/${addedQuestionID}/answers`,
      newAnswer
    );
    addedAnswerID = response.data.id;
    expect(response.status).toBe(200);
    expect(response.data.text).toEqual("Can you provide more details?");
  });
  test("gets an answer", async () => {
    const response = await axios.get(
      `${baseURL}/api/questions/${addedQuestionID}/answers`
    );
    expect(response.status).toBe(200);
    expect(response.data[0].text).toEqual("Can you provide more details?");
  });
  test("deletes an answer", async () => {
    const response = await axios.delete(
      `${baseURL}/api/questions/${addedQuestionID}`
    );
    expect(response.status).toBe(200);
  });
});
```

We use `beforeAll()` to create a question and `afterAll()` to delete that same question. These run before and then after all the tests.

We have three tests in the function, one that creates an answer to the question, one that gets the answer to the question, and one that deletes the answer to the question.

Notice in the second test, when we get the answers, the response is a _list_ of answers. We are expecting our answer to be the first one in the list, since we just barely created the question and answered it during our test.

## Important point

We have written and tested the entire back end before writing any front end code. This is a great way to write an application. Once your back end is written and tested, you can write your front end, confident that any calls you make to the API will work. 

We have given you a front end in [part 5](./part5.md).