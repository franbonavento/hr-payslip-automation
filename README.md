# HR Payslip Distribution Automation

**Production workflow automation for a multi-branch SME**

> An anonymized case study of a real automation implemented in production. No employee, payroll, client, credential, or internal business data is included in this repository.

## Overview

A multi-branch Argentine SME needed a low-cost and traceable way to distribute monthly payslips to approximately 60 employees.

The solution was designed and implemented with **Make**, using email as the input channel, **Google Sheets** as the deterministic employee mapping and audit layer, **Google Drive** for monthly document backup, and **SMTP/IMAP** for email integration.

The workflow is currently in production and was developed from initial analysis through testing and deployment.

## Business problem

Each month, an external accountant sends individual payslip PDFs that must be distributed to the corresponding employees.

The process presents several operational challenges:

- Around 60 individual documents per monthly run.
- File names are not always consistent between employees or months.
- Payroll documents contain sensitive information.
- Every PDF must be associated with the correct recipient.
- The process needs traceability so HR can verify the result.
- The client should be able to operate the solution without depending on the developer for each monthly run.

Because an incorrect match could expose confidential payroll information, the workflow prioritizes **deterministic validation over probabilistic matching**.

## Solution architecture

```text
External accountant
       |
       v
HR email inbox
       |
       v
Make automation
       |
       +--> Read attachments
       |
       +--> Iterate PDFs
       |
       +--> Extract file key
       |
       v
Google Sheets
Employee mapping
       |
       +--> Exact match? ---- No ----> Human review / no automatic send
       |
      Yes
       |
       +--> Google Drive monthly backup
       |
       +--> Personalized email + employee PDF
       |
       v
Employee
       |
       v
Google Sheets
Delivery log / audit trail
```

## Workflow

1. The accountant sends one email containing the employee payslip PDFs.
2. Make detects the incoming message and processes its attachments.
3. Each PDF is handled individually.
4. A key is extracted from the file name.
5. The key is matched against an employee mapping table in Google Sheets.
6. Only an exact valid match proceeds through the automated delivery path.
7. The PDF is backed up in the corresponding monthly folder in Google Drive.
8. A personalized email is generated and sent with the employee's PDF attached.
9. The result is written to an audit log for later verification.
10. Exceptions are reviewed instead of being guessed automatically.

## Why deterministic matching?

One of the main design challenges was inconsistent PDF naming.

Real-world payroll files may use a surname, first name, compound surname, abbreviated name, or a naming convention that changes between months.

A fuzzy or AI-based match could improve convenience, but it also introduces an unacceptable risk: **sending a confidential payslip to the wrong employee**.

The production workflow therefore uses a maintained mapping table:

```text
FILE_KEY -> EMPLOYEE -> EMAIL -> STATUS
```

Only deterministic matches are eligible for automatic delivery. Ambiguous or unmatched cases require review.

This was a deliberate design decision: **reliability and privacy take priority over full automation**.

## Controls and traceability

The solution includes several operational controls:

- Employee mapping maintained separately from the automation logic.
- Exact matching before sending.
- Human checkpoint for unresolved cases.
- Monthly backup of processed PDFs.
- Delivery log in Google Sheets.
- Pre-run reconciliation of received PDFs against the employee mapping.
- Employee status to handle cases that should not be sent automatically.

Additional controls are planned as future improvements.

## Development and deployment

The workflow was not built directly against production data.

The implementation followed an iterative process:

1. Understand the HR process and constraints.
2. Design the automation and employee mapping.
3. Test with a personal email account and copies of the working spreadsheet.
4. Resolve file-name and mapping edge cases.
5. Rebuild/configure the final scenario in the client's Make environment.
6. Execute and validate the first production run.
7. Document the solution so the client can operate it independently.

## Technical challenges

| Challenge | Approach |
| --- | --- |
| Inconsistent PDF file names | Deterministic mapping table instead of deriving identity only from a formula |
| Compound, abbreviated or changing names | Explicit file-key mapping and human review for exceptions |
| Sensitive payroll information | No probabilistic auto-matching |
| Processing multiple PDFs | Iterate attachments individually |
| Drive folder branching | Controlled folder search/create logic |
| Make router paths cannot converge | If/else + merge pattern where convergence is required |
| Attachment metadata/data mapping | Map file name and binary data from the iterator output |
| Moving the scenario between accounts | Configure the production workflow in the client's environment rather than relying on connection IDs from imported blueprints |

## Production scale

Approximate current scale:

- **~60 employees**
- **Monthly execution**
- **~60 payslip emails per run**
- **~259 Make operations per monthly run**

These figures are intentionally approximate and contain no employee-level information.

## Stack

- **Make** — workflow orchestration
- **Google Sheets** — employee mapping, statuses and delivery log
- **Google Drive** — monthly PDF backup
- **IMAP / SMTP** — email integration using the client's domain
- **Claude** — development and reconciliation assistance

AI is used as a development/support tool, **not as the production decision mechanism for employee-to-document matching**.

## Example data

This repository contains only synthetic examples.

See [`examples/employee_mapping.csv`](examples/employee_mapping.csv) for a fictional representation of the mapping structure.

## Privacy and confidentiality

This repository intentionally does **not** contain:

- Client name
- Employee names or email addresses
- Real payslips or salary information
- Production spreadsheets or logs
- Email server configuration
- Folder or spreadsheet IDs
- Credentials or connection IDs
- Exported Make blueprints
- Screenshots containing production data

The objective is to document the engineering and process-design decisions without exposing confidential information.

## Current status

**In production.**

The core workflow has completed a successful production run and supports the monthly distribution process.

Potential future improvements include:

- Automated logging of unmatched files.
- Duplicate-key validation.
- Validation that the document month matches the execution period.
- More automated reconciliation reporting while retaining human approval for ambiguous cases.

## What this project demonstrates

This project is less about a single automation tool and more about solving an operational problem end-to-end:

**process analysis -> requirements -> data mapping -> automation -> validation -> testing -> production -> traceability**

It demonstrates practical experience in:

- Process automation
- Business-rule translation
- Data mapping and validation
- Exception handling
- Privacy-aware workflow design
- Testing and deployment
- Operational documentation
- Continuous improvement

---

**Author:** Franco Bonavento  
Data Analyst | Business Intelligence & Operations
