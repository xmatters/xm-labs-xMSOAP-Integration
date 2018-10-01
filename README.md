# xM SOAP HTTP POST Integration
The xMatters Integration Builder is capable of receiving an parsing a variety of HTTP POST content.  In this case, this integration is designed to specifically consume an xMSOAP object via the xMatters Integration Builder.  Historically this would be one possible integration mechanism between an "External Product" and the "xMatters Integration Agent".

As this is an HTTP POST, it is entirely possible to remove the Integration Agent from the picture assuming the "External Product" (Any product integrating to xMatters via the HHTP API of the Integration Agent using the xMSOAP format) can post to the xMatters onDemand instance instead.

This is an example integration provided to show the nuances within the Integration Builder coding which are needed to consume an xMSOAP based HTTP Post and convert it into something usable by xMatters on Demand.

**Please note:** This could also be used for any SOAP based HTTP Post with some modification to manage an alternate SOAP Envelope.

**xMatters Communication Plan Inbound Integrations**
* **0 - SOAP HTTP Post Entrypoint**: This inbound integration receives the HTTP Post containing the xMSOAP envelope.  It will parse the envelope and convert the SOAP properties into Communication Form properties and then pass the result to the Sample Form EntryPoint 
* **Sample Form EntryPoint**: This inbound integrations receives the HTTP Post from the SOAP HTTP Post Entrypoint in the required JSON format for the underlying form.

<kbd>
  <img src="https://github.com/xmatters/xMatters-Labs/raw/master/media/disclaimer.png">
</kbd>

# Pre-Requisites
* xMatters account - If you don't have one, [get one](https://www.xmatters.com)!

# Files
* [SamplexMSOAPConversion.zip](SamplexMSOAPConversion.zip) - The example Communication Plan including both the Sample Code and Sample Form. 

# How it works
This integration provides an example of consuming an xMSOAP HTTP Post and converting it into a xMatters Communciation Form JSON Object.  This includes providing examples addressing several of the xMSOAP nuances around data (eventToken) management. 

Assuming the "External Product" is capable of targetting xMatters onDemand directly, adjusting the endpoint used by the "External Product" from the Integration Agent to the 0 - SOAP HTTP Post EntryPoint and extract the xMatters Integration Agent from the process.

Although this example is specific to xMSOAP, it can definately be used as a basic example for any other SOAP Envelope based HTTP Post from an "Enternal Product". 

This integration is set up to pass the following values from the "Enternal Product" to xMatters: 
* person_id - a comma delimited list of one or more recipients
* sample message - a long message including carriage returns
* sample priority - a single list item selection
* sample boolean - a yes/no
* sample country - a multilist of countries impacted which also requires a delimiter field

**NOTE**: The Integration Agent may have custom code that will also need to be considered/ported into the Integration Builder Code but that would be specific to the Integration and workflow.


# Installation

## xMatters set up

### Create an integration user
This integration requires a user who can authenticate REST web service calls when injecting events.

This user needs to be able to work with events, but does not need to update administrative settings. While you can use the default Company Supervisor role to authenticate REST web service calls, the best method is to create a user specifically for this integration with the "REST Web Service User" role that includes the permissions and capabilities.

__Note__: If you are installing this integration into an xMatters trial instance, you don't need to create a new user. Instead, locate the "Integration User" sample user that was automatically configured with the REST Web Service User role when your instance was created and assign them a new password. You can then skip ahead to the next section.

__To create an integration user:__

1. Log in to the target xMatters system.
2. On the __Users__ tab, click the __Add New User__ icon.
3. Enter the appropriate information for your new user. 
    Example User Name __integration__
4. Assign the user to the __REST Web Service User__ role.
5. Click __Save__.
6. On the next page, set the web login ID and password. 

Make a note of these details; you will need them when configuring other parts of this integration.

This users details will be user for URL based authentication

<br><br><br>
### Create users and groups that will receive notifications

The integration will notify the recipients listed in the person_id property, these recipients will need to be present in xMatters.

*This integration does not synchronize users and groups. Make sure you have created your users and groups in xMatters before using this integration.*

