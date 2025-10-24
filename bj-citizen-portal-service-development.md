
For internal use

# Beninese Citizen Portal
## Service Development

**BJ-SDG-SD**  
**Version 0.3**  
**12.07.2020**


---


# Table of Contents

## 1. Introduction
* 1.1. Glossary ........................................................................................................ 1  
* 1.2. References .................................................................................................... 1  

## 2. Basic Concepts
* 2.1. Data Handling ............................................................................................ 3  

## 3. Configuration Files .......................................................................................... 5  

## 4. An Example
* 4.1. Base ........................................................................................................... 7  
* 4.2. Start stage ................................................................................................. 7  
* 4.3. Intermediate stage ..................................................................................... 8  
* 4.4. Main stage .................................................................................................. 9  
* 4.5. Official Intermediate stage ........................................................................ 10  
* 4.6. Final stages ............................................................................................... 10  

## 5. Translations .................................................................................................... 12  

## 6. Main Stage ..................................................................................................... 13  

## 7. Intermediate Stages
* 7.1. Start Stage ............................................................................................... 15  
* 7.2. UI Stage .................................................................................................... 16  
* 7.3. Payment Stage .......................................................................................... 17  
* 7.4. UXP Integration Stage .............................................................................. 18  
* 7.5. UXP REST Integration Stage ..................................................................... 21  
* 7.6. Document Generation Stage ..................................................................... 22  
* 7.7. E-mail Stage ............................................................................................. 23  
* 7.8. Notification Stage ..................................................................................... 24  
  - 7.8.1. Target ............................................................................................... 25  
* 7.9. Hardcoded Data Stage ............................................................................. 26  
* 7.10. Blacklist Stage ....................................................................................... 26  
* 7.11. Person Data Stage .................................................................................. 27  
* 7.12. Alert Stage ............................................................................................. 29  
* 7.13. Share Files Stage .................................................................................. 29  

## 8. Transitions
* 8.1. Main Transition ....................................................................................... 31  
  - 8.1.1. Automatic Transitions ....................................................................... 32  
* 8.2. Intermediate stage transitions ............................................................... 33  
  - 8.2.1. Single Transition ............................................................................... 33  
  - 8.2.2. Map by Meta Transition ..................................................................... 33  
  - 8.2.3. Exception handling ........................................................................... 34  


---


# 9. Permissions
## 9.1. Hardcoded Array Permissions
## 9.2. Meta Array Permissions
## 9.3. Public Services

# 10. Appendix


---


# 1. Introduction

This document describes the process of developing services for use with the service engine of the Beninese Citizen Portal.

## 1.1. Glossary

**Application** - Formal request for a service provided via Citizen Portal.  
**Automatic stage** - Intermediate stage that is run automatically and requires no input from the user during its operations.  
**CatIS** - CatIS is supplementary instrument for coordination of state information systems, the support systems for the development of information systems and services, the tool for maintenance interoperability assets.  
**Intermediate stage** - Building block of the service process flow, representing a process done during the transition between main stages.  
**Main stage** - Building block of the service process flow, representing a state of completion of a part of the process.  
**Manual stage** - Intermediate stage that requires input from the user to complete its process.  
**Path** - A flow path of intermediate stages between main stages.  
**Service** - Digital service which can be used via Citizen Portal.  
**Service configuration** - JSON file which holds the process flow defined for a service version. Can start in exactly one main stage, but can branch internally, having multiple potential end main stages.  
**UXP** - Unified eXchange Platform is a technology that enables peer-to-peer data exchange over encrypted and mutually authenticated channels. It is based on a decentralised architecture where each peer has an information system that will be connected with other peers’ systems.

## 1.2. References

* [CatIS] Beninese Catalogue of Interoperable Solutions. https://catis.gouv.bj/  
* [KKiaPay] KKiaPay Official Documentation. https://docs.kkiapay.me/  
* [BJ-SDG-UI] Beninese Citizen Portal: User Interface Configuration Specification. *bj-citizen-portal-ui-stage.adoc*  
* [BJ-SDG-UXP] Beninese Citizen Portal: UXP Integration. *bj-citizen-portal-uxp-integration.adoc*  
* [BJ-SDG-PDF] Beninese Citizen Portal: PDF Generation. *bj-citizen-portal-pdf-generation.adoc*  


---


# 2. Basic Concepts

Citizen Portal provides a catalogue of public services. The non-technical description of services are described in **Catalogue of Governmental Institutions and Public Services (CatIS)**. There are three main types of services:

* External (digital) services — these are services which exist on external websites and the Citizen Portal redirects the user to the correct page.
* Manual services — these are services which only contain a description and the user must apply in person in the respective institution.
* E-services — these are services which are implemented and used in the Citizen Portal. This is the core functionality of the Citizen Portal

This document focuses on e-services (hereafter services) — the parts which make up a service and how to configure it.

An *e-service* is a service that can be used via the Citizen Portal. Each service is described as a process where a user of the portal submits a digital *application*. The application is then processed automatically or manually. The processing of the application consist of different stages that use information from the application to interact with external systems and modify the application.

The system has multiple types of stages that can be used to build services. For example, a simple e-service might have the following process that the applications follow:

<table>
<thead>
<tr>
  <td>New application</td>
<td>UI Stage</td>
<td>Payment</td>
<td>UXP Stage</td>
<td>PDF gen.</td>
<td>Completed application</td>
</tr>
</thead>
<tbody>
<tr>
  <td>
    <pre>{
"id": "123"
}</pre>
    Empty application
  </td>
<td>
    <pre>{
"id": "123",
"p": "OK"
}</pre>
    Filled application
  </td>
<td>
    <pre>{
"id": "123",
"p": "OK"
}</pre>
    Paid application
  </td>
<td>
    <pre>{
"id": "123",
"p": "OK",
"r": "321"
}</pre>
    Submitted application
  </td>
<td>
    <pre>{
"id": "123",
"p": "OK",
"r": "321"
} + PDF</pre>
  </td>
<td>
    Completed application and certificate
  </td>
</tr>
</tbody>
</table>

**Figure 1. e-Service Process Overview**

1. A *UI stage* allows the user to enter the initial information for the application.  
2. A *Payment stage* allows the user to pay the necessary processing fee.  
3. A *UXP stage* submits the information to an external system via UXP and retrieves a response.  
4. A *PDF generation stage* generates a document with the response to the application that the user can download.

The service process consists of **main stages** and **intermediate stages**.

Main stages do not define a specific operation themselves, but represent a state that an application has reached (e.g. *FINISHED*, *REJECTED*, *ISSUED* etc). The subflows, which are used to move between main stages, are called **paths**. Every main stage can have multiple


---


outgoing paths. These paths define, which users are allowed to perform it, using a **permissions** configuration.

The paths between main stages consist of intermediate stages. Each intermediate stage defines an action to be performed with the application. Intermediate stages are divided into **automatic** and **manual** stages.

* Automatic stages are operations, which are performed without any user input (e.g. sending an email to the applicant). If multiple automatic stages follow each other in the flow, they are all processed sequentially without interruptions.
* A manual stage is a process, which requires input from the user (e.g. a form to be filled out).

Each intermediate stage also defines a **transitions** configuration, which defines the next stage the application will move to. This allows conditional branching within paths.

## 2.1. Data Handling

Different values in the application can be referred to using their *path*. The path uses a simple *dot notation*. For example, `data.name` refers to a value of the *name* field of an object that is the value of field `data`. The top-level object must always be one of the following:

**data**  
Contains information that must be retrieved for an individual application and cannot be used in conditions for transitions. This should be the preferred object to store most of the application information.

