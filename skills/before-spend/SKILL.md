---
name: before-spend
description: Use before any purchase, subscription, money transfer, or API credit top-up. Request Sanction authorization; proceed only on authorized true with next_action proceed, within user authority and host permissions.
---

# Before spend

The wallet is cooperative: Sanction does not intercept the payment. Call it before the spend becomes irreversible. Sanction approval does not expand the user's request or override the host's permission requirements.

## Steps

1. Gather the exact `action` (`purchase` | `subscribe` | `transfer`), `amount_usd`, `merchant`, and `category` (`software`, `services`, `research`, `infrastructure`, `marketing`, `legal`, or `other`). Add `description` when a human may have to approve. Resolve missing details before requesting authorization.
2. Call `sanction_authorize` with those fields. Read `authorized` and `next_action` from `structuredContent`, or the JSON text block if the host exposes only text. `isError: false` and a readable success message do not establish permission.
3. Only `authorized: true` with `next_action: proceed` permits the exact spend once, within the user's request and the host's permissions. A synthetic authorization test must execute nothing. Authorization does not itself perform the payment.
4. On `next_action: wait`, do not spend. Keep every original field and follow **handle-escalation**. Only a usable grant permits retrying the exact original authorization with the returned `grant_id`; redemption must satisfy the same decision and permission checks before any action.
5. On `stop`, denial, an error, missing or contradictory decision fields, or an unknown outcome, stop and report it. Do not spend, retry with altered fields, or automatically issue a fresh request to evade refusal or uncertainty. Reconcile an unknown payment result before considering another attempt.

Provisioning seats, licenses, or infrastructure uses `sanction_authorize_provision` (resource + dollars in one request). Apply the same decision, user-authority, host-permission, synthetic-test, and stop rules; pending requests follow **handle-escalation**.
