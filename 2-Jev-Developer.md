# Hands-on 2: Jev for Developers

Try six short examples in the [TypeSafe Playground](https://console.typesafe.ai/). Each mode has one Single and one Multi example.

1. Paste each **STATE** block into State.
2. Paste each **QUESTIONS** block into Questions / Prompts.
3. Select Run / Evaluate. Answers appear under each question ID.

---

# Mode 1 — Noul

## Noul 1 — Single: Does the CI log report a test failure?

### STATE

```json
{
  "ci_log": "Build passed. Unit tests passed. Integration test test_payment_timeout failed: expected 200, got 504."
}
```

### QUESTIONS

```json
{
  "has_test_failure": {
    "type": "noul",
    "instructions": "Does the CI log report at least one test failure?"
  }
}
```

**Result:** Check `answers.has_test_failure.noul`. This checks the log, not the cause of the error.

## Noul 2 — Multi: Check a data quality note

### STATE

```json
{
  "dataset_note": "The table has 12,400 rows. The customer_id column is missing for 3% of rows. Event timestamps use UTC. The note does not describe how duplicate events are handled."
}
```

### QUESTIONS

```json
{
  "mentions_missing_values": {
    "type": "noul",
    "instructions": "Does the note explicitly mention missing values in a column?"
  },
  "states_timezone": {
    "type": "noul",
    "instructions": "Does the note specify the timezone used for event timestamps?"
  },
  "describes_duplicate_handling": {
    "type": "noul",
    "instructions": "Does the note explain how duplicate events are handled?"
  }
}
```

**Result:** Check the three IDs separately. They do not prove that the dataset has no other problems.

---

# Mode 2 — Choice

## Choice 1 — Single: Classify an incident note

### STATE

```json
{
  "incident": "The API started returning 503 after the database connection pool reached its configured limit. Restarting the API temporarily restored service."
}
```

### QUESTIONS

```json
{
  "primary_area": {
    "type": "choice",
    "instructions": "Which area is most directly implicated by the incident note?",
    "criteria": {
      "database_capacity": "Database connections or database capacity are the primary issue",
      "application_bug": "Application logic or an application defect is the primary issue",
      "network": "Network connectivity or routing is the primary issue",
      "unknown": "The note does not support choosing one of the listed areas"
    }
  }
}
```

**Result:** Check `answers.primary_area.choice`, `probabilities`, and `confidence`. This is not root-cause analysis.

## Choice 2 — Multi: Classify a pull request

### STATE

```json
{
  "pull_request": "Add an index on events.created_at and update the dashboard query to filter by time range. Includes a migration and query-plan check."
}
```

### QUESTIONS

```json
{
  "change_area": {
    "type": "choice",
    "instructions": "What is the main engineering area of this pull request?",
    "criteria": {
      "database": "Schema, index, migration, or database-query change",
      "frontend": "User interface or browser-side change",
      "infrastructure": "Deployment, networking, or runtime-platform change",
      "documentation": "Documentation-only change",
      "other": "The summary does not fit the listed areas"
    }
  },
  "risk_kind": {
    "type": "choice",
    "instructions": "What kind of review risk is most relevant based on the summary?",
    "criteria": {
      "performance": "Query latency, resource use, or performance regression",
      "data_migration": "Migration correctness, locking, or compatibility of stored data",
      "user_interface": "Visual or interaction regression in the user interface",
      "no_clear_risk": "The summary does not indicate a specific risk area"
    }
  },
  "change_scope": {
    "type": "choice",
    "instructions": "How broad is the described change?",
    "criteria": {
      "narrow": "A localized change affecting one small component or query",
      "moderate": "Several related parts of one service or workflow",
      "broad": "Multiple services or unrelated areas are changed"
    }
  }
}
```

**Result:** Review `change_area`, `risk_kind`, and `change_scope` separately.

---

# Mode 3 — Score

## Score 1 — Single: Score a bug report

### STATE

```json
{
  "bug_report": "Export fails in Safari for large reports. Export still works in Chrome. No data loss has been observed."
}
```

### QUESTIONS

```json
{
  "bug_severity": {
    "type": "score",
    "instructions": "How severe is the user impact described in this bug report?",
    "criteria": [
      "Cosmetic: no meaningful functionality is affected",
      "Limited: a feature is degraded, but a practical workaround is available",
      "Blocking: a key function is unavailable and no workaround is stated"
    ]
  }
}
```

**Result:** Check `answers.bug_severity.score` with its `legend`, `probabilities`, and `confidence`. It does not confirm impact for every user.

## Score 2 — Multi: Score an experiment summary

### STATE

```json
{
  "experiment": "Model v2 was evaluated against the existing baseline on a held-out set of 8,000 records. The report says the metric improved, but does not name the metric, provide a value, or describe the split strategy."
}
```

### QUESTIONS

```json
{
  "method_detail": {
    "type": "score",
    "instructions": "How clearly does the summary describe the evaluation method?",
    "criteria": [
      "Low: no evaluation method or dataset is described",
      "Partial: an evaluation set or comparison is mentioned, but key method details are missing",
      "Clear: the evaluation set, comparison, and key method details are stated"
    ]
  },
  "result_reproducibility": {
    "type": "score",
    "instructions": "How reproducible are the reported results from the information in the summary?",
    "criteria": [
      "Low: no metric or usable result is reported",
      "Partial: a result or metric is mentioned, but important values or setup details are missing",
      "High: metrics, values, data split, and enough setup detail are provided to repeat the comparison"
    ]
  },
  "claim_specificity": {
    "type": "score",
    "instructions": "How specific is the claim about model performance?",
    "criteria": [
      "Low: only a vague claim such as 'better' is made",
      "Partial: a direction or metric is named, but quantitative evidence is incomplete",
      "High: the metric, measured values, comparison, and evaluation context are stated"
    ]
  }
}
```

**Result:** Compare method detail, reproducibility, and claim specificity. Do not combine the scores automatically.

---

## Try it yourself

Change the CI log, add evidence to the incident, or add metric and data-split details to the experiment. Compare the results.
