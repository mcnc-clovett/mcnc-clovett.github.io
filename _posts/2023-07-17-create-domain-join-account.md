---
layout: post
title:  "Creating a Domain Join Account in AD"
date:   2023-07-17 00:00:00 -0400
categories: [Services, Active Directory]
tags: [windows, domain, security, active directory]
---
Joining a computer to a domain only requires certain permissions to certain places in Active Directory. Since limiting access as much as possible is crucial to Active Directy security, you should only delegate control to accounts that need it. These would be technician accounts or accounts that are used by programs like SCCM. Here we with cover the proper permissions needed to join/remove a computer from Active Directory (AD).

## Creating an Account
First, either choose an account that already exists, or create one that you will use for joining computers to the domain. We'll use the account **domjoin** for this example.

![Domain Join User](/images/domjoin/01_domjoinuser.png)

## Delegate Control
Now that we have an account to delegate control to, we'll have to apply the appropriate permissions in the right place. By default, all computers that are added to AD go in the **Computers** container. This is where we'll delegate control to the **domjoin** account.

In **AD Users and Computers**, right click on the **Computers** container and click **Delegate Control**.

![Delegate Control](/images/domjoin/02_delegatecontrol.png)

Oh the wizard, click **Next**, then click **Add** to select the account you wish to delegate control to. In this case, the **domjoin** account. Once you have your account added, click **Next**.

![Select Your User](/images/domjoin/03_userselect.png)

Select **Create a custom task to delegate**, then click **Next**.

![Tasks to Delegate](/images/domjoin/04_delegatetask.png)

Select **Only the following objects in the folder:**, then choose **Computer objects**. Ensure to check the box for **Create selected objects in this folder** and **Delete selected objects in this folder**. Now click **Next**.

![Active Directory Object Type](/images/domjoin/05_objecttype.png)

Ensure **General** is checked and choose the following from the list:

- Reset Password
- Read and write account restrictions
- Validated write to DNS host name
- Validated write to service principal name

Then, click **Next**.

![Reset Password & R/W Account Restrictions](/images/domjoin/06_permissions1.png)
![Validated Write to DNS and SPN](/images/domjoin/07_permissions2.png)

On the final page, review your settings and click **Finish**.

![Settings Summary](/images/domjoin/08_finish.png)

## Conclusion
Now that you have delegated control to the account on the **Computers** container, you can use it to add and remove computers on the domain as you normally would.
