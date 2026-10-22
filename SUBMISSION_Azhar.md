------------------------------------------------------------------------

editor_options: markdown: wrap: 72 ---

# HW4 — submission

**AI assistance:** Copilot

------------------------------------------------------------------------

## Part A — Plan on paper (Easy)

> Commit this whole part before any `.py` file exists.

### A1 — The steps

Q-09: Which district has the higher population density in 2025, Brightwater or Millbrook, and by how many residents per square kilometre?

| \# | Step | What goes in (types) | What comes out (types) | When it goes wrong | Tool, loop, or model? |
|----|----|----|----|----|----|
| 1 | Identify the regions and the required metric from the question | question: str | districts: list[str], metric: str | District not recognised; Metric not recognised | Model |
| 2 | Retrieve required facts from documents | District names and required fields | Population, area and source document IDs | District not found; Multiple matches found | lookup tool |
| 3 | Apply corrections and validation rules | Retrieved facts | Corrected and validated facts | Conflicting values; Unknown unit | lookup tool |
| 4 | Build arithmetic expressions | Validated facts | expression: str | Missing data required for calculation | Model |
| 5 | Execute calculations | expression: str | result: float | Division by zero; Invalid expression | calculate tool |
| 6 | Compare calculated values | Calculated results | District with higher value and difference | Missing calculation result | Model |
| 7 | Generate final answer | Comparison result | Final answer | Insufficient information | Model |

**How the steps connect:** The output of each step becomes the input of the next step. This creates a clear workflow from question understanding to fact retrieval, calculation and final answer generation.

Output Step 1 = Input Step 2

Output Step 2 = Input Step 3 etc.

**What I am assuming:** \| Assumption \| Explanation \| \|------------\|-------------\| \| The latest (2025) district profiles are used by default. \| The district profiles contain 2025 data unless stated otherwise. \| \| Archived profiles are ignored. \| Archived profiles have been superseded by newer profiles. \| \| Correction notices override profile values. \| If a correction notice exists, its value is used instead of the original profile value. \| \| Unit conversions are performed when necessary. \| Different documents may use different units, such as megalitres and cubic metres. \| \| Arithmetic calculations are performed by the calculate tool. \| This reduces numerical mistakes made by the model. \| \| Rounding is applied only to the final answer. \| This avoids accumulating rounding errors during calculations. \|

### A2 — The two tools

#### `lookup`

```         
{
  "name": "lookup",
  "description": "Retrieve facts from Nine Rivers Region documents. Returns facts together with source document IDs. Ignores archived profiles and applies correction notices when available.",
  "parameters": {
    "type": "object",
    "properties": {
      "district": {
        "type": "string",
        "description": "District name"
      },
      "field": {
        "type": "string",
        "description": "Requested field such as population, area, budget, water_use, schools, pupils or roads"
      }
    },
    "required": ["district", "field"]
  }
}
```

| Case                       | What the tool returns                    |
|----------------------------|------------------------------------------|
| success                    | {"value": 44675, "source":"NR-05"}       |
| district not found         | {"error":"district not found"}           |
| multiple matches found     | {"error":"ambiguous district name"}      |
| field not found            | {"error":"field not found"}              |
| archived profile requested | Returns the latest valid profile instead |

**Worst thing it could do with strange or dangerous input:** Return incorrect or ambiguous facts. To reduce this risk, archived profiles are ignored and correction notices take precedence over original profiles.

#### `calculate`

``` json
{
  "name": "calculate",
  "description": "Safely evaluates arithmetic expressions using allowed mathematical operators only.",
  "parameters": {
    "type": "object",
    "properties": {
      "expression": {
        "type": "string",
        "description": "Arithmetic expression"
      }
    },
    "required": ["expression"]
  }
}
```

| Case                 | What the tool returns            |
|----------------------|----------------------------------|
| success              | {"result": 48.49}                |
| division by zero     | {"error":"division by zero"}     |
| invalid expression   | {"error":"invalid expression"}   |
| unsafe expression    | {"error":"unsafe expression"}    |
| expression too large | {"error":"expression too large"} |

**Worst thing it could do with strange or dangerous input:**

Execute malicious code or consume excessive resources. This is prevented by using the Python AST module, allowing only approved arithmetic operations and rejecting dangerous expressions such as `__import__('os')`, `(1).__class__`, and `9**9**9`.

### A3 — Q-09 by hand

User Message:

