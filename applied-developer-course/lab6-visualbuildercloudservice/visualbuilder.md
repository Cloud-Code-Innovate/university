<!-- Updated June 15, 2022 -->

# Provision Visual Builder Cloud Service Instance

## Introduction

This lab walks you through the steps to quickly provision an Visual Builder Cloud Service on Oracle Cloud. In this lab we will create Mobile application which will take photo of insurance ID card and then store it in Object Storage. Further we will also use Vision service to get required detail from the image that user took and store it in Autonomous Database.

Estimated lab time: 2 hours

### Objectives

- Provision a new Visual Builder Cloud Service Instance
- Learn how to connect to your new autonomous database using ORDS and Vision Service.

### Prerequisites

- This lab requires completion of the **Get Started** section in the Contents menu on the left.

## Task 1: Create Visual Builder Cloud Service Instance

1. Log in to the Oracle Cloud at cloud.oracle.com. Cloud Account Name is howarduniversity. Click “Next”.
2. Click on “Direct Sign-In” and enter your Cloud Account email and password.
3. Once you are logged in, you are taken to the cloud services dashboard where you can see all the services available to you. Click the navigation menu in the upper left to show top level navigation choices.
4. Click **Developer Services --> Visual Builder**.

   ![](images/task1/oci-menu-vbcs.png " ")

5. Select the **CareClinics** compartment and then click **Create Instance**.

   ![](images/task1/compartment.png " ")

6. Type in the name for your VBCS instance. For example: **CareClinicVB**. Then click **Create Visual Builder Instance**.

   ![](images/task1/create-vbcs-instance.png " ")

7. It will take a couple of minutes to create the instance. Once the status changes to active, click on the **Service Homepage** button.

   ![](images/task1/vbcs-instance-details.png " ")

8. This will take you to the landing page for Visual builder cloud service. Then click on **New Application** to create an application.

   ![](images/task1/vbcs-landing-page.png " ")

9. Enter an application name and click **Finish**. This will create your application and take you to the application builder landing page.

   ![](images/task1/vbcs-create-application.png " ")

## Task 2: Configure Service Connections

In this task, we will create 4 service connections. Prior to creating service connections to OCI services, you first have to get details to generate OCI signature like tenancy OCID, user OCID, fingerprint and key.

### Service Connection Pre-requisite: Build OCI Signature

1. Log in to the Oracle Cloud at cloud.oracle.com. Cloud Account Name is howarduniversity. Click “Next”.
2. Click on “Direct Sign-In” and enter your Cloud Account email and password.
3. Once you are logged in, click on the avatar icon on the top right corner.
   ![](images/task2/oci-user-icon.png " ")

4. Click on **API Keys** and then **Add API Key**.
   ![](images/task2/oci-user-api-keys.png " ")

5. Click on **Generate API Key Pair**.
6. Download the keys to your system and store them at a secure location.
7. Click Add.
   ![](images/task2/oci-user-api-keys-generate.png " ")

8. Copy the Configuration File Preview and paste this in a notepad text file. We will use this while adding a service connection to OCI services.
   ![](images/task2/oci-signature-config-file-overview.png " ")

### Service Connection 1: Object Storage

This is a service connection to the Object Storage Service. We built this bucket in the "Introduction to OCI" workshop. We will need the "namespaceName" & "bucketName" from that lab for later portions of this lab.

