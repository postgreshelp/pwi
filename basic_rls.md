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

Other commands

ALTER TABLE ... DISABLE ROW LEVEL SECURITY;
ALTER TABLE .. FORCE ROW LEVEL SECURITY;
ALTER TABLE .. NO FORCE ROW LEVEL SECURITY;

Ref: https://www.enterprisedb.com/postgres-tutorials/how-implement-column-and-row-level-security-postgresql
