# Hospital Admin Insights from Oracle Analytics Cloud
## Introduction
Up to this point in the course, you have worked with and manipulated all kinds of hospital and patient data. Now, we will perform some data analysis on the patient data gathered into the Autonomous Data Warehouse. In this lab you will learn how to visualization and sharing of data insights through Oracle Analytics Cloud.

### Objectives

In this lab, you will:
* Create an Oracle Analytics Cloud Instance
* Connect to an Autonomous Data Warehouse data source
* Visualize Hospital Statistics


## Pre-requisites 

* You must have an **IDCS user** set up. If you have completed the "Introduction to OCI" Lab, you already have an IDCS user.
* We recommend you install and use a **Google Chrome** or **Firefox** browser for this exercise


## Task 1: Provision an Oracle Analytics Cloud (OAC) Instance & Enable Auto-Insights

1. Navigate to the Oracle Cloud Infrastructure Console accessing from **Oracle Home Page** ([oracle.com](oracle.com)). Click on **View Account** and **Sign in to Cloud**.

    ![View Accounts Sign In](./images/Task-1/0-oracle-com.png)

2. Enter your **Cloud Account Name** (Tenant Name), hit **Next**, and sign in with your cloud login.
    ![Enter Cloud Account](./images/Task-1/1-sign-on.png)

    **Verify that you are logged in as an OCI user** 

    Check that your username is shown as:

    -  oracleidentitycloudservice/&lt;your username&gt;
    
    Then you are **signed in** as a **Single Sign-on** user.

    ![OCI User](./images/Task-1/1-oci-user.png)

    If your user does not contain the identity provider (**oracleidentitycloudservice**) then you are logged in as a **Oracle Cloud Infrastructure** user. In this case, please logout and re-authenticate using **Single Sign On** instead.

    ![Oracle Console SignIn](./images/Task-1/1-console-signin.png)
    To be capable of using **Oracle Analytics Cloud** we need to Sign-On as a **Single Sign-On** (SSO) user.

    ![Oracle Console SSO](./images/Task-1/1-sso.png)

    *For more information about federated users, see [User Provisioning for Federated Users](https://docs.cloud.oracle.com/en-us/iaas/Content/Identity/Tasks/usingscim.htm).*

3. From the **Home Console Page**, navigate to **Analytics & AI**  and select **Analytics Cloud**. This is how you will always navigate to your list of Analytics Cloud instances.

    ![Oracle Cloud Console](./images/Task-1/1-console.png)

    ![Oracle Analytics Console](./images/Task-1/1-console-menu.png)

    > **Note**: You must be connected as a **Single Sign On** (**Federated user**) user to a tenancy, which has available cloud credits to see this menu item. Local OCI users are not able to do this.

4. Select **Create Instance**.

    ![OAC Create Instance](./images/Task-1/3-create-instance.png)

    Complete the form using the following information:

    - **Instance Name**: `WORKSHOPADWOAC`
        ```
        <copy>WORKSHOPADWOAC</copy>
        ```
    - **Compartment**: Select a valid compartment in your tenancy

    - **Description**: &lt;Optional&gt;
        ```
        <copy>Analytics Instance for the cloud</copy>
        ```
    - **Feature Set**: Enterprise Analytics (important)
    
    - **Capacity**: 1 - Non Production
    
    - **License Type**: License Included "Subscribe to a new Analytics Cloud software > license and the Analytics Cloud." (You will use this service as part of the free Oracle Cloud trial that you requested for this workshop).

    - **Edition**: Enterprise Edition "Deploy an instance with enterprise modeling, reporting, and data visualization"

    ![Oracle Analytics Console](./images/Task-1/3-edition.png)


5. Select **Create**.

6. The Analytics instance page will be displayed with a status of **CREATING**.

     > **Note**: Provisioning an Oracle Analytics Cloud instance can take over **40 minutes**.

    ![OAC Instance Creating](./images/Task-1/5-creating.png)

7. Once the Analytics instance page displays a status of **ACTIVE** (green), select the **Analytics Home Page**. 

    ![OAC Instance Active](./images/Task-1/6-active.png)

    The home page of your new analytics instance will open in a new tab.

     ![OAC Homepage](./images/Task-1/6-analytics-home-menu.png)

8. To ensure that **Auto Insights** is enabled, select the hamburger menu and navigate to the **Console**.

     ![OAC Homepage](./images/Task-1/7-home-menu.png)

9. From this page you can do anythign from *manage users*, to *create backups*, to *schedule emails* containing analytics reports. For our purposes, we will choose the **System Settings** page.

     ![OAC Console](./images/Task-1/8-oac-console.png)

10. Select **Performance and Compatibility** from the outline meny, and confirm that **Auto Insights** is **ON** ![ON](./images/on.png)

    ![Auto Insights](./images/Task-1/9-auto-insights.png)

11. Use the **back arrow** and **menu** to navigate back to the OAC Home Page for Task 2

    ![Home](./images/Task-1/10-home.png)

## Task 2: Download Secure Wallet from Autonomous Data Warehouse (ADW)

In this task, you will navigate to the Autonomous Data Warehouse (ADW) cloud console, and download the credential wallet file so we can use it to connect to your data from Oracle Analytics Cloud (OAC).

> **Note**: If you have already downloaded the wallet file from the database cloud console (of format 'Wallet_dbname.zip'), you may skip to the next task.

1. Navigate to the **Oracle cloud console** at cloud.oracle.com. If you are not already signed in, you may log in again. 

    ![Cloud Console](images/Task-2/1-console.png)

2. Use the **hamburger menu** to navigate to **Oracle Database** > **Autonomous Data Warehouse** inventory list

    ![Menu Select ADW](images/Task-2/2-menu.png)

3. You may not see a database list right away. First, select the **Compartment** where your pre-existing database lives, and then **select the database name** (a blue link). This will take you to the database console for your Autonomous Data Warehouse

    ![Choose Compartment and DB](images/Task-2/3-compartment-db.png)

4. Once on the ADW console, select the **DB Connection** button

    ![DB Connection](images/Task-2/4-db-connection.png)

5. Ensure that **Instance Wallet** is the Wallet type, and select **Download Wallet**

    ![Download Wallet](images/Task-2/5-download.png)

6. **Enter a password** and **make sure to copy this password to paper or a text editor**, because you will need it at a later step.

    > **Note**: Password must be 8 to 60 characters and contain at least 1 alphabetic and 1 numeric character.

    ![Enter Password](images/Task-2/6-password.png)

7. **(Optional)** Save your wallet file from your downloads into a more relevant file on your local system for later reference.


## Task 3: Connect OAC to Autonomous Data Warehouse (ADW)

1. Navigate to the **Analytics Home Page** and select **Create**

    ![OAC Home](./images/Task-2/1-analytics-home.png)

2. We will be creating a new **Connection** 

    ![Create Connection](./images/Task-2/2-create-connection.png)

3. There are many connection types available to OAC, but today we will connect to the **Oracle Autonomous Data Warehouse**.

    ![Select ADW](./images/Task-2/3-select-adw.png)

4. Enter the database details as below

    ![Select ADW](./images/Task-2/4-db-detail.png)

    - **Connection Name**: `Hospital Database`
        ```
        <copy>Hospital Database</copy>
        ```

    - **Description**: &lt;Optional&gt;
       
    - **Client Credentials**: Click `Select` and Choose from File your Database Wallet, as gathered in Step 1 of this Task.
    - **Feature Set**: Enterprise Analytics (important)
    
    - **Capacity**: 1 - Non Production
    
    - **License Type**: License Included "Subscribe to a new Analytics Cloud software > license and the Analytics Cloud." (You will use this service as part of the free Oracle Cloud trial that you requested for this workshop).

    - **Edition**: Enterprise Edition "Deploy an instance with enterprise modeling, reporting, and data visualization"

5. 

## Task 4: Create Data Mapping Between ADW Tables in OAC


## Task 5: Create a Workbook in OAC
## Task 6: Explain & Language Narrative Features for quick insights
## Task 7: Table & Map Visualization
## Task 8: Create dashboards for Patients and Labs Analysis
## Task 9: Use Data Actions to Navigate and Filter

## Task Example: Create a Digital Assistant Instance and Import the Skill
1. Log in to the Oracle Cloud at cloud.oracle.com. Cloud Account Name is howarduniversity. Click “Next”.
2. Click on “Direct Sign-In” and enter your Cloud Account email and password.
3. Once you are logged in, you are taken to the cloud services dashboard where you can see all the services available to you. Click the navigation menu in the upper left to show top level navigation choices.
4. Click **Digital Assistant**

  ![](images/odanav.png " ")

5. Select the CareClinics compartment and create a Digital Assistant instance. 

  ![](images/1-createoda.png " ")

6. After the instance is successfully created. Click on the Service Console to open the ODA console. 

  ![](images/1-servconsole.png " ")

7. Now, select the navigation on the top left menu, select Skills under Development and import the Skill which you downloaded.

  ![](images/1-imp.png " ")
```
    <copy>
    find a doc
    help from doctor
    how can I find the doctor?
    looking for a doctor
    need doctor assistance
    Where is the doc?
    </copy>
    ```
