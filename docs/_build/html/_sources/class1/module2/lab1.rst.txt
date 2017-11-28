Lab 2 – Deploy an F5 Web Application Firewall using the Azure Security Center
--------------------------------------------------------------------------------

This lab will teach you how to deploy a WordPress server in Azure and protect
the application with an F5 WAF via the Azure Security Center (ASC). The ASC
will automatically discover vulnerabilities within your Azure resources. Things
like publicly accessible IPs listening on HTTP and HTTPS will be flagged in
ASC and marked for review. The customer can take action which can include
deployment of a WAF. Therefore, we'll expose a WordPress server and let
ASC find the problem for us. After a few questions and answers, a WAF will be
deployed automatically.

Lab 2 – Topology
~~~~~~~~~~~~~~~~

   .. image:: /_static/lab-2-topology.png

Task 1 – Setup basic cloud components in Azure
----------------------------------------------

Basic cloud components are needed first. Things like Virtual Networks and
Resource Groups are required prior to deployment of virtual machines.
You will configure these items in preparation for the WordPress
deployment.

#. Log into the Microsoft Azure Portal – https://portal.azure.com
#. Click the green **+** sign at the top left corner of the screen
#. Click on **Networking**
#. Click on **Virtual network**

   .. image:: /_static/image57.png
      :scale: 50 %

   Use the information provided in Table 2.1 below to create a virtual network.

   Table 2.1

   +-----------------------+------------------------------+
   | Key                   | Value                        |
   +=======================+==============================+
   | Name                  | user<student number>_vnet    |
   +-----------------------+------------------------------+
   | Address space         | 10.10.0.0/16                 |
   +-----------------------+------------------------------+
   | Subscription          | <User Unique>                |
   +-----------------------+------------------------------+
   | Resource group        | Create new: wordpress        |
   +-----------------------+------------------------------+
   | Location              | <Closest Azure DC>           |
   +-----------------------+------------------------------+
   | Address Range         | 10.10.0.0/22                 |
   +-----------------------+------------------------------+

   .. image:: /_static/image58.png
      :scale: 50 %

#. Click **Create**

Task 2 – Deploy and configure WordPress within Azure
----------------------------------------------------

In this task you will deploy another virtual machine and install the
WordPress application to be placed behind the BIG-IP. Let's go back to
the Microsoft Azure Portal.

#. Click the green **+** sign at the top left corner of the screen
#. Start searching the marketplace by typing 'bitnami wordpress' in the
   search field and hit **Enter**

   .. image:: /_static/image32.png
      :scale: 50 %

#. Select **WordPress Certified by Bitnami**

   .. image:: /_static/image33.png
      :scale: 50 %

#. Click on **Create** at the bottom of the screen

   Use the information in Table 2.2 to complete the “Basics” configuration
   page during this deployment.

   Table 2.2

   +-----------------------+-------------------------------------------------+
   | Key                   | Value                                           |
   +=======================+=================================================+
   | Name                  | user<student number>wordpress                   |
   +-----------------------+-------------------------------------------------+
   | VM disk type          | SSD                                             |
   +-----------------------+-------------------------------------------------+
   | User name             | f5bigipuser<student number>                     |
   +-----------------------+-------------------------------------------------+
   | Authentication type   | SSH public key                                  |
   +-----------------------+-------------------------------------------------+
   | SSH public key        | From Lab 1, Task 1                              |
   +-----------------------+-------------------------------------------------+
   | Subscription          | <User Unique>                                   |
   +-----------------------+-------------------------------------------------+
   | Resource Group        | Use existing: wordpress                         |
   +-----------------------+-------------------------------------------------+
   | Location              | <Closest Azure DC>                              |
   +-----------------------+-------------------------------------------------+

   .. image:: /_static/image59.png
      :scale: 50 %

#. Click **OK** at the bottom of the page

   Use the information in Table 2.3 to complete the “Choose a size”
   configuration page during this deployment.

   Table 2.3

   +-------------+------------+
   | Key         | Value      |
   +=============+============+
   | Disk Type   | HHD        |
   +-------------+------------+
   | Size        | A1 Basic   |
   +-------------+------------+

#. Choose **A1 Basic**

   .. image:: /_static/image35.png
      :scale: 50 %

