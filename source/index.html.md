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
* src/index.js 
* src/starter_app.js 
* src/patient.js 
* src/observation.js 
* src/util.js 
* src/draw_visualization.js and 
* lib folder
* dist folder ( we will use this folder to keep our ES5 code )

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

> starter_app.js

```javascript
/*jshint esversion: 6 */
import Util from './util';
import Patient from './patient';

class StarterApp {
  static extractData() {
   
    const ret = $.Deferred();     
    
    function onError() {
      console.log('Loading error', arguments);
      ret.reject();
    }     

    function onReady(smart)  {
      if (smart.hasOwnProperty('patient')) { 
        const patient = smart.patient;
        const pt = patient.read();
        const obv = smart.patient.api.fetchAll({
                      type: 'Observation', 
                      query: {
                        code: {
                          $or: ['http://loinc.org|8302-2', 'http://loinc.org|8462-4',
                                'http://loinc.org|8480-6', 'http://loinc.org|2085-9',
                                'http://loinc.org|2089-1']
                              }
                             }
                    });
        
        $.when(pt, obv).fail(onError);

        $.when(pt, obv).done(function(patient, obv) {
          const byCodes = smart.byCodes(obv, 'code');
          const gender = patient.gender;
          const dob = new Date(patient.birthDate);     
          const day = dob.getDate(); 
          const monthIndex = dob.getMonth() + 1;
          const year = dob.getFullYear();

          const dobStr = monthIndex + '/' + day + '/' + year;
          
          const fname = patient.name[0].given.join(' ');
          const lname = patient.name[0].family.join(' ');
          
          const height = byCodes('8302-2');
          const systolicbp = byCodes('8480-6');
          const diastolicbp = byCodes('8462-4');
          const hdl = byCodes('2085-9');
          const ldl = byCodes('2089-1');

          let p = new Patient();          
          p.birthday = dobStr;
          p.gender = gender;
          p.fname = fname;
          p.lname = lname;
          p.age = parseInt(Util.calculateAge(dob));

          if(typeof height[0] !== 'undefined') {
            p.obv.height = height[0].valueQuantity.value;
          }
          
          if(typeof systolicbp[0] !== 'undefined') {
            p.obv.systolicbp = systolicbp[0].valueQuantity.value;
          }

          if(typeof diastolicbp[0] !== 'undefined') {
            p.obv.diastolicbp = diastolicbp[0].valueQuantity.value;
          }
          
          if(typeof hdl[0] !== 'undefined') {
            p.obv.hdl = hdl[0].valueQuantity.value;
          }

          if(typeof ldl[0] !== 'undefined') {
            p.obv.ldl = ldl[0].valueQuantity.value;
          }
          ret.resolve(p);
        });
      } else { 
        onError();
      }
      
    }

    FHIR.oauth2.ready(onReady, onError);

    return ret.promise();
  }
}

export default StarterApp;

```

> index.js

```javascript
import StarterApp from './starter_app';

window.StarterApp = StarterApp;
```

> util.js

```javascript
/*jshint esversion: 6 */
class Util{
  static isLeapYear(year) {
    return new Date(year, 1, 29).getMonth() === 1;
  }

  static calculateAge(date) {
    if (Object.prototype.toString.call(date) === '[object Date]' && !isNaN(date.getTime())) {
      const d = new Date(date), now = new Date();
      let years = now.getFullYear() - d.getFullYear();
      d.setFullYear(d.getFullYear() + years);
      if (d > now) {
        years--;
        d.setFullYear(d.getFullYear() - 1);
      }
      const days = (now.getTime() - d.getTime()) / (3600 * 24 * 1000);
      return years + days / (this.isLeapYear(now.getFullYear()) ? 366 : 365);
    }
    else {
      return undefined;
    }
    
  }
}

export default Util;
```

> patient.js

```javascript
/*jshint esversion: 6 */
import Observations from './observations';

class Patient {  
  constructor() {
    this.fname = '';
    this.lname = '';
    this.gender = '';
    this.birthday = '';
    this.age = '';
    this.obv = new Observations();
  }
}

export default Patient;
```

> observation.js

```javascript
/*jshint esversion: 6 */
class Observations {  
  constructor() {
    this.height = '';
    this.systolicbp = '';
    this.diastolicbp = '';
    this.ldl = '';
    this.hdl = '';
  }
}

export default Observations;
```

# Transpiling the ES6 code to ES5 code
We will use bable to transpile out ES6 code in src folder to ES5 code in dist folder

# Displaying the Resource

We will put the display logic in draw_visualization.js file. Here is what it should look like
>draw_visualization.js

```javascript
/*jshint esversion: 6 */
function drawVisualization(p) { 
  $('#holder').show();
  $('#fname').html(p.fname);
  $('#lname').html(p.lname);
  $('#gender').html(p.gender);
  $('#birthday').html(p.birthday);  
  $('#age').html(p.age);
  $('#height').html(p.obv.height);
  $('#systolicbp').html(p.obv.systolicbp);
  $('#diastolicbp').html(p.obv.diastolicbp);
  $('#ldl').html(p.obv.ldl);
  $('#hdl').html(p.obv.hdl);
}
```
>index.html

```javascript
<!DOCTYPE html>
<html>  
  <head>
    <meta http-equiv='Content-Type' content='text/html; charset=utf-8' />
    <meta http-equiv='X-UA-Compatible' content='IE=edge' />
    <title>SMART STARTER APP</title>    
  
    <link rel='stylesheet' type='text/css' href='./dist/css/starter_app.min.css'>
  </head>
  <body style='margin: none;'>
    <h2>SMART STARTER APP</h2>
    <div id='errors'>
    </div>
    <div id='holder' style='display:none;'>

      <h2>Patient Resource</h2>
      <table>
        <tr>
          <th>First Name:</th>
          <td id='fname'></td>            
        </tr>
        <tr>
          <th>Last Name:</th>
          <td id='lname'></td>
        </tr>
        <tr>
          <th>Gender:</th>
          <td id='gender'></td>          
        </tr>
        <tr>
          <th>Date of Birth:</th>
          <td id='birthday'></td>            
        </tr>
        <tr>
          <th>Age:</th>
          <td id='age'></td>            
        </tr>       
      </table>


      <h2>Observation Resource</h2>
      <table>
        <tr>
          <th>Height:</th>          
          <td id='height'></td>            
        </tr>
        <tr>
          <th>Systolic Blood Pressure:</th>
          <td id='systolicbp'></td>
            
        </tr>
        <tr>
          <th>Diastolic Blood Pressure:</th>
          <td id='diastolicbp'></td>
        </tr>
        <tr>
          <th>LDL:</th>
          <td id='ldl'></td>
        </tr>
        <tr>
          <th>HDL:</th>
          <td id='hdl'></td>
        </tr>
      </table>
    </div>   
  </body>
  <script src='./lib/fhir-client.js'></script>
  <script src='./dist/js/starter_app.min.js'></script>
  <script src='./lib/jquery.min.js'></script>
  <script src='./dist/js/draw_visualization.js'></script>
  <script>
    StarterApp.extractData().then(
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






