---
title: "Mulesoft, Dynamics CRM, and Kerberos Authentication"
description: "Learn how to connect Mulesoft to Microsoft Dynamics CRM using Kerberos authentication."
date: 2017-10-14T12:29:26-04:00
draft: false
slug: mulesoft-dynamics-crm-kerberos
---

## Overview
I have been evaluating Mulesoft Anypoint platform for a large project I am involved in. One of the use cases being reviewed is to see how Mulesoft can integrate with Microsoft Dynamics CRM. The biggest challenge was getting authentication between Mulesoft and Microsoft Dynamics CRM working properly. This post will walk through the steps and values needed.

## Walkthrough

We are going to start in Anypoint Studio with the connector already installed. Off the bat, the connector, like the majority of the connectors Mulesoft is pretty easy to use. The biggest difficulty, and the area that has the least documentation out there is how to authenticate with CRM. There are a number of authentication options available through the connector.

{{< load-photoswipe >}}

First, add a Poll scope to the flow as a container for the Microsoft Dynamics CRM Connector.
{{< figure src="/images/2017-10-10-mulesoft-dynamics-crm-kerberos-auth-1.png" caption="Poll scope on the flow.">}}

Next, add the Dynamics CRM Connector to the Poll scope on the flow. Once on the flow, click the + to the right of the Connector Configuration to bring up the connection detail dialog.
{{< figure src="/images/2017-10-10-mulesoft-dynamics-crm-kerberos-auth-2.png" caption="CRM Connector inside the Poll scope on the flow.">}}

Select the "Microsoft Dynamics CRM: Kerberos Connection" option and then press the "Ok" button.
Next, add the Dynamics CRM Connector to the Poll scope on the flow. Once on the flow, click the + to the right of the Connector {{< figure src="/images/2017-10-10-mulesoft-dynamics-crm-kerberos-auth-3.png" caption="CRM Connector Global Type dialog.">}}

The Microsoft Dynamics CRM: Kerberos Connection dialog is where the fun begins. The documentation is limited, and the error messaging doesn't help narrow the problem down. It required a fair amount of trial and error with the values to get it working properly.
{{< figure src="/images/2017-10-10-mulesoft-dynamics-crm-kerberos-auth-4.png" caption="CRM Kerberos Connection Type dialog.">}}

We need to identify and populate a number of items in order to establish the connection. The listed order is slightly different than the order on the Kerberos Connection Type dialog to better help with finding the needed information:

* Organization Service URL of the CRM server.
* Username and Password used to authenticate with CRM.
* SPN (Service Principal Name) of the CRM server.
* Realm of the Active Directory domain the CRM server resides in.
* KDC (Key Distribution Center) for the domain the CRM server resides in.

For the post, the CRM server is accessible at http://crm.appdomain.domain.com, resides on the subdomain.domain.com Active Directory domain, and is named crmsvr1. To clarify, CRM is accessible via multiple subdomains.

### Organization Service URL
This will be the URL that you access your Dynamics CRM server with "/XRMServices/2011/Organization.svc" added to the end. It is possible that your installation has a different service URL, so check with your CRM Admins if it doesn't work. The full Url will look like http://crm.appdomain.domain.com/XRMServices/2011/Organization.svc.

### Username and Password
Credentials for a user account that exists on the same Active Directory domain that the CRM server is running on. The best way to identify the domain if you are unsure is to navigate in a browser to the WSDL for the Organization Service URL identified above https://crm.appdomain.domain.com/XRMServices/2011/Organization.svc?wsdl
{{< figure src="/images/2017-10-10-mulesoft-dynamics-crm-kerberos-auth-5.png" caption="Results from Organization Service WSDL">}}
Look for the **Identity/Upn** node. In the example, the domain is **subdomain.domain.com**, so the account should be on the same domain and have appropriate access to CRM.

### SPN (Service Principal Name)
This is a typically auto-generated on a Windows server when it is added to a domain. A complete list of SPNs can be retrieved by running the following from a command prompt on the Dynamics CRM server.
{{< highlight bash >}}
C:\>setspn -l crmsvr1
{{< / highlight >}}

There will typically be a number of responses, but the one we want looks like so: **HTTP/crm.subdomain.domain.com**.

In cases where the CRM server is part of a cluster, the individual servers will each have duplicate SPNs as well as the SPN of the cluster's load balancer. We have two servers crmsvr1.appdomain.domian.com and crmsvr2.appdomain.domain.com sitting behind a load balancer listening for crm.appdomain.domain.com. Each server will have SPN entries for each server and the load balancer. Use the load balancer's SPN.

### Realm
The fully qualified domain name in Active Directory. This is case sensitive and in every situation I have found, it should be uppercase. In the example, the Realm is **SUBDOMAIN.DOMAIN.COM**

### KDC (Key Distribution Center)
For Kerberos, this is a service that runs on every Active Directory domain controller and is used to supply session tickets and temporary keys to users. Mulesoft requires the fully qualified name of one of the domain controllers where the CRM server exists. A server on the subdomain.domain.com Active Directory domain is dc1, so the value for the KDC is dc1.subdomain.domain.com. This is a bit odd because it doesn't seem to support outages to the identified server, but I have not tried to kill the domain controller just to see what happens.

If all goes well, testing the connection will return a success message.

## References
The Microsoft Dynamics CRM Connector was created by Mulesoft with assistance from Microsoft, and is supported. For more information on the connector, visit Mulesoft's <a href="https://www.mulesoft.com/exchange/org.mule.modules/mule-module-ms-dynamics-crm" target="_blank">Anypoint Exchange</a> to learn more.
