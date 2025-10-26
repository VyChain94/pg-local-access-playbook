# pg-local-access-playbook
PostgreSQL + pgAdmin Local User Access + Permission Management Playbook

# PostgreSQL & pgAdmin Local User Access + Permission Management Playbook

**Role:** Hybrid — DBA / Production Application Support Engineer 
**System:** macOS  
**Database:** PostgreSQL 17 (Homebrew)  
**GUI:** pgAdmin 4  
**Root Cause:** Authentication misconfiguration, port mismatch, and ownership ambiguity caused PostgreSQL service startup failures, connection errors, and unintended database visibility across roles.

---

## 1. Symptom Overview

- `brew services` showed service startup failures.  
- `psql` returned `connection refused` errors.  
- `pgAdmin` defaulted to port `5433` causing connection failures.  
- After fixing authentication, database `staging` appeared for multiple users unexpectedly.

**Diagnoses:**

- `pg_hba.conf` had invalid text blocks causing PostgreSQL startup failure.  
- Port mismatch between server and pgAdmin configuration.  
- `staging` database was owned by `postgres` but visible to all users due to PostgreSQL cluster-wide database model.

---

## 2. Resolution Timeline

### Phase 1 — Service Recovery & Authentication

1. Identified invalid connection type errors in `pg_hba.conf`.

2. Cleared template noise and applied minimal secure configuration:

   
   ```plaintext
   local   all             all                                     md5
   host    all             all             127.0.0.1/32            md5
   host    all             all             ::1/128                 md5

4. Restarted PostgreSQL successfully with:

   `brew services restart postgresql@17`
5. Verified interactive psql access and pgAdmin connection at 127.0.0.1:5432.

### Phase 2 — User Account Password Management
1. Reset ivytigsjr user password via:

`ALTER ROLE ivytigsjr WITH PASSWORD 'NewStrongPassword';`
2. Confirmed login with:

`psql -U ivytigsjr -h 127.0.0.1 -d postgres`
3. Updated pgAdmin connection properties to correct port and credentials.

### Phase 3 — Database Ownership & Access Control
1. Identified staging database owned by postgres.
2. Explained PostgreSQL cluster database visibility: all users see all databases they have privileges on.
3. Implement either of the two resolution paths:
- Option 1 - Transfer Ownership:
``ALTER DATABASE staging OWNER TO ivytigsjr;``
- Option 2 - Adjust Privileges:
``GRANT CONNECT ON DATABASE staging TO ivytigsjr;
GRANT ALL PRIVILEGES ON DATABASE staging TO ivytigsjr;``
4. Verified permissions via `\l` and pgAdmin UI.

## 3. Post-Resolution Commands Cheat Sheet
- List Databases:
`\l`
- List Roles:
`\du`
- Switch Databases:
`\c staging`
- Create Database for Specific User:
`CREATE DATABASE mydb OWNER ivytigsjr;`
- Grant Access:
`GRANT ALL PRIVILEGES ON DATABASE mydb TO ivytigsjr;`
- Reset Password:
`ALTER ROLE ivytigsjr WITH PASSWORD 'NewStrongPassword';`

## 4. Best Practices for App Support Engineers
- Keep pg_hba.conf minimal and clean. Remove example/template noise.

- Use md5 or scram-sha-256 auth; avoid trust mode.

- Create non-superuser accounts for application access.

- Separate database ownership and privileges for cleaner access control.

- Backup working configs:
`cp /opt/homebrew/var/postgresql@17/pg_hba.conf ~/pg_hba.conf.backup`
- Always verify port settings in GUI tools like pgAdmin.

- Separate ownership and privileges to avoid unintended visibility.

- Document and version changes for quick rollback.

## 5. Troubleshooting Port Mismatch
- Verify the actual port Postgres is running on:
`lsof -i :5432`
- If pgAdmin attempts port 5433, correct the Connection > Port field in the server settings.
- Default port: 5432.

## 6. Role, Database & Cluster Visibility Quick Reference
| Concept | Scope | Notes |
| :------- | :------: | -------: |
| Role (User) | Cluster-wide  | Can access any DB with privileges  |
| Database  | Cluster-wide  | Owned by a role but visible to others if privileges exist  |
| pg_hba.conf rules | Cluster-level access control | Determines how users authenticate |

## 7. Future Enhancements / Production Considerations
- Enable scram-sha-256 for stronger password hashing:
`password_encryption = scram-sha-256`
- Consider SSL/TLS (hostssl) for external connections.
- Set up replication if scaling is needed in the future.
- Automated script that backs up pg_hba.conf and audits role privileges (\du, \l, etc.)
- A Bash-based setup script that configures local PostgreSQL access for a dev team with secure defaults

##Final Status
- ✅ PostgreSQL running cleanly with secure authentication.

- ✅ PostgreSQL service restored and stable.

- ✅ Authentication and port configuration fixed.

- ✅ Ownership and privileges properly enforced.

- ✅ Full command history documented for escalation & future incident response.

- ✅ Both DBA and App Support functions were handled by a single engineer.

- Author: Ivy Tigs Jr — Hybrid DBA / Production Application Support Engineer
- Revision: 3.0
- Date: 2025-10-25
- Tags: `PostgreSQL` `pgAdmin` `DBA` `App Support` `Access Control` `Authentication` `Ownership` `Support Playbook` `Runbook`

