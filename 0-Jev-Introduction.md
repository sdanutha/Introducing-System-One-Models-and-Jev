# 0 : Jev Introduction

## What is Jev?

**Jev** is a TypeSafe **System One** model for software decisions. You send it a **State** (the information to review) and one or more **Questions**. It returns structured answers that your code can use.

```text
State + Questions → Jev → typed answers → application code
```

Jev is not a chatbot. It classifies, checks, or scores information; your code controls what happens next.

## Three question types

| Type | Use it for | Result |
|---|---|---|
| **Noul** | A yes/no question | `noul`: probability of “Yes” from 0 to 1 |
| **Choice** | Pick one option from a list | `choice`, `probabilities`, `confidence` |
| **Score** | Rate something on an ordered scale | `score`, `legend`, `probabilities`, `confidence` |

A **mode** means one of these question types. A **model** is what processes the request. Names such as `jev-latest` are model aliases, not new modes.

Probability and confidence show uncertainty. They do not guarantee that an answer is correct. Keep human review and application rules in control of real actions.

## Hands-on guides

1. [1-Jev-Beginner.md](1-Jev-Beginner.md) — Basic examples
2. [2-Jev-Developer.md](2-Jev-Developer.md) — IT, software, and data science
3. [3-Jev-NeoWork.md](3-Jev-NeoWork.md) — Sample NeoWork cases
4. [4-Jev-NeoWork-Advanced.md](4-Jev-NeoWork-Advanced.md) — Mix all three types in one request
