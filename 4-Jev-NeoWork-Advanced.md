# Hands-on 4: Advanced NeoWork

Run one sample case with one State and one Questions set. Questions use Noul, Choice, and Score together.

This is fictional training data, not real NeoWork or factory data. Never use these results to release a hold, approve a release, close a case, or control production.

## How to use the Playground

1. Open the [TypeSafe Playground](https://console.typesafe.ai/).
2. Paste **STATE** into State and **QUESTIONS** into Questions / Prompts. If the Playground asks for one question at a time, add each question by its ID.
3. Select Run / Evaluate. Find each answer by its question ID.

## STATE

```json
{
  "case": {
    "case_id": "NW-DEMO-ADV-001",
    "site": "NRM",
    "title": "Multiple lots on SPC hold after APC error",
    "description": "Five lots are on hold after an APC termination failure. The event was observed again within 24 hours. The root cause has not been confirmed.",
    "lot_count_reported": 5,
    "hold_reason": "ENGINEERING",
    "hold_code": "SPC_Shutdown APC_TERMFAIL_AVG_LOW",
    "current_status": "HOLD",
    "repeat_within_24h_reported": true
  },
  "evidence": {
    "hold_snapshot": {
      "status": "listed_as_available",
      "contents": "No real Oracle extract is included; this is only a sample source label."
    },
    "wip_snapshot": {
      "status": "listed_as_available",
      "contents": "No real WIP data is included; this is only a sample source label."
    },
    "engineer_assessment": {
      "status": "missing",
      "contents": null
    },
    "root_cause_verification": {
      "status": "not_provided",
      "contents": null
    },
    "production_or_customer_impact": {
      "status": "unknown",
      "contents": null
    },
    "containment_verification": {
      "status": "not_provided",
      "contents": null
    },
    "release_approval": {
      "status": "not_provided",
      "contents": null
    }
  },
  "known_facts": [
    "The case description reports five lots on hold.",
    "The stated current status is HOLD.",
    "The case description reports that the event was observed again within 24 hours.",
    "The case description says the root cause has not been confirmed."
  ],
  "unknowns": [
    "The actual production, customer, or shipping impact is unknown.",
    "No verified engineer assessment is provided.",
    "No root-cause verification or containment verification is provided.",
    "No authorized release approval is provided."
  ],
  "safety_constraints": [
    "This is synthetic training data, not a real production case.",
    "Do not release, disposition, close, or change production status from this model assessment.",
    "A qualified human must verify evidence and authorize any operational action."
  ]
}
```

## QUESTIONS

```json
{
  "case_type": {
    "type": "choice",
    "instructions": "Which category best describes `case.hold_code` and `case.description`, using only the supplied state? Do not infer a root cause.",
    "criteria": {
      "SPC_HOLD": "A statistical process control or process-monitoring hold",
      "WIP_CAPACITY": "A work-in-progress group capacity or fullness constraint",
      "EQUIPMENT_CONSTRAINT": "An equipment availability, failure, or qualification constraint",
      "DATA_QUALITY": "A data integrity, missing-data, or measurement-quality issue",
      "OTHER_OR_UNCLEAR": "The available facts do not support the other categories"
    }
  },
  "triage_severity": {
    "type": "choice",
    "instructions": "Classify the triage severity using only impact explicitly documented in the state. Do not treat unknown production, customer, or shipping impact as confirmed impact.",
    "criteria": {
      "LOW": "No material operational impact is documented; routine review may be sufficient",
      "MEDIUM": "A limited issue or repeated event is reported, but broad impact is not established",
      "HIGH": "Material operational impact is explicitly supported by supplied evidence",
      "CRITICAL": "Severe immediate production, safety, or customer impact is explicitly supported by supplied evidence",
      "UNKNOWN": "The supplied state does not support a reliable severity classification"
    }
  },
  "next_review_stage": {
    "type": "choice",
    "instructions": "Choose the most appropriate review stage, not an operational action. Do not select a choice that releases or changes the hold.",
    "criteria": {
      "INITIAL_TRIAGE": "Confirm case details and identify the responsible owner",
      "ENGINEERING_INVESTIGATION": "A responsible engineer should investigate the event and supporting evidence",
      "EVIDENCE_COMPLETION": "Collect or verify missing evidence before further review",
      "PROMPT_HUMAN_ESCALATION": "A qualified owner should promptly assess explicitly documented high impact",
      "AUTHORIZED_CLOSURE_REVIEW": "Evidence is ready for an authorized person to review closure; this is not automatic closure"
    }
  },
  "primary_evidence_gap": {
    "type": "choice",
    "instructions": "Select the most important evidence gap that blocks a well-supported next review, based only on the supplied state.",
    "criteria": {
      "ROOT_CAUSE_VERIFICATION": "Evidence that tests or verifies the suspected root cause",
      "IMPACT_SCOPE": "Verified scope of affected lots and production, customer, or shipping impact",
      "CONTAINMENT_VERIFICATION": "Evidence that containment was applied and verified",
      "ENGINEER_ASSESSMENT": "A qualified engineer's assessment or disposition rationale",
      "RELEASE_APPROVAL": "Authorized approval required for release or closure",
      "OTHER_OR_UNCLEAR": "Another gap is present or the supplied state does not identify one clearly"
    }
  },
  "event_repeated_within_24h": {
    "type": "noul",
    "instructions": "Does `case.description` explicitly say that the event was observed again within 24 hours?",
    "criteria": {
      "true": "The description explicitly reports a repeat within 24 hours",
      "false": "The description does not report a repeat within 24 hours"
    }
  },
  "root_cause_confirmed": {
    "type": "noul",
    "instructions": "Does the supplied state provide verified evidence that the root cause is confirmed?",
    "criteria": {
      "true": "A confirmed root cause is explicitly supported by evidence in the state",
      "false": "The state says the cause is unconfirmed or supplies no verification"
    }
  },
  "release_approval_provided": {
    "type": "noul",
    "instructions": "Does the supplied state include an authorized release approval?",
    "criteria": {
      "true": "An explicit authorized release approval is included",
      "false": "No authorized release approval is included"
    }
  },
  "rca_evidence_readiness": {
    "type": "score",
    "instructions": "How ready is the supplied case information for an authorized root-cause-analysis review? Rate evidence completeness, not the likelihood that a guessed cause is correct.",
    "criteria": [
      "Not ready: key evidence, impact details, or verification is missing",
      "Partly ready: some useful facts are present, but important evidence or validation is still missing",
      "Review-ready: relevant evidence, impact, and verification are documented for an authorized review"
    ]
  },
  "impact_documentation": {
    "type": "score",
    "instructions": "How complete is the documentation of operational impact in the supplied state?",
    "criteria": [
      "Insufficient: no verified affected scope or operational impact is documented",
      "Partial: some affected scope is stated, but verified production, customer, or shipping impact is missing",
      "Complete: affected scope and verified operational impact are documented with supporting evidence"
    ]
  }
}
```

## Read the results

- **Choice:** Check `choice`, `probabilities`, and `confidence` for case type, severity, review stage, and evidence gap.
- **Noul:** Check `noul` (0–1) for the yes/no questions.
- **Score:** Check `score`, `legend`, `probabilities`, and `confidence` for evidence readiness.

Questions are evaluated separately using the same State. Compare the answers, but do not treat them as proof, approval, or commands. Application code and qualified people must control real actions.

## Try it yourself

Keep Questions unchanged. Change one part of the State at a time: remove the repeat-event note, add fictional impact details, or add fictional RCA evidence. Run again and compare the related answer IDs.
