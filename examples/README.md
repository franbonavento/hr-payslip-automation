# Synthetic example data

The files in this directory are **fictional** and exist only to illustrate the design of the production workflow.

They do not reproduce employee records, email addresses, branches, payroll information, identifiers, or other confidential information from the real implementation.

## employee_mapping.csv

Illustrates the concept of a deterministic mapping table used to associate a key extracted from a PDF file name with an employee and destination email.

The example intentionally includes an unresolved row (`REVIEW`) to show that ambiguous or incomplete cases should not proceed automatically.

The production implementation uses human review for exceptions rather than probabilistic matching.
