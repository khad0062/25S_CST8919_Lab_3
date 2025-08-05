# 25S_CST8919_Lab_3

# Azure Policy Lab – MapleTech Secure Foundation

##  Summary
This lab demonstrates the implementation of governance, security, and compliance controls in Azure using **Azure Policy**.  

The scenario involves **MapleTech Solutions**, a growing Canadian cloud-native company, where developers are:
- Deploying resources globally
- Exposing public IP addresses
- Skipping mandatory tags

All policies were grouped into a **Policy Initiative** for easier management and assigned to a Resource Group with **Enforcement mode enabled**.

---

##  Lab Objectives
- Create **three custom Azure Policies**:
  1. **Only-CanadaCentral** – Restricts deployments to the Canada Central region.
  2. **Require-ProjectName-Tag** – Requires a `ProjectName` tag.
  3. **Deny-Public-IP** – Blocks creation of Public IP addresses.
- Group policies into a **Policy Initiative**.
- Assign the initiative to a **Resource Group**.
- Test enforcement by simulating developer actions.

---

## Policies

### 1. Only-CanadaCentral
**Effect:** Deny  
**Description:** Denies any resource that is not deployed in `canadacentral`.  
**Purpose:** Enforces compliance with regional deployment policy.

```
{
  "properties": {
    "displayName": "Only-CanadaCentral",
    "policyType": "Custom",
    "mode": "All",
    "description": "Denies any resource that is not being deployed in the Canada Central region.",
    "metadata": { "category": "Security" },
    "parameters": {},
    "policyRule": {
      "if": {
        "not": { "field": "location", "equals": "canadacentral" }
      },
      "then": { "effect": "Deny" }
    }
  }
}

```

### 2. Require-ProjectName-Tag
**Effect:** Deny  
**Description:** Requires all resources to have a `ProjectName` tag.  
**Purpose:** Ensures proper tagging for cost tracking and governance.

```
{
  "properties": {
    "displayName": "Require-ProjectName-Tag",
    "policyType": "Custom",
    "mode": "Indexed",
    "description": "Requires all resources to include a ProjectName tag.",
    "metadata": { "category": "Compliance" },
    "parameters": {},
    "policyRule": {
      "if": {
        "field": "[concat('tags[', 'ProjectName', ']')]",
        "exists": "false"
      },
      "then": { "effect": "Deny" }
    }
  }
}

```

### 3. Deny-Public-IP
**Effect:** Deny  
**Description:** Prevents creation of Public IP addresses.  
**Purpose:** Reduces attack surface by avoiding unnecessary public exposure.
```
{
  "properties": {
    "displayName": "Deny-Public-IP",
    "policyType": "Custom",
    "mode": "All",
    "description": "Prevents creation of Public IP addresses.",
    "metadata": { "category": "Security" },
    "parameters": {},
    "policyRule": {
      "if": {
        "field": "type",
        "equals": "Microsoft.Network/publicIPAddresses"
      },
      "then": { "effect": "Deny" }
    }
  }
}

```


---

##  Policy Initiative
**Name:** MapleTech Secure Foundation  
**Category:** Security  
**Policies Included:**
- Only-CanadaCentral
- Require-ProjectName-Tag
- Deny-Public-IP

**Assignment Scope:** Specific Resource Group  
**Enforcement Mode:** **Enabled** (Deny non-compliant resources)

---

##  Lessons Learned
- **Azure Policy** provides proactive enforcement — resources are blocked before deployment if non-compliant.
- Grouping policies into an **initiative** simplifies management and ensures consistent governance.
- **Enforcement Mode** must be enabled for Deny effects to take effect — otherwise policies only audit.
- Tagging policies help maintain cost management, compliance, and organizational visibility.

---

## Challenges
- Remembering to enable **Policy Enforcement** during assignment — otherwise test cases won’t deny resources.
- Deploying via the Azure Portal is straightforward, but for automation, **ARM/Bicep** templates would be more efficient.
- Testing policies requires creating/destroying resources, which can consume time and budget if not planned.

## 🔍 Test Cases

| Test Action | Expected Result | Outcome |
|-------------|----------------|---------|
| Deploy VM in East US | ❌ Denied | ✅ Passed |
| Deploy Storage Account without `ProjectName` tag | ❌ Denied | ✅ Passed |
| Create a Public IP | ❌ Denied | ✅ Passed |
| Deploy VM in Canada Central with `ProjectName` tag | ✅ Allowed | ✅ Passed |

---

## 📸 Screenshots
Screenshots are stored in the `/screenshots` folder:
- `east-us-denied.png`
- `no-tag-denied.png`
- `public-ip-denied.png`
- `canada-central-success.png`

---

## 📂 Repository Structure
