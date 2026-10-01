# Script-Controlled ACL – Restrict Record Access Based on Field Value

## 📌 Project Overview

This ServiceNow lab demonstrates how to implement **script-controlled Access Control Lists (ACLs)** to restrict access to records based on user roles and the value of a specific field.

In this scenario:

* Users with the **bb1** role can view only records where **Branch = EEE**.
* Users with **bb2** can create records.
* Users with **bb3** can edit records.
* Users with **bb4** can delete records.
* **Administrators** have full access to all records.

This lab demonstrates record-level security using **READ, CREATE, WRITE, and DELETE ACLs**.

---

## 🎯 Objectives

* Create users and custom roles in ServiceNow.
* Create a custom table for student/institution records.
* Create different types of ACLs.
* Use scripts to control record-level access.
* Restrict READ access based on the `Branch` field.
* Control CREATE, WRITE, and DELETE operations using roles.
* Verify ACL behavior using user impersonation.

---

## 🛠️ Technologies Used

* **ServiceNow**
* Access Control Lists (ACL)
* JavaScript
* ServiceNow Roles
* Custom Tables
* User Administration

---

# 1. 👤 User Creation

Log in to the ServiceNow instance with **Admin** access.

Navigate to:

**User Administration → Users → New**

Create the following user:

| Field      | Value                                         |
| ---------- | --------------------------------------------- |
| User ID    | EEE User                                      |
| First Name | EEE                                           |
| Last Name  | User                                          |
| Email      | [eeeuser@gmail.com](mailto:eeeuser@gmail.com) |

Save the user record.

---

# 2. 🔐 Role Creation

Navigate to:

**User Administration → Roles → New**

Create the following custom roles:

* `bb1`
* `bb2`
* `bb3`
* `bb4`

Example:

| Role | Purpose       |
| ---- | ------------- |
| bb1  | READ access   |
| bb2  | CREATE access |
| bb3  | WRITE access  |
| bb4  | DELETE access |

Assign the required roles to the **EEE User**.

---

# 3. 🗃️ Custom Table Creation

Navigate to:

**System Definition → Tables → New**

Create the following table:

| Field         | Value                 |
| ------------- | --------------------- |
| Label         | Institution Details   |
| Name          | u_institution_details |
| Extends Table | False                 |

Click **Submit**.

---

# 4. 📋 Table Fields

Add the following fields to the `u_institution_details` table:

| Field               | Type         | Values / Description    |
| ------------------- | ------------ | ----------------------- |
| Student Roll Number | Auto Number  | Automatically generated |
| Student Name        | Reference    | User                    |
| Faculty Name        | Reference    | User                    |
| Branch              | Choice       | ECE, EEE, CSE           |
| Email               | String       | Student email           |
| Phone Number        | String       | Student phone           |
| Description         | Multi String | Additional information  |

Save the table.

Create multiple records with different branch values such as:

* EEE
* ECE
* CSE

---

# 5. 🔒 READ ACL

Navigate to:

**System Security → Access Control (ACL)**

Elevate the **security_admin** role.

Click **New**.

Configure the ACL as follows:

| Field     | Value                 |
| --------- | --------------------- |
| Type      | Record                |
| Operation | Read                  |
| Name      | u_institution_details |
| Active    | True                  |
| Advanced  | True                  |

### Required Role

Add:

`bb1`

### Data Condition

Set:

**Branch is EEE**

### Script

```javascript
(function () {
    // Allow admin users full access
    if (gs.hasRole('admin')) {
        return true;
    }

    // Allow users with bb1 role
    if (gs.hasRole('bb1')) {
        return true;
    }

    // Deny access for all others
    return false;
})();
```

Click **Submit**.

---

## 🧪 READ ACL Verification

Impersonate the **EEE User** and open the table list.

Use:

```text
u_institution_details.list
```

Verify that the user can access the appropriate EEE records.

Then impersonate an **Admin** user and open the same list.

### Expected Result

| User               | Expected Access       |
| ------------------ | --------------------- |
| User with `bb1`    | Can view EEE records  |
| User without `bb1` | Cannot access records |
| Admin              | Can view all records  |

---

# 6. ➕ CREATE ACL

Create another ACL with:

| Field     | Value                 |
| --------- | --------------------- |
| Type      | Record                |
| Operation | Create                |
| Name      | u_institution_details |
| Active    | True                  |

### Required Role

Add:

`bb2`

No data condition is required.

Save the ACL.

### Verification

A user having:

* `bb1`
* `bb2`

can:

* View EEE records
* See the **New** button
* Create new records

---

# 7. ✏️ WRITE ACL

Create another ACL:

| Field     | Value                 |
| --------- | --------------------- |
| Type      | Record                |
| Operation | Write                 |
| Name      | u_institution_details |
| Active    | True                  |

