Lab 1 – Deploy a Standalone F5 BIG-IP Application Delivery Controller in Azure
------------------------------------------------------------------------------

This lab will teach you how to manually install a |bip| |ve| in your Azure cloud environment.

Task 1 - Create an SSH Key Pair
-------------------------------

Before you begin the deployment process, you first need to generate an
SSH Key Pair which will be used for authentication to the F5 BIG-IPs in
this and subsequent labs.

.. HINT::
   Make sure to save both public and private keys (do not protect with passphrase) in an
   easily accessible place on your laptop.

**For Linux / Mac Users:**

A terminal window or CLI is the preferred method.

#. Open the CLI and run ``ssh-keygen -t rsa``
#. Enter file location ``/tmp/azure``
#. Enter passphrase if needed (empty for no passphrase)

Result:

-  Your private key has been saved in /tmp/azure
-  Your public key has been saved in /tmp/azure.pub

Example private RSA key Linux / Mac:

   .. image:: /_static/image3.png
      :scale: 50 %

Example public RSA key Linux / Mac:

   .. image:: /_static/image4.png
      :scale: 50 %

**For Windows Users:**

The PuTTY Key Generator is the preferred method. This is part of the PuTTY toolset.

#. Open PuTTYgen
#. Click **Generate**

   .. image:: /_static/image5.png
      :scale: 50 %

#. Click **Save public key** and **Save private key**

   .. image:: /_static/image6.png
      :scale: 50 %

Example public RSA key Windows:

   .. image:: /_static/image7.png
      :scale: 50 %

.. NOTE::
   In future steps, making a connection to WordPress or |bip| via SSH will
   use the private key in PuTTY for authentication.

Task 2 - Deploy a new F5 BIG-IP VE in Azure
-------------------------------------------

In this task you will deploy a new Azure Resource Group, F5 BIG-IP VE,
and other supporting configuration items. Below is a list of items
created during this step.

-  Resource Group
-  Virtual Machine for BIG-IP
-  Network Interface for BIG-IP
-  Public IP Address for BIG-IP
-  Network Security Group
-  Storage Account
-  Virtual Network

Let's get started.

#. Log into the portal – https://portal.azure.com
#. Click the green **+** sign at the top left corner of the screen
#. Search the marketplace by typing 'F5' in the search field and hit **Enter**.
   Take your time to view the different F5 products available.

#. Click **F5 BIG-IP ADC BETTER – BYOL**

   .. image:: /_static/image9.png
      :scale: 50 %

   .. NOTE::
      An appropriate license is delivered by your lab proctor. The proctor
      will explain why you use **BYOL** in this lab.

#. At the bottom of the next page, selected **Resource Manager** as the
   deployment model and not **Classic**

   .. image:: /_static/image10.png
      :scale: 50 %

#. Click **Create**

   You will now start the deployment process. Use the information provided
   in Table 1.1 below to complete the “Create virtual machine” Basics page.

   Table 1.1

   +-----------------------+----------------------------------------+
   | Key                   | Value                                  |
   +=======================+========================================+
   | BIG-IP Image          | F5 BIG-IP ADC BETTER – BYOL            |
   +-----------------------+----------------------------------------+
   | Deployment Model      | Resource Manager                       |
   +-----------------------+----------------------------------------+
   | Name                  | f5bigipuser<student number>bigip1      |
   +-----------------------+----------------------------------------+
   | VM disk type          | SSD                                    |
   +-----------------------+----------------------------------------+
   | User name             | f5bigipuser<student number>            |
   +-----------------------+----------------------------------------+
   | Authentication Type   | SSH public key                         |
   +-----------------------+----------------------------------------+
   | SSH public key        | From Lab1 Step 1                       |
   +-----------------------+----------------------------------------+
   | Subscription          | <User Unique>                          |
   +-----------------------+----------------------------------------+
   | Resource group        | Create new                             |
   +-----------------------+----------------------------------------+
   | Resource group name   | f5bigipuser<student number>usergroup   |
   +-----------------------+----------------------------------------+
   | Location              | <Closest Azure DC>                     |
   +-----------------------+----------------------------------------+

   Example:

   .. image:: /_static/image11.png
      :scale: 50 %