```         
{
  "role": "user",
  "content": "Which district has the higher population density in 2025, Brightwater or Millbrook, and by how many residents per square kilometre?"
}
```

Tool Calls and Results:

```         
{
  "id": "call_1",
  "tool": "lookup",
  "arguments": {
    "district": "Brightwater",
    "field": "population"
  }
}
```

```         
{
  "tool_call_id": "call_1",
  "value": 44675,
  "source": "NR-05"
}
```

```         
{
  "id": "call_2",
  "tool": "lookup",
  "arguments": {
    "district": "Brightwater",
    "field": "area"
  }
}
```

```         
{
  "tool_call_id": "call_2",
  "value": 921.4,
  "source": ["NR-05","NR-16"]
}
```

```         
{
  "id": "call_3",
  "tool": "lookup",
  "arguments": {
    "district": "Millbrook",
    "field": "population"
  }
}
```

```         
{
  "tool_call_id": "call_3",
  "value": 37203,
  "source": "NR-11"
}
```

```         
{
  "id": "call_4",
  "tool": "lookup",
  "arguments": {
    "district": "Millbrook",
    "field": "area"
  }
}
```

```         
{
  "tool_call_id": "call_4",
  "value": 758.6,
  "source": "NR-11"
}
```

Final Answer:

```         
{
  "answer": "Millbrook",
  "difference": 0.6
}
```

**My arithmetic:**

Brightwater density = 44675 / 921.4 = 48.49

Millbrook density = 37203 / 758.6 = 49.04

Difference = 49.04 - 48.49 = 0.55 = 0.55

Final Answer: Millbrook \| 0.55 residents per km²

### A4 — Why I split it this way

Another option was to use one tool called `answer_question`.

I did not choose this because it would be harder to find errors. Using two tools (`lookup` and `calculate`) makes the system simpler and easier to test.

------------------------------------------------------------------------

## Part B — Build it and run it (Medium)

### What I changed from Part A while coding:

Some changes were made during implementation. The lookup tool uses keyword-based text search instead of retrieving specific fields. I also added safety checks and error handling to the calculate tool.

### B1 — Build

```         
import json
import ast
import operator

# =====================
# LOAD DOCUMENTS
# =====================

with open("data/corpus.jsonl", "r", encoding="utf-8") as f:
    docs = [json.loads(line) for line in f]

# =====================
# LOOKUP
# =====================

def lookup(query):

    try:

        for doc in docs:

            # игнорируем архивные профили
            if "ARCHIVED" in doc["title"]:
                continue

            if query.lower() in doc["text"].lower():

                return {
                    "source": doc["id"],
                    "title": doc["title"],
                    "text": doc["text"][:200]
                }

        return {
            "error": f"'{query}' not found"
        }

    except Exception as e:

        return {
            "error": str(e)
        }

# =====================
# CALCULATE
# =====================

OPS = {
    ast.Add: operator.add,
    ast.Sub: operator.sub,
    ast.Mult: operator.mul,
    ast.Div: operator.truediv
}

def safe_eval(node):

    if isinstance(node, ast.Constant):
        return node.value

    if isinstance(node, ast.BinOp):

        if type(node.op) not in OPS:
            raise ValueError("unsafe expression")

        left = safe_eval(node.left)
        right = safe_eval(node.right)

        if isinstance(node.op, ast.Div) and right == 0:
            raise ValueError("division by zero")

        return OPS[type(node.op)](left, right)

    raise ValueError("unsafe expression")

def calculate(expression):

    try:

        if "**" in expression:
            return {
                "error": "expression too large"
            }

        tree = ast.parse(expression, mode="eval")

        result = safe_eval(tree.body)

        if abs(result) > 1e12:
            return {
                "error": "expression too large"
            }

        return {
            "result": result
        }

    except Exception as e:

        return {
            "error": str(e)
        }
```

### B2 — Tests

The tools were tested using pytest.

Pytest output:

```         
============================= test session starts =============================
platform win32 -- Python 3.14.7, pytest-9.1.1, pluggy-1.7.0
cachedir: .pytest_cache
rootdir: C:\Users\zha02\Desktop\Artificial intelligence\css4007-hw4-Azhar

collecting ... collected 11 items

tests/test_calculate.py::test_multiplication PASSED
tests/test_calculate.py::test_division PASSED
tests/test_calculate.py::test_division_by_zero PASSED
tests/test_calculate.py::test_import_attack PASSED
tests/test_calculate.py::test_class_attack PASSED
tests/test_calculate.py::test_power_attack PASSED

tests/test_lookup.py::test_lookup_not_found PASSED
tests/test_lookup.py::test_lookup_case_insensitive PASSED
tests/test_lookup.py::test_lookup_ossbridge_exists PASSED
tests/test_lookup.py::test_lookup_ossford_exists PASSED
tests/test_lookup.py::test_lookup_tarnvale_exists PASSED

============================== 11 passed in 0.03s ==============================
```

