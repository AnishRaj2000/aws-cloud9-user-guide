.. Copyright 2010-2018 Amazon.com, Inc. or its affiliates. All Rights Reserved.

   This work is licensed under a Creative Commons Attribution-NonCommercial-ShareAlike 4.0
   International License (the "License"). You may not use this file except in compliance with the
   License. A copy of the License is located at http://creativecommons.org/licenses/by-nc-sa/4.0/.

   This file is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND,
   either express or implied. See the License for the specific language governing permissions and
   limitations under the License.

.. _vpc-settings:

######################################
VPC Settings for |envfirsttitleplural|
######################################

.. meta::
    :description:
        Describes Amazon Virtual Private Cloud (Amazon VPC) requirements for use by certain AWS Cloud9 development environments in an AWS account.

Every |envfirstlong| associated with an |VPClong| (|VPC|) must meet specific 
VPC requirements. These |envplural| include |envec2plural|, as well as |envsshplural| associated with an |EC2| instance that runs 
within a VPC.

* :ref:`vpc-settings-requirements`
* :ref:`vpc-settings-create-vpc`
* :ref:`vpc-settings-create-subnet`

.. _vpc-settings-requirements:

|VPC| Requirements for |AC9|
============================

The |VPC| that |AC9| uses requires the following settings. If you're already familiar with these requirements and just want to quickly create
a compatible VPC, skip ahead to :ref:`vpc-settings-create-vpc`.

