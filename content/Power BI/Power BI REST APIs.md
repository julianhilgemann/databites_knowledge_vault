# Power BI REST APIs & Automation

The Power BI REST API is the backbone of enterprise automation. It allows programmatic management of workspaces, datasets, capacities, and user permission without clicking through the UI.

## 1. Authentication (Service Principals)

Never automate production processes using an individual user account (due to MFA and employee turnover). Always use a **Service Principal** (an App Registration in Entra ID/Azure AD).

**Requirements for Service Principals:**
1.  Register in Azure AD with a Client ID and Secret.
2.  Power BI Tenant Admin must enable "Allow service principals to use Power BI APIs".
3.  Add the Service Principal to the target workspace as a Contributor or Admin.

## 2. The Semantic Model Refresh Pipeline

The standard UI scheduled refresh is limited (8x per day for Pro, 48x for Premium) and inflexible (statically scheduled regardless of when upstream ETL finishes).

Using the REST API, you can orchestrate an event-driven refresh.
When Azure Data Factory or [[dbt]] finishes loading the [[SQL]] data warehouse, it triggers an Azure Function or Logic App which POSTs to:
`POST https://api.powerbi.com/v1.0/myorg/groups/{groupId}/datasets/{datasetId}/refreshes`

*   **Pro Tip:** If you have a Premium capacity workspace, you can trigger an **Enhanced Refresh** via the API. By sending a JSON body with the request, you can refresh *specific partitions* or specific [[Tables|tables]] instead of the whole model.

```json
// Example Body for Enhanced Refresh
{
  "type": "DataOnly",
  "objects": [
    {
      "database": "SalesModel",
      "table": "FactSales",
      "partition": "2024-Q1"
    }
  ]
}
```

## 3. Scanner APIs (Admin Governance)

Large enterprises quickly lose track of what assets exist. The Admin Scanner APIs (`GetWorkspaceInfo`, `GetScanResult`) allow an administrator to bulk extract the entire metadata of a tenant:
*   What datasets exist?
*   Who has access to what, exactly?
*   Are there personal gateways pointing to dangerous unencrypted [[SQL]] endpoints?
*   What [[Tables|tables]] and columns are actually being used by reports?

## 4. Automation via PowerShell

Microsoft maintains the `MicrosoftPowerBIMgmt` module natively.
```powershell
Connect-PowerBIServiceAccount -ServicePrincipal -Credential $credential -TenantId $tenant
Invoke-PowerBIRestMethod -Url "groups/$groupId/datasets/$datasetId/refreshes" -Method Post
```

### Summary Checklist
- [ ] Is production automation running via Service Principals rather than user accounts?
- [ ] Are heavy dataset refreshes triggered via API exactly when the upstream ETL finishes?
- [ ] Is the business leveraging the Scanner APIs to maintain a central catalog of artifacts?
