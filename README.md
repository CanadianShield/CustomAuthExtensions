# CustomAuthExtensions
Example to use Custom authentication extensions with Logic Apps


# 1️⃣ Create the logic app

You can use a consumption logic app as long as the information you want to retrieve and the data manipulation you want to do can be done using the Logic App connectors.

Important points:
- Pick the HTTP Trigger also known as "When a HTTP trigger is received"
- Send and HTTP 200 response with the header `Content-Type` set to `application/json`.
- Here is an example that returns an hardcoded date of birth:
<img width="305" alt="image" src="https://github.com/CanadianShield/CustomAuthExtensions/assets/22434561/c0cc7faa-f2bd-41f8-bd7a-fc59dcece346">

Here is the code used in the Response step:
```
{
  "data": {
    "@@odata.type": "microsoft.graph.onTokenIssuanceStartResponseData",
    "actions": [
      {
        "@@odata.type": "microsoft.graph.tokenIssuanceStart.provideClaimsForToken",
        "claims": {
          "DateOfBirth": "01/01/2000"
        }
      }
    ]
  }
}
```

Note the double `@` signs. Save the logic app and got back to the designer view. Then select the trigger and you should be able to see the `HTTP POST URL` value.

<img width="301" alt="image" src="https://github.com/CanadianShield/CustomAuthExtensions/assets/22434561/2f6c42da-dd64-4ed0-84aa-4d4711d7162f">

Something like this:

`https://prod-77.eastus.logic.azure.com:443/workflows/1d48cedefa5d421cb9afa523034ad06b/triggers/manual/paths/invoke?api-version=2016-10-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=EwyuG3f0Fg_txAw0loukj5kg4BneIGk-46HoBQZpQvw`

Remove everything inlcuding and after the `sv` query string. You should be left with:

`https://prod-77.eastus.logic.azure.com:443/workflows/1d48cedefa5d421cb9afa523034ad06b/triggers/manual/paths/invoke?api-version=2016-10-01&sp=%2Ftriggers%2Fmanual%2Frun`

That is the string we will need in your extension. We'll get back to the logic app later.

# 2️⃣ Create the custom extension

In the Azure AD portal, go to the **Enterprise Applications** blade, then select **Custom authentication extensions**. Click Create a custom extension. 
