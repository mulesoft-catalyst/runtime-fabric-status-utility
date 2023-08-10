Table of Contents
=================

   * [Runtime Fabric Status Utility](#runtime-fabric-status-utility)
      * [Features](#features)
      * [Utility Details](#utility-details)
      * [Requirements](#requirements)
      * [Steps to configure and test the utility](#steps-to-configure-and-test-the-utility)
      * [Steps to configure the Utility as a Functional Monitor](#steps-to-configure-the-utility-as-a-functional-monitor)
      * [Considerations](#considerations)

# Runtime Fabric Status Utility

Runtime Fabric Status Utility is a MuleSoft application intended to monitor the health of all the Fabrics configured within a MuleSoft Business Group. 

## Features
* This utility has the capability to share the Fabric details in response if they are in any of below inactive statuses. 
    * Disconnected
    * Degraded
* Functional monitoring can be configured for this utility to receive the alerts proactively. The monitor responds with the details of each fabric that is in Inactive status. Application logs can be checked for the same as well.

## Utility Details
This utility is a MuleSoft API with an end-point to retrieve the status of all the fabrics, and respond with details of the fabrics in inactive state. The response can be one of the following.
* Success Response - No Fabrics configured
* Success Response - All Fabrics are in Active State
* Error Response - Some Fabrics are in Inactive State(Disconnected/Degraded) with Response code as 503

## Requirements
* Mule Runtime 4.4.0 or above
* Deployment models supported: CloudHub, Runtime Fabric. (This utility has been tested in CloudHub)
* Anypoint Platform credentials - 
    * A Connected App (client credentials) with the “Read Runtime Fabrics" scope for the appropriate business groups

## Steps to configure and test the utility
* Clone or download the project from GitHub ```git clone git@github.com:mulesoft-catalyst/runtime-fabric-status-utility.git```
* Edit below properties
    ```yaml
    platform:
      orgid: "<orgId>"
    connectedapp:
      client_id: "<client_id>"
      client_secret: "<client_secret>"
    ```
* Default configurations are defined in ``` /src/main/resources/properties.yaml ```
* Make sure to encrypt all sensitive data using the Secure Properties Module: https://docs.mulesoft.com/mule-runtime/4.4/secure-configuration-properties.
* Run the application and test it locally - go to your browser/postman and execute http://localhost:8081/fabrics/list.
* Once the application is deployed in your deployment environment, execute ```http://<appurl>/fabrics/list```. Update ```<appurl>```appropriately.

## Steps to configure the Utility as a Functional Monitor
Create a monitor in Functional monitoring of desired schedule and notification methods with below configurations.
* Method: GET
* Endpoint URL: ``` http://<appurl>/fabrics/list```. Update ```<appurl>``` appropriately.
* Assertions: Status Code Must equal 200

If any of the Fabrics are Inactive, then the Assertion fails and an alert will be triggered with the configured notification methods. You can view the Execution detail in monitor history. The response code will be “503 Service Unavailable” and the response body will be as per below sample.
```json
{
  "message": "ALERT!!! Some fabrics are in either Degraded or Disconnected State!",
  "messageText": [
    {
      "fabricId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "fabricName": "rtf1",
      "fabricStatus": "Disconnected"
    },
    {
      "fabricId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "fabricName": "rtf2",
      "fabricStatus": "Disconnected"
    },
    {
      "fabricId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "fabricName": "rtf3",
      "fabricStatus": "Degraded"
    }
  ],
  "statusCode": 503
}
```

## Considerations
1. This utility can be deployed in CloudHub or Runtime Fabric. However, if the Runtime Fabric itself gets into an inactive state, then the utility will be inaccessible by the functional monitor. 
2. This utility currently exposes HTTP endpoint to get the status of Fabrics. Customize this app and update to https if needed.

## Platform APIs
Below are the Platform APIs used to build this utility. Refer to the [developer portal](https://anypoint.mulesoft.com/exchange/portals/anypoint-platform/) to view all the differnt platform APIs.
1. ```https://anypoint.mulesoft.com//accounts/api/v2/oauth2/token```
2. ```https://anypoint.mulesoft.com/runtimefabric/api/organizations/{orgId}/fabrics```

*This is an UNLICENSED software, please review the considerations. If you need assistance on extending this application, contact your MuleSoft Customer Success representative or MuleSoft Professional Services. Alternatively, you can customize this application as per your requirements.*
