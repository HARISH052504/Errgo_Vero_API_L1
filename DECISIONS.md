# Decisions

**Name:** Harish G 

**Date started:** 19/06/2026 

**Date submitted:** 23/06/2026 (Delayed Due to Repository Access Issue)

In two or three sentences, describe your overall approach before getting into specifics.
What did you read first? What did you prioritise, and why?

Before making any changes, I read the README, project structure, and existing tests to understand the expected behaviour. I focused first on getting the project to compile, then fixing failing tests, and finally adding meaningful test coverage. My goal was to address the business-critical issues before making any additional improvements.

---

## 1. Code & Design Decisions

**The codebase includes an `Auditable` abstract class that is not currently used by any
entity. What did you do with it, if anything? Walk through your reasoning — what is the
purpose of the Auditable pattern, what are the tradeoffs of using it versus not, and why
did you make the choice you did?**

Answer:- I kept the Auditable class unchanged. Its purpose is to provide common audit fields such as createdAt and updatedAt that can be shared across multiple entities. Using a shared parent class reduces duplication and keeps the model consistent. Since it was not directly related to the failing tests or requirements, I chose not to modify it.


**`TransactionResponse` is used as the outbound DTO for the API. What changes did you
make to it, if any? Why does the shape of a response DTO matter — and what is the risk
of returning an entity directly from a controller?**

Answer:- I updated TransactionResponse so the API returns a DTO instead of exposing the Transaction entity directly. A DTO allows the API response to be controlled independently from the database model. Returning entities directly can expose internal fields and make future changes more difficult.


**The `BudgetCalculator` requires grouping and sorting data. What data structure or
approach did you choose to implement it? Walk through the alternatives you considered
and why you landed where you did.**

Answer:- I implemented BudgetCalculator using Java Streams. Transactions are grouped by category, total spending is calculated for each category, then the results are sorted in descending order and stored in a LinkedHashMap. I chose this approach because it is concise, readable, and preserves the required ordering.


**Were there any decisions you made that are not covered by the questions above? Describe
the most significant one and your reasoning.**

Answer:- Yes,I added validation for invalid month values in the monthly spend calculation. This prevents requests such as month=13 from being processed and improves the reliability of the API.
 

---

## 2. Bug Fixes & Issues Found

**Describe each problem you found in the codebase. For each one: where was it, how did
you identify it, what did it cause, and how did you fix it?**

Answer:- I found three main issues in the codebase. The monthly spend calculation excluded transactions from the first day of the month, which I fixed by correcting the date comparison. The BudgetCalculator method returned an empty map, so I implemented the required calculation logic. I also completed the unfinished getTransactionsByDateRange() method so it returns the correct results.

**Were there any problems you noticed but chose not to fix? If so, explain why.**

Answer:- Yes,I noticed some areas that could be optimized for better performance, but they were not affecting functionality or test results. Since the assessment focused on fixing bugs and ensuring correctness, I chose not to make those changes.

---

## 3. Testing Decisions

**What tests did you write in `TransactionCandidateTest.java`? For each test, explain
what behaviour it validates and why you chose to cover that behaviour.**

Answer:- I wrote tests for the changes I made, including date range filtering and spending-related calculations. These tests verify that the business logic works correctly and help prevent the same bugs from returning in the future.


**What did you deliberately not test, and why? If you had more time, what would be the
next most important test to add?**

Answer:- I did not add extensive controller or integration tests because the focus was on validating the fixes I implemented. If I had more time, I would add MockMvc tests to verify API requests, responses, and validation behaviour.


**What is the difference between what `TransactionServiceTest` covers and what your
`TransactionCandidateTest` covers? Are they testing the same things?**

Answer:- No, they are not testing the same things. TransactionServiceTest focuses on the existing service-layer business logic, while TransactionCandidateTest focuses on validating the fixes and functionality I added. Together, they provide broader coverage of the application.


---

## 4. AI Tool Usage

AI tool usage is expected and encouraged. Using AI is not cheating — it is a core skill
of modern engineering. What we are evaluating is whether you used it thoughtfully:
whether you understood and verified what it produced, and whether you can recognise
when its output should not be trusted.

**Which AI tools did you use? (e.g. ChatGPT, Claude, GitHub Copilot, Cursor, other)**

Answer:- I used ChatGPT to help analyze bugs, understand test failures, and review implementation ideas during the project.

**Give two or three specific examples of how you used AI on this project. For each:
what did you prompt it with, what did it return, and what did you accept, change, or
reject?**

Answer:- I used AI to investigate the failing monthly spend test, which helped identify the date boundary issue. I also used AI to review the implementation of BudgetCalculator and generate ideas for grouping and sorting transactions. I verified the suggestions against the project requirements before applying them.


**Describe a moment where AI gave you something wrong, incomplete, or subtly misleading.
How did you catch it, and what did you do?**

Answer:- In one case, AI suggested using a repository method that did not exist in the project. I caught this by checking the actual repository interface and adjusted the solution to match the existing codebase.

**What is your general philosophy on using AI when writing backend code? Where does it
help, and where do you not trust it?**

Anwer:- I use AI as a tool to speed up analysis, debugging, and learning. However, I do not trust its suggestions blindly and always verify them through the code, tests, and project requirements before making changes.


---

## 5. What You'd Do Next

**If you had two more days on this project, what would you build or fix first?
List in priority order, with one sentence of justification for each.**

Answer:- If I had two more days, I would focus on adding controller and integration tests, improving validation and error handling, optimizing database queries, and covering more edge cases to make the application more reliable and maintainable.

**What is the biggest remaining risk or weakness in the code you have submitted?**

Answer:- The biggest remaining weakness is the limited integration-level testing. While the business logic is covered by unit tests, additional controller and database integration tests would provide greater confidence that all layers work together correctly.
