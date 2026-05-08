# Active Directory

Active Directory Domain Services (AD DS) is Microsoft’s centralized directory service for Windows domain environments. It acts as a network catalog that stores information about users, computers, groups, organizational units, and other domain objects.

In a Windows enterprise network, Active Directory provides identity, authentication, authorization, policy management, and centralized administration.

---

## Active Directory Domain Services

AD DS stores information about network objects and allows administrators to manage them from a central location.

Common Active Directory objects include:

- User accounts
- Computer accounts
- Security groups
- Organizational Units
- Domain Controllers
- Group Policy Objects

These objects allow administrators to control who can log in, what resources they can access, and what security policies apply to them.

---

## Users

User accounts represent people or services that need to authenticate to the domain.

Users are **security principals**, meaning they can be authenticated and assigned permissions.

Examples include:

- Employee accounts
- Administrator accounts
- Service accounts for applications such as IIS, MSSQL, or backup software

Service accounts should follow the principle of least privilege. They should only have the permissions required for the service they support.

---

## Machines

When a computer joins an Active Directory domain, a computer account is created for it.

Computer accounts are also security principals. They have limited domain rights and are used by the domain to identify and authenticate the machine.

Computer accounts usually follow this naming format:

```text
COMPUTERNAME$
```

Example:

```text
DC01$
```

Machine account passwords are long, randomly generated, and automatically rotated by Windows.

---

## Security Groups

Security groups are used to assign permissions to multiple users or machines at once.

Instead of assigning access rights directly to individual users, administrators place users into groups and assign permissions to the group.

Groups can contain:

- Users
- Computers
- Other groups

Common default groups include:

| Group | Purpose |
|---|---|
| Domain Admins | Full administrative privileges over the domain |
| Server Operators | Can administer Domain Controllers |
| Backup Operators | Can access files for backup purposes |
| Account Operators | Can create and modify domain accounts |
| Domain Users | Contains standard domain user accounts |
| Domain Computers | Contains domain-joined computers |
| Domain Controllers | Contains Domain Controllers |

Security groups are mainly used for **access control**.

---

## Organizational Units

Organizational Units, or OUs, are containers used to organize users, computers, and other objects.

OUs are commonly structured around:

- Departments
- Locations
- Device types
- Administrative responsibilities

Examples:

```text
Sales
Marketing
IT
Workstations
Servers
Domain Controllers
```

OUs are important because they allow administrators to apply policies to specific sets of objects.

A user or computer can only exist in one OU at a time, but it can be a member of multiple security groups.

---

## OUs vs Security Groups

OUs and security groups are often confused, but they serve different purposes.

| Feature | Organizational Units | Security Groups |
|---|---|---|
| Primary use | Apply policies and delegate administration | Grant access to resources |
| Membership | Object exists in one OU | Object can belong to many groups |
| Used with | Group Policy and delegation | File shares, permissions, access control |
| Example | Put laptops in a Workstations OU | Add Alice to Helpdesk group |

A simple way to remember the difference:

```text
OUs organize and apply policy.
Groups grant access.
```

---

## Default Containers

Active Directory includes several default containers.

| Container | Purpose |
|---|---|
| Builtin | Default groups available to Windows hosts |
| Computers | Default location for newly joined computers |
| Domain Controllers | Contains all Domain Controllers |
| Users | Default users and groups |
| Managed Service Accounts | Stores managed service accounts |

The default **Computers** container is not ideal for long-term organization because Group Policy and delegation are easier to manage with a clean OU structure.

---

## Managing Active Directory

The main graphical management tool is **Active Directory Users and Computers**.

It can be used to:

- Create users
- Delete users
- Reset passwords
- Create groups
- Manage group membership
- Create OUs
- Move objects between OUs
- Delegate administrative control

---

## Protected OUs

OUs often have accidental deletion protection enabled by default.

If an administrator tries to delete a protected OU, Active Directory blocks the action.

To delete a protected OU:

1. Open **Active Directory Users and Computers**.
2. Enable **Advanced Features** from the View menu.
3. Right-click the OU.
4. Open **Properties**.
5. Go to the **Object** tab.
6. Uncheck **Protect object from accidental deletion**.
7. Delete the OU.

Deleting an OU also deletes its child objects, so this action should be performed carefully.

---

## Delegation

Delegation allows administrators to give specific users limited administrative control over an OU without making them Domain Admins.

Common delegated tasks include:

- Resetting passwords
- Unlocking accounts
- Creating users
- Modifying group membership
- Managing a department-specific OU

Example:

A helpdesk employee may be delegated permission to reset passwords for users in the Sales OU, without having access to the entire domain.

Delegation supports least privilege by allowing users to perform only the tasks they need.

---

## Organizing Computers

By default, most domain-joined machines are placed in the **Computers** container.

A better design is to organize machines by purpose.

Common device categories include:

| Category | Description |
|---|---|
| Workstations | Daily-use employee desktops and laptops |
| Servers | Systems providing services such as databases, file shares, or applications |
| Domain Controllers | Critical systems that host and manage Active Directory |

Workstations should not normally be used by privileged users because they are more exposed to phishing, web browsing, and user-driven compromise.

Domain Controllers require the strongest protections because they store sensitive authentication material and control the domain.

---

## Group Policy Objects

Group Policy Objects, or GPOs, are collections of settings that can be applied to users and computers.

GPOs are commonly linked to OUs.

They can configure:

- Password policies
- Desktop settings
- Firewall settings
- Software restrictions
- Security baselines
- Login scripts
- Browser settings
- Audit policies

There are two broad policy categories:

| Policy Type | Applies To |
|---|---|
| User Configuration | User accounts |
| Computer Configuration | Machines |

A common workflow is:

1. Create OUs based on departments or device roles.
2. Create GPOs for each policy requirement.
3. Link GPOs to the appropriate OUs.
4. Allow policies to apply automatically to objects inside those OUs.

---

## Trees

A tree is a collection of domains that share the same namespace.

Example:

```text
thm.local
uk.thm.local
us.thm.local
```

Each domain has its own users, computers, groups, and policies.

Domain Admins control their own domain, not necessarily every domain in the tree.

---

## Forests

A forest is a collection of one or more trees.

Forests allow organizations with different namespaces to exist within the same larger Active Directory structure.

Example:

```text
thm.local
mht.local
```

A forest is the highest-level security boundary in Active Directory.

Important forest-wide groups include:

| Group | Scope |
|---|---|
| Domain Admins | Administrative control over one domain |
| Enterprise Admins | Administrative control across the forest |

---

## Trust Relationships

Trust relationships allow users in one domain to access resources in another domain.

A trust does not automatically grant access. It only makes cross-domain authentication possible. Users still need explicit authorization to access resources.

### One-Way Trust

In a one-way trust, one domain trusts another domain.

Example:

```text
Domain AAA trusts Domain BBB
```

This means users from BBB can potentially access resources in AAA, if permissions allow it.

The trust direction is opposite the access direction.

### Two-Way Trust

In a two-way trust, both domains trust each other.

Users from either domain can potentially access resources in the other domain, if permissions allow it.

Two-way trusts are the default for domains within the same tree or forest.

---

## Key Takeaways

- Active Directory is a centralized identity and policy management system.
- Users and machines are security principals.
- Security groups grant access to resources.
- OUs organize objects and apply policies.
- GPOs enforce configuration and security settings.
- Delegation allows limited administration without giving Domain Admin rights.
- Trees group related domains under the same namespace.
- Forests group multiple trees.
- Trusts allow cross-domain authentication but do not automatically grant access.