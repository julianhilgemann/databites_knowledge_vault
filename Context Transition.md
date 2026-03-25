This is the "Hardcore Quant" explanation. If you use this, you will prove that you understand the mathematical foundations of data aggregation.

We will map **[[DAX]] [[Context]] Transition** directly to the statistical concept of **Conditional Expectation** using a dataset as our "Discrete Density Function."

### The Dataset ($\Omega$)

Imagine a dataset with 3 observations (rows).
Let $X$ be the Category (the Condition).
Let $Y$ be the Value (the Random Variable).

| Row ID ($i$) | Category ($X$) | Value ($Y$) |
| :--- | :--- | :--- |
| 1 | **A** | **10** |
| 2 | **A** | **30** |
| 3 | **B** | **80** |

---

### 1. The Math: Unconditional Expectation ($E[Y]$)
*This is **Row [[Context]]** without Transition.*

You are standing at Row 1 ($x_1=A, y_1=10$). You ask for the Mean (Average).
Without a filter, you are calculating the expected value over the **marginal density** of $Y$ (the whole population).

$$E[Y] = \sum_{all} y_i \cdot P(y_i)$$

*   **The Calculation:** $\frac{10 + 30 + 80}{3} = \mathbf{40}$
*   **The Result:** If you put this in a [[Calculated Column]] without `CALCULATE`, **every row shows 40**.
*   **Math Meaning:** You ignored the fact that Row 1 is Category "A". You integrated over the entire domain.

#### The [[DAX]] Code:
```dax
'Table'[Unconditional_Mean] = AVERAGE( 'Table'[Value] )
// Result: 40, 40, 40
```

---

### 2. The Math: Conditional Expectation ($E[Y | X]$)
*This is **[[Context]] Transition**.*

You are standing at Row 1 ($x_1=A$).
Now, you apply **Context Transition**. You are no longer asking for the global mean. You are asking for the **Conditional Mean** given the current row's $X$ value.

$$E[Y \mid X = x_i] = \sum y \cdot \frac{P(X=x_i, Y=y)}{P(X=x_i)}$$

*   **The Calculation for Row 1 (where $X=A$):**
    *   We slice the distribution. We ignore Row 3 ($X=B$).
    *   The subset is $\{10, 30\}$.
    *   Calculation: $\frac{10 + 30}{2} = \mathbf{20}$

*   **The Calculation for Row 3 (where $X=B$):**
    *   We slice the distribution. We ignore Rows 1 & 2 ($X=A$).
    *   The subset is $\{80\}$.
    *   Calculation: $\frac{80}{1} = \mathbf{80}$

#### The [[DAX]] Code:
```dax
'Table'[Conditional_Mean] = 
CALCULATE(
    AVERAGE( 'Table'[Value] )
)
// Result Row 1: 20
// Result Row 2: 20
// Result Row 3: 80
```

---

### 3. The Grand Unification (Math + Audio + DAX)

Here is how you synthesize this for practical application to show you are a master of both sides.

**The Explanation:**

> "In pure math terms, **Context Transition** is the operator that converts an **Unconditional Expectation** $E[Y]$ into a **Conditional Expectation** $E[Y|X]$.
>
> 1.  **Row Context** is just identifying the current observation, $x_i$.
> 2.  **`CALCULATE`** is the function that forces the mathematical conditioning. It takes $x_i$ and applies it as a filter to the probability density function.
>
> It’s exactly like **Audio Engineering**:
> *   The whole dataset is 'White Noise' (Full Frequency Spectrum).
> *   Row Context is pointing at '440Hz'.
> *   **Context Transition** is applying a **Band-Pass Filter** so you *only* hear 440Hz."

### Summary Table

| Concept | The DAX Syntax | The Math Notation | The Result (Row 1) |
| :--- | :--- | :--- | :--- |
| **No Context Transition** | `AVERAGE(Col)` | $E[Y]$ (Global Mean) | **40** (Wrong) |
| **Context Transition** | `CALCULATE(AVERAGE(Col))` | $E[Y \mid X = x_i]$ (Conditional Mean) | **20** (Correct) |