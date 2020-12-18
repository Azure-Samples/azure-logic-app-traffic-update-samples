---
page_type: sample
languages:
- azurecli
- json
products:
- azure-logic-apps
- azure-front-door
- azure-app-service
---
# Logic App sample for automatic traffic diversion for Azure App Service

This repository contains Logic App sample code which shows you how to automatically update traffic weights of an Azure Front Door instance when a upgrade notification event was received from the Azure App Service platform. It also contains the set up instructions of the sample app.

## Why and when do you use this?

The Azure App Service is a PaaS solution offering in Azure which constantly updates their platform. Although it provides lots of features with which you can minimize the risk of long downtime, there are still chances of cold starts and app restarts caused by such updates. Also it's still possible that your web app might go down by a platform regression. To avoid such an unexpected downtime, one of the best practices is to deploy your app to multiple regions, set up either Azure Front Door or Traffic Manager and put your apps behind it. This way, you can quickly change the traffic flow in case your app in one region has an issue.

If you want to automate this process, you need:

* An update notification event from the platform.
* Code which can change the traffic weight of your Azure Front Door instance.
* An action triggered by the event and kicks off the code.

The App Service sends notification events to Azure Monitor when an update to scale units in a region is about to start, and when the update is either completed or canceled. It also sends periodic "in progress" reports every 12 hours until the update is finished. You can set up action rules in Azure Monitor to hook up the events and take necessary actions:

* When an update start event in a region is notified, flow all the traffic to the other region.
* When an update complete event is notified, set the original traffic weight back (e.g. even distribution.)
* Optionally, post a Teams message when the traffic weight is changed.

This repository shows how to achieve all the above with Logic Apps.

Note that the update notifications are only available for App Service Environments (ASEs.) Apps hosted on regular App Service scale units aren't supported yet.

## Getting Started

### Prerequisites

* Azure subscription
* App Service Environments (ASEs) which are located in different regions (e.g. East US and West US)

### Quickstart - Part 1: Create web apps and Azure Front Door

*Skip this part if you already have web apps and an Azure Front Door instance.*

1. Open **Azure portal**, sign in with your credentials.
2. Click the big plus sign (+) on the top left corner to create a resource.
3. Choose **Web App**.
4. Choose a subscription that owns your ASEs.
5. Select or create a resource group and name your app. Make sure to choose the correct region.
6. Click **Review + Create**, and click **Create** button. Wait for the creation operation completes.
7. Repeat the same steps to create another web app, except for choosing another region. For example, if you chose *West US* at step 5, then choose *East US* here.
8. Go to Home of the Azure portal.
9. Click the big plus sign (+) to create a resource.
10. type `front door` in the search box, choose **Front Door**.
11. Click **Create**.
12. Choose a resource group and a location. Click **Configuration** button.
13. Click the plus sign at the top left corner of the *Frontends/domains* box. Type a frontend host name (e.g. myfrontdoor.azurefd.net), click **Add**.
14. Click the plus sign at the top left corner of *Backend pools* box. Type a pool name (e.g. `mypools`). Click **Add a backend** in the **BACKENDS** section, add your app you created at step 6. Add the other app you created at step 7 as well. Click **Add**.
15. Click the plus sign at the top left corner of *Routing rules* box. Type a rule name (e.g. `myrule`). Make sure to set *Backend pool* to the backend pool name that you have created at step 14. Click **Add**.
16. Click **Create**. Wait for the creation operation completes.

### Quickstart - Part 2: Deploy a Logic app using Azure CLI and Azure portal

1. Open `templates/parameters.json` with text editor.
2. Customize parameters in the file and save. If you are going to create your Logic app in the same subscription as your Front Door instance, remove `Frontdoors_myfrontdoor_subscription` setting. Similarly, if your Logic app will be created into the same resource group, remove `Frontdoors_myfrontdoor_resourcegroup` setting.
3. Sign in to Azure by running this command:

    ```bash
    az login
    ```

4. Choose a subscription by running this command:

    ```bash
    az account set -s '<SUBSCRIPTION_ID_OR_SUBSCRIPTION_NAME>'
    ```

5. Create a new resource group by running this command:

    ```bash
    az group create -n <RESOURCE_GROUP_NAME>
    ```

6. Execute a deployment by running this command:

    ```bash
    az deployment group create --name MyTestDeployment --resource-group <RESOURCE_GROUP_NAME> --template-file ./templates/template.json --parameters @./templates/parameters.json
    ```

7. Now your Logic app has been created. However you still need to authenticate a connection used between your Logic app and Front Door. Open **Azure portal**, sign in with your credentials.
8. Search your Logic app by typing the name at the top corner search box. Navigate to the app.
9. Check the Status is currently *Unauthorized*. If it's *Connected*, you can skip the following steps.
10. Click **Edit API connection**
11. Click **Authorize** button. Complete the sign in steps.
12. Click **Save**.
13. Go back to the Overview menu item. Make sure the Status is now *Connected*.

*Note: If you have a service principal available, you can use it instead of manual authorization. Refer to [Authenticate Connections](https://docs.microsoft.com/azure/logic-apps/logic-apps-azure-resource-manager-templates-overview#authenticate-connections) for details.*

### Quickstart - Part 3: Set up an alert

1. Open **Azure portal**, sign in with your credentials.
2. Search an icon named **Monitor** and click it. If you cannot see it, click the big right arrow on the right to show **All services**, then search `Monitor`.
3. In the left menu items, click **Alerts**.
4. Click **Service Health**.
5. Click **Add service health alert** at the top center.
6. In the Condition section, choose the subscription that owns your ASEs.
7. At the Service(s) box, choose all items starting with `App Service`:
   * App Service
   * App Service \ Web Apps
   * App Service (Linux)
   * App Service (Linux) \ Web App for Containers
   * App Service (Linux) \ Web App.
8. At the Region(s) box, make sure to check the regions of the ASEs.
9. At the Event type box, check **Planned maintenance**.
10. In the Actions section, click **Add action groups**.
11. Click **Create alert rule**.
12. Select the subscription that you have created your Logic app during the previous part.
13. Choose a resource group and name an action group. Set Display name to something you can easily identify the action (**IMPORTANT**: The display name will be shown in every email/SMS/post of the notifications.)
14. (Optional) If you want to receive text notifications, in the Notifications section, choose **Email/SMS message/Push/Voice** at the Notification type. Then choose output channels you need (For example, Email and SMS.) Put email addresses or phone number as necessary.
15. In the Actions section, choose **Logic App** at the Action type. Put a name into the Name. Select your Logic app.
16. Press **Save changes**. The page will go back to the Rules management page.
17. In the Alert rule details section, set a name.
18. Click **Save**.

## Resources

* [The Ultimate Guide to Running Healthy Apps in the Cloud](https://azure.github.io/AppService/2020/05/15/Robust-Apps-for-the-cloud.html)
* [Azure Logic Apps documentation](https://docs.microsoft.com/azure/logic-apps/)
* [Azure Front Door documentation](https://docs.microsoft.com/azure/frontdoor/)
* [Azure Monitor documentation](https://docs.microsoft.com/azure/azure-monitor/)
* [Authenticate connections](https://docs.microsoft.com/azure/logic-apps/logic-apps-azure-resource-manager-templates-overview#authenticate-connections)
