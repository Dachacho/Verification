# Java Verification (I2I 2)

## *(this is going to be helpful notes to keep by your side when going through slides, chapter by chapter)*

***

### 1. Correctness of Programs

ensuring the correctnes of programs is vital to prevent costly errors in critical systems. so that's where verification comes in.

#### there are different approaches to have program behave as intended  

* Careful engineering
* Testing systematically
* proof of correctness(verification)
  * for verification we use the tool - **Assertions**.

#### example from the slides (simple GCD program)

```java
public class GCD {
  public static void main (String[] args) {
    int x, y, a, b;
    a = read(); x = a;
    b = read(); y = b;
    while (x != y)
        if (x > y) 
            x = x - y;
        else 
            y = y - x;
    assert(x == y);
    write(x);
  } // End of definition of main();
}//End of class GCD ;
```

in this case `assert()` function checks if `x == y` and throws `AssertionError` if it's false **otherwise** program continues.

### 1.1 Program Verification

firstly we are going to only use `MiniJava` in this case to limit what we are using.

* `int` variables only.
* only `if` and `while`.

we are going to use this for our program to be easier to understand and prove.  
*(at least that's my understanding why he decided to use it).*

so we are going to annotate each point with a formula, as in `assert()` and we are going to prove that those assertions are valid with logic.

#### Logic

#### take a look at logic example provided in the slides

**Assertion**:  

* All humans are mortal.
* socrates is a human.
* so by the means of logic we can say that **socrates is mortal**.
* **`∀ x. human(x) ⇒ mortal(x)`** *(logical representation)*

**deduction**:

* if for all x some P(x) holds, than P(a) should hold for any specific a.
* If A ⇒ B and A holds, then B must hold as well!
  * this means that if A implies B and A is true, than B must also B true.
* so going back to socrates example  
  * if **`∀ x. human(x) ⇒ mortal(x)`**  
  * **`human(socrates) ⇒ mortal(socrates)`**
  * this is assertion hold true because if one holds true than other must hold true as well.

**tautology**:

* **`A ∨ ¬A`**  
  * this expression **"A or not A"** always holds true, because it covers all possible cases.
* **`∀ x ∈ Z. x < 0 ∨ x = 0 ∨ x > 0`**
  * for any integer x, it can either be negative, positive or zero. Thus this holds true for any integer.

**drawing conclusion from this**:

* **Assertion** - A statment that expresses a fact/truth.
* **Deduction** - drawing some sort of conclusion from the premise given.
* **Tautology** - Statement that always holds true.

#### now lets continue with an example

![cFlow1](cFlow1.PNG)  
this is an example program that we used earlier(GCD program), take each edge of this control flow as a program point.

so we are going to require assertions (as mentioned above).

**some background info we need**:

**gcd(x, y) = 0, if x = y = 0, otherwise greatest number d that divides both of them.**

**by that this laws hold**:

* `gcd(x, 0) = |x|`
* `gcd(x, x) = |x|`
* `gcd(x, y) = gcd(x, y − x)`
* `gcd(x, y) = gcd(x − y, y)`

**explanation *(if you care)***:

* so this laws are for calculating gcd *(of course)*.
* first two laws are emphesize the behaviour of gcd when one of the numbers is 0 or if they are they are the same *(clear enough)*.
* last two laws provide a way to simplify gcd calculation by reducing the values, ultimately finding the gcd *(this are the laws that we use in the main GCD program)*.

#### so now we start to introduce some assertions in our program

* initially nothing holds *(as we haven't started to do anything)*.
* after `a = read(); x = a` assertion `x = a` holds, we do the same for `b = y`.
* **Before entering and during the loop, we should have `A ≡ gcd(a, b) = gcd(x, y)`:**
  * this assertion labeled "A", emphesizes that both gcd's should be the same before and during execution.
* **At program exit, we should have `B ≡ A ∧ x = y`:**
  * so assertion labeled "B", states that after execution both the assertion "A" and the condition `x = y` should be true.
* **These assertions should be locally consistent:**  
  * The term **"locally consistent"** implies that within specific sections of the program, the assertions should hold true and not contradict each other.

#### the control flow from before becomes

![cFlow2](cflow2.PNG)

where **`A ≡ gcd(a, b) = gcd(x, y)`** and **`B ≡ A ∧ x = y`** *(as stated above)*.

#### how can we prove that this assertions are locally consistant

#### Sub-problem 1: Assignments

**so lets take example: `x = y + z`**

* Post-condition *(after the assignment)*: we want **x > 0**
* Pre-condition *(before the assignment)*: we need **y + z > 0**
* General principle:
  * every assignment transforms post-condition into some minimal assumption *(pre-condition)* that must be true to ensure it's consistency.
  * this means that we need to identify minimal condition that must be true before execution/assignment so that we get desired outcome.

**extension of the general principle**:

* for an assignment `x = e` the weakest pre-condition *(so minimal assumption)* is given by `WP[[x = e;]] (B) ≡ B[e/x]`
  * annotation: the weakest pre-condition is found by replacing every variable x with expression e in the post-condition B.

**example application:**

* if the assignment `x = x - y` and post-condition is `x > 0`, then our weakest pre-condition becomes `x - y > 0`.
* but we can get even stronger pre-condition such as `x - y > 2` or even stronger with `x - y = 3` to get desired outcome *(if needed)*.
  
**an arbitrary pre-condition A for statement s is  valid whenever `A ⇒ WP[s] (B)`**

* so pre-condition A is considered valid if it implies the weakest pre-condition for B. this ensures that the pre-condition is strong enough to guarantee ourselves the desired outcome *(post-condition)*.

#### getting back to our GCD program

*(The substitution of (a, b) with (x, y) is a way to emphasize the generic nature of the analysis).*

##### example 1

1. take assignment `x = x - y`
2. we have post-condition **A**
3. `weakest pre-condition = A[x − y/x] ≡ gcd(a,b) = gcd(x − y,y) ≡ gcd(a,b) = gcd(x,y)
≡ A` *(explanation as this confused the shit out of me)*:  
   * `A[x − y/x]` so we substite every `x` with `x - y`
     * `gcd(a,b) = gcd(x − y,y)`
   * simplify `gcd(a,b)=gcd(x,y)`
     * we can do this by above given simplification laws. so `gcd(x - y,y) = gcd(x,y)`
   * as we can see this holds so we get result: **A *(Post-condition is our Weakest Pre-condition)***
  
##### example 2

1. take assignment `y = y - x`
2. we have post-condition **A**
3. `weakest pre-condition:
A[y − x/y] ≡ gcd(a, b) = gcd(x, y − x)
≡ gcd(a, b) = gcd(x, y)
≡ A` *(explanation)*:
   * `A[y − x/y]` so we substitute every `y` with `y - x`
   * simplify `gcd(a,b)=gcd(x, y − x) ≡ gcd(a, b)=gcd(x, y)` which in turn equals `A`
   * so we end up with Result: **A *(Post-condition equals Weakest Pre-condition)***

**summary *(to not get lost in the details)*:**

* Analyze two assignments using weakest pre-condition calculations.
* Result: Both assignments maintain gcd(a, b) = gcd(x, y) as the weakest pre-condition, matching the post-condition (A).
* Outcome: GCD program preserves the desired property (A) throughout execution.

#### Wrap Up

![alt text](wrapUp.PNG)

**Let's go over it one by one:**  

1. `WP[;](B) ≡ B`
   * Weakest Pre-condition in this case is denoted by [;], meaning no action is performed. So, in simple terms, this expression is stating that the weakest pre-condition for a statement that does nothing is exactly the post-condition itself.  
2. `WP[x = e;](B) ≡ B[e/x]`
   * this expression represents the weakest pre-condition for an assignment `x = e`
   * so the weakest pre-condition ensures that B still holds after we perform `x = e`. The substitution `B[e/x]` ensures that post-condition is adapted to the changes that are made by the assignment.
3. `WP[x = read();](B) ≡ ∀ x. B`
   * this represents the weakest pre-condition for an input statement `x = read()`
   * as `x` is an input variable and can be anything user wants we need `B` to hold for every x possible, hence `∀ x. B`
4. `WP[write(e);](B) ≡ B`
   * represents the weakest pre-condition calculation for an output statement `write(e)`
   * the weakest pre-condition for an output statement is essentially the post-condition itself, as the output operation doesn't introduce any changes to the program state.

#### so back to our GCD program

##### *(see control flow above as nothing has changed idk why he drew it again but can't be bothered)*

For statements `b = read(); y = b;`

For `y = b`:

* we calculate weakest pre-condition by `WP[y = b;] (A) ≡ A[b/y]
≡ gcd(a, b) = gcd(x, b)
`
  * so we are calculating weakest pre-condition to ensure assertion `A` holds afterwards
  * `A[b/y]
≡ gcd(a, b) = gcd(x, b)` we have substitued the y with b
  * so before the assignment `A` was `gcd(a,b) = gcd(x,y)`, but after the assignment `y = b` assertion A became `gcd(a,b) = gcd(x,b)`

For `b = read()`:

* we calculate weakest pre-condition by `WP[b = read();] (gcd(a, b) = gcd(x, b))
≡ ∀ b. gcd(a, b) = gcd(x, b) ⇐ a = x`
  * this is calculating weakest pre-condition so that the assertion `gcd(a, b) = gcd(x, y)` holds for any give value of b
  * this is further restricted by implication `⇐ a = x` indicating that our weakest pre-condition is only valid if `a = x`
  
For the statements `a = read(); x = a;`
For `x = a`:

* we calculate weakest pre-condition by `WP[x = a;] (a = x) ≡ a = a
≡ true`
  * this is cacluating weakest pre-condition for assignment `x = a` so that we get assertion `a = x` to be true afterwards.
  * assignment `x = a` does nothing to introduce any new conditions or modify the relationship of `a` and `x` therefore itself is weakest pre-condition. thats why we get `a = a`

For `a = read()`

* we calculate weakest pre-condition by `WP[a = read();] (true) ≡ ∀ a. true
≡ true`
  * before input `a = read()` assertion `true` must be `true` for any possible value of `a`. this is because input statement doesn't impose any specific condition on value of `a`

#### Sub-problem 2: Conditionals

![alt text](conditionals1.PNG)

Take a look at above give control flow following statements should hold:

* `A ∧ ¬b ⇒ B₀` *(A holds and b conditional gives false goes to B0)*
* `A ∧ b ⇒ B₁` *(A holds and b conditional gives true goes to B1)*

if `A` implies weakest pre-condition branching:

* `WP[b] (B₀, B₁) ≡ ((¬b) ⇒ B₀) ∧ (b ⇒ B₁)` *(It asserts that the weakest pre-condition is satisfied if the assertion A implies the necessary conditions for both branches)*
* this can be rewritten *(according to slides)* into this `(b ∨ B₀) ∧ (¬b ∨ B₁)` which in can be rewritten into `(¬b ∧ B₀) ∨ (b ∧ B₁) ∨ (B₀ ∧ B₁)` and finally into `(¬b ∧ B₀) ∨ (b ∧ B₁)` *(by means of boolean simplification)* and we get final expression that provides clear understanding of the condition required for assertion A to hold for specified branches.

#### example

we have branch conditions:

* `B₀ ≡ x > y ∧ y > 0`
* `B₁ ≡ y > x ∧ x > 0`
* we can assume that condition b is `y > x`

then we can calculate weakest pre-condition by `(x ≥ y ∧ x > y ∧ y > 0) ∨ (y > x ∧ y > x ∧ x > 0)
≡ (x > y ∧ y > 0) ∨ (y > x ∧ x > 0)
≡ x > 0 ∧ y > 0 ∧ x != y`

* so we use the general formula we calculated earlier `(¬b ∧ B₀) ∨ (b ∧ B₁)` and we substitute with our give conditionals so we get `(x ≥ y ∧ x > y ∧ y > 0) ∨ (y > x ∧ y > x ∧ x > 0)` and after which we can simplify into `x > 0 ∧ y > 0 ∧ x != y` which indicates that the weakest pre-condition for the condition `y > x` is met when `x > 0, y > 0, x != y`
  * *(for conditionals weakest pre-conditions are to ensure that branches satisfy their given respective conditions)*

#### back to our GCD program

we have branches:

condition `b = y > x`

* `¬b ∧ A ≡ x ≥ y ∧ gcd(a, b) = gcd(x, y)`
* `b ∧ A ≡ y > x ∧ gcd(a, b) = gcd(x, y)`

*(we are going to use formula that we calculated above and instead of `b` we used the condition and instead of assertion `A` we gave it the expression gcd(a, b) = gcd(x, y))*

`WP[b] (B₀, B₁) ≡ (¬b ∧ B₀) ∨ (b ∧ B₁) ≡ WP[y > x](x ≥ y ∧ gcd(a, b) = gcd(x, y)) ≡ (¬(y > x) ∧ (x >= y ∧ gcd(a, b) = gcd(x, y))) ∨ ((y > x) ∧ (y > x ∧ gcd(a, b) = gcd(x, y)))`

with further simplification we get `gcd(a, b) = gcd(x, y)` resulting that weakest pre-condition is the same as assertion `A`

![alt text](cflow2.PNG)

now for the outer loop

conditional: `b ≡ x!=y`

branches:

* `¬b ∧ B ≡ B`
* `b ∧ A ≡ A ∧ x != y`

using our general formula: `WP[b](B₀, B₁) (¬b ∧ B₀) ∨ (b ∧ B₁)` 

we get `WP[x != y] (B, A ∧ x != y) ≡ (¬(x != y) ∧ B) ∨ ((x != y) ∧ (A ∧ x != y)`

through further simplification we get `(B ∧ y == x) ∨ (A ∧ x != y)` and if we substitute `B` with its expression `A ∧ x = y` we get `(A ∧ x == y ∧ y == x) ∨ (A ∧ x != y)` *(i did this because i wanted it to be same as the slides and you might also get confused)*

#### Summary of all this

1. Assertion Annotation
   * Annotate each point of the program with a logical condition
   * begin the program with the assertion `true`
2. Verification Process
   * For every action or statement between two logical conditions confirm that initial condition logically leads to *(implies)* the expected outcome
3. Conditional Branching
   * For every decision in the program check if the preceeding condition guarantees the expected results for both possible outcomes
4. local Consistancy
   * local consistency is about making sure that each logical step in your program flows seamlessly from the previous one, forming a solid and sensible chain of conditions throughout the execution
