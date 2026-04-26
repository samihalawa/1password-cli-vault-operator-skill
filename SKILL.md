---
name: 1password-cli-vault-operator-skill
description: Use 1Password CLI fast and non-interactively through OP_SERVICE_ACCOUNT_TOKEN for vault/item search, read, test, create, update, consolidation, and archive tasks.
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

## Safety

- Never print full secrets in the final response.
- Use short prefixes/suffixes only when identifying secrets.
- Preserve source item title, source item ID, field label, usage context, and test status when consolidating.
- Archive only after the canonical target has been read back and verified.
- Keep distinct human/service identities separate even if they share a domain.

## Final Format

Report:
- auth status
- vault used
- items found, changed, archived, and left alone
- credential test status with prefixes only
- canonical item title and ID
- exact blockers, if any
