<pre>
  xls: XXXX
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

The XRP Ledger currently requires every amendment that reaches the 80% validator support threshold to observe a fixed 14 day activation period before becoming enabled. Continuing with this fixed period creates significant risk when critical security vulnerabilities or zero day attacks are discovered.

This proposal introduces an optional per amendment **activationTag** (integer 0–4) embedded directly in the amendment definition. The tag allows the author to specify a shorter, risk-appropriate activation window while keeping the process fully transparent and backward compatible. This new field helps mitigate the combined delays from bug-fix development, dUNL validator response times, testing, and the mandatory two-week activation period when critical vulnerabilities or attacks are discovered.

## 2. Motivation

The XRPL currently has no built in mechanism to shorten the activation window for amendments in emergency situations. To maintain ledger uptime, security, and integrity, a more flexible approach is needed to address time sensitive issues.

Recently, dUNL response times have drastically improved thanks to the XRPL Foundation. While this improves amendment implementation times, the fixed 14 day activation period still creates an unacceptable delay when dealing with critical vulnerabilities or zero-day attacks. Even with these ongoing improvements, the current system is not sufficient for urgent situations.

## 3. Specification

The proposed solution introduces a new optional field in every amendment definition. If the `activationTag` field is omitted, the amendment defaults to value **4**.

| Field Name    | Constant | Required | Internal Type | Default Value | Description |
|---------------|----------|----------|---------------|---------------|-------------|
| activationTag | No       | No       | UINT8         | 4             | A value of 0-4 correlating the number of days required for activaton. |

**Activation Tag Values:**

| Value | Duration          | Recommended Use Case |
|-------|-------------------|----------------------|
| 4     | 14 days (default) | Standard features, improvements, and edge-case fixes. Every amendment should use this value unless it addresses security, stability, or integrity concerns. |
| 3     | 10 days           | Moderate bugs that are not critical but would benefit from a faster rollout. |
| 2     | 3 days            | Critical vulnerabilities with network wide impact that are moderately easy to discover and exploit. |
| 1     | 24 hours          | Critical vulnerabilities with network wide impact that have not yet been actively exploited and are relatively easy to discover and exploit. |
| 0     | Immediate         | Rare cases involving active zero-day attacks that threaten ledger stability, security, and integrity. |

**Amendment Author Guidance** 

Authors should choose the lowest `activationTag` value that is appropriate for the urgency and risk of the change. Using a lower tag signals to the network that faster activation is justified.

These recommended values provide a well balanced approach, giving node operators sufficient time to upgrade while also providing options to address urgent incidents in a reasonable timeframe.

## Diagram: Amendment Activation Process with activationTag
                +-------------------------+
                | 1. Author submits       |
                |    Amendment + optional |
                |    activationTag        |
                +-------------------------+
                            |
                            v
                +-------------------------+
                | 2. Amendment broadcast  |
                |    to network           |
                |    (activationTag is    |
                |     fully transparent)  |
                +-------------------------+
                            |
                            v
                +-------------------------+
                | 3. dUNL Validators      |
                |    review & vote        |
                +-------------------------+
                            |
                            v
                +-------------------------+
                | 4. 80% supermajority    |
                |    reached              |
                +-------------------------+
                            |
                            v
                +-------------------------+
                | 5. Activation timer     |
                |    starts               |
                |    (based on tag)       |
                +-------------------------+
                            |
                            v
                +-------------------------+
                | 6. Amendment activates  |
                |    on next ledger       |
                +-------------------------+

## 4. Rationale

Two main factors determining how quickly an amendment goes live include the two-week activation period, after 80% dUNL validator approval, and the current activity level of those validators. In the best-case scenario (80% approval within 24 hours), an amendment would still require 15 days to activate. The current reality is that it takes months.

Creating a method to address critical vulnerabilities when they are found or actively being used needs to be created. The XRPL Foundation has helped improve dUNL response time drastically in the last year but it is still a work in progress. To mitigate these wait concerns that dUNL Validators can cause by not voting

Even in the best case scenerio, waiting 15 days to address a known critical vulnerability or worse, a zero-day attack is not acceptable. With the right checks and balances, a mechanism should exist to allow an amendment to activate as soon as it reaches the 80% threshold when urgent events are affecting ledger performance, security, or integrity. Adding a simple `activationTag` makes the XRP Ledger far more resilient, helping maintain its uptime and reputation.

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

Besides offseting bug fix development times, dUNL Validator delays, testing time, and the two week activation period this proposal helps remove the vulnerability of announcing an update due to a vulnerability that was found. 

If a zero-day attack is occurring on the XRP Ledger that affects wallet funds or network integrity, the protocol currently has no built-in mechanism to respond quickly. The same is true for a critical vulnerability that could heavily impact network performance and security.

Downgrading to a previous version is not always a viable solution, as it may not fully resolve the vulnerability or stop an active attack. Fixes themselves also take time to develop and test. This tag counters both dUNL validator inactivity and long fix development times.

This proposal addresses these risks by introducing the optional `activationTag`. Since the tag is entirely optional and defaults to the existing 14-day period, it cannot accidentally shorten activation windows on any amendment unless the author explicitly chooses a faster value.

All activation behavior remains fully enforced on-ledger through the normal amendment voting process (80% supermajority required). Validators and node operators retain complete control, they can refuse to vote for any amendment that uses an aggressive tag, or they can upgrade and vote quickly when an emergency tag is justified.

The `activationTag` is embedded directly in the amendment definition and is visible in the ledger and rippled source code before any vote begins, giving the entire network full transparency. No new attack surface is introduced — the mechanism simply provides a safer, faster, and more controlled way to respond to genuine zero-day vulnerabilities and critical bugs that already threaten ledger integrity today.

# Appendix

## Appendix A: FAQ

### A.1: [Question]
