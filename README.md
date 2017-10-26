# Control-M v9

## Integration Overview

BMC Control-M delivers is business automation solution that simplifies and automates diverse batch application workloads. When events meet predetermined criteria, this integration with xMatters relays critical BMC Control-M insight data to the correct people and systems to help coordinate and resolve incidents faster. xMatters leverages your group on-call schedules and rotations, escalation rules, and user device preferences to quickly engage the right resources with customizable response and collaboration options.

---------

<kbd>
  <img src="https://github.com/xmatters/xMatters-Labs/raw/master/media/disclaimer.png">
</kbd>

---------


## Automate Handoffs accross your toolcahin

Integrating xMatters with your other tools allows you to automatically transfer key BMC Control-M alert data throughout your systems and drive workflows forward. Notifications and collaboration invites embedded with BMC Control-M insights allow your resolution teams to take immediate action.
  * Create a service management ticket with the BMC Control-M insights
  * Invite people across multiple teams to a conference call with context from BMC Control-M
  * Initiate a targeted chat room via Slack, HipChat or Hubot
  * Record chat room activity back into a service management ticket

# Pre-Requisites
* BMC Control-M Workload Automation v9
* BMC CONTROL-M/EM API v9
* xMatters OnDemand
* xMatters integration agent 5.x (5.0 patch 007 or later)