.. note:: For these procedures, we recommend you sign in to the |console| and open the |VPC|
   console (https://console.aws.amazon.com/vpc) using credentials for an
   |IAM| administrator user in your AWS account. If you can't do this, check with your AWS account administrator.

#. **The VPC must exist in the AWS account, and that VPC must be in the same AWS Region that AWS Cloud9 will create the EC2 environment in.**

   *Related tasks*

   * **Get the list of VPCs that are available for AWS Cloud9 to use in the account for an AWS Region**: 
     In
     the navigation
     bar of the |VPC| console, choose the AWS Region that |AC9| will create the |env| in. Then choose
     :guilabel:`Your VPCs` in the navigation pane.
   * **Create a VPC for AWS Cloud9 to use**: See :ref:`vpc-settings-create-vpc`.

#. **The VPC must have a public subnet for AWS Cloud9 to use.** A subnet is public if
   its traffic is routed to an internet gateway.

   *Related tasks*

   * **Get the list of subnets for a VPC**: In the |VPC| console, choose :guilabel:`Your VPCs`
     in the navigation pane. Note the VPC's ID in the :guilabel:`VPC ID` column. Then choose :guilabel:`Subnets`,
     and look for subnets that contain that ID in the :guilabel:`VPC` column.
   * **See whether a subnet is public**: In the |VPC| console, choose :guilabel:`Subnets` in the
     navigation pane. Select the box next to the subnet you want |AC9| to use. On the :guilabel:`Route Table` tab,
     if there is an  entry in the :guilabel:`Target` column that starts with :guilabel:`igw-`, the subnet is public.
   * **Create a subnet in a VPC**: In the |VPC| console, choose :guilabel:`Subnets` in the navigation
     pane. Choose :guilabel:`Create Subnet`, and then follow the on-screen directions.
   * **See or change the settings for an internet gateway**: In the |VPC| console, choose :guilabel:`Internet
     Gateways` in the navigation pane. Select the box next to the internet gateway. To see the settings,
     look at each of the tabs. To change a setting on a tab, choose :guilabel:`Edit`, and then follow the on-screen directions.
   * **Create an internet gateway**: In the |VPC| console, choose :guilabel:`Internet Gateways`
     in the navigation pane. Choose :guilabel:`Create Internet Gateway`, and then follow the on-screen directions.
   * **Attach an internet gateway to a VPC**: In the |VPC| console, choose :guilabel:`Internet
     Gateways` in the navigation pane. Select the box next to the internet gateway. Choose :guilabel:`Attach to
     VPC`, and then follow the on-screen directions.

#. **The VPC's public subnet must have a route table, and we recommend that the route table have the following minimum settings.**

   .. list-table::
      :widths: 1 1 1 1
      :header-rows: 1

      * - **Destination**
        - **Target**
        - **Status**
        - **Propagated**
      * - CIDR-BLOCK
        - local
        - Active
        - No
      * - 0.0.0.0/0
        - igw-INTERNET-GATEWAY-ID
        - Active
        - No

   In these settings, :samp:`{CIDR-BLOCK}` is the subnet's CIDR block, and :samp:`igw-{INTERNET-GATEWAY-ID}`
   is the ID of a compatible internet gateway.

   *Related tasks*

   * **See whether the VPC's public subnet has a route table**: In the |VPC| console, choose
     :guilabel:`Subnets` in the navigation pane. Select the box next to the VPC's public subnet that you want |AC9| to use.
     On the :guilabel:`Route table` tab, if there is a value for :guilabel:`Route Table`, the public subnet has a route table.
   * **See or change the settings for a route table**: In the |VPC| console, choose :guilabel:`Route
     Tables` in the navigation pane. Select the box next to the route table. To see the settings, look at each
     of the tabs. To change a setting on a tab, choose :guilabel:`Edit`, and then follow the on-screen directions.
   * **Create a route table**: In the |VPC| console, choose :guilabel:`Route Tables` in the navigation
     pane. Choose :guilabel:`Create Route Table`, and then follow the on-screen directions.

#. **The VPC must allow specific inbound and outbound traffic.**

   At a minimum, the VPC must allow the following inbound and outbound traffic.

   * **Inbound**: All IP addresses using SSH over port 22. However, you can restrict these IP addresses to only those that |AC9| uses. For more information, see 
     :ref:`Inbound SSH IP Address Ranges <ip-ranges>`.
   * **Inbound**: For |envec2plural|, and for |envsshplural| associated with |EC2| instances running Amazon Linux, all IP addresses using TCP over ports 32768-61000. 
     For more information, and for port ranges for other |EC2| instance types, see :vpc-user-guide:`Ephemeral Ports <VPC_ACLs.html#VPC_ACLs_Ephemeral_Ports>` in the |VPC-ug|.
   * **Outbound**: All traffic sources using any protocol and port.

   You can set this behavior at the security group level. For an additional level of security, you can also use a network ACL.

   For example, to add inbound and outbound rules to a security group, you could set up those rules as follows.
   
   Inbound rules:

   .. list-table::
      :widths: 1 1 1 1
      :header-rows: 1

      * - **Type**
        - **Protocol**
        - **Port Range**
        - **Source**
      * - SSH (22)
        - TCP (6)
        - 22
        - 0.0.0.0 
          (But see :ref:`Inbound SSH IP Address Ranges <ip-ranges>`.)
      * - Custom TCP Rule
        - TCP (6)
        - 32768-61000
          (For Amazon Linux instances. For other instance types, see :vpc-user-guide:`Ephemeral Ports <VPC_ACLs.html#VPC_ACLs_Ephemeral_Ports>`.)
        - 0.0.0.0/0

   Outbound rules:

   .. list-table::
      :widths: 1 1 1 1
      :header-rows: 1

      * - **Type**
        - **Protocol**
        - **Port Range**
        - **Source**
      * - ALL Traffic
        - ALL
        - ALL
        - 0.0.0.0/0
   
   If you also choose to add inbound and outbound rules to a network ACL, you could set up those rules as follows.

   Inbound rules:

   .. list-table::
      :widths: 1 1 1 1 1 1
      :header-rows: 1

      * - **Rule #**
        - **Type**
        - **Protocol**
        - **Port Range**
        - **Source**
        - **Allow / Deny**
      * - 100
        - SSH (22)
        - TCP (6)
        - 22
        - 0.0.0.0 
          (But see :ref:`Inbound SSH IP Address Ranges <ip-ranges>`.)
        - ALLOW
      * - 200
        - Custom TCP Rule
        - TCP (6)
        - 32768-61000
          (For Amazon Linux instances. For other instance types, see :vpc-user-guide:`Ephemeral Ports <VPC_ACLs.html#VPC_ACLs_Ephemeral_Ports>`.)
        - 0.0.0.0/0
        - ALLOW  
      * - :code:`*`
        - ALL Traffic
        - ALL
        - ALL
        - 0.0.0.0/0
        - DENY

   Outbound rules:

   .. list-table::
      :widths: 1 1 1 1 1 1
      :header-rows: 1

      * - **Rule #**
        - **Type**
        - **Protocol**
        - **Port Range**
        - **Source**
        - **Allow / Deny**
      * - 100
        - ALL Traffic
        - ALL
        - ALL
        - 0.0.0.0/0
        - ALLOW
      * - :code:`*`
        - ALL Traffic
        - ALL
        - ALL
        - 0.0.0.0/0
        - DENY
   
   For more information about security groups and network ACLs, see the following in the |VPC-ug|.

   * :VPC-ug:`Security <VPC_Security>`
   * :VPC-ug:`Security Groups for your VPC <VPC_SecurityGroups>`
   * :VPC-ug:`Network ACLs <VPC_ACLs>`

   *Related tasks*

   * **See whether the VPC has at least one network ACL**: In the |VPC| console, choose
     :guilabel:`Your VPCs` in the navigation pane. Select the box next to the VPC you want |AC9| to use. On the :guilabel:`Summary` tab, if there is a value for
     :guilabel:`Network ACL`, the VPC has at least one network ACL.
   * **See all network ACLs for a VPC**: In the |VPC| console, choose
     :guilabel:`Network ACLs` in the navigation pane. In the :guilabel:`Search Network ACLs` box, 
     type the VPC's ID or name. Network ACLs for that VPC appear in the list of search results.
   * **See or change the settings for a network ACL**: In the |VPC| console, choose :guilabel:`Network
     ACLs` in the navigation pane. Select the box next to the network ACL. To see the settings, look at
     each of the tabs. To change a setting on a tab, choose :guilabel:`Edit`, and then follow the on-screen directions.
   * **Create a network ACL**: In the |VPC| console, choose :guilabel:`Network ACLs` in the navigation
     pane. Choose :guilabel:`Create Network ACL`, and then follow the on-screen directions.
   * **See all security groups for a VPC**: In the |VPC| console, choose :guilabel:`Security Groups` in the navigation pane. In the :guilabel:`Search Security Groups` box, 
     type the VPC's ID or name. Security groups for that VPC appear in the list of search results.
   * **See or change the settings for a security group**: In the |VPC| console, choose :guilabel:`Security Groups` in the navigation pane. Select the box next to the security group. 
     To see the settings, look at each of the tabs. To change a setting on a tab, choose :guilabel:`Edit`, and then follow the on-screen directions.
   * **Create a security group**: In the |VPC| console, choose :guilabel:`Security Groups` in the navigation pane. Choose :guilabel:`Create Security Group`, and then follow the on-screen directions.

.. _vpc-settings-create-vpc:

Create an |VPC| for |AC9|
=========================

You can use the |VPC| console to create an |VPC| that is compatible with |AC9|.

.. note:: For this procedure, we recommend you sign in to the |console| and open the |VPC| console using credentials for an |IAM|
   administrator user in your AWS account. If you can't do this, check with your AWS account administrator.

   Some organizations may not allow you to create VPCs on your own. If you cannot create a VPC, check with your AWS account administrator or network administrator.

#. If the |VPC| console isn't already open, sign in to the |console| and open the |VPC| console at https://console.aws.amazon.com/vpc.
#. In the navigation bar, if the AWS Region isn't the same as the |env|, choose
   the correct AWS Region.
#. Choose :guilabel:`VPC Dashboard` in
   the navigation pane, if the :guilabel:`VPC Dashboard` page isn't already displayed.
#. Choose :guilabel:`Start VPC Wizard`.
#. For :guilabel:`Step 1: Select a VPC Configuration`, with :guilabel:`VPC with a Single Public Subnet` already selected, choose :guilabel:`Select`.
#. For :guilabel:`Step 2: VPC with a Single Public Subnet`, we recommend that you leave the following default settings. (However, you can change the CIDR settings if
   you have custom CIDRs you want to use. For more information, see :vpc-user-guide:`VPC and Subnet Sizing <VPC_Subnets.html#VPC_Sizing>` in the |VPC-ug|.)

   * :guilabel:`IPv4 CIDR block`: :guilabel:`10.0.0.0/16`
   * :guilabel:`IPv6 CIDR block`: :guilabel:`No IPv6 CIDR Block`
   * :guilabel:`Public subnet's IPv4 CIDR`: :guilabel:`10.0.0.0/24`
   * :guilabel:`Availability Zone`: :guilabel:`No Preference`
   * :guilabel:`Enable DNS hostnames`: :guilabel:`Yes`
   * :guilabel:`Hardware tenancy`: :guilabel:`Default`

