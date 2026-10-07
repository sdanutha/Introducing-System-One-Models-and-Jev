# Hands-on 1: Jev for Beginners

Try six short examples in the [TypeSafe Playground](https://console.typesafe.ai/). Each mode has one Single and one Multi example.

1. Paste each **STATE** block into State.
2. Paste each **QUESTIONS** block into Questions / Prompts.
3. Select Run / Evaluate. Answers appear under each question ID.

---

# Mode 1 — Noul

## Noul 1 — Single: Does the text mention a cat?

### STATE

```json
{
  "text": "A cat is sleeping on the chair."
}
```

### QUESTIONS

```json
{
  "mentions_cat": {
    "type": "noul",
    "instructions": "Does the text mention a cat?"
  }
}
```

**Result:** Check `answers.mentions_cat.noul`. Near 1 means Yes is more likely; near 0 means No.

## Noul 2 — Multi: Check a message and policy

### STATE

```json
{
  "ticket": {
    "message": "My order arrived late. What are my options?"
  },
  "policy": "Late deliveries may be eligible for a refund."
}
```

### QUESTIONS

```json
{
  "customer_requested_refund": {
    "type": "noul",
    "instructions": "Does `ticket.message` explicitly request a refund?"
  },
  "policy_mentions_refund": {
    "type": "noul",
    "instructions": "Does `policy` say that a refund may be available?"
  },
  "customer_reports_late_delivery": {
    "type": "noul",
    "instructions": "Does `ticket.message` say that the order arrived late?"
  }
}
```

**Result:** Check each question ID. They test separate facts in the same State.

---

# Mode 2 — Choice

## Choice 1 — Single: What color is the sky?

### STATE

```json
{
  "observation": "The clear daytime sky appears blue."
}
```

### QUESTIONS

```json
{
  "sky_color": {
    "type": "choice",
    "instructions": "What color does the observation say the sky appears?",
    "criteria": {
      "blue": "The sky is described as blue",
      "green": "The sky is described as green",
      "red": "The sky is described as red"
    }
  }
}
```

**Result:** Check `answers.sky_color.choice`, `probabilities`, and `confidence`.

## Choice 2 — Multi: Classify a customer message

### STATE

```json
{
  "message": "My headphones arrived broken. I would like a replacement."
}
```

### QUESTIONS

```json
{
  "issue_type": {
    "type": "choice",
    "instructions": "What is the main issue described in the message?",
    "criteria": {
      "damaged_item": "The item arrived broken or damaged",
      "shipping_delay": "The delivery is late or has not arrived",
      "billing_problem": "There is an issue with a charge or payment",
      "other": "The issue does not fit the listed options"
    }
  },
  "requested_resolution": {
    "type": "choice",
    "instructions": "What resolution does the customer ask for?",
    "criteria": {
      "replacement": "Send the same item again",
      "refund": "Return the money",
      "repair": "Repair the existing item",
      "information": "The customer asks a question but requests no action"
    }
  },
  "tone": {
    "type": "choice",
    "instructions": "What tone does the customer use?",
    "criteria": {
      "calm": "Neutral or polite wording",
      "frustrated": "Clearly dissatisfied but not strongly angry",
      "angry": "Strongly angry or hostile wording"
    }
  }
}
```

**Result:** Compare `issue_type`, `requested_resolution`, and `tone` answers.

---

# Mode 3 — Score

## Score 1 — Single: How hot is the coffee?

### STATE

```json
{
  "coffee": "The coffee is steaming hot and too hot to drink right now."
}
```

### QUESTIONS

```json
{
  "temperature": {
    "type": "score",
    "instructions": "How hot is the coffee according to the text?",
    "criteria": [
      "0 - Cold: described as cold or chilled",
      "1 - Warm: warm but comfortable to drink",
      "2 - Hot: steaming or too hot to drink immediately"
    ]
  }
}
```

**Result:** Check `answers.temperature.score` and the related `legend`, `probabilities`, and `confidence`.

## Score 2 — Multi: Score a bug report

### STATE

```json
{
  "bug_report": "The mobile app crashes when I tap Save. It happened three times on Android 14. I do not know how to reproduce it reliably."
}
```

### QUESTIONS

```json
{
  "problem_detail": {
    "type": "score",
    "instructions": "How clearly does the report describe the problem?",
    "criteria": [
      "0 - Vague: only says something is wrong",
      "1 - Partial: names the problem but gives little detail",
      "2 - Clear: describes the failure and when it happens"
    ]
  },
  "reproduction_readiness": {
    "type": "score",
    "instructions": "How ready is the report for an engineer to reproduce the problem?",
    "criteria": [
      "0 - Not ready: no environment or reproduction information",
      "1 - Some information: environment or occurrence details are present, but steps are unclear",
      "2 - Ready: clear reproduction steps and relevant environment are provided"
    ]
  },
  "impact_level": {
    "type": "score",
    "instructions": "How serious is the reported impact on the user?",
    "criteria": [
      "0 - Low: cosmetic issue with no meaningful function impact",
      "1 - Medium: a feature is degraded but a workaround may exist",
      "2 - High: a key function is blocked and no workaround is stated"
    ]
  }
}
```

**Result:** Compare the scores for detail, reproduction readiness, and impact. A score is not a confirmed fact.

---

## Try it yourself

Change one State at a time and run its Questions again: ask about a refund, change the customer issue, or add clear bug reproduction steps.
