# R-OLAP Cube & Warehouse Schema (`AbsenceDW`)

This folder documents the **R-OLAP (Relational OLAP)** star schema / analytical cube structure stored in SQL Server (`AbsenceDW`) after running the Talend ETL jobs.

## Prerequisites

1. **Source Database Populated:** `AbsenceDB` is created and populated via `01-source-db/AbsenceDB_source.sql`.
2. **Target Database & ETL Completed:** `AbsenceDW` exists and the Talend jobs in `02-talend-etl/` (`jDim_Categorie`, `jDim_Employee`, `jDim_Temps2`, and `jFait_Absence`) have been executed to populate the warehouse tables.

## R-OLAP Cube / Star Schema Overview

In an **R-OLAP** architecture, multidimensional OLAP concepts (dimensions, hierarchies, facts, and measures) are stored directly in relational SQL Server tables rather than a separate MOLAP cube file:

* **Fact Table (`fait_Absence`)** — Stored at the atomic request grain (`num_demande`, 100 rows), with measures (`nb_jour`, `taux_absenteisme`), dimension foreign keys (`code_emp`, `id_categorie`), and role-playing date surrogate keys (`date_demande`, `date_debut`, `date_fin`).
* **Dimension Tables:**
  * `dim_Categorie` — Absence category attributes (`id_categorie`, `nom_cat`, `description_`).
  * `dim_Employee` — Employee and organizational/geographic attributes (`code_emp`, `nom`, `prenom`, `service`, `ville`, `region`, `pays`), supporting roll-ups such as `service → ville → region → pays`.
  * `dim_Temps` — Calendar dimension (`SK_Temps` as `yyyyMMdd` `INT`, `date_complete`, `jour`, `mois`, `trimestre`, `annee`) supporting role-playing date relationships for `date_demande`, `date_debut`, and `date_fin`.

## How to Use `rolap_schema.sql`

1. Open **SQL Server Management Studio (SSMS)** and connect to your SQL Server instance.
2. Open `03-dw-rolap/rolap_schema.sql`.
3. Execute the script (`F5`) against an empty `AbsenceDW` database before loading via Talend (or run the `ALTER TABLE ... FOREIGN KEY` statements against an already-populated `AbsenceDW` database to enforce referential integrity).

## Verification Checklist

* Confirm `fait_Absence` has **100** rows and `dim_Categorie` has **12** rows.
* Confirm all 5 foreign key constraints on `fait_Absence` (`code_emp`, `id_categorie`, `date_demande`, `date_debut`, `date_fin`) are enabled and have no orphaned keys.
* Confirm `taux_absenteisme` and `nb_jour` aggregate cleanly across time (`annee`, `trimestre`, `mois`, `jour`) and employee/location attributes (`service`, `ville`, `region`).