Go to this page to know more about Object Storage REST endpoints [Object Storage API](https://docs.oracle.com/en-us/iaas/api/#/en/objectstorage/20160918/Object/PutObject)

We will use following API to upload Object to Object Storage.

```copy
https://objectstorage.us-{region}-1.oraclecloud.com/n/{namespaceName}/b/{bucketName}/o/{objectName}
```

Note: In above API we will replace region, namespaceName, bucketName and objectName

1. Go back to VBCS application. Click on **services** in left panel and then click on **plus sign** to add new service connection

   ![](images/task2/vbcs-service-connections.png " ")

2. Select **Define by Endpoint** to add REST endpoints
   ![](images/task2/vbcs-service-connections-endpoint.png " ")

3. Enter the Object Storage URL. Method will be **PUT** since we will upload image to Object Storage. Click **Next**
   ![](images/task2/vbcs-service-connections-configuration.png " ")

4. By default you will be in **Overview** tab. Change **Service Name**
   ![](images/task2/vbcs-service-connections-name.png " ")

5. Click on **Server**, under Server Variables change region variable to your region, here we are using Ashburn region and we will change the authentication to `Oracle Cloud Infrastructire API signature 1.0`
   ![](images/task2/vbcs-service-connections-server-config.png " ")

6. Click on **pencil icon** to add API key ( < tenancyocid >/< userocid >/< fingerprint >) and Private Key. Click **Save**
   ![](images/task2/vbcs-service-connections-credentials.png " ")

7. Click on **Request > Body** and change Media Type to `application/octet-stream`
   ![](images/task2/vbcs-service-connections-body.png " ")

### Service Connection 2: ORDS Autonomous Database

This is a service connection to the Autonomous Database. We built this API in the "Oracle Apex and ADW" workshop under Task 5 (REST Apis for POST - Patient_Insurance and GET - Patient tables). No authentication for these APIs.

Repeat the above process to create another service connection to fetch user information.

1. Here we will have get method and hint will be get one. Click **Next**.

   ![](images/task2/vbcs-service-connections-2a-config.png " ")

2. Rename the service if you want and go to test, and enter id value and click on send request. Once we get 200 response, click **save as example response** and then click **Create**.

   ![](images/task2/vbcs-service-connections-2a-test.png " ")

3. We need to add another endpoint under the same server to create an entry in the ATP DB for the patient's insurance details. Therefore, click **+Endpoint**.

   ![](images/task2/vbcs-service-connections-2b-add.png " ")

4. This time the method is **POST**, url is as per your configuration of the ATP ORDS Endpoint and action hint is **CREATE**. Then click **Save**.

   ![](images/task2/vbcs-service-connections-2b-configure.png " ")

### Service Connection 3: Vision Service

This is a service connection to the OCI Vision Service. The APIs are available out of the box. OCI Signature is the authentication scheme for these APIs (similar to what we did in Service Connection 1).

Go to this page to know more about OCI Vision REST endpoints [OCI Vision APIs](https://docs.oracle.com/en-us/iaas/api/#/en/vision/20220125/AnalyzeDocumentResult/AnalyzeDocument)

We will use following API to extract text from the insurance cards.

```
<copy>
https://vision.aiservice.us-ashburn-1.oci.oraclecloud.com/20220125/actions/analyzeDocument
</copy>
```

1. Repeat Steps **1-2** from the "Service Connection 1: Object Storage". Enter the Vision Service URL. Method will be **POST**.

2. Repeat Steps **4-6 for authentication** from the "Service Connection 1: Object Storage". Click on **Request > Body** and paste the following:

   ```
   <copy>
   {
      "features" : [ {
      "featureType" : "TEXT_DETECTION"
   } ],
   "language" : "ENG",
         document: {
         source: "OBJECT_STORAGE",
         namespaceName: "EXAMPLE-namespaceName-Value",
         bucketName: "EXAMPLE-bucketName-Value",
         objectName: "EXAMPLE-objectName-Value"
         }
   }
   </copy>
   ```

Note: Replace the values of namespaceName, bucketName and objectName in above upload.

![](images/task2/vbcs-service-connections-3-body.png " ")

## Task 3: Create Mobile Application

In this task we will now actually create mobile application that end users can interact with.

### Page 1: main-start

1. Click on **Mobile Applications** in left panel and then click on **plus sign**.
   ![](images/task3/vbcs-create-mobile-app.png " ")

2. Give name to application, select **none** as Navigation Style and click **Next**.
   ![](images/task3/mobile-app-name.png " ")

3. Click **Custom** as template and click **Create**.
   ![](images/task3/mobile-app-template.png " ")

4. Once page is loaded you can see the structure in left panel, by default main-start is landing page. In canvas you can see components where we can filter different components and drag and drop in our canvas. Also elements in canvas can be seen under structure.

   ![](images/task3/canvas-overview.png " ")

5. Search for input under components and drag and drop to **Input Text** to canvas
   ![](images/task3/add-input-compnent.png " ")

6. On right hand side under properties, rename **Label hint** to Enter Id and check the required field. Also add class of `oj-sm-padding-5x` which make it look bit nicer.
   ![](images/task3/input-label-hint.png " ")
   ![](images/task3/input-class.png " ")

7. Repeat the same process to add **Button**.

8. Under structure, click on **Flex Container** and then under properties click on **Center** in Justify and again **Center** in Align
   ![](images/task3/entry-page-styling.png " ")

9. Here we are creating variable to store the user input. Click on **Input Text** and then under **Properties** go to **Data** and next to value click on **down arrow** and finally click **Create Variable**.

![](images/task3/id-create-variable.png " ")

10. Give variable ID and then click **Create**
    ![](images/task3/id-name.png " ")

11. Click on Button under structure and then under events in Properties click on **New Event** and on ojAction.
    ![](images/task3/login-button.png " ")

12. This will take you to Action page and create new action for button click event. Rename ID to `LoginButtonActionChain` and then drag and drop **Call REST** to canvas
    ![](images/task3/login-action-chain .png " ")

13. Select **Call Rest** and under **Properties** click on **Select** next to Endpoint
    ![](images/task3/login-action-rest-call.png " ")

14. Select getUser endpoint and click **Select**.
    ![](images/task3/select-get-user-rest.png " ")

15. Under Properties now you should see **Input Parameters** and click on **Assign**
    ![](images/task3/configure-id-input-param.png " ")

16. Drag userId from Sources to id under Target.
    ![](images/task3/map-id.png " ")

17. Now if our REST call is success we want to Navigate to other page, fo that drag and drop **Navigate**. Under **Properties** click **Create** next to Page.
    ![](images/task3/login-action-navigate.png " ")

18. Give name to page and select **Custom**, click **Create**.
    ![](images/task3/create-cardImage-page.png " ")

### Page 2: main-cardImage

1. In the left navigation pane, find the newly created page and click on "main-cardimage".
   ![](images/task3/page-2/vbcs-cardImage-page.png " ")

2. In the main-cardpage, go to **types** and click on **+type** and then **From Endpoint**.
   ![](images/task3/page-2/create-type-from-endpoint.png " ")

3. Select the getUser under services.
   ![](images/task3/page-2/select-getUser-rest.png " ")

4. Select attributes `first_name,last_name and patient_id` and then click **Finish**.
   ![](images/task3/page-2/select-id-info.png " ")

5. Then go to **Variables** and click "+Variable", name it **userDetails** and select the type as the one you created in step 4. Then click **Create**. Create another variable **image** of type `Any`.  
   ![](images/task3/page-2/create-var-from-type.png " ")

6. Choose the option **Required** under the input parameters on the right side of the screen.
   ![](images/task3/page-2/var-as-required.png " ")

7. Go back to the **main-start** page and open the action chain that will be called on click of the login button. Click on the **navigate to main-cardimage** activity. You should now see under input parameters the newly created input parameter for main-cardimage page i.e. userDetails. Click on **Assign**.
   ![](images/task3/page-2/login-action-navigate-pass-param.png " ")

8. Assign the userDetails on the right side the following code:

`{ "patient_id": $chain.results.callRestGetPatientId.body.patient_id, "first_name": $chain.results.callRestGetPatientId.body.first_name, "last_name": $chain.results.callRestGetPatientId.body.last_name }`

![](images/task3/page-2/user-details-var-body.png " ")

Then click Save.

_Note: callRestGetPatientId could be different for you if you changed the rest call id step right above the navigate step. The idea is to assign the returned fields from the rest call to your next page's input parameter._

9. Go back to your **main-cardimage** page, drag and drop a form layout to the canvas.
10. Also add **oj-sm-padding-4x-horizontal** class on the right side properties panel.
    ![](images/task3/page-2/carImage-page-styling.png " ")

11. Next drag and drop an "Input Text" inside the form layout and on the right properties panel goto data.

12. For the value field under the data tab, hover over the field and you will see an arrow on the far right. Click on that arrow and select "first_name" under the userDetails variable.
    ![](images/task3/page-2/user-info-input-text.png " ")

13. Repeat step 12 for the "Last Name" field.
    ![](images/task3/page-2/user-info-input-text.png " ")

14. Drag and drop a camera component under the form layout in the canvas. In the properties panel on the right, deselect video as the app will only accept images. Also align it centrally.
    ![](images/task3/page-2/camera-component.png " ")

15. Then in the properties panel, go to **events** tab and click on **+New Event** and select **On Selected Files**. This will create new action chain.
    ![](images/task3/page-2/camera-event.png " ")

16. Drag and drop an **Assign Variables** option to the canvas. Then click **Assign**.
    ![](images/task3/page-2/camera-action-chain.png " ")

17. From the left assign Files.item[0] to the image variable and make sure to select **expression** and click **Save**.
    ![](images/task3/page-2/camera-assign-var.png " ")

18. After the assign variable, drag and drop a **fire notification** activity and in the **properties** panel on the right update the details accordingly.  
    ![](images/task3/page-2/camera-success-notification.png " ")

19. Go back to Page Designer, drag and drop **Button** from components in canvas under Take Photo/Video. Change Label and Align center. Add class `oj-sm-margin-2x-top`
    ![](images/task3/page-2/upload-button.png " ")

20. Click on **Events**, and then click on **New Event -> ojAction**
    ![](images/task3/page-2/upload-button-event.png " ")

21. Drag and drop **Call Rest** in canvas and click on **Select** Endpoint in **properties**.
    ![](images/task3/page-2/upload-rest-action.png " ")

22. Select PUT endpoint for uploading image to Object Storage.
    ![](images/task3/page-2/upload-rest-select-endpoint.png " ")

23. Enter Bucket Name, Namespace in target and for objectName we will use firstname_lastname.jpeg. Click **Save**

    ```
     <copy>
    `${$page.variables.userDetails.first_name}_${$page.variables.userDetails.last_name}.jpeg`
     </copy>
    ```

    ![](images/task3/page-2/upload-rest-assign-var.png " ")
    ![](images/task3/page-2/upload-rest-assign-body.png " ")

24. Drag and drop **Fire Notification**, enter Summary and change display mode to **Transient** and Notification Type to **confirmation**
    ![](images/task3/page-2/upload-rest-success-notification.png " ")

25. Drag and drop Call rest after **Fire Notification**. Select analyzeDocument API and then click on body in Parameters.
    ![](images/task3/page-2/upload-rest-call-ai-service.png " ")

26. Paste the Body as follow, and replace fields like namespace and bucketname with yours. For objectName copy the syntax we used above for uploading image to Object Storage.

```
<copy>
{
	"document": {
		"source": "OBJECT_STORAGE",
		"namespaceName": "namespaceName",
		"bucketName": "bucketName",
		"objectName": `${$page.variables.userDetails.first_name}_${$page.variables.userDetails.last_name}.jpeg`
	},
	"features": [{
		"featureType": "TEXT_DETECTION"
	}]
}
</copy>
```

_Note: Replace namespaceName and bucketName in above payload_

    ![](images/task3/page-2/024.png " ")

27. Drag and drop **Call Function**. Here we will write our own javascript function to parse the output we get after analyzing image.

    ![](images/task3/page-2/upload-call-function.png " ")

28. Click on **JavaScript** Tab and paste the following function inside PageModule class.

    ```
    <copy>
        getDetailsFromCard(arrOfLines) {
        let currLine;
        let returnVar = { "memberId": null, "groupNum": null };

        for (let i = 0; i < arrOfLines.length; i++) {
            currLine = arrOfLines[i].text;

            if (returnVar.memberId != null && returnVar.groupNum != null)
            break;

            if (currLine.toLowerCase().indexOf("id") > -1) {
            if (currLine.indexOf(":") > -1 && currLine.split(":")[1].trim().length > 0) {
                returnVar.memberId = currLine.split(":")[1].trim();
            } else {
                returnVar.memberId = arrOfLines[i + 1].text;
            }
            }

            if (currLine.toLowerCase().indexOf("group") > -1) {
            if (currLine.indexOf(":") > -1 && currLine.split(":")[1].trim().length > 0) {
                returnVar.groupNum = currLine.split(":")[1].trim();
            } else {
                returnVar.groupNum = arrOfLines[i + 1].text;
            }
            }
        }

        return returnVar;
        }
    </copy>
    ```

    ![](images/task3/page-2/upload-js.png " ")

29. Go back to **Actions** page and then click on **Call Function**, under properties select the Function Name that we just added.

    ![](images/task3/page-2/upload-select-js.png " ")

30. We need to pass the output from endpoint to function, click on Assign next to Input Parameters and expand results, look for Lines array and map it to arrOfLines in target

    ![](images/task3/page-2/upload-assign-js-body.png " ")

31. Add another Rest Call and select patient_insurance endpoint to update patient insurance details in DB.

    ![](images/task3/page-2/upload-patient-insurance-rest.png " ")

32. Replace all the fields in body. Member id and Group Number we will get from our JS function and patient id is already there in variable. Construct the URL for image URL
    ![](images/task3/page-2/upload-patient-insurance-rest-body.png " ")

33. Add **Navigate** at the end in Success fork, and select the **main-success** page.
    ![](images/task3/page-2/upload-navigate-to-success.png " ")

### Page 3: main-success

1. Click on Flex Container under Structure and on the right properties panel, select the following:
   ![](images/task3/success-page-styling.png " ")

2. Drag and drop a icon component onto the canvas and click on the icon option in the properties panel.
   ![](images/task3/success-page-icon.png " ")

3. Search for success and select the **Success S** option and then click on **Select**.
   ![](images/task3/select-success-icon.png " ")

4. Add the **oj-icon-color-success** class to the icon.
   ![](images/task3/success-icon-style.png " ")

5. Go to the **All tab** for the icon properties and then search for style. Then add **font-size:100px;**
   ![](images/task3/success-icon-size.png " ")

6. Drag and drop a **Text** component right under the icon component.
7. Change the value to _Success! You can now return to the Chatbot_
   ![](images/task3/success-text.png " ")

## Task 4: Publish Application and test

### Publish the Application

1. Click on the app name, click on **SETTINGS** and then **PWA**. Enable the Progressive Web App option so that VBCS can generate the barcode for you to scan and test.
   ![](images/task4/enable-pwa.png " ")

2. Then click the right hand side top corner menu and click **STAGE**. Then click **STAGE** again.
   ![](images/task4/stage-app.png " ")
   ![](images/task4/stage-app-button.png " ")

3. Then click the right hand side top corner menu and click **PUBLISH**. Then click **PUBLISH** again.
   ![](images/task4/publish-app.png " ")
   ![](images/task4/publish-app-button.png " ")

### Test the application

1. Go to the visual builder top level page where you can see all the applications. Click on **LIVE** and then the app name. This will open another window.

   ![](images/task4/live-app-url.png " ")

2. Here on the right side of the screen you can see the barcode to scan to test this on your own mobile device.

   ![](images/task4/app-qr-code.png " ")

3. Enter the ID, and then click on **LOGIN**.

   ![](images/task4/test-login-page.png " ")

4. Verify the details of the user, click on **Take Photo** and then capture an image of your health insurance card. Now you can either take a picture of your own health insurance card or take a picture of the sample below.

Once that is done, you should see a notification saying that photo was taken successfully.

![](images/task4/HealthCardSample.png " ")

![](images/task4/take-photo-test.png " ")

5. Next click on the upload button and wait for the notifications to pop up, this might take some time depending on network latency as the image is being uploaded to the bucket.

   ![](images/task4/upload-photo-test.png " ")

6. If everything goes smooth, it should navigate you to the success page.

   ![](images/task4/success-page-test.png " ")

## Homework

1. In Task 3 Page 2, we are not updating the **RxBin**, there number is coming in the array of lines returned from the vision service. Figure out a way to parse that and extract the RxBin number. Hint: Go to the JS code and make another condition similar to the one we have for member ID and group number.

2. In Task 3, on click of any button we are not showing a spinner or loader. Figure out a way to show that spinner while the action chains are running and to hide it once the action chain finished running.

3. Build a web application with a table containing patient visit data from the ATP database (Patient_visit table). Refer to [this link](https://www.youtube.com/watch?v=9DNBAh0UTeY&t=941s&ab_channel=OracleDevelopers) to learn how to create a web application.

Step-by-Step Instructions to Complete the Homework Assignment

1. For HW 3, you will be creating a web app but the overall application will stay the same. So click the **plus** on web app and give it a name. Open **main-start** and drag and drop a table component to the page.

   ![](images/homework/main-start-table.png " ")

2. Go to service connections, add a service connection. Similar to Task 2 Service connection 2. Add a **GET** method ORDS endpoint.

   ![](images/homework/vbcs-service-connection-ords-endpoint.png " ")

   ![](images/homework/vbcs-service-connection-ords-endpoint-added.png " ")

3. Go back to **main-start**, click on the **table** and on the right side properties, under **Quick Start** click on **Add Data**.

   ![](images/homework/main-start-table-add-data.png " ")

4. Select the newly added ORDS endpoint, select the columns you want to display on the table.

   ![](images/homework/main-start-table-select-columns.png " ")

5. In the last step, assign **User Email** under the System to the **email** URIParameters.

   ![](images/homework/main-start-table-data-input-to-rest-call.png " ")

6. Click play and test the app. If the patient visits table in ATP contains your email address then this app will pull data and show it on the page.

   ![](images/homework/app-test.png " ")

## Troubleshoot Tips

### Using Dev Tools

You can always use chrome dev tools to debug issues if you get stuck. The network tab and console are always useful to see what's going on behind the app. There is blog explaining in more detail. [How To Debug VBCS Applications](https://blogs.oracle.com/vbcs/post/debugging-and-troubleshooting-visual-builder-logic)

    If you are having problems with any of the labs, please visit the Need Help? tab.
