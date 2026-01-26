Live URL:  https://aminbiography.github.io/js-sql-iams/
 
---          
        
# JS + SQL for IAMS   

This repository documents **essential JavaScript and SQL skills** used in **Identity and Access Management Systems (IAMS)**, including automation, integrations, access governance, auditing, and reporting.
Each section includes **examples and expected outputs**.

---   

## Core Concepts

```
JavaScript: variables, functions, async, APIs, JSON, validation, logging
SQL: select, joins, constraints, indexing, transactions, auditing, RBAC models
IAMS: users, roles, permissions, groups, entitlements, access requests, audit trails
```

---

## 1) Environment Setup

### JavaScript (Node.js)

Check Node and npm:

```bash
node -v
npm -v
```

**Output (example):**

```text
v20.11.1
10.2.4
```

Initialize a project:

```bash
mkdir js-sql-iams
cd js-sql-iams
npm init -y
```

**Output (example):**

```text
Wrote to .../package.json
```

### SQL (PostgreSQL or MySQL)

Check database client:

```bash
psql --version
# or
mysql --version
```

---

## 2) JavaScript Basics (Variables and Types)

```js
const username = "amein";
let active = true;
let attempts = 3;

console.log(username);
console.log(active);
console.log(attempts);
```

**Output:**

```text
amein
true
3
```

---

## 3) Objects and JSON (IAMS Data Model)

```js
const user = {
  id: "u_1001",
  name: "Alice",
  email: "alice@example.com",
  status: "ACTIVE",
  roles: ["IAM_ADMIN", "AUDITOR"]
};

console.log(user.email);
console.log(JSON.stringify(user));
```

**Output:**

```text
alice@example.com
{"id":"u_1001","name":"Alice","email":"alice@example.com","status":"ACTIVE","roles":["IAM_ADMIN","AUDITOR"]}
```

---

## 4) Functions (Access Decision Example)

```js
function hasRole(userRoles, requiredRole) {
  return userRoles.includes(requiredRole);
}

console.log(hasRole(["USER", "AUDITOR"], "AUDITOR"));
console.log(hasRole(["USER"], "AUDITOR"));
```

**Output:**

```text
true
false
```

---

## 5) Conditions and Validation (Login Guard)

```js
function canLogin(isActive, attemptsLeft) {
  if (!isActive) return "BLOCKED: inactive account";
  if (attemptsLeft <= 0) return "BLOCKED: too many attempts";
  return "ALLOW";
}

console.log(canLogin(true, 2));
console.log(canLogin(true, 0));
console.log(canLogin(false, 5));
```

**Output:**

```text
ALLOW
BLOCKED: too many attempts
BLOCKED: inactive account
```

---

## 6) Arrays (Entitlements)

```js
const entitlements = ["READ_REPORTS", "EXPORT_DATA", "MANAGE_USERS"];
console.log(entitlements[0]);
console.log(entitlements.length);
```

**Output:**

```text
READ_REPORTS
3
```

---

## 7) Loops (Provisioning Simulation)

```js
const roles = ["USER", "APP_ADMIN", "AUDITOR"];

for (const role of roles) {
  console.log("Provisioning role:", role);
}
```

**Output:**

```text
Provisioning role: USER
Provisioning role: APP_ADMIN
Provisioning role: AUDITOR
```

---

## 8) Error Handling (Fail Safe Access)

```js
function parseAccessPolicy(jsonText) {
  try {
    return JSON.parse(jsonText);
  } catch {
    return { error: "Invalid policy JSON" };
  }
}

console.log(parseAccessPolicy('{"rule":"ALLOW"}'));
console.log(parseAccessPolicy("{bad-json}"));
```

**Output:**

```text
{ rule: 'ALLOW' }
{ error: 'Invalid policy JSON' }
```

---

## 9) Async and APIs (Identity Provider Integration)

Example using `fetch` in Node (Node 18+):

```js
async function getUsers() {
  const res = await fetch("https://jsonplaceholder.typicode.com/users");
  console.log("Status:", res.status);

  const data = await res.json();
  console.log("Count:", data.length);
}

getUsers();
```

**Output (example):**

```text
Status: 200
Count: 10
```

---

## 10) SQL Fundamentals (Users Table)

```sql
CREATE TABLE users (
  id          SERIAL PRIMARY KEY,
  username    VARCHAR(50) UNIQUE NOT NULL,
  email       VARCHAR(120) UNIQUE NOT NULL,
  status      VARCHAR(20) NOT NULL DEFAULT 'ACTIVE',
  created_at  TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

Insert users:

```sql
INSERT INTO users (username, email) VALUES
('alice', 'alice@example.com'),
('bob', 'bob@example.com');
```

Query:

```sql
SELECT id, username, status FROM users;
```

**Output (example):**

```text
1 | alice | ACTIVE
2 | bob   | ACTIVE
```

---

## 11) Roles, Permissions, and Mapping (Core IAMS Schema)

```sql
CREATE TABLE roles (
  id   SERIAL PRIMARY KEY,
  name VARCHAR(50) UNIQUE NOT NULL
);

