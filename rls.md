### Objective: Implement a database security model for the Pghyd organization to manage employee data while ensuring data privacy and compliance.

#### Key Requirements:
###### User Roles and Permissions:

1. **pghyd_viewer**: Limited access to specific columns (e.g., cannot see salary or phone number).
2. **pghyd_core**: Full access to all data and operations.
3. **pwi_core**: Manage PWI-specific records with full access.
4. **pwi_regular**: Read-only access to PWI records, with no modification rights.

#### Data Privacy and Security:

1. Implement Row-Level Security (RLS) to restrict data access based on roles.
2. Prevent certain roles from deleting records to maintain data integrity.

#### Column-Level Security:

1. Protect sensitive information (salary, phone number) using encryption and restrict access to specific roles.

#### Auditing and Compliance with pgAudit:

Ensure that all actions on employee records comply with data governance policies, with tracking of access for audits.


### Implementation: 

| **Role**                           | **Permissions**                                                               | **RLS Policies**                                                                            | **Notes**                                                                             |
|------------------------------------|-------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------|
| pghyd_viewer                       | - SELECT access to name, department, organization, and finalized columns only | - Can see all records in pghyd_employees, but limited to specified columns                  | Cannot see salary and phone_number columns due to view-based security.                |
| pghyd_core                         | - Full access (SELECT, INSERT, UPDATE, DELETE) to all columns                 | - Full access to all data (RLS policy allows USING (true))                                  | Can manage any record in the pghyd_employees table.                                   |
| pwi_core                           | - Full access (SELECT, INSERT, UPDATE, DELETE) to all columns                 | - Full access to PWI data (RLS policy allows USING (organization = 'PWI'))                  | Can manage records where the organization is 'PWI'.                                   |
| pwi_regular                        | - SELECT access to all columns                                                | - Can only select PWI data (RLS policy allows USING (organization = 'PWI'))                 | Restricted to non-finalized records (if additional restrictive policies are applied). |
| Restrictive Policy for pwi_regular | - Restricts DELETE operations on any records                                  | - Prevents the pwi_regular role from deleting any records (RLS policy using USING (FALSE))  | Ensures data integrity by preventing deletion.                                        |
| Restrictive Policy for pwi_core    | - Restricts DELETE operations on PWI records                                  | - Prevents the pwi_core role from deleting any PWI records (RLS policy using USING (FALSE)) | Ensures data integrity by preventing deletion of PWI records.                         |

### Scripts

