# Part I Multitenant Architecture
The multitenant architecture enables an Oracle database to function as a multitenant container database (CDB).
https://docs.oracle.com/en/database/oracle/oracle-database/21/cncpt/multitenant-architecture.html#GUID-FC2EB562-ED31-49EF-8707-C766B6FE66B8

## 2 CDBs and PDBs

### About Containers in a CDB
A **container** is a collection of schemas, objects, and related structures in a **multitenant container database (CDB)**. Within a CDB, each container has a unique ID and name.

A **CDB** includes 0-N created pluggable databases (PDBs) and application containers

Figure 2-1 Containers in a CDB

![img_15.png](images%2Fimg_15.png)

Every CDB has the following containers:
- 1 CDB root container (also called simply the root `CDB$ROOT`)
  - The [CDB root](https://docs.oracle.com/en/database/oracle/oracle-database/21/cncpt/glossary.html#GUID-1C63428A-905A-4918-95AA-F3BBA3D2B825) is a collection of schemas, schema objects, and nonschema objects to which all PDBs belong 
  - The root stores Oracle-supplied metadata and common users
- 1 system container
  - The [system container](https://docs.oracle.com/en/database/oracle/oracle-database/21/cncpt/glossary.html#GUID-0E3286DF-A941-47D6-A2C8-333D9B4644B3) includes the root CDB and all PDBs in the CDB. Thus, the system container is the logical container for the CDB itself.
- 0-N application containers
- 0-N user-created PDBs
- 1 [seed PDB](https://docs.oracle.com/en/database/oracle/oracle-database/21/cncpt/glossary.html#GUID-D05C6A55-5452-4C82-942C-59DB245671BE)
  - The seed PDB is a system-supplied template that the CDB can use to create new PDBs.
  - The seed PDB is named `PDB$SEED`. 
  - You cannot add or modify objects in PDB$SEED.

### PDBs
A PDB is a user-created set of schemas, objects, and related structures that appears logically to a client application as a separate database.

Every PDB is owned by `SYS`, regardless of which user created the PDB. `SYS` is a [common user](https://docs.oracle.com/en/database/oracle/oracle-database/21/cncpt/glossary.html#GUID-879D936A-45FE-4E4D-8099-5060F3F55353) in the CDB, which means that this user that has the same identity in the root and in every existing and future PDB within the CDB.

#### Types of PDBs
All PDBs are user-created with the `CREATE PLUGGABLE DATABASE` statement except for `PDB$SEED`, which is Oracle-supplied.

You can create the following types of PDBs.

##### Standard PDB
`CREATE PLUGGABLE DATABASE` without specifying the PDB as a seed, proxy PDB, or application root.

##### Application Root
Consider an application root as an application-specific root container. It serves as a repository for a master definition of an application back end, including common data and metadata.
```sql
-- Create the application container called saas_sales_ac
CREATE PLUGGABLE DATABASE saas_sales_ac AS APPLICATION CONTAINER
  ADMIN USER saas_sales_ac_adm IDENTIFIED BY manager; 

-- Open the application root
ALTER PLUGGABLE DATABASE saas_sales_ac OPEN;
```

##### Seed PDBs
The seed is a template for the creation of PDBs that support applications
- Seed PDB plugged in the CDB root (`PDB$SEED`)
- Application seed PDB (`CREATE PDB ... AS SEED`)

##### Proxy PDBs
A [proxy PDB](https://docs.oracle.com/en/database/oracle/oracle-database/21/cncpt/glossary.html#GUID-57D0350C-AA01-441A-AB35-E7E76065BA21) refers to a remote PDB, called the **referenced PDB**. When you issue a statement in a proxy PDB while the PDB is open, the statement executes in the referenced PDB.

```sql
CONNECT sales_admin@saas_sales_ac
Password: ***********

CREATE PLUGGABLE DATABASE sales_sync_pdb AS PROXY FROM saas_sales_test_ac@cdb_dev_rem;
```

### Database Links Between PDBs
By default, a user connected to one PDB must use database links to access objects in a different PDB.

Figure 2-4 Database Link Between PDBs

In this illustration, a PDB administrator is connected to the PDB named `hrpdb1`. By default, during this user session,` c##dba` cannot query the `emp2` table in `hrpdb2` without specifying a database link.

![img_16.png](images%2Fimg_16.png)

## 3 Application Containers
Trong CDB, bạn có thể tạo vùng chứa dữ liệu ứng dụng và siêu dữ liệu mà các PDB có thể chia sẻ.

### About Application Containers
An **application container** is an optional, user-created CDB component that stores data and metadata for one or more application back ends.

Trong **application container**, **application** là tập hợp common data và metadata được đặt tên, phiên bản được lưu trữ trong application root.

The `CREATE PLUGGABLE DATABASE` statement with the `AS APPLICATION CONTAINER` clause creates the application root of the application container, and thus implicitly creates the application container itself. 
When you first create the application container, it contains no PDBs. To create application PDBs, you must connect to the application root, and then execute the `CREATE PLUGGABLE DATABASE` statement.

#### Purpose of Application Containers
In some ways, an application container functions as an application-specific CDB within a CDB. An application container, like the CDB itself, can include multiple PDBs, and enables these PDBs to share metadata and data.

The application root enables application PDBs to share an application, which in this context means a named, versioned set of common metadata and data. A typical application installs application common users, metadata-linked common objects, and data-linked common objects.

#### Application Root
https://docs.oracle.com/en/database/oracle/oracle-database/21/cncpt/app-containers.html#GUID-CA425AC1-CA1C-498C-BC09-AD246BA67346

#### Application PDBs
An application PDB is a PDB that resides in an application container. Every PDB in a CDB resides in either zero or one application containers.

#### Application Seed
An application seed is an optional, user-created PDB within an application container. An application container has either zero or one application seed.