#. For :guilabel:`VPC name`, type a name for the VPC.
#. For :guilabel:`Subnet name`, type a name for the subnet in the VPC.
#. Choose :guilabel:`Create new VPC`.

   |VPC| creates the following resources that are compatible with |AC9|:

   * A VPC.
   * A public subnet for the VPC.
   * A route table for the public subnet with the minimum required settings.
   * An internet gateway for the public subnet.
   * A network ACL for the public subnet with the minimum required settings.

#. By default, the VPC allows incoming traffic from all types, protocols, ports, and IP addresses. 
   You can restrict this behavior to allow only IP addresses coming from |AC9| using SSH over port 22. One approach is to 
   set incoming rules on the VPC's default network ACL, as follows.

   #. In the navigation pane of the |VPC| console, choose :guilabel:`Your VPCs`.
   #. Select the box for the VPC you just created.
   #. On the :guilabel:`Summary` tab, choose the link next to :guilabel:`Network ACL`.
   #. Select the box next to the network ACL that is displayed.
   #. On the :guilabel:`Inbound Rules` tab, choose :guilabel:`Edit`.
   #. For :guilabel:`Rule # 100`, for :guilabel:`Type`, choose :guilabel:`SSH (22)`.
   #. For :guilabel:`Source`, type one of the CIDR blocks in the :ref:`Inbound SSH IP Address Ranges <ip-ranges>` list that matches the AWS Region for this VPC.
   #. Choose :guilabel:`Add another rule`.
   #. For :guilabel:`Rule #`, type :code:`200`.
   #. For :guilabel:`Type`, choose :guilabel:`SSH (22)`. 
   #. For :guilabel:`Source`, type the other CIDR block in the :ref:`Inbound SSH IP Address Ranges <ip-ranges>` list that matches the AWS Region for this VPC.
   #. At minimum, you must also allow incoming traffic from all IP addresses using TCP over ports 32768-61000 for Amazon Linux instance types. 
      (For background, and for port ranges for other |EC2| instance types, see :vpc-user-guide:`Ephemeral Ports <VPC_ACLs.html#VPC_ACLs_Ephemeral_Ports>` in the |VPC-ug|). To do this, choose :guilabel:`Add another rule`.
   #. For :guilabel:`Rule #`, type :code:`300`.
   #. For :guilabel:`Type`, choose :guilabel:`Custom TCP Rule`.
   #. For :guilabel:`Port Range`, type :code:`32768-61000` (for Amazon Linux instance types).
   #. For :guilabel:`Source`, type :code:`0.0.0.0/0`.
   #. Choose :guilabel:`Save`.
   #. You might need to add more inbound or outbound rules to the network ACL, depending on how you plan to use |AC9|. See the documentation for the 
      web services or APIs you want to allow to communicate into or out of the VPC for the :guilabel:`Type`, :guilabel:`Protocol`, :guilabel:`Port Range`, 
      and :guilabel:`Source` values to specify for these rules.

