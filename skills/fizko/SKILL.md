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
fizko tax purchases --company <uuid> [--period-year 2026] [--period-month 1] [--document-type 33]
fizko tax sales --company <uuid> [--period-year 2026] [--period-month 1] [--document-type 33]
fizko tax honorarios --company <uuid> [--period-year 2026] [--period-month 1]
fizko tax documents --company <uuid> [--period 2026-01] [--start-date 2026-01-01] [--end-date 2026-01-31]
fizko tax documents-summary --company <uuid>     # count by type, last 3 months
fizko tax contacts --company <uuid>              # suppliers and clients

# Individual records
fizko tax purchase <uuid>
fizko tax sale <uuid>
fizko tax document <uuid>

# F29 and tax forms
fizko tax f29 --company <uuid> [--year 2026]
fizko tax f29-codes --company <uuid> --period 2026-01
fizko tax f29-export --company <uuid> --code <code> --period 2026-01
fizko tax summary --company <uuid> --period 2026-01
fizko tax iva --company <uuid> --period 2026-01
fizko tax timeline --company <uuid>
fizko tax ddjj --company <uuid> [--year 2025]

# Checkers (discrepancy analysis)
fizko tax checker-tributario --company <uuid> --period 2026-01   # SII book vs Fizko
fizko tax checker-impuestos --company <uuid> --period 2026-01    # F29 declared vs Fizko
```

### Accounting
```bash
# Plan de cuentas y asientos
fizko accounting accounts --company <uuid> [--tree]
fizko accounting journal-entries --company <uuid> [--period 2026-01]
fizko accounting journal-entry <uuid>
fizko accounting progress --company <uuid> [--period 2026-01]

# Reportes financieros
fizko accounting balance-report --company <uuid> --period-from 2026-01 --period-to 2026-12 [--cost-center-id <uuid>]
fizko accounting income-statement --company <uuid> --period-from 2026-01 --period-to 2026-12 [--cost-center-id <uuid>]
fizko accounting general-journal --company <uuid> --period 2026-01 [--page <n>] [--page-size <n>]
fizko accounting general-ledger --company <uuid> --period-from 2026-01 --period-to 2026-12 [--account-id <uuid>]
fizko accounting classified-balance --company <uuid> [--period 2026-12] [--period-from 2026-01 --period-to 2026-12]
fizko accounting rli-balance --company <uuid> --period-from 2026-01 --period-to 2026-12

# Centros de costo
fizko accounting cost-centers --company <uuid>

# Obligaciones (cuentas por pagar/cobrar)
# --status: pending, partial, paid, overdue
# --obligation-type: payable, receivable
# --source-document-type: sales_document, purchase_document, honorarios_receipt, payroll_period, form29
fizko accounting obligations --company <uuid> [--status pending] [--obligation-type payable] [--source-document-type purchase_document] [--search texto] [--from 2026-01-01] [--to 2026-01-31] [--conciliado true|false] [--contabilizado true|false]
fizko accounting obligation <uuid>

fizko accounting suggest-lines <uuid>            # sugerencia de líneas de asiento
fizko accounting contabilizar <uuid> [--lines '[{"account_code":"5101","debit":1000,"credit":0}]'] [--account-code 5101] [--description "texto"]
fizko accounting update-journal <uuid> --lines '[...]' [--description "texto"]
fizko accounting descontabilizar <uuid>

fizko accounting abonar <uuid> --amount 50000 --payment-type cash [--notes "texto"] [--no-contabilizar]
  # payment-type: cash, check, third_party, international, factoring, bad_debt
fizko accounting payments <uuid>                 # listar abonos de una obligación
fizko accounting reverse-payment <uuid> --payment-id <uuid>

