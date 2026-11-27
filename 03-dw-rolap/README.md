# R-OLAP Warehouse & SSAS Multidimensional Cube (`AbsenceDW`)

This folder contains the **R-OLAP (Relational OLAP)** star schema DDL (`rolap_schema.sql`) in SQL Server (`AbsenceDW`) and the **SQL Server Analysis Services (SSAS) Multidimensional** project folder (`AbsenceDW_Cube/`) built on top of it.

## Prerequisites

1. **Source Database Populated:** `AbsenceDB` is created and populated via `01-source-db/AbsenceDB_source.sql`.
2. **Warehouse & ETL Completed:** `AbsenceDW` is created and populated by running the Talend jobs in `02-talend-etl/` (`jDim_Categorie`, `jDim_Employee`, `jDim_Temps2`, and `jFait_Absence`).
3. **SSAS Service Permissions:** The local SQL Server Analysis Services instance (`localhost`) has read access (`db_datareader`) to `AbsenceDW`.

## Folder Structure (`03-dw-rolap/`)

- `rolap_schema.sql` — SQL Server DDL script defining the `AbsenceDW` star schema tables and foreign key constraints.
- `AbsenceDW_Cube/` — Visual Studio Analysis Services Multidimensional project folder containing:
  - Solution / project files (`.sln`, `.dwproj`)
  - Data Source (`.ds`) pointing to SQL Server and `AbsenceDW`
  - Data Source View (`.dsv`) mapping `fait_Absence`, `dim_Categorie`, `dim_Employee`, and `dim_Temps`
  - Dimension definitions (`.dim`) for `Dim Categorie`, `Dim Employee`, and `Dim Temps` (including role-playing date dimensions and hierarchies)
  - Cube definition (`.cube`) for `Absence DW`

## Star Schema & Cube Architecture

- **Fact Table (`fait_Absence`)** — Stored at the atomic request grain (`num_demande`, 100 rows), containing base measures (`nb_jour`, `taux_absenteisme`), dimension foreign keys (`code_emp`, `id_categorie`), and role-playing date keys (`date_demande`, `date_debut`, `date_fin`).
- **Dimensions & Hierarchies:**
  - **`Dim Categorie` (`dim_Categorie`)** — Absence category attributes (`id_categorie`, `nom_cat`, `description_`).
  - **`Dim Employee` (`dim_Employee`)** — Employee and organizational/geographic attributes (`code_emp`, `nom`, `prenom`, `service`, `ville`, `region`, `pays`) with the **`Localisation`** user-defined hierarchy (`Pays → Region → Ville`).
  - **`Dim Temps` (`dim_Temps`)** — Calendar dimension (`SK_Temps` as `yyyyMMdd` `INT`, `date_complete`, `jour`, `mois`, `trimestre`, `annee`) supporting three role-playing cube dimensions (`Date Demande`, `Date Debut`, `Date Fin`) and the **`Temps`** hierarchy (`Annee → Trimestre → Mois → Jour`), configured with composite `KeyColumns` (e.g., `annee` + `trimestre`) and clean `NameColumn` labels to avoid cross-year attribute collisions.

## How to Use `AbsenceDW_Cube/` and `rolap_schema.sql`

1. **Apply Relational Schema (`rolap_schema.sql`):**
   - Connect to SQL Server in **SSMS**, open `03-dw-rolap/rolap_schema.sql`, and execute (`F5`) against `AbsenceDW` to create or enforce the star schema tables and foreign keys.
2. **Open & Configure the SSAS Project (`AbsenceDW_Cube/`):**
   - Double-click the `.slnx` file inside `AbsenceDW_Cube/` to open it in **Visual Studio** (with the Microsoft Analysis Services Projects extension installed).
   - In **Solution Explorer**, open **Data Sources** and update the connection string if your SQL Server instance name or authentication differs from the original machine.
   - Check project deployment properties (`Project → Properties → Deployment`) to verify the target **Server** (`localhost`) and **Database** (`AbsenceDW_Cube`).
3. **Build, Deploy & Process:**
   - Right-click the project in **Solution Explorer** and select **Deploy** (or press `F5` / Build + Deploy) to push metadata changes to `localhost` and process the dimensions and `Absence DW` cube.
   - To refresh cube data after re-running Talend ETL jobs without changing structure, open the cube/dimensions or connect in **SSMS** to `localhost` and run **Process** on `AbsenceDW_Cube`.

## Validation (SSMS MDX & Cube Browser)

- **Data Integrity:** `fait_Absence` contains **100** rows, `dim_Categorie` contains **12** rows, and all 5 foreign keys on `fait_Absence` are active with zero orphaned keys.
- **Cube Processing:** Deployment and processing complete with **0 errors and 0 warnings**.
- **MDX & Hierarchy Verification:** Verified in SSMS via MDX queries (`Descendants(...)` and level `.Members` queries) that `Temps` (`Annee → Trimestre → Mois`) and `Localisation` (`Pays → Region → Ville`) roll up accurately across all levels.