**metaData**  
Contains information that will be always retrieved with the list of applications and can be used to check conditions for transitions.

> The notation does not support wildcards and access to individual list elements.

Every application has its own json data tree, which has two root components **data** and **metaData**. This data tree can be read and modified by implementations of intermediate stages. This is the main method of passing data between stages.

The **metaData** subtree should hold data fields that are necessary for application flow handling (evaluating transition outcomes, permissions, etc.) and displaying basic application data (e.g. comments from officials in the timeline). Generally everything else should be held in the data subtree.

Many stages in the service configuration can point to fields in the application data tree. This is usually done with a path that uses dots as a delimiter. For example, given an initial data tree of an application:


---


```json
{
  "metaData": {
    "citizen": ["1234567890"]
  },
  "data": { }
}
```

Accessing the owning citizen’s NPI could be done with the path `metaData.citizen.0`, which points to the value `1234567890`. If a part of the path is a number, then it is considered an array index.


---


# 3. Configuration Files

## Service example structure

```
 ├──   templateRoot
 │     ├──   PS00000
 │     │     └──  0.1
 │     │          ├──   email
 │     │          │     └──  demo-service-email-body.txt
 │     │          ├──   notification
 │     │          │     └──  demo-service-sms-body.txt
 │     │          ├──   pdf
 │     │          │     ├──  footer-demo-service-certificate.html
 │     │          │     ├──  header-demo-service-certificate.html
 │     │          │     └──  demo-service-certificate.html
 │     │          ├──   ui
 │     │          │     └──  demo-service-ui.json
 │     │          ├──   uxp
 │     │          │     └──  uxp-request.xml
 │     │          └──   PS00000.serviceconf.json
```

All the service configuration files are located in the `templateRoot` directory, which can be configured in the Citizen Portal configuration file.

### citizen-portal.ini

```
...
[Application-Management]
templateRoot=/etc/citizen-portal/templates
...
```

The service configuration file must be placed in the correct sub-directory.

* First level directory name must match the service id (`PS00000` in the example above) of the service configuration json.
* Second level directory name must match the service version (`0.1` in the example above) of the service configuration json.

Therefore, if the service configuration has the following service id and version:

### dev-guide.serviceconf.json

```json
{
      "serviceId": "dev-guide",
      "serviceVersion": "1.0",
      ...
}
```

Then the file must be placed in:


---


```
└──    templateRoot
       └──  dev-guide
            └──  1.0
                 └── dev-guide.serviceconf.json
```

When the Citizen Portal application is started, the `templateRoot` directory is scanned for files with the `.serviceconf.json` extension and the database is automatically updated. Template files are kept in their respective directories (*i.e. UI configurations are in ui sub-directory, pdf templates in pdf sub-directory etc*). Each stage can specify which files they need, but all are relative to the path of the service `/templateRoot/serviceId/serviceVersion/`.


---


# 4. An Example

This example provides a general overview. See further sections in the document for details and technical specifications about each topic. This section will go through writing a simple service configuration and explain the different parts of the configuration. The full service configuration can be found in the Appendix.

## 4.1. Base

```json
{
  "serviceId": "dev-guide",
  "serviceVersion": "1.0",
  "stages": {}
}
```

The service configuration is a JSON object that contains three mandatory fields on the root level. The `serviceId` and `serviceVersion` fields are strings, which link the service with the service description in CatIS. In order for the service to be usable (i.e. for the Apply button to appear) the service must have a configuration loaded to the portal’s database and the e-service section in CatIS must have an "Active" ("Actif") status. When the Apply button is clicked a new application is started.

The `stages` field is an object which contains all the main and intermediate stages in no particular order. The keys of the object are the ids of the stages. These ids are used to reference the stage from other stages and must be unique within the service.

The convention is to use CAPITAL letters for main stages (e.g. "ISSUED") and lowercase for intermediate stages (e.g. 'generate-pdf').

## 4.2. Start stage


---


```json
"start": {
  "type": "start",
  "shortTitle": {
    "en": "STARTED",
    "fr": "DÉBUT"
  },
  "title": {
    "en": "Application started",
    "fr": "Demande soumise"
  },
  "pathTitle": {
    "en": "Continue application creation",
    "fr": "Poursuivre la demande"
  },
  "permissions": {
    "type": "meta-array",
    "actor": "CITIZEN",
    "metaPathToArray": ["citizen"]
  },
  "transitions": {
    "type": "single",
    "nextStage": "citizen-input"
  }
}
```

Every service must contain exactly one **start stage**. This stage defines some basic information that is shown to the user in the applications detail view, until the first main stage is reached. It also sets up the rules of who can interact with the application in the starting path of the application. A more detailed description can be found in Section Start Stage.

The `permissions` object defines, which users can perform operations that follow the start stage. In this example, the `permissions` object’s type declares that the list of these users can be found in the `metaData` of the application. The `actor` field states the strings in the array reference NPIs of citizens who use the portal.

When a new application is started a `citizen` array is created within the `metaData`, which contains the NPI of the applicant. The `metaPathToArray` defines the path in `metaData` used to find an array of strings. Here it points to the `citizen` array created by the engine, which contains the NPI of the citizen as a string. This makes sure that the citizen who created the application can interact with the following operations.

More information about permissions can be found in Section Start Stage.

## 4.3. Intermediate stage

```json
"citizen-input": {
  "type": "ui",
  "uiConfiguration": "citizen-input.json",
  "transitions": {
    "type": "single",
    "nextStage": "REQUESTED"
  }
}
```


---


The two mandatory fields for every stage configuration are `type` and `transitions`. For further details see Section Intermediate Stages.

* `type` specifies the type of the stage.
* `transitions` specifies the next stage.

In the example above, the first operational stage of this application is a UI Stage. This stage shows the applicant a form, defined by the `uiConfiguration` JSON file (citizen-input.json describes the UI). Once the user submits the form, the application processing moves on to the `REQUESTED` stage.

#### 4.4. Main stage

```json
"REQUESTED": {
  "type": "main",
  "shortTitle": {
    "en": "Requested"
  },
  "title": {
    "en": "Certificate has been requested."
  },
  "transitions": [
    {
      "id": "review",
      "title": {
        "en": "Review the application"
      },
      "nextStage": "official-review",
      "resultMainStages": ["APPROVED", "REJECTED"],
      "permissions": {
        "type": "hardcoded-array",
        "actor": "OFFICIAL",
        "array": ["Official-Group-1", "Official-Group-2"]
      }
    }
  ]
}
```

This is the first main stage that the application will reach.

In the application list, the `shortTitle` of the last main stage reached by the application is shown to the user. This field should be short, preferably a single word.  
The `title` is used in the timeline view of the application to describe the main stage.

For main stages, the `transitions` field is an array of objects instead of an object. Each object is one path that can be started from the main stage. These paths define the first stage they move to, when starting the path and the permissions of who can start the path.

See Main Stage for more detailed information.


---


# 4.5. Official Intermediate stage

```json
"official-review":  {
  "type": "ui",
  "uiConfiguration": "official-review.json",
  "transitions": {
    "type": "map-by-meta",
    "metaPathToKey": ["official", "choice"],
    "map": {
      "approve": "APPROVED",
      "reject": "REJECTED"
    },
    "onStageException": {
      "REQUIRED_FIELDS_MISSING": "search-terms",
      "FIELDS_NOT_ALLOWED": "search-terms"
    }
  }
}
```

The next stage is a UI stage similar to `citizen-input`, but this one gives the user the option to branch in different directions. The ui form will set the `metaData.official.choice` value to either `approve` or `reject`. Depending on that value, after the processing of the ui stage, the next stage is evaluated.

