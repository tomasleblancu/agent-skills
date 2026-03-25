---
name: fizko
description: |
  Use this skill whenever the user asks about accounting data, bank movements, reconciliation, IVA, F29, obligaciones, asientos contables, or any financial/accounting data for their Chilean business. This skill controls the `fizko` CLI (npm package) to query data from the Fizko API. Use it even if the user doesn't mention "fizko" explicitly — if they're asking about their empresa's financial data, this skill applies. This skill is READ-ONLY — it cannot modify data.
---

# Fizko CLI Skill

You have access to the `fizko` CLI, which communicates with the Fizko API for Chilean tax and accounting data.

**This skill is read-only.** You can query and view data, but cannot classify, reconcile, contabilizar, or modify any records. If the user asks to perform a write action, tell them to do it from the Fizko web platform.

## Setup check

Before running any command, verify the CLI is ready:

```bash
fizko status
```

If the output says "No autenticado", tell the user to run `fizko login` and wait for them. If `fizko` is not found, tell them to run `npm install -g fizko-cli`.

The status output also shows the active company. If it says "Sin empresa activa", handle it as described below.

## Active company

Almost every command needs a company context. The CLI resolves it automatically in this order:

1. `--company <uuid>` flag (explicit override for this command only)
2. Saved active company (`fizko companies use <uuid>` sets this persistently)
3. Auto-activate if the user has exactly 1 company

If none of the above apply, the command fails with an error asking to run `fizko companies use <uuid>`.

**When there's no active company** and the user has multiple companies:
```bash
fizko companies list          # show available companies
fizko companies use <uuid>    # activate one persistently
```

Once activated, all subsequent commands work without `--company`. You don't need to ask again during the conversation.

**To check or change the active company:**
```bash
fizko companies active        # show current active company
fizko companies use <uuid>    # switch to a different company
```

## Command reference

### Companies
```bash
fizko companies list                        # page 1 (20 results)
fizko companies list --search "ATAL"        # filter by name or RUT
fizko companies list --page 2               # next page
fizko companies get <uuid>
fizko companies use <uuid>    # activate as default
fizko companies active        # show current active company
```

**Finding a company by name:** always use `--search` first. Never loop over pages manually.
```bash
fizko companies list --search "nombre o RUT"
```

### Tax
```bash
# Resúmenes — --company opcional si hay empresa activa
fizko tax summary --period 2026-01
fizko tax iva --period 2026-01
fizko tax timeline

# Checkers (discrepancy analysis)
fizko tax checker-tributario --period 2026-01   # SII book vs Fizko
fizko tax checker-impuestos --period 2026-01    # F29 declared vs Fizko
```

### Accounting
```bash
# Plan de cuentas y asientos — --company opcional si hay empresa activa
fizko accounting accounts [--tree]
fizko accounting journal-entries [--period 2026-01]
fizko accounting journal-entry <uuid>
fizko accounting progress [--period 2026-01]

# Reportes financieros
fizko accounting balance-report --period-from 2026-01 --period-to 2026-12 [--cost-center-id <uuid>]
fizko accounting income-statement --period-from 2026-01 --period-to 2026-12 [--cost-center-id <uuid>]
fizko accounting classified-balance [--period 2026-12] [--period-from 2026-01 --period-to 2026-12]
fizko accounting rli-balance --period-from 2026-01 --period-to 2026-12
fizko accounting general-ledger [--period YYYY-MM] [--period-from --period-to] [--account-code <code>] [--cost-center-id <uuid>]
fizko accounting general-journal [--period YYYY-MM] [--period-from --period-to] [--cost-center-id <uuid>]
fizko accounting cash-flow [--start-date YYYY-MM-DD] [--weeks 5]
fizko accounting cash-flow-detail --tab ventas|compras|honorarios

# Centros de costo
fizko accounting cost-centers

# Obligaciones (cuentas por pagar/cobrar)
# --status: pending, partial, paid, overdue
# --obligation-type: payable, receivable
# --source-document-type: sales_document, purchase_document, honorarios_receipt, payroll_period, form29
fizko accounting obligations [--status pending] [--obligation-type payable] [--source-document-type purchase_document] [--search texto] [--from 2026-01-01] [--to 2026-01-31] [--conciliado true|false] [--contabilizado true|false]
fizko accounting obligation <uuid>
fizko accounting suggest-lines <uuid>            # sugerencia de líneas de asiento
fizko accounting payments <uuid>                 # listar abonos de una obligación
```

### Banking (movimientos bancarios)
```bash
# Read — --company opcional si hay empresa activa
# --status: pending, reconciled, matched, split
# --classification-status: classified | unclassified
fizko banking movements [--status pending] [--classification-status unclassified] [--type abonos|cargos] [--search texto] [--from 2026-01-01] [--to 2026-01-31] [--period 2026-01] [--bank-account <uuid>]
fizko banking movement <uuid>
fizko banking list-reconciliations [--period 2026-01]
fizko banking suggestions --movement-id <uuid>
fizko banking status [--period 2026-01]
```

## Handling output

All commands return JSON. When presenting results to the user:

- **Lists**: Show a summary table with the most relevant fields, not the raw JSON dump. For bank movements show date, description, amount, status.
- **Single records**: Show the key fields the user likely cares about, and mention there's more detail available if they want.
- **Errors**: If a command exits with an error (stderr), explain what went wrong in plain language and suggest the fix.
- **Pagination**: If results show a `count` much larger than `results`, let the user know there are more pages and offer to fetch them with `--offset`.

## Common workflows

### "¿Cuánto IVA debo este mes?"
1. `fizko tax iva --period YYYY-MM`
2. Show debito, credito, and balance clearly.

### "Muéstrame los movimientos bancarios pendientes"
1. `fizko banking movements --status pending`
2. Show a table with date, description, amount.

### "¿Cómo vamos con la contabilidad de enero?"
1. `fizko accounting progress --period YYYY-MM`
2. Show booked vs pending counts.

## Tips

- Use `--compact` flag when you need to parse the output programmatically (no indentation).
- Period format is always `YYYY-MM` (e.g., `2026-01`).
- When the user says "este mes" or "el mes pasado", calculate the actual period before running the command.
- UUIDs: if the user pastes a partial UUID or a name instead of UUID, run `companies list` or the relevant list command to find the right ID.
- `fizko accounting cost-centers` lists the available cost centers (needed before using `--cost-center-id`).
