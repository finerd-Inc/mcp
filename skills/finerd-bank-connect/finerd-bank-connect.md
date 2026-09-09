---
name: finerd-bank-connect
description: Help a user connect a bank account to Finerd by finding a supported institution, providing its connection link, and checking imported accounts and sync status. Use for bank-linking and first-time bank setup requests, not manual account creation, payments, or transaction cleanup.
---

# Connect a bank to Finerd

Help the user find their institution and complete bank linking in their browser.
The Finerd tools discover institutions and report connection status; the user
signs in and selects accounts on the bank/provider pages. Keep credentials,
verification codes, and access tokens out of the chat.

## Prerequisites and scope

Use the connected Finerd tools, which may have client-specific name prefixes:
`list_spaces`, `get_space_primary_currency`, `search_institutions`, and
`list_connected_institutions`.

If a required tool is unavailable, explain which capability is missing and ask
the user to enable or reconnect the Finerd connector. Do not invent connection
URLs or substitute a manual `create_bank_account` call: that creates an account
record without linking a bank.

This workflow covers bank accounts. The institution search does not establish
support for crypto exchanges or brokerage connections. Installing or reading this
skill alone is not a request to connect an account; use it for the user's stated
bank-linking task.

## Choose the space

Honor a space the user has already selected. Otherwise, call `list_spaces`; use
the sole available space or ask which one to use when several are available.
Pass that `space_id` consistently throughout the task.

Check `get_space_primary_currency`. A `RESOURCE_NOT_FOUND` error from this check
can indicate unfinished account setup: ask the user to complete currency
selection in the Finerd web app before continuing. Do not guess a currency or
recreate their space. Treat other errors according to their actual code.

## Find the institution

Use the bank name already provided, or ask which bank or card issuer the user
wants to link. Work through multiple banks individually. If the country is known,
pass its code to `search_institutions`; otherwise ask when needed to distinguish
similarly named banks. `country_code` affects ranking, not filtering: inspect each
result's `region`.

Interpret the results as follows:

- `children`: a group of regional institutions. Children can share the same name;
  distinguish them by region and let the user choose when the target is ambiguous.
- `connect_url`: the returned link for the selected institution. Share it exactly
  as returned; do not construct, edit, or guess the URL.
- No `connect_url`: the row cannot be connected directly. Check its children if
  present; otherwise explain that no direct connection link was returned.
- `connection_started: true`: a provider handshake has started. This does not prove
  that accounts were imported; on a group it can refer to only one child.
- No matching result: check the exact bank name with the user. If `has_more` is
  true, narrow the query. An empty search does not establish that an entire
  country is unsupported. After a refined search still fails, report that the
  requested institution was not found and offer to continue with another bank.

Use returned institutions as the source of connection options; avoid promising
coverage based on memory.

## Complete linking in the browser

Give the user the selected institution's `connect_url` and explain the steps:

1. Open the link and authenticate on the bank/provider page.
2. Select the accounts to import and finish saving the selection.
3. Return to the chat when finished.

Account selection is essential: completing bank authentication alone does not
finish the import. Leave authentication and account selection to the user.

If `connection_started` was already true, check `list_connected_institutions`
before claiming anything is complete. When there is no matching imported-account
entry, the returned connection link can be shared again so the user can finish.

## Verify the result

After the user says they have finished, call `list_connected_institutions` and
match the intended institution. Each row represents one connection; duplicate
bank names can represent separate connections, so do not merge them in the report.

If no matching row appears, explain that imported accounts are not visible yet.
Ask whether account selection was saved. Recheck after the user confirms; if the
result remains empty, leave the outcome unverified and suggest checking the
browser flow or contacting Finerd support. Avoid repeated polling.

For a matching row, report the account count and interpret `sync_status`:

| Status | Meaning and next step |
| --- | --- |
| `IMPORT_IN_PROGRESS` or `WAITING_FOR_FIRST_TRANSACTION` | Linking has progressed to import; transactions may still be arriving. Do not ask for reauthorization solely for this status. |
| `ACTIVE` | The connection is active. Report the returned account count. |
| `CREDENTIAL_UPDATE_REQUIRED` or `USER_ACTION_REQUIRED` | The provider needs user action. Offer a connection link returned for that institution so the user can complete it. |
| `DEACTIVATED` | The connection is inactive. Offer to help the user reconnect through a returned link. |
| Missing or unfamiliar status | Report the status as returned and avoid asserting that syncing is healthy. |

If no appropriate link is available, search for the institution again rather than
constructing one. Finish with the verified bank, account count, and any pending
user action or import. Continue with other banks only within the user's request.

## Errors that change the workflow

- `SPACE_HAS_ACTIVE_MIGRATION`: a data import is running. Explain that linking must
  wait for it to finish; do not repeatedly retry or start transaction cleanup.
- `401`: the connector may need reauthentication. Ask the user to reconnect rather
  than requesting their token.
- `403`: access to the selected space is unavailable. Refresh `list_spaces` and
  confirm an available target if necessary; do not assume the space was deleted.
- Other failures: report the available error code and trace ID without claiming
  the bank is unsupported or the connection succeeded.
