---
# required metadata

title: Investigate activities using the API - Cloud App Security | Microsoft Docs
description: This article provides information on how to use the API to investigate user activity in Cloud App Security.
keywords:
author: rkarlin
ms.author: rkarlin
manager: rkarlin
ms.date: 03/26/2019
ms.topic: conceptual
ms.collection: M365-security-compliance
ms.prod:
ms.service: cloud-app-security
ms.technology:
ms.assetid: 0f2f971d-10e3-496d-8004-96d9fad71cae

# optional metadata

#ROBOTS:
#audience:
#ms.devlang:
ms.reviewer: reutam
ms.suite: ems
#ms.tgt_pltfrm:
ms.custom: seodec18

---
# Investigate activities using the API

*Applies to: Microsoft Cloud App Security*

Microsoft Cloud App Security provides you with a fully supported REST API to enable you to programmatically interact with the service.

You can use the Microsoft Cloud App Security APIs to investigate the activities performed by your users across connected cloud apps. 

The Cloud App Security activities API mode is optimized for scanning and retrieval of large quantities of data (over 5,000 activities). The API scan queries the activity data repeatedly until all the results have been scanned. 

> [!NOTE] 
> For large quantities of activities and large scale deployments, we recommended that you use the [SIEM agent](siem.md) for activity scanning.

**To use the activity scan API:**

1. Run the query on your data.
1. If there are more records than could be listed in a single scan, you will get a return command with `nextQueryFilters` that you should run. You will get this command each time you scan until the query has returned all the results.
 
 
**Request body parameters**:
- “filters”: Filter objects with all the search filters for the request, see [Activity filters](activity-filters.md) for more information. To avoid having your requests be throttled, make sure to include a limitation on your query, for example, query the last day’s activities, or filter for a particular app.
- “isScan”: Boolean. Enables the scanning mode.
- “sortDirection”: The sorting direction, possible values are “asc” and “desc” 
- “sortField”: Fields used to sort activities. Possible values are: 
    - date - The date when then the activity occurred (this is the default).
    - created - The timestamp when the activity was saved.
- “limit”: Integer. In scan mode, between 500 and 5000 (defaults to 500). Controls the number of iterations used for scanning all the data. 

**Response parameters**:
- “data”: the returned data. Will contain up to “limit” number of records each iteration. If there are more records to be pulled (hasNext=true), the last few records will be dropped to ensure that all data is listed only once.
- “hasNext”: Boolean. Denotes whether another iteration on the data is needed.
- “nextQueryFilters”: If another iteration is needed, it contains the consecutive JSON query to be run. Use this as the “filters” parameter in the next request.

The following Python example gets all the activities from the past day from Exchange Online.

      import requests
      import json
      ACTIVITIES_URL = 'https://<your_tenant>.portal.cloudappsecurity.com/api/v1/activities/'
    
      your_token = '<your_token>'
         headers = {
         'Authorization': 'Token {}'.format(your_token),
        }
    
        filters = {
          # optionally, edit to match your filters
          'date': {'gte_ndays': 1},
          'service': {'eq': [20893]}
        }
        request_data = {
         'filters': filters,
         'isScan': True
        }
        
        records = []
        has_next = True
        while has_next:
            content = json.loads(requests.post(ACTIVITIES_URL, json=request_data, headers=headers).content)
            response_data = content.get('data', [])
            records += response_data
            print('Got {} more records'.format(len(response_data)))
            has_next = content.get('hasNext', False)
            request_data['filters'] = content.get('nextQueryFilters')
        
        print('Got {} records in total'.format(len(records)))
        
 
## Next steps
[Daily activities to protect your cloud environment](daily-activities-to-protect-your-cloud-environment.md)   

[Premier customers can also create a new support request directly in the Premier Portal.](https://premier.microsoft.com/)  
  
