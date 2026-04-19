# Module: Design for governance

**Module on Learn:** [Design for governance](https://learn.microsoft.com/en-us/training/modules/design-governance/)

**Scenario (from Learn):** Tailwind Traders — a fictional retailer; you think like the CTO who needs **governance** so Azure stays manageable, compliant, and cost-visible.

---

## 1. Introduction

**Unit:** [Introduction](https://learn.microsoft.com/en-us/training/modules/design-governance/1-introduction)

### In plain terms

- **Governance** = defining **rules and policies**, then making sure they are **actually enforced** (not just documented).
- **Why it matters in Azure:** Without it, many teams and subscriptions can produce sprawl, inconsistent security, and unclear spend.

### When governance matters most

- Several engineering teams on Azure
- Many **subscriptions** to coordinate
- **Regulatory** or legal requirements
- Organization-wide **standards** (for example encryption, naming, regions)

### Learning objectives (this module)

You learn how to design for:

- Governance (overall)
- **Management groups**
- **Subscriptions**
- **Resource groups**
- **Azure Policy**
- **Resource tags**
- **Azure landing zones**

### Your notes (fill as you go)

- **What “governance” means to me in one sentence:**
- **My org’s top risk if we skipped governance:**

---

## 2. Design for governance

**Unit:** [Design for governance](https://learn.microsoft.com/en-us/training/modules/design-governance/2-design-for-governance)

### In plain terms

Per [Microsoft Learn — Design for governance](https://learn.microsoft.com/en-us/training/modules/design-governance/2-design-for-governance), **governance** is the set of **mechanisms and processes** that keep **control** over applications and resources in Azure. It is not only documentation: it includes **figuring out requirements**, **planning initiatives**, and **setting priorities** so the environment matches what the business and compliance need.

You apply governance **through a hierarchy**: structure the org’s Azure estate so you can attach rules (and visibility) **at the right scope**. This module’s concrete **governance tools** are **Azure Policy** and **resource tags** (covered in later units).

### The Azure hierarchy (four levels)

Learn presents a typical structure (often shown top-down):

| Level | Role in one line |
|--------|------------------|
| **Management groups** | Group subscriptions so you can manage **access**, **policy**, and **compliance** across many subscriptions at once. |
| **Subscriptions** | **Logical containers** for workloads: unit of **management and scale**, and a **billing boundary**. |
| **Resource groups** | **Logical containers** where resources are **deployed** and **managed** together (lifecycle and organization). |
| **Resources** | **Instances** of Azure services (for example VMs, storage accounts, SQL databases). |

### Tenant root group (easy to overlook)

- The **tenant root group** sits at the top and **contains** all management groups and subscriptions.
- Use it when you need **directory-wide** (tenant-level) effect: **Azure Policy** and **Azure role assignments** applied at the broadest scope.

### Design angle (why the hierarchy matters)

- **Without** a clear hierarchy, you either **over-restrict** (one-size-fits-all) or **under-govern** (exceptions everywhere).
- **With** levels, you can push **broad rules** high (for example compliance baselines) and **narrower rules** lower (for example team-specific tags or policies).

### Your notes (optional)

- **Where our org would attach “must apply everywhere” vs “team-specific” rules:**

### Remember for the exam

- Governance = **control** via **process + mechanisms**; needs **requirements**, **planning**, **priorities**.
- Know the **four hierarchy levels** and what each is **for** (especially: subscriptions = **billing** + scale/management; resource groups = **deployment/management** grouping).
- **Tenant root group** = top container; supports **global** **Policy** and **RBAC** at directory scope.
- This module’s called-out **governance strategies**: **Azure Policy** and **resource tags**.

---

## 3. Design for management groups

**Unit:** [Design for management groups](https://learn.microsoft.com/en-us/training/modules/design-governance/3-design-for-management-groups)

Official reference: [Management groups in Azure](https://learn.microsoft.com/en-us/azure/governance/management-groups/overview).

### In plain terms

**Management groups** are **containers** that let you govern **access**, **Azure Policy**, and **compliance** across **multiple subscriptions** at once—without repeating the same assignments on every subscription.

Typical reasons to use them (from the unit):

- **Policy at scale:** For example, **restrict which regions** VMs can be created in, **across** subscriptions.
- **Access at scale:** **One role assignment** at a management group can **inherit** down so users reach **multiple** subscriptions (where inheritance applies).
- **Visibility:** **Monitor and audit** **role** and **policy** assignments **across** subscriptions.

### Things to know (characteristics)

| Topic | Detail |
|--------|--------|
| **Policy** | Management groups **aggregate** **policy** and **initiative** assignments (via **Azure Policy**). |
| **Depth limit** | A management group tree supports up to **six levels of depth**. That **does not** count the **tenant root** level or the **subscription** level. |
| **RBAC on management groups** | **Azure RBAC** for management group **operations** is **not enabled by default** (you can turn it on where needed). |
| **New subscriptions** | **By default**, new subscriptions are placed under the **root management group**. |

### Things to consider (design patterns)

The unit uses **Tailwind Traders** (Sales with **West** / **East**, **Corporate** with HR/Legal, **IT** with R&D and production, etc.) to illustrate **how** you might shape the tree—not a single “correct” answer, but **design dimensions**:

- **Governance-first:** Put **Azure Policy** at the **management group** level for workloads that need the **same** security, compliance, connectivity, and feature posture.
- **Depth:** Keep the tree **reasonably flat**. The module suggests planning for **about three or four** management group levels for Tailwind. **Too flat** → less flexibility for large orgs; **too deep** → harder to operate and reason about.
- **Top-level management group:** Often useful for **organization-wide** “**platform**” **policies** and **Azure role** assignments that should apply **everywhere** (under the tenant root).
- **By org / department:** Align groups to **business structure** (for example **Sales**, **Corporate**, **IT**).
- **By geography:** Separate groups when **regional compliance** or rules differ (for example **West** vs **East** sales regions).
- **Production:** A **production** management group can carry **product-wide** or **corporate product** policies.
- **Sandbox:** A **sandbox** group lets people **experiment**, **isolated** from dev / test / production and from rules that apply to “official” workloads.
- **Sensitive workloads:** **Isolate** high-sensitivity data (for example **Corporate** / HR / Legal) in a **separate** management group with **standard plus enhanced** compliance policies.

### Hierarchy sketch (conceptual)

Child management groups and subscriptions hang under parents; **policy and RBAC** can flow down the tree (subject to how you assign scopes—exam questions often test **scope** and **inheritance**).

```mermaid
flowchart TD
  TR[Tenant root group]
  TL[Top-level org / platform MG]
  S[Sandbox MG]
  D1[Department or region MGs...]
  SUB[Subscriptions under a MG]

  TR --> TL
  TR --> S
  TR --> D1
  D1 --> SUB
```

### Your notes (optional)

- **Our org:** would we align MGs by **business unit**, **environment**, **region**, or a **mix**—and why?

### Remember for the exam

- Management groups = **multi-subscription** governance for **Policy**, **compliance**, and **access** (inherited role assignments where applicable).
- **Six** depth levels max in the MG tree (**excluding** tenant root and subscription).
- **RBAC** on management group **operations**: **off by default**.
- **New subscriptions** → **root management group** by default.
- Design trade-off: **flat vs deep** hierarchy; use cases for **platform** MG, **sandbox**, **geo**, **sensitive** isolation.

---

## 4. Design for subscriptions

**Unit:** [Design for subscriptions](https://learn.microsoft.com/en-us/training/modules/design-governance/4-design-for-subscriptions)

Related docs: [Azure subscription and service limits](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/azure-subscription-service-limits), [subscription offer types](https://azure.microsoft.com/support/legal/offer-details/).

### In plain terms

An Azure **subscription** is a **logical container**: it is a **unit of management and scale**, a **billing boundary**, and where **limits and quotas** apply. You need at least one subscription to create and pay for Azure services.

Organizations use subscriptions to **group cost and resources** (for example by team, environment, or workload) and to **apply governance** at a scope below the tenant but above resource groups.

### Things to know (characteristics)

| Topic | Detail |
|--------|--------|
| **Billing** | Subscriptions can separate **billing environments** (for example **development**, **test**, **production**). |
| **Compliance** | **Policies** scoped to a subscription can target **different compliance** needs than other subscriptions. |
| **Scale** | You can place **specialized workloads** in their **own** subscription to grow past **limits** of an existing subscription. |
| **Cost visibility** | Subscriptions help **manage and track** spend in line with **organizational** structure. |
| **Offer types** | There are **several subscription types** (for example Enterprise Agreement, Pay-As-You-Go)—see [offer details](https://azure.microsoft.com/support/legal/offer-details/). |

### Things to consider (design decisions)

- **Subscriptions as a “democratized” management unit:** Shape them around **business needs and priorities**, not a single template.
- **Under management groups:** Put subscriptions that need the **same** **policies** and **Azure role assignments** under the **same** management group so they **inherit** those settings (example from the module: **West** and **East** sales subscriptions inheriting from the **Sales** management group).
- **Shared services subscription:** Often use a **dedicated** subscription for **shared network** platforms so spend is **grouped** and workloads stay **isolated** from app teams—examples called out: **ExpressRoute**, **Virtual WAN**.
- **Scale limits:** Treat a subscription as a **scale unit**. Large or specialized footprints (**HPC**, **IoT**, **SAP**) often deserve **separate** subscriptions so you do not hit **service limits** (the module cites an example such as a limit of **50 Azure Data Factory integrations**—confirm current numbers in [subscription limits](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/azure-subscription-service-limits)).
- **Administration:** Subscriptions are a **management boundary** and **separation of duties**—decide if each subscription needs **distinct** administrators (example: **Corporate** might use **one** subscription for both **HR** and **Legal**).
- **Where to assign Azure Policy:** Both **management groups** and **subscriptions** are valid **assignment scopes**. For strict isolation (example: **PCI** workloads), a **subscription** alone can provide the boundary instead of creating **extra** management groups with only **one or two** subscriptions—keeps the **MG** structure from sprawl.
- **Networking:** A **virtual network** does **not** span subscriptions. Cross-subscription connectivity uses patterns such as **virtual network peering** or **VPNs**—when designing subscriptions, consider **which workloads must talk to which** other workloads.
- **Ownership:** Make **subscription owners** clear on **roles and responsibilities**; the unit suggests **access reviews** (for example **quarterly** or **biannual**) using **Microsoft Entra Privileged Identity Management** so privilege does not grow unchecked.

### Subscription vs management group (quick contrast)

| Question | Lean toward |
|----------|-------------|
| Same policies/RBAC for many subs | **Management group** inheritance |
| Hard isolation, billing, or quota (PCI, huge workload) | Often a **dedicated subscription** |
| Too many tiny MGs each with 1–2 subs | Rebalance: use **subscriptions** for some isolation instead |

### Your notes (optional)

- **Environments we’d put in separate subscriptions vs one subscription + tags/RGs:**

### Remember for the exam

- Subscription = **management + scale** + **billing**; **quotas/limits** apply at subscription scope.
- **VNets are per subscription** (not shared across subscriptions); link with **peering** / **VPN** as needed.
- **Shared services** subscription for common networking (**ExpressRoute**, **Virtual WAN**) and consolidated billing.
- **Policy** can be assigned at **MG** or **subscription**; **PCI-style** isolation can be **subscription**-scoped to avoid **management group** sprawl.
- **Governance hygiene:** subscription **owners** + **PIM** **access reviews**.

The module closes with: **one size does not fit all**—what works for one business unit may not suit another ([Design for subscriptions](https://learn.microsoft.com/en-us/training/modules/design-governance/4-design-for-subscriptions)).

---

## 5. Design for resource groups

**Unit:** [Design for resource groups](https://learn.microsoft.com/en-us/training/modules/design-governance/5-design-for-resource-groups)

Background: [Azure Resource Manager and resource groups](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/overview).

### In plain terms

A **resource group** is a **logical container** for Azure resources you deploy and manage together (web apps, databases, storage, and so on). It is a **management and governance** boundary **below** the subscription—not a second network or billing boundary by itself.

You use resource groups to:

- Group resources by **similar usage, type, or location**
- Align resources that share a **life cycle** (create, update, or **delete** as a unit)
- Apply **RBAC** (who administers a **set** of resources)
- Use **resource locks** to block accidental delete or change

### Things to know (characteristics)

| Topic | Detail |
|--------|--------|
| **Resource group “location”** | Each resource group has a **region**; that region stores **metadata** for the resource group. |
| **Metadata outage** | If that region is **temporarily unavailable**, you **cannot update** resources **in** that resource group (metadata unavailable). Resources running in **other** regions may **keep working**, but **management** updates can be blocked. |
| **Resource regions** | Resources **inside** a resource group **can** live in **different** regions than each other and than the RG’s metadata region. |
| **Cross–resource group** | A resource can **depend on or connect to** resources in **another** resource group (example from the module: web app in one RG, database in another). |
| **Move** | Resources can be **moved** between resource groups (and subscriptions) with [documented exceptions](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/move-resource-group-and-subscription). |
| **Membership** | You can **add** or **remove** a resource from a resource group **at any time** (subject to support for the move). |
| **Nesting** | Resource groups **cannot** be nested. |
| **Cardinality** | Each resource belongs to **exactly one** resource group. |
| **Rename** | Resource groups **cannot** be **renamed** (plan the name up front). |

### Things to consider (design patterns)

The module uses **Tailwind Traders** with **App1** and **App2** (each with web tier, SQL, VMs, storage) to illustrate choices—not one mandatory layout:

- **By resource type:** Useful for **shared** or **on-demand** services **not** tied to a single app (example pattern: **SQL-RG** for databases, **WEB-RG** for web workloads).
- **By application:** When everything shares the **same policies** and **life cycle**, put **one app** in its **own** resource group (App1 RG vs App2 RG); also common for **test** or **prototype** environments.
- **Other axes:** **Department**, **region**, **billing / cost center**—less universal but valid when the business needs it.
- **Combination:** Often **mix** strategies (for example app-based groups plus a shared type-based group).
- **Life cycle:** If you must **deploy, update, or tear down** components **together**, favor putting them in the **same** resource group.
- **Administration overhead:** More resource groups = more to operate; match **centralized** vs **decentralized** admin models.
- **Access control:** At the resource group scope you can assign **Azure Policy**, **Azure RBAC**, and **[resource locks](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/lock-resources)** for critical workloads.
- **Compliance:** Ask whether **resource group metadata** must live in a **specific region** for data-residency or policy reasons.

### Your notes (optional)

- **Our default:** group by **app**, by **environment**, or by **shared platform**—and why:

### Remember for the exam

- Resource group = **logical** grouping for **management**; **not** “everything must be in one region.”
- RG **region** = **metadata** storage; outage there can **block updates** to resources in that RG even if workloads run elsewhere.
- **Cannot nest** RGs; **cannot rename**; each resource **exactly one** RG.
- Design patterns: **type** vs **app** vs **hybrid**; align RG boundaries to **life cycle** and **RBAC/Policy/locks**.

---

## 6. Design for resource tags

**Unit:** [Design for resource tags](https://learn.microsoft.com/en-us/training/modules/design-governance/6-design-for-resource-tags)

- **Summary:**
- **Tagging strategy (cost center, env, owner, etc.):**
- **Remember for the exam:**

---

## 7. Design for Azure Policy

**Unit:** [Design for Azure Policy](https://learn.microsoft.com/en-us/training/modules/design-governance/7-design-for-azure-policy)

- **Summary:**
- **Policy vs RBAC (what each controls):**
- **Remember for the exam:**

---

## 8. Design for role-based access control (RBAC)

**Unit:** [Design for RBAC](https://learn.microsoft.com/en-us/training/modules/design-governance/8-design-for-role-based-access-control)

- **Summary:**
- **Scope:** management group / subscription / RG / resource
- **Remember for the exam:**

---

## 9. Design for Azure landing zones

**Unit:** [Design for Azure landing zones](https://learn.microsoft.com/en-us/training/modules/design-governance/9-design-for-landing-zones)

- **Summary:**
- **What a landing zone gives you (baseline, networking, identity hooks):**
- **Remember for the exam:**

---

## 10. Module assessment

**Unit:** [Knowledge check](https://learn.microsoft.com/en-us/training/modules/design-governance/10-knowledge-check)

- **Questions I missed / why:**

---

## 11. Summary and resources

**Unit:** [Summary and resources](https://learn.microsoft.com/en-us/training/modules/design-governance/11-summary-resources)

- **Links or docs I want to revisit:**
