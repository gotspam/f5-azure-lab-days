Lab 3 – Deploy an HA Pair of F5 BIG-IP in Azure Using an ARM Template
---------------------------------------------------------------------

In this step you will be deploying the F5 BIG-IP via an ARM template.
F5 offers a wide variety of ARM templates for various deployment
scenarios and licensing options. There are supported and experimental
templates. This lab will use a template from the supported folder to
do all the heavy lifting.

To see a list of all F5 Azure ARM templates, go to
https://github.com/F5Networks/f5-azure-arm-templates. Take your time
to review the README.md.

.. Tip::
   The supported templates are fully supported by F5, and customers
   can call for issues. The experimental folder contains other
   template options.

For this lab, we will be using a template from the following location:

https://github.com/F5Networks/f5-azure-arm-templates/tree/master/supported/cluster/1nic/new_stack

It's recommended to review the README.md on this page as well to become
familiar with the template, prerequisites, topology, and more.

Lab 3 – Topology
~~~~~~~~~~~~~~~~

.. image:: /_static/lab-3-topology.png

Task 1 – Deploy an ARM template 
-------------------------------

This task will walk through a series of steps in order to choose
and deploy an ARM template in the Azure Portal. Let's start by
going to the Azure Portal.

#. Open a browser and go to https://portal.azure.com

   .. image:: /_static/lab3-image001.png
      :scale: 50 %

#. Keep the browser window open
#. Open a new browser and go to
   https://github.com/F5Networks/f5-azure-arm-templates/tree/master/supported/cluster/1nic/new_stack
#. Scroll down to the README.md section and click **Installation**

   .. image:: /_static/lab3-image002.png
      :scale: 50 %

#. Notice the different licensing options
#. Choose **BYOL** by right-clicking and opening in a new tab

   .. image:: /_static/lab3-image003.png
      :scale: 50 %

   .. Note::
      You will be directed back to the Azure Portal.

      Alternatively, you can initiate a new template from Azure which
      will require a few more clicks and some copy/paste skills.

      - Go to your Azure Portal dashboard
      - Click the green **+** sign at the top left corner of the screen
      - Start searching the marketplace by typing 'template' in the search field
      - Hit **Enter**
      - Select the **Template Deployment** and click **Create**
      - Find the ARM template on github
      - View the github azuredeploy.json file in "Raw"
      - Select all text and copy
      - Go back to Azure Portal
      - Select all text and paste into the "Edit Template" window
      - Click **Save**
      - Or...just click the easy deploy button from github

#. You should now be back at the Azure Portal in "Custom Deployment"

   Use the information provided in Table 3.1 to complete the Custom
   deployment process. Leave all other settings as default.

   Table 3.1

   +--------------------------+---------------------------------+
   | Key                      | Value                           |
   +==========================+=================================+
   | Subscription             | <User Unique>                   |
   +--------------------------+---------------------------------+
   | Resource group           | Create new                      |
   +--------------------------+---------------------------------+
   | Resource group name      | bigipuser<student number>-aa    |
   +--------------------------+---------------------------------+
   | Location                 | <Closest Azure DC>              |
   +--------------------------+---------------------------------+
   | Admin Password           | Demo123Demo123!                 |
   +--------------------------+---------------------------------+
   | Dns Name for Public IP   | f5bigipuser<student number>aa   |
   +--------------------------+---------------------------------+
   | License Key 1            | <Provided by proctor>           |
   +--------------------------+---------------------------------+
   | License Key 2            | <Provided by proctor>           |
   +--------------------------+---------------------------------+

   .. image:: /_static/lab3-image004.png
      :scale: 50 %

#. Scroll down and check “I agree to the terms and conditions stated
   above”
#. Click on **Purchase**

   .. image:: /_static/lab3-image005.png
      :scale: 50 %

#. Review "Notifications" in the top right and notice your new deployment.

   .. image:: /_static/lab3-image006.png
      :scale: 50 %

   .. Note::
      You just setup two BIG-IPs as a cluster without having to know any
      specific knowledge about the F5. Deployment can take 10-15
      minutes so be patient. Both instances are active and are not setup
      the traditional way because of the network limitation in Azure.
      There is no L2 connectivity (similar to AWS). As a result, HA is
      assisted with placing an ALB in front of the F5 instances and also
      by using F5 sync groups.

