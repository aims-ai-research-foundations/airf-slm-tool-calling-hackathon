# African SLM Tool Calling Hackathon: dataset

> Can a small language model turn everyday requests from across Africa into the right tool call?

Each example is one English request from someone in Nigeria, Ghana, Kenya, Uganda, Tanzania, Rwanda, South Africa, Zambia or Ethiopia, and the 4–6 tools on offer: the tool (or tools) it needs, plus distractors. The model returns the tool to call and **every one of its parameters**, taking each value from the request and leaving it `null` when the request does not say it.

## Files

| File | What it holds |
|---|---|
| `tools.json` | The 10 tool schemas |
| `train.jsonl` | 15,000 requests with their answers, and scenario metadata (`meta`) |
| `dev.jsonl` | 2,000 requests without answers (dev leaderboard) |
| `test.jsonl` | 4,000 requests without answers, released for the final phase |
| `sample_submission.json` | The submission format |

## Format

A training example:

```json
{"id": "train-00042",
 "request": "Abeg send 5k naira to Tunde on MTN MoMo.",
 "tools": ["send_money", "check_balance", "buy_airtime", "convert_currency"],
 "tool": "send_money",
 "parameters": {"recipient": "Tunde", "amount": 5000, "currency": "NGN", "provider": "MTN MoMo"},
 "meta": {"country": "NG", "city": "Lagos", "style": "local", "category": "single", ...}}
```

A request that needs several actions has `tool_calls`, a list of calls in the order they should run, instead of `tool` and `parameters`:

```json
{"id": "train-00107",
 "request": "Check my M-Pesa balance, then send 500 bob from it to Otieno.",
 "tools": ["send_money", "check_balance", "convert_currency", "buy_data", "report_outage"],
 "tool_calls": [{"tool": "check_balance", "parameters": {"provider": "M-Pesa"}},
                {"tool": "send_money", "parameters": {"recipient": "Otieno", "amount": 500, "currency": "KES", "provider": "M-Pesa"}}]}
```

Dev and test have only `id`, `request` and `tools`. For each one, predict an **answer**: one call, `{"tool": ..., "parameters": {...}}`, or a list of calls when the request needs several actions.

## Rules for the correct answer

1. **Pick the tool that does what the user asks**, from the tools offered. Every request can be done with the offered tools; the others are distractors.
2. **Give every parameter of the tool. Every value comes from the request; use `null` for a value the request does not say.** Never infer, guess or fill in a default: "buy airtime for me" has no phone number, "will it rain in Gulu?" has no date, and "remind me to pray tomorrow morning" has no time.
3. **Take values as the user wrote them**, in the formats below. Greetings, forms of address and small talk ("Oga,", "Chale,", "I'm a nurse.", "It's for the rent.") never give a value.
4. **Currency** (`send_money`, `buy_electricity_token`) is the ISO code of the currency written with the amount: ₦5,000, 500 bob, R200 and $50 are NGN, KES, ZAR and USD. **When no currency is written with the amount, it is `null`**, even if a place, wallet, name or greeting suggests a country: "Abeg send 5k to Tunde on MTN MoMo" and "Send 2,000 to Wanjiru in Nakuru" both have currency `null`. With no amount, the currency is `null` too. Electricity companies and brands (KPLC, Eskom, LUKU, ECG) are not a parameter. The other tools have no currency parameter, so a currency written there only tells you the amount ("ETB 500 data" has amount 500).
5. **Converting money**: `amount` and `from_currency` are the money the user has; `to_currency` is the money they want. Both currencies must be named: "how much is $100 in my local money?" has `to_currency` null. A rate question with no amount converts from the first currency to the second: "the dollar to naira rate" is USD to NGN, with amount `null`.
6. **Recipient**: the person's name as written, without relationship words or titles (Mr, Mrs, Dr, Pastor, Aunty, Uncle, Madam). "my sister Amaka" is `Amaka`, "Mr Okafor" is `Okafor`, "Aunty Bisi" is `Bisi`, and a full name stays full (`Kwame Mensah`). A relationship or title alone ("my landlord", "my pastor") is not a name, so it is `null`; "my landlord Tiyamike" is `Tiyamike`. A phone number used as the recipient is written as a phone number.
7. **Places**: the most specific place named for that action. "Yaba, Lagos" is `Yaba`, and a market is named without the word "market" ("Mile 12 market" is `Mile 12`, "Gulu Main market" is `Gulu Main`). A value given in passing counts: "I'm in Gulu. Will it rain here tomorrow?" is `Gulu`, and in "I'm going to Narok tomorrow. Will it rain there, and what's the price of maize?" both calls use `Narok` and the weather is for `tomorrow`.
8. **Several actions** become a list of calls in the order asked, one call per action: "Send 2k to Kemi and 3k to Bisi" is two `send_money` calls. A value said once for both applies to both: "both on MTN", "Check my OPay and PalmPay balances", "the price of maize and beans in Eldoret", "Tomorrow, remind me to … at 7am and … at 6pm", "check my M-Pesa balance, then send 500 from it". A value said for one action only stays with that action. Two places, items, days or currencies joined by "and" are two actions: "Will it rain in Gulu and Lira?" and "the weather in Rusizi tomorrow and on Thursday" are two `get_weather` calls; "convert 10,000 rand to cedis and to GBP" is two `convert_currency` calls.
9. **Airtime or data**: airtime is also called credit or call credit, and recharging or topping up a phone line means airtime. When the user says data, a bundle, internet, or a size in MB or GB, it is data ("top up my phone with data" is data). The `amount` is the money spent, not the bundle size: "the 2GB bundle for 1,000" has amount 1000. Recharging or topping up a **meter** is electricity.
10. **Reminder task**: what to do, word for word as written (keeping "my", "the" and pronouns), without the asking words ("remind me to", "make sure I remember to", "nudge me to", "ping me … so I remember to") or the day and time. "Remind me to pay my rent on Friday at 9am" has task `pay my rent`; "I keep forgetting to call them. Remind me tomorrow" has `call them`; "a reminder for the chama contribution" has `the chama contribution`, and "a reminder for my job interview" has `my job interview`.

