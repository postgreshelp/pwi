```
GRANT SELECT (name, department,organization, finalized) ON pghyd_employees TO pghyd_viewer;

kfc_db=# create user kfc1 with password 'kfc1' noinherit;
CREATE ROLE
kfc_db=# create user kfc2 with password 'kfc2'
kfc_db-# ;
CREATE ROLE

kfc_db=# grant kfc_user to kfc2;
GRANT ROLE

kfc_db=# grant kfc_user to kfc1;
GRANT ROLE

[postgres@rel92 ~]$ psql -U kfc1 -d kfc_db
psql (17.0)
Type "help" for help.

kfc_db=> select * from kfc_user.emp;
ERROR:  permission denied for schema kfc_user
LINE 1: select * from kfc_user.emp;
                      ^
kfc_db=> \q
[postgres@rel92 ~]$ psql -U kfc2 -d kfc_db
psql (17.0)
Type "help" for help.

kfc_db=> select * from kfc_user.emp;
 id | sal 
----+-----
(0 rows)

kfc_db=> 

create table emp(id int, name varchar(100), phone int);
insert into emp values(1,'user1', 12345);
insert into emp values(2,'user2', 12345);
insert into emp values(3,'user3', 12345);

select * from emp;

create user user1 with password 'user1' noinherit;
create user user2 with password 'user2' noinherit;
create user user3 with password 'user3' noinherit;

grant select on emp to user1;
grant select on emp to user2;
grant select on emp to user3;

set role user1;
select * from emp;

\c - postgres
create policy user1_policy on emp for all to public using (name=current_user);
\c postgres user1

select * from emp;

SELECT nspname AS schema_name, 
       pg_catalog.pg_get_userbyid(nspowner) AS owner, 
       nspacl 
FROM pg_namespace 
;

Other commands

ALTER TABLE ... DISABLE ROW LEVEL SECURITY;
ALTER TABLE .. FORCE ROW LEVEL SECURITY;
ALTER TABLE .. NO FORCE ROW LEVEL SECURITY;

Ref: https://www.enterprisedb.com/postgres-tutorials/how-implement-column-and-row-level-security-postgresql

SELECT defaclrole::regrole AS grantor, defaclnamespace::regnamespace AS schema, defaclobjtype AS object_type, defaclacl AS privileges from pg_default_acl;

revoke usage on schema public from auser; ALTER DEFAULT PRIVILEGES IN SCHEMA public REVOKE ALL PRIVILEGES ON TABLES FROM postgres;


```
