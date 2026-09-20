# Oracle 21c XE PDB Assignment

This repo covers Assignment II for the Oracle Multitenant course: creating a pluggable database, creating and deleting a temporary one, getting Enterprise Manager Express running, and writing this up.

## Environment

Windows machine, Oracle Database 21c XE, connecting through Oracle SQL Developer. There's actually two Oracle homes on this box, `dbhomeXE` and `homes\OraDB21Home1`, and the second one is broken (installer never finished, missing its `bin` folder entirely) even though the listener config points at it. That caused some pain in an earlier project (networked sqlplus connections kept throwing `ORA-12543` because `where sqlplus` resolved to the broken home), but SQL Developer doesn't care about any of that since it talks to the listener directly over JDBC instead of shelling out to a local sqlplus binary. So for this assignment everything just went through SQL Developer, connected as `sys` with the SYSDBA role, host `localhost`, port `1521`, SID `xe`.

## Task 1 — Creating a PDB and a user inside it

Naming convention was first-two-letters-of-first-name + `_pdb_` + student ID, so this came out to `AX_PDB_29273`. The one thing that tripped things up here was the seed path for `FILE_NAME_CONVERT`. The first guess was that it'd live under `dbhomeXE\oradata\...` since that's the "working" home from before, but it turned out to actually be one level up, at `C:\APP\MACAX\PRODUCT\21C\ORADATA\XE\PDBSEED\`. Confirmed it by querying `v$datafile` directly instead of guessing again.

Once the PDB was created and opened, creating the user (`axel_plsqlauca_29273`) went fine until the quota grant:

```
ALTER USER axel_plsqlauca_29273 QUOTA UNLIMITED ON USERS;
```

threw `ORA-00959: tablespace 'USERS' does not exist`. Turns out a PDB cloned straight from the bare `pdbseed` doesn't come with a `USERS` tablespace by default — only `XEPDB1` has one because it was provisioned with extras during install. Fix was just to create one:

```sql
CREATE TABLESPACE users
DATAFILE 'C:\APP\MACAX\PRODUCT\21C\ORADATA\XE\AX_PDB_29273\USERS01.DBF'
SIZE 100M AUTOEXTEND ON;
```

and then the quota grant went through fine.

## Task 2 — Create and delete a PDB

Same idea, named `AX_TO_DELETE_PDB_29273` per the naming convention. Created it, opened it, verified it existed via `dba_pdbs`, then closed it and dropped it:

```sql
ALTER PLUGGABLE DATABASE ax_to_delete_pdb_29273 CLOSE IMMEDIATE;
DROP PLUGGABLE DATABASE ax_to_delete_pdb_29273 INCLUDING DATAFILES;
```

Confirmed it was actually gone by re-running the `dba_pdbs` query and getting zero rows back. No real issues on this one, it went smoothly once Task 1 had already sorted out the path/tablespace stuff.

## Task 3 — Enterprise Manager Express

This one took a bit of back and forth. First attempt was logging into EM Express as `sys` and switching the container dropdown at the top to `AX_PDB_29273` — that technically works to show the PDB, but it doesn't really prove the PDB is "yours" the way logging in as your own user does.

So instead, tried logging in directly as `axel_plsqlauca_29273`. First attempt failed with `ORA-01017: invalid username/password`, and for a minute it looked like maybe SYS-based logins and PDB-user logins didn't mix, or like the account somehow needed re-authorizing after being switched. Turned out to be two separate silly things stacked on top of each other: the connection in SQL Developer had the role still set to SYSDBA (which the PDB user obviously isn't), and it was pointed at SID `xe` (the root container) instead of connecting by service name to `AX_PDB_29273`. Fixed both — role back to Default, switched from SID to Service Name and typed in `AX_PDB_29273` — and it connected fine.

Also granted `EM_EXPRESS_ALL` to the user explicitly just to be safe:

```sql
ALTER SESSION SET CONTAINER = AX_PDB_29273;
GRANT EM_EXPRESS_ALL TO axel_plsqlauca_29273;
```

After that, logging into `https://localhost:5500/em` with container `AX_PDB_29273` and the PDB user's credentials worked and showed the dashboard scoped to the right PDB with the username visible.

## Integrity statement

All work here was done individually, following the assignment's naming conventions and structure as given.

## Submission details

- **Repository Link:** https://github.com/Axello47/oracle_pdb_ass_II_29273_axel
- **PDB Name Created:** AX_PDB_29273
- **Issues Encountered:** Yes (see Task 1 and Task 3 notes above for what came up and how it got fixed)