#. Click **Select**

   Use the information in Table 2.4 to complete the “Settings” configuration
   page during this deployment.

   .. NOTE::
      On the Settings page you’ll see a warning concerning the VM size
      chosen.

   Table 2.4

   +---------------------+---------+
   | Key                 | Value   |
   +=====================+=========+
   | Storage Type        | HHD     |
   +---------------------+---------+
   | Use managed disks   | No      |
   +---------------------+---------+

#. Change the "Disk type" to **HDD**
#. Set “Use managed disk” to **No**
#. Keep the other configurations unmodified

   .. image:: /_static/image60.png
      :scale: 50 %

#. Click **OK**
#. Verify the summary

   .. image:: /_static/image37-top.png
      :scale: 50 %

#. Supply your email and phone number for validation

   .. image:: /_static/lab-instance-validation.png

#. Click **Purchase** or **Create**
#. Go to **Resource groups** and click on your resource group
#. Select your WordPress “Public IP address”

   .. image:: /_static/image61.png
      :scale: 50 %

   .. image:: /_static/image62.png
      :scale: 50 %

   .. Note::
      Remember the WordPress public IP address. This will be used in
      subsequent steps.

Task 3 – Access WordPress instance and launch a SQL Injection attack
--------------------------------------------------------------------

The next task involves testing the application and checking for open
vulernabilities. You will need to access your WordPress instance and
launch a simple SQL Injection attack.

#. Open a web browser and navigate to \http://<wordpress-public-IP>
#. Navigate to the **Search** box. You can do this via two methods:

   - Scrolling down the page with the browser scroll bars
   - Or...
     
     - Click the **X** in the lower right corner of the screen
     - Close the **Manage** link
     - Click the arrow in bottom right corner of the screen

#. In the search box, enter the string ``'or 1=1#`` to launch the SQL
   Injection attack.

   .. image:: /_static/image63.png
      :scale: 50 %

#. Hit **Enter**
#. Perform this task several times to simulate an attack

   Although the WordPress application does not respond with any records,
   there are in fact no safeguards against this SQL injection attack.

   .. NOTE::
      ``'or 1=1#`` is an example of a simple SQL Injection attack. A
      \ `SQL injection <https://www.owasp.org/index.php/SQL_injection>`__
      attack consists of insertion or "injection" of a SQL query via the
      input data from the client to the application. A successful SQL
      injection exploit can read sensitive data from the database, modify
      database data (Insert/Update/Delete), execute administration
      operations on the database (e.g. shutdown the DBMS), recover the
      content of a given file present on the DBMS file system, and in
      some cases issue commands to the operating system.

Task 4 – Accept EULA for F5 WAF in Azure Marketplace
----------------------------------------------------------------------

Prior to using Azure Security Center or other Marketplace items, you must
enable that particular item in Azure Marketplace (e.g. accept EULA). In
this task you will go to the Azure Marketplace and enable **the F5 WAF Solution for ASC**.

.. Note::
   If you have already performed this step in your Azure account,
   then you can skip this task and move to the next task.

#. Open a browser and go to https://azuremarketplace.microsoft.com/en-us/marketplace/apps/f5-networks.f5-web-application-firewall 

   .. image:: /_static/image64.png
      :scale: 50 %

#. Click **GET IT NOW**
#. Complete the sign in process using the email address used to set
   up your account.
#. Accept the EULA by clicking **Continue**

   .. image:: /_static/image65.png
      :scale: 50 %

Task 5 – Launch Azure Security Center and deploy the F5 WAF
----------------------------------------------------------------------

Among other things, Azure Security Center (ASC) makes recommendations to
optimize and secure your web applications. You will now follow the
recommendation from ASC to deploy the F5 pre-configured WAF in front
of your WordPress application.

#. Go back to the Microsoft Azure portal and navigate to Azure Security
   Center.

   .. image:: /_static/image66.png
      :scale: 50 %

#. Click on **Security Center -> Welcome**
#. Click **Launch Security Center** and notice that ASC has recommendations
   for your environment

   .. image:: /_static/image67.png
      :scale: 50 %

#. Click on **Recommendations**

   .. Tip::
      Recommendations are created by the Azure Security Center to make your
      applications more secure. One of the recommendations is to
      **Add a web application firewall**.