Task 2 – Review Results of ARM Template
---------------------------------------

In the previous task, you used an ARM template to automate the deployment
of F5 instances and Azure networking components. Now is a good time to
review other pieces that were created by the ARM template. Let's go back
to the Azure Portal.

#. Go to **Resource groups** and click on your resource group
#. Look for the resource type load balancer and select it

   .. image:: /_static/image96.png
      :scale: 50 %

   .. Note::
      This load balancer is an Azure Load Balancer (ALB) which will be in
      front of the two BIG-IPs and used to support the setup of the cluster.

   The ALB has some important NAT rules to explore. These will direct
   management traffic to the appropriate F5. Review the NAT rules first.

#. From the Resource Group, click **Inbound NAT rules**

   .. image:: /_static/image97.png
      :scale: 50 %

   .. Note::
      Remember the different service ports. These will be used in subsequent steps
      to connect to the F5 for management purposes.

#. From the Resource Group, click **Backend pools**
#. Expand **loadBalancerBackEnd** to view the IP addresses of the
   F5 BIG-IPs sitting behind the Azure ALB

   .. image:: /_static/image98.png
      :scale: 50 %

   .. Note::
      Take note of these IP addresses. These are the applications running
      on the F5 instances. The ALB is load balancing each F5 in active/active.

   Next you will need to create some Azure load balancing rules. These will
   help direct client/server traffic to the appropriate F5 based on F5
   health within the active/active cluster. However, first Azure requires
   that health probes are created.

#. From the Resource Group, click **Health probes**
#. Then click the **+ Add** button

   .. image:: /_static/lab03-arm01.png

   Use the information provided in Table 3.2 to complete the "Add health probe"
   page. Leave all other settings as default.

   Table 3.2

   +--------------------------+---------------------------------+
   | Key                      | Value                           |
   +==========================+=================================+
   | Name                     | lbprobeHTTP1                    |
   +--------------------------+---------------------------------+
   | Protocol                 | HTTP                            |
   +--------------------------+---------------------------------+
   | Port                     | 8081                            |
   +--------------------------+---------------------------------+

   .. image:: /_static/lab03-arm02.png
   
#. Scroll down and select **OK** and the results will show one new health probe

   .. image:: /_static/lab03-arm03.png

   The health probe is created and now you'll need to create the
   Azure load balancing rules.

#. From the Resource Group, click **Load balancing rules**
#. Then click the **+ Add** button

   .. image:: /_static/lab03-arm04.png

   Use the information provided in Table 3.3 to complete the "Add load balancing rule"
   page. Leave all other settings as default.

   Table 3.3

   +--------------------------+---------------------------------+
   | Key                      | Value                           |
   +==========================+=================================+
   | Name                     | APP1-HTTP                       |
   +--------------------------+---------------------------------+
   | Backend port             | 8081                            |
   +--------------------------+---------------------------------+

   .. image:: /_static/lab03-arm05.png

#. Scroll down and select **OK** and the results will show one new load balancing rule

   .. image:: /_static/lab03-arm06.png

   .. Note::
      Take note of the different port mappings.

   You will now connect to the F5 BIG-IPs. To do so you first need to
   identify the BIG-IPs’ public IP addresses.

#. Go to **Resource groups** and click on your resource group
#. From the Resource Group, click one of the F5 virtual machines to see its public IP

   .. image:: /_static/image100.png
      :scale: 50 %

   .. image:: /_static/image101.png
      :scale: 50 %

#. Then click the other F5 virtual machine and notice that both F5 BIG-IPs have
   the same public IP address

   .. Hint::
      You can access each individual unit by using the service ports identified
      earlier in the NAT rules.

#. Open a browser and connect to the BIG-IPs' management GUIs using the NAT'd
   service ports. In our case, the service ports are 8443 and 8444.

   - F5 #1 = \https://<public-IP>:8443
   - F5 #2 = \https://<public-IP>:8444

   .. image:: /_static/image119.png
      :scale: 50 %