.. _vpc-settings-create-subnet:

Create a Subnet for |AC9|
=========================

You can use the |VPC| console to create a subnet for a VPC that is compatible with |AC9|.

If you followed the previous procedure, you do not also need to follow this procedure. This is because the :guilabel:`Create new VPC` wizard creates a subnet for you 
automatically.

.. important::

   * The AWS account must already have a compatible VPC in the same AWS Region for the |env|. For
     more information, see the VPC requirements in :ref:`vpc-settings-requirements`.
   * For this procedure, we recommend you sign in to the |console|, and then open the |VPC| console using
     credentials for an |IAM|
     administrator user in your AWS account. If you can't do this, check with your AWS account administrator.
   * Some organizations may not allow you to create subnets on your own. If you cannot create a subnet, check with your AWS account administrator or network administrator.

#. If the |VPC| console isn't already open, sign in to the |console| and open the |VPC| console at https://console.aws.amazon.com/vpc.
#. In the navigation bar, if the AWS Region isn't the same as the AWS Region for the |env|, choose
   the correct AWS Region.
#. Choose :guilabel:`Subnets` in the navigation
   pane, if the :guilabel:`Subnets` page isn't already displayed.
#. Choose :guilabel:`Create Subnet`.
#. In the :guilabel:`Create Subnet` dialog box, for :guilabel:`Name tag`, type a name for the subnet.
#. For :guilabel:`VPC`, choose the VPC to associate the subnet with.
#. For :guilabel:`Availability Zone`, choose the Availability Zone within the AWS Region for the subnet to use, or choose :guilabel:`No Preference` to let AWS choose an Availability Zone for you.
#. For :guilabel:`IPv4 CIDR block`, type the range of IP addresses for the subnet to use, in CIDR format. This range of IP addresses must be a subset of IP addresses in the VPC.

   For information about CIDR blocks, see :vpc-user-guide:`VPC and Subnet Sizing <VPC_Subnets.html#VPC_Sizing>` in the |VPC-ug|.
   See also `3.1. Basic Concept and Prefix Notation <http://tools.ietf.org/html/rfc4632#section-3.1>`_ in RFC 4632 or
   `IPv4 CIDR blocks <http://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing#IPv4_CIDR_blocks>`_ in Wikipedia.

#. After you create the subnet, be sure to associate it with a compatible route table and an internet gateway, as well as security groups, a network ACL, or both. For more information, see the requirements in :ref:`vpc-settings-requirements`.
