# Oracle 21c XE PDB Assignment

Shema Axel
ID: 29273

Assignment II for the Oracle Multitenant course. Covers creating a pluggable database and a user inside it, creating and deleting a temporary PDB, setting up Enterprise Manager Express, and writing it all up.

## Environment

Windows machine running Oracle Database 21c XE. Connected through Oracle SQL Developer as sys with the SYSDBA role, host localhost, port 1521, SID xe.

## Task 1: Creating a PDB and a user inside it

Created the pluggable database AX_PDB_29273 and opened it. After that, created a user inside it called axel_plsqlauca_29273 and gave it CONNECT, RESOURCE and DBA privileges plus a quota on the USERS tablespace.

```sql
CREATE PLUGGABLE DATABASE AX_PDB_29273
ADMIN USER pdbadmin IDENTIFIED BY "Oracle@121233"
FILE_NAME_CONVERT = ('C:\APP\MACAX\PRODUCT\21C\ORADATA\XE\PDBSEED\',
'C:\APP\MACAX\PRODUCT\21C\ORADATA\XE\AX_PDB_29273\');

ALTER PLUGGABLE DATABASE AX_PDB_29273 OPEN;

CREATE USER axel_plsqlauca_29273 IDENTIFIED BY "Oracle@121233";
GRANT CONNECT, RESOURCE, DBA TO axel_plsqlauca_29273;
CREATE TABLESPACE users
DATAFILE 'C:\APP\MACAX\PRODUCT\21C\ORADATA\XE\AX_PDB_29273\USERS01.DBF'
SIZE 100M AUTOEXTEND ON;
ALTER USER axel_plsqlauca_29273 QUOTA UNLIMITED ON USERS;
```

## Task 2: Creating and deleting a PDB

Created a temporary pluggable database called AX_TO_DELETE_PDB_29273, opened it, and confirmed it existed by querying dba_pdbs. Then closed it and dropped it, and confirmed it was gone by running the same query again and getting no rows back.

```sql
CREATE PLUGGABLE DATABASE AX_TO_DELETE_PDB_29273
ADMIN USER pdbadmin IDENTIFIED BY "Oracle@121233"
FILE_NAME_CONVERT = ('C:\APP\MACAX\PRODUCT\21C\ORADATA\XE\PDBSEED\',
'C:\APP\MACAX\PRODUCT\21C\ORADATA\XE\AX_TO_DELETE_PDB_29273\');

ALTER PLUGGABLE DATABASE AX_TO_DELETE_PDB_29273 OPEN;

ALTER PLUGGABLE DATABASE AX_TO_DELETE_PDB_29273 CLOSE IMMEDIATE;
DROP PLUGGABLE DATABASE AX_TO_DELETE_PDB_29273 INCLUDING DATAFILES;
```

## Task 3: Enterprise Manager Express

Logged into EM Express as the PDB user axel_plsqlauca_29273 rather than as sys, so the dashboard clearly shows it belongs to that account. Granted the user the EM_EXPRESS_ALL privilege first.

```sql
ALTER SESSION SET CONTAINER = AX_PDB_29273;
GRANT EM_EXPRESS_ALL TO axel_plsqlauca_29273;
```

Logged into https://localhost:5500/em using container AX_PDB_29273 and the PDB user's credentials. The dashboard loaded scoped to that PDB with the username visible in the corner.

## Integrity statement

All work here was done individually, following the assignment's naming conventions and structure.

## Submission details

Repository link: https://github.com/Axello47/oracle_pdb_ass_II_29273_axel
PDB name created: AX_PDB_29273