## Value formats

| Value | Format | Examples |
|---|---|---|
| Amounts | A number, without the currency | "5k", "5,000", "₦5,000", "five thousand" → `5000`; "1.5 million" → `1500000` |
| Currencies | ISO code | see the table below |
| Phone and meter numbers | Digits as written, without spaces or dashes; keep a leading `+` | "0803 123 4567" → `"08031234567"`, "+254 712 345 678" → `"+254712345678"` |
| Dates | `today`, `tomorrow`, a weekday in lowercase, or `D Month` with the month in full | "tonight", "this morning", "this afternoon", "this evening" → `today`; "tmrw" → `tomorrow`; "this Fri" → `friday`; "15th March", "March 15", "the 15th of Mar" → `15 March`; "6th Sep" → `6 September` |
| Times | 24-hour `HH:MM` | "7pm" → `19:00`; "7.30am" → `07:30`; "12:00 p.m.", "noon", "midday" → `12:00`; "8 o'clock in the morning" → `08:00`; "half past 6 in the evening" → `18:30`; "morning" alone is not a time (`null`) |
| Wallets (`provider`) and networks | As written | "MoMo" → `MoMo`; "MTN Mobile Money" → `MTN Mobile Money`; "Saf" → `Saf` ("Mpesa" and "M-Pesa" match, since punctuation is ignored) |
| Crops | The crop as written, without quantities | "a bag of maize" → `maize`; `sukuma wiki`; `tomatoes` |
| Places, names, tasks | As written (rules 6, 7 and 10) | `Kariakoo`, `Dar`, `Kwame Mensah`, `pay my rent` |

### Currencies

| Currency | ISO code | Written as |
|---|---|---|
| Nigerian naira | `NGN` | naira, ₦, N (N5,000) |
| Ghanaian cedi | `GHS` | cedis, Ghana cedis, GH₵, GHC, GHS |
| Kenyan shilling | `KES` | Kenyan (or Kenya) shillings, KSh, Ksh, KES, bob |
| Ugandan shilling | `UGX` | Ugandan (or Uganda) shillings, UGX, USh |
| Tanzanian shilling | `TZS` | Tanzanian shillings, TSh, TZS |
| Rwandan franc | `RWF` | Rwandan francs, RWF, Frw |
| South African rand | `ZAR` | rand, R (R200), ZAR |
| Zambian kwacha | `ZMW` | kwacha, K or k before the number (K50, k150), ZMW; "150k" is 150,000 |
| Ethiopian birr | `ETB` | birr, Ethiopian birr, ETB |
| US dollar, euro, British pound | `USD`, `EUR`, `GBP` | dollars, $; euros, €; pounds, £ |

Currency words and symbols are read in any case: chat-style requests write "r5k", "n5,000" or "ksh 200" for R5k, N5,000 and KSh 200.

## Scoring

**Exact-match accuracy**: a prediction is correct only when the tool and all of its parameters match. For a request with several actions, the whole list must match, in order.

- A list of one call is the same as the call itself.
- A parameter you leave out counts as `null`.
- Numbers are compared as numbers (`5000` and `5000.0` match; the text `"5000"` does not).
- Text ignores case, punctuation and extra spaces, so `"m-pesa"` matches `"M-Pesa"`.

## How the data was made

Every example starts from a scenario: a country, a city, a persona, the tool or tools, which values are stated, and a language style (formal, casual, local greetings, chat shorthand, short, or detailed with some background). The answer is fixed before any text is written, and every answer is checked against its request: each value must be stated in it, and each `null` must have nothing in the request that could fill it. All names, numbers and codes are synthetic.

| Category | Share | Example |
|---|---:|---|
| `single` | 60% | "Will it rain in Kumasi tomorrow?" |
| `missing_info` | 20% | "Send 2,000 naira to Tunde." (`provider` is `null`) |
| `multiple` | 20% | "Buy 500 airtime and 1,000 data for 0803 123 4567 on MTN." |

`meta` in `train.jsonl` describes each scenario (`category`, `kind`, `country`, `city`, `style`, `n_calls`, `null_params`), so you can see what kinds of examples exist and generate more of your own.

**Dev and test use different sentence templates, greetings, names, neighbourhoods, towns and markets from train**, so a model has to generalise rather than memorise phrasings.
