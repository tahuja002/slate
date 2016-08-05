---
title: SMART on FHIR Starter APP

language_tabs:
  - javascript

search: true
---

# Introduction

This document will help you get started on how to use SMART on FHIR API to create your app with CERNER

# Project Set up
Create your project on any hosting platform and create following files:

* launch.html 
* index.html 
* load_data.js and 
* lib folder

# Download fhir-client.js
Download [**fhir-client.js**](https://github.com/smart-on-fhir/client-js/blob/master/dist/fhir-client.js) and place it under lib folder. This is the open source FHIR javascript library which would help us with OAUTH2 transactions.

The SMART on FHIR JavaScript client library helps you build browser-based SMART apps that interact with a FHIR REST API server. It can help your app get authorization tokens, provide information about the user and patient record in context, and issue API calls to fetch clinical data. This tutorial will lead you through the basics of building a SMART app using the JavaScript client.

The SMART JS client uses the open-source fhir.js for interfacing with SMART API servers. Upon successfully initializing and negotiating the SMART authorization sequence, the client will expose one or two instances of the fhir.js client as applicable via the following handles :

* *smart.api* Non-context aware API for executing operations on all authorized resources
* *smart.patient.api* Context aware API which automatically applies its operations to the patient in context

# Register APP
Once we have this done and hosted our APP, we will now register our APP with CERNER. Go to the link [FHIR Application Authorization Request](http://www.cerner.com/FHIR_Application_Authorization_Request/). and fill up following details

Field | Description
--------- | -----------
Application Name | Any name for your APP you want to give
Redirect URL | Just put your base app url. Like https://app_url/
Email Address | Your email address to get the details for client id
Logo URL | 
SMART Launch URL | URL to the launch.html file . Like https://app_url/launch.html
Scope Required | Select User-Specific Scope. Select online_access, launch, openid, profile and Patient.read
FHIR version | Select DSTU Final

and click Submit. This will send request to CERNER FHIR group for them to create client id for the app authorization.

After this you will receive an email stating what your Client ID and Launch URL is.

# Initializing the client
Before you are able to run any operations against the FHIR API using the JS client, you will need to initialize it first.

```javascript
<html lang="en">
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <title>SMART on FHIR Starter APP</title>
    <script src="./lib/fhir-client.js"></script>
    <script>
      FHIR.oauth2.authorize({
        "client_id": "CLIENT_ID",
        "scope":  "patient/Patient.read launch"
      });
    </script>
  </head>
  Loading...
</html>
```

> Make sure to replace CLIENT_ID with the client id provided in the email.

# Obtaining the context
Once the client is initialized, you can obtain the context in which it was launched (the user who authorized the client and, if applicable, the patient that has been selected) by using the following methods:

* *smart.user.read()*
* *smart.patient.read()*
	
Both of these return a jQuery Deferred object which you can register a success callback to process the returned FHIR resource.

```javascript
(function(window){
  window.extractData = function() {
    var ret = $.Deferred();

    FHIR.oauth2.ready(function(smart){
      var patient = smart.patient;
      var pt = patient.read();

      $.when(pt).done(function(patient){
         var gender = patient.gender;
         var dob = new Date(patient.birthDate);
         var fname = patient.name[0].given.join(" ");
         var lname = patient.name[0].family.join(" ");

          p = defaultPatient();
          p.birthday = {value:dob};
          p.gender={value:gender};
          p.givenName={value:fname};
          p.familyName={value:lname};

          ret.resolve(p);
      });
    });
    return ret.promise();
  };


  function defaultPatient(){
    return {
      'givenName':    {'value': null}
      ,'familyName':  {'value': null}
      ,'gender':      {'value': null}
      ,'birthday':    {'value': null}
    }
  };


})(window);
```

# Displaying the Resource
```javascript
<!DOCTYPE html>
<html>

  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <title>SMART ON FHIR STARTER APP</title>

   <script src="./lib/fhir-client.js"></script>
    <script src='./load_data.js'></script>
    <script src='./lib/jquery.min.js'></script>

    <script>
      extractData().then(function(p){

          var table = '<table><thead><th>First Name</th><th>Last Name</th><th>Gender</th><th>Birth Date</th></thead><tbody>';
          table += '<tr>';                      
          table += '<td>' + p.givenName.value + '</td>';
          table += '<td>' + p.familyName.value + '</td>';
          table += '<td>' + p.gender.value + '</td>';
          table += '<td>' + p.birthday.value + '</td>';
          table += '</tr>';
          table += '</tbody></table>';
          $( '#holder').html(table);         

      });

    </script>
  </head>
  <body style="margin: none;">
    <h1>SMART on FHIR STARTER APP ( PATIENT DEMOGRAPHICS)</h1>
    <div id='holder' style='display: none;'>
    </div>

  </body>
</html>
```

# Test
Commit your changes and hit the link mentioned in the email https://APP_URL/launch.html?iss=ABC&launch=XYZ. You should see your index.html page with Patient demographics in it

# Supported API operations

All operations available in fhir.js are supported. These include:

* read Read the current state of a given resource
* search Obtain a resource bundle matching specific search criteria
* fetchAll Retrieve the complete set of resources matching specific search criteria
and many others
* Please see the fhir.js documentation for the complete list of available operations.

# Reading a single resource

Instance-level operations in FHIR work with a single resource instance at a time. Two key operations are read and vread:

* smart.api.read({type: resourceType, id: resourceId}) Read the current state of a given resource
* smart.api.vread({type: resourceType, id: resourceId, versionId: versionId}) Read a specific version of a given resource
Just change resourceType to the name of a FHIR resource from this complete list.

# Searching for resources

The most important type-level operation is search, which allows you to find resources of a given type, with a set of filters applied.

To search for resources you’ll use either:

* smart.api.search({type: resourceType, query: queryObject}) when you want to search across patients, or

* smart.patient.api.search({type: resourceType, query: queryObject}) when you only want results for the patient in context.

(Note: in any case your app will only be able to read the data it’s authorized to see – so you could be lazy and query for “all” the data knowing that authorization restrictions will limit the data returned. But it’s a good practice to write explicit patient-level queries when that’s what you have in mind, because it will make your intentions more clear – and make your code more portable.)

# FHIR Search Parameters

When you perform a FHIR search operation, you’ll often want to apply filters to limit the results you get back. For example, you may want to fetch a list of HbA1c lab results when you’re not interested in cholesterol readings. You can create an expressive set of filters using FHIR’s “search parameters,” a set of constraints passed along as URL parameters. The fhir.js client used in the SMART on FHIR JS client has built-in support for all search parameters defined in FHIR.

To learn how these work, let’s look at an example from the FHIR spec. We’ll examine the Patient resource. FHIR defines a set of search parameters for this resource (and all other resources) at the bottom of the resource documentation page. Let’s say that we’d like to find all patients that have “John” or “Bob” in their first name and “Smith” in their last name. Here is how we can do this using the SMART on FHIR JS client:

smart.api.search({type: "Patient", query: {given: ["John", "Bob"], family: "Smith"})
Please see the fhir.js documentation for further examples.



