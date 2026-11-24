# HW1 AI Use-Case Memo: Triage of Failed Autotests

## 1 Describe the situation

I work as a QA automation engineer in a bank website team

Our regression suite has 994 autotests (Python, Playwright, pytest, Allure)

After each run some tests fail. A QA engineer opens the Allure report and checks every failure by hand: error text, screenshot, logs

Then the engineer decides what it is: a real bug, a flaky test, a test data problem or an environment problem. For real bugs the engineer writes a bug report in Jira

This is slow and repetitive. Many failures have the same cause, but they are still checked one by one

## 2 Split the task

| Layer | What it does |
|---|---|
| 1.0 Code | Reads Allure results, groups failures with the same error, reruns each failed test once, checks that stage is up, masks personal data in logs |
| 2.0 ML | A classifier trained on past triage decisions. Returns a class (bug, flaky, test data, environment) and a probability in milliseconds |
| 3.0 LLM | Works only for the "bug" class. Writes a short draft bug report: title, steps, expected, actual |

## 3 Inputs and output

**Input**

1. Allure results JSON of the current run
2. Results of the same tests in the last 10 runs
3. Masked logs without customer data

**Output:** one JSON object per group

1. List of tests in the group
2. Class and probability
3. Reason in one sentence
4. Draft bug report (only for "bug")

Code checks the JSON with Pydantic. An invalid answer goes to manual triage

## 4 Control and taboo

**Human approves**

1. QA confirms or changes the class of every group
2. A bug report goes to Jira only after QA clicks "Create"
3. Probability below 0.8 means manual check

**The model must never**

1. Create, close or edit Jira tickets
2. Run anything on production or change test data
3. Skip, disable or delete tests
4. See customer data, OTP codes, passwords or tokens
5. Follow instructions found inside logs or page text

**Result:** QA time goes to real bugs, not to sorting failures
