# real-world-defender-for-cloud-foundations

# Defender for Cloud Real-World Project

## Project Overview

This project demonstrates how to use **Microsoft Defender for Cloud** in a more realistic Azure security scenario.

The project focuses on protecting an Azure virtual machine with **Defender for Servers**, reviewing security recommendations, investigating security assessments, exporting Defender findings to Log Analytics, and using KQL to analyze security data.

The goal is to simulate how a cloud security engineer could use Defender for Cloud to monitor and improve the security posture of a production workload.

---

## Objectives

By completing this project, you will learn how to:

- Create a production-style Azure virtual machine
- Enable Defender for Servers
- Review Defender for Cloud recommendations
- Investigate security assessments
- Review VM-related vulnerabilities and security findings
- Configure continuous export
- Send Defender findings to Log Analytics
- Query Defender data using KQL
- Understand Defender workflow automation
- Clean up resources and disable paid plans

---

## Architecture

```text
Azure Virtual Machine
        |
        v
Defender for Servers
        |
        +-- Security Assessments
        +-- Recommendations
        +-- Vulnerability Findings
        +-- Threat Protection
        |
        v
Microsoft Defender for Cloud
        |
        v
Continuous Export
        |
        v
Log Analytics Workspace
        |
        v
KQL Queries
```

---

## Prerequisites

You will need:

- An Azure subscription
- Azure CLI installed
- Access to the Azure portal
- Permission to create resources
- Permission to manage Defender for Cloud settings

Sign in:

```bash
az login
```

Check your subscription:

```bash
az account show \
  --output table
```

---

## Step 1 — Configure Variables

```bash
RG="rg-defender-production"
LOCATION="canadacentral"

VM="vm-defender-production"
LAW="law-defender-production-$RANDOM"
ADMIN="azureuser"
```

---

## Step 2 — Create the Resource Group

```bash
az group create \
  --name "$RG" \
  --location "$LOCATION"
```

Verify:

```bash
az group show \
  --name "$RG" \
  --output table
```

---

## Step 3 — Create Log Analytics

```bash
az monitor log-analytics workspace create \
  --resource-group "$RG" \
  --workspace-name "$LAW" \
  --location "$LOCATION"
```

Verify:

```bash
az monitor log-analytics workspace show \
  --resource-group "$RG" \
  --workspace-name "$LAW" \
  --output table
```

The workspace will later receive Defender for Cloud security data.

---

## Step 4 — Create the Virtual Machine

Create a Linux VM without a public IP:

```bash
az vm create \
  --resource-group "$RG" \
  --name "$VM" \
  --image Ubuntu2204 \
  --size Standard_B2s \
  --admin-username "$ADMIN" \
  --generate-ssh-keys \
  --public-ip-address ""
```

Verify:

```bash
az vm show \
  --resource-group "$RG" \
  --name "$VM" \
  --show-details \
  --output table
```

Avoiding a public IP reduces unnecessary internet exposure.

---

## Step 5 — Check Defender Plans

View your current Defender pricing configuration:

```bash
az security pricing list \
  --query "[].{Plan:name,Tier:pricingTier,SubPlan:subPlan}" \
  --output table
```

You can also check Defender for Servers directly:

```bash
az security pricing show \
  --name VirtualMachines \
  --output jsonc
```

If the plan shows:

```text
pricingTier: Free
```

Defender for Servers is not enabled yet.

---

## Step 6 — Enable Defender for Servers Plan 2

> Note: Defender for Servers Plan 2 is a paid Defender for Cloud plan. Use a sandbox or lab subscription where possible.

Enable Plan 2:

```bash
az security pricing create \
  --name VirtualMachines \
  --tier standard \
  --subplan P2
```

Verify:

```bash
az security pricing show \
  --name VirtualMachines \
  --query "{Plan:name,Tier:pricingTier,SubPlan:subPlan,Trial:freeTrialRemainingTime}" \
  --output table
```

Expected result:

```text
Plan             Tier      SubPlan
---------------  --------  -------
VirtualMachines  Standard  P2
```

---

## Step 7 — Review Defender for Cloud

In the Azure portal, navigate to:

```text
Microsoft Defender for Cloud
        |
        v
Environment settings
        |
        v
Your Subscription
        |
        v
Defender plans
```

Confirm that:

```text
Servers = On
```

Then go to:

```text
Defender for Cloud
        |
        v
Inventory
```

Locate:

```text
vm-defender-production
```

---

## Step 8 — Review Security Recommendations

Navigate to:

```text
Defender for Cloud
        |
        v
Recommendations
```

Filter by:

```text
Resource type: Virtual Machines
```

or search for:

```text
vm-defender-production
```

Review any recommendations related to:

- Operating system vulnerabilities
- Missing patches
- Network exposure
- Authentication
- Endpoint protection
- Disk security
- Security configuration

---

## Step 9 — Query Security Assessments

List all Defender assessments:

```bash
az security assessment list \
  --output jsonc
```

Show only unhealthy assessments:

```bash
az security assessment list \
  --query "[?status.code=='Unhealthy']" \
  --output jsonc
```

View detailed sub-assessments:

```bash
az security sub-assessment list \
  --output jsonc
```

This allows you to investigate more detailed security findings.

---

## Step 10 — Remediate a Recommendation

Select one genuine Defender recommendation affecting the VM.

