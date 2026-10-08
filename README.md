# African SLM Tool Calling Hackathon

> **Can a small language model turn everyday requests from across Africa into the right actions?**

AI assistants are moving beyond simply answering questions.

Instead of telling you how to buy airtime, an assistant can **buy the airtime for you**. Instead of explaining how to send money, it can **send the money**. Instead of telling you tomorrow's weather, it can **look it up for you**.

To do this, an AI assistant needs to know how to use **tools**.

A tool is simply a function that lets an AI system take an action or retrieve information. For example:

```text
send_money(recipient, amount, currency, provider)
```

The challenge is that the model has to figure out **which tool to use and what information to give it**.

That's what you will build.

Your task is to train a **small language model** that can take an everyday request, look at the tools available to it, and produce the correct tool call.

And the requests won't be generic.

They will come from everyday life across Africa:

> "Abeg send 5k naira to Tunde on MTN MoMo."

> "Buy 1,000 naira airtime for this number."

> "What's the maize price around Nakuru today?"

> "How much is 50,000 naira in Kenyan shillings?"

> "Check my M-Pesa balance, then send 500 bob from it to Otieno."

The model must understand the request, select the right tool, extract the information it needs, and return a structured call.

## The Challenge

For every request, your model receives a small set of available tools.

Some tools are relevant. Others are distractors.

For example:

**Request**

> "Abeg send 5k naira to Tunde on MTN MoMo."

**Available tools**

```text
send_money
check_balance
buy_airtime
convert_currency
```

The model should recognise that this is a `send_money` request and extract the relevant information.

The expected output is:

```json
{
  "tool": "send_money",
  "parameters": {
    "recipient": "Tunde",
    "amount": 5000,
    "currency": "NGN",
    "provider": "MTN MoMo"
  }
}
```

That's the entire challenge:

> **Given a request and a set of tools, make the correct tool call.**

## Why Small Language Models?

Large language models can be very capable at tool use. But they are also expensive and difficult to run in resource-constrained environments.

This hackathon asks a different question:

> **How capable can a small language model become when we train it specifically for tool calling?**

You will work with models of up to **4 billion parameters** and experiment with techniques such as fine-tuning, synthetic data generation, prompting and inference strategies.

The goal is not to build the biggest model.

The goal is to build the **most accurate small model**.

## Rules

- Use any open-weight model with **at most 4B parameters**. Gemma 3 270M, 1B and 4B, and FunctionGemma 270M are recommended.
- No API calls or larger models may be used when generating predictions.
- Any training technique is allowed, including fine-tuning, synthetic data generation, prompting and decoding strategies.
- You may augment the training data.
- You may not manually label development or test examples.

# Dataset

The dataset is deliberately simple.

Each training example contains:

```text
request
available tools
correct tool
parameters
```

The dataset teaches the model to map **English-language requests in African contexts** to the appropriate tool and its parameters.

## Tool Schemas

Before the competition, you receive the schemas for the tools available in the benchmark.

A tool schema tells the model what a tool does and what parameters it expects.

For example:

```json
{
  "name": "send_money",
  "description": "Send money to another person.",
  "parameters": {
    "recipient": "string",
    "amount": "number",
    "currency": "string",
    "provider": "string"
  }
}
```

The complete benchmark contains **10 tools**:

