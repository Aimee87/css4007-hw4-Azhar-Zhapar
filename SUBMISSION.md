# HW4 — submission

**AI assistance:** <!-- what you used, and for what. "None" is an answer too. -->

---

## Part A — Plan on paper (Easy)

> Commit this whole part before any `.py` file exists.

### A1 — The steps

Q-09: Which district has the higher population density in 2025, Brightwater or Millbrook, and by how many residents per square kilometre?

| # | Step | What goes in (types) | What comes out (types) | When it goes wrong | Tool, loop, or model? |
|---|---|---|---|---|---|
| 1 |Identify the regions and the required metric from the question|question: str|districts: list[str], metric: str|District not recognised; Metric not recognised|Model|
| 2 |Retrieve required facts from documents|District names and required fields|Population, area and source document IDs|District not found; Multiple matches found|lookup tool|
| 3 |Apply corrections and validation rules|Retrieved facts|Corrected and validated facts|Conflicting values; Unknown unit|lookup tool|
| 4 |Build arithmetic expressions|Validated facts|expression: str|Missing data required for calculation|Model|
| 5 |Execute calculations|expression: str|result: float|Division by zero; Invalid expression|calculate tool|
| 6 |Compare calculated values|Calculated results|District with higher value and difference|Missing calculation result|Model|
| 7 |Generate final answer|Comparison result|Final answer|Insufficient information|Model|

**How the steps connect:**
The output of each step becomes the input of the next step. This creates a clear workflow from question understanding to fact retrieval, calculation and final answer generation.

Output Step 1 = Input Step 2

Output Step 2 = Input Step 3 etc.

**What I am assuming:**
|The latest (2025) district profiles are used by default|

|Archived profiles are ignored|

|Correction notices override information in district profiles|

|Unit conversions are performed when necessary|

|Arithmetic calculations are performed by the calculate tool|

|Rounding is applied only to the final answer|

|Each fact returned by lookup includes its source document ID|

|Questions are answered using only the provided documents|

### A2 — The two tools

#### `lookup`

```json
```

| Case | What the tool returns |
|---|---|
| success | |
| | |

**Worst thing it could do with strange or dangerous input:**

#### `calculate`

```json
```

| Case | What the tool returns |
|---|---|
| success | |
| | |

**Worst thing it could do with strange or dangerous input:**

### A3 — Q-09 by hand

```json
```

**My arithmetic:**

### A4 — Why I split it this way

<!-- one other way to split the task, and why you did not choose it -->

---

## Part B — Build it and run it (Medium)

### What I changed from Part A while coding

<!-- what changed in your plan, and why. "None" is an answer too. -->

### B2 — Tests

```text
<!-- pytest output -->
```

### B3 — The loop

- Round limit, and why:
- Where the logs are:
- Real Q-09 conversation compared with my A3 version, what is different:

### B4 — Three strategies

**What I kept the same:**

<!-- model, temperature, k, prompts, answer format, anything else -->

**My rule for a correct answer** (written before the first full run):

| Run | Strategy | Correct /20 | Unanswered | Input tokens | Output tokens | Median seconds per question | Total seconds | Cost ($) |
|---|---|---:|---:|---:|---:|---:|---:|---:|
| 1 | S1 direct | | | | | | | |
| 1 | S2 think and vote (k = ) | | | | | | | |
| 1 | S3 tools | | | | | | | |
| 2 | S1 direct | | | | | | | |
| 2 | S2 think and vote (k = ) | | | | | | | |
| 2 | S3 tools | | | | | | | |

**Per question (run 1):**

| Question | S1 | S2 | S3 |
|---|:---:|:---:|:---:|
| Q-01 | | | |
| Q-02 | | | |
| Q-03 | | | |
| Q-04 | | | |
| Q-05 | | | |
| Q-06 | | | |
| Q-07 | | | |
| Q-08 | | | |
| Q-09 | | | |
| Q-10 | | | |
| Q-11 | | | |
| Q-12 | | | |
| Q-13 | | | |
| Q-14 | | | |
| Q-15 | | | |
| Q-16 | | | |
| Q-17 | | | |
| Q-18 | | | |
| Q-19 | | | |
| Q-20 | | | |

---

## Part C — Explain and choose (Hard)

### C1 — Does the tool description matter?

| Question | Calls (your description) | Calls (vague) | Answer (yours) | Answer (vague) |
|---|---:|---:|---|---|
| | | | | |

### C2 — Three mistakes explained

### C3 — Which strategy I would use, and what would change it

### C4 — How sure I can be

### Final questions

1.
2.
3.
4.
5.
6.
7.
8.
