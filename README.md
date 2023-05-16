This is an example of how to use Custom authentication extensions with Logic Apps.

## 1️⃣ Create the logic app

You can use a consumption logic app as long as the information you want to retrieve and the data manipulation you want to do can be done using the Logic App connectors. Let's call it **Get-DOB** just for the sake of this example.

Important points:
- Pick the HTTP Trigger also known as "When a HTTP trigger is received"
- Send and HTTP 200 response with the header `Content-Type` set to `application/json`.
- Here is an example that returns an hardcoded date of birth:
<img width="305" alt="image" src="/pictures/Picture1.png">

Here is the code used in the Response step:
```
{
  "data": {
    "@@odata.type": "microsoft.graph.onTokenIssuanceStartResponseData",
    "actions": [
      {
        "@@odata.type": "microsoft.graph.tokenIssuanceStart.provideClaimsForToken",
        "claims": {
          "DOB": "01/01/2000"
        }
      }
    ]
  }
}
```

Note the double `@` signs. Save the logic app and got back to the designer view. Then select the trigger and you should be able to see the `HTTP POST URL` value.

<img width="301" alt="image" src="/pictures/Picture2.png">

Something like this:

`https://prod-77.eastus.logic.azure.com:443/workflows/1d48cedefa5d421cb9afa523034ad06b/triggers/manual/paths/invoke?api-version=2016-10-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=EwyuG3f0Fg_txAw0loukj5kg4BneIGk-46HoBQZpQvw`

Remove everything including and after the `sv` query string. You should be left with:

`https://prod-77.eastus.logic.azure.com:443/workflows/1d48cedefa5d421cb9afa523034ad06b/triggers/manual/paths/invoke?api-version=2016-10-01&sp=%2Ftriggers%2Fmanual%2Frun`

That is the string we will need in your extension. We'll get back to the logic app later.

## 2️⃣ Create the custom extension

In the Azure AD portal, go to the **Enterprise Applications** blade, then select **Custom authentication extensions**. Click Create a custom extension.  

In the **Endpoint Configuration** tab, enter the name of your API (ie. "Logic App Get-DOB").
Enter the URL of your Logic App trigger (the sanitized one, without the signature information). 

In the **API Authentication**, select **Create new app registration**. Enter the name of the application like "Logic App Get-DOB".

In the **Claims** tab, enter the claim you want to get out of this logic app call, let's call it `DOB` like this:

<img width="422" alt="image" src="/pictures/Picture3.png">

Then review...

<img width="918" alt="image" src="/pictures/Picture4.png">

And create the extension.

## 3️⃣ Consent to the permissions of the application

You can do that right from the creation confirmation page:

<img width="937" alt="image" src="/pictures/Picture5.png">

Also, note the App ID from this page as you'll need it in the next step.

## 4️⃣ Configure the Logic App to use OAuth

By default, the HTTP trigger will expect a signature. In our case, we will not send a signature, but we will obtain a token to use this Logic App (we will be hitting the HTTP trigger URL with a bearer token instead of a signature in the URL).

Go to your Logic App and select the **Authorization** blade and add a policy. Give it the name you'd like, leave it set to AAD and enter the following claims:

- **Issuer**: `https://login.microsoftonline.com/<tenant ID>/v2.0` where `tenant ID` is your Azure AD tenant ID
- **Audience** (standard claim): <the App ID of the application you created in the previous step>
- **azp** (custom claim): `99045fe1-7639-4a75-9d4a-577b6ca3810f` (this is the hardcoded ID of the Custom Authentication Extension ID, it is the same for everyone.

Example:

<img width="581" alt="image" src="/pictures/Picture6.png">

Make sure you don't have a space at the beginning and at the end of the identifiers.
  
## 5️⃣ Issue the claim for your application
  
In this example, I am using the [Claim X-Ray](https://adfshelp.microsoft.com/ClaimsXray/TokenRequest) enterprise application. Open the **Single sign-on** blade and edit the **Attributes & Claims** section. Expand the **Advanced settings** and edit the **Custom claims provider** section. Pick your extension and click **Save**.

<img width="698" alt="image" src="/pictures/Picture7.png">

Your claim is now available for issuance. Still in the **Attributes & Claims** page, click **Add new claim** and pick the DOB claim from the **Source attribute** drop down:
  
<img width="664" alt="image" src="/pictures/Picture8.png">

Save all that and give it a try! Still from the **Single sign-on** scroll all the way down to the **Test** button:

<img width="643" alt="image" src="/pictures/Picture9.png">

Select **Sign in as current user** and check if you have the claim in the output:

  <img width="415" alt="image" src="/pictures/Picture10.png">
