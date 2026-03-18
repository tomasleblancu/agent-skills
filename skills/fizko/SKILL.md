---
name: fizko
description: |
  Use this skill whenever the user asks about tax documents, accounting data, bank movements, reconciliation, IVA, F29, facturas, compras, ventas, honorarios, obligaciones, asientos contables, or any financial/accounting data for their Chilean business. Also use it when the user asks to classify a bank movement, reconcile a movement with a document, create a journal entry, or perform any accounting action. This skill controls the `fizko` CLI (npm package) to query and act on data from the Fizko API. Use it even if the user doesn't mention "fizko" explicitly — if they're asking about their empresa's financial data or want to do something with a bank movement or tax document, this skill applies.
---

# Fizko CLI Skill

You have access to the `fizko` CLI, which communicates with the Fizko API for Chilean tax and accounting data.

## Setup check

Before running any command, verify the CLI is ready:

```bash
fizko status
```

If the output says "No autenticado", tell the user to run `fizko login` and wait for them. If `fizko` is not found, tell them to run `npm install -g fizko-cli`.

## The golden rule: always need a company UUID

Almost every command requires `--company <uuid>`. If the user hasn't provided one, run this first:

```bash
fizko companies list
```

Show them the list and ask which company they want to work with. Once you have the UUID, use it for all subsequent commands in the conversation — don't ask again.

## Command reference

### Companies
```bash
fizko companies list
fizko companies get <uuid>
```

### Tax documents
```bash
# Lists (paginated)
fizko tax purchases --company <uuid> [--period-year 2026] [--period-month 1]
fizko tax sales --company <uuid> [--period-year 2026] [--period-month 1]
fizko tax honorarios --company <uuid> [--period-year 2026]
fizko tax documents --company <uuid> [--period 2026-01]   # purchases + sales + honorarios unified
fizko tax documents-summary --company <uuid>              # count by type, last 3 months

# Individual records
fizko tax purchase <uuid>
fizko tax sale <uuid>
fizko tax document <uuid>

# F29 and tax forms
fizko tax f29 --company <uuid> [--year 2026]
fizko tax summary --company <uuid> --period 2026-01
fizko tax iva --company <uuid> --period 2026-01
fizko tax timeline --company <uuid>

# Checkers (discrepancy analysis)
fizko tax checker-tributario --company <uuid> --period 2026-01
fizko tax checker-impuestos --company <uuid> --period 2026-01
```

### Accounting
```bash
# Read
fizko accounting accounts --company <uuid> [--tree]
fizko accounting journal-entries --company <uuid> [--period 2026-01]
fizko accounting journal-entry <uuid>
fizko accounting progress --company <uuid> [--period 2026-01]

# Reports
fizko accounting balance-report --company <uuid> --period-from 2026-01 --period-to 2026-12
fizko accounting income-statement --company <uuid> --period-from 2026-01 --period-to 2026-12
fizko accounting general-journal --company <uuid> --period 2026-01
fizko accounting general-ledger --company <uuid> --period-from 2026-01 --period-to 2026-12
fizko accounting classified-balance --company <uuid> --period 2026-12
fizko accounting rli-balance --company <uuid> --period-from 2026-01 --period-to 2026-12

# Obligations (cuentas por pagar/cobrar)
fizko accounting obligations --company <uuid> [--status pending] [--obligation-type payable]
fizko accounting obligation <uuid>
fizko accounting contabilizar <uuid> [--lines '[{"account_code":"5101","debit":1000,"credit":0}]']
fizko accounting descontabilizar <uuid>
fizko accounting abonar <uuid> --amount 50000 --payment-type cash [--notes "texto"]
  # payment-type options: cash, check, third_party, international, factoring, bad_debt
```

### Banking (movimientos bancarios)
```bash
# Read
fizko banking movements --company <uuid> [--status pending] [--period 2026-01]
fizko banking movement <uuid>
fizko banking list-reconciliations --company <uuid> [--period 2026-01]
fizko banking suggestions --company <uuid> --movement-id <uuid>
fizko banking status --company <uuid> [--period 2026-01]

# Actions on a specific movement
fizko banking classify <uuid> --classification 'Gasto Operacional' [--comment "texto"]
fizko banking contabilizar <uuid> --account-code 5101 [--description "texto"]
fizko banking descontabilizar <uuid>

# Reconciliation
fizko banking reconcile --movement-id <uuid> --obligation-id <uuid> [--amount 50000]
fizko banking reconcile-multi --movement-id <uuid> --obligation-ids UUID1,UUID2
fizko banking unmatch <reconciliation-uuid>
fizko banking obligation-from-movement <uuid> [--classification 'Gasto Operacional']
```

## Handling output

All commands return JSON. When presenting results to the user:

- **Lists**: Show a summary table with the most relevant fields, not the raw JSON dump. For example, for bank movements show date, description, amount, status — not every field.
- **Single records**: Show the key fields the user likely cares about, and mention there's more detail available if they want.
- **Errors**: If a command exits with an error (stderr), explain what went wrong in plain language and suggest the fix.
- **Pagination**: If results show a `count` much larger than `results`, let the user know there are more pages and offer to fetch them with `--offset`.

## Common workflows

### "¿Cuánto IVA debo este mes?"
1. `fizko tax iva --company <uuid> --period YYYY-MM`
2. Show debito, credito, and balance clearly.

### "Muéstrame los movimientos bancarios pendientes"
1. `fizko banking movements --company <uuid> --status pending`
2. Show a table with date, description, amount. Offer to classify or reconcile any of them.

### "Concilia este movimiento con su factura"
1. If user gives movement ID and obligation ID: `fizko banking reconcile --movement-id <uuid> --obligation-id <uuid>`
2. If user only gives movement ID: run `fizko banking suggestions --company <uuid> --movement-id <uuid>` first to show candidates, then reconcile after confirmation.
3. If no matching obligation exists: `fizko banking obligation-from-movement <uuid>`

### "Contabiliza este movimiento"
1. `fizko accounting accounts --company <uuid>` to help user pick the account code if they don't know it.
2. `fizko banking contabilizar <uuid> --account-code <code>`

### "¿Cómo vamos con la contabilidad de enero?"
1. `fizko accounting progress --company <uuid> --period YYYY-MM`
2. Show booked vs pending counts.

## Tips

- Use `--compact` flag when you need to parse the output programmatically (no indentation).
- Period format is always `YYYY-MM` (e.g., `2026-01`).
- When the user says "este mes" or "el mes pasado", calculate the actual period before running the command.
- UUIDs: if the user pastes a partial UUID or a name instead of UUID, run `companies list` or the relevant list command to find the right ID.