For more information about creating users and devices in xMatters, refer to the [xMatters On-Demand help](https://help.xmatters.com/ondemand/xmatters.htm).

<br><br><br>
### Import the xMatters Communication Plan

The next step is to import the communication plan.

To import the communication plan:

1. In the target xMatters system, on the __Developer__ tab, click __Import Plan__.
2. Click __Browse__, and then locate the downloaded communication plan: [SamplexMSOAPConversion.zip](SampleXMSOAPConversion.zip)
3. Click __Import Plan__.
4. Once the communication plan has been imported, click __Plan Disabled__ to enable the plan.
5. In the __Edit__ drop-down list, select __Access Permissions__ a
6. Enter the REST API user you created above, and then click __Save Changes__.
7. In the __Edit__ drop-down list, select __Forms__.
8. For the __New Incident__ form, in the __Not Deployed__ drop-down list, click Create __Event Web Service__.
9. After you create the web service, the drop-down list label will change to __Web Service Only__.
10. In the __Web Service Only__ drop-down list, click __Permissions__.
11. Enter the REST API user you created above, and then click __Save Changes__.


<br><br><br>
### Accessing web service URLs

Each integration service has its own URL, one is used to target the Integration from the "External Product" the other is used to target the Sample Form from the initial integration Service (This is configured via an Integration Builder Constant).

__To get a web service URL for an integration service:__

1. On the Integration Builder tab, expand the list of inbound integrations.
2. Click on the inbound integration.
3. Under __How to trigger the integration__ search for the REST API user you created above and copy/record the trigger associated to that user account.

__NOTE:__ You will need the URL for each integration service.

### Configuring access to the Form

For the inbound integration __0 - SOAP HTTP Post Entrypoint__ to successfully hit the Sample Form, the Samnple Form EntryPoint URL must be configured in code.  We have made this simple by providing it as a constant.

1. On the Integration Builder tab, click on the __Edit Constants__ button.
2. Edit the __FORM_URL__ constant and update the value to match the trigger for the Sample Form EntryPoint from the prior step __NOTE__: This is only from the __api/...__ portion of the URL and does not include the port and base domain reference found in the trigger.

### Initiating the integration

The trigger obtained from __0 - SOAP HTTP Post Entrypoint__ is sufficient to initiate the integration, assuming the xMSOAP passed via the post matches what is outlined above.  An example of the xMSOAP Post body which works which this integration will also be provided below.

# Testing
## Sample SOAP Object
Below is a sample SOAP Object aligned to the underlying inbound integration and subsequent form.  This can be used for testing.  I use https://client.restlet.com as my testing tool.

```<?xml version="1.0" encoding="UTF-8"?>
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/">
   <soapenv:Header />
   <soapenv:Body>
      <sch:AddEvent xmlns:sch="http://www.alarmpoint.com/webservices/schema">
         <sch:user>test</sch:user>
         <sch:password>abc123</sch:password>
         <sch:clientTimestamp>23/08/2018 7:38:27 AM</sch:clientTimestamp>
         <sch:clientIP xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:nil="true" />
         <sch:clientOSUser xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:nil="true" />
         <sch:company>ACME</sch:company>
         <sch:eventTokens>
            <sch:eventToken>
               <sch:name>agent_client_id</sch:name>
               <sch:value>xMNonProd</sch:value>
            </sch:eventToken>
            <sch:eventToken>
               <sch:name>sample message</sch:name>
               <sch:value>This is a sample message

INCIDENT START TIME: 25-Sep-2018 12:08 GMT
ESTIMATED RESOLUTION TIME: No ETR
NEXT UPDATE TIME: 25-Sep-2018 18:00 GMT

TICKET:  INC000001

REFERENCE:  http://google.com</sch:value>
            </sch:eventToken>
            <sch:eventToken>
               <sch:name>sample priority</sch:name>
               <sch:value>P1</sch:value>
            </sch:eventToken>
            <sch:eventToken>
               <sch:name>sample boolean</sch:name>
               <sch:value>No</sch:value>
            </sch:eventToken>
            <sch:eventToken>
               <sch:name>sample country</sch:name>
               <sch:value>Afghanistan|Albania|Algeria|Angola|Argentina|Armenia|Aruba|Australia</sch:value>
            </sch:eventToken>
            <sch:eventToken>
               <sch:name>sample country-list-item-delimiter</sch:name>
               <sch:value>|</sch:value>
            </sch:eventToken>
            <sch:eventToken>
               <sch:name>person_id</sch:name>
               <sch:value>Operations,bsmith</sch:value>
            </sch:eventToken>
         </sch:eventTokens>
      </sch:AddEvent>
   </soapenv:Body>
</soapenv:Envelope>```
