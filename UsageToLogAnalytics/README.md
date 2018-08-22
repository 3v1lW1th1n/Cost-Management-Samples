Moving Usage logs to Log Analytics

Scenario: Customers would like to want their usage data to be moved to
Log Analytics.

Pre-requisites:

-   Visual Studio 2017 (Download from
    [here](https://visualstudio.microsoft.com/vs/whatsnew/).)

-   Azure Functions and WebJobs Tools (Download from
    [here](https://marketplace.visualstudio.com/items?itemName=VisualStudioWebandAzureTools.AzureFunctionsandWebJobsTools))

Step 1: Create a Log Analytics workspace in the subscription you want to
see the usage for

Go to the portal
<https://ms.portal.azure.com/#blade/HubsExtension/Resources/resourceType/Microsoft.OperationalInsights%2Fworkspaces>

![](media/image2.png)

Click Add button and fill the Workspace details.

![](media/image3.png)

2\. Enable SqlAuditLogs on the Sql Server Database on the subscription
you want. You can see that the newly created workspace.

![](media/image4.png)

![](media/image5.png)

Once the sql audit logs are being enabled to go into Log Analytics, lets
work on moving the usage data to Log Analytics.

4\. Get the code from Git Hub and build the application. There are 2
projects in that solution.

Function App, which is a time, triggered and the core dll. This function
app runs every 1 hour. You can change the frequency by changing the cron
expression. This if found in UCDDHourlyToOMS.cs file.

public static void Run(\[TimerTrigger(\"0 0 \*/1 \* \* \*\")\]TimerInfo
myTimer, TraceWriter log)

The Function App UCDDHourly2OMSFuncApp needs the following secrets to be
created in KeyVault.

1.  omsworkspaceid

    i.  The oms workspaceId can be found in the advanced settings of the
        Log Analytics

2.  omsworkspacekey

    ii. This is the Primary key that can be found in the advanced
        settings of Log Analytics

3.  ![](media/image6.png)

4.  Get the Connection String of the storage account where the usage
    data is available.

5.  Open a new browser and go to the KeyVault where you would want to
    store and create the secrets.

6.  Once its created copy the Keyvault url and paste in the
    localsettings.json file in UCDDToOMSFun project.

7.  Now let's publish the Function App to Azure.

8. Open the project in VS2017 and find the UCDDToOMSFunc project.

    ![](media/image7.png)

9. Right click on the project and hit Publish. We first need to create
    a profile which has the subscription details, Resource Group,
    Storage Account and what APP Service Plan you want to use.

   ![](media/image8.png)

10. Click Create new profile

![](media/image9.png)

11. Select Create new Azure Function App

![](media/image10.png)

12. Give an appropriate name and select the subscription where it has to
    run.

13. ![](media/image11.png)

14. Select the right App Service Plan. If your data size compressed in
    around 10 MB (gzip compression), then selecting smaller SKUs is not
    enough for processing. So, pick the size depending on the size of
    usage data.

![](media/image12.png)

15. Hit Create

16. ![](media/image13.png)

17. It starts deploying the host services the Function App needs.

    ![](media/image14.png)

    Once deployment is completed you should be able to see the App
    Service and the FuncApp that got created. The function App is still
    empty, and no functions are deployed. You can search in the
    subscription for resource type "Function Apps"

    ![](media/image15.png)

    Now we need to go the App Settings in the portal for that Function
    App. Whatever settings we have in localsettings.json need to be
    copied to App Settings. Add "KeyVaultURL" setting.

    Next we need to enable Managed Service Identity to give Azure
    Identity to this Func App. Go to Platform Features and find

    ![](media/image16.png)

    ![](media/image17.png)

    Hit Save.

    Now we need to go to the Keyvault where we stored our secrets and
    give this newly created MSI of Func App access to secrets.

    Go to KeyVault -\> Access policies and click Add New

    ![](media/image18.png)

    And in Select Prinicpal search for Function App.

    ![](media/image19.png)

    And select Get and List in Secret permissions.

    ![](media/image20.png)

    And don't forget to save it.

    Now the Function App has access to Keyvault.

    Now let's enable App Insights for logging so that we can see the
    logs. Go to all Resources in azure portal home page and search for
    "Application Insights". Pick the correct the subscription and create
    new

    ![](media/image21.png)

    ![](media/image22.png)

    Once created click on the overview and get the instrumentation key
    and save it somewhere.

    ![](media/image23.png)

    Now we must provide this instrumentation key in the App Settings of
    the FuncApp.

Add this as setting name "APPINSIGHTS\_INSTRUMENTATIONKEY" and
instrumentation key which you copied earlier. And Save the App Settings.
After few seconds you should see Application Insights in Configured
features.

![](media/image24.png)

Now that the Host is setup and Keyvault access is granted lets deploy
the code from VS2017. Make sure you build the project in release mode
and right click on publish and select the profile which we created and
click Publish. You should see the status in Output window

![](media/image25.png)

Now you should see the function under Functions. If you want to see the
logs, go to the Monitor tab and see the runs.

After a successful run go to your log Analytics and under Custom logs
you should see "Usage_CL" table.