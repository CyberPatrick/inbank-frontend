# Feedback to ticket 401
## Backend
It looks like you want to update the behavior of the `DecisionEngineController` so that when a `NoValidLoanException` occurs, it should return a `400 Bad Request` error instead of `404 Not Found`.

**Why?**

-   `404 Not Found` implies that the requested resource does not exist, which is incorrect in this case because the request is properly routed and processed.

-   `400 Bad Request` is more appropriate because it signals that the client **should not retry the same request without changes**—which aligns with your scenario.
___
You're suggesting that `creditModifier`, currently an attribute of the `DecisionEngine` class, should be converted into a local variable within the `calculateApprovedLoan` function and passed as an argument to `highestValidLoanAmount`.

**Why?**

-   **Class attributes** are meant to define properties of an object. Since `creditModifier` is tied to each function call rather than the object's state, it doesn't need to be an attribute.

-   **Local variables** should be used when a value is only relevant within a specific function call, preventing unnecessary state persistence in the class.

-   **Function arguments** improve clarity and reduce unintended side effects by explicitly passing necessary values between functions.
___
It looks like there is logical issues in `DecisionEngine`:

**The credit score formula is not being applied**

-   The given formula:

    credit score = ((credit modifier / loan amount) * loan period) / 10
-   This formula is crucial for determining loan eligibility, but it is **not currently being used** in the decision-making process.

-   As a result, some valid loans might be rejected, or some invalid loans might be approved.
 ___
You’ve identified **duplicate classes** (`Decision` and `DecisionResponse`), which can lead to unnecessary complexity.

**Possible Solutions**

1.  **Remove one of the classes** (if they are truly identical).

    -   If `DecisionResponse` is only a copy of `Decision` without additional functionality, it should be removed to avoid redundancy.

2.  **Keep both but optimize conversion** (if `DecisionResponse` might evolve separately).

    -   Add a **constructor** to `DecisionResponse` that accepts a `Decision` object.

    -   This would prevent manually setting fields every time a `DecisionResponse` is created from a `Decision`.
___
It might also be worth adding some constants from `DecisionEngineConstants` to `application.properties`, as this would allow values to be changed **without the need to recompile the code**.
___
The constants specify a maximum loan period of 60 months, but it should be 48 months.
## Frontend
The form specify a maximum loan period of 60 months, but it should be 48 months.
___
In the file `loan_form.dart`, on lines 42-43, variables can be used instead of parsing again.
___
**The calculator is not returning the maximum possible loan amount**

-   The requirement states that the system should determine **the highest possible loan amount** a person qualifies for, **regardless** of their requested amount.
-   But now, it returns only requested amount.
___
There is a **performance issue.**
Currently, every time the slider is moved, the client makes a request to the server, resulting in too many unnecessary requests. It would be better to add a button that, when pressed, sends a request to the server for the credit response. Additionally, caching the results can be implemented to prevent repeated requests with the same input data.
## The most important shortcoming of TICKET-101
The frontend is missing the button for the request as well as the display for the maximum loan amount. Also, change maximum load period to 48.
**Fixed.**