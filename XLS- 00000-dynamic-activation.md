<pre>
  xls: [XLS number]
  title: Dynamic Activation
  description: Per ammendment configurable activation period of 14 days, 10 days, 3 days, 1 day, and now to address ledger vulnerabilities.
  author: Ivan Cinseros
  category: Amendment
  status: Idea
  created: 2026-05-25
  updated: 2026-05-25
</pre>

# Ammendment Idea: XLS-XXXX Dynamic Activation

## 1. Abstract

Currently, how fast an amendment starts the two week activation period is an unknown factor creating security risks when implementatio . To improve reasonable implementation times for critical 

This proposal introduces an optional per-amendment **activationTag** (integer 0–4) embedded directly in the amendment definition. The tag lets the author specify a shorter, risk-appropriate activation window while keeping the process fully transparent and backward compatible.

Once an amendment vote reaches the 80% validator support required to pass the 14 day (2-week) activation period starts counting down

This proposal introduces a per-amendment activation tag (an integer 0–4) that allows the amendment’s author to set a shorter, risk-appropriate 
activation window. The tag is embedded directly in the amendment definition, making the process transparent.

influence (by voting or not), but there is no built-in, explicit mechanism to shorten the activation safely. This creates unnecessary exposure
windows for the ledger when speed is critical.

## 2. Motivation

The XRP Ledger community has become accustomed to the standard two-week activation period required for amendments to become active after reaching the 80% dUNL validator support threshold. This creates significant risk when critical vulnerabilities or zero-day attacks are discovered because there are no mitigation options.

The XRPL currently has no built-in mechanism to shorten the activation window for amendments in emergency situations. To maintain ledger uptime, security, and integrity, a more flexible approach is needed to address time sensitive issues.

## 3. Specification

The proposed solution introduces a new optional field in every amendment definition. If the `activationTag` field is omitted, the amendment defaults the activationTag to a value, 4.

| Field Name    | Constant | Required | Internal Type | Default Value | Description |
|---------------|----------|----------|---------------|---------------|-------------|
| activationTag | No       | No       | UINT8         | 4             | A value of 0-4 correlating the number of days required for activaton. |

**Activation Tag Values:**

| Activation Tag | Duration          |
|-------|-------------------|
| 4     | 14 days (default) |
| 3     | 10 days           |
| 2     | 3 days            |
| 1     | 24 hours          |
| 0     | Immediate         |

--
flowchart TD
    A[1. Author submits Amendment<br/>with optional activationTag] 
    --> B[2. Amendment broadcast to network<br/>activationTag is fully transparent]
    B --> C[3. dUNL Validators review & vote]
    C --> D{80% supermajority reached?}
    D -->|No| C
    D -->|Yes| E[4. Activation timer starts<br/>based on activationTag value]
    E --> F[Wait the specified duration<br/>of continuous 80% support]
    F --> G[5. Amendment activates on next ledger]
--



## 4. Rationale

Two main factors determining how quickly an amendment goes live include the two-week activation period, after 80% dUNL validator approval, and the current activity level of those validators. In the best-case scenario (80% approval within 24 hours), an amendment would still require 15 days to activate. The current reality is that it takes months.

Creating a method to address critical vulnerabilities when they are found or actively being used needs to be created. The XRPL Foundation has helped improve dUNL response time drastically in the last year but it is still a work in progress. To mitigate these wait concerns that dUNL Validators can cause by not voting

Even in the best case scenerio, waiting 15 days to address a known critical vulnerability or worse, a zero-day attack is not acceptable. With the right checks and balances, a mechanism should exist to allow an amendment to activate as soon as it reaches the 80% threshold when urgent events are affecting ledger performance, security, or integrity. Adding a simple `activationTag` makes the XRP Ledger far more resilient, helping maintain its uptime and reputation.

By incorporating different values we maintain still providing 
a tag other than 4 should only be used when there's a concern for the ledgers security, stability, and integrity. 

## 5. Backwards Compatibility

By making `activationTag` an optional field that defaults to the current 14-day activation period (value 4), this proposal maintains full backward compatibility. Existing and future amendments are completely unaffected, and the current standards of the XRP Ledger remain unchanged.

## 6. Test Plan

The following tests will be implemented in the `rippled` codebase to validate the new `activationTag` functionality:

- Create five separate test amendments, each using a different `activationTag` value (0 through 4).
- Verify that each amendment activates after exactly the expected duration of continuous 80% dUNL supermajority support:

  `activationTag` 4 → after 14 days
  `activationTag` 3 → after 10 days
  `activationTag` 2 → after 3 days
  `activationTag` 1 → after 24 hours
  `activationTag` 0 → immediately (on the next ledger after 80% is reached)

- Confirm that omitting the `activationTag` field defaults to value 4.
- Test cases where amendment support fluctuates above and below 80% during the activation window (the activation clock must reset correctly).
- Verify that invalid tag values are rejected or default to 4.
- Verify that amendments with the same or different `activationTag` values can be active simultaneously on the same ledger.
- Run full regression tests on the existing amendment activation logic to ensure no impact on current or previously activated amendments.
- Verify that `activationTag` is correctly stored in the amendment object and visible via the `feature` command and ledger inspection.

## 7. Reference Implementation

TBA.

## 8. Security Considerations

Downgrading versions may not always resolve a vulnerability or attack
fixes take time time too dont forget!
downgrade might not fix things
The Author has the sole discretion and decides the activationTag value. Here are the recommendations.

4 - Every ammendment defaults to the current ledger state with a 14 day activation period. If the ammendment is not addressing the network's
Recommended Use Case:Standard features, improvements, edge-case fixes
3. Moderate bugs (not critical, but worth faster rollout)

2.Critical bugs with known impact

1. Serious security issues

0 Zero-day vulnerabilities

- activates on the next flagged ledger after 80% is reached
- If no tag is specified, the amendment defaults to 4 (14 days).
If a zero-day attack was occuring on the XRP Ledger, affecting wallet funds and network integrity, what options are there to address the attack?

If a critical vulnerability was found that coukd heavily impact network performance and security, should the ammendment wait the 2 week activation period like other standard ammendments? 

Ammendment Author can counter a slow vote by incorporating a  

# Appendix _(Optional)_

## Appendix A: FAQ _(Optional)_

### A.1: [Question]