#. In the "Recommendations" page, select the **Add a web application firewall**
#. Click on the name of the application to the right of the screen

   Example: *user<student number>wordpress-ip* in the screenshot below

   .. image:: /_static/image68.png
      :scale: 50 %

   .. Note::
      If the name of your WordPress does not appear, please wait a few
      minutes until Azure Security Center can create the Recommendations.

#. Click on **Create New**

   .. image:: /_static/image69.png
      :scale: 50 %

#. Select **F5 Networks**

   .. image:: /_static/image70.png
      :scale: 50 %

   .. Note::
      There are two deployment methods available today for the
      pre-configured F5 WAF:

      - “Automatically provisioned”
      - “Semi-automatically provisioned”

      For this lab you will be using the **“Semi-automatically provisioned”** method.
      The former will most likely be depracated. The latter, semi-automatically provisioned,
      gives the user more control in the deployment.

#. Select the option for **Semi-automatically provisioned**

   .. image:: /_static/lab02-waf01.png
      :scale: 50 %

#. Click **Create**

   Use the information in Table 2.5 to complete the “Basics” page
   during this deployment. Leave all other settings as default.

   Table 2.5

   +-----------------------+-------------------------------------------------+
   | Key                   | Value                                           |
   +=======================+=================================================+
   | Subscription          | <User Unique>                                   |
   +-----------------------+-------------------------------------------------+
   | Resource Group        | Create new: wordpress-acs<student number>       |
   +-----------------------+-------------------------------------------------+
   | Location              | <User Unique>                                   |
   +-----------------------+-------------------------------------------------+

   .. image:: /_static/lab02-waf02.png

#. Click **OK**

   Use the information in Table 2.6 to complete the “Insfrastructure Settings” page
   during this deployment. Leave all other options as default.

   Table 2.6

   +------------------------+-------------------------------------+
   | Key                    | Value                               |
   +========================+=====================================+
   | Deployment Name        | F5waf<student number>               |
   +------------------------+-------------------------------------+
   | BIG-IP Version         | Choose latest 13x available         |
   +------------------------+-------------------------------------+
   | F5 WAF Password        | Demo123Demo123!                     |
   +------------------------+-------------------------------------+
   | Confirm Password       | Demo123Demo123!                     |
   +------------------------+-------------------------------------+
   | License token          | <license provided by the proctor>   |
   +------------------------+-------------------------------------+

   .. image:: /_static/lab02-waf03.png

#. Click **OK**

   Use the information in Table 2.7 to complete the “Network Settings” page
   during this deployment. Leave all other options as default.

   Table 2.7

   +------------------------+---------------------------------------------+
   | Key                    | Value                                       |
   +========================+=============================================+
   | Domain name label      | f5waf<student number>                       |
   +------------------------+---------------------------------------------+
   | Subnets                | You'll need to hit **Configure Subnets**    |
   +------------------------+---------------------------------------------+

   .. image:: /_static/lab02-waf04.png

#. Select **Configure Subnets**
#. Leave all options as default on the "Subnets" page

   .. image:: /_static/lab02-waf05.png

   .. Note::
      This will create a 3-nic F5 instance.

#. Click **OK** to go back to the "Network Settings" page.
#. Notice the "Subnets" field will change to *Review subnet configuration*.

   .. image:: /_static/lab02-waf06.png

   .. Note::
      There is no need to hit **Review subnet configuration**. This simply means
      there are now subnets configured whereas before there were none.

#. On the "Network Settings", click **OK** to proceed to the next page

   Use the information in Table 2.8 to complete the “Application Settings” page
   during this deployment. Leave all other options as default.

   Table 2.8

   +----------------------------------------+----------------------------------------+
   | Key                                    | Value                                  |
   +========================================+========================================+
   | Application Protocol(s)                | HTTP                                   |
   +----------------------------------------+----------------------------------------+
   | Application Address                    | <wordpress-public-IP>                  |
   +----------------------------------------+----------------------------------------+

   .. image:: /_static/lab02-waf07.png

#. Click **OK** to proceed to the next page
#. Review the "Summary Page". You should receive **Validation passed**

   .. image:: /_static/lab02-waf08.png