CREATE TABLE permissions (
  id   SERIAL PRIMARY KEY,
  name VARCHAR(80) UNIQUE NOT NULL
);

CREATE TABLE user_roles (
  user_id INT REFERENCES users(id),
  role_id INT REFERENCES roles(id),
  PRIMARY KEY (user_id, role_id)
);

CREATE TABLE role_permissions (
  role_id       INT REFERENCES roles(id),
  permission_id INT REFERENCES permissions(id),
  PRIMARY KEY (role_id, permission_id)
);
```

---

## 12) SQL Joins (Who Has What Access)

Example query: list users with their roles:

```sql
SELECT u.username, r.name AS role
FROM users u
JOIN user_roles ur ON ur.user_id = u.id
JOIN roles r ON r.id = ur.role_id
ORDER BY u.username;
```

**Output (example):**

```text
alice | IAM_ADMIN
alice | AUDITOR
bob   | USER
```

---

## 13) Least Privilege Check (Permissions by User)

```sql
SELECT u.username, p.name AS permission
FROM users u
JOIN user_roles ur ON ur.user_id = u.id
JOIN role_permissions rp ON rp.role_id = ur.role_id
JOIN permissions p ON p.id = rp.permission_id
ORDER BY u.username, p.name;
```

**Output (example):**

```text
alice | MANAGE_USERS
alice | READ_REPORTS
bob   | READ_REPORTS
```

---

## 14) SQL Constraints and Integrity (Prevent Bad Data)

Ensure status is controlled:

```sql
ALTER TABLE users
ADD CONSTRAINT chk_status CHECK (status IN ('ACTIVE', 'LOCKED', 'DISABLED'));
```

Try invalid insert:

```sql
INSERT INTO users (username, email, status)
VALUES ('eve', 'eve@example.com', 'HACKED');
```

**Output (example):**

```text
ERROR:  new row for relation "users" violates check constraint "chk_status"
```

---

## 15) Transactions (Safe Provisioning)

```sql
BEGIN;

INSERT INTO users (username, email) VALUES ('charlie', 'charlie@example.com');

-- assign USER role id=1 (example)
INSERT INTO user_roles (user_id, role_id) VALUES (3, 1);

COMMIT;
```

If something fails:

```sql
ROLLBACK;
```

---

## 16) Auditing (Track Access Changes)

```sql
CREATE TABLE audit_log (
  id         SERIAL PRIMARY KEY,
  actor      VARCHAR(50) NOT NULL,
  action     VARCHAR(100) NOT NULL,
  target     VARCHAR(100) NOT NULL,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

Insert an audit record:

```sql
INSERT INTO audit_log (actor, action, target)
VALUES ('iam_service', 'ASSIGN_ROLE', 'user=alice role=IAM_ADMIN');
```

Query:

```sql
SELECT actor, action, target FROM audit_log ORDER BY id DESC LIMIT 1;
```

**Output:**

```text
iam_service | ASSIGN_ROLE | user=alice role=IAM_ADMIN
```

---

## 17) JS + SQL Together (Provision Role Workflow)

Pseudo workflow:

1. Validate request (JS)
2. Write role assignment (SQL)
3. Write audit trail (SQL)
4. Return success (JS)

Example JS (conceptual):

```js
function validateRole(role) {
  const allowed = ["USER", "APP_ADMIN", "IAM_ADMIN", "AUDITOR"];
  if (!allowed.includes(role)) throw new Error("Role not allowed");
  return true;
}

try {
  validateRole("IAM_ADMIN");
  console.log("Validation OK. Proceed to SQL transaction.");
} catch (e) {
  console.log("Blocked:", e.message);
}
```

**Output:**

```text
Validation OK. Proceed to SQL transaction.
```

---

## 18) Advanced SQL: Indexing (Faster Access Lookups)

```sql
CREATE INDEX idx_user_roles_user_id ON user_roles(user_id);
CREATE INDEX idx_role_permissions_role_id ON role_permissions(role_id);
```

---

## 19) Advanced JS: Logging for Security Events

```js
function logSecurityEvent(eventType, details) {
  console.log(JSON.stringify({
    eventType,
    details,
    timestamp: new Date().toISOString()
  }));
}

logSecurityEvent("LOGIN_FAILURE", { username: "alice", ip: "45.33.32.156" });
```

**Output (example):**

```text
{"eventType":"LOGIN_FAILURE","details":{"username":"alice","ip":"45.33.32.156"},"timestamp":"2025-12-15T01:40:12.123Z"}
```

---

## 20) IAMS Practical Use Cases

* User onboarding and offboarding automation
* Role-based access control (RBAC) and permission mapping
* Audit reports (who has access, when it changed, who approved)
* Detect risky access (admin roles, dormant accounts, privilege creep)
* Integrations with IdPs (Google Workspace, Azure AD, Okta, Keycloak) via APIs
* Access request workflows and approvals

---