```
-- Create the database
CREATE DATABASE pghyd_db;

-- Connect to the new database
\c pghyd_db;

-- Create the employees table
CREATE TABLE pghyd_employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50),
    department VARCHAR(50),
    salary NUMERIC,
    organization VARCHAR(50),
    finalized BOOLEAN,
    phone_number BYTEA,  -- Encrypted phone number using pgcrypto
    CONSTRAINT phone_number_encrypt CHECK (phone_number IS NOT NULL)
);

Step 2: Create Roles

-- Create roles
CREATE ROLE pghyd_viewer;
CREATE ROLE pghyd_core;
CREATE ROLE pwi_core;
CREATE ROLE pwi_regular;

Step 3: Grant Permissions to Roles

-- Grant full access to pghyd_core and pwi_core roles
GRANT SELECT, INSERT, UPDATE, DELETE ON pghyd_employees TO pghyd_core;
GRANT SELECT, INSERT, UPDATE, DELETE ON pghyd_employees TO pwi_core;

-- Grant SELECT access to pwi_regular role
GRANT SELECT ON pghyd_employees TO pwi_regular;

-- Grant SELECT access on few columns to pghyd_viewer
GRANT SELECT (name, department,organization, finalized) ON pghyd_employees TO pghyd_viewer;

Step 4: Enable Row-Level Security

-- Enable row-level security on the table
ALTER TABLE pghyd_employees ENABLE ROW LEVEL SECURITY;

Step 5: Create Row-Level Security Policies

-- pghyd_core: Full access to all data
CREATE POLICY pghyd_core_access ON pghyd_employees
FOR ALL
TO pghyd_core
USING (true);

-- pwi_core: Full access to PWI data
CREATE POLICY pwi_core_access ON pghyd_employees
FOR ALL
TO pwi_core
USING (organization = 'PWI');  -- Assuming organization is used to filter PWI data

-- pwi_regular: Select access to PWI data
CREATE POLICY pwi_regular_access ON pghyd_employees
FOR SELECT
TO pwi_regular
USING (organization = 'PWI');  -- Restricting to only PWI data

-- pwi_core: Cannot delete PWI data
CREATE POLICY restrict_deletes_pwi_core
ON pghyd_employees
AS RESTRICTIVE
FOR DELETE
TO pwi_core
USING (FALSE);

Step 6: Insert Sample Data

-- Sample data insertion (you will need to use pgcrypto for phone_number)
-- Sample data insertion with pgcrypto for phone_number

## Create extension pgcrypto

CREATE extension IF NOT EXISTS pgcrypto;

INSERT INTO pghyd_employees (name, department, salary, organization, finalized, phone_number)
VALUES
    ('Frank', 'Engineering', 85000, 'PWI', FALSE, pgp_sym_encrypt('8888888888', 'mysecretpass')),
    ('Grace', 'HR', 65000, 'PGHYD', TRUE, pgp_sym_encrypt('9999999999', 'mysecretpass')),
    ('Heidi', 'Marketing', 72000, 'PWI', FALSE, pgp_sym_encrypt('1231231234', 'mysecretpass')),
    ('Ivan', 'Marketing', 68000, 'PGHYD', TRUE, pgp_sym_encrypt('2342342345', 'mysecretpass')),
    ('Judy', 'Finance', 92000, 'PWI', FALSE, pgp_sym_encrypt('3453453456', 'mysecretpass')),
    ('Mallory', 'Engineering', 78000, 'PWI', TRUE, pgp_sym_encrypt('4564564567', 'mysecretpass')),
    ('Niaj', 'HR', 62000, 'PGHYD', FALSE, pgp_sym_encrypt('5675675678', 'mysecretpass')),
    ('Olivia', 'Finance', 82000, 'PGHYD', TRUE, pgp_sym_encrypt('6786786789', 'mysecretpass')),
    ('Peggy', 'Engineering', 80000, 'PWI', FALSE, pgp_sym_encrypt('7897897890', 'mysecretpass')),
    ('Trent', 'Marketing', 71000, 'PGHYD', TRUE, pgp_sym_encrypt('8908908901', 'mysecretpass'));
```

### Testing

```
Step 7: Testing Access
Now you can test access for different roles to ensure that the policies work as intended.
For pghyd_viewer User:

SET ROLE pghyd_viewer;
SELECT name, department,organization, finalized,phone_number
FROM pghyd_employees;  -- Should exclude salary and phone_number

SELECT name, department,organization, finalized
FROM pghyd_employees;  -- Should exclude salary and phone_number

For pwi_regular User:

SET ROLE pwi_regular;
SELECT id, name, department, organization, finalized
FROM pghyd_employees;  -- Should only see non-finalized PWI records

For pwi_core User:

SET ROLE pwi_core;
SELECT id, name, department, salary, organization, finalized, pgp_sym_decrypt(phone_number, 'mysecretpass') AS decrypted_phone_number
FROM pghyd_employees;  -- Full access to all records

For pghyd_core User:

SET ROLE pghyd_core;
SELECT id, name, department, salary, organization, finalized, pgp_sym_decrypt(phone_number, 'mysecretpass') AS decrypted_phone_number
FROM pghyd_employees;  -- Full access to all records

pghyd_db=# alter user pghyd_viewer BYPASSRLS;
ALTER ROLE
pghyd_db=#

-- Create pgaudit extension
dnf install pgaudit_16
shared_preload_libraries='pgaudit'
alter system set pgaudit.log='all';

-- Set up pgaudit

SELECT name, setting 
FROM pg_settings
WHERE name LIKE 'pgaudit%';
```
