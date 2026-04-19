# Design for governance — Q&A

Supplement to **main notes:** [design-governance.md](./design-governance.md).

This file merges overlapping questions into **topic sections**. Read **Quick summary** first, then drill into any heading.

---

## Table of contents

1. [Quick summary](#quick-summary)
2. [Study sheet: RBAC vs Policy](#study-sheet-rbac-vs-policy-and-overlapping-scopes)
3. [Azure hierarchy & resource groups](#azure-hierarchy--resource-groups)
4. [Networking: VNets, subscriptions, access](#networking-vnets-subscriptions-access)
5. [Azure Policy: scope, inheritance, portal](#azure-policy-scope-inheritance-portal)
6. [Azure RBAC: scopes, IAM, role assignment wizard](#azure-rbac-scopes-iam-role-assignment-wizard)
7. [When Policy and RBAC overlap](#when-policy-and-rbac-overlap)

---

## Quick summary

- **Order of containment:** `Tenant → Management groups (tree only) → Subscriptions → Resource groups → Resources`. **Subscriptions** and **resource groups** do **not** nest inside themselves.
- **Azure Policy** = rules on **resource configuration** (compliance). Assignments at **MG / subscription / RG / resource** are **cumulative**; **Deny** **blocks** that operation; a child scope **cannot** undo a parent **Deny** without **exemption** / **exclusions** ([Policy scope](https://learn.microsoft.com/en-us/azure/governance/policy/concepts/scope), [exemptions](https://learn.microsoft.com/en-us/azure/governance/policy/concepts/exemption-structure)).
- **Azure RBAC** = **who** may run **Azure management** actions. **Allow** roles **add** (**union**). A **weaker** role at a **child** scope does **not** remove a **stronger** allow from a **parent** scope. **Deny assignments** (RBAC) **do** block specific actions.
- **Portal:** **Access control (IAM)** = RBAC on that scope. **Policy** service = assignments, compliance, initiatives—not the same blade.
- **Resource group** = **logical** management boundary (lifecycle, RBAC, Policy, locks)—**not** a network perimeter; **not** a “tier” unless **you** design it that way.
- **VNet** = **one** subscription; connect across subs with **peering** / VPN / ExpressRoute. **NSG** applies to **subnet/NIC**; RG only **stores** the NSG resource.

---

## Study sheet: RBAC vs Policy and overlapping scopes

*Corrects common wrong answers (e.g. “Reader at RG overrides Contributor at subscription”). [Azure RBAC overview](https://learn.microsoft.com/en-us/azure/role-based-access-control/overview).*

### Side-by-side

| | **Azure RBAC** | **Azure Policy** |
|---|----------------|------------------|
| **Answers** | **Who** may act on the **control plane** (CRUD, etc.). | Whether a **resource’s configuration** is **allowed** / **compliant**. |
| **Mechanism** | Role assignments at **MG / sub / RG / resource**. | Policy & initiative assignments; **effects** (Audit, Deny, …). |
| **Multiple assignments** | **Allow** permissions **union** (add). | All applicable policies **evaluate**; **Deny** blocks that request. |
| **“Deny”** | **[Deny assignments](https://learn.microsoft.com/en-us/azure/role-based-access-control/deny-assignments)** override allows for those actions. | **Deny** effect blocks non-compliant **create/update**. |

**Analogy:** Policy = **building code** for the result. RBAC = **keys** for **who** may work on the site.

### RBAC: default and “reject”

- **No** allow until a **role assignment** grants it (**implicit deny** for allows).
- **Deny assignments** explicitly **reject** actions even if a role would allow them.

### Policy vs RBAC on the same action

If RBAC **allows** the API call but the **outcome** would **break** policy (**Deny** effect), the operation is **blocked**.

### Overlapping **allow** roles (union — not “narrower wins”)

| Scenario | Result |
|----------|--------|
| **Contributor** at subscription + **Reader** at one RG | Still **Contributor** in that RG for what Contributor allows. |
| **Reader** at subscription + **Contributor** at one RG | **Contributor** in that RG; **Reader** elsewhere in the sub. |
| **Contributor** at subscription only | **Contributor** on all RGs in that sub (unless **Deny assignment**). |

Wrong: “**Reader** at RG **overrides** **Contributor** at subscription” for normal **allow** roles.

### Upper scope “constraints”

A **less** powerful **allow** at a **child** scope does **not** cancel a **broader** **allow**. To **restrict**, use **least privilege** (don’t grant wide roles), **Deny assignments**, or **ABAC** conditions—not **Reader** at RG hoping it removes **Contributor** from above.

### Inheritance wording

- **Policy:** “Inherits” means **every assignment whose scope includes the resource is still in effect**—stacked evaluation.
- **RBAC:** Roles at a **parent** scope **grant** permissions on **child** resources (permissions flow **down** the scope).

---

## Azure hierarchy & resource groups

### What a resource group is for

Without RGs, a subscription is one undifferentiated pile of resources. RGs help **lifecycle** (delete/deploy together), **RBAC** and **Policy** at a **team/app** slice, **organization**, and **metadata** region (control plane; outage there can block **updates** to resources in that RG).

**RG is not:** a network boundary, a billing boundary (that’s mainly **subscription**), or a nested folder (**RGs cannot nest**).

### “Tier” (web / app) vs resource group

A RG is **not** automatically a tier. You **may** group by tier **by choice**; many teams group **by app** or **environment** instead.

### Is an RG only “logical”?

**Yes**—logical container for management; resources inside can still sit in **different regions** than the RG metadata region.

### NSGs and resource groups

An **NSG** is a normal **resource** stored **in** an RG. It **filters** traffic on **subnets** / **NICs**, not on “the RG object.”

### Network “policies”

- **Azure Policy** (including network-related definitions) can be assigned at **RG** (or sub/MG).
- **NSG** = resource you deploy; not “policy on the RG” in the Policy sense.

### Quick hierarchy

`Tenant → Management groups (nested) → Subscriptions → Resource groups → Resources`

---

## Networking: VNets, subscriptions, access

### Why a VNet does not “span” two subscriptions

A **VNet** is **one ARM resource** in **one** subscription (and region). There is no single VNet whose ownership spans two `subscriptionId`s. **Traffic** can cross subscriptions via **peering**, **VPN**, **ExpressRoute**, **vWAN** ([peering overview](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-network-peering-overview)).

### Two people, one VNet (Tom / John)

The VNet lives in **one** subscription. The other person needs **RBAC** on that scope to manage it; that does **not** put the VNet “in both” subscriptions—only **access** into one sub.

### Can the other user create a VNet there?

**Yes**, if they have a role that allows it (e.g. **Network Contributor** / **Contributor**) on a scope **in that subscription**. **Identity + RBAC**, not “only the subscription owner’s user account.”

---

## Azure Policy: scope, inheritance, portal

### Cumulative assignments (same as “do top policies pass down?”)

Every **policy assignment** whose **scope** includes the resource **still applies**. They **add**; a **child** assignment does **not** replace a **parent**. If **any** applicable policy hits **Deny** for that request, that request is **blocked**. A lower scope **cannot** “allow away” a parent **Deny** without **exemption** / design ([exemptions](https://learn.microsoft.com/en-us/azure/governance/policy/concepts/exemption-structure)).

### One assignment = one primary scope

You assign at **MG** **or** **subscription** **or** **RG** (or resource). **Stacking** comes from **multiple** assignments over time—not one wizard that means “sub + RG” as two unrelated policies.

In **Assign policy**, picking **Subscription** then **Resource group** is **navigation** (RG lives under a sub). The path `subscription/rg-name` is **one** scope: that **RG**.

### Policy Overview vs where to assign

- **Overview** = **dashboard** (charts, filters). **Scope: X selected** only **filters** what you **see**.
- **Assignments → Assign** = actually **attach** a policy/initiative. **Definitions / Authoring** = **custom** definitions. **Compliance** / **Remediation** = drill down and fix.

### Subscription vs RG for Policy (same feature, different blast radius)

| Scope | Effect |
|-------|--------|
| **Subscription** | Rules for **all** RGs in that sub (unless **exclusions**). |
| **Resource group** | Rules **only** for resources **in that RG**. |

---

## Azure RBAC: scopes, IAM, role assignment wizard

### Subscription vs resource group (same roles, different reach)

| Scope | RBAC meaning |
|-------|----------------|
| **Subscription** | Role applies to **all** RGs and resources in that sub (for what the role allows). |
| **Resource group** | Role applies **only** to resources **in that RG**. |

### Portal: IAM vs Policy

- **Access control (IAM)** on sub/RG/resource = **RBAC** (role assignments, Deny assignments, custom roles).
- **Policy** service = **Policy** assignments and compliance—not IAM.

### “Add role assignment” — what you are doing

You create: **principal** (who) + **role** (Reader, …) + **scope** (the subscription/RG/resource you opened IAM from). **Reader** on a **subscription** = read across **everything** in that sub—not “Reader of nothing.”

### “Assign access to”

| Option | Meaning |
|--------|---------|
| **User, group, or service principal** | Directory **users**, **groups**, or **app** (**service principal**). One radio bucket—not three login methods for one person. |
| **Managed identity** | Identity for an **Azure resource** (VM, App Service, …) so the **service** can access other Azure resources without you storing app secrets in code. |

**Select members** can list **users**, **groups**, or **service principals**; each row has a **Type**. You can add **several** members of **mixed** types in one assignment.

### Does a role assignment create API keys?

**No.** It is **authorization** only. **Authentication** (password, MFA, app **client secret**, managed identity tokens) is **separate** and configured **outside** this wizard.

### Common misunderstandings

- **Lester** as **Type: User** means one **user** identity—he does **not** “switch” to service principal for the same person.
- **Service principal** is for **apps**, not an alternate login for a human.

---

## When Policy and RBAC overlap

### Policy “conflicts” (Deny vs Audit)

All policies that apply are evaluated. **Deny** wins for the operation it covers. Example: MG **Deny** region + RG **Audit** tags → wrong region still **blocked**.

### RBAC “conflicts” (allows + Deny assignments)

- **Two allows** at different scopes → **union** (see [study sheet](#study-sheet-rbac-vs-policy-and-overlapping-scopes)).
- **Contributor** at sub + **Deny assignment** on delete at RG → **cannot delete** where Deny applies.

### Reader at subscription + Contributor at RG (e.g. database)

**Contributor** on the **RG** that holds the DB gives **write** for what that role allows on that resource. **Reader** at subscription does **not** cancel **Contributor** at the RG.

### Policy + RBAC together

| | Policy | RBAC (allows) |
|---|--------|----------------|
| Multiple scopes | All apply; **Deny** blocks; child cannot cancel parent Deny without exemption design. | Allows **merge**; **Deny assignment** blocks listed actions. |

Official: [Policy scope](https://learn.microsoft.com/en-us/azure/governance/policy/concepts/scope), [RBAC overview](https://learn.microsoft.com/en-us/azure/role-based-access-control/overview).

---

## One-line pointers (exam)

- **Policy** = **compliance / resource state**; **RBAC** = **who** can act; use **both**.
- **Policy** assignments **stack**; **RBAC** allows **union**; **RBAC Deny** and **Policy Deny** are different mechanisms.
- **RG** under **one** subscription; **IAM** = RBAC; **Policy** blade = Policy.
- **VNet** = one **subscription**; **NSG** on **subnet/NIC**.
