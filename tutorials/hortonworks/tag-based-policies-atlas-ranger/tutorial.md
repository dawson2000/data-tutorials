---
layout: tutorial
title: Tag Based Policies with Apache Ranger and Apache Atlas
tutorial-id: 660
tutorial-series: Security and Governance
tutorial-version: hdp-2.5.0
intro-page: true
components: [ atlas, ranger ]
---

## Introduction

Hortonworks has recently announced the integration of Apache Atlas and Apache Ranger, and introduced the concept of tag or classification based policies. Enterprises can classify data in Apache Atlas and use the classification to build security policies in Apache Ranger.

This tutorial walks through an example of tagging data in Atlas and building a security policy in Ranger.

## Prerequisites

- [Download Hortonworks 2.5 Sandbox Technical Preview](http://hortonworks.com/tech-preview-hdp-2-5/)
- Complete the [Learning the Ropes of the Hortonworks Sandbox tutorial,](http://hortonworks.com/hadoop-tutorial/learning-the-ropes-of-the-hortonworks-sandbox/) you will need it for logging into Ambari as an administrator user.

## Outline

- [1: Start Kafka, HBase, Ambari Infra, Atlas and Ranger](#start-services)
      - [1.1: View the Services Page](#view-service-page)
      - [1.2: Start Kafka Service](#start-kafka)
- [2: General Information](#general-information)
- [3: Access Without Tag Based Policies](#access-without-tag)
- [4: Create Tag and Tag Based Policy](#create-tag-based-policy)
- [5: Summary](#summary)

## 1. Start Kafka, HBase, Ambari Infra, Atlas and Ranger <a id="start-services"></a>

### 1.1 View the Services Page <a id="view-service-page"></a>

Started by logging into Ambari as an admin user.

![new_ambari_home_page](/assets/tag-based-policies-atlas-ranger/new_ambari_home_page.png)

From the Dashboard page of Ambari, click on Kafka from the list of installed services.

![new_select_kafka](/assets/tag-based-policies-atlas-ranger/new_select_kafka.png)

### 1.2 Start Kafka Service <a id="start kafka"></a>

From the Kafka page, click on `Service Actions -> Start`

![start_kafka](/assets/tag-based-policies-atlas-ranger/start_kafka.png)

Check the box and click on `Confirm Start`:

![confirmation_kafka](/assets/tag-based-policies-atlas-ranger/confirmation_kafka.png)

Wait for Kafka to start (It may take a few minutes to turn green)

![new_started_kafka](/assets/tag-based-policies-atlas-ranger/new_started_kafka.png)

Now do ssh into the terminal:

~~~
ssh root@127.0.0.1 -p 2222
~~~

![sshTerminal](/assets/tag-based-policies-atlas-ranger/sshTerminal.png)

Let’s us give proper permissions. Type the following command on terminal:

~~~
chmod 777 /etc/ranger/admin/.rangeradmin.jceks.crc
~~~

Now start other required services as well and restart Ranger. Your Ambari dashboard page should look like this:

![new_ambari_dashboard](/assets/tag-based-policies-atlas-ranger/new_ambari_dashboard.png)

## 2. General Information <a id="general-information"></a>

Atlas and Ranger can be accessed using the following credentials:

User id – **admin**
Password – **admin**

In the tutorial steps below, we are going to be using user ids admin and maria_dev. You can login into Ambari view using the following credentials

User id – **admin**
Password – **same password that you reset in Learning the Ropes of HDP Sandbox tutorial**.

User id – **maria_dev**
Password – **maria_dev**

## 3. Access Without Tag Based Policies <a id="access-without-tag"></a>

Let’s create a hive table employee from Ambari `Hive View`.

Go to Hive view from menu icon, and type the following create table query:

~~~
create table employee (ssn string, name string, location string)
row format delimited
fields terminated by ','
stored as textfile;
~~~

And click on green `Execute` button.

![create_hive_table](/assets/tag-based-policies-atlas-ranger/create_hive_table.png)

You can check whether your table gets created or not by refreshing database explorer. Click on default database, you will see a new employee table.

![list_hive_table](/assets/tag-based-policies-atlas-ranger/list_hive_table.png)

Now let’s put some records into this table.

Create a new file called `employeedata.txt` from the terminal:

~~~
vi employeedata.txt
~~~

And put following data:

~~~
111-111-111,James,San Jose
222-222-222,Mike,Santa Clara
333-333-333,Robert,Fremont
~~~

Exit the vi shell by typing Esc-->wq!
Now copy this file to HDFS where employee table data is being stored.

~~~
hadoop fs -copyFromLocal employeedata.txt /apps/hive/warehouse/employee
~~~

Now let’s go back to hive view to view this data.

Go to database explorer and press the button right next to employee table:

![view_data](/assets/tag-based-policies-atlas-ranger/view_data.png)

You will be able to see the data.

In the first scenario, you have an `employee` data table in Apache Hive with `ssn, name and location` as part of the columns. The ssn and location information is deemed sensitive and users should not have access to it.

You need to create a Ranger policy which allows for access to name column except ssn and location. This policy will be assigned to both admin and maria_dev user for now.

Go to Ranger UI on:
`127.0.0.1:6080`

You will see a login page like below. Use username `admin` and password `admin`:

![ranger_login_page](/assets/tag-based-policies-atlas-ranger/ranger_login_page.png)

Press `Sign In` button, the home page of Ranger will be displayed

![ranger_home_page](/assets/tag-based-policies-atlas-ranger/ranger_home_page.png)

Click on `Sandbox_hive` and then `Add New Policy`:

![new_sandbox_hive_policies](/assets/tag-based-policies-atlas-ranger/new_sandbox_hive_policies.png)

Enter following values:

~~~
Policy Names - policy for all users
Hive Databases - default
table - employee
Hive_column - ssn, location (NOTE : Do NOT forget to exclude these columns)
Description - Any description
~~~

In the `Allow Conditions`, it should have the following values:

~~~
Select Group – blank, no input
Select User – admin, maria_dev
Permissions – Click on the + sign next to Add Permissions and click on select and then green tick mark.
~~~

![add_permission](/assets/tag-based-policies-atlas-ranger/add_permission.png)

You should have your policy configured like this:

![policy_configured](/assets/tag-based-policies-atlas-ranger/policy_configured.png)

Click on `Add` and you can see the list of policies that are present in `Sandbox_hive`.

![new_policy_added](/assets/tag-based-policies-atlas-ranger/new_policy_added.png)

You have to disable the `Hive Global Tables Allow` to test out the one that you just created. Go inside to this policy and toggle to disable it.

![new_hive_global_policy_disabled](/assets/tag-based-policies-atlas-ranger/new_hive_global_policy_disabled.png)

To check the access, get back to Ambari as `maria_dev` user.

![maria_dev_ambari_login](/assets/tag-based-policies-atlas-ranger/maria_dev_ambari_login.png)

Go directly to `Hive View`, click on default database and employee table. Press the button to view the table data.

![maria_dev_access_error](/assets/tag-based-policies-atlas-ranger/maria_dev_access_error.png)

You will get an authorization error. This is expected as the user does not have access to 2 columns in this table (ssn and location). To verify this, you can also view the Audits in Ranger. Go back to Ranger and click on `Audits=>Access` and select `Sandbox_hive` in Service Name. You will see the entry of Access Denied for maria_dev.

![new_policy_audit](/assets/tag-based-policies-atlas-ranger/new_policy_audit.png)

Now coming back to Hive View, try running a query for selective column.

~~~
SELECT name from employee LIMIT 100;
~~~

![maria_dev_access_successful](/assets/tag-based-policies-atlas-ranger/maria_dev_access_successful.png)

The query runs successfully.
Even, **admin** user cannot not see all the columns the location and SSN. We would provide access to this user to all columns later.

![admin_access_error](/assets/tag-based-policies-atlas-ranger/admin_access_error.png)

## 4. Create Tag and Tag Based Policy <a id="create-tag-based-policy"></a>

As a first step, login into ATLAS web app using `http://127.0.0.1:21000/` and use username **holger_gov** and password **holger_gov**.

![new_atlas_login](/assets/tag-based-policies-atlas-ranger/new_atlas_login.png)

Go to `Tags` tab and press `Create Tag` button.
Give it a name as **PII** and description as **Personal Identifiable Information**.

![new_create_tag](/assets/tag-based-policies-atlas-ranger/new_create_tag.png)

Now go to `Search` tab and write `employee` in the box. It will give all the entities related to word employee. Click on the **+** sign of ssn column.

![new_employee_search](/assets/tag-based-policies-atlas-ranger/new_employee_search.png)

Select `PII` from the drop down and click on `Add` to get PII assigned to ssn :

![new_select_pii_for_ssn](/assets/tag-based-policies-atlas-ranger/new_select_pii_for_ssn.png)

Do the same thing for location column. You should see your PII tag getting assigned to both ssn and location columns:

![new_pii_tag_assigned](/assets/tag-based-policies-atlas-ranger/new_pii_tag_assigned.png)

Now let’s go back to Ranger UI. The tag and entity relationship will be automatically inherited by Ranger. In Ranger, we can create a tag based policy by accessing it from the top menu. Go to `Access Manager → Tag Based Policies`.

![create_tag_home_page](/assets/tag-based-policies-atlas-ranger/create_tag_home_page.png)

Click `+` button to create a new tag service.

![create_sandbox_tag](/assets/tag-based-policies-atlas-ranger/create_sandbox_tag.png)

Give it a name `Sandbox_tag` and click `Add`.
You can see it added in the list.

![view_sandbox_tag](/assets/tag-based-policies-atlas-ranger/view_sandbox_tag.png)

Click on `Sandbox_tag` to add a policy.

![sandbox_tag_home_page](/assets/tag-based-policies-atlas-ranger/sandbox_tag_home_page.png)

Click on `Add New Policy` button.
Give following details:

~~~
Policy Name – PII column access policy
Tag – PII
Description – Any description
Audit logging – Yes
~~~

![pii_column_access_policy](/assets/tag-based-policies-atlas-ranger/pii_column_access_policy.png)

In the Allow Conditions, it should have the following values:

~~~
Select Group - blank
Select User - admin
Component Permissions - Select Hive
~~~

You can select the component permission through this popup:

![new_allow_permissions](/assets/tag-based-policies-atlas-ranger/new_allow_permissions.png)

This signifies that only admin is allowed to do any operation on the columns that are specified by PII tag. Click `Add` to view it created.

![sandbox_tag_policies](/assets/tag-based-policies-atlas-ranger/sandbox_tag_policies.png)

Now click on `Resource Based Policies` and edit `Sandbox_hive` repository by clicking on the button next to it.

![editing_sandbox_hive](/assets/tag-based-policies-atlas-ranger/editing_sandbox_hive.png)

Click on `Select Tag Service` and select `Sandbox_tag`. Click on `Save`.

![new_edited_sandbox_hive](/assets/tag-based-policies-atlas-ranger/new_edited_sandbox_hive.png)

The Ranger tag based policy is now enabled for **admin** user. You can test it by running the query on all columns in employee table.

![admin_access_successful](/assets/tag-based-policies-atlas-ranger/admin_access_successful.png)

The query executes successfully. The query can be checked in the Ranger audit log which will show the access granted and associated policy which granted access. Select Service Name as `Sandbox_hive` in the search bar.

![new_audit_results](/assets/tag-based-policies-atlas-ranger/new_audit_results.png)

> **NOTE**: There are 2 policies which provided access to admin user, one is a tag based policy and the other is hive resource based policy. The associated tags (PII) is also denoted in the tags column in the audit record).

## 5. Summary <a id="summary"></a>

Ranger traditionally provided group or user based authorization for resources such as table, column in Hive or a file in HDFS.
With the new Atlas -Ranger integration, administrators can conceptualize security policies based on data classification, and not necessarily in terms of tables or columns. Data stewards can easily classify data in Atlas and use in the classification in Ranger to create security policies.
This represents a paradigm shift in security and governance in Hadoop, benefiting customers with mature Hadoop deployments as well as customers looking to adopt Hadoop and big data infrastructure for first time.