#. Click **OK** to proceed to the next page
#. Review the "Terms and use" page

   .. image:: /_static/lab02-waf09-top.png

#. Scroll down to review the remaining "Terms and use" page
#. Supply your email and phone number for validation

   .. image:: /_static/lab02-waf09-bottom.png

#. Click **Create**

   .. Note::
      Deployment time can take up to 30 minutes.

Task 6 – Review F5 WAF Configurations and Policies
--------------------------------------------------

Take this time to review the various components that are automatically
provisioned as part of the Azure Security Center.

#. Click on the Resource Group that deployed the F5 WAF

   .. Hint::
      It will be named wordpress-asc…

#. Click on **Public IP address** for the F5 device

   .. image:: /_static/lab02-waf10.png

   .. Note::
      Remember the F5 public IP address. This will be used in
      subsequent steps.

   .. image:: /_static/lab02-waf11.png

#. Open a web browser and go to the BIG-IP GUI at \https://<F5-Public-IP>
   to see when the platform completes the deployment
#. Login as admin (or azureuser) and use the password you entered during the WAF
   deployment process.

   .. image:: /_static/lab02-waf12.png

   .. WARNING::
      The deployment takes time. If you observe it from the GUI,
      you will see a reboot. This automated background deployment
      (licensing, creating the pool and virtual server) may take 10 minutes
      or longer. Please be patient and do not interrupt this process.
      Once the Virtual Server is created, the setup of F5 WAF is complete.

#. Review the F5 configurations by first going to **LTM -> Virtual Servers**

   .. image:: /_static/lab02-waf13.png

#. Notice that the Azure Security Center WAF deployment automatically created
   the required virtual server
#. Select the virtual server to view properties
#. Review the various settings on the "Properties" tab
#. Then select the "Resources" tab
#. Notice the pool has been automatically created and added
#. Also notice the **Policies** section has a *Local Traffic Policy* assigned.
   This will direct traffic of interest to the WAF policy on the F5.

   .. image:: /_static/lab02-waf14.png

#. Review the **LTM -> Pools**

   .. image:: /_static/lab02-waf15.png

   .. Note::
      This pool contains the public IP address of the WordPress server you initially
      created in the earlier section of this lab.

#. Notice that the Azure Security Center WAF deployment automatically created
   the required pools

   .. Hint::
      If you look more closely, you'll realize that the Azure Security Center actually
      deployed the F5 base provisioning, downloaded the WAF policy, and then ran a
      declarative call to automate the provisioning of all required F5 L4-L7 services
      using F5 iApps.

   Time permitting, go explore the iApps in the F5 GUI under **iApps -> Application Services**.
   You can also review the F5 Application Security Manager (ASM = WAF) section under
   **Security -> Application Security**.

Task 7 – Demonstrate F5 WAF blocking functionality
--------------------------------------------------

As part of the WAF deployment, a new F5 VIP (virtual IP/listener) has been
configured for the WordPress application that sits behind an Azure NAT rule.
Additionally, a base WAF policy has been configured automaticaly for
the application. To test the WAF policy, you will repeat the SQL injection
attack from a previous lab against the WordPress application. However this
time you will access the WordPress application through the F5 protected WAF policy.

First, you need to identify the public IP address for the Azure load balancer.

#. Click on the Resource Group that deployed the F5 WAF

   .. Hint::
      It will be named wordpress-asc…

   .. image:: /_static/lab02-waf16.png

#. Copy the **Public IP address** for the Azure load balancer device

   .. image:: /_static/lab02-waf17.png

   .. Note::
      Remember the Azure LB public IP address. This will be used in
      subsequent steps.

#. Open a web browser and go to \http://<azure-lb-public-ip>

   .. image:: /_static/lab02-waf18.png

   .. Note::
      The Azure NATs found within the Azure load balancer (ALB)
      control the NAT decisions. This allows proper traffic direction
      depending on if it is F5 management traffic or client/server traffic.

      If you want to explore the Azure load balancer NAT and load balancer
      rules, then stay on the Load Balancer page and review the various settings.
      Now would be a good time to raise hands for any questions.

   Let's proceed with an attack through the F5!