fizko accounting set-cost-center <uuid> --cost-center <uuid>
fizko accounting sync --company <uuid> [--period 2026-01] [--obligation-type payable] [--force]
```

### Banking (movimientos bancarios)
```bash
# Read
# --status: pending, reconciled, matched, split
# --classification-status: classified | unclassified
fizko banking movements --company <uuid> [--status pending] [--classification-status unclassified] [--type abonos|cargos] [--search texto] [--from 2026-01-01] [--to 2026-01-31] [--period 2026-01] [--bank-account <uuid>]
fizko banking movement <uuid>
fizko banking list-reconciliations --company <uuid> [--period 2026-01]
fizko banking suggestions --company <uuid> --movement-id <uuid>
fizko banking status --company <uuid> [--period 2026-01]

# Clasificación
fizko banking classify <uuid> --classification 'Gasto Operacional' [--document-type 'Factura'] [--contact-id <uuid>] [--comment "texto"]
fizko banking update <uuid> [--category texto] [--comment texto] [--cost-center <uuid>] [--pin true|false]
fizko banking bulk-classify --company <uuid> --movement-ids UUID1,UUID2 --classification 'Gasto Operacional'

# Splits
fizko banking split <uuid> --splits '[{"amount":1000,"description":"texto"},...]'

# Contabilización
fizko banking contabilizar <uuid> --account-code 5101 [--description "texto"]
fizko banking contabilizar-reconciliaciones <uuid> --entries '[{"reconciliation_id":"UUID","lines":[{"account_code":"5101","debit":1000,"credit":0}]}]'
fizko banking update-journal <uuid> --lines '[...]' [--description "texto"]
fizko banking descontabilizar <uuid>

# Conciliación
fizko banking reconcile --movement-id <uuid> --obligation-id <uuid> [--amount 50000] [--notes "texto"]
fizko banking reconcile-multi --movement-id <uuid> --obligation-ids UUID1,UUID2
fizko banking unmatch <reconciliation-uuid>
fizko banking obligation-from-movement <uuid> [--classification 'Gasto Operacional'] [--obligation-type payable]
```

## Handling output

All commands return JSON. When presenting results to the user:

- **Lists**: Show a summary table with the most relevant fields, not the raw JSON dump. For bank movements show date, description, amount, status.
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
2. If user only gives movement ID: `fizko banking suggestions --company <uuid> --movement-id <uuid>` to show candidates, then reconcile after confirmation.
3. If no matching obligation exists: `fizko banking obligation-from-movement <uuid>`

### "Contabiliza este movimiento"
1. `fizko accounting accounts --company <uuid>` if user doesn't know the account code.
2. `fizko banking contabilizar <uuid> --account-code <code>`

### "Contabiliza las reconciliaciones de este movimiento"
1. `fizko banking movement <uuid>` to see the reconciliations and their IDs.
2. `fizko banking contabilizar-reconciliaciones <uuid> --entries '[{"reconciliation_id":"UUID","lines":[...]}]'`

### "Contabiliza esta obligación"
1. `fizko accounting suggest-lines <uuid>` to get the suggested account lines.
2. `fizko accounting contabilizar <uuid> --lines '<suggested lines>'`

### "¿Cómo vamos con la contabilidad de enero?"
1. `fizko accounting progress --company <uuid> --period YYYY-MM`
2. Show booked vs pending counts.

### "Sincronizar obligaciones"
1. `fizko accounting sync --company <uuid> [--period YYYY-MM]`

## Tips

- Use `--compact` flag when you need to parse the output programmatically (no indentation).
- Period format is always `YYYY-MM` (e.g., `2026-01`).
- When the user says "este mes" or "el mes pasado", calculate the actual period before running the command.
- UUIDs: if the user pastes a partial UUID or a name instead of UUID, run `companies list` or the relevant list command to find the right ID.
- `fizko accounting contabilizar` without `--lines` auto-generates the journal entry; use `suggest-lines` first to see what lines the system recommends.
- `fizko accounting cost-centers` lists the available cost centers (needed before using `--cost-center-id`).
