# HSE Aurora USM Integration with AIE Test Plan

| Author: | Status: (WIP \| In Review \| Approved \| Obsolete) |
| :--- | :--- |
| Engineering Manager: Rodolfo Pirotti | Created: |
| DRC Sponsor:  Milestone/Initiative: |

## Changelog (if we use git, the git change history could replace this section)

| Date:  | Author: | Change: |
| :--- | :--- | :--- |
| 2022-xx-xx | \<name\> | description |
| 2022-xx-xx | \<name\> | description |

## Introduction

The HPE Trusted OS Security – User Space Monitor (HPETOS-USM) is a loadable kernel module, responsible for monitoring user-space applications looking for potential injections and attacks. Upon detecting possible attacks on a process, alerts are sent to all registered listeners. In this case, a second component in this process is the HSS-USMD, which is a user-space application responsible for notifying all services who have been registered to be informed when such alerts are detected by the kernel module.  

Events and metrics generated by USM components must be sent to Aurora so they can be handled. For this, it is necessary to integrate the USM with Aurora, which is the purpose of this feature. In general, this integration consists of the PAPA creating a subscription in the USMD so that the Device-Events can receive the events sent by the USMD, and in addition, it consists of the Metric-System being able to receive the metrics generated by the USM components or by some other point that makes up this integration. As pointed out at a high level by the following points:

* USMD and HPETOS need to be installed;

* The PAPA must create the USMD subscription using SPIRE certificates. If the creation fails, an alert must be generated to the metrics system;

* Device-Events should receive alerts from USMD;

* Metric-System should receive metrics from USM components.

The purpose of this document is to plan the tests that will be necessary to validate that the modifications or implementations of new components for this integration are working as expected.

Any change on components that compose the integration between USM and AIE may impact the creation, modification, or deprecation of test cases.

## Test Approach

The testing for this feature will be done via automated tests that will run nightly in an automated test environment. These test cases will be created with the Agile methodology. For the most part, tests will be created so that they can run (1) directly on a developer’s test environment and as part of the automated test pipeline on a nightly basis.  

## Test Plan Checklist

* [ ] UI Testing
* [ ] Negative Testing
* [ ] Usability testing
* [ ] Component testing
* [ ] Integration testing
* [ ] API & Contract testing
* [ ] HA/DR testing
* [ ] Scale & Performance
* [ ] Security testing
* [ ] Soak and Longevity testing

## Risks and Assumptions

## Scope

### Testing Scope (IS)

This feature consists of the integration of USM with AIE, so that events and metrics monitored in the user space are received by Aurora. To ensure that this integration is working as expected, it is necessary to validate different aspects of the feature, as shown below:

* **Deploy**
  * The create_install_bundle.sh script should be able to download the RPM files and install correctly the USMD (daemon), and HPETOS-USM (kernel module);
  * Both components should be up and running;
  * USMD should be able to work with SPIRE certificate.

* **Subscription**
  * The hss-platform-agent should be able to communicate and create the subscription on USMD. This communication will be mTLS using SPIRE-Based certificates;
  * The information passed when creating the subscription must be stored in the database.  

* **Events**
  * The hss-device-events must receive the alerts sent by the USMD according to the type of attack made. This must be true for all existing events;
  * Received events must be parsed and saved correctly in the database.

* **Metrics**
  * The hss-metric-system must receive metrics correctly according to the reproduced failures;
  * If the subscription process fails, the PAPA should send a metric to server and the hss-metric-system should receive and parse it correctly;
  * Depending on the design on VF-5642, if it is decided to turn Metric System into an AP, it is necessary to validate if the operations (GET and POST, as example), of the endpoint are working as expected;
  * Depending on the design on VF-5642, the hss-metric-system should receive correctly the HPETOS-USM metrics dmesg, parse and store in the database.

> Note: Feature Testing Scope planning depends on some designs yet to be done, for this reason some points can be added or removed.  

### Feature Testing Scope (IS-NOT)

As previously discussed, the scope of this feature is the integration of USM with AIE. For this reason, what is included in coverage will be the tests related to installing USMD and its dependencies toghether Aurora components and validating its correct functioning with Aurora components like PAPA, Device-Events, Event-System, Metric-System components. Tests to validate USM itself, which are not related to USM and AIE integration, will NOT be covered under the scope of this test plan.

Regarding (1), if a test case has limitations for which it cannot be run on a virtual machine, this will be documented.

## prerequisite

HPETOS-USM and USMD should be deployed and working as expected before testing. Specific requirements have been documented in each task.

## Test Cases/Tasks

### USM Install

