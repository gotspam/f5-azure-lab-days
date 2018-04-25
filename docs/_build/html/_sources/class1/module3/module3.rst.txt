Module 3 – Deploy an F5 BIG-IP Active/Active HA Pair Using ARM Templates
========================================================================

In this lab you will be deploying an active/active pair of F5 BIG-IPs
in Azure using the Azure Resource Manager (ARM) templates. ARM templates
automate many of the deployment steps and ensure a validated topology
and configuration of your F5 BIG-IP instances within Azure.

To see a list of all F5 Azure ARM templates, go to
https://github.com/F5Networks/f5-azure-arm-templates. Take your time
to review the README.md.

.. Tip::
   The supported templates are fully supported by F5, and customers
   can call for issues. The experimental folder contains other
   template options.

For this lab, we will be using a template from the following location:

https://github.com/F5Networks/f5-azure-arm-templates/tree/master/supported/cluster/failover-lb/1nic/new_stack

It's recommended to review the README.md on this page as well to become
familiar with the template, prerequisites, topology, and more.
**Lab 3 – Topology**

.. image:: /_static/lab-3-topology.png
.. toctree::
   :maxdepth: 1
   :glob:

   lab*