Follow this process:

```text
Identify Recommendation
        |
        v
Review Severity
        |
        v
Inspect Affected Resource
        |
        v
Read Remediation Guidance
        |
        v
Apply Security Fix
        |
        v
Verify Azure Configuration
        |
        v
Wait for Defender Reassessment
```

The goal is to practice using Defender for Cloud as the source of the security finding rather than randomly applying hardening changes.

---

## Step 11 — Configure Continuous Export

In the Azure portal, navigate to:

```text
Defender for Cloud
        |
        v
Environment settings
        |
        v
Your Subscription
        |
        v
Continuous export
```

Configure:

```text
Export Target:
Log Analytics Workspace
```

Select:

- Security recommendations
- Security alerts

Choose your workspace:

```text
law-defender-production-xxxxx
```

Enable streaming export and save.

---

## Step 12 — Query Defender Data with KQL

Open:

```text
Log Analytics Workspace
        |
        v
Logs
```

View security recommendations:

```kusto
SecurityRecommendation
| take 20
```

View recent recommendations:

```kusto
SecurityRecommendation
| where TimeGenerated > ago(24h)
| take 50
```

View security alerts:

```kusto
SecurityAlert
| where TimeGenerated > ago(24h)
| take 20
```

Count alerts by severity:

```kusto
SecurityAlert
| summarize Alerts=count() by AlertSeverity
| order by Alerts desc
```

This demonstrates how Defender for Cloud integrates with Log Analytics and KQL.

---

## Step 13 — Understand Security Automation

Defender for Cloud can trigger workflow automation when:

- Security alerts occur
- Recommendations change
- Compliance findings change

A typical workflow might look like:

```text
Defender Alert
      |
      v
Workflow Automation
      |
      v
Logic App
      |
      +-- Email
      +-- Teams
      +-- ITSM Ticket
      +-- Automated Response
```

This is useful for security operations and incident-response workflows.

---

## Troubleshooting

### Defender for Servers still shows Free

Check:

```bash
az security pricing show \
  --name VirtualMachines \
  --output jsonc
```

If necessary, enable Plan 2 again:

```bash
az security pricing create \
  --name VirtualMachines \
  --tier standard \
  --subplan P2
```

---

### AuthorizationFailed

Check the current account:

```bash
az account show \
  --output table
```

Retrieve your user object ID:

```bash
MY_ID=$(az ad signed-in-user show \
  --query id \
  --output tsv)
```

Check your role assignments:

```bash
az role assignment list \
  --assignee "$MY_ID" \
  --all \
  --output table
```

---

### No recommendations appear

Verify the VM exists:

```bash
az vm show \
  --resource-group "$RG" \
  --name "$VM" \
  --output table
```

Check Defender assessments:

```bash
az security assessment list \
  --output jsonc
```

Defender for Cloud may take some time to evaluate newly created resources.

---

### Log Analytics shows no Defender data

Confirm that Continuous Export is configured.

Then try a broad query:

```kusto
SecurityRecommendation
| take 10
```

or:

```kusto
SecurityAlert
| take 10
```

Security data may not appear until new assessments or alerts are generated.

---

## Clean Up

Because Defender for Servers Plan 2 can generate charges, disable it when you are finished with the lab:

```bash
az security pricing create \
  --name VirtualMachines \
  --tier free
```

Verify:

```bash
az security pricing show \
  --name VirtualMachines \
  --query "{Plan:name,Tier:pricingTier,SubPlan:subPlan}" \
  --output table
```

Then delete the project resources:

```bash
az group delete \
  --name "$RG" \
  --yes \
  --no-wait
```

---

## Key Concepts Learned

This project introduced:

- Microsoft Defender for Cloud
- Defender for Servers
- Cloud workload protection
- Security recommendations
- Security assessments
- Sub-assessments
- Vulnerability findings
- Continuous Export
- Log Analytics
- KQL
- Security automation
- Cloud security remediation

---

## Useful Commands

```bash
# View Defender plans
az security pricing list \
  --query "[].{Plan:name,Tier:pricingTier,SubPlan:subPlan}" \
  --output table

# Check Defender for Servers
az security pricing show \
  --name VirtualMachines \
  --output jsonc

# Enable Defender for Servers Plan 2
az security pricing create \
  --name VirtualMachines \
  --tier standard \
  --subplan P2

# Disable Defender for Servers
az security pricing create \
  --name VirtualMachines \
  --tier free

# List all security assessments
az security assessment list \
  --output jsonc

# View unhealthy assessments
az security assessment list \
  --query "[?status.code=='Unhealthy']" \
  --output jsonc

# View detailed findings
az security sub-assessment list \
  --output jsonc
```

---

## Project Outcome

By completing this project, you have practiced a realistic Defender for Cloud security workflow:

```text
Deploy Azure Workload
        |
        v
Enable Defender Protection
        |
        v
Monitor Security Posture
        |
        v
Identify Security Findings
        |
        v
Investigate Recommendations
        |
        v
Remediate Security Issues
        |
        v
Export Security Data
        |
        v
Analyze with KQL
        |
        v
Improve Cloud Security
```

This project provides a practical foundation for more advanced topics such as Defender for Containers, Microsoft Sentinel, Azure Policy, security automation, regulatory compliance, and full DevSecOps security monitoring.