Transitions of intermediate stages can include a `onStageException` map, which maps certain exceptions to stages. If the stage processing returns an exception of a type that is specified in the map, the process flow will continue to the stage it is mapped to and ignore the transition configuration.

> ❗ If the stage processing returns an excpetion and it is not in the `onStageException` map, then it will be ignored and the process flow will move on as usual.

# 4.6. Final stages


---


```json
"APPROVED": {
  "type": "main",
  "shortTitle": {
    "en": "Approved"
  },
  "title": {
    "en": "The request for a certificate has been approved"
  },
  "flags": ["final", "accept"],
  "transitions": []
},

"REJECTED": {
  "type": "main",
  "shortTitle": {
    "en": "Rejected"
  },
  "title": {
    "en": "The request for a certificate has been rejected"
  },
  "flags": ["final", "reject"],
  "transitions": []
}
```

The service has two possible final main stages, depending on the choice made by the official. These main stages have strings in their `flags` array. The `final` string marks a main stage, that when in this stage, the application can be considered completed. The `accept` and `reject` flags make the stage show up with a special color in the application list and timeline.


---


# 5. Translations

Certain fields denote UI strings. These fields are objects, in which the key represents a language code (e.g. `en` or `fr`) and the value defines the translation for that language. This kind of language maps are common in many configuration structures of the services for values that are displayed to the user. If a translation is not specified for a language, the stage ID will be displayed to the user.

```json
"start": {
  ...
  "shortTitle": {
    "en": "STARTED",
    "fr": "DÉBUT"
  },
  "title": {
    "en": "Application started",
    "fr": "Demande soumise"
  },
  "pathTitle": {
    "en": "Continue application creation",
    "fr": "Poursuivre la demande"
  }
  ...
}
```


---


# 6. Main Stage

A main stage represents a state an application has reached. It performs no operations itself, but connects different processing paths together.

When the application needs to be passed between different actors, it needs to go through a main stage. For example, if the application is created by a citizen, the citizens path should end in a main stage, from which an official can start their own path.

Example

```json
"REQUESTED": {
  "type": "main",
  "shortTitle": {
    "en": "Requested"
  },
  "title": {
    "en": "Certificate has been requested."
  },
  "flags": [],
  "transitions": [
    {
      "id": "review",
      "title": {
        "en": "Review the application"
      },
      "nextStage": "official-review",
      "resultMainStages": ["APPROVED", "REJECTED"],
      "permissions": {
        "type": "hardcoded-array",
        "actor": "OFFICIAL",
        "array": ["Official-Group-1", "Official-Group-2"]
      }
    }
  ]
}
```

Table 1. Fields  

<table>
    <thead>
    <tr>
        <th>Field name</th>
        <th>Description</th>
        <th>Requir
ed</th>
    </tr>
    </thead>
    <tr>
        <td>type</td>
<td>Type of the stage. main for main stages.</td>
<td>Yes</td>
    </tr>
<tr>
        <td>shortTitle</td>
<td>State of the application shown in the application list if this is
the last main stage that has been reached.</td>
<td>Yes</td>
    </tr>
<tr>
        <td>title</td>
<td>State of the application shown in the timeline.</td>
<td>Yes</td>
    </tr>
<tr>
        <td>flags</td>
<td>List of strings that can be set for a main stage. See the
Flags table for more information.</td>
<td>Yes</td>
    </tr></table>



---



<table>
  <thead>
    <tr>
      <th>Field name</th>
      <th>Description</th>
      <th>Required</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>transitions</td>
<td>List of paths outgoing from the main stage. See <a href="#">Main Transition</a> for description of the path configuration. Only a single path is supported here.</td>
<td>Yes</td>
    </tr>
  </tbody>
</table>

Table 2. Flags

<table>
  <thead>
    <tr>
      <th>Flag</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>final</td>
<td>Marks the main stage as a final stage. When the last main stage that an application has reached has this flag, the application is considered completed for statistics. <b>Does not functionally prevent the application from continuing the process.</b> For almost all cases the final flag should only be used on the last stage.</td>
    </tr>
<tr>
      <td>accept</td>
<td>Marks the main stage as an accepted state. Modifies the colors of the stage in the frontend to be green (positive).</td>
    </tr>
<tr>
      <td>reject</td>
<td>Marks the main stage as a rejected state. Modifies the colors of the stage in the frontend to be red (negative).</td>
    </tr>
  </tbody>
</table>



---


# 7. Intermediate Stages

Many intermediate stages make use of separate template files. For information on where these files need to be stored, see Configuration Files.

> All the intermediate stages share some common fields, these fields are not duplicated in the table for each description, but can be used for every intermediate stage.

<table>
<thead>
<tr>
<th>Field name</th>
<th>Description</th>
<th>Required</th>
</tr>
</thead>
<tbody>
<tr>
<td>type</td>
<td>Type of the stage.</td>
<td>Yes</td>
</tr>
<tr>
<td>transitions</td>
<td>The configuration of the transition after the stage process.</td>
<td>Yes</td>
</tr>
</tbody>
</table>

## 7.1. Start Stage

The `start` stage is an intermediate stage that has special functionality within the service engine. Every service must contain exactly one start stage. It defines the entry point of the service and the permissions of the starting path. As long as an application has not reached any main stages, the application list and timeline will display information defined in the start stage.

Example

```json
"start": {
  "type": "start",
  "shortTitle": {
    "en": "Started"
  },
  "title": {
    "en": "Creating the application has been started"
  },
  "pathTitle": {
    "en": "Continue application creation"
  },
  "permissions": {
    "type": "meta-array",
    "actor": "CITIZEN",
    "metaPathToArray": ["citizen"]
  },
  "transitions": {
    "type": "single",
    "nextStage": "citizen-input"
  }
},
```

Table 4. Fields


---


BJ-SDG-SD    For internal use

<table>
    <thead>
    <tr>
        <th>Field name</th>
        <th>Description</th>
        <th>Requir
ed</th>
    </tr>
    </thead>
    <tr>
        <td>shortTitle</td>
<td>State of the application shown in application list before any

main stages are reached. Should be kept short, preferrably

one word.</td>
<td>Yes</td>
    </tr>
<tr>
        <td>title</td>
<td>State of the application shown in application timeline before

any main stages are reached.</td>
<td>Yes</td>
    </tr>
<tr>
        <td>pathTitle</td>
<td>Title of the path that continue application creation, shown in

the application timeline.</td>
<td>Yes</td>
    </tr>
<tr>
        <td>permissions</td>
<td>Permission configuration of who can interact with the

starting path of the application.</td>
<td>Yes</td>
    </tr></table>

ℹ️ The start stage has no exceptions that can be mapped.

## 7.2. UI Stage

The `ui` stage allows for the application process to take input from the user. A ui configuration is defined in a separate JSON file, which is sent to the frontend. Based on this ui configuration, a form is built and displayed to the user. When the user submits the form, the frontend sends a json object that contains the `data` and `metaData` subtrees. Every field of the subtrees on the root level is then shallow merged into the applications data tree.

For example, given the state of the application data tree:

```json
"metaData": {
  "foo":  {
    "bar": "baz",
    "old": "value"
  },
  "citizen":  ["1"]
}
```

And input from the ui stage:

```json
"metaData": {
  "foo":  {
    "old": "newValue",
    "new": "foobar"
  },
  "newRoot":  "value"
}
```

The resulting `metaData` value after the ui stage would be:



---


```json
"metaData":  {
  "foo": {
    "old": "newValue",
    "new": "foobar"
  },
  "newRoot": "value",
  "citizen": ["1"]
}
```