#. Once done, click **OK**

   You now need to select the Virtual Machine disk type and image size.
   Using the information in Table 1.2 complete the “Size” page.

   .. NOTE::
      For a complete list of compatible Azure instance sizes, refer to
      the “BIG-IP Virtual Edition and Microsoft Azure: Setup” guide.

   Table 1.2

   +-------------+-------------------+
   | Key         | Value             |
   +=============+===================+
   | Disk Type   | HHD               |
   +-------------+-------------------+
   | Size        | D2\_V2 Standard   |
   +-------------+-------------------+

#. Select **HDD** from “Supported disk type”
#. Then select **View all** to browse the available VM sizes and features

   .. image:: /_static/image12.png
      :scale: 50 %

#. Select **D2\_V2 Standard**

   .. image:: /_static/image13.png
      :scale: 50 %

#. Click **Select**

   In the “Settings” page, provide the remaining information required for
   the BIG-IP deployment and associated resources. Using the information in
   Table 1.3 to complete the “Settings” page.

   Table 1.3

   +---------------------+---------+
   | Key                 | Value   |
   +=====================+=========+
   | Storage Type        | HHD     |
   +---------------------+---------+
   | Use managed disks   | No      |
   +---------------------+---------+

#. Under Settings, change "Disk type" to **HDD** and "Use managed disk" to **No**

   Look around at the various configurable items but leave them unchanged.

   .. image:: /_static/image14.png
      :scale: 50 %

#. Once done, click **OK**
#. Review the "Summary" page and the purchase you are about to make

   .. Note:: In the screenshot below:

      -  Notice “Validation passed”
      -  Notice the F5 license BYOL is *not* charged
      -  Notice the VM where the BIG-IP VE will reside is charged.

   .. image:: /_static/image15.png
      :scale: 50 %

#. Click **Purchase** or **Create**

Task 3 - Allow management and HTTP access to the BIG-IP
-------------------------------------------------------

In this task you will permit management access and HTTPS access to the
BIG-IP by modifying the Network Security Group “Inbound” network access
rule set.

#. Go to **Resource groups**

   .. image:: /_static/image16.png
      :scale: 50 %

#. Expand your Resource group and select the Network security group

   .. image:: /_static/image17.png
      :scale: 50 %

#. Review the existing ruleset. Notice that you only have an inbound
   rule allowing SSH.

   .. image:: /_static/image18.png
      :scale: 50 %

   Now you will add rules to allow HTTPS for F5 BIG-IP management and
   data plane by clicking on “Inbound security rules”
   (to the left of the screen below).

#. Click **Inbound security rules**

   .. image:: /_static/image19.png
      :scale: 50 %

#. Click **+ Add**

   .. image:: /_static/image20.png
      :scale: 50 %

   Using the information provided in Table 1.4, add a rule to allow F5
   BIG-IP management traffic.

   Table 1.4

   +--------------------+-------------------+
   | Key                | Value             |
   +====================+===================+
   | Source             | Any               |
   +--------------------+-------------------+
   | Source Port        | \*                |
   +--------------------+-------------------+
   | Destination        | Any               |
   +--------------------+-------------------+
   | Destination Port   | 8443              |
   +--------------------+-------------------+
   | Protocol           | Any               |
   +--------------------+-------------------+
   | Action             | Allow             |
   +--------------------+-------------------+
   | Priority           | 100               |
   +--------------------+-------------------+
   | Name               | f5-allow-mgmt     |
   +--------------------+-------------------+

   .. image:: /_static/image21.png
      :scale: 50 %

