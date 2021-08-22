# Installation and Setup

### Installation Steps

* Install **the latest available Exago version**, that includes the server and client Exago applications. 
  * Download the Exago package or docker image from the K2View download page.
* Install **Storage Management DB**, the database that keeps full report definition such as report type and metadata, currency, decimal setting, colors and more. [Click for more information about the Exago Storage Management](https://support.exagoinc.com/hc/en-us/articles/360042587313-Storage-Management-Introduction).
* (Optional) Install **Storage Management Utility**, the UI tool that allows setting up the folders' access permissions. [Click for more information about the Exago Storage Management Utility](https://support.exagoinc.com/hc/en-us/articles/360053801773-Storage-Management-Utility-v2021-1-).
* Update **Fabric config.ini** with Exago host and Storage Management details - TO ADD THE PARAMETER NAMES

*Link to the K2View DevOps document explaining how to install the Exago components - TBD.*

### Installation Recommendations

Following are the installation and setup recommendations:

- It is recommended to have 3 separate Exago installations: for Dev, QA and Production.
  - Linux installation is a must for UAT / Production environments.
  - Installation on Windows (as a docker) can be done for development or demo purposes.
- The recommended Storage Management DB type is PostgreSQL.
  - PostgreSQL is a must for UAT / Production environments, but it is preferable to use it for Dev and QA as well.
  - Default SQLite DB can be used for demo purposes. 
- To re-use the same Storage Management DB for several environments of the same type (for example, several QA environments), define a unique Table Prefix as part of the Storage Management DB details.

 