> The whole `foo` object from the old state was overwritten by the `foo` object from the user input, losing some old data. This needs to be kept in mind, if some existing nested fields are not returned by the ui stage, but are necessary after the ui stage, they should be kept in a separate root object.

For a detailed description of the ui configuration, see [BJ-SDG-UI].

Example UI stage configuration

```json
"citizen-input":  {
  "type": "ui",
  "uiConfiguration": "citizen-input.json",
  "transitions": {
    "type": "single",
    "nextStage": "REQUESTED"
  }
}
```

<table>
  <thead>
    <tr>
      <th>Field name</th>
      <th>Description</th>
      <th>Required</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>uiConfiguration</td>
<td>Name of the ui configuration JSON file.</td>
<td>Yes</td>
    </tr>
  </tbody>
</table>

<table>
  <thead>
    <tr>
      <th>Exception identifier</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>REQUIRED_FIELDS_MISSING</td>
<td>At least one field that the ui validation rules, matching the given state, require, was not provided.</td>
    </tr>
<tr>
      <td>FIELDS_NOT_ALLOWED</td>
<td>At least one provided field was not allowed by the ui validation rules matching the given state.</td>
    </tr>
  </tbody>
</table>

## 7.3. Payment Stage

The `pay` stage is a manual stage that requires the user to pay a certain amount, through the kkiapay service. For more information see [KKiaPay] official documentation.


---


Example

```json
"payment": {
  "type": "pay",
  "amount": "5000",
  "sandbox": true,
  "publicKey": "<public-key>",
  "privateKey": "<secret>",
  {
  "single",
```

<table>
    <thead>
    <tr>
        <th>"secret":

"transitions":

"type":

"nextStage":

}
}

Table 7. Fields</th>
        <th>"next-request"</th>
        <th></th>
    </tr>
    </thead>
    <tr>
        <td>Field name</td>
<td>Description</td>
<td>Requir
ed</td>
    </tr>
<tr>
        <td>amount</td>
<td>The amount of money required as payment. The service
fee will be added to this. Can not be negative.</td>
<td>No¹</td>
    </tr>
<tr>
        <td>dynamicAmount</td>
<td>Path in the application data tree to the field that contains
the amount to be payed (e.g. data.paymentAmount).
Can not be negative.</td>
<td>No¹</td>
    </tr>
<tr>
        <td>publicKey</td>
<td>Public key of the transaction receiver, provided by the

kkiapay service.</td>
<td>Yes</td>
    </tr>
<tr>
        <td>privateKey</td>
<td>Private key of the transaction receiver, provided by the

kkiapay service.</td>
<td>Yes</td>
    </tr>
<tr>
        <td>secret</td>
<td>Secret of the transaction receiver, provided by the kkiapay

service.</td>
<td>Yes</td>
    </tr>
<tr>
        <td>sandbox</td>
<td>When set to true, the payment is in sandbox mode,
meaning that test inputs can be used to 'perform' the
payement without actually paying any money.</td>
<td>No
(Default
: false)</td>
    </tr></table>

* ¹: Exactly one of `amount` and `dynamicAmount` is **mandatory**.

Exceptions  
Payment stage exceptions are **not mappable**. When an exception occurs (*transaction was unsuccessful, payment amount did not match, service unreachable, etc*) it’s not possible to move to the next stage until a successful payment is done.

## 7.4. UXP Integration Stage

For a detailed description of UXP integration, refer to [BJ-SDG-UXP].

The `uxp` stage performs a single SOAP request to a UXP service, using a template. The


---


template is an xml file that contains a SOAP request, with some fields containing template variables. The UXP identifier of the client who makes the request, user id and id of the request should all be specified with variables, as seen in the example.

The UXP identifier of the client is taken from the portal’s configuration and substituted into the template. For other template variables, that aren’t one of the predefined `uxp.*` variables (full list can be found in section 4 of [BJ-SDG-UXP]), are taken from the `data` subtree of the application data tree (no `data.` prefix is required).

Template Example

```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"
 xmlns:xro="http://x-road.eu/xsd/xroad.xsd" xmlns:iden="http://x-road.eu/xsd/identifiers" xmlns:wsdl="http://x-road.eu/xsd/connector/serviceName/v1">
  <soapenv:Header>
    <xro:client iden:objectType="SUBSYSTEM">
      <iden:xRoadInstance>${uxp.clientInstance}</iden:xRoadInstance>
      <iden:memberClass>${uxp.clientMemberClass}</iden:memberClass>
      <iden:memberCode>${uxp.clientMemberCode}</iden:memberCode>
      <!--Optional:-->
      <iden:subsystemCode>${uxp.clientSubsystemCode}</iden:subsystemCode>
    </xro:client>
    <xro:service iden:objectType="SERVICE">
      <iden:xRoadInstance>TARGET_INSTANCE</iden:xRoadInstance>
      <iden:memberClass>TARGET_MEMBER_CLASS</iden:memberClass>
      <iden:memberCode>TARGET_MEMBER_CODE</iden:memberCode>
      <!--Optional:-->
      <iden:subsystemCode>TARGET_SUBSYSTEM</iden:subsystemCode>
      <iden:serviceCode>serviceName</iden:serviceCode>
      <iden:serviceVersion>v1</iden:serviceVersion>
    </xro:service>
    <xro:userId>${uxp.userId}</xro:userId>
    <xro:id>${uxp.queryId}</xro:id>
    <xro:protocolVersion>4.0</xro:protocolVersion>
  </soapenv:Header>
  <soapenv:Body>
    <wsdl:serviceName>
      <VALUE>${userInputValue}</VALUE>
    </wsdl:serviceName>
  </soapenv:Body>
</soapenv:Envelope>
```


---


# Example

```json
"uxp-check": {
  "type": "uxp",
  "template": {
    "templateFile": "uxp-template.xml",
    "namespaces": {
      "soapenv": "http://schemas.xmlsoap.org/soap/envelope/",
      "wsdl": "http://x-road.eu/xsd/connector/serviceName/v1"
    },
    "responseMap": {
      "metaData.RESPONSE": {
        "xpath": "/soapenv:Envelope/soapenv:Body/wsdl:checkResponse/RESPONSE",
        "required": true
      }
    }
  },
  "transitions": {
    "type": "single",
    "nextStage": "move-forward"
  }
}
```

## Table 8. Fields

<table>
    <thead>
    <tr>
        <th>Field name</th>
        <th>Description</th>
        <th>Requir
ed</th>
    </tr>
    </thead>
    <tr>
        <td>template</td>
<td>Parameters of the UXP template file</td>
<td>Yes</td>
    </tr>
<tr>
        <td>template.template

File</td>
<td>Name of the file that holds the request template.</td>
<td>Yes</td>
    </tr>
<tr>
        <td>template.namesp

aces</td>
<td>A mapping of prefixes to namespace URIs, for simplifying

XPath expressions</td>
<td>Yes</td>
    </tr>
<tr>
        <td>template.respons

eMap</td>
<td>Map of response objects, where the key is the target path in
the application data tree.</td>
<td>Yes</td>
    </tr></table>

## Table 9. Exceptions

<table>
  <thead>
    <tr>
      <th>Exception identifier</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>CONFIG_ERROR</td>
<td>Error occured while parsing values from the service configuration or uxp request template.</td>
    </tr>
<tr>
      <td>CONNECTION_ERROR</td>
<td>Sending the UXP request or processing the response failed.</td>
    </tr>
<tr>
      <td>DATA_ERROR</td>
<td>The response is missing a required node, XPath evaluation failed or the extraction of an attachment failed.</td>
    </tr>
  </tbody>
</table>



---