#. Click **OK**
#. Repeat the previous step to add another rule using the information
   provided in Table 1.5, this time allowing external HTTPS traffic via the
   F5 BIG-IP.

   Table 1.5

   +--------------------+---------------------------+
   | Key                | Value                     |
   +====================+===========================+
   | Source             | Any                       |
   +--------------------+---------------------------+
   | Source Port        | \*                        |
   +--------------------+---------------------------+
   | Destination        | Any                       |
   +--------------------+---------------------------+
   | Destination Port   | 443                       |
   +--------------------+---------------------------+
   | Protocol           | Any                       |
   +--------------------+---------------------------+
   | Action             | Allow                     |
   +--------------------+---------------------------+
   | Priority           | 101                       |
   +--------------------+---------------------------+
   | Name               | f5-allow-external-https   |
   +--------------------+---------------------------+

#. When complete, verify the end results look as follows:

   .. image:: /_static/image22.png
      :scale: 50 %

Task 4 - License and Apply Base BIG-IP Configuration
----------------------------------------------------

In this task you will connect to the BIG-IP CLI and GUI, license the
device, and complete a base configuration. First, you need to identify
the BIG-IP's public IP address to which you will connect.

#. Return to the **Resource group**
#. Select **Network Interface** to see the F5 BIG-IP's private
   and public IP addresses

   .. image:: /_static/image23.png
      :scale: 50 %

   .. Note::
      Remember the F5 BIG-IP's public IP address. This will be used in
      subsequent steps.

   .. image:: /_static/image24.png
      :scale: 50 %

#. Wait until the deployment is completed. To view status, click on the
   bell symbol in the upper right corner of the screen. You will see
   “Deployments succeeded” under “Notifications”.

   .. image:: /_static/image25.png
      :scale: 50 %

   You now need to connect to the F5 BIG-IP CLI in order to license the F5
   BIG-IP, configure the hostname, create an admin account, and set the
   password.

#. SSH to the F5 public IP address

   **Connectivity for Linux / Mac Users:**

   - Open the CLI
   - Connect using ``ssh -i <private_key> f5bigipuser<Student Number>@<F5
     BIG-IP public IP>``

   .. Note::
      The reference to **private_key** is the file corresponding to the
      public key created during BIG-IP deployment by Azure. The f5bigipuserx
      is the user you created during the same step (“Create virtual machine/Basics”).

   Example:

   .. image:: /_static/image26.png
      :scale: 50 %

   **Connectivity for Windows Users:**

   - Open PuTTY
   - Enter the public IP
   - Go to **Connection -> SSH -> Auth**
   - Browse to the location of your private key
   - Select **Open** to start the connection

   .. image:: /_static/image8.png
      :scale: 50 %

#. License your F5 BIG-IP by typing ``SOAPLicenseClient --basekey <license>``

   Example:

   .. image:: /_static/image27.png
      :scale: 50 %

   .. Note::
      License key is provided by your instructor.

#. Change the hostname and replace x with the number assigned by
   your proctor

   .. admonition:: TMSH

      tmsh modify sys global-settings hostname f5bigipuserx.azure.local

   Example:

   .. image:: /_static/image28.png
      :scale: 50 %

#. Change the password for f5bigipuserx and replace x with the
   number assigned by your proctor

   .. admonition:: TMSH 

      tmsh modify auth user f5bigipuserx password "Demo123"

   Example:

   .. image:: /_static/image29.png
      :scale: 50 %

#. Wait until the system prompt changes to the following:

   .. code-block:: bash

      [f5bigipuserx@f5bigipuserx:Active:Standalone] ~ #

   .. WARNING::
      Changes made in the CLI are not present in the running configuration
      until they are saved.

#. Save the system configuration

   .. admonition:: TMSH 

      tmsh save sys config

   Example:

   .. image:: /_static/image30.png
      :scale: 50 %

#. Open your favorite web browser
#. Connect to the F5 GUI by going to \https://<F5-BIG-IP-public-IP>:8443
#. Accept the SSL certificate warning
#. Log into the BIG-IP using the credentials configured in the previous steps

   .. image:: /_static/image31.png
      :scale: 50 %

#. Click **Log in**

