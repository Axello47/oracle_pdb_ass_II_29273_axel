# Oracle 21c XE PDB Assignment

Assignment II for the Oracle Multitenant course — create a PDB, create/delete a temp PDB, get EM Express running, document it.

## Environment

Windows, Oracle 21c XE, SQL Developer connecting as `sys` (SYSDBA, host `localhost`, port `1521`, SID `xe`). There are two Oracle homes on the machine and one (`OraDB21Home1`) is a broken/incomplete install — not relevant here since SQL Developer connects over JDBC, not through the local sqlplus binary.

## Task 1 — Create PDB + user

- PDB: `AX_PDB_29273`, user: `axel_plsqlauca_29273`

**Issue:** `FILE_NAME_CONVERT` first guessed the seed path as under `dbhomeXE\oradata\...` — wrong. Fix: queried `v$datafile` directly, actual path was `C:\APP\MACAX\PRODUCT\21C\ORADATA\XE\PDBSEED\`.

**Issue:** `ALTER USER ... QUOTA UNLIMITED ON USERS` failed with `ORA-00959: tablespace 'USERS' does not exist`. A PDB cloned from bare `pdbseed` has no `USERS` tablespace. Fix: created one manually before granting quota.

```sql
CREATE TABLESPACE users
DATAFILE 'C:\APP\MACAX\PRODUCT\21C\ORADATA\XE\AX_PDB_29273\USERS01.DBF'
SIZE 100M AUTOEXTEND ON;
```

## Task 2 — Create and delete a PDB

- Temp PDB: `AX_TO_DELETE_PDB_29273`
- Created → verified via `dba_pdbs` → closed → dropped → re-checked `dba_pdbs` returned 0 rows.

No issues here (Task 1 already sorted out the path/tablespace setup).

## Task 3 — Enterprise Manager Express

Goal: log in as the PDB user (`axel_plsqlauca_29273`) directly, not as `sys`, so the dashboard is clearly tied to that account.

**Issue:** Login failed with `ORA-01017: invalid username/password`. Actual cause was two config mistakes in the SQL Developer connection used to test credentials, not the password: role was still set to `SYSDBA` (wrong — this is a regular user) and it was connecting to SID `xe` (root) instead of by service name to `AX_PDB_29273`. Fixed both.

Also granted the EM Express privilege explicitly:

```sql
ALTER SESSION SET CONTAINER = AX_PDB_29273;
GRANT EM_EXPRESS_ALL TO axel_plsqlauca_29273;
```

Logged into `https://localhost:5500/em` with container `AX_PDB_29273` and the PDB user's credentials — dashboard loaded scoped to that PDB, username visible.

## Integrity statement

All work done individually, following the assignment's naming conventions and structure.

## Submission details

- **Repository Link:** https://github.com/Axello47/oracle_pdb_ass_II_29273_axel
- **PDB Name Created:** AX_PDB_29273
- **Issues Encountered:** Yes (see Task 1 and Task 3 above)
