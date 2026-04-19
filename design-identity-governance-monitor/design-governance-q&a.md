# Design for governance — Q&A

Follow-up questions and answers for this module. **Main notes:** [design-governance.md](./design-governance.md).

_Add entries below as you ask clarifications, drill questions, or edge-case questions._

---

## Questions

### Q: Do “top” policies pass down to children? If the top allows, does the bottom allow? If top and bottom conflict, what wins? Can the bottom allow something the top rejected? Does this apply to management groups and subscriptions?

**A:** This is about **Azure Policy** (not the same rules as **RBAC**—see the next question for a side-by-side).

**Inheritance (how it really works)**  
A resource is evaluated against **every policy assignment whose scope includes that resource**—from the hierarchy above it (management groups → subscription → resource group → resource, depending on where assignments exist). So higher scopes **do** affect children in the sense that **those assignments still apply** to resources underneath. It is **not** a simple “one inheritance switch” in the UI; it is **cumulative evaluation** of all applicable assignments. Official concept: [Understand scope in Azure Policy](https://learn.microsoft.com/en-us/azure/governance/policy/concepts/scope).

**“Top allows, so bottom allows”?**  
Azure Policy effects are things like **audit**, **deny**, **deployIfNotExists**, **modify**, **append**, etc.—not a generic “allow everything” toggle. There is **no** child assignment that **overrides** a parent **deny** by “allowing harder.” If **any** applicable policy results in **deny** for that request, the create/update is **blocked** (most restrictive / explicit deny behavior).

**Top and bottom “conflict”**  
All relevant policies run. If one assignment would **deny** and another only **audits** or does nothing conflicting, **deny still wins** for enforcement. You do not pick “the lower scope wins.”

**Top rejects (denies); can bottom still allow?**  
**No.** A subscription or resource group **cannot** undo a management group **deny** by assigning a more permissive policy. To carve out an exception you use an **exemption** (or adjust **assignment** design: **exclusions** / `notScopes` where appropriate), not a competing “allow” policy. See [Azure Policy exemption structure](https://learn.microsoft.com/en-us/azure/governance/policy/concepts/exemption-structure).

**Management groups and subscriptions?**  
**Azure Policy:** Yes. Assignments can be attached at **management group**, **subscription**, or **resource group** scope. Resources in a subscription inherit **all** assignments from **parent management groups** **plus** any on the **subscription** (and **resource group** if applicable). They **add together**; the child does **not** replace the parent.

**RBAC (different model):** **Role assignments** inherit down the hierarchy (for example a role on a management group applies to child subscriptions). **Permissions** are effectively **additive** for allowed actions, but **Azure RBAC deny assignments** and classic **Deny** behavior are evaluated with **deny taking precedence** over allows. So do **not** mix the “parent deny / child allow” mental model from Policy 1:1 with RBAC—same *word* “deny,” different mechanism. For exam prep, keep **Policy = compliance/enforcement effects** and **RBAC = who can do what** separate.

### Q: RBAC and Azure Policy—why is “inheritance” different?

**A:** They solve **different problems**, so “inheritance” means something different in each.

| | **Azure Policy** | **Azure RBAC (roles)** |
|---|-------------------|-------------------------|
| **Job** | **What** may exist or **how** it must be configured (tags, SKUs, regions, encryption, etc.). | **Who** may **call Azure** to create, read, update, delete resources (control plane). |
| **“Inheritance” mental model** | Every assignment whose **scope** contains your resource **still applies**. All of them are **checked** together (stacked rules). Parent scope **does not** get replaced by child scope. | A role assigned at a **parent** scope (management group, subscription, RG) **grants the same permissions** on **child** resources under that scope (access **flows down**). |
| **Extra at lower scope** | More assignments **add more rules** (usually **stricter** overall if anything denies). | More role assignments **add more permissions** for those principals (wider access), unless a **deny** blocks. |
| **“Deny”** | Policy **Deny** effect = block non-compliant **resource changes**. | **Deny assignment** (RBAC) = block that identity from an action even if a role would allow it. |

**One-line analogy**

- **Policy** = building code for the **house** (must meet standards no matter which contractor works on it).
- **RBAC** = **keys** for **who** is allowed on the construction site.

Same hierarchy (management group → subscription → resource group → resource), **two layers** of control: **keys** (RBAC) and **rules** (Policy).

**Why the earlier answer sounded confusing**  
For **Policy**, “pass down” really means **“all scopes that include this resource still count.”** For **RBAC**, “pass down” means **“access granted at parent applies to objects below.”** Same words, different mechanics—so yes, **inheritance is different** between the two.

### Q: Why doesn’t a virtual network “span” across subscriptions?

**A:** In Azure, a **virtual network (VNet)** is **one resource** that belongs to **exactly one subscription** (and **one region**). The platform **does not** offer a single VNet object whose **ownership**, **quotas**, **RBAC**, and **billing** are split across two subscription IDs.

**What “doesn’t span” means here**

- **Not:** “IP packets cannot cross subscription boundaries.” They can, if you **connect** networks.
- **Yes:** “There is **no one VNet resource** that lives in Subscription A **and** Subscription B at the same time.” Each subscription has **its own** VNet resources with **their own** address spaces and subnets.

**Why that matters for design (from the governance module)**

- **Isolation:** Subscriptions are often used as **billing** and **administrative** boundaries. Keeping the VNet **inside** one subscription keeps **network ownership** aligned with that boundary.
- **Connecting two subs:** You **link** separate VNets—**virtual network peering** (including [cross-subscription peering](https://learn.microsoft.com/en-us/azure/virtual-network/create-peering-different-subscriptions)), **VPN**, **ExpressRoute**, **vWAN**, etc. Traffic can flow, but each side remains **its own** VNet **resource**.

**Short analogy**

- A VNet is like **one deed** to a plot of land: it sits in **one** “account” (subscription). Two companies (subscriptions) each have **their own** plot; if they need a **gate** between plots, you **add a connection** (peering/VPN)—you don’t merge the deeds into one plot across both accounts.

See also: [Virtual network peering](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-network-peering-overview).

### Q: If there are two subscriptions (“Tom” and “John”), can both users manage one VNet?

**A:** The VNet is still **one Azure resource** in **one** subscription—it is not “half Tom’s and half John’s.”

**Both can manage the same VNet** if you want that:

- Put the VNet in **one** subscription (say **Tom’s**).
- Grant **John** access with **Azure RBAC** on that subscription, the **resource group** that holds the VNet, or the **VNet** itself (for example **Contributor** or **Network Contributor**, depending on least privilege). John might be a **guest user** in Tom’s tenant or a user in the same directory—either way, access is **roles**, not “splitting” the subscription.

**What does *not* work:** There is no mode where **one** VNet object is **owned by two subscriptions at once**. If each person must keep **full** isolation and **separate** billing/ownership with **no** shared RBAC into the other sub, then you use **two VNets** (one per subscription) and **peer** them so workloads can still talk.

**Exam-style takeaway:** Shared **management** of one network = **RBAC** on the scope where that **single** VNet lives. Shared **IP space as one resource across two subscription IDs** = not supported; use **peering** or **centralize** the VNet in one sub and delegate access.

### Q: If John can manage Tom’s VNet, isn’t the VNet “in both” subscriptions?

**A:** **No**—for Azure, the VNet still exists in **one** subscription only. **John’s subscription** does **not** contain a copy of that VNet or a second “half” of it.

**Two different ideas:**

| Idea | What it means here |
|------|---------------------|
| **Where the resource lives** (residency / billing / ARM scope) | The VNet’s **ARM ID** includes **one** `subscriptionId`. That is **Tom’s** subscription (in this example). **John’s** subscription ID does **not** appear in the resource’s identity. |
| **Who may change it** (RBAC) | **John** gets **permission** to call Azure APIs **against** that resource. He is **acting on** Tom’s subscription’s resource—like being given a key to **Tom’s** building, not moving the building onto **John’s** land. |

So: **John can manage it** = **access** into **Tom’s** scope. It does **not** mean the VNet is **registered** in **two** subscriptions. Billing and quota for that VNet still attach to **Tom’s** subscription (unless you move the resource, which is a separate operation).

**Why this matters for the exam:** “Does not span subscriptions” refers to **resource placement** and **one** subscription per resource, not “only one human can touch it.”

### Q: So John cannot create a VNet in Tom’s subscription?

**A:** **He can**, if **Tom** (or an owner) grants him **Azure RBAC** on a scope inside **Tom’s** subscription where VNets are allowed—for example **Contributor** or **Network Contributor** on a **resource group** or the **subscription** (use least privilege in real life).

- **If John has no role** on Tom’s subscription/RG → he **cannot** create anything there (Azure will deny the operation).
- **If John has a role** that includes `Microsoft.Network/virtualNetworks/write` (directly or via a role like **Network Contributor** / **Contributor**) on the right scope → he **can** create a VNet **in Tom’s subscription**; that new VNet is still **only** in **Tom’s** subscription ID.

**Same rule for “who pays”:** Usage for that VNet still bills to **Tom’s** subscription (unless you use cross-billing patterns, which are a separate topic).

So: **subscription boundary** = where the **resource** is registered, not **which human** is allowed—**identity + RBAC** decides whether John may create resources **inside** Tom’s subscription.

### Q: Is a resource group like a “tier” (web tier, app tier)? Can I create network policies on it?

**A: Resource group vs tier**

A **resource group** is **not** automatically a “tier.” It is a **logical container** for Azure resources (any mix of types). You **may** choose to put **all web-tier** resources in one RG and **data-tier** resources in another—that is a **design choice** (similar to “group by type” in the module). Many teams instead group **by app** or **by environment** (dev/test/prod), which might include web + app + data **together** in one RG.

So: **tier** is an **architecture** idea; **resource group** is a **management** boundary (lifecycle, RBAC, Policy, locks). They only line up if **you** design them to.

**A: “Network policies”**

Depends what you mean:

| If you mean… | At resource group scope |
|--------------|-------------------------|
| **Azure Policy** (governance: allowed regions, required tags, “NSG must be attached to subnet,” etc.) | **Yes.** You can **assign** Azure Policy (and initiatives) at **management group**, **subscription**, or **resource group** scope. That includes many **network-related** policy definitions. |
| **Network Security Groups (NSGs)** / **firewall rules** | Those are **Azure resources** (or rules on subnets/NICs). You **create** NSGs **inside** a resource group like any other resource; you don’t “create an NSG *on* the RG” as a special RG feature—the RG **holds** the NSG resource. |
| **Kubernetes NetworkPolicy** (AKS) | That is a **Kubernetes** concept inside a cluster—not the same as an Azure resource group. |

**Exam-style takeaway:** Resource group = **container + governance scope**; **tier** = optional **grouping pattern**. **Azure Policy** can enforce network-related rules at RG scope; **NSGs** are **resources** you deploy and associate with subnets or NICs.

### Q: So NSG is not “for” a resource group?

**A:** Correct in this sense: an **NSG does not attach to or protect “the resource group”** as an object. A **resource group** has no network traffic to filter.

What is true:

- An **NSG** is an Azure **resource**, so it **lives in** a **resource group** (for billing, RBAC, and organization—like a VM or a VNet).
- **Where it works:** You **associate** an NSG with a **subnet** and/or **network interface (NIC)**. That is where **allow/deny rules** apply to **packets**.

So: **RG** = where the NSG **resource is stored**. **Subnet/NIC** = where the NSG **does** its job. Not “NSG for RG,” but “NSG **in** an RG, **applied to** subnet/NIC.”

### Q: Does a resource group have network restrictions? Can I put a resource group inside another? Can a subscription contain a subscription? Is nesting only for management groups? Is an RG “just logical”?

**A: Network and resource groups**

A **resource group does not define a network boundary.** It does not block or allow traffic. **Network restrictions** come from **virtual networks**, **subnets**, **NSGs**, **firewalls** (Azure Firewall, NVAs), **private endpoints**, **route tables**, **service tags**, etc. You might **choose** to put all networking resources for an app in **one** RG for management—that is still **organizational**, not a built-in “RG firewall.”

**A: Nesting resource groups**

**No.** Resource groups **cannot** be nested. You cannot place one resource group **inside** another.

**A: Nesting subscriptions**

**No.** A **subscription** does **not** contain another **subscription**. Subscriptions sit **under** the **management group** hierarchy (and tenant); they are **siblings** in the tree under MGs, not parents of each other.

**A: Where nesting *does* exist**

- **Management groups** form a **tree** (parent/child management groups). **Subscriptions** attach to **one** management group (as a child of that branch). **Subscriptions are not nested inside each other.**
- **Resource groups:** flat under a subscription—**no** nesting.

**A: “Resource group is just logical?”**

**Yes**, in the sense the module uses: it is a **logical container** for **management**—deploy, organize, lifecycle, **RBAC**, **Policy**, **locks**. It is **not** a network perimeter, **not** a billing boundary (that is mainly the **subscription**), and **not** a parent for other resource groups.

**Quick hierarchy (exam-friendly)**

`Tenant → Management groups (tree) → Subscriptions → Resource groups → Resources`

- **Nested:** management groups (only among themselves).
- **Not nested:** resource groups; subscriptions inside subscriptions.

### Q: What is the point of having a resource group?

**A:** A subscription can hold **many** resources. Without resource groups you would have **no structured way** to group them for **day‑to‑day operations** and **governance** at a finer grain than the whole subscription.

**Practical reasons (why Azure has RGs):**

1. **Life cycle** — Delete, redeploy, or hand off **everything for one app** or **one environment** together instead of picking resources one by one.
2. **Access control (RBAC)** — Grant someone **Contributor** (or a custom role) on **one** RG instead of the entire subscription or each resource individually.
3. **Governance** — Assign **Azure Policy** and **resource locks** at **RG** scope so one app or one environment gets the right rules.
4. **Organization** — Reflect how **you** work: by **app**, by **tier**, by **env**, or **mixed**—so humans can find and own resources.
5. **Metadata** — The RG has its **own** region for **control-plane metadata** (with the availability implications from the module if that region is down).

**What an RG is *not*:** It is **not** what creates network isolation or billing separation—that is **VNet / NSG / firewall** and **subscription** (respectively).

**One line:** Resource groups exist so you can **manage, secure, and tear down related Azure resources as a unit** without treating the whole subscription as one blob.

### Q: Can I set Policy and RBAC at management group, subscription, and resource group? If MG and RG “conflict,” what happens? Same for RBAC—with examples.

**A: Where you can assign**

| Scope | Azure Policy assignment | Azure RBAC role assignment |
|--------|-------------------------|------------------------------|
| **Management group** | Yes | Yes (roles that support MG scope) |
| **Subscription** | Yes | Yes |
| **Resource group** | Yes | Yes |
| **Single resource** | Yes | Yes |

So **yes**: both systems support **multiple** scopes; **MG → subscription → RG → resource** is the usual chain.

---

**Azure Policy — how “conflicts” resolve**

**Rule of thumb:** Every assignment that **applies** to the resource is **evaluated**. Effects are **cumulative**; **Deny** **wins** for enforcement. A **lower** scope **cannot** cancel a **higher** scope **Deny** with a “permissive” policy.

**Example 1 — real conflict (deny vs audit)**

- **Management group:** Policy **Deny** — “VMs may only be created in `East US`.”
- **Resource group:** Policy **Audit** — “Require tag `CostCenter` on VMs.”

**Result:** Both apply. **Creating a VM in `West Europe`** in that RG is **blocked** (Deny). **Creating in `East US`** without the tag **may** be **allowed** for the create, but the **Audit** policy flags **non-compliance** for the tag until fixed (depending on policy timing). **Deny** still governs **region**.

**Example 2 — two “opposite” intents (deny at top)**

- **MG:** Policy **Deny** — “Block public blob access on storage accounts.”
- **RG:** Assign a policy that **does not** deny that (or only Audit).

**Result:** The **MG Deny** still **blocks** public blob configuration **unless** you use an **exemption** or **notScopes** on the MG assignment. The RG **cannot** “allow” what the MG **Deny** forbids.

---

**Azure RBAC — how “conflicts” resolve**

**Different from Policy:** **Allow** roles **add** together. Your **effective permissions** on a resource are generally the **union** of all **role assignments** that apply to you at **or above** that resource (same tenant, same rules). **Deny assignments** (Azure RBAC **Deny**) **override** **allows** for those actions.

**Example 1 — two allows (no deny)**

- **Management group:** **User** has **Reader** on the MG (inherits to all subs underneath).
- **Resource group `App1-RG`:** Same **User** has **Contributor** on **only** `App1-RG`.

**Result:** On a VM **inside** `App1-RG`, the user has **Contributor** (create/update/delete) **plus** inherited **Reader** elsewhere. On **that** VM, **Contributor** is what matters for writes. **No conflict**—**permissions combine** (more permissive **allow** for that scope).

**Example 2 — allow vs deny assignment**

- **Subscription:** **User** has **Contributor** on the subscription (wide access).
- **Resource group `Prod-RG`:** **Deny assignment** (RBAC) for that same user **blocking** `Microsoft.Compute/virtualMachines/delete` on `Prod-RG` (or on a scope that covers those VMs).

**Result:** **Deny wins** for **delete** on those VMs: **cannot** delete even though **Contributor** would normally allow it.

**Example 3 — narrow vs wide allow**

- **Subscription:** **User** = **Reader** on subscription.
- **RG:** **User** = **Contributor** on `App1-RG` only.

**Result:** In `App1-RG`, user can **change** resources (**Contributor**). In **other** RGs in the same sub, user is **Reader** only. **Not** a conflict—**different scopes**, **different** effective access.

---

**Exam-style takeaway**

- **Policy:** **Stack** rules; **Deny** is a hard stop; **child** cannot **override** parent **Deny** without **exemption** / assignment design.
- **RBAC:** **Allows** **merge** (union); **Deny assignments** **block** specific actions even if a role would allow them.

Official: [Azure Policy scope](https://learn.microsoft.com/en-us/azure/governance/policy/concepts/scope), [Azure RBAC overview](https://learn.microsoft.com/en-us/azure/role-based-access-control/overview).

### Q: Policy — does any Deny anywhere “win everything”? RBAC — what does “Reader high” and “add up” mean? If I am Reader on the subscription but Contributor on the RG with my database, can I write to the DB?

**A: Azure Policy (more precise)**

It is **not** “one Deny turns off the whole world.” For **each create or update request**, Azure evaluates **every policy assignment** that applies to that resource. If **any** of them triggers the **Deny** effect **for that specific request**, that **request is blocked** (for that action). Other policies can still **Audit** or **DeployIfNotExists** on other aspects.

So: **Deny wins for the operation it applies to**, not “ignore all levels.” **Level** matters for **which resources** are **in scope** of each assignment, not for “MG Deny vs RG Audit” in a vague way—**both** run; **Deny** still **blocks** what it blocks.

**A: RBAC — “add up” means union of allows**

**“Reader high”** was sloppy wording. Better: **Reader on a broad scope** (for example the **whole subscription**) plus **Contributor on a narrow scope** (one **resource group**).

Azure RBAC **allow** assignments **combine** for the same user (same **principal**): you get the **union** of permissions from **every role assignment** that applies to you **at or above** that resource. A **subscription-level Reader** does **not** “lock” you to read-only **inside** an RG where you also have **Contributor**.

**Your database example**

- **Subscription:** **Reader** → can **read** resources **everywhere** in that subscription (including all RGs).
- **Same user** on **Resource group** `Data-RG` (where the **database** lives): **Contributor** (or a DB-specific role that includes write).

**Result on the DB in `Data-RG`:** The **Contributor** (or equivalent) assignment on **`Data-RG`** gives you **write** actions the role allows on that database. The **subscription Reader** does **not** override that downward to “read only only.” You are **not** “subscription read only” for that DB—you have **both** assignments, and **Contributor** includes **write** for that scope.

**Where subscription “controls” RG:** Not by **downgrading** a stronger RG role. **Subscription** is just **another scope** where you can assign roles. If the **only** assignment were **Reader at subscription** and **nothing** at RG → read only in all RGs. If you **add** **Contributor at RG** → **write** appears for resources in that RG.

**Exception:** **Azure RBAC Deny assignments** (explicit denies) or **Azure ABAC** conditions can still **block** specific actions even when a role would allow them—different from “Reader at sub prevents Contributor at RG.”

**Short compare**

| | **Policy** | **RBAC (allows)** |
|---|------------|-------------------|
| Multiple scopes | **All** apply; **Deny** effect **blocks** that operation; child **cannot** cancel parent **Deny** without exemption design. | **All allow** assignments **merge**; **broader Reader** + **narrower Contributor** on same user → **Contributor** wins for **writes** on resources **in** that narrow scope (unless **Deny assignment** says otherwise). |

---

### Simple reference: subscription vs resource group (what you actually *do*)

Same **kinds** of Policy and RBAC exist at both levels—the difference is **how much** they apply to.

#### Azure Policy

| Where you assign it | What you are doing in plain English |
|---------------------|-------------------------------------|
| **Subscription** | “These rules apply to **everything** in this subscription (all resource groups and resources under it), unless the assignment **excludes** some scopes.” You pick **policy definitions** or **initiatives** (groups of policies) and attach them **here**. |
| **Resource group** | “These rules apply **only** to resources **inside this one** resource group.” Same policy types as at subscription—just a **smaller** target. |

**What you do in the portal (either scope):** *Policy* → *Assignments* → *Assign policy* → choose **scope** = that subscription **or** that resource group → pick the policy → save.

**Why pick subscription vs RG:** Use **subscription** for rules that should hit **the whole** sub. Use **RG** when only **one** project or environment needs extra rules (or you are delegating that slice to a team).

*(Management groups are **above** subscription: same idea—assign there to cover **many** subscriptions at once.)*

#### Azure RBAC (roles)

| Where you assign it | What you are doing in plain English |
|---------------------|-------------------------------------|
| **Subscription** | “This person or group can do **X** on **everything** in this subscription (every RG and resource), **for the actions that role allows**.” Examples: **Owner**, **Contributor**, **Reader** at subscription scope = wide access. |
| **Resource group** | “This person or group can do **X** **only** on resources **in this resource group**.” Same role **names** (Contributor, Reader, etc.)—just **limited** to that RG. |

**What you do in the portal (either scope):** open the **subscription** or **resource group** → *Access control (IAM)* → *Add role assignment* → pick **role** → pick **member** → save.

**Why pick subscription vs RG:** Use **subscription** for platform admins or org-wide access. Use **RG** to give a team **only** access to **their** app’s resources **without** touching the rest of the subscription.

#### One glance

| | **Policy** | **RBAC** |
|---|------------|----------|
| **At subscription** | Rules for **all** resources in that sub (unless you exclude scopes). | Permissions for **all** resources in that sub (for that role). |
| **At resource group** | Rules for **only** resources in **that** RG. | Permissions for **only** resources in **that** RG. |

**Nothing special** about “subscription Policy” vs “RG Policy” or “subscription RBAC” vs “RG RBAC”—it is the **same** features; only the **scope** (how big the blast radius is) changes.

### Portal: IAM vs Policy

- **Access control (IAM)** on a subscription, resource group, or resource = where you manage **Azure RBAC** (role assignments, custom roles, **Deny assignments** for RBAC, view who has access). **IAM** here is the **screen** for **identity + role permissions** on that scope.
- **Policy** (search **Policy** in the portal, or **Microsoft.Policy** in the menu) = where you manage **Azure Policy** (assignments, compliance, initiatives, exemptions).

So: **roles and who can do what** → **IAM** on the resource. **Rules about what resources must look like** (tags, regions, etc.) → **Policy** service. They are **related** (both governance) but **different** blades.

### Q: On “Add role assignment,” am I creating a role assignment? Where is the “resource”? Reader of *what*?

**A:** Yes — you are creating a **role assignment**: **who** (member) + **what role** (Reader) + **where that role applies** (**scope**).

In Azure RBAC, the **scope** is always **some** Azure object. The portal does not always show the word “resource,” but the **subscription**, **resource group**, or **single resource** you opened **IAM** from **is** the assignment’s scope.

**Your screenshot (subscription IAM):**

- You started from **Subscriptions → … → Access control (IAM)**.
- So the scope is the **whole subscription** (`sandpit-htx-Individual-yangjing`).
- **Reader** here means: **Reader on that subscription**, which **includes** (inherits to) **every resource group and every Azure resource** in that subscription, for actions the **Reader** role allows (view resources, read metadata—not change them unless the role says so).

So **Reader of what?** → **Reader of everything under this subscription** (VMs, storage, networking, etc.), **not** “Reader of nothing.” The “resource” the role is attached to **is the subscription**; everything below is **in scope** for that assignment.

If you had opened **IAM** on **one** resource group instead, the same **Reader** assignment would mean **Reader only** on resources **in that RG**. If you opened IAM on **one** storage account, **Reader** would apply **only** to that account.

**Three parts (every time):** **Principal** (who) · **Role definition** (Reader, Contributor, …) · **Scope** (subscription, RG, or resource).

### Q: “Assign access to: User, group, or service principal” vs “Managed identity” — what does that mean?

**A:** Both choices answer **who** gets the role. The difference is **what kind of identity** that is.

| Option | What it is | Typical use |
|--------|------------|-------------|
| **User, group, or service principal** | **People** (users), **Entra ID groups**, or a **service principal** from an **app registration** (an application’s identity in the directory). | A person logging into the portal; a team (group); an app or script that signs in as the registered app. |
| **Managed identity** | An identity **Azure manages for a specific Azure resource** (VM, App Service, Function, etc.)—**system-assigned** (tied to one resource) or **user-assigned** (a resource you create and attach to one or more resources). | Let **the Azure service itself** call other Azure APIs **without** storing passwords—e.g. a web app reading secrets from Key Vault, a VM accessing Storage. |

**In one line**

- **User / group / service principal** = **human or external app identity** in Microsoft Entra ID.
- **Managed identity** = **this Azure workload’s** built-in identity; you assign roles **to that identity** so the **service** can access something else in Azure.

You pick **Managed identity** when the **principal** that needs Reader/Contributor/etc. is **not** a person but **an Azure resource** acting on its own behalf.

### Q: After I create a role assignment with “User, group, or service principal,” does Azure generate API keys?

**A:** **No.** A **role assignment** only answers: **“This identity is allowed to do these actions on this scope.”** It does **not** create passwords, **client secrets**, **API keys**, or certificates.

**Two different ideas**

- **Authentication** = proving **who you are** (sign-in, token, secret, certificate, managed identity).
- **Authorization** = what you are **allowed** to do (**RBAC** roles and assignments).

Creating a role assignment is **authorization only**.

**How each principal type actually signs in (separate from the role assignment)**

| Principal | Credentials (not created by RBAC assignment) |
|-----------|-----------------------------------------------|
| **User** | Normal Microsoft Entra sign-in (password, MFA, etc.). |
| **Group** | Members use **their own** user credentials; the group just bundles role assignments. |
| **Service principal** (app) | The app uses whatever you configure on the **App registration**: **client secret**, **certificate**, or federated credentials—**you** (or an admin) create those in **Entra ID** / the app blade if the app needs them. The role assignment does **not** generate them. |
| **Managed identity** | Azure obtains tokens for you; **no** classic API key to paste for that identity. |

So: **role assignment** = **permission**. **Keys/secrets** = configured **elsewhere** when an app or automation needs them, or not needed for users and managed identities in the same way.

### Q: I set a member—does that mean they can use *any* of User, group, or service principal to prove who they are?

**A:** **No.** “**User, group, or service principal**” is **not** a menu of **login methods** for one person. It is **which kind of identity** you are attaching the role **to**—you pick **one** category, then pick **specific** users, **one** group, or **one** service principal.

| What you chose as the member | Who proves identity | How |
|------------------------------|---------------------|-----|
| **A user** (e.g. Denny) | **That user only** | They sign in as **that** Entra user (password, MFA, etc.). No one else “becomes” Denny via another option. |
| **A group** | **Each user who is in the group** | Each person signs in as **their own** user account. The **group** is how RBAC targets many people at once; they don’t pick “service principal” instead of their user. |
| **A service principal** | **The application** (automation, script, backend) | The **app** authenticates as the **registered app** (client ID + secret/cert/federation)—**not** your personal login. Humans don’t “use service principal” as an alternate way to sign in as themselves. |

**Important:** A **person** and an **app registration** are **different identities**. You don’t assign Reader to “Denny **or** the app—whichever shows up first.” You assign to **Denny** (user) **or** to **the app** (service principal), depending on who should have the permission.

**Managed identity** (separate radio button) is again **one** identity type: an **Azure resource’s** identity—not a mix-and-match with user login.

**One line:** The wizard asks **who** gets the role (**user**, **group**, **app**, or **managed identity**). Each of those proves identity in **its own** way; you don’t get three interchangeable proof methods for a single member.

### Q: I selected the bullet “User, group, or service principal” but you said it becomes one of them—when? Can Lester choose how to use the role?

**A:** The portal uses **one** radio label for **three related kinds** of directory objects (people, groups, apps). It is **not** asking Lester to pick a login style.

**Two top-level choices (pick one):**

1. **User, group, or service principal** — identities in **Microsoft Entra ID** (humans, **or** a **group** of humans, **or** an **app registration’s** service principal).
2. **Managed identity** — identities **Azure attaches to a resource** (not the same bucket as above).

You chose **(1)**. Then you clicked **+ Select members** and added **Lester**. The table shows **Type: User** because **Lester is a user account**. That row was **always** a user; nothing “switches” later. If you had added a **group**, **Type** would say **Group**; if you had added an **app**, **Type** would say **Service principal**.

**Can Lester “decide” between user, group, and service principal?** **No.** He is **one** identity: a **user** (guest). He signs in as **Lester** only. He does **not** choose service principal mode for himself—that option exists only for **applications**, not as an alternate path for a person.

**Summary:** The long label means **“search the directory for users, groups, or app identities”**—not **“each member may authenticate three different ways.”**

**Select members pane:** The list can include **users**, **groups**, and **service principals** depending on your search. You may add **one or more** rows; each row is **one** of those types (shown in **Type**). You are **not** required to pick only users—e.g. you can assign the role to a **group** so everyone in the group gets access, or to a **service principal** for an app.

### Q: Is a resource group managed inside my subscription?

**A:** **Yes.** Every resource group **belongs to one subscription** and is created **under** it. Hierarchy: **subscription → resource groups → resources**. Billing and subscription-level limits still apply to everything in those RGs.

### Q: Policy Overview doesn’t show creating policies or attaching to subscription/RG—where is that?

**A:** **Overview** is a **dashboard** (compliance charts, how many policies/initiatives, scope filter). It is **not** the main screen to **assign** or **author** policies.

**Typical places in the Policy blade**

| What you want | Where to go |
|----------------|-------------|
| **Attach** a built-in or custom policy to a subscription, MG, or RG | **Assignments** → **Assign** (or **+ Assign policy**). There you set **Scope** = your subscription or resource group and pick a **Policy** or **Initiative**. |
| **See** what is assigned | **Assignments** (list of assignments and scopes). |
| **Create** your own policy **definition** (advanced) | **Definitions** (or **Authoring** in some portal layouts) → create **custom** policy definition. Most people start with **built-in** definitions when assigning. |
| **Compliance** details | **Compliance** — drill into non-compliant resources. |
| **Fix** drift automatically where supported | **Remediation** |

The **Scope: X selected** at the top of **Overview** only **filters the dashboard** to those subscriptions/MGs—it does **not** by itself create assignments; use **Assignments → Assign**.

**Shortcut:** Open a **subscription** or **resource group** in the portal → **Policy** in the left menu (under **Settings** or **Operations** depending on layout) → **Assign policy** scoped to **that** resource.

### Q: Is policy “one for subscription and one for RG” combined? Do I select subscription **and** RG before creating a policy?

**A:** Each **assignment** has **one** **primary scope** you choose in the wizard:

- **Management group**, **or**
- **Subscription**, **or**
- **Resource group** (sometimes down to a resource)

You **do not** normally pick “subscription **and** RG” as a **single merged** scope in one assignment. You pick **one** level:

| If you set scope to… | What the assignment applies to |
|----------------------|--------------------------------|
| **Subscription** | All resource groups and resources **under that subscription** (unless you add **exclusions**). |
| **One resource group** | Only resources **in that RG**. |

**“Combined” effect:** If you have **multiple** assignments (e.g. one at **subscription** and another at **another** RG, or two different policies), **all** that apply to a resource are **evaluated together**—that is the **cumulative** effect we discussed earlier. It is **not** one wizard step that means “subscription + RG” as one scope.

**Exclusions:** You *can* assign at **subscription** scope and use **exclusion** / **notScopes** to **leave out** specific RGs or resources—advanced, but useful.

**Policy Overview “Scope: 2 selected”:** That only filters **which subscriptions** (or MGs) the **dashboard** shows. It is **not** “one assignment covering two scopes” in one go.

**Practical rule:** Want the rule **everywhere** in the sub? Scope = **subscription**. Want it **only** for one app folder? Scope = **that resource group** only.

**Portal nuance:** In **Assign policy**, the **Scope** picker often shows **Subscription** and **Resource group** as two steps because an RG **lives under** a subscription—you choose the sub **first**, then **optionally** narrow to **one** RG (e.g. `subscription-name/rg-law`). That path is still **one** assignment scope (that RG), not two independent policies.