Task 5 - Deploy and configure WordPress within Azure
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

   Use the information in Table 1.6 to complete the “Basics” configuration
   page during this deployment.

   Table 1.6

   +-----------------------+---------------------------------------------+
   | Key                   | Value                                       |
   +=======================+=============================================+
   | Name                  | f5bigipuser<student number>wordpress        |
   +-----------------------+---------------------------------------------+
   | VM disk type          | SSD                                         |
   +-----------------------+---------------------------------------------+
   | User name             | f5bigipuser<student number>                 |
   +-----------------------+---------------------------------------------+
   | Authentication type   | SSH public key                              |
   +-----------------------+---------------------------------------------+
   | SSH public key        | From Lab 1 Step 1                           |
   +-----------------------+---------------------------------------------+
   | Subscription          | <User Unique>                               |
   +-----------------------+---------------------------------------------+
   | Resource Group        | Use existing previously created in step 1   |
   +-----------------------+---------------------------------------------+
   | Location              | <Closest Azure DC>                          |
   +-----------------------+---------------------------------------------+

   .. image:: /_static/image34.png
      :scale: 50 %

#. Click **OK** at the bottom of the page

   Use the information in Table 1.7 to complete the “Choose a size” configuration
   page during this deployment.

   Table 1.7

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

   Use the information in Table 1.8 to complete the “Settings” configuration
   page during this deployment.

   .. NOTE::
      On the Settings page you’ll see a warning concerning the VM size
      chosen.

   Table 1.8

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

   .. image:: /_static/image36.png
      :scale: 50 %

#. Click **OK**
#. Verify the summary

   .. image:: /_static/image37.png
      :scale: 50 %

#. Click **Create**
#. Go to **Resource groups** and click on your resource group
#. Select your WordPress “Public IP address”

   .. image:: /_static/image38.png
      :scale: 50 %

   .. image:: /_static/image39.png
      :scale: 50 %

   .. Note::
      Remember the WordPress public IP address. This will be used in
      subsequent steps.

   We now want to configure the WordPress system to accept HTTPS traffic only.

#. SSH to the WordPress public IP address

   **For Linux / Mac Users:**

   - Open the CLI
   - Connect using ``ssh -i <private_key> f5bigipuser<Student Number>@@<WordPress VM public IP>``

   .. Note::
      The reference to **private_key** is the file corresponding to the
      public key created during Wordpress deployment by Azure.

   Example:

   .. image:: /_static/image40.png
      :scale: 50 %

   **For Windows Users:**

   - Open PuTTY
   - Enter the public IP
   - Go to **Connection -> SSH -> Auth**
   - Browse to the location of your private key
   - Select **Open** to start the connection

   .. image:: /_static/image8.png
      :scale: 50 %

#. Modify the wp-config.php file with a text editor (e.g. vim)

   .. code-block:: bash

      sudo vim /opt/bitnami/apps/wordpress/htdocs/wp-config.php

#. In the vim editor, type ``/SITEURL`` to search and jump to the
   2 lines you need to modify.

   .. Note::
      The "/" followed by text will do a "Find" function. You are
      essentially trying to find references of SITEURL. The lines
      you are going to modify are the following two lines:

      define('WP_SITEURL', '\http://' . $_SERVER['HTTP_HOST'] . '/');

      define('WP_HOME', '\http://' . $_SERVER['HTTP_HOST'] . '/');

#. Type “i” to enter “edit/insert mode” in the vim editor
#. Change http to https. The end results should look as follows:

   .. image:: /_static/image41.png
      :scale: 50 %

#. Hit the **Escape** key
#. Then type the key sequence **:wq** to tell vim to quit
#. Hit **Enter** when asked to save and exit
#. Restart Apache for the changes to take effect

   .. code-block:: bash

      sudo /opt/bitnami/ctlscript.sh restart apache

#. Verify that \https://<WordPress-Public-IP> displays the
   Wordpress blog

   - You may have to accept the security warning

   .. image:: /_static/image42.png
      :scale: 50 %

   You now need to modify the Network security group to remove direct
   inbound access to the WordPress application. Let's go back to the
   Microsoft Azure portal.

#. Go to **Resource groups** and click on your resource group
#. Select your WordPress Network security group

   .. image:: /_static/image43.png
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

   .. image:: /_static/image46.png
      :scale: 50 %

