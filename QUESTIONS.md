# Questions

Answer each question in your own words. Aim for three to eight sentences per answer —
enough to show that you understand the concept, not so much that you are restating a
textbook entry. We are looking for clarity of thinking, not exhaustive coverage.

---

## Java & Object-Oriented Design

**1.** The `Auditable` abstract class in this codebase uses `@MappedSuperclass`. Explain
what this annotation tells JPA, and describe what would happen to the database schema if
you removed it. Why is it better to put `createdAt` and `updatedAt` in a shared abstract
class rather than adding those fields directly to `Transaction` and `Account` separately?

Answer :- @MappedSuperclass tells JPA to include the fields of the parent class in the tables of its child entities. No separate table is created for the Auditable class. Without this annotation, fields like createdAt and updatedAt may not be mapped correctly. Keeping these fields in a shared class avoids code duplication and makes maintenance easier.

**2.** `TransactionService` is defined as a Java interface, with `TransactionServiceImpl`
as its only implementation. A new engineer on the team asks: "Why bother with the interface
if there's only one implementation? Isn't it just extra boilerplate?" How do you respond?
Give at least one concrete scenario where the interface pays off.

Answer:- Even though TransactionService currently has only one implementation, using an interface is still useful. It keeps the code flexible and makes future changes easier. For example, if another implementation is needed later, it can be added without affecting the rest of the application. Interfaces also make testing easier through mocking.

**3.** `Category` is modelled as an enum rather than a plain `String` field on
`Transaction`. What does storing it as `@Enumerated(EnumType.STRING)` in the database
actually produce in the table? What would go wrong if a future developer added a new
category value to the enum but forgot to handle database migration?

Answer:- Using @Enumerated(EnumType.STRING) stores enum values such as FOOD and TRANSPORT as text in the database. This makes the data easier to read and understand. If a new category is added but related logic is not updated, some features may not handle it correctly. This could lead to incorrect reports or unexpected behavior.

**4.** `BudgetCalculator` is a `final` class with a private constructor and a single
static method. What pattern is this, and why is it appropriate for this specific utility?
In your implementation, what data structure did you use as an intermediate step before
building the final sorted map, and why?

Answer:- BudgetCalculator follows the Utility Class pattern. It is final and has a private constructor because it only contains helper methods and does not need to be instantiated. I used a Map<Category, BigDecimal> to group transactions and calculate totals. The final result was stored in a LinkedHashMap to preserve the sorted order.

---

## Spring Boot & REST API Design

**5.** The original `POST /api/transactions` endpoint returned a `ResponseEntity<Transaction>`
rather than a `ResponseEntity<TransactionResponse>`. Explain specifically what was wrong
with this. What does a DTO (data transfer object) protect against, and what risks does
returning an entity directly introduce?

Answer:- Returning ResponseEntity<Transaction> exposed the database entity directly to API clients. A DTO acts as a layer between the API and the database and lets us control the data that is returned. Returning entities directly can expose internal details and make future changes more difficult. Using TransactionResponse makes the API cleaner and easier to maintain.

**6.** When a `POST` request arrives at `TransactionController`, describe the complete
journey from HTTP request to database insert. Name each layer the request passes through,
what each layer is responsible for, and what would happen if the `@Valid` annotation were
removed from the method parameter.

Answer:- When a POST request arrives, it first reaches the controller. The controller validates the request and passes it to the service layer, which handles the business logic. The service then calls the repository, and JPA saves the data to the database. If @Valid is removed, invalid data could be stored in the database.

**7.** Spring Boot uses `@RestController`, `@Service`, and `@Repository` as stereotype
annotations. They all ultimately do the same thing (register a bean). Why does Spring
provide three different annotations instead of one? What does the distinction communicate
to a developer reading the code?

Answer:- Spring provides different annotations to make the purpose of each class clear. @RestController handles HTTP requests, @Service contains business logic, and @Repository manages database operations. This separation improves readability and maintainability. A developer can quickly understand the responsibility of a class by looking at its annotation.

**8.** The `GET /api/transactions/monthly-spend` endpoint accepts `year` and `month` as
query parameters. What HTTP status code should this endpoint return if `month=13` is
passed? Who is responsible for validating it — the controller, the service, or Spring
itself — and how would you implement that validation?