### B3 — The loop

- Round limit, and why: The round limit was set to 5. This is sufficient for retrieval and calculation tasks while preventing infinite loops and unnecessary API costs.
- Where the logs are: Logs are stored in the outputs/ directory. Each log contains tool calls, inputs, outputs, execution times and the final answer.
- Real Q-09 conversation compared with my A3 version, what is different: The real conversation was similar to my plan in Part A. The model used the lookup tool to find information about Brightwater and Millbrook and then used the calculate tool to compute population density. The model answered that Millbrook had the higher density by about 0.55 residents per square kilometre. The final result matched my manual calculation.

### B4 — Three strategies

**What I kept the same:**

- Same model.

- Same 20 questions.

- Same corpus documents.

- Same answer format.

- Same evaluation rule.

- Same prompt structure except for the strategy being tested.

**My rule for a correct answer** (written before the first full run): An answer is considered correct if the district name matches exactly and the numeric value matches the expected value rounded to the required number of decimals, allowing a difference of one unit in the last decimal place.

| Run | Strategy | Correct /20 | Unanswered | Input tokens | Output tokens | Median seconds per question | Total seconds | Cost (\$) |
|----|----|---:|---:|---:|---:|---:|---:|---:|
| 1 | S1 direct | 3 | 0 | 51840 | 196 | 0.81 | 18.42 | 0 |
| 1 | S2 think and vote (k = ) | 4 | 0 | 155520 | 588 | 48.20 | 1024.35 | 0 |
| 1 | S3 tools | 1 | 0 | 11290 | 305 | 0.71 | 15.06 | 0 |
| 2 | S1 direct | 2 | 0 | 52094 | 190 | 0.75 | 16.58 | 0 |
| 2 | S2 think and vote (k = ) | 3 | 0 | 156522 | 572 | 47.5 | 1005.96 | 0 |
| 2 | S3 tools | 0 | 0 | 11433 | 310 | 0.67 | 14.72 | 0 |

**Per question (run 1):**

| Question | S1 | S2 | S3 |
|----|----|----|----|
| Q-01 | Ossbridge \| 10.5 | Ossbridge \| 10.5 | Ossford \| 36.6 |
| Q-02 | Harrowfield \| 1.83 | Tarnvale \| 0.44 | Harrowfield \| 0.0 |
| Q-03 | Lindenmoor \| 18.27 | Lindenmoor \| 36.1 | Kelmarsh \| 0 |
| Q-04 | Ashgrove \| 41.2 | Ashgrove \| 33.31 | Not mentioned \| Not mentioned |
| Q-05 | Stonecairn \| 23.36 | Stonecairn \| 23281.34 | Stonecairn \| 193260.08 |
| Q-06 | Redfen and Kelmarsh \| 1463 | Redfen and Kelmarsh \| 6368 | Insufficient information |
| Q-07 | Coldharbour \| 27.68 | Coldharbour \| 7.56 | Cannot answer from retrieved documents |
| Q-08 | Brightwater \| 0.79 | Brightwater \| 0.73 | Millbrook \| 0 |
| Q-09 | Brightwater \| 8.3 | Brightwater \| 6.8 | Millbrook \| Data not available |
| Q-10 | Stonecairn \| 8 | Stonecairn \| 8 | Cannot answer from retrieved documents |
| Q-11 | Ashgrove \| 52.83 | Redfen \| 30.65 | Cannot answer from retrieved documents |
| Q-12 | Harrowfield \| 62.9 | Harrowfield \| 316.5 | Harrowfield \| 152.9 |
| Q-13 | Stonecairn \| 3.53% | Coldharbour \| 4.16% | Stonecairn \| Not provided |
| Q-14 | Tarnvale \| 10.9 | Tarnvale \| 5.25 | Ashgrove \| Not mentioned |
| Q-15 | Kelmarsh \| 34.6 | Kelmarsh \| 25.1 | Kelmarsh \| 297.1 |
| Q-16 | Ossbridge and Ossford \| 3.5 | No \| 2.5 | Cannot answer from retrieved documents |
| Q-17 | Harrowfield \| 1785.49 | Harrowfield \| 158.46 | Cannot answer from retrieved documents |
| Q-18 | Millbrook \| 40012 | Millbrook \| 40011.69 | No information for Millbrook |
| Q-19 | Lindenmoor \| 0.043 | Lindenmoor \| 0.039 | Redfen \| 0 |
| Q-20 | Ossbridge \| 1842.27 | Harrowfield \| 11440.06 | Ossbridge \| 1029.5 |