<table>
<thead>
<tr>
<th>Exception identifier</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td>DATA_WARNING</td>
<td>A node was not found or multiple nodes were found where only one was expected.</td>
</tr>
</tbody>
</table>

# 7.5. UXP REST Integration Stage

The `uxp-rest` stage performs a single HTTP request to a UXP service. This service must accept requests and send its responses in JSON format.

The specific details of the request are configured in the request template. The mapping of the response to the application’s data tree is defined in the stage configuration. For a detailed explanation of these, refer to [BJ-SDG-UXP].

Example

```json
"get-post-data": {
  "type": "uxp-rest",
  "templateFile": "get-post.template.json",
  "responseMapping": {
    "data.post.body": {
      "pointer": "/body"
    },
    "data.post.title": {
      "pointer": "/title",
      "required": true
    }
  },
  "transitions": {
    "type": "single",
    "nextStage": "demo-ui"
  }
}
```

Table 10. Fields

<table>
<thead>
<tr>
<th>Field name</th>
<th>Description</th>
<th>Required</th>
</tr>
</thead>
<tbody>
<tr>
<td>templateFile</td>
<td>Path to the field that holds the request template.</td>
<td>Yes</td>
</tr>
<tr>
<td>responseMapping</td>
<td>Object of mappings, that define how data in the response is to be placed into the application’s data tree.</td>
<td>Yes</td>
</tr>
</tbody>
</table>

Table 11. Exceptions

<table>
<thead>
<tr>
<th>Exception identifier</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td>PROCESSING_ERROR</td>
<td>Something unexpected happened during the processing of the request.</td>
</tr>
</tbody>
</table>



---



<table>
<thead>
<tr>
<th>Exception identifier</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td>INVALID_CONFIGURATION</td>
<td>The request template is not valid.</td>
</tr>
<tr>
<td>MISSING_TEMPLATE_VARIABLE</td>
<td>A value for a variable in the request template cannot be found.</td>
</tr>
<tr>
<td>MISSING_REQUIRED_NODE</td>
<td>A node that is marked as required in the response mapping cannot be found in the response.</td>
</tr>
<tr>
<td>WEB_APPLICATION_EXCEPTION</td>
<td>The request returned a response with an error status code.</td>
</tr>
<tr>
<td>CONNECTION_ERROR</td>
<td>The connection failed when making the request.</td>
</tr>
</tbody>
</table>

## 7.6. Document Generation Stage

For a detailed description of the document generation process, refer to [BJ-SDG-PDF].

The `gen-document` stage can be used to generate a PDF from an html template and attach it to the application. The html template can contain variables that reference fields in the application data tree.

Example

```json
"gen-cert": {
  "type": "gen-document",
  "template": "certificate.html",
  "fileName": "certificate.pdf",
  "transitions": {
    "type": "single",
    "nextStage": "email-official"
  }
}
```

Table 12. Fields

<table>
<thead>
<tr>
<th>Field name</th>
<th>Description</th>
<th>Required</th>
</tr>
</thead>
<tbody>
<tr>
<td>template</td>
<td>Name of the HTML template file.</td>
<td>Yes</td>
</tr>
<tr>
<td>fileName</td>
<td>Name of the file that is generated</td>
<td>Yes</td>
</tr>
<tr>
<td>bindToFile</td>
<td>Attach files related files to the document (bindToFile corresponds to the file uploadId in UI configuration)</td>
<td>No</td>
</tr>
</tbody>
</table>

Table 13. Exceptions


---



<table>
<thead>
<tr>
<th>Exception identifier</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td>NO_TEMPLATE</td>
<td>No template was found for this pdf generator.</td>
</tr>
<tr>
<td>READING_FILE_CONTENT_FAILED</td>
<td>Reading the data from the stream returned by the pdf generation api failed.</td>
</tr>
<tr>
<td>PERSISTING_FILE_CONTENT_FAILED</td>
<td>Either the generated file size was larger than the allowed maximum, or something went wrong when streaming the file into the database.</td>
</tr>
<tr>
<td>API_EXCEPTION</td>
<td>The request to the pdf generation api returned an error code.</td>
</tr>
<tr>
<td>CONNECTION_ERROR</td>
<td>The request to the pdf generation api failed.</td>
</tr>
</tbody>
</table>

## 7.7. E-mail Stage

The `email` stage can be used to automatically send emails with templated title and body.  
The body and title templates allow variables that reference fields in the application data tree in the form of `${variablePath}`. For example, the variable `${metaData.citizen.0}` would be replaced with the NPI of the citizen who started the application.

Example

```json
"email-official": {
  "type": "email",
  "bodyTemplatePath": "official-template.txt",
  "targetType": "email-file",
  "targetsFile": "official-emails.txt",
  "subjectTemplate": "RCCM",
  "transitions": {
    "type": "single",
    "nextStage": "GENERATED"
  }
}
```

Table 14. Fields

<table>
<thead>
<tr>
<th>Field name</th>
<th>Description</th>
<th>Required</th>
</tr>
</thead>
<tbody>
<tr>
<td>bodyTemplatePath</td>
<td>Name of the file that contains the template of the email body</td>
<td>Yes</td>
</tr>
<tr>
<td>targetType</td>
<td>Defines which type of target field is used for getting the emails of the recipients. (<code>email-file</code>, <code>hardcoded-email</code> or <code>path-to-email</code>)</td>
<td>Yes</td>
</tr>
<tr>
<td>targetsFile</td>
<td>Name of the file that contains one email of a recipient on each line</td>
<td>If <code>targetType: email-file</code></td>
</tr>
</tbody>
</table>



---



<table>
    <thead>
    <tr>
        <th>Field name</th>
        <th>Description</th>
        <th>Required</th>
    </tr>
    </thead>
    <tr>
        <td>targets</td>
<td>JSON array of recipient emails</td>
<td>If
targetType:
hardcoded-
email</td>
    </tr>
<tr>
        <td>targetPath</td>
<td>Path in the application data tree that points to a string
that holds the recipients email</td>
<td>If
targetType:
path-to-
email</td>
    </tr>
<tr>
        <td>subjectTemplate</td>
<td>Template of the email subject. Can contain variables

just like the body template</td>
<td>Yes</td>
    </tr>
<tr>
        <td>enableUndefine

dVariableExcep

tion</td>
<td>If true, having a variable in the templates that cannot
be found in the application data tree, will result in an
exception.</td>
<td>No (Default:
true)</td>
    </tr></table>

<table>
<thead>
<tr>
<th>Exception identifier</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td>NO_TARGET_CONFIGURED</td>
<td>The target parameter for the configured <code>targetType</code> is not set in the stage configuration</td>
</tr>
<tr>
<td>FAILED_TO_GET_TARGET_NODE</td>
<td>The target node was not found in the application data tree, or the found node was not of an appropriate type (string).</td>
</tr>
<tr>
<td>NO_TEMPLATE</td>
<td>The template file could not be found.</td>
</tr>
<tr>
<td>NOTIFICATION_SERVICE_ERROR</td>
<td>The notification service API call returned an error.</td>
</tr>
</tbody>
</table>

## 7.8. Notification Stage

The `notification` stage can be used to automatically send SMS notifications to users.  
The message body template allows variables that reference fields in the application data tree in the form of `${variablePath}`. For example, the variable `${metaData.citizen.0}` would be replaced with the NPI of the citizen who started the application.

Example

```json
"citizen-notification-approved": {
  "type": "notification",
  "template": "citizen-approved.txt",
  "targetType": "path-to-npi",
  "target": "metadata.citizen.0",
  "transitions": {
    "type": "single",
    "nextStage": "APPROVED"
  }
}
```


