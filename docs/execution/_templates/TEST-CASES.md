# <Module> test cases

Every test written for this module, how it was tested and the latest result. Update it in the same commit as the tests. One row per test case; link the ticket it proves.

| ID | Ticket | What is tested | Type | How it was tested (steps / input) | Expected result | Test file | Result (date) |
| -- | ------ | -------------- | ---- | --------------------------------- | --------------- | --------- | ------------- |
| <PREFIX>-TC-001 | <PREFIX>-NNN | <behaviour, rule or error case> | unit / integration / e2e / manual | <the input sent or the steps clicked> | <status code, exact shared message, what the UI shows> | `<path>` | pass / fail (YYYY-MM-DD) |

Types:
- **unit**: Jest, one function or schema (`shared/`, backend services, frontend components).
- **integration**: Jest + supertest against the API and the `_test` database.
- **e2e**: Playwright against the running app.
- **manual**: checked by hand (for example Figma screenshot comparison); say what was compared.

## Validation parity

For every form field rule: the same invalid input, the message from `shared/`, and the three places it was checked.

| Field | Invalid input | Shared message key | Schema test (shared) | API test (`error.fields`) | UI test (on screen) |
| ----- | ------------- | ------------------ | -------------------- | ------------------------- | ------------------- |

## Not tested

<What could not be tested and why, so nobody assumes it was.>
