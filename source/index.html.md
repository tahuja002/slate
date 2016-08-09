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
* load_data.js 
* draw_visualization.js and 
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
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <title>SMART on FHIR Starter APP</title>    
  </head>
  Loading...
  <body>
    <script src="./lib/fhir-client.js"></script>
    <script>
      FHIR.oauth2.authorize({
        'client_id': 'df7c5a17-52dd-4c88-8a32-cdfb557ba758',
        'scope':  'patient/Patient.read launch online_access openid profile'
      });
    </script>
  </body>
</html>
```

> Make sure to replace CLIENT_ID with the client id provided in the email.

# Obtaining the context
Once the client is initialized, you can obtain the context in which it was launched (the user who authorized the client and, if applicable, the patient that has been selected) by using the following methods:

* *smart.user.read()*
* *smart.patient.read()*
	
Both of these return a jQuery Deferred object which you can register a success callback to process the returned FHIR resource.


Following operations available in [fhir.js](https://github.com/FHIR/fhir.js) are supported:

* *read* Read the current state of a given resource
* *search* Obtain a resource bundle matching specific search criteria
* *fetchAll* Retrieve the complete set of resources matching specific search criteria
and many others

Please see the fhir.js documentation for the complete list of available operations.

```javascript
(function(window){
  window.extractData = function() {
    const ret = $.Deferred();
    
    function isLeapYear(year) {
      return new Date(year, 1, 29).getMonth() === 1;
    }

    function calculateAge(date) {
      const d = new Date(date), now = new Date();
      let years = now.getFullYear() - d.getFullYear();
      d.setFullYear(d.getFullYear() + years);
      if (d > now) {
          years--;
          d.setFullYear(d.getFullYear() - 1);
      }
      const days = (now.getTime() - d.getTime()) / (3600 * 24 * 1000);
      return years + days / (isLeapYear(now.getFullYear()) ? 366 : 365);
    }

  
    function defaultPatient() {
      return {
       'fname' : {'value': null},
       'lname' : {'value': null},
       'gender' : {'value': null},
       'birthday' : {'value': null},
       'age' : {'value': null}
      };
    }

    function onError() {
      console.log('Loading error', arguments);
      ret.reject();
    }

    function onReady(smart)  {
      if (smart.hasOwnProperty('patient')) { 
        const patient = smart.patient;
        const pt = patient.read();
        
        $.when(pt).fail(onError);

        $.when(pt).done(function(patient) {
          const gender = patient.gender;
          const dob = new Date(patient.birthDate);     
          const day = dob.getDate(); 
          const monthIndex = dob.getMonth() + 1;
          const year = dob.getFullYear();

          const dobStr = monthIndex + '/' + day + '/' + year;
          
          const fname = patient.name[0].given.join(' ');
          const lname = patient.name[0].family.join(' ');
          const age = parseInt(calculateAge(dob));
          
          let p = defaultPatient();
          p.birthday = {value:dobStr};
          p.gender = {value:gender};
          p.fname = {value:fname};
          p.lname = {value:lname};
          p.age = {value:age};
          ret.resolve(p);
        });
      } else { 
        onError();
      }
      
    
      
    }

    FHIR.oauth2.ready(onReady, onError);

    return ret.promise();
  };
  
})(window);
```

# Displaying the Resource

We will put the display logic in draw_visualization.js file. Here is what it should look it
>draw_visualization.js

```javascript
function drawVisualization(p) { 
    let table = '<table><thead><th>First Name</th><th>Last Name</th><th>Gender</th><th>Birth Date</th><th>Age</th></thead><tbody>';
    table += '<tr>';            
    table += '<td>' + p.fname.value + '</td>';
    table += '<td>' + p.lname.value + '</td>';
    table += '<td>' + p.gender.value + '</td>';
    table += '<td>' + p.birthday.value + '</td>';
    table += '<td>' + p.age.value + '</td>';
    table += '</tr>';
    table += '</tbody></table>';
    $('#holder').html(table);  
}
```
>index.html

```javascript
<!DOCTYPE html>
<html>  
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <title>SMART STARTER APP</title>    
  </head>
  <body style="margin: none;">
    <h2>SMART STARTER APP ( PATIENT DEMOGRAPHICS )</h2>
    <div id='errors'>
    </div>
    <div id='holder'>
    </div>
   
  </body>
  <script src="./lib/fhir-client.js"></script>
  <script src='./load_data.js'></script>
  <script src='./lib/jquery.min.js'></script>
  <script src='./draw_visualization.js'></script>
  <script>
      extractData().then(
        function(p) {          
          drawVisualization(p);
        }, 

        function() {
          $('#errors').html('<p> Failed to call FHIR Service </p>');
        }
      );

  </script>
</html>
```

# Test
Commit your changes and hit the link mentioned in the email https://APP_URL/launch.html?iss=ABC&launch=XYZ. You should see your index.html page with Patient demographics in it

# Reading a single resource

Instance-level operations in FHIR work with a single resource instance at a time. Two key operations are read and vread:

* *smart.api.read({type: resourceType, id: resourceId})* Read the current state of a given resource
* *smart.api.vread({type: resourceType, id: resourceId, versionId: versionId})* Read a specific version of a given resource

Just change resourceType to the name of a FHIR resource from this complete list.

# Searching for resources

To search for resources you’ll use either:

* *smart.api.search({type: resourceType, query: queryObject})* when you want to search across patients, or

* *smart.patient.api.search({type: resourceType, query: queryObject})* when you only want results for the patient in context.