```json
[
  {
    "name": "send_money",
    "description": "Send money to another person.",
    "parameters": {
      "recipient": "string",
      "amount": "number",
      "currency": "string",
      "provider": "string"
    }
  },
  {
    "name": "buy_airtime",
    "description": "Buy mobile airtime.",
    "parameters": {
      "phone": "string",
      "amount": "number",
      "network": "string"
    }
  },
  {
    "name": "buy_data",
    "description": "Buy a mobile data bundle.",
    "parameters": {
      "phone": "string",
      "amount": "number",
      "network": "string"
    }
  },
  {
    "name": "check_balance",
    "description": "Check a mobile money account balance.",
    "parameters": {
      "provider": "string"
    }
  },
  {
    "name": "buy_electricity_token",
    "description": "Purchase an electricity token.",
    "parameters": {
      "meter_number": "string",
      "amount": "number",
      "currency": "string"
    }
  },
  {
    "name": "report_outage",
    "description": "Report an electricity outage.",
    "parameters": {
      "location": "string"
    }
  },
  {
    "name": "get_crop_price",
    "description": "Get the current market price of a crop.",
    "parameters": {
      "crop": "string",
      "location": "string"
    }
  },
  {
    "name": "get_weather",
    "description": "Get the weather forecast for a location.",
    "parameters": {
      "location": "string",
      "date": "string"
    }
  },
  {
    "name": "convert_currency",
    "description": "Convert an amount from one currency to another.",
    "parameters": {
      "amount": "number",
      "from_currency": "string",
      "to_currency": "string"
    }
  },
  {
    "name": "set_reminder",
    "description": "Set a reminder for the user.",
    "parameters": {
      "task": "string",
      "date": "string",
      "time": "string"
    }
  }
]
```

Each individual example contains **4–6 of these tools**, including the relevant tool and distractors.

## Training Examples

The training data is intentionally simple.

### Example 1

```json
{
  "request": "Abeg send 5k naira to Tunde on MTN MoMo.",
  "tools": [
    "send_money",
    "check_balance",
    "buy_airtime",
    "convert_currency"
  ],
  "tool": "send_money",
  "parameters": {
    "recipient": "Tunde",
    "amount": 5000,
    "currency": "NGN",
    "provider": "MTN MoMo"
  }
}
```

### Example 2

```json
{
  "request": "Buy 1,000 naira airtime for this number 08031234567 on Airtel.",
  "tools": [
    "buy_airtime",
    "buy_data",
    "send_money",
    "check_balance"
  ],
  "tool": "buy_airtime",
  "parameters": {
    "phone": "08031234567",
    "amount": 1000,
    "network": "Airtel"
  }
}
```

### Example 3

```json
{
  "request": "What's the maize price around Nakuru today?",
  "tools": [
    "get_crop_price",
    "get_weather",
    "convert_currency",
    "set_reminder"
  ],
  "tool": "get_crop_price",
  "parameters": {
    "crop": "maize",
    "location": "Nakuru"
  }
}
```

### Example 4

```json
{
  "request": "How much is 50,000 naira in Kenyan shillings?",
  "tools": [
    "convert_currency",
    "send_money",
    "get_crop_price",
    "get_weather"
  ],
  "tool": "convert_currency",
  "parameters": {
    "amount": 50000,
    "from_currency": "NGN",
    "to_currency": "KES"
  }
}
```

### Example 5: Multiple Tool Calls

Some requests require several actions.

```json
{
  "request": "Check my M-Pesa balance, then send 500 bob from it to Otieno.",
  "tools": [
    "check_balance",
    "send_money",
    "buy_airtime",
    "buy_data"
  ],
  "tool_calls": [
    {
      "tool": "check_balance",
      "parameters": {
        "provider": "M-Pesa"
      }
    },
    {
      "tool": "send_money",
      "parameters": {
        "recipient": "Otieno",
        "amount": 500,
        "currency": "KES",
        "provider": "M-Pesa"
      }
    }
  ]
}
```

For multiple calls, the calls must be returned **in the correct order**.

## Missing Information

Not every request contains everything a tool requires.

For example:

> "Send 2,000 naira to Tunde."

The request does not specify the provider.

In the training data, the missing value is represented as `null`:

```json
{
  "request": "Send 2,000 naira to Tunde.",
  "tools": [
    "send_money",
    "check_balance",
    "buy_airtime",
    "convert_currency"
  ],
  "tool": "send_money",
  "parameters": {
    "recipient": "Tunde",
    "amount": 2000,
    "currency": "NGN",
    "provider": null
  }
}
```

**A model should never invent information that is not in the request.**

## African Contexts

The language of the benchmark is **English**, but the world represented in the data is African.

Requests span:

- Nigeria
- Ghana
- Kenya
- Uganda
- Tanzania
- Rwanda
- South Africa
- Zambia
- Ethiopia