Task 3 – Deploy and configure WordPress within Azure
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

   Use the information in Table 3.4 to complete the “Basics” configuration
   page during this deployment.

   Table 3.4

   +-----------------------+----------------------------------------+
   | Key                   | Value                                  |
   +=======================+========================================+
   | Name                  | user<student number>wordpress          |
   +-----------------------+----------------------------------------+
   | VM disk type          | SSD                                    |
   +-----------------------+----------------------------------------+
   | User name             | azureuser<student number>              |
   +-----------------------+----------------------------------------+
   | Authentication type   | SSH public key                         |
   +-----------------------+----------------------------------------+
   | SSH public key        | From Lab 1, Task 1                     |
   +-----------------------+----------------------------------------+
   | Subscription          | <User Unique>                          |
   +-----------------------+----------------------------------------+
   | Resource Group        | Existing: bigipuser<student number>-aa |
   +-----------------------+----------------------------------------+
   | Resource Group        | bigipuser<student number>-aa           |
   +-----------------------+----------------------------------------+
   | Location              | <Closest Azure DC>                     |
   +-----------------------+----------------------------------------+

   .. image:: /_static/image120.png
      :scale: 50 %

#. Click **OK** at the bottom of the page

   Use the information in Table 3.5 to complete the “Choose a size” configuration
   page during this deployment.

   Table 3.5

   +-------------+------------+
   | Key         | Value      |
   +=============+============+
   | Disk Type   | HDD        |
   +-------------+------------+
   | Size        | A1 Basic   |
   +-------------+------------+

#. Choose **A1 Basic**

   .. image:: /_static/image35.png
      :scale: 50 %

#. Click **Select**

   .. NOTE::
      On the Settings page you’ll see a warning concerning the VM size
      chosen.

   Use the information in Table 3.6 to complete the “Settings”
   configuration page during this deployment.

   Table 3.6

   +---------------------+---------+
   | Key                 | Value   |
   +=====================+=========+
   | Storage Type        | HDD     |
   +---------------------+---------+
   | Use managed disks   | No      |
   +---------------------+---------+

#. Change the "Disk type" to **HDD**
#. Set “Use managed disk” to **No**
#. Keep the other configurations unmodified

   .. image:: /_static/image121.png
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

   .. image:: /_static/image123.png
      :scale: 50 %

   .. image:: /_static/image124.png
      :scale: 50 %

   .. Note::
      Remember the WordPress private and public IP addresses. These will be used
      in subsequent steps.

#. Open a browser and navigate to \http://<wordpress-public-IP>

   .. image:: /_static/image01-wordpress.png
      :scale: 50 %

Task 4 – Configure an F5 BIG-IP Pool and VIP for the WordPress application
--------------------------------------------------------------------------

In this task you will configure the BIG-IP with a Virtual Server and
Pool to allow inbound Internet access to the WordPress application. Let's go back
to the Microsoft Azure Portal.

.. Hint::
   We will need the private IP address for the WordPress instance from Task 3.

#. Connect to the BIG-IP0 using \https://<public-IP>:8443
#. From the BIG-IP GUI, go to **Local traffic -> Pools -> Pool List** and
   click on the **+** sign. Configure the pool using the information provided
   in Table 3.7 below leaving all other fields set to defaults.

   Table 3.7

   +-------------------+---------------------------------------+
   | Key               | Value                                 |
   +===================+=======================================+
   | Name              | wordpress_pool                        |
   +-------------------+---------------------------------------+
   | Health Montitor   | http_head_f5                          |
   +-------------------+---------------------------------------+
   | Node Name         | wordpress                             |
   +-------------------+---------------------------------------+
   | Address           | <your WordPress private IP address>   |
   +-------------------+---------------------------------------+
   | Service Port      | 80                                    |
   +-------------------+---------------------------------------+

   .. image:: /_static/image126.png
      :scale: 50 %

#. Click **Finished**. When configured correctly, the pool status will be green.

   .. image:: /_static/image127.png
      :scale: 50 %

#. Check that the pool is synced to BIG-IP1 (same public IP, port 8444).

   - Connect to the BIG-IP1 using \https://<public-IP>:8444
   - From the BIG-IP GUI, go to **Local traffic -> Pools -> Pool List**
   - Verify that new pool "wordpress_pool" is successfully synced

   .. Hint::
      The Azure Load Balancer NATs HTTPS traffic 80 -> **8081**. This is why
      the F5 virtual server you will create below listens on port 8081.

   Now that the pool is created, you need a virtual server to accept traffic
   and load balance to the pool. You will need to go back to BIG-IP0 (port 8443)
   and create a transparent virtual server. A transparent virtual server is a
   special type of VS that listens on all IP addresses/ports and looks like 0.0.0.0/0.

