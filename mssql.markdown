---
layout: default
title: MSSQL Server
permalink: /mssql-server/
---

# Tips on MSSQL  server 

<!-- This is a comment in Markdown -->

### Creation of a new table from an existing table with simillar structure.  

```sql
CREATE TABLE Table_1 AS SELECT * FROM Table_0 WHERE 1=0;  
--or   
SELECT * INTO Table_1 FROM Table_0 WHERE 1=0;
```
Code above creates a new table ``Table_1`` with similar structure to ``Table_0``
without rows of data this is ensured by condition ``WHERE 1=0`` therefore ``Table_1`` is empty.

<!-- This is a comment in Markdown -->

### Create a User and Assign to DB using mssql commands

```sql 
--set master database   
USE master;  
--create a login account  
CREATE  LOGIN  user_Database_X WITH PASSWORD='password';  
--select database to assign the login  
USE Database_X;  
--create user for the login account  
CREATE USER user_Database_X FROM LOGIN user_Database_X;  
--assign user roles to access the database  
EXEC sp_addrolemember db_owner, user_Database_X;
``` 
>In above command we first create a login account for **authentication** then the user account to that
database for **authorization**.  
The concept of Authentication and Authorization allows only valid & allowed users to access a database.    
Its paramount to ensure all logins don't have **sysadmin** role but only **public**.  
Only the **sa (system administrator)** account is given full rights to all databases.  
**Authentication:** ***proof of who you say you are***  
**Authorization:** ***proof of what access  you have***

### Difference between truncate and delete

**TRUNCATE** clears all rows in a table and resets the identity counter.  
**DELETE** removes one or more rows in a table but the identity counter is not reset.  

**TRUNCATE** is a fast  and can't be rolled back, whereas with **DELETE** its 
possible to roll back before the transaction is commited.
