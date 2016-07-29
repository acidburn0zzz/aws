
#Palo Alto VM-Series for AWS

![](media/image1.png){width="5.347222222222222in"
height="2.851851487314086in"}

**GlobalProtect in an AWS Auto Scaling Environment **

<http://www.paloaltonetworks.com>

Table of Contents

[1. Global Protect 3](#_Toc452984006)

[2. GlobalProtect and AutoScaling 3](#globalprotect-and-autoscaling)

[3. About CFTs 4](#about-cfts)

[4. About this guide 4](#about-this-guide)

[5. AWS Services Used 5](#_Toc452984010)

[5.1 AWS Auto Scaling 5](#aws-auto-scaling)

[5.2 AWS Lambda 5](#aws-lambda)

[5.3 Amazon S3 6](#amazon-s3)

[5.4 Amazon SNS 6](#amazon-sns)

[5.5 Amazon CloudWatch 6](#amazon-cloudwatch)

[6. Template Deployment Prerequisites 6](#_Toc452984016)

[6.1 Create an AWS account 6](#_Toc452984017)

[6.2 Add a credit card to your AWS account 6](#_Toc452984018)

[6.3 Review and accept the EULA 7](#_Toc452984019)

[6.4 Create and download an SSH keypair 9](#_Toc452984020)

[7. PAN-OS Prerequisites 11](#pan-os-prerequisites)

[8. Download the Files 11](#_Toc452984022)

[9. Create S3 Buckets 12](#create-s3-buckets)

[10. Launch the CFT 15](#_Toc452984024)

[11. Launch the First Gateway 20](#_Toc452984025)

[12. Scale-out/Scale-In Policy 24](#scale-outscale-in-policy)

[13. Trigger an Auto Scale Event 25](#trigger-an-auto-scale-event)

[14. Cleanup 31](#cleanup)

[15. Conclusion 35](#_Toc452984029)

<span id="_Toc287877361" class="anchor"><span id="_Toc287877384"
class="anchor"></span></span>

<span id="_Toc319837950" class="anchor"><span id="_Toc452984006" class="anchor"></span></span>Global Protect
============================================================================================================

GlobalProtect allows remote users to access corporate resources and
internet resources using the same security policy enforcement as though
there were on premises. To provide Next-Generation Security Platform
closer to the remote users, GlobalProtect gateways can be deployed in
AWS. This allows instantiation of portals and gateways near remote users
without the additional cost of infrastructure.

Leveraging the global presence and built in redundancy provided by AWS,
Global Protect can be quickly deployed worldwide where your users are.
Traffic is inspected with the same PAN-OS security as the corporate
firewall but in a globally diverse deployment. The result is security
that follows users – even when they are mobile and a better user
experience.

GlobalProtect and AutoScaling
=============================

Running within the AWS infrastructure provides the ability to scale-out
and scale-in on demand. Take the use cases of both planned and unplanned
need for dynamic scaling. Planned events such as a conferences or sales
kick-offs where lots of users all try to connect to a regional gateway.
And unplanned events such as “snowmageddon” where users are snowed in
and work from home. Another very common scenario is a surge of users
logging on to the gateway at the start of the day and thus a need for a
dynamic scale out of gateways (and scale in at the end of the day)

![](media/image2.png){width="6.5in" height="3.189583333333333in"}

About CFTs
==========

AWS CloudFormation Templates (CFTs), are JSON files that can launch
nearly all AWS resources including VPCs, subnets, security groups, route
tables, plus many more. AWS CFTs are used for ease of deployment and are
key to any auto-scaling environment.

For more information on CFTs and sample CFTs refer to Amazon’s
documentation

<https://aws.amazon.com/cloudformation/aws-cloudformation-templates/>

There are also many sample templates available here
<https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/sample-templates-services-us-west-2.html>

Additional templates provided by Palo Alto Networks can be found on the
following Github page

<https://github.com/paloaltonetworks/aws>

About this guide
================

This guide will walk through the process of launching an AWS CFT that
will deploy a GlobalProtect Portal and an AWS AutoScaling group with the
ability to automatically launch and configure additional GlobalProtect
Gateways based on customizable thresholds.

If new to Cloudformation templates, please refer to the following guide:

<https://s3-us-west-2.amazonaws.com/sample-cft/AWS_CFT_How_To_Guide_v7.docx>

The template referenced by this document will deploy all the resources
needed for the purposes of the demo and setup the appropriate
permissions and get to the topology below:

![](media/image3.emf){width="5.638888888888889in"
height="4.263888888888889in"}

<span id="_Toc319837951" class="anchor"><span id="_Toc452984010" class="anchor"></span></span>AWS Services Used
===============================================================================================================

The template utilizes several AWS services. Some of the main ones are:

AWS Auto Scaling
----------------

Auto Scaling is an AWS service designed to launch or terminate Amazon
EC2 instances automatically based on user-defined policies, schedules,
and health checks. Auto Scaling Groups (ASG) define minimum EC2
instances, maximum EC2 instances, and metrics used to determine when to
scale in or scale out.

<https://docs.aws.amazon.com/autoscaling/latest/userguide/WhatIsAutoScaling.html>

AWS Lambda
----------

AWS lambda is a compute (micro)service that allows a user to run small
snippets of code (JavaScript or Python scripts) to accomplish various
tasks. This eliminates the use of Linux instances as worker nodes and
having to maintain them.

<https://docs.aws.amazon.com/lambda/latest/dg/welcome.html>

**Note:** AWS Lambda service is currently only available in 4 regions
(Northern Virginia, Oregon, Frankfurt, Ireland and Tokyo). Please refer
to the following page to check for further updates:
<https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/>

Amazon S3
---------

Amazon Simple Storage Service is storage service where the necessary
scripts and bootstrap files are stored.

<https://docs.aws.amazon.com/AmazonS3/latest/dev/Introduction.html>

Amazon SNS
----------

Amazon Simple Notification Service is a web service that coordinates and
manages the delivery or sending of messages to subscribing endpoints or
clients. In this case the service is used to send messages to trigger
Lambda functions

<https://docs.aws.amazon.com/sns/latest/dg/welcome.html>

Amazon CloudWatch
-----------------

Amazon CloudWatch monitors your Amazon Web Services (AWS) resources and
the applications you run on AWS in real-time. You can use CloudWatch to
collect and track metrics, which are the variables you want to measure
for your resources and applications. CloudWatch alarms send
notifications or automatically make changes to the resources you are
monitoring based on rules that you define.

<https://docs.aws.amazon.com/AmazonCloudWatch/latest/DeveloperGuide/WhatIsCloudWatch.html>

<span id="_Toc319837952" class="anchor"><span id="_Toc452984016" class="anchor"></span></span>Template Deployment Prerequisites 
================================================================================================================================

Here are the prerequisites required to successfully launch this
template.

<span id="_Toc319837953" class="anchor"><span id="_Toc452984017" class="anchor"></span></span>Create an AWS account
-------------------------------------------------------------------------------------------------------------------

If you do not have an AWS account already, go to
<https://aws.amazon.com/console/> and create an account.

<span id="_Toc319837954" class="anchor"><span id="_Toc452984018" class="anchor"></span></span>Add a credit card to your AWS account
-----------------------------------------------------------------------------------------------------------------------------------

In order to continue you will need to add a method of payment to your
AWS account. Use the following
<https://console.aws.amazon.com/billing/home#/paymentmethods>

If creating a new account, you may receive a phone call from AWS for
verification purposes.

<span id="_Accept_the_EULA" class="anchor"><span id="_Toc319837955" class="anchor"><span id="_Toc452984019" class="anchor"></span></span></span>Review and accept the EULA
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------

If this is your first time using AWS to launch a VM-Series firewall
bundle, you will need to review and accept the software license
agreement for the VM-Series.

Click on **AWS Marketplace** and search for **Palo Alto Networks
firewall**:

![](media/image4.png){width="6.5in" height="3.5319444444444446in"}

![](media/image5.emf){width="6.494827209098863in"
height="2.956266404199475in"}

Select **VM-Series Next Generation Firewall Bundle 2**

![](media/image6.emf){width="6.175395888013998in"
height="1.58998687664042in"}

![](media/image7.emf){width="6.28862532808399in"
height="3.9611111111111112in"}

Click **Continue**.

![](media/image8.emf){width="6.797133639545057in"
height="4.586111111111111in"}

Click on **Manual** Launch, Review the agreement and then click **Accept
Software Terms**

You should see this screen:

![](media/image9.emf){width="6.43125in" height="1.0743449256342956in"}

You can now close the browser tab or window and continue with the next
step.

<span id="_Create_and_download" class="anchor"><span id="_Toc319837956" class="anchor"><span id="_Toc452984020" class="anchor"></span></span></span>Create and download an SSH keypair
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Sign into the AWS console <https://www.amazon.com> and click on **EC2**

![](media/image10.emf){width="6.5in" height="3.3472222222222223in"}

Click **KeyPairs**

![](media/image11.emf){width="6.396008311461068in"
height="1.9504516622922135in"}

Click **Create Key Pair**

![](media/image12.png){width="6.5in" height="0.6222222222222222in"}

Give the key-pair a name:

![](media/image13.emf){width="6.412250656167979in"
height="2.581266404199475in"}

And click **Create**. This should now prompt you to save the just
generated private key. Save the key.

PAN-OS Prerequisites 
=====================

This template relies on the bootstrapping feature that is part of PAN-OS
7.1. For more information on bootstrapping the VM-Series in AWS please
refer to the following documentation:
<https://www.paloaltonetworks.com/documentation/71/virtualization/virtualization/bootstrap-the-vm-series-firewall/bootstrap-the-vm-series-firewall-in-aws>

<span id="_Download_the_Files" class="anchor"><span id="_Toc452984022" class="anchor"></span></span>Download the Files
======================================================================================================================

The CloudFormation template can be found here:

<https://s3.amazonaws.com/gp-asg/asg-lambda.json>

The bootstrap files can be found here:

<https://s3.amazonaws.com/gp-asg/bootstrap-gateway.zip>

<https://s3.amazonaws.com/gp-asg/bootstrap-portal.zip>

And the accompanying scripts can be found here:

<https://s3.amazonaws.com/gp-asg/config_fw.zip>

Download and save the asg-lambda.json, config\_fw.zip,
bootstrap-gateway.zip and bootstrap-portal.zip files and store them in a
known locations.

Unzip the bootstrap-gateway.zip and bootstrap-portal.zip files into
their corresponding directories.

The “bootstrap-gateway” and “bootstrap-portal” directories have
bootstrapping and configuration information for the gateways and portal
respectively.

They also have configuration information that configures the firewall as
a GlobalProtect Gateway or GlobalProtect Portal.

The config\_fw.zip file contains all the necessary lambda scripts

Create S3 Buckets
=================

In order to launch the demo template three S3 buckets will be required.
Log into the AWS console and click on **S3:**

![](media/image14.png){width="6.485416666666667in"
height="4.593836395450569in"}

And click “**Create Bucket”** button**:**

![](media/image15.png){width="4.364583333333333in"
height="1.7604166666666667in"}

Type-in a bucket name (needs to be unique) and select the region in
which the bucket will reside. This should be the same region where the
template will be deployed ***and*** Lambda is available. This bucket
will be used to store bootstrap files for the GP Gateway.

**Note:** Keep in mind that as of writing this document AWS lambda is
only supported in 5 regions (N. Virginia, Oregon, Ireland, Frankfurt,
and Tokyo).

**Note:** If launching the CFT in N. Virginia, select the region as
“**US Standard”**

![](media/image16.png){width="6.5in" height="3.10625in"}

Once the bucket is created, click on the bucket name to navigate to the
bucket and create **four** sub-buckets called **config, content,
license** and **software** respectively. Navigate to the **config**
bucket and click “**Upload”**

![](media/image17.png){width="4.069444444444445in" height="1.25in"}

![](media/image18.png){width="4.083333333333333in"
height="2.5694444444444446in"}

In the next dialog box, drag and drop the two files – **init-cfg.txt**
and **bootstrap.xml** from the **gateway** folder created in the
[previous step](#_Download_the_Files) and click “**Start Upload**”:

![](media/image19.png){width="3.5in" height="1.3020833333333333in"}

![](media/image20.png){width="6.5in" height="3.8930555555555557in"}

Repeat the above steps and create a bucket to upload the portal’s
bootstrap configuration files.

Lastly create an S3 bucket in the same region as the gateway and portal
bootstrap buckets and upload the config\_fw.zip file into that bucket
(this is the bucket where the lambda scrips reside).

<span id="_Toc319837957" class="anchor"><span id="_Toc452984024" class="anchor"></span></span>Launch the CFT
============================================================================================================

Login in to the AWS console <https://console.aws.amazon.com> and click
on **CloudFormation**

![](media/image21.png){width="6.5in" height="2.1777777777777776in"}

**Note: Please make sure that your region (on the top right) is set to a
region where Lambda is available (and the S3 buckets were created).**

Click **Create Stack**:

![](media/image22.emf){width="6.447732939632546in"
height="1.2673129921259843in"}

Select “**Upload a template to S3**” and click the “**Choose File**”
button. Then select the asg-lambda.json template file. Then, click
**Next**:

![](media/image23.png){width="6.5in" height="2.8583333333333334in"}

In the next screen specify a “**Stack Name**”. Specify the gateway
bootstrap bucket name for the **GatewayBootstrapBucketName** parameter,
portal bootstrap bucket name for the **PortalBootstrapBucketName**
parameter and the bucket name where the lambda scripts reside for the
**LambdaBucketName** parameter.

Select a **Serverkey** for which you have the private key. Refer to
[section 2.4](#_Create_and_download) on how to generate a keypair. Once
satisfied, click **Next**.

![](media/image24.png){width="6.5in" height="3.0618055555555554in"}

On the next screen you can specify tags (optional) otherwise click
**Next**.

![](media/image25.emf){width="6.546196412948381in"
height="2.48533573928259in"}

Next, review and check acknowledge at the bottom and click **Create**.

![](media/image26.png){width="6.5in" height="3.783333333333333in"}

Once launched you should be able to monitor the stack creation progress
in the next screen by clicking on the **Events** tab.

**Note: The template takes about 10-15 minutes to fully deploy and be
operational.**

![](media/image27.png){width="6.5in" height="2.451388888888889in"}

If the CFT was successfully launched, you should see an event as below:

![](media/image28.png){width="6.5in" height="3.401388888888889in"}

If there were any errors during the creation of the stack, you will need
to drill down to the specific event in the **Events** tab and
**Outputs** tab to debug and then create a new stack after fixing any
errors. Scroll through the logs and find the chronologically first
error. Normally, subsequent errors are a result of the initial error and
the first error is the actual issue.

For instance, if you did not accept the VM-Series EULA, then you will
get an error as seen below

![](media/image29.emf){width="6.52238845144357in"
height="3.081266404199475in"}

Refer to [section 2.3](#_Accept_the_EULA) to review and accept the EULA
for the VM-Series NGFW

**Note: If you need to relaunch the CFT, first delete the current
stack under Actions, Delete Stack. **

![](media/image30.emf){width="6.456101268591426in"
height="3.0754516622922137in"}

<span id="_Toc319837964" class="anchor"><span id="_Toc452984025" class="anchor"></span></span>Launch the First Gateway
======================================================================================================================

After the template has been deployed successfully, go back to the
console and click on “**EC2**”:

![](media/image31.png){width="6.5in" height="1.325in"}

And click on “**Auto Scaling Groups**”

![](media/image32.png){width="6.5in" height="4.26875in"}

You should see the auto scaling group created by the template. Click
“**Edit**”

![](media/image33.png){width="6.5in" height="2.8826388888888888in"}

and change the “**Desired”** and “**Min**” fields to “1” and click
“**Save**”:

![](media/image34.png){width="6.5in" height="3.2527777777777778in"}

This will trigger an auto scale event and launch a new VM-Series
firewall and bootstrap it as a GlobalProtect Gateway. You can see this
by clicking on “**Instances**” on the left:

![](media/image35.png){width="6.5in" height="1.3159722222222223in"}

From launch to configured, it takes about 10 minutes. You can monitor
the logs if so desired by clicking on “**CloudWatch**” on the AWS
console

![](media/image36.png){width="6.5in" height="2.3243055555555556in"}

and then on “**Logs”** on the left:

![](media/image37.png){width="6.5in" height="5.106944444444444in"}

A good indication that the GlobalProtect Gateway is up is the creation
of the “**PushMetrics**” log group:

![](media/image38.png){width="6.5in" height="1.5590277777777777in"}

This indicates that the gateway is up and publishing metrics to
CloudWatch.

Scale-out/Scale-In Policy
=========================

In order to determine when to scale-out (add new gateways) or scale-in
(remove gateways from service) the number of active sessions on a
gateway is published to CloudWatch. Each gateway that is part of the
Auto Scale Group will publish its active session count per minute.

The template – for demonstration purposes – has set a very low threshold
for an auto scale event to occur.

To verify click on “**Alarms**” on the left and click on the “**OK**”
link:

![](media/image39.png){width="6.5in" height="0.9888888888888889in"}

In the above screen capture, you see two alarms. MAX\_SESS\_REACHED and
MIN\_SESS\_REACHED. These alarms are triggered based on metrics
published by the gateways in the Auto Scale Group.

If the number of active sessions is greater than 20 for more than 3
minutes, the MAX\_SESS\_REACHED alarm is triggered, which in turn
triggers scale-out event, which deploys a new GP gateway within the Auto
Scale Group

![](media/image40.png){width="6.5in" height="2.0625in"}

Similarly, if the number of sessions is less than 5 and stays that way
for 30 minutes, the MIN\_SESS\_REACHED alarm is triggered (as in the
screenshot above) and a scale-in event happens and a GP gateway is
terminated.

**Note:** There will always be at least one gateway (depending on your
configuration for the minimum number of gateways)

![](media/image41.png){width="6.5in" height="2.5770833333333334in"}

Trigger an Auto Scale Event
===========================

This should ideally be done on a VM so as to not disrupt any other
activity. For the purpose of this guide and demonstration, a Windows VM
was used.

To determine the GP Portal address, head on over to CloudFormation in
the AWS console

![](media/image42.png){width="6.5in" height="1.988888888888889in"}

and select the stack that was just deployed. Under the “**Outputs**” tab
note the **GPPortalPublicIP** address

![](media/image43.png){width="6.5in" height="1.4131944444444444in"}

In the GlobalProtect client panel specify the above **GPPortalPublicIP**
address as the Portal address. The username and password is
gp-user1/paloalto or gp-user2/paloalto

![](media/image44.png){width="6.5in" height="5.201388888888889in"}

If you get an error (as seen above) that the client could not connect to
the portal, you may need to disable the GlobalProtect client on your
host machine. The portal and gateways use self-signed certificate and so
corporate firewalls may not allow the connection to go through.

Once connected you should see the Portal and Gateway information in the
client “**Details**” tab:

![](media/image45.png){width="6.5in" height="5.192361111111111in"}

To verify that by connecting to the gateway the session count goes up
head on over the AWS console and click on CloudWatch and you should see
a small spike in the metrics for MAX\_SESSION\_REACHED:

![](media/image46.png){width="4.644212598425197in"
height="3.5828379265091863in"}

To trigger an event, in the VM open up a web browser and pick a few of
your favorite websites to visit:

![](media/image47.png){width="5.468287401574803in"
height="4.426040026246719in"}

This will cause the session count to go up, trigger the alarm and cause
a scale-out event and a new gateway will be provisioned, bootstrapped,
configured and added to the Auto Scale Group.

![](media/image48.png){width="4.495833333333334in"
height="3.4814818460192476in"}

![](media/image49.png){width="6.5in" height="1.0833333333333333in"}

Once the new gateway is up, it will be added to the pool of gateways.
Verify this in the GP Client by selecting “**Rediscover Network**”:

![](media/image50.png){width="6.5in" height="5.241666666666666in"}

To trigger a scale-in event, close all the browser sessions (or shutdown
the VM) and that will trigger a scale-in event

![](media/image51.png){width="5.805154199475066in"
height="4.581481846019248in"}

and a gateway will be removed from service

![](media/image52.png){width="6.5in" height="0.9986111111111111in"}

Cleanup
=======

Once done with the template, feel free to play around with various
things. If done, cleanup as follows. In the AWS management console,
click on **CloudFormation**:

![](media/image21.png){width="6.5in" height="2.1777777777777776in"}

Under **Actions**, click **Delete Stack:**

![](media/image53.png){width="6.5in" height="1.854861111111111in"}

This should delete all the resources created via the template.

**Note:** There are cases when deleting a stack fails due to some
dependencies that AWS is not able to automatically delete. So, if stack
deletion fails, head on over to the VPC console

![](media/image54.png){width="6.5in" height="4.576388888888889in"}

and select **VPCs**

![](media/image55.png){width="6.5in" height="4.315972222222222in"}

Select the VPC in question and select “**Delete VPC**” from the
“**Actions**” menu and click “**Yes**, **Delete**”

![](media/image56.png){width="6.5in" height="1.5291666666666666in"}

![](media/image57.png){width="6.5in" height="3.5729166666666665in"}

<span id="_Toc319837967" class="anchor"><span id="_Toc452984029" class="anchor"></span></span>Conclusion
========================================================================================================

You have successfully deployed and demonstrated GlobalProtect in an AWS
Auto Scaling environment