#. Navigate to the **Search** box. You can do this via two methods:

   - Scrolling down the page with the browser scroll bars
   - Or...
     
     - Click the **X** in the lower right corner of the screen
     - Close the **Manage** link
     - Click the arrow in bottom right corner of the screen

#. In the search box, enter the string ``'or 1=1#`` to launch the SQL
   Injection attack.

   .. image:: /_static/image63.png
      :scale: 50 %

#. Hit **Enter**
#. Perform this task several times to simulate an attack. Notice that the F5 BIG-IP WAF policy is now protecting the WordPress
   application from this SQL injection attack.

   .. image:: /_static/image80.png
      :scale: 50 %

#. Open another web browser and go to the BIG-IP GUI at
   \https://<F5-public-IP>
#. Go to **Security -> Event Logs -> Application -> Requests**

   .. image:: /_static/image81.png
      :scale: 50 %

#. Click on the line with the highest “Violation Rating” link
   to view full request information

   .. image:: /_static/image82.png
      :scale: 50 %

#. Click on **Attack signature detected**

   .. image:: /_static/image83.png
      :scale: 50 %

#. Click on **View details...**

   .. image:: /_static/image84.png
      :scale: 50 %

   .. Note::
      The F5 WAF has successfully detected the SQL injection attack
      and protect the WordPress application.

Task 8 – Finalize the WAF Deployment
------------------------------------

Now that you have successfully tested the path to WordPress through the
F5 BIG-IP, you need to finalize the WAF deployment. Currently access
still works direct to the WordPress application via public IP address
\http://<wordpress-public-IP> as demonstrated in Task 1 of this lab.
Finalizing the WAF deployment will eliminate the ability to access
the WordPress application directly. Access to the WordPress
application will only be available through the F5 BIG-IP.

#. Go back to the Microsoft Azure portal and navigate to Azure Security
   Center
#. Click on **Security Center -> Overview**

   .. image:: /_static/image85.png
      :scale: 50 %

#. Click **Recommendations**
#. Select **Finalize web application firewall setup**

   .. image:: /_static/image86.png
      :scale: 50 %

#. Click on the WordPress application

   .. image:: /_static/image87.png
      :scale: 50 %

#. You will be presented a message stating to complete the remaining tasks
   via the *Solutions Center*.

   .. image:: /_static/lab02-waf19.png

#. Click **OK**
#. Go back to Azure Security Center and select **Security solutions**

   .. image:: /_static/lab02-waf20.png

#. In the "Connected solutions", choose your WAF by selecting **View**
#. On the next screen, select your WAF instance and then choose **Finalize application protection**

   .. image:: /_static/lab02-waf21.png

#. On the "Finalize application protection" screen, select your WAF instance

   .. image:: /_static/lab02-waf22.png

#. Read the message and perform the necessary actions

   .. Hint::
      At this point, you need to take some type of action outside of Azure Security
      Center. In this case, you need to update the WordPress instance's network security
      group to restrict the inbound HTTP/HTTPS access to only the F5 (if doing single 1-nic)
      deployment or the Azure LB public IP (if doing multiple-nic deployment).

      Now is a good time to raise your hand with questions.

#. When done, refresh the **Security solutions** page again
#. Notice the health of the solution is now green

   .. image:: /_static/lab02-waf23.png

   .. Note::
      If the health status is still red, then please review the NSG linked to the WordPress
      instance. Come back to the Solutions center and finalize the WAF again.

      Also, after some time the solution will disappear once there is no more action to take.
      This is a good sign that the finalization tasks are complete.

#. Once all actions are perfomed (e.g. lock down NSG), then go back to Azure Security Center
#. View **Recommendations** again and notice that "Finalize application protection" for your
   WAF instance is marked as *Resolved*

   .. image:: /_static/lab02-waf24.png

#. Open a web browser and go to \http://<wordpress-public-IP>
#. Notice that the page no longer loads

   .. image:: /_static/image89.png
      :scale: 50 %

#. Sanity check...test access via the F5 WAF again and go to \http://<F5-public-IP>

   .. image:: /_static/image01-wordpress.png
      :scale: 50 %

   .. ATTENTION::
      Testing WordPress by going through the F5 should successfully load.
      Testing WordPress IP directly should fail.

   .. image:: /_static/image56.gif
      :scale: 50 %

**This concludes Lab 2**
