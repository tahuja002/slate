---
title: SMART on FHIR Starter APP

language_tabs:
  - javascript

search: true
---

# Introduction

This document will help you get started on how to use SMART on FHIR API to create your app with CERNER.

# APP Launch Flow

![alt text](images/ehr_launch_seq.PNG "High Level EHR APP Launch Flow")

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
* dist folder ( we will use this folder to keep our ES5 code )

You can also clone the project structure from [smart-app-starter](https://github.com/parthivbhagat/smart-app-starter). You will need `node.js` to be installed to be able to build this project.

# Install fhir-client.js

> package.json

```javascript
{
  "name": "smart-starter-app",
  "version": "1.0.0",
  "author": "Cerner Corporation, Cloud App Dev Platform",
  "description": "== README",
  "main": "./src/starter_app.js",
  "style": "dist/css/starter_app.css",
  "less": "src/less/manifest.less",
  "license": "== LICENSE",
  "homepage": "https://github.com/parthivbhagat/pb026393.github.io",
  "repository": {
    "type": "git",
    "url": "git@github.com:parthivbhagat/pb026393.github.io.git"
  },
  "dependencies":{
    "fhirclient": "^0.1.9",
    "jquery": "^2.1.4"
  },
  "scripts": {
    "build": "npm run build-es5 && npm run build-webpack && npm run build-minify",
    "build-doc": "jsdoc  --package package.json -d doc ./src/**/*.js",
    "build-es5": "babel src --out-dir dist",
    "build-webpack": "webpack --progress --colors --display-error-details",
    "build-minify": "npm run build-minify-js && npm run build-minify-css",
    "build-minify-js": "uglifyjs dist/js/starter_app.js > dist/js/starter_app.min.js",
    "build-minify-css": "lessc --clean-css dist/css/starter_app.css dist/css/starter_app.min.css",
    "lint": "npm run lint-js && npm run lint-css",
    "lint-js": "jshint src/*.js --config .jshintrc; exit 0",
    "test": "karma start test/karma.conf.js --single-run --no-auto-watch",
    "debug": "karma start test/karma.conf.js --browsers=Chrome --no-single-run --auto-watch --debug"
    
  },
  "sourceFiles": {
    "js": [
      "./src/js"
    ],
    "css": [
      "./src/css"
    ]
  },
  "testFiles": [
    "./test/spec"
  ],
  
  "devDependencies": {
    "babel-cli": "^6.9.0",
    "babel-eslint": "^6.0.4",
    "babel-loader": "^6.2.4",
    "babel-polyfill": "^6.9.1",
    "babel-preset-es2015": "^6.9.0",
    "commander": "^2.9.0",
    "css-loader": "^0.23.1",
    "csslint": "^0.10.0",
    "eslint": "^2.11.1",
    "extract-text-webpack-plugin": "^1.0.1",
    "jasmine-core": "^2.4.1",
    "jsdoc": "^3.4.0",
    "jshint": "^2.9.1",
    "karma": "^0.13.22",
    "karma-chrome-launcher": "^1.0.1",
    "karma-coverage": "^1.1.1",
    "karma-firefox-launcher": "^1.0.0",
    "karma-jasmine": "^0.3.8",
    "karma-phantomjs-launcher": "^1.0.0",
    "karma-requirejs": "^0.2.6",
    "karma-spec-reporter": "0.0.26",
    "karma-webpack": "^1.7.0",
    "less": "^2.6.1",
    "less-loader": "^2.2.3",
    "less-plugin-clean-css": "^1.5.1",
    "matchdep": "^1.0.0",
    "prettyjson": "^1.1.3",
    "requirejs": "^2.2.0",
    "style-loader": "^0.13.1",
    "uglify-js": "^2.6.2",
    "sinon": "^1.15.4",
    "webpack": "^1.13.0",
    "webpack-dev-server": "^1.14.1"
  }
}
```
In your package.json file create dependencies section and put fhirclient in it. It should look something like this

`"dependencies":{
    "fhirclient": "^0.1.9",
    "jquery": "^2.1.4"
}`

This will include fhir-client.js library when you run `npm install` on your local copy.

The SMART on FHIR JavaScript client library helps you build browser-based SMART apps that interact with a FHIR REST API server. It can help your app get authorization tokens, provide information about the user and patient record in context, and issue API calls to fetch clinical data. This tutorial will lead you through the basics of building a SMART app using the JavaScript client.

The SMART JS client uses the open-source fhir.js for interfacing with SMART API servers. Upon successfully initializing and negotiating the SMART authorization sequence, the client will expose one or two instances of the fhir.js client as applicable via the following handles :

* *smart.api* Non-context aware API for executing operations on all authorized resources
* *smart.patient.api* Context aware API which automatically applies its operations to the patient in context

# Registering SMART APP
Once we have the SMART App created per the Project Setup step, get the application hosted. Your application is now ready to be registered with CERNER. Go to the link [Developer Portal APP Registration](https://developerportal.devcernerpowerchart.com/register), sign into your CERNER Care Account and fill up following details:

Field | Description
--------- | -----------
App Name | Any name for your APP you want to give
SMART Launch URI | URL to the launch.html file . Like https://app_url/launch.html
Redirect URI | Just put your base app url. Like https://app_url/
App Type | SMART App type. Provider facing or Patient facing App.
Fhir Spec | The FHIR server version you want to target your SMART App against.
Authorized | Select yes. Authorized App will go through secured OAuth2 login.
Standard Scopes | These are standard scopes that are required to launch SMART App.
User Scopes | Select appropriate user scopes
Patient Scopes | Select appropriate patient scopes

and click Register. This will send request to CERNER FHIR group for them to create client id for the app authorization.

After this you will receive an email stating what your Client ID and Launch URL is.

# Initializing the client

> launch.html

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
    <script src='./dist/js/starter_app.min.js'></script>
    <script>
      FHIR.oauth2.authorize({
        'client_id': 'df7c5a17-52dd-4c88-8a32-cdfb557ba758',
        'scope':  'patient/Patient.read patient/Observation.read launch online_access openid profile'
      });
    </script>
  </body>
</html>
```

> Make sure to replace CLIENT_ID with the client id provided in the email.

Before you are able to run any operations against the FHIR API using the JS client, you will need to initialize it first.

Based on the client_id, current EHR user, the EHR makes a decision to approve or deny access. This decision is communicated to the app by redirection to the app's registered redirect URL. So always make sure we replace the CLIENT_ID with client id provided after your APP gets registered.

When an EHR user launches your app, you get a “launch request” notification. Just ask for the permissions you need using OAuth scopes like patient/*.read and once you’re authorized you’ll have an access token with the permissions you need – including access to clinical data and context like:

* which patient is in-context in the EHR
* which encounter is in-context in the EHR
* the physical location of the EHR user

Below are different scopes we can use.

Scope | Grants
--------- | -----------
patient/*.read | Permission to read any resource for the current patient
user/\*.\* | Permission to read and write all resources that the current user can access
openid profile | Permission to retrieve information about the current logged-in user
launch | Permission to obtain launch context when app is launched from an EHR
launch/patient | When launching outside the EHR, ask for a patient to be selected at launch time
offline_access | Request a refresh_token that can be used to obtain a new access token to replace an expired one, even after the end-user no long is online after the access token rexpires
online_access | Request a refresh_token that can be used to obtain a new access token to replace an expired one, and that will be usable for as long as the end-user remains online.

For our APP we will use Patient.read, Observation.read.
We will always include launch, online_access, openid profile scopes to our APP.

**Note:** Cerner does not allow use of wildcards(*). So instead of patient/\*.read you will need specify a particular scope of resource you will be using. Something like patient/Patient.read, patient/Observation.read etc. For list of resources visit [http://fhir.cerner.com/](http://fhir.cerner.com/) 

# Obtaining the context

> starter_app.js

```javascript
/*jshint esversion: 6 */
import Util from './util';
import Patient from './patient';
import FHIRClient from '../../node_modules/fhirclient/fhir-client.js';
import $ from '../../node_modules/jquery/src/jquery';

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

# Displaying the Resource

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
  <script src='./dist/js/starter_app.min.js'></script>
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

We will put the display logic in draw_visualization.js file. Here is what it should look like.

# Test
Commit your changes and hit the link mentioned in the email https://APP_URL/launch.html?iss=ABC&launch=XYZ. You should see your index.html page with Patient demographics in it.

# Reading a single resource

Instance-level operations in FHIR work with a single resource instance at a time. Two key operations are read and vread:

* *smart.api.read({type: resourceType, id: resourceId})* Read the current state of a given resource
* *smart.api.vread({type: resourceType, id: resourceId, versionId: versionId})* Read a specific version of a given resource

Just change resourceType to the name of a FHIR resource from this complete list.

# Searching for resources

To search for resources you’ll use either:

* *smart.api.search({type: resourceType, query: queryObject})* when you want to search across patients, or

* *smart.patient.api.search({type: resourceType, query: queryObject})* when you only want results for the patient in context.

# Create tests

> karma.conf.js

``` javascript
// Karma configuration
var webpack = require('karma-webpack');
var webpackConfig = require('../webpack.config');
var ExtractTextPlugin = require('extract-text-webpack-plugin');

webpackConfig.module.loaders = [
  {
    test: /\.(js|jsx)$/, exclude: /(node_modules)/,
    loader: 'babel-loader'
  },
  {
    test: /\.less$/,
    loader: ExtractTextPlugin.extract("style-loader", "css-loader!less-loader")
  }
];

module.exports = function(config) {
  config.set({

    // base path that will be used to resolve all patterns (eg. files, exclude)
    basePath: '../',


    // frameworks to use
    // available frameworks: https://npmjs.org/browse/keyword/karma-adapter
    frameworks: ['jasmine'],


    // list of files / patterns to load in the browser
    files: [
      'test/**/*_spec.js',
      'lib/**.js'
    ],
    plugins: [
      webpack, 
      'karma-jasmine',
      'karma-chrome-launcher',
      'karma-firefox-launcher',
      'karma-phantomjs-launcher',
      'karma-coverage',
      'karma-spec-reporter'
    ],
    browsers: [ 'PhantomJS' ],
    preprocessors: {
      'test/**/*_spec.js': ['webpack'],
      'src/**/*.js': ['webpack', 'coverage']
    },
    reporters: ['spec', 'coverage'],
    coverageReporter: {
      dir: 'target/reports/coverage',
      reporters: [
        { type: 'html', subdir: 'report-html' },
        { type: 'lcov', subdir: 'report-lcov' },
        { type: 'cobertura', subdir: '.', file: 'cobertura.txt' }
      ]
    },
    webpack: webpackConfig,
    webpackMiddleware: { noInfo: true }
  });
};

```
> starter_app_spec.js

```javascript
import StarterApp from 'js/starter_app';
import Patient from 'js/patient';

describe ('StarterApp', function() {
  
  describe ('FHIR service', function (){
    it ('checked if ready method gets invoked', function () {
      var mock = jasmine.createSpyObj('FHIR.oauth2', ['ready']);
      mock.ready();
      expect(mock.ready).toBeDefined();
      expect(mock.ready).toHaveBeenCalled();
     
    });
  });
  
  describe ('Extract Data', function (){
    it ('checked if onReady method gets invoked', function () {
      var mock = jasmine.createSpyObj('StarterApp.extractData', ['onReady']);
      mock.onReady();
      expect(mock.onReady).toBeDefined();
      expect(mock.onReady).toHaveBeenCalled();      
    });
  });
 
  describe ('Patient Constructor', function (){
    it ('checked if patient Constructor returns object with blank elements', function () {
      var patient = new Patient();
      
      expect(patient.fname).toEqual('');
      expect(patient.lname).toEqual('');
      expect(patient.gender).toEqual('');
      expect(patient.birthday).toEqual('');
      expect(patient.age).toEqual('');
      expect(patient.obv.height).toEqual('');
      expect(patient.obv.systolicbp).toEqual('');
      expect(patient.obv.diastolicbp).toEqual('');
    });
  });




});

```
We are going to use karma and jasmine to write our tests

**Suites:** `describe` Your Tests

A test suite begins with a call to the global Jasmine function `describe` with two parameters: a string and a function. The string is a name or title for a spec suite – usually what is being tested. The function is a block of code that implements the suite.

**Specs**

Specs are defined by calling the global Jasmine function `it`, which, like `describe` takes a string and a function. The string is the title of the spec and the function is the spec, or test. A spec contains one or more expectations that test the state of the code. An expectation in Jasmine is an assertion that is either true or false. A spec with all true expectations is a passing spec. A spec with one or more false expectations is a failing spec.

**Expectations**

Expectations are built with the function `expect` which takes a value, called the actual. It is chained with a Matcher function, which takes the expected value.

**Spies:** `createSpyObj`

In order to create a mock with multiple spies, use `jasmine.createSpyObj` and pass an array of strings. It returns an object that has a property for each string that is a spy.

For more information look at [Jasmine](http://jasmine.github.io/2.1/introduction.html)


# Run tests

1. Include in script section of package.json following:
`"test": "karma start test/karma.conf.js --single-run --no-auto-watch"`

2. Run `npm run test` to run tests that we have created