# Files
* [Communication Plan](/components/BMCControlMIntegration%20Comm%20Plan.zip)
* [Integration Service](/components/control-m_v9%20Integration%20Service.zip)
* [Control-M EM API Documentation](/components/CTM_API_9.0.00.200_480104.pdf)
* [Integration Agent](https://support.xmatters.com/hc/en-us/articles/201463419-Integration-Agent-for-xMatters-5-x-xMatters-On-Demand)

# How it works

The Control-M integration leverages shout destinations which submit events to xMatters Integration Agent via the Apclient.bin submission mechanism. Once the Integration Agent receives the events, the event information is POST'd to xMatters via a REST API.

# Installation
## 1. Installation and Configuration

This document provides information about installing the xMatters (IT) for BMC Control-M Workload Automation integration. This document also contains complete instructions on how to configure xMatters, BMC Control-M, and the integration components.

Installing the integration service and updating the integration agent

To configure the integration agent for the BMC Control-M integration, you must copy the integration components into the integration agent; this process is similar to patching the application, where instead of copying files and folders one by one, you copy the contents of a single folder directly into the integration agent folder (<IAHOME>). The folder structure is identical to the existing integration agent installation, so copying the folder's contents automatically installs the required files to their appropriate locations. Copying these files will not overwrite any existing integrations. This integration includes the following components:

* **bin/Controlm-APClient.bat** & **bin/Controlm-APClient-Del.bat**
Batch files targeted by the SHOUT destinations from within Control-M. They take the SHOUT message and call APClient.bin.exe with the correct map-data for either the controlm25 service or the del service, depending on which SHOUT destination was used

* **conf/deduplicator-filter.xml**
The filtering mechanism used to suppress duplicate messages. By default, the filter checks the job_name, status and run_num values; if they are all the same within a two minute window, only the first message will be sent through to xMatters.

* **controlm-config.js:** The main configuration file for the integration; includes API connection information and user information.

* **conf/bmccontrolm.pwd:** Stores the password for the Control-M API user used by the integration service. If you change the name of this file, you must also update the controlm-config.js files to point to the correct password file.


Once you have installed the files and folders, you will need to modify the controlm-config.js and IAConfig.xml files to suit your deployment configuration.

_**Note:** If you have more than one integration agent providing the BMC Control-M service, repeat the following steps for each one._

**To install the integration service:**

1. Copy all of the contents of the /components/integration-agent/ folder from the extracted integration archive to the <IAHOME> folder.

2. Open the IAConfig.xml file found in <IAHOME>/conf and add the following line to the “service-configs” section: <path>controlm25/controlm.xml</path>

3. Open the controlm-config.js file (now located in <IAHOME>/integrationservices/controlm25/ folder, and set the values for the following variables:
   * **CONTROL_M_USER:** The user name to be used for the BMC Control-M API Server; default value is "username".
   * **CONTROL_M_HOST_NAME:** Hostname of the BMC Control-M API Server; default value is "localhost".
   * **CONTROLM_PASSWORD_FILE:** Location of the password file for the API user; default value is "conf/bmccontrolm.pwd".
   * **CTMEMAPI_PROPERTIES_FILE:** Location of the CTMEMAPI_PROPERTIES_FILE; default value is "integrationservices/controlm25/ctmemapi.properties". For more information about this setting, see the following section, "Installing Control M/EM API server properties files"
   * **JACORB_PROPERTIES_FILE:** Location of the JACORB_PROPERTIES_FILE; default value is "integrationservices/controlm25/jacorb.properties". For more information about this setting, see the following section, "Installing Control M/EM API server properties files".
   * **SLEEP_PERIOD_BETWEEN_CALLBACK:** The amount of time (in milliseconds) to sleep between receiving a SHOUT Message and making the callback to the BMC Control-M API for the job details. (Default is 3000, or three seconds.)
   * **DEDUPLICATOR_FILTER:** Name of the deduplicator filter; i.e., the attribute name for the element filter in the deduplicator-filter.xml file. The default value is controlm25.

4. Save and close the file.

5. Open the Controlm-APClient.bat and Controlm-APClient-Del.bat files (now located in `<IAHOME>/bin` ) in a text editor, and change the value for the "IAHOME" variable to the location of the `<IAHOME>/bin` folder on your machine. (The default is `C:/PROGRA~1/xmatters/integrationagent/bin` ).

6. Save and close the files.

7. Restart the integration agent. 

On Windows, the integration agent runs as a Windows Service; on Unix, it runs as a Unix daemon

## 2. Installing Control M/EM API server properties files

### 2.1 EM API Install
This procedure describes how to install Control-M/EM API on Windows and UNIX. 

Due to Java limitations, BMC Software recommends that you do not install Control-M/EM API in a directory with a path that contains spaces or other special characters. If you install Control-M/EM API on an account where an earlier version of Control-M/EM API is installed, see attached Control-M/EM API pdf (on page 19) before continuing. 

**Before you begin**
Ensure the following is installed: 

   * Control-M/EM in your network environment 
   * Java Developer's Kit (JDK) version 1.6.x or later or Java Runtime Environment (JRE) version 1.6.x or later on the computer hosting your project's working directory. `JAVA_HOME` environment variable should point to the JDK/JRE directory/library.

_**Note:** The `JAVA_HOME` environment refers to the directory where the JRE is installed. The JDK contains the JRE, but at a different level in the file hierarchy. For example, if the Java 2 SDK or JRE was installed in `/ctm_em/user1`, `JAVA_HOME` would be either:`/ctm_em/user1/jdk1.6.x/jre [JDK]` or `/ctm_em/user1/jre1.6.x [JRE]`.

**To Install Control-M/EM API:** 

Do one of the following to create the **emapi-900** directory:

* **Microsoft Windows:** Unzip the `emapi-900-nt.zip` file from the cd `Path/Setup_files/TOOLS/EMAPI_FILES` directory on the Control-M/EM installation CD to any location.
* **UNIX:**
   * Solaris, AIX, HP-UX or HP-UX Itanium: Locate the `emapi-900-UNIX.TAR.Z` tar file in the `cdPath/Setup_files/TOOLS/EMAPI_FILES` directory on the Control-M/EM installation CD and type the following command to open and uncompress the file to any directory: uncompress `-c cdPath/TOOLS/EMAPI_FILES/emapi-900-UNIX.TAR.Z | tar xvf` 
   * RedHat or Suse: Locate the `emapi-900-UNIX.TAR.gz` tar file in the `cdPath/Setup_files/TOOLS/EMAPI_FILES` directory on the Control-M/EM installation CD and type the following command to open and uncompress the file to any directory: 

`-gunzip -c cdPath/TOOLS/EMAPI_FILES/emapi-900-UNIX.TAR.gz | tar xvf-`
 
All Control-M/EM API files and sub-directories are located in this directory. See attached Control-M/EM API primary subdirectories (on page 12). 

_**NOTE:** To uninstall Control-M/EM, delete the Control-M/EM API directory according to your version. For example, for version 9.0.00 delete the emapi-900 directory. Do not copy the Control-M/EM API version 9.0.00 files directly over the previous installation This may cause unpredictable behavior._

### 2.2 Configure xMatters Integration Agent

When you install the Control-M/EM API server, it creates two properties files within its installation folder: ctmemapi.properties and jacorb.properties . Both of these files are required by the integration to access and communicate with the API server.

To ensure that the integration can access these properties files, do one of the following:

   * Copy both ctmemapi.properties and jacorb.properties from the Control-M/EM API server installation folder to the `<IAHOME>/integrationservices/controlm25` folder.
   * The files are located in the following location: 
     * **ctmemapi.properties:** `<API Directory>/emapi-900/`
     * **jacorb.properties:** `<API Directory>/emapi-900/etc`
   * Copy the **emapi.jar** and **jacorb.jar** from the Control-M API to the `integrationservices/lib` directory.
     * Both files can be found here: `<API Directory>/emapi-900/classes`
   * Update the `CTMEMAPI_PROPERTIES_FILE` and `JACORB_PROPERTIES_FILE` parameters in the `controlmconfig.js` file to point to the location of the files within the API server folder.

## 3. Updating the user password

The password for the BMC Control-M user is stored in an encrypted password file, stored in the <IAHOME>/conf subfolder. For the integration to connect to the API, you will have to replace the default value ("password") with the password of the database user specified in the controlm-config.js file (as described in the table above.)

_**Note:** For more information about the integration agent's IAPassword feature used to encrypt the password, see the xMatters integration agent guide._

To change the encrypted password:

1. Navigate to the <IAHOME>/bin subfolder, and then run the following command, replacing <newPassword> with the password of the BMC Control-M user:

`./iapassword.sh --new "<newPassword>" --old password --file conf/bmccontrolm.pwd`

_**Note:** that if you want to change this password again, you will have to replace "password" in the above command with the existing password._

## 4. Configuring BMC Control-M Workload Automation

The following sections describe how to configure BMC Control-M to combine with xMatters.

### 4.1 Define shout destinations

The first step in configuring BMC Control-M is to specify the destinations for the xMatters and xMattersDel shout destinations.

**To define shout destinations:**

1. In the BMC Control-M Workload Automation Configuration Manager, open the Shout Destinations Manager.

2. Select  Actions > Add Shout table (or  Update Shout Table to modify an existing table).

3. Add a Shout Destination with the following properties:

   * **Logical Name:** xMatters
   * **Address:** Server
   * **Destination:** Program
   * **Value:** Type the location of the integration agent's bin folder where the Controlm-APClient.bat file was installed; for example:

    `C:/PROGRA~1/xmatters/integrationagent/bin/Controlm-APClient.bat`

<kbd>
  <img src="media/Shout%20Destination%20Definition.png">
</kbd>


4. Add another Shout Destination with the following properties:

   * **Logical Name:** xMattersDel
   * **Address:** Server
   * **Destination:** Program
   * **Value:** Type the location of the integration agent's bin folder where the Controlm-APClient-Del.bat file was installed; for example:

   `C:/PROGRA~1/xmatters/integrationagent/bin/Controlm-APClient-Del.bat`
   
   ### 4.2 Using shout destinations

Once you have defined the shout destinations, you can add them to a job in your system. The xMatters shout (Controlm-APClient.bat) sends notifications into xMatters, and allows users to take actions depending on the current state of the job when the notification is sent. The xMattersDel shout (Controlm-APClient-Del.bat) is used to clean up outstanding notifications; you can use this when a job ends with an OK state to remove any outdated notifications from xMatters.

**To use the xMatters shout destination for a job:** 
   
1. In the BMC Control-M Workload Automation Configuration Manager, open the job to which you want to add the shout destination.

2. Open the **Actions** tab, and add an On-Do Action for the job.

3. In the **On** drop-down list, select the action you want to use as the trigger for the notification.

4. In the **Do** drop-down list, select Notify.

5. In the **Destination** field, select xMatters, and select an Urgency.

6. In the **Message** field, type `%%ORDERID` and then use the following syntax to specify the recipients and any custom message you want to add:

`%%ORDERID;<userID>,<userID2>;<message>`

Note that you must use semi-colons (;) to separate the components in the message field, and commas (,) to separate the xMatters User IDs that identify the recipients. For example:

`%%ORDERID;bsmith,cogrady,admin;The job was completed and assigned.`

<kbd>
  <img src="media/Shout%20Destination.png">
</kbd>


7. Click OK  to save the action.

**To use the xMattersDel shout destination for a job:**

1. In the BMC Control-M Workload Automation Configuration Manager, open the job to which you want to add the shout destination.

2. Open the **Actions** tab, and add an On-Do Action for the job.

3. In the **On** drop-down list, select the action you want to use as the trigger.

4. In the **Do** drop-down list, select Notify .

5. In the **Destination** field, select xMattersDel , and select an Urgency .

6. In the **Message** field, type `%%ORDERID`

7. Click **OK** to save the action.

## 5. Configuring xMatters

Configuring xMatters to combine with BMC Control-M Workload Automation requires the following step:

   * Import of Communication plan
   * Access Inbound Integration URL's
   * REST API User
   * Permissions
   
### 5.1 Import of Communication Plan
1. Login to xMatters OnDemand
2. Navigate to the Developer tab
3. Import the Control-M [Communication Plan](/components/BMCControlMIntegration%20Comm%20Plan.zip)

### 5.2 Access Inbound Integration URL's
To successfully configure the controlm-config.js navigate to xMatters OnDemeand to retrieve the inbound integration URL's.

1. Login to xMatters OnDemand
2. Navigate to the Developer tab
3. Find the newly imported Control-M communication plan
4. Select Edit > Integration Builder
5. From within the Integration Builder, open Inbound Integrations
6. From within the Inbound Integrations, individually select the link for each inbound integration, and map accordingly to the controlm-config.js file.
7. The Inbound Integration URL's should be mapped to `XMATTERS_IB_INBOUND_INTEGRATIONS` in the controlm-config.js

### 5.3 REST API User
To successfully integration and submit to xMatters OnDemand, it is necessary to configure a REST API User in xMatters.

To create an integration user:

1. Log in to the target xMatters system.
2. On the Users tab, click Add.
3. Enter the appropriate information for your new user. Because this user will affect how messages appear for recipients and how events will be displayed in the reports and Communication Center, you may want to identify the user as specific to Control-M; for example:
  * First Name: Control-M
  * Last Name: Integration
  * User ID: controlm.rest
  * Assign the user to the REST Web Service User, Group Supervisor, and Person Supervisor roles.
  * Set the user ID and password.
  * Make a note of these details; you will need them when configuring other parts of the integration.
  * Click Save.

### 5.4 Permissions
   
#### 5.4.1 Sender Permissions
In the Edit drop-down list for the plan, select Forms.
For the Incident Alerts form, click the Not Deployed drop-down list, and then select Create Event Web Service (in some deployments, Enable for Web Service).
In the Web Service Only drop-down list, click Permissions.
Add the REST API user you created above, and then click Save Changes.
Repeat steps 6-8 for the Engage with xMatters and Conference Bridge forms.
#### 5.4.2 Access Permissions
1. Login to xMatters OnDemand
2. Navigate to the Developer tab
3. Find the newly imported Control-M communication plan.
4. From the Edit dropdown select Access Permisssions
5. Add the following role:
   * Rest Web Services User

#### 5.4.3 Edit Endpoints
To configure the endpoints:

1. On the communication plan, click the Integration Builder tab, and then click the Edit Endpoints button to display the endpoints for the integrations.
2. Click the xMatters endpoint, assign the endpoint to the integration user you created in step 5.3 and then click Save Changes.
  
## 6. Configuration Validation

The following sections will test the combination of xMatters and BMC Control-M for notification delivery and response.

After configuring xMatters and BMC Control-M, you can validate that communication is properly configured. It is recommended that you start the components in the following order:

   * BMC Control-M
   * xMatters integration agent
   * xMatters

### 6.1 Triggering a notification

To trigger a notification, run a new job in BMC Control-M, and cause it to enter a state for which you have defined an xMatters Shout Message:

<kbd>
  <img src="/media/Triggering%20Notification.png">
</kbd>

### 6.2 Responding to a notification
This section describes how to respond to a notification from xMatters. In the following example, the notification is
received in an email, but the process is similar for all Devices.

**To respond to a notification:**
1. When a notification arrives for the User, open it to view the details:
2. The list of possible replies is located at the bottom of the notification. For email notifications, click one of the
response option to send your response to BMC Control-M.
Note that in the default integration configuration, you can also select response options to retrieve information about
the job from BMC Control-M.
3. To view the results of the response, view the job report details in BMC Control-M. In the following image, you can
see that the notified user selected the "Rerun" response option, causing the job to be rerun:
<kbd>
  <img src="/media/Response.png">
</kbd>

## 7 Troubleshooting
* Consult the [Control-M EM API Documentation](/components/CTM_API_9.0.00.200_480104.pdf) if experiencing any issues related to the Control-M API Installation.
* Consult the [Integration Agent Documentation](https://support.xmatters.com/hc/en-us/articles/201463419-Integration-Agent-for-xMatters-5-x-xMatters-On-Demand) if experiencing any issues related to configuring the Integration Agent
* Consult [xMatters Help](https://help.xmatters.com/) if experiencing any issues within xMatters OnDemand.