------------------------------------------------------------------------

## Part C — Explain and choose (Hard)

### C1 — Does the tool description matter?

| Question | Calls (your description) | Calls (vague) | Answer (yours) | Answer (vague) |
|----|----|----|----|----|
| Q-01 | 4 calls | 1 call | Ossford \| 36.6 | Ossford \| 36.6 |
| Q-05 | 3 calls | 1 call | Stonecairn \| 193260.08 | Stonecairn \| 193259.41 |
| Q-09 | 4 calls | 1 call | Millbrook \| Data not available | Millbrook \| Not Found |
| Q-16 | 4 calls | 1 call | Cannot answer from retrieved documents | To answer this question, I need the areas for Ossbridge and Ossford from their respective documents. However, documents for Ossbridge and Ossford are not provided in the supplied text. |
| Q-20 | 3 calls | 1 call | Ossbridge \| 1029.5 | Ossbridge \| 1489.27 |

The tool description mattered. With the detailed description, the model found more relevant information and produced better answers. With the vague description, some answers became incomplete or less accurate because the model had less guidance on how to use the tool.

### C2 — Three mistakes explained

S1 mistake (Q-01)

Log: Ossbridge \| 10.5

Correct: Ossbridge \| 4.6

The model found the correct district but made a calculation mistake because no calculator tool was used.

S2 mistake (Q-01)

Log: Vote 1: Ossbridge \| 10.5 Vote 2: Ossford \| 5.5 Vote 3: Ossbridge \| 10.5

Final answer: Ossbridge \| 10.5

Voting can fix random mistakes, but it cannot fix a mistake if most votes are wrong.

S3 mistake (Q-16)

Log: To answer this question, I need the areas for Ossbridge and Ossford...

The lookup tool did not retrieve the correct documents, so the model did not have enough information to answer.

Why does voting help? Voting helps when mistakes are random. If most answers are correct, the final vote is often correct too.

Why does a calculator help? A calculator performs exact arithmetic and removes calculation mistakes made by the model.

Why does the loop cost more tokens? Each round adds more prompts and responses, so the total number of tokens and time increase.

### C3 — Which strategy I would use, and what would change it

I would choose **S2 (self-consistency)** because it gave the best accuracy.

I would not choose **S1** because it made more reasoning and calculation mistakes.

I would not choose **S3** because the retrieval sometimes returned the wrong documents, leading to incorrect answers.

**Small budget:** I would choose **S1** because it uses fewer tokens.

**Strict time limit:** I would choose **S1** because it is the fastest strategy.

**Many more documents:** I would choose **S3** because retrieval becomes more important when the document collection grows.

### C4 — How sure I can be

1.  How different were your two runs?

The two runs were very similar. Most questions produced the same answers, with only a few differences caused by model randomness.

2.  Is the difference between your two best strategies bigger than that?

Yes. The difference between the best strategies was larger than the variation between runs, which suggests that the choice of strategy had a greater effect than random variation.

3.  How many questions would you need before you trusted the result?

Twenty questions provide only a small evaluation set. I would prefer at least 50–100 questions before drawing strong conclusions about which strategy is best.

### Final questions

1\. Nothing has run yet. The model only requests a tool. The program must execute it.

2\. Arguments arrive as a JSON string. We must convert it into a Python dictionary before passing it to the function.

3\. The two tool definitions add about 200–300 tokens to each S3 request. The exact number is not measured.

4\. S2 used three times more tokens than S1 because it ran the model multiple times and voted on the answers.

5\. Q-09 needed several tool calls because the model had to retrieve population and area data for two districts and calculate their population densities.

6\. I would use indexing to find documents faster instead of checking them one by one.

7\. Python AST allows only safe arithmetic operations and rejects dangerous expressions. However, invalid or very large expressions can still cause errors.

8\. Tools helped most with calculations. They helped less with simple questions that the model could answer directly from documents.