---


# 7.8.1. Target

The target phone number that the SMS will be sent to can be evaluated in different ways.

* `targetType: hardcoded-nr`: **The value of** `target` **is a string that contains the phone number.**
* `targetType: path-to-nr`: **The value of** `target` **is a path in the application that points to a value node that holds a phone number or an array node containing phone numbers.**
* `targetType: path-to-npi`: **The value of** `target` **is a path in the application that points to a value node that holds a NPI or an array node that holds multiple NPIs. These NPIs will then be queried against the user (citizen) database to find the phone numbers of the given users.**

> For the `path-to-nr` and `hardcoded-nr` scenarios, the default country code of +229 is used.

<table>
<thead>
<tr>
<th>Field name</th>
<th>Description</th>
<th>Required</th>
</tr>
</thead>
<tbody>
<tr>
<td>targetType</td>
<td>Defines how the value of the <code>target</code> parameter will be interpreted. Possible values: <code>path-to-nr</code>, <code>path-to-npi</code> and <code>hardcoded-nr</code></td>
<td>Yes</td>
</tr>
<tr>
<td>target</td>
<td>Defines the target phone number, the SMS will be sent to.</td>
<td>Yes</td>
</tr>
</tbody>
</table>

<table>
<thead>
<tr>
<th>Exception identifier</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td>FAILED_TO_GET_TARGET_NODE</td>
<td>The target number is supposed to come from the application data tree, but the field is not found.</td>
</tr>
<tr>
<td>INVALID_PHONE_NUMBER</td>
<td>The found phone number is not numerical.</td>
</tr>
<tr>
<td>NO_VALID_TARGETS</td>
<td>No valid targets are found from the application data tree.</td>
</tr>
<tr>
<td>NO_TEMPLATE</td>
<td>The template for the notification is not found.</td>
</tr>
<tr>
<td>NOTIFICATION_SERVICE_ERROR</td>
<td>An exception occurred during sending the notification to the notification service.</td>
</tr>
<tr>
<td>INVALID_CONFIG</td>
<td>The target type in the service config is not supported.</td>
</tr>
</tbody>
</table>



---


# 7.9. Hardcoded Data Stage

The `hardcoded-data` stage can be used to insert data into the application data tree. This can be used for example, to put up flags, when the application has passed through a certain branch.

Example

```json
"data": {
  "type": "hardcoded-data",
  "writeData": {
    "metaData": {
    },
    "data": {
      "paymentDone": "true"
    }
  },
  "transitions": {
    "type": "single",
    "nextStage": "loop-to-start"
  }
}
```

<table>
  <thead>
    <tr>
      <th>Field name</th>
      <th>Description</th>
      <th>Required</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>writeData</td>
<td>JSON object that contains the <code>metaData</code> and <code>data</code> objects that contain fields to be added to the application data tree. Merging is analogous to <a href="#">UI Stage</a> input merging.</td>
<td>Yes</td>
    </tr>
  </tbody>
</table>

# 7.10. Blacklist Stage

The `blacklist` stage can be used to remove certain fields from the application data tree.

For example, when a previous path adds a comment to the `metaData` and the current path doesn’t want to overwrite it, it can be removed with the stage in the example. This would prevent it from showing up on the current path as well, once it is complete.

Example

```json
"blacklist-comment": {
  "type": "blacklist",
  "blacklist": [
    "metaData._comment"
  ],
  "transitions": {
    "type": "single",
    "nextStage": "email-official-changes"
  }
},
```


---



<table>
<thead>
<tr>
<th>Field name</th>
<th>Description</th>
<th>Required</th>
</tr>
</thead>
<tbody>
<tr>
<td>blacklist</td>
<td>Array of strings that are paths in the application data tree of fields that are to be removed.</td>
<td>Yes</td>
</tr>
</tbody>
</table>

# 7.11. Person Data Stage

The `person-data` stage can be used to take data about a user (citizen) from the database and place it into the application data tree so other stages can access it.

Example

```json
"person-data": {
  "type": "person-data",
  "npiSource": "metaData.citizen.0",
  "skipIfNoNpi": false,
  "dataPoints": {
    "email": "data.person_ui.email",
    "phone_country_code": "data.person.phone_code",
    "phone_number": "data.person.phone_number",
    "first_name": "data.person.first_name",
    "last_name": "data.person.last_name",
    "birth_date": "data.person.birth_date"
  },
  "transitions": {
    "type": "single",
    "nextStage": "choose-type"
  }
}
```

<table>
<thead>
<tr>
<th>Field name</th>
<th>Description</th>
<th>Required</th>
</tr>
</thead>
<tbody>
<tr>
<td>npiSource</td>
<td>Path in the application data tree that points to a field that holds the NPI of the target user.</td>
<td>Yes</td>
</tr>
<tr>
<td>dataPoints</td>
<td>Map of data fields that can be queried from the database, with the values defining the field where the values should be placed in the application data map.</td>
<td>Yes</td>
</tr>
<tr>
<td>required</td>
<td>Takes effect when the NPI cannot be found from the data tree. If true, the CANNOT_GET_NPI exception is returned. If false, the stage will return successfully, without doing anything.</td>
<td>No (Default: true)</td>
</tr>
</tbody>
</table>

Table 21. Available data fields



---


BJ-SDG-SD    For internal use

<table>
    <tr>
        <td>Field name</td>
<td>Description</td>
    </tr>
<tr>
        <td>first_name</td>
<td>First name of the citizen.</td>
    </tr>
<tr>
        <td>last_name</td>
<td>Last name of the citizen.</td>
    </tr>
<tr>
        <td>mother_npi</td>
<td>NPI of the mother of the citizen.</td>
    </tr>
<tr>
        <td>father_npi</td>
<td>NPI of the father of the citizen.</td>
    </tr>
<tr>
        <td>marital_name</td>
<td>Marital name of the citizen.</td>
    </tr>
<tr>
        <td>phone_country_cod

e</td>
<td>Country code of the citizen’s phone number.</td>
    </tr>
<tr>
        <td>phone_number</td>
<td>Phone number of the citizen.</td>
    </tr>
<tr>
        <td>email</td>
<td>Email of the citizen.</td>
    </tr>
<tr>
        <td>nationality</td>
<td>Nationality of the citizen.</td>
    </tr>
<tr>
        <td>sex</td>
<td>Gender of the citizen.</td>
    </tr>
<tr>
        <td>birth_date</td>
<td>Date of birth of the citizen.</td>
    </tr>
<tr>
        <td>birth_country_code</td>
<td>Country code of the citizen’s birth country.</td>
    </tr>
<tr>
        <td>birth_department</td>
<td>Citizen’s birth department</td>
    </tr>
<tr>
        <td>birth_town</td>
<td>Citizen’s birth town.</td>
    </tr>
<tr>
        <td>birth_district</td>
<td>Citizen’s birth district.</td>
    </tr>
<tr>
        <td>birth_village</td>
<td>Citizen’s birth village.</td>
    </tr>
<tr>
        <td>birth_place</td>
<td>Citizen’s birth place.</td>
    </tr>
<tr>
        <td>residence_departme

nt</td>
<td>Citizen’s residence department.</td>
    </tr>
<tr>
        <td>residence_town</td>
<td>Citizen’s residence town.</td>
    </tr>
<tr>
        <td>residence_district</td>
<td>Citizen’s residence district.</td>
    </tr>
<tr>
        <td>residence_village</td>
<td>Citizen’s residence village.</td>
    </tr>
<tr>
        <td>residence_address</td>
<td>Citizen’s residence address.</td>
    </tr></table>

Table 22. Exceptions


---