#. Open or change browsers back to BIG-IP0 (port 8443)
#. Create a virtual server by going to
   **Local Traffic -> Virtual Servers -> Virtual Server List** and click
   on the **+** sign. Configure the Virtual Server using the information
   provided in Table 3.8 below leaving all other fields set to defaults.

   Table 3.8

   +------------------------------+-------------------+
   | Key                          | Value             |
   +==============================+===================+
   | Name                         | vs_wordpress      |
   +------------------------------+-------------------+
   | Destination Address          | 0.0.0.0/0         |
   +------------------------------+-------------------+
   | Service Port                 | 8081              |
   +------------------------------+-------------------+
   | Source Address Translation   | Auto Map          |
   +------------------------------+-------------------+
   | Default Pool                 | wordpress_pool    |
   +------------------------------+-------------------+

   .. image:: /_static/image128.png
      :scale: 50 %

   .. image:: /_static/image129.png
      :scale: 50 %

#. Click **Finish**
#. Check the virtual server is green

   .. image:: /_static/image130.png
      :scale: 50 %

#. Check that the virtual server is synced to BIG-IP1 (same public IP, port 8444).

   - Connect to the BIG-IP1 using \https://<public-IP>:8444
   - From the BIG-IP GUI, go to **Local traffic -> Virtual Servers -> Virtual Server List**
   - Verify that new virtual server "vs_wordpress" is successfully synced

   .. Note::
      Since the virtual server is listening on port 8081, make sure
      the network security group for the Azure F5 instance allows
      port 8081. We'll take care of that in the next steps.

#. Go back to **Resource groups** and click on your resource group
#. Select your F5 Network security group

   .. image:: /_static/lab03-arm07.png

#. Select **Inbound security rules** and review the current ruleset

   .. image:: /_static/lab03-arm08.png

#. Click **+ Add**

   Using the information provided in Table 3.9, add a rule to allow traffic to
   the new application on port 8081.

   Table 3.9

   +--------------------+-------------------+
   | Key                | Value             |
   +====================+===================+
   | Source             | Any               |
   +--------------------+-------------------+
   | Source Port        | \*                |
   +--------------------+-------------------+
   | Destination        | Any               |
   +--------------------+-------------------+
   | Destination Port   | 8081              |
   +--------------------+-------------------+
   | Protocol           | Any               |
   +--------------------+-------------------+
   | Action             | Allow             |
   +--------------------+-------------------+
   | Priority           | 103               |
   +--------------------+-------------------+
   | Name               | f5-allow-8081     |
   +--------------------+-------------------+

   .. image:: /_static/lab03-arm09.png

   .. Note::
      This new inbound rule allows the client/server traffic to traverse
      the Azure load balancer, then hit the F5 virtual server on port 8081,
      then load balance to the WordPress server.

#. Click **OK** to complete the rule and to review the results

   .. image:: /_static/lab03-arm10.png

   To verify proper functionality, let's browse the site and
   verify F5 statistics.

#. Open a browser to to \http://<alb-public-IP> and ensure it
   displays your WordPress blog.

   .. image:: /_static/image01-wordpress.png
      :scale: 50 %

#. Now check the statistics of your virtual server to verify traffic flow,
   by navigating to **Statistics -> Module Statistics -> Local Traffic**
#. Under **Statistics Type**, select **Virtual Servers**

   .. image:: /_static/image55.png
      :scale: 50 %

   You have now completed the BIG-IP configuration for the WordPress
   application. There are still some Azure tasks to complete.

Task 5 – Restrict access to WordPress through the F5 BIG-IP only
----------------------------------------------------------------

This task involves locking down the network security group to restrict
access to the WordPress application. We only want to allow access
through the F5. Currently access still works direct to the WordPress
application via public IP address \http://<wordpress-public-IP> as
demonstrated in the previous task. You now need to modify
the network security group to remove direct inbound access to the
WordPress application. Let's go back to the Azure Portal.

#. Go to **Resource groups** and click on your resource group
#. Select your WordPress Network security group

   .. image:: /_static/image133.png
      :scale: 50 %

