# Source Database (`AbsenceDB`)

This folder contains the SQL Server initialization and seed script (`AbsenceDB_source.sql`) for the **AbsenceDB** OLTP source database used in the BI & Data Mining project.

## Prerequisites

- **Microsoft SQL Server** (2017 or later recommended)
- **SQL Server Management Studio (SSMS)** (or `sqlcmd` / Azure Data Studio)

## How to Run `AbsenceDB_source.sql`

1. Open **SQL Server Management Studio (SSMS)** and connect to your local SQL Server instance.
2. Open `AbsenceDB_source.sql` (`File` $\rightarrow$ `Open` $\rightarrow$ `File...`).
3. Click **Execute** (or press `F5`).

> **Note:** The script switches to `master`, drops `AbsenceDB` if it already exists, recreates it from scratch, creates all tables and constraints, inserts the seed records, computes `taux_absenteisme`, and runs a verification query at the end.

## Database Schema Overview

| Table | Primary Key | Description |
| :--- | :--- | :--- |
| `Ville` | `id_ville` | Geographic reference table (`nom`, `region`, `pays`) across Morocco and Europe |
| `Employee` | `code_emp` | Employee records (`nom`, `prenom`, `service`) linked to `Ville` (`id_ville`) |
| `Categorie` | `id_categorie` | Absence categories (`nom_cat`, `description_`) such as annual leave, sick leave, maternity, etc. |
| `Demande_Absence` | `num_demande` | Absence requests containing `date_demande`, `date_debut`, `date_fin`, `status_`, and `id_categorie` |
| `Detail_Absence` | `num_demande` | Request details linking `num_demande` to `code_employee`, `nb_jour`, and `taux_absenteisme` |

## Expected Verification Output

At the end of execution, the script outputs row counts for all five tables. Confirm your result set matches:

| Table (`tbl`) | Expected Rows (`nb`) |
| :--- | :---: |
| `Ville` | 100 |
| `Employee` | 100 |
| `Categorie` | 12 |
| `Demande_Absence` | 100 |
| `Detail_Absence` | 100 |

## Business Rule: `taux_absenteisme`

The script automatically computes each employee's absenteeism rate (`taux_absenteisme`) across all rows belonging to that employee using only **approved** requests (`status_ = N'Approuvée'`) against a fixed annual theoretical baseline of 20 working days:

$$\text{taux\_absenteisme} = \frac{\sum \text{nb\_jour (Approuvée)}}{20} \times 100$$