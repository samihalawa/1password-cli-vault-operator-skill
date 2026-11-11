---
name: 1password-cli-vault-operator-skill
description: Use 1Password CLI fast and non-interactively through OP_SERVICE_ACCOUNT_TOKEN for vault/item search, read, test, create, update, consolidation, archive, and safe provider API credential fallback when a UI is signed out or a shell token is absent.
---

# 1Password CLI Vault Operator

Use this when the user asks to work with 1Password from the terminal and wants no browser, biometric, or repeated sign-in prompts.

## Workflow

1. Load the token from the user's global env first:
   ```bash
   source ~/.env
   export OP_VAULT="${OP_VAULT:-Master Vault}"
   op vault list --format=json
   ```
2. If auth fails, check only these basics, then stop with the exact error:
   ```bash
   env | grep '^OP_SERVICE_ACCOUNT_TOKEN=' | sed 's/=.*/=SET/'
   op --version
   op vault list --format=json 2>&1
   ```
3. Read before changing:
   ```bash
   op item list --vault "$OP_VAULT" --format=json
   op item get "$ITEM_ID" --vault "$OP_VAULT" --format=json
   ```
4. Classify items before cleanup:
   - keep separate account logins, recovery codes, unrelated services, and ambiguous records
   - consolidate API tokens/OAuth secrets only when provenance and all useful values are preserved
   - archive duplicate junk with `op item delete --archive`; do not permanently delete unless explicitly asked
5. Test credentials through low-risk identity endpoints when possible. Never use destructive provider actions as a test.
6. Create or update canonical items with files for notes to avoid shell quoting corruption:
   ```bash
   op item create --vault "$OP_VAULT" --category secure-note --title "$TITLE" notesPlain="$(cat notes.md)"
   op item edit "$ITEM_ID" --vault "$OP_VAULT" notesPlain="$(cat notes.md)"
   ```

## Provider Audit Fallback

When a provider UI is signed out or a shell credential is absent:

1. search item metadata by title/category without revealing fields;
2. inspect field labels and types, not field values;
3. reveal only the required field into a short-lived scoped shell variable and
   pass it directly to the provider command without `echo` or a disk file;
4. test a read-only identity, project-list, or resource-list endpoint first;
5. record only the resource identity and whether the credential worked.

This is a valid recovery path for providers such as Coolify and PostHog. For a
PostHog project audit, compare the provider-returned public `api_token` with the
deployed source token without printing either value. A logged-out UI alone is
not proof that the provider is inaccessible.

## Safety

- Never print full secrets in the final response.
- Use short prefixes/suffixes only when identifying secrets.
- Preserve source item title, source item ID, field label, usage context, and test status when consolidating.
- Archive only after the canonical target has been read back and verified.
- Keep distinct human/service identities separate even if they share a domain.
- Avoid broad item JSON in logs. Prefer item titles, IDs, categories, and field
  labels until the one required secret is consumed directly by its command.
- Do not use verbose process listings, environment-dumping process flags, or
  full command-line inspection around credential-bearing tools. Process argv
  and MCP launch arguments can contain secrets. When process proof is needed,
  read only the PID, executable basename, and working directory.

## Final Format

Report:
- auth status
- vault used
- items found, changed, archived, and left alone
- credential test status with prefixes only
- canonical item title and ID
- exact blockers, if any

## Execute and verify the workflow

When a user authorizes an operation, carry it out through the native provider or executable workflow and verify the resulting state. A vault entry, guide, checklist, or task record is not proof that authentication, Autofill, signing, or submission worked. Use 1Password for actual credentials and their Autofill metadata. Keep reusable execution instructions in the owning skill or agent, and do not create a Secure Note as a substitute for executing the task.

For Autofill, keep the exact account on one Login item with the provider URL and native username/password fields. For multi-field forms, capture missing form fields on that existing item using the supported extension workflow, then verify from a fresh form that the suggestion reaches the signed-in provider. Keep signing keys, encryption secrets, and unrelated authentication methods in their appropriate existing records. Verify a deduplication with the native CLI's archive-inclusive item list and explicit ARCHIVED state; an exact-ID get can still resolve an archived item.