<table>
<thead>
<tr>
<th>Exception identifier</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td>CANNOT_GET_NPI</td>
<td>The stage couldn’t find a NPI from the given path.</td>
</tr>
<tr>
<td>CANNOT_FIND_PERSON</td>
<td>A user with the given NPI couldn’t be found from the database.</td>
</tr>
</tbody>
</table>

## 7.12. Alert Stage

The `alert` stage can be used to add a custom alert to the application.

Example

```json
"invalid-response": {
  "type": "alert",
  "alertType": "ERROR",
  "message": {
    "en": "The service returned a negative response."
  },
  "transitions": {
    "type": "single",
    "nextStage": "citizen-input"
  }
}
```

Table 23. Fields

<table>
<thead>
<tr>
<th>Field name</th>
<th>Description</th>
<th>Required</th>
</tr>
</thead>
<tbody>
<tr>
<td>alertType</td>
<td>Type of the alert to create (<code>ERROR</code>, <code>WARNING</code> or <code>SUCCESS</code>).</td>
<td>Yes</td>
</tr>
<tr>
<td>message</td>
<td>Language map of the message text that is shown to the user.</td>
<td>Yes</td>
</tr>
<tr>
<td>clearMessages</td>
<td>If set to true, will dismiss all messages (<em>errors/alerts</em>) that have previously accumulated to the application.</td>
<td>No (Default: <code>true</code>)</td>
</tr>
</tbody>
</table>

## 7.13. Share Files Stage

The `share-files` stage can be used to generate a unique link, that can be used to download a zip-package of all files linked to the application.

> The list of files bound to the application is evaluated during the download, so files added after the `share-files` stage are placed into the package as well.



---


The link generated requires no authentication and has no expiration time.  
The length of the unique code generated for the link can be configured in the Shared-Files configuration. This stage should only be used, if no other means of sharing files is not feasible.

Example

```json
"share-files": {
  "type": "share-files",
  "codePath": "data.files.code",
  "baseUrlPath": "data.files.baseUrl",
  "fileNamePath": "data.files.name",
  "zipFileName": "Application - ${data.person.first_name} ${data.person.last_name}",
  "transitions": {
    "type": "single",
    "nextStage": "email-official"
  }
}
```

With the given configuration, the file package would be available from:  
`${data.files.baseUrl}/api/portal/shared/files/${data.files.code}`

<table>
    <thead>
    <tr>
        <th>Table 24. Fields</th>
        <th></th>
        <th></th>
    </tr>
    </thead>
    <tr>
        <td>Field name</td>
<td>Description</td>
<td>Requir
ed</td>
    </tr>
<tr>
        <td>codePath</td>
<td>The path in the application data tree, where the generated
access code of the file package is placed.</td>
<td>Yes</td>
    </tr>
<tr>
        <td>baseUrlPath</td>
<td>The path in the application data tree, where the base URL
of the portal is placed. The base URL is taken from the
Shared-Files configuration.</td>
<td>Yes</td>
    </tr>
<tr>
        <td>fileNamePath</td>
<td>The path in the application data tree, where the generated
file name is placed.</td>
<td>Yes</td>
    </tr>
<tr>
        <td>zipFileName</td>
<td>The template for the file name of the packaged zip file of all
the files.</td>
<td>Yes</td>
    </tr></table>

<table>
  <thead>
    <tr>
      <th>Exception identifier</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>CANT_GENERATE_UNIQUE_CODE</td>
<td>The maximum number of attempts for generating a unique code for the link was exceeded.</td>
    </tr>
  </tbody>
</table>



---


# 8. Transitions

Transitions define the way the applications of a service move between stages.

## 8.1. Main Transition

The transitions of main stages define **paths**, that exit the main stage. Each one of these paths define their own permissions, of which users are able to start the path.

> ⚠️ Only a single path is currently supported.

Example (with irrelevant parts removed)

```json
"REQUESTED": {
  "type": "main",
  "transitions": [
    {
      "id": "review",
      "nextStage": "official-review",
      "title": {
        "en": "Review the application"
      },
      "resultMainStages": [
        "APPROVED",
        "REJECTED",
        "CHANGES-REQUESTED"
      ],
      "permissions": {
        "type": "hardcoded-array",
        "actor": "OFFICIAL",
        "array": [
          "Official"
        ]
      }
    }
  ]
}
```

Table 26. Fields

<table>
  <thead>
    <tr>
      <th>Field name</th>
      <th>Description</th>
      <th>Required</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>id</td>
<td>Identifier of the path. Has to be unique across the whole service configuration.</td>
<td>Yes</td>
    </tr>
<tr>
      <td>nextStage</td>
<td>The first stage that the application moves to, when starting this path</td>
<td>Yes</td>
    </tr>
<tr>
      <td>title</td>
<td>The language map of the path title. This text is shown on the timeline as the action.</td>
<td>Yes</td>
    </tr>
  </tbody>
</table>



---


BJ-SDG-SD    For internal use

<table>
    <thead>
    <tr>
        <th>Field name</th>
        <th>Description</th>
        <th>Requir
ed</th>
    </tr>
    </thead>
    <tr>
        <td>resultMainStages</td>
<td>List of all main stages, that this path could possibly lead to.

Determines who can see the application. This must be

correct, for the application visibility permissions to work.</td>
<td>Yes</td>
    </tr>
<tr>
        <td>permissions</td>
<td>Configuration of who can access the path. See

Permissions.</td>
<td>Yes</td>
    </tr>
<tr>
        <td>automatic</td>
<td>If set to true, the transition is automatic.</td>
<td>No
(Default
:
false)</td>
    </tr>
<tr>
        <td>automaticPortalId</td>
<td>Identifier of the portal in which the automatic transition

should run.</td>
<td>If
autom
atic:
true</td>
    </tr></table>

## 8.1.1. Automatic Transitions

Up to one transition on a main stage can be marked as automatic. If the application reaches a main stage and sees an automatic transition in the main stage, it checks its portal id against the configured portal id for the automatic transition. If the portal ids match, the portal immediately starts the new automatic path.

All intermediate stages in this path should be automatic, to allow fully automatic completion, without input from the user.

> ⚠️ This feature is useful once the engine supports multiple stand-alone instances of the portal with the application store connecting them.

Example of an automatic transition (with irrelevant parts removed)

```json
"REQUESTED": {
  "type": "main",
  "transitions": [
    {
      "id": "process",
      "nextStage": "info-request",
      "title": { },
      "resultMainStages": ["GENERATED"],
      "permissions": {
        "type": "hardcoded-array",
        "actor": "OFFICIAL",
        "array": ["Official-Group-1"]
      },
      "automatic": true,
      "automaticPortalId": "official-portal"
    }
  ]
}
```


---


# 8.2. Intermediate stage transitions

Each intermediate stage must have a transition configured. This transition is run after the stage has finished processing.

## 8.2.1. Single Transition

The `single` transition is a basic transition that moves the application to the specified stage.

Example

```json
"transitions": {
  "type": "single",
  "nextStage": "gen-extract"
}
```

Table 27. Fields

<table>
  <thead>
    <tr>
      <th>Field name</th>
      <th>Description</th>
      <th>Required</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>nextStage</td>
<td>The stage the application must move to</td>
<td>Yes</td>
    </tr>
  </tbody>
</table>

## 8.2.2. Map by Meta Transition

The `map-by-meta` transition can be used to branch the application flow based on the value of a field in the application data tree.

> The first matching transition is chosen.

Example

```json
"transitions": {
  "type": "map-by-meta",
  "metaPathToKey": ["submit"],
  "map": {
    "proceed": "payment",
    "search": "search-request"
  }
}
```