Answer:- If month=13 is passed, the endpoint should return HTTP 400 (Bad Request) because the input is invalid. I would perform the validation in the service layer since it contains the business rules. This ensures the validation is applied wherever the service is used. If the value is outside the range 1–12, an exception can be thrown and returned as a 400 response.



---

## Data Access & SQL

**9.** `TransactionRepository` extends `JpaRepository<Transaction, Long>`. Spring Data JPA
can generate a query from a method named `findByAccountId`. Explain the mechanism behind
this — what is Spring doing at startup to turn that method name into SQL? When would you
write a `@Query` annotation instead of relying on derived query methods?

Answer:- Spring Data JPA examines repository method names at startup and automatically generates the required queries. For example, findByAccountId creates a query that retrieves records matching the accountId field. This reduces the amount of boilerplate code developers need to write. I would use @Query when the query becomes too complex for a method name, such as joins or aggregations.

**10.** `calculateMonthlySpend` had a bug in the date boundary comparison. Describe the
bug in plain language — what was the incorrect behaviour, what caused it at the code level,
and what kind of test input reliably exposes this class of off-by-one error? Why is this
type of bug particularly common in date/time logic?

Answer:- The bug caused transactions on the first day of the month to be excluded from the monthly spend calculation. This happened because isAfter(startOfMonth) ignores dates that are exactly equal to the start date. A transaction dated 2024-12-01 is a good test case for exposing this issue. These bugs are common because date comparisons often involve inclusive and exclusive boundaries.

**11.** The application uses H2 in-memory for development. Describe exactly what you
would change to point this application at a PostgreSQL database in production. Be specific:
which files, which properties, and which Maven dependency. What is the risk of using
`spring.jpa.hibernate.ddl-auto=create-drop` in production?

Answer:- To use PostgreSQL in production, I would replace the H2 dependency with the PostgreSQL dependency in pom.xml. I would also update the database URL, username, and password in application.properties. This would allow the application to connect to PostgreSQL instead of H2. Using create-drop in production is risky because it can delete existing tables and data.

---

## Testing

**12.** `TransactionServiceTest` uses `@Mock` on `TransactionRepository` and
`@InjectMocks` on `TransactionServiceImpl`. Explain what Mockito is doing here. What is
the repository being replaced with, and what does the test actually verify? What category
of bug can this test suite catch — and what category can it not?

Answer:- Mockito creates a fake TransactionRepository and injects it into TransactionServiceImpl. This allows the service logic to be tested without using a real database. The tests verify business logic such as monthly spend calculations. They can catch logic errors, but they cannot detect database or API-related issues.

**13.** A teammate argues that because the service tests cover all the logic, there is no
need to write controller tests. Do you agree? Describe one specific type of bug that a
controller-level test (using `MockMvc`) would catch that the service tests in this project
would miss entirely.

Answer:- I do not agree. Service tests are important, but controller tests verify that HTTP requests and responses work correctly.For example, a MockMvc test could catch a bug where the controller returns the wrong HTTP status code or the wrong response format. Service tests would not detect this because they do not test the API layer.



**14.** Looking at the tests you wrote in `TransactionCandidateTest.java`: what was the
first test you wrote, and why did you choose to start there? What does the order in which
you wrote tests tell you about how you approached the problem?

Answer:- The first test I wrote was for the monthly spend calculation because it was one of the main bugs identified in the project.I started with that test because it directly verified the business logic and helped confirm that the date boundary issue was fixed. The order of my tests shows that I focused on validating the most important business requirements first before moving to other cases.



---

## AI & Modern Engineering

**15.** Describe how you used AI tools during this project. For at least two specific
examples: what did you prompt the tool with, what did it return, and what did you change
or reject? Identify one place where the AI output was immediately trustworthy and one
place where it required meaningful scrutiny before you used it.

Answer:- I used AI to help analyze failing tests and review possible fixes. For example, it helped identify the date boundary issue in the monthly spend calculation and suggested an implementation for BudgetCalculator. I trusted the date bug fix because the failing test clearly confirmed it. However, I reviewed the BudgetCalculator logic carefully before using it to ensure it matched the project requirements.