Examples vary across:

- Countries, cities, towns and neighbourhoods
- Names, relationships and titles
- Currencies and amounts
- Mobile money providers and networks
- Electricity providers
- Agricultural markets
- Dates and times
- Formal and informal English
- Local expressions and chat shorthand
- Short and detailed requests

For example:

> "Abeg send 5k to Tunde."

> "Please transfer R200 to Thandi using Vodacom."

> "Can you check whether it'll rain in Kumasi tomorrow?"

> "How much is maize going for around Nakuru?"

> "Set a reminder for me to call Mum at 6pm tomorrow."

The **dataset structure stays simple**. The diversity comes from the situations and language used in the requests.

## Dataset Split

| Split | Examples | Participants receive |
|---|---:|---|
| **Train** | 15,000 | Request + tools + answer |
| **Dev** | 2,000 | Request + tools |
| **Test** | 4,000 | Request + tools |

The answers for **dev and test remain hidden**.

The dev and test sets use different names, neighbourhoods, towns, markets and wording from the training data. The aim is to test whether your model has **learned tool calling rather than memorised examples**.

The full labelling rules and value formats (amounts as numbers, ISO currency codes, phone numbers as digits, dates such as `15 March`, times such as `19:00`) are in the dataset README, [`data/README.md`](data/README.md).

# Evaluation

There is **one metric**:

## Exact-Match Accuracy

A prediction is correct only when the selected tool and **all of its parameters** match the ground truth.

For multiple tool calls, **every call must be correct and in the correct order**.

For example, the correct answer is:

```json
{
  "tool": "send_money",
  "parameters": {
    "recipient": "Tunde",
    "amount": 5000,
    "currency": "NGN",
    "provider": "MTN MoMo"
  }
}
```

A prediction with the correct tool but `amount: 500` is wrong.

A prediction with the correct tool and amount but the wrong provider is wrong.

A prediction that adds a value not present in the request is wrong.

When answers are compared:

- Text ignores case, punctuation and extra spaces, so `"m-pesa"` matches `"M-Pesa"`.
- Numbers are compared as numbers: `5000` matches `5000.0`, but the text `"5000"` does not.
- A parameter left out counts as `null`.
- A list containing one call is the same as that call.

### Score

```text
Exact-Match Accuracy =
Completely correct predictions / Total predictions
```

For example, 3,400 completely correct predictions out of 4,000 test examples gives:

**85% Exact-Match Accuracy**

> **One metric. One leaderboard. Highest exact-match accuracy wins.**

# Competition

## Phase 1: Development

Train your model using `train.jsonl`.

Use `dev.jsonl` to test your approach and submit predictions to the public leaderboard.

Experiment with:

- Fine-tuning
- LoRA / QLoRA
- Synthetic data augmentation
- Data curation
- Prompting
- Self-consistency
- Ensembling
- Other techniques you believe will improve tool-calling accuracy

## Phase 2: Final

Submit predictions for the hidden `test.jsonl`.

The final leaderboard is determined by **Exact-Match Accuracy on the test set**.

# Submission

Submit a `predictions.json` file containing the prediction for every example.

For example:

```json
{
  "dev-01234": {
    "tool": "send_money",
    "parameters": {
      "recipient": "Aisha",
      "amount": 2500,
      "currency": "NGN",
      "provider": "Airtel Money"
    }
  },
  "dev-01235": [
    {
      "tool": "check_balance",
      "parameters": {
        "provider": "M-Pesa"
      }
    },
    {
      "tool": "send_money",
      "parameters": {
        "recipient": "Otieno",
        "amount": 500,
        "currency": "KES",
        "provider": "M-Pesa"
      }
    }
  ]
}
```

Place `predictions.json` at the top level of your submission ZIP file.

The [starter notebook](starter_notebook.ipynb) will provide a baseline model and show how to train it and generate a submission.

# The Question

Large models can call tools.

**But can a small model learn to do it reliably?**

That's your challenge.

**Train it. Test it. Make it call the right tool.**
