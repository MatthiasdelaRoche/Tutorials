---
title: Use Free Tier to Create a Service Instance for Data Attribute Recommendation
description: Create a service instance and the associated service key for Data Attribute Recommendation using the free tier service plan.
auto_validation: true
time: 15
tags: [tutorial>beginner, topic>machine-learning, topic>artificial-intelligence, topic>cloud, software-product>sap-business-technology-platform, software-product>sap-ai-business-services, software-product>data-attribute-recommendation, tutorial>free-tier]
primary_tag: topic>machine-learning
author_name: Juliana Morais
author_profile: https://github.com/Juliana-Morais
---

## Prerequisites
- You have created an account on SAP BTP to try out free tier service plans: [Get an Account on SAP BTP to Try Out Free Tier Service Plans](btp-free-tier-account)
- You are entitled to use the Data Attribute Recommendation service: [Manage Entitlements Using the Cockpit](btp-cockpit-entitlements)

## Details
### You will learn
  - How to check your Data Attribute Recommendation entitlements
  - How to create a service instance of Data Attribute Recommendation
  - How to create a service key for your service instance

---

[ACCORDION-BEGIN [Step 1: ](Access the SAP BTP cockpit)]

1. Open the [SAP BTP cockpit](https://account.hana.ondemand.com/cockpit#/home/allaccounts).

2. Access your global account.

    !![Access Subaccount](global-account.png)

3. Click the tile that represents the subaccount that you'll use throughout these tutorials.

!![Access Subaccount](access-subaccount.png)

[DONE]
[ACCORDION-END]


[ACCORDION-BEGIN [Step 2: ](Check your entitlements)]

To use Data Attribute Recommendation, you need to make sure that your account is properly configured.

1. On the navigation side bar, click **Entitlements** to see a list of all eligible services. You are entitled to use every service in this list according to the assigned service plan.

2. Search for `Data Attribute Recommendation`. ***If you find the service in the list, you are entitled to use it. Now you can set this step to **Done** and proceed with Step 3.***

!![Check Entitlements](check-entitlements.png)

***ONLY if you DO NOT find the service in your list, proceed as follows:***

  1.  Click **Configure Entitlements**.

    !![Configure Entitlements](configure-entitlements.png)

  2.  Click **Add Service Plans**.

    !![Add Service Plans](add-service-plans.png)

  3.  In the dialog, select `Data Attribute Recommendation` and choose the `free` service plan. Click **Add 1 Service Plan**.

    >You can also perform this tutorial series using the `standard` service plan. For that, choose the `standard` plan in this step (instead of free). For more information on the service plans available for Data Attribute Recommendation and their usage details, see [Service Plans](https://help.sap.com/docs/Data_Attribute_Recommendation/105bcfd88921418e8c29b24a7a402ec3/e28c50aa9b5b41de8ce8d6d46f2a5aac.html).

    !![Add Service Plan](add-service-plan.png)

  4.  Click **Save** to save your entitlement changes.

    !![Save Entitlements](save-entitlements.png)

You are now entitled to use Data Attribute Recommendation and create instances of the service.

[DONE]
[ACCORDION-END]


[ACCORDION-BEGIN [Step 3: ](Access service via Service Marketplace)]

The Service Marketplace is where you find all the services available on SAP BTP.

1.  To access it, click **Service Marketplace** on the navigation side bar.

    !![Access Marketplace](access-marketplace.png)

2.  Next, search for **Data Attribute Recommendation** and click the tile to access the service.

    !![Access Service](access-service.png)

[DONE]
[ACCORDION-END]


[ACCORDION-BEGIN [Step 4: ](Create service instance)]

Next, you will create an instance of the Data Attribute Recommendation service.

1. Click **Create** to start the service instance creation dialog.

    !![Create Instance](create-instance.png)

2. In the dialog, choose the `free` plan. Enter a name for your new instance, for example, `dar-instance` and click **Create**.

    >Choose `standard` in this step (instead of free) if you're using the `standard` plan to perform this tutorial series.

    !![Create Instance](create-instance-dialog.png)

3. In the following dialog, click on **View Instance** to navigate to the list of your service instances.

    !![View Instances](view-instances.png)

You have successfully created a service instance for Data Attribute Recommendation.

[DONE]
[ACCORDION-END]


[ACCORDION-BEGIN [Step 5: ](Create service key)]

You are now able to create a service key for your new service instance. Service keys are used to generate credentials to enable apps to access and communicate with the service instance.

1. Click the dots to open the menu and select **Create Service Key**.

    !![Service Key](create-service-key.png)

2. In the dialog, enter `dar-service-key` as the name of your service key. Click **Create** to create the service key.

    !![Create Service Key](create-service-key-name.png)

You have successfully created a service key for your service instance. You can now view the service key in the browser or download it.

!![View Service Key](view-service-key.png)

[VALIDATE_1]
[ACCORDION-END]
