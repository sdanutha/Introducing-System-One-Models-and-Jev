# Hands-on 3: Sample NeoWork

Try six short examples in the [TypeSafe Playground](https://console.typesafe.ai/). Each mode has one Single and one Multi example.

1. Paste each **STATE** block into State.
2. Paste each **QUESTIONS** block into Questions / Prompts.
3. Select Run / Evaluate. Answers appear under each question ID.

---

# Mode 1 — Noul

## Noul 1 — Single: Does the case report a repeated event?

### STATE

```json
{
  "case": {
    "description": "Five lots are on hold after an APC termination failure. The event was observed again within 24 hours. The root cause has not been confirmed.",
    "repeat_within_24h": true
  }
}
```

### QUESTIONS

```json
{
  "event_repeated_within_24h": {
    "type": "noul",
    "instructions": "Does the case description say that the event was observed again within 24 hours?"
  }
}
```

**Result:** Check `answers.event_repeated_within_24h.noul`. It checks the text, not a factory system.

## Noul 2 — Multi: Check evidence gaps

### STATE

```json
{
  "case": {
    "description": "Five lots are on hold after an APC termination failure. The event was observed again within 24 hours. The root cause has not been confirmed.",
    "current_status": "HOLD"
  },
  "evidence": {
    "engineer_assessment": "missing",
    "production_impact": "unknown",
    "release_approval": "not_provided"
  }
}
```

### QUESTIONS

```json
{
  "root_cause_confirmed": {
    "type": "noul",
    "instructions": "Does the state provide a confirmed root cause for the case?"
  },
  "production_impact_documented": {
    "type": "noul",
    "instructions": "Does the state document a verified production impact?"
  },
  "release_approval_provided": {
    "type": "noul",
    "instructions": "Does the state provide an authorized release approval?"
  }
}
```

**Result:** Check root cause, production impact, and release approval separately. Missing here does not mean missing from a source system.

---

# Mode 2 — Choice

## Choice 1 — Single: Classify the case

### STATE

```json
{
  "case": {
    "title": "Multiple lots on SPC hold after APC error",
    "description": "Five lots are on hold after an APC termination failure. The event was observed again within 24 hours. The root cause has not been confirmed.",
    "hold_code": "SPC_Shutdown APC_TERMFAIL_AVG_LOW"
  }
}
```

### QUESTIONS

```json
{
  "case_type": {
    "type": "choice",
    "instructions": "Which category best describes the operational case, using only the supplied facts?",
    "criteria": {
      "SPC_HOLD": "A statistical process control or process-monitoring hold",
      "WIP_CAPACITY": "A work-in-progress group capacity or fullness constraint",
      "EQUIPMENT": "An equipment availability, failure, or qualification constraint",
      "DATA_QUALITY": "A data integrity, missing-data, or measurement-quality issue",
      "OTHER_OR_UNCLEAR": "The supplied facts do not support the other categories"
    }
  }
}
```

**Result:** Check `answers.case_type.choice`, `probabilities`, and `confidence`. This does not confirm the root cause.

## Choice 2 — Multi: Choose review steps and evidence gaps

### STATE

```json
{
  "case": {
    "title": "Multiple lots on SPC hold after APC error",
    "description": "Five lots are on hold after an APC termination failure. The event was observed again within 24 hours. The root cause has not been confirmed.",
    "current_status": "HOLD",
    "repeat_within_24h": true
  },
  "evidence": {
    "verified_engineer_assessment": "missing",
    "production_or_shipping_impact": "unknown",
    "containment_verification": "not_provided",
    "release_approval": "not_provided"
  },
  "safety_note": "This is a training example. Any operational decision requires qualified human review and authorized approval."
}
```

### QUESTIONS

```json
{
  "triage_severity": {
    "type": "choice",
    "instructions": "Classify triage severity using only impact facts explicitly present. Do not assume unknown production, customer, or shipping impact.",
    "criteria": {
      "LOW": "No material operational impact is stated; routine handling may be sufficient",
      "MEDIUM": "A limited issue or repeated event is stated, but broad impact is not established",
      "HIGH": "Material operational impact is explicitly supported by the supplied facts",
      "CRITICAL": "Severe and immediate production, safety, or customer impact is explicitly supported",
      "UNKNOWN": "Available facts do not support a reliable severity classification"
    }
  },
  "next_review_stage": {
    "type": "choice",
    "instructions": "Choose a review stage, not an operational action. Do not choose a stage that releases or changes the hold.",
    "criteria": {
      "INITIAL_TRIAGE": "Confirm case details and identify the responsible owner",
      "ENGINEERING_INVESTIGATION": "A responsible engineer should investigate evidence and possible cause",
      "PROMPT_HUMAN_ESCALATION": "A qualified owner should promptly assess explicitly supported high impact",
      "EVIDENCE_COMPLETION": "Collect or verify missing evidence before further review",
      "AUTHORIZED_CLOSURE_REVIEW": "Evidence is ready for an authorized person to review closure; this is not automatic closure"
    }
  },
  "most_important_evidence_gap": {
    "type": "choice",
    "instructions": "Choose the most important missing evidence category for the next review, based only on the state.",
    "criteria": {
      "ROOT_CAUSE": "Verified evidence testing or identifying the suspected root cause",
      "IMPACT_SCOPE": "Verified evidence of affected lots or production, customer, or shipping impact",
      "CONTAINMENT": "Evidence that containment was applied and verified",
      "ENGINEER_ASSESSMENT": "A qualified engineer's assessment or disposition rationale",
      "RELEASE_APPROVAL": "Required authorized approval for release or closure",
      "OTHER_OR_UNCLEAR": "A different gap is present or the supplied facts do not identify one clearly"
    }
  }
}
```

**Result:** Review severity, next review stage, and evidence gap separately. These are not operational commands.

---

# Mode 3 — Score

## Score 1 — Single: Is the information ready for RCA review?

### STATE

```json
{
  "case": {
    "description": "Five lots are on hold after an APC termination failure. The event was observed again within 24 hours. The root cause has not been confirmed.",
    "current_status": "HOLD"
  },
  "evidence": {
    "engineer_assessment": "missing",
    "production_impact": "unknown",
    "containment_verification": "not_provided"
  }
}
```

### QUESTIONS

```json
{
  "rca_evidence_readiness": {
    "type": "score",
    "instructions": "How ready is the supplied case information for an authorized root-cause-analysis review? Rate evidence completeness, not the probability that any suspected cause is correct.",
    "criteria": [
      "Not ready: key evidence, impact details, or verification is missing",
      "Partly ready: some useful facts are present, but important evidence or validation is still missing",
      "Review-ready: relevant evidence, impact, and verification are documented for an authorized review"
    ]
  }
}
```

**Result:** Check `answers.rca_evidence_readiness.score` and its related fields. It does not prove the RCA or approve case closure.

## Score 2 — Multi: Score three evidence areas

### STATE

```json
{
  "case": {
    "description": "Five lots are on hold after an APC termination failure. The event was observed again within 24 hours. The root cause has not been confirmed.",
    "current_status": "HOLD"
  },
  "evidence": {
    "affected_lots": "Five lots are reported on hold; no source extract is attached.",
    "production_impact": "Unknown; no impact assessment is included.",
    "containment": "No containment verification is included.",
    "engineer_assessment": "No verified engineer assessment is included."
  }
}
```

### QUESTIONS

```json
{
  "impact_documentation": {
    "type": "score",
    "instructions": "How complete is the documentation of operational impact in the supplied state?",
    "criteria": [
      "Insufficient: affected scope or impact is not described",
      "Partial: some affected scope is stated, but verified operational impact is missing",
      "Complete: affected scope and verified production, customer, or shipping impact are documented"
    ]
  },
  "containment_verification": {
    "type": "score",
    "instructions": "How complete is the evidence that containment was applied and verified?",
    "criteria": [
      "Insufficient: no containment action or verification is documented",
      "Partial: an action is mentioned, but completion or verification is unclear",
      "Complete: containment action, scope, and verification evidence are documented"
    ]
  },
  "technical_assessment": {
    "type": "score",
    "instructions": "How complete is the qualified technical assessment for an RCA review?",
    "criteria": [
      "Insufficient: no qualified assessment or supporting evidence is provided",
      "Partial: an assessment is mentioned, but supporting evidence or validation is incomplete",
      "Complete: a qualified assessment is supported by documented evidence and validation"
    ]
  }
}
```

**Result:** Compare impact, containment, and technical assessment. Ask a responsible person to check unclear results.

---

## Try it yourself

Change one State detail at a time: add sample impact evidence, an engineer assessment, or containment verification. Then run the same Questions. Use synthetic data only; people and application code must control real actions.