### Required Role

Add:

`bb3`

No data condition is required.

Save the ACL.

### Verification

A user having:

* `bb1`
* `bb2`
* `bb3`

can:

* View EEE records
* Create records
* Edit records

---

# 8. 🗑️ DELETE ACL

Create another ACL:

| Field     | Value                 |
| --------- | --------------------- |
| Type      | Record                |
| Operation | Delete                |
| Name      | u_institution_details |
| Active    | True                  |

### Required Role

Add:

`bb4`

No data condition is required.

Save the ACL.

### Verification

A user having:

* `bb1`
* `bb2`
* `bb3`
* `bb4`

can:

* View EEE records
* Create records
* Edit records
* Delete records

---

# 🔐 ACL Access Summary

| Operation | Required Role | Access           |
| --------- | ------------- | ---------------- |
| READ      | `bb1`         | View EEE records |
| CREATE    | `bb2`         | Create records   |
| WRITE     | `bb3`         | Edit records     |
| DELETE    | `bb4`         | Delete records   |
| ADMIN     | `admin`       | Full access      |

---

# 🔄 Access Flow

```text
                    ┌─────────────────────┐
                    │      User Login     │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │    Check ACLs       │
                    └──────────┬──────────┘
                               │
              ┌────────────────┼────────────────┐
              │                │                │
              ▼                ▼                ▼
           READ             CREATE            WRITE
           bb1               bb2               bb3
              │                │                │
              ▼                ▼                ▼
        EEE Records        New Record       Edit Record
              │
              └────────────────┬───────────────┐
                               │
                               ▼
                            DELETE
                              bb4
                               │
                               ▼
                         Delete Record
```

---

# 🧪 Testing

The ACL configuration should be tested using ServiceNow's **Impersonate User** functionality.

### Test 1 – READ

Login/impersonate a user with `bb1`.

Expected:

```text
EEE records → Accessible
Other branch records → Restricted
```

### Test 2 – CREATE

Add `bb2` to the user.

Expected:

```text
New button → Available
```

### Test 3 – WRITE

Add `bb3`.

Expected:

```text
Edit existing accessible records → Allowed
```

### Test 4 – DELETE

Add `bb4`.

Expected:

```text
Delete → Allowed
```

### Test 5 – ADMIN

Impersonate an administrator.

Expected:

```text
All records → Accessible
All operations → Allowed
```

---

# 📊 Final Permission Model

```text
                 EEE User
                    │
          ┌─────────┴─────────┐
          │                   │
        Roles               Records
          │                   │
   ┌──────┼──────┐            │
   │      │      │            │
  bb1    bb2    bb3    bb4     │
   │      │      │      │      │
   ▼      ▼      ▼      ▼      ▼
 READ   CREATE  WRITE  DELETE  Branch
                               │
                         ┌─────┴─────┐
                         │           │
                        EEE       ECE/CSE
                         │           │
                      Allowed     Restricted
```

---

# 🎓 Learning Outcomes

After completing this lab, learners understand:

* How ServiceNow ACLs control access to records.
* How to create and assign custom roles.
* How to create custom ServiceNow tables.
* How READ ACLs restrict record visibility.
* How CREATE ACLs control record creation.
* How WRITE ACLs control record modification.
* How DELETE ACLs control record deletion.
* How JavaScript can be used inside ACL scripts.
* How roles and record field values can be combined to enforce security.
* How to test ACLs using user impersonation.

---

# 🏁 Outcome

This lab demonstrates how **script-controlled READ, CREATE, WRITE, and DELETE ACLs** can be used to enforce record-level security in ServiceNow.

The implementation ensures that access to the **Institution Details** table is controlled according to user roles and branch information, while administrators retain full access.

The lab provides practical experience in protecting sensitive records from unauthorized viewing, creation, modification, and deletion.

---

## 📁 Project Structure

A suggested GitHub repository structure is:

```text
Script-Controlled-ACL/
│
├── README.md
│
├── ACL/
│   ├── Read-ACL.txt
│   ├── Create-ACL.txt
│   ├── Write-ACL.txt
│   └── Delete-ACL.txt
│
├── Screenshots/
│   ├── User-Creation.png
│   ├── Role-Creation.png
│   ├── Table-Creation.png
│   ├── Read-ACL.png
│   ├── Create-ACL.png
│   ├── Write-ACL.png
│   └── Delete-ACL.png
│
└── Documentation/
    └── Test-Results.md
```

---

## 👨‍💻 Lab Information

**Platform:** ServiceNow
**Lab:** Script-Controlled ACL
**Table:** `u_institution_details`
**Custom Roles:** `bb1`, `bb2`, `bb3`, `bb4`
**Security Administrator:** `security_admin`
