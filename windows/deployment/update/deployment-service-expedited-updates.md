---
title: Deploy expedited updates
titleSuffix: Windows Update for Business deployment service
description: Learn how to use Windows Update for Business deployment service to deploy expedited updates to devices in your organization. 
ms.prod: windows-client
ms.technology: itpro-updates
ms.topic: conceptual
ms.author: mstewart
author: mestew
manager: aaroncz
ms.collection:
  - tier1
ms.localizationpriority: medium
appliesto: 
- ✅ <a href=https://learn.microsoft.com/windows/release-health/supported-versions-windows-client target=_blank>Windows 11</a>
- ✅ <a href=https://learn.microsoft.com/windows/release-health/supported-versions-windows-client target=_blank>Windows 10</a>
ms.date: 08/29/2023
---

# Deploy expedited updates with Windows Update for Business deployment service
<!--7512398-->

In this article, you will:
> [!div class="checklist"]
>
> * [Open Graph Explorer](#open-graph-explorer)
> * [Run queries to identify test devices](#run-queries-to-identify-devices)
> * [List catalog entries for expedited updates](#list-catalog-entries-for-expedited-updates)
> * [Create a deployment](#create-a-deployment)
> * [Add members to the deployment audience](#add-members-to-the-deployment-audience)
> * [Delete a deployment](#delete-a-deployment)

## Prerequisites

All of the [prerequisites for the Windows Update for Business deployment service](deployment-service-prerequisites.md) must be met.

### Permissions

<!--Using include for Graph Explorer permissions-->
[!INCLUDE [Windows Update for Business deployment service permissions using Graph Explorer](./includes/wufb-deployment-graph-explorer-permissions.md)]

## Open Graph Explorer

<!--Using include for Graph Explorer sign in-->
[!INCLUDE [Graph Explorer sign in](./includes/wufb-deployment-graph-explorer.md)]

## Run queries to identify devices

<!--Using include for Graph Explorer device queries-->
[!INCLUDE [Graph Explorer device queries](./includes/wufb-deployment-find-device-name-graph-explorer.md)]

## List catalog entries for expedited updates

Each update is associated with a unique [catalog entry](/graph/api/resources/windowsupdates-catalogentry). You can query the catalog to find updates that can be expedited. The `id` returned is the **Catalog ID** and is used to create a deployment. The following query lists all security updates that can be deployed as expedited updates by the deployment service. Using `$top=1` and ordering by `ReleaseDateTimeshows` displays the most recent update that can be deployed as expedited.

```msgraph-interactive
GET https://graph.microsoft.com/beta/admin/windows/updates/catalog/entries?$filter=isof('microsoft.graph.windowsUpdates.qualityUpdateCatalogEntry') and microsoft.graph.windowsUpdates.qualityUpdateCatalogEntry/isExpeditable eq true&$orderby=releaseDateTime desc&$top=1
```

The following truncated response displays a **Catalog ID** of  `e317aa8a0455ca604de95329b524ec921ca57f2e6ed3ff88aac757a7468998a5` for the `08/08/2023 - 2023.08 B SecurityUpdate for Windows 10 and later` security update:

```json
{
    "@odata.context": "https://graph.microsoft.com/beta/$metadata#admin/windows/updates/catalog/entries",
    "value": [
        {
            "@odata.type": "#microsoft.graph.windowsUpdates.qualityUpdateCatalogEntry",
            "id": "e317aa8a0455ca604de95329b524ec921ca57f2e6ed3ff88aac757a7468998a5",
            "displayName": "08/08/2023 - 2023.08 B SecurityUpdate for Windows 10 and later",
            "deployableUntilDateTime": null,
            "releaseDateTime": "2023-08-08T00:00:00Z",
            "isExpeditable": true,
            "qualityUpdateClassification": "security",
            "catalogName": "2023-08 Cumulative Update for Windows 10 and later",
            "shortName": "2023.08 B",
            "qualityUpdateCadence": "monthly",
            "cveSeverityInformation": {
                "maxSeverity": "critical",
                "maxBaseScore": 9.8,
                "exploitedCves@odata.context": "https://graph.microsoft.com/$metadata#admin/windows/updates/catalog/entries('e317aa8a0455ca604de95329b524ec921ca57f2e6ed3ff88aac757a7468998a5')/microsoft.graph.windowsUpdates.qualityUpdateCatalogEntry/cveSeverityInformation/exploitedCves",
                "exploitedCves": [
                    {
                        "number": "ADV230003",
                        "url": "https://msrc.microsoft.com/update-guide/vulnerability/ADV230003"
                    },
                    {
                        "number": "CVE-2023-38180",
                        "url": "https://msrc.microsoft.com/update-guide/vulnerability/CVE-2023-38180"
                    }
                ]
            }
        }
    ]
}
```

The deployment service can display more information about updates that were released on or after January 2023. Using [product revision](/graph/api/resources/windowsupdates-productrevision) gives you additional information about the updates, such as the KB numbers, and the `MajorVersion.MinorVersion.BuildNumber.UpdateBuildRevision`. Windows 10 and 11 share the same major and minor versions, but have different build numbers. 
<!--8092737-->
Use the following to display the product revision information for the most recent quality update:

```msgraph-interactive
GET https://graph.microsoft.com/beta/admin/windows/updates/catalog/entries?$expand=microsoft.graph.windowsUpdates.qualityUpdateCatalogEntry/productRevisions&$orderby=releaseDateTime desc&$top=1
```


The following truncated response displays information about KB5029244 for Windows 10, version 22H2, and KB5029263 for Windows 11, version 22H2:

```json
{
    "@odata.context": "https://graph.microsoft.com/beta/$metadata#admin/windows/updates/catalog/entries(microsoft.graph.windowsUpdates.qualityUpdateCatalogEntry/productRevisions())",
    "value": [
        {
            "@odata.type": "#microsoft.graph.windowsUpdates.qualityUpdateCatalogEntry",
            "id": "e317aa8a0455ca604de95329b524ec921ca57f2e6ed3ff88aac757a7468998a5",
            "displayName": "08/08/2023 - 2023.08 B SecurityUpdate for Windows 10 and later",
            "deployableUntilDateTime": null,
            "releaseDateTime": "2023-08-08T00:00:00Z",
            "isExpeditable": true,
            "qualityUpdateClassification": "security",
            "catalogName": "2023-08 Cumulative Update for Windows 10 and later",
            "shortName": "2023.08 B",
            "qualityUpdateCadence": "monthly",
            "cveSeverityInformation": {
                "maxSeverity": "critical",
                "maxBaseScore": 9.8,
                "exploitedCves@odata.context": "https://graph.microsoft.com/beta/$metadata#admin/windows/updates/catalog/entries('e317aa8a0455ca604de95329b524ec921ca57f2e6ed3ff88aac757a7468998a5')/microsoft.graph.windowsUpdates.qualityUpdateCatalogEntry/cveSeverityInformation/exploitedCves",
                "exploitedCves": [
                    {
                        "number": "ADV230003",
                        "url": "https://msrc.microsoft.com/update-guide/vulnerability/ADV230003"
                    },
                    {
                        "number": "CVE-2023-38180",
                        "url": "https://msrc.microsoft.com/update-guide/vulnerability/CVE-2023-38180"
                    }
                ]
            },
            "productRevisions@odata.context": "https://graph.microsoft.com/beta/$metadata#admin/windows/updates/catalog/entries('e317aa8a0455ca604de95329b524ec921ca57f2e6ed3ff88aac757a7468998a5')/microsoft.graph.windowsUpdates.qualityUpdateCatalogEntry/microsoft.graph.windowsUpdates.qualityUpdateCatalogEntry/productRevisions",
            "productRevisions": [
                {
                    "id": "10.0.19045.3324",
                    "displayName": "Windows 10, version 22H2, build 19045.3324",
                    "releaseDateTime": "2023-08-08T00:00:00Z",
                    "version": "22H2",
                    "product": "Windows 10",
                    "osBuild": {
                        "majorVersion": 10,
                        "minorVersion": 0,
                        "buildNumber": 19045,
                        "updateBuildRevision": 3324
                    },
                    "knowledgeBaseArticle@odata.context": "https://graph.microsoft.com/beta/$metadata#admin/windows/updates/catalog/entries('e317aa8a0455ca604de95329b524ec921ca57f2e6ed3ff88aac757a7468998a5')/microsoft.graph.windowsUpdates.qualityUpdateCatalogEntry/microsoft.graph.windowsUpdates.qualityUpdateCatalogEntry/productRevisions('10.0.19045.3324')/knowledgeBaseArticle/$entity",
                    "knowledgeBaseArticle": {
                        "id": "KB5029244",
                        "url": "https://support.microsoft.com/help/5029244"
                    }
                },
                {
                    "id": "10.0.22621.2134",
                    "displayName": "Windows 11, version 22H2, build 22621.2134",
                    "releaseDateTime": "2023-08-08T00:00:00Z",
                    "version": "22H2",
                    "product": "Windows 11",
                    "osBuild": {
                        "majorVersion": 10,
                        "minorVersion": 0,
                        "buildNumber": 22621,
                        "updateBuildRevision": 2134
                    },
                    "knowledgeBaseArticle@odata.context": "https://graph.microsoft.com/beta/$metadata#admin/windows/updates/catalog/entries('e317aa8a0455ca604de95329b524ec921ca57f2e6ed3ff88aac757a7468998a5')/microsoft.graph.windowsUpdates.qualityUpdateCatalogEntry/microsoft.graph.windowsUpdates.qualityUpdateCatalogEntry/productRevisions('10.0.22621.2134')/knowledgeBaseArticle/$entity",
                    "knowledgeBaseArticle": {
                        "id": "KB5029263",
                        "url": "https://support.microsoft.com/help/5029263"
                    }
                },
```

## Create a deployment

When creating a deployment, there are [multiple options](/graph/api/resources/windowsupdates-deploymentsettings) available to define how the deployment behaves. The following example creates a deployment for the `08/08/2023 - 2023.08 B SecurityUpdate for Windows 10 and later` security update with catalog entry ID `e317aa8a0455ca604de95329b524ec921ca57f2e6ed3ff88aac757a7468998a5`, and defines the `expedite` and `userExperience` deployment options in the request body.

```msgraph-interactive
POST https://graph.microsoft.com/beta/admin/windows/updates/deployments
content-type: application/json

{
    "@odata.type": "#microsoft.graph.windowsUpdates.deployment",
    "content": {
        "@odata.type": "#microsoft.graph.windowsUpdates.catalogContent",
        "catalogEntry": {
            "@odata.type": "#microsoft.graph.windowsUpdates.qualityUpdateCatalogEntry",
            "id": "e317aa8a0455ca604de95329b524ec921ca57f2e6ed3ff88aac757a7468998a5"
        }
    },
    "settings": {
        "@odata.type": "microsoft.graph.windowsUpdates.deploymentSettings",
        "expedite": {
            "isExpedited": true
        },
        "userExperience": {
            "daysUntilForcedReboot": 2
        }
    }
}
```

The request returns a 201 Created response code and a [deployment](/graph/api/resources/windowsupdates-deployment) object in the response body for the newly created deployment, which includes:

- The **Deployment ID** `de910e12-3456-7890-abcd-ef1234567890` of the newly created deployment.
- The **Audience ID** `d39ad1ce-0123-4567-89ab-cdef01234567` of the newly created deployment audience.

```json
{
    "@odata.context": "https://graph.microsoft.com/beta/$metadata#admin/windows/updates/deployments/$entity",
    "id": "de910e12-3456-7890-abcd-ef1234567890",
    "createdDateTime": "2023-02-09T22:55:04.8547517Z",
    "lastModifiedDateTime": "2023-02-09T22:55:04.8547524Z",
    "state": {
        "effectiveValue": "offering",
        "requestedValue": "none",
        "reasons": []
    },
    "content": {
        "@odata.type": "#microsoft.graph.windowsUpdates.catalogContent",
        "catalogEntry@odata.context": "https://graph.microsoft.com/beta/$metadata#admin/windows/updates/deployments('de910e12-3456-7890-abcd-ef1234567890')/content/microsoft.graph.windowsUpdates.catalogContent/catalogEntry/$entity",
        "catalogEntry": {
            "@odata.type": "#microsoft.graph.windowsUpdates.qualityUpdateCatalogEntry",
            "id": "693fafea03c24cca819b3a15123a8880f217b96a878b6d6a61be021d476cc432",
            "displayName": null,
            "deployableUntilDateTime": null,
            "releaseDateTime": "2023-01-10T00:00:00Z",
            "isExpeditable": false,
            "qualityUpdateClassification": "security"
        }
    },
    "settings": {
        "schedule": null,
        "monitoring": null,
        "contentApplicability": null,
        "userExperience": {
            "daysUntilForcedReboot": 2
        },
        "expedite": {
            "isExpedited": true
        }
    },
    "audience@odata.context": "https://graph.microsoft.com/beta/$metadata#admin/windows/updates/deployments('de910e12-3456-7890-abcd-ef1234567890')/audience/$entity",
    "audience": {
        "id": "d39ad1ce-0123-4567-89ab-cdef01234567",
        "applicableContent": []
    }
}
```

## Add members to the deployment audience

The **Audience ID**, `d39ad1ce-0123-4567-89ab-cdef01234567`, was created when the deployment was created. The **Audience ID** is used to add members to the deployment audience. After the deployment audience is updated, Windows Update starts offering the update to the devices according to the deployment settings. As long as the deployment exists and the device is in the audience, the update will be expedited.

The following example adds two devices to the deployment audience using the **Azure AD ID** for each device:

```msgraph-interactive
POST https://graph.microsoft.com/beta/admin/windows/updates/deploymentAudiences/d39ad1ce-0123-4567-89ab-cdef01234567/updateAudience
content-type: application/json

{
  "addMembers": [
    {
      "@odata.type": "#microsoft.graph.windowsUpdates.azureADDevice",
      "id": "01234567-89ab-cdef-0123-456789abcdef"
    },
    {
      "@odata.type": "#microsoft.graph.windowsUpdates.azureADDevice",
      "id": "01234567-89ab-cdef-0123-456789abcde0"
    }
  ]
}
```

To verify the devices were added to the audience, run the following query using the **Audience ID** of `d39ad1ce-0123-4567-89ab-cdef01234567`:

   ```msgraph-interactive
   GET https://graph.microsoft.com/beta/admin/windows/updates/deploymentAudiences/d39ad1ce-0123-4567-89ab-cdef01234567/members
   ```

## Delete a deployment

To stop an expedited deployment, DELETE the deployment. Deleting the deployment will prevent the content from being offered to devices if they haven't already received it. To resume offering the content, a new approval will need to be created.


The following example deletes the deployment with a **Deployment ID** of `de910e12-3456-7890-abcd-ef1234567890`:

```msgraph-interactive
DELETE https://graph.microsoft.com/beta/admin/windows/updates/deployments/de910e12-3456-7890-abcd-ef1234567890
```


<!--Using include for Update Health Tools log location-->
[!INCLUDE [Windows Update for Business deployment service permissions using Graph Explorer](./includes/wufb-deployment-update-health-tools-logs.md)]