Table 28. Fields

<table>
  <thead>
    <tr>
      <th>Field name</th>
      <th>Description</th>
      <th>Required</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>metaPathToKey</td>
<td>Array of strings that form a path in the <code>metaData</code> subtree of the application data tree to the field to be checked.</td>
<td>Yes</td>
    </tr>
  </tbody>
</table>



---



<table>
  <thead>
    <tr>
      <th>Field name</th>
      <th>Description</th>
      <th>Required</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>map</td>
<td>Key-value map, where the key is a value of the variable in metaData and the value is the next stage (<i>if the variable has the given value</i>). Key * can be used as a wildcard if no matches were found.</td>
<td>Yes</td>
    </tr>
  </tbody>
</table>

## 8.2.3. Exception handling

Intermediate stages can complete their processing by returning an exception. If the behavior of the transition, when encountering the returned exception is not defined, the transition runs as usual, and the application flow continues. To define behavior on exceptions, the `onStageException` map can be used on all intermediate stage transitions.

Example of exception handling

```json
"transitions": {
  "type": "map-by-meta",
  "metaPathToKey": ["submit"],
  "map": {
    "proceed": "payment",
    "search": "search-request"
  },
  "onStageException": {
    "REQUIRED_FIELDS_MISSING": "choose-company",
    "FIELDS_NOT_ALLOWED": "choose-company"
  }
}
```

If the returned exception is defined as a key in the map, the value corresponding to that key is evaluated as the next stage, and the transitions implementation is not executed.


---


# 9. Permissions

Permission configurations define which users can access specific paths. Each path will have a single permission configuration throughout its whole operation. The permission configurations are placed in Main Transition and Start Stage blocks.

> Permissions can only be set to paths, which means intermediate stages within a path all have the same permissions.

The permission configurations typically define an actor class, which the permissions target. Depending on this class, the values handled by the permissions have different meaning.

* **OFFICIAL** - For official types, the values handled by the permissions are official AD groups. If the official has at least one group that matches the permission configuration, they have access to the application.
* **CITIZEN** - For citizen types, the value handled by the permissions are the citizen’s NPIs.

## 9.1. Hardcoded Array Permissions

The `hardcoded-array` permissions configuration can be used to hardcode a list of users that can access the path into the service configuration.

Example

```json
"permissions": {
     "type": "hardcoded-array",
     "actor": "OFFICIAL",
     "array": ["Official"]
}
```

Table 29. Fields

<table>
<thead>
<tr>
<th>Field name</th>
<th>Description</th>
<th>Required</th>
</tr>
</thead>
<tbody>
<tr>
<td>actor</td>
<td>Type of the actor that is given permissions by this configuration (<code>OFFICIAL</code> or <code>CITIZEN</code>).</td>
<td>Yes</td>
</tr>
<tr>
<td>array</td>
<td>List of hardcoded values that are given permission.</td>
<td>Yes</td>
</tr>
</tbody>
</table>

## 9.2. Meta Array Permissions

The `meta-array` permissions configuration can be used to grant access based on data from the application data tree. The permission can only access the `metaData` subtree since the path must be possible to evaluate without the presence of the `data` subtree.

Since the application creation places the NPI of the citizen into the `citizen array`, the following configuration can be used to allow the citizen owning the application access to the path.



---


Example

```json
"permissions": {
  "type": "meta-array",
  "actor": "CITIZEN",
  "metaPathToArray": ["citizen"]
}
```

<table>
  <thead>
    <tr>
      <th>Field name</th>
      <th>Description</th>
      <th>Required</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>actor</td>
<td>Type of the actor that is given permissions by this configuration (<code>OFFICIAL</code> or <code>CITIZEN</code>).</td>
<td>Yes</td>
    </tr>
<tr>
      <td>metaPathToArray</td>
<td>Array of path segments relative to the <code>metaData</code> subtree, that points to the array containing the list of users.</td>
<td>Yes</td>
    </tr>
  </tbody>
</table>

### 9.3. Public Services

Services can be made public, which means that an application for a service can be started by a user that is not authenticated. In this case, the frontend will generate a session token, which will be linked to the application. Requests made with that session token will then be able to see the application timeline and perform operations on paths which are marked as public.

To allow starting applications without authentication, the optional `public` field at the root of the service configuration has to be set to `true`. To set a path as public, the optional `public` field on the permissions object (available on all permissions types) has to be set to `true`.

> ⚠️ For public services, the start stage’s permissions should always be set as public.

Example of a public path

```json
"permissions": {
  "type": "meta-array",
  "actor": "CITIZEN",
  "metaPathToArray": ["citizen"],
  "public": true
}
```

If a path is set as public and the application was created by an anonymous user, the permissions type implementation is ignored and the session token provided in the request is compared against the session token that was used to create the application.

> ⚠️ If the user’s browser loses the session token or the application id, there is no reasonable way to access the application again. Therefore this functionality should be used only if it’s necessary and on simple services.


---


# 10. Appendix

Full source of the example service configuration.

```json
{
  "serviceId": "dev-guide",
  "serviceVersion": "1.0",
  "stages": {
    "start": {
      "type": "start",
      "shortTitle": {
        "en": "STARTED",
        "fr": "DÉBUT"
      },
      "title": {
        "en": "Application started",
        "fr": "Demande soumise"
      },
      "pathTitle": {
        "en": "Continue application creation",
        "fr": "Poursuivre la demande"
      },
      "permissions": {
        "type": "meta-array",
        "actor": "CITIZEN",
        "metaPathToArray": ["citizen"]
      },
      "transitions": {
        "type": "single",
        "nextStage": "citizen-input"
      }
    },

    "citizen-input": {
      "type": "ui",
      "uiConfiguration": "citizen-input.json",
      "transitions": {
        "type": "single",
        "nextStage": "REQUESTED"
      }
    },

    "REQUESTED": {
      "type": "main",
      "shortTitle": {
        "en": "REQUESTED",
        "fr": "DEMANDÉ"
      },
      "title": {
        "en": "Certificate has been requested.",
        "fr": "Le certificat a été demandé."
      },
      "transitions": [
        {
          "id": "review",
          "title": {
```


---


```json
{
  "en": "Review the application",
  "fr": "Examiner la demande"
},
"nextStage": "official-review",
"resultMainStages": ["APPROVED", "REJECTED"],
"permissions": {
  "type": "hardcoded-array",
  "actor": "OFFICIAL",
  "array": ["Official-Group-1", "Official-Group-2"]
}
},
"official-review": {
  "type": "ui",
  "uiConfiguration": "official-review.json",
  "transitions": {
    "type": "map-by-meta",
    "metaPathToKey": ["official", "choice"],
    "map": {
      "approve": "APPROVED",
      "reject": "REJECTED"
    },
    "onStageException": {
      "REQUIRED_FIELDS_MISSING": "search-terms",
      "FIELDS_NOT_ALLOWED": "search-terms"
    }
  }
},
"APPROVED": {
  "type": "main",
  "shortTitle": {
    "en": "APPROVED",
    "fr": "VALIDÉ"
  },
  "title": {
    "en": "The request for a certificate has been approved"
  },
  "flags": ["final", "accept"],
  "transitions": []
},
"REJECTED": {
  "type": "main",
  "shortTitle": {
    "en": "Rejected",
    "fr": "REJETÉ"
  },
  "title": {
    "en": "The request for a certificate has been rejected"
  },
  "flags": ["final", "reject"],
  "transitions": []
}
}
}
```


---

BJ-SDG-SD    For internal use

Beninese Citizen Portal: Service Development    0.3

12.07.2020                                      39 / 39