```Robot
*** Test Cases ***

Check if USMD is installed by HSS Agents deployment and it is up and running with correct configurations as expected
  [Documentation] for PyTest
  ...
  ...    == Test Case Name  ==
  ...     USM Rpm validation 
  ...
  ...    == Description ==
  ...     As part of HSS Agents Deployment USM RPMs (hpetos-usm and hss-usmd) are also installed.
  ...     So, at the end of installation process, HPETOS-USM kernel module must be correctly
  ...     loadeded and USM daemon (USMD) must be active and up and running.
  ...
  ...    ==  Workflow ==
  ...     Aurora USM Workflow
  ...
  ...    == Automation ==
  ...     Automated 
  ...
  ...    == Precondition ==
  ...     HSS Agents Deployment should be already performed with USM RPMs included on bundle (hpetos-usm and hss-usmd)
  ...
  ...    == Test Steps ==
  ...    Checking if configuration files for both HPETOS-USM and USMD are set as expected. Also check if HPETOS-USM module
  ...    is loaded on kernel and USMD is up and running.
  ...
  ...     As steps we have:
  ...
  ...     1. Checks that the 'USMD' JSON configuration file is configured as expected
  ...     2. Checks that the 'HPETOS-USM' configuration file is configured as expected
  ...     3. Checks that the 'HPETOS-USM' module is loaded on kernel
  ...
  ...    == Expected Result ==
  ... 
  ...     4. Checks that the 'USMD' service is up and running
  ...
  ...    == Type ==
  ...    Type: Functional | Load | Performance
  ...
  ...    == Jira Task ==
  ...     VF-5624 (test related to the code implemented on VF-5623)
```

## Secutity

The current security scans must run successfully with out regressions.

## Shared Library and Test Automation Suggestions

* JSON Reading Structure;

* Authenticate an API request via mTLS using trust bundle, SVID and key certs;

* Method that checks if the Platform agent service is running;

* Method that restarts the Platform Agent.

## References

### Presentation

* [HPETOS User Space Monitor - Deep Dive](https://hpe.sharepoint.com/:p:/s/msteams_d72ff1/EZIYPCLfJPdDrclLwwtoboQB9YhmMRpS9gHU1dKiA5IlnQ?e=aHllOK)
* [HPETOS User Space Monitor – Deep Dive – Video](https://hpe-my.sharepoint.com/personal/gustavo_knuppe_hpe_com/_layouts/15/onedrive.aspx?ga=1&id=%2Fpersonal%2Fgustavo%5Fknuppe%5Fhpe%5Fcom%2FDocuments%2FRecordings%2FAurora%20Architecture%20Deep%20Dive%2D20220818%5F143416%2DMeeting%20Recording%2Emp4&parent=%2Fpersonal%2Fgustavo%5Fknuppe%5Fhpe%5Fcom%2FDocuments%2FRecordings)
  
### Wiki

* [Dime Wiki USMD](https://github.hpe.com/hpe/dime-wiki/tree/master/design/usmd)

### Feature Design

* [USM Integration with AIE](https://hpe.sharepoint.com/:w:/r/teams/brsec/_layouts/15/Doc.aspx?sourcedoc=%7B2394E0E9-BEF1-4414-B4A9-3F4BFFB80669%7D&file=USM%20Integration%20with%20AIE.docx&action=default&mobileredirect=true)
* [HSS-USMD TLS and client authorization](https://hpe.sharepoint.com/sites/msteams_d72ff1/_layouts/15/Doc.aspx?sourcedoc=%7B2BEC353F-A140-4AB4-A337-5A250C7EA3B6%7D&file=HSS-USMD%20spire%20client.docx&action=default&mobileredirect=true)

### Paper

* [Monitoring the integrity of critical applications and services](https://hpe.sharepoint.com/sites/F5/CTO/labs/library/Tech%20Con%202022/Forms/AllItems.aspxid=%2Fsites%2FF5%2FCTO%2Flabs%2Flibrary%2FTech%20Con%202022%2FAbstracts%20submitted%20for%202022%2F51%2Epdf&parent=%2Fsites%2FF5%2FCTO%2Flabs%2Flibrary%2FTech%20Con%202022%2FAbstracts%20submitted%20for%202022)

## Glossary

* **AIE** – Aurora Integrity Engine

* **USM** – User Space Monitor

* **USMD** – User Space Monitor Daemon

* **HPETOS-USM** – It is a kernel module that will monitor user space processes and send the metrics to USMD

* **PAPA** – Project Aurora Platform Agent

* **SVID** – SPIFFE Verifiable Identity Document, the document with which a workload proves its identity to a resource or caller. An SVID is considered valid if it has been signed by an authority within the SPIFFE ID’s trust domain.

* **SPIRE** – Is a production-ready implementation of the SPIFFE APIs that perform node and workload attestation in order to securely issue SVIDs to workloads, and verify the SVIDs of other workloads, based on a predefined set of conditions.

* **SPIFFE** – The Secure Production Identity Framework for Everyone, is a set of open-source standards for securely identifying software systems in dynamic and heterogeneous environments

* **SPIFFE ID** – Is a string that uniquely and specifically identifies a workload

* **mTLS** – Mutual Transport Layer Security, is a mutual method of authentication that Aurora uses to communicate

* **Device-Events** – It is the component that receives events that were generated externally to Aurora, for example by iLO and now by USMD  

* **Event-System** – It is the component that parses incoming events and stores them in the database

* **Metric-System** – It is the component that parses the received metrics and stores them in the database

## Checklist

* [ ] Item 1
* [x] Item 2

## Reviews/Approvals

| Role:  | Reviewer | Approved | Rejected | Notes: |
| :--- | :--- | :--- | :--- | :--- |
| Program: | \<assigned pm\> | X | | |
| Architect: | \<assigned architect \> | | X | |
| Test Lead:| \<assigned test lead \> | X | | |