#. Remove the HTTP and HTTPS inbound rules while leaving only SSH access

   .. Note::
      You will only allow web access to the WordPress blog via the F5 BIG-IP.

   .. image:: /_static/image44.png
      :scale: 50 %

#. Click on the **…** link at the far right side of the rule to be deleted

   .. image:: /_static/image45.png
      :scale: 50 %

#. Click **Delete**
#. Confirm the delete action when prompted by clicking **Yes**
#. Now it's time to confirm web access has been restricted to WordPress.
   Open a private browser window (not a normal window...choose **private**)
#. Verify that \https://<WordPress-Public-IP> and \http://<WordPress-Public-IP>
   do *NOT* display the WordPress blog

   .. image:: /_static/image134.png
      :scale: 50 %

   .. Note::
      You should now have an active/active F5 pair running in Azure with the
      WordPress application as a pool member. Traffic should flow successfully
      through the F5. However, traffic should fail if going direct to WordPress.

Task 6 – Test HA within Azure
-----------------------------

In this task you will perform a system failover and validate that the
WordPress application remains available.

#. Open two separate *private* browser windows

   .. Tip::
      Use private browser windows to have both GUIs open at the same time.
      Since both BIG-IPs have the same IP, you cannot have two GUIs opened
      at the same time in one browser (it saves cookies).

#. Connect to the BIG-IP0 using \https://<public-IP>:8443
#. Go to **Device Management -> Devices**

   .. image:: /_static/image135.png
      :scale: 50 %

#. Select **bigip0.azure.local (Self)**

   .. image:: /_static/image136.png
      :scale: 50 %

#. Scroll to the bottom of the page
#. Click **Force Offline**
#. Observe that you can still access WordPress.

   .. Hint::
      Traffic flow is successful because the Azure LB is routing traffic
      to the other F5 instance.

   Try forcing both F5 BIG-IPs offline and access the WordPress site again.

#. Connect to the BIG-IP1 using \https://<public-IP>:8444 and repeat the
   same **Force Offline** procedure

   - Go to **Device Management -> Devices**
   - Select **bigip0.azure.local (Self)**
   - Scroll to the bottom of the page
   - Click **Force Offline**
 
   .. Note::
      Were you able to access WordPress?

   The next test is to bring one or both F5 instances back online and then
   test WordPress again.

   .. Hint::
      You will need to “Release Offline” one of the BIG-IPs.

#. Connect to the BIG-IP1 using \https://<public-IP>:8444
#. Go to **Device Management**
#. Select **bigip1.azure.com (Self)**
#. Scroll to the bottom of the page
#. Click **Release Offline**

   .. image:: /_static/image137.png
      :scale: 50 %

   .. Note::
      Can you access the site now? Check that \http://<public-IP> is available.

   .. image:: /_static/image56.gif
      :scale: 50 %

**This concludes Lab 3**

Appendix – Troubleshooting
--------------------------

[azureuser\@bigip0:Active:In Sync] ~ # tmsh run cm sniff-updates

Listening for commit_id_update on -i internal:h port 6699 (^C to exit)

[10:30:53] bigip0.azure.local (v13.0.0) -> bigip-azure-sync-failover:
UPDATE CID 75.0 (bigip0.azure.local) at 10:30:53 FORCE_SYNC

[10:30:53] 10.10.1.101:42506 -> bigip-azure-sync-failover: SYNC_REQ CID
1.0 (bigip0.azure.local) at 10:25:34

[10:30:53] bigip1.azure.local (v13.0.0) -> bigip-azure-sync-failover:
UPDATE CID 75.0 (bigip0.azure.local) at 10:30:53 FORCE_SYNC

[10:32:27] bigip1.azure.local (v13.0.0) -> bigip-azure-sync-failover:
UPDATE CID 31.0 (bigip1.azure.local) at 10:32:27 FORCE_SYNC

[10:32:27] 10.10.1.100:59354 -> bigip-azure-sync-failover: SYNC_REQ CID
75.0 (bigip0.azure.local) at 10:30:53

[10:32:27] bigip0.azure.local (v13.0.0) -> bigip-azure-sync-failover:
UPDATE CID 31.0 (bigip1.azure.local) at 10:32:27 FORCE_SYNC