Task 6 - Allow Internet access to WordPress through the BIG-IP
--------------------------------------------------------------

In this task you will configure the BIG-IP with a Virtual Server and
Pool to allow inbound Internet access to the WordPress application. First we
need to identify the private IP address for the WordPress instance. Let's go
back to the Microsoft Azure Portal.

#. Select your WordPress Network Interface

   .. image:: /_static/image47.png
      :scale: 50 %

   .. Note::
      Remember WordPress private IP address. This will be used in
      subsequent steps.

   .. image:: /_static/image48.png
      :scale: 50 %

   This completes work in the Microsoft Azure Portal. You will now
   configure the F5 BIG-IP.

#. Connect to the BIG-IP using \https://<F5-public-IP>:8443
#. From the BIG-IP GUI, go to **Local traffic -> Pools -> Pool List** and
   click on the **+** sign. Configure the pool using the information
   provided in Table 1.8 below leaving all other fields set to defaults.

   Table 1.8

   +-------------------+---------------------------------------+
   | Key               | Value                                 |
   +===================+=======================================+
   | Name              | wordpress_pool                        |
   +-------------------+---------------------------------------+
   | Health Montitor   | HTTPS                                 |
   +-------------------+---------------------------------------+
   | Node Name         | wordpress                             |
   +-------------------+---------------------------------------+
   | Address           | <your WordPress private IP address>   |
   +-------------------+---------------------------------------+
   | Service Port      | 443                                   |
   +-------------------+---------------------------------------+

   .. image:: /_static/image49.png
      :scale: 50 %

#. Click **Finished**. When configured correctly, the pool status will be green.

   .. image:: /_static/image50.png
      :scale: 50 %

   You now need to configure the Virtual server. To do this, you first need to
   find the private IP of your F5 BIG-IP.

#. From the BIG-IP GUI, go to **Network -> Self IPs** and note the IP Address

   .. image:: /_static/image51.png
      :scale: 50 %

#. Create a virtual server by going to
   **Local Traffic -> Virtual Servers -> Virtual Server List** and click
   on the **+** sign. Configure the Virtual Server using the information
   provided in Table 1.9 below leaving all other fields set to defaults.

   Table 1.9

   +------------------------------+-----------------------------------+
   | Key                          | Value                             |
   +==============================+===================================+
   | Name                         | vs_wordpress                      |
   +------------------------------+-----------------------------------+
   | Destination Address          | <Self IP address of the BIG-IP>   |
   +------------------------------+-----------------------------------+
   | Service Port                 | 443                               |
   +------------------------------+-----------------------------------+
   | SSL Profile (Client)         | clientssl                         |
   +------------------------------+-----------------------------------+
   | SSL Profile (Server)         | serverssl                         |
   +------------------------------+-----------------------------------+
   | Source Address Translation   | Auto Map                          |
   +------------------------------+-----------------------------------+
   | Default Pool                 | wordpress_pool                    |
   +------------------------------+-----------------------------------+

   .. image:: /_static/image52.png
      :scale: 50 %

   .. image:: /_static/image53.png
      :scale: 50 %

#. Click **Finish**

   You have now completed the BIG-IP configuration for the WordPress
   application. To verify proper functionality, let's browse the site and
   verify F5 statistics.

#. Open a browser to to \https://<F5-public-VIP-IP> and ensure it
   displays your WordPress blog.

   .. NOTE::
      As part of this task, you will see a certificate warning. You can
      ignore this as in this lab you did not generate the server certificates.
      In real life, you would ensure you have installed valid certificates.

   .. image:: /_static/image54.png
      :scale: 50 %

#. Now check the statistics of your virtual server to verify traffic flow,
   by navigating to **Statistics -> Module Statistics -> Local Traffic**
   Under **Statistics Type**, select **Virtual Servers**

   .. image:: /_static/image55.png
      :scale: 50 %

   .. image:: /_static/image56.gif
      :scale: 50 %

**This concludes Lab 1**
