- [Introduction](#introduction)
- [Required-Code-Changes](#required-code-changes)
# Introduction
This document will detail about the aproach we take in phase1 to implement the basic SRE functionalities.
In Phase1 , we wanted to leverage the existing grafana functionalities and write the custom grafana functionalities to implement our features. In phase2 , we will gradually refactor the codebase and will write more independent modules.


please refer to the diagrams below for phase1 and phase2 architecture:
### phase1- architecture

![phase1-architecture](https://raw.githubusercontent.com/AppkubeCloud/appkube-architectures/main/LayeredArchitecture-phase1.svg)

### phase2- architecture

![phase2-architecture](https://raw.githubusercontent.com/AppkubeCloud/appkube-architectures/main/LayeredArchitecture-phase2.svg)


# Phase1 Aproach

## Terminology --
---

### **SUI** - Our Stand Alone react based UI
---

The UI will mostly call cmdb api and will show different clouds and their element details:

Please refer the design details of AWS explorer as below:

[AWSExplorer](https://www.figma.com/proto/tmzdMgCegtVSQLVgHR6uc3/Netlifi-Usecase-file?page-id=0%3A1&node-id=37%3A16358&viewport=184%2C-681%2C0.04&scaling=scale-down&starting-point-node-id=37%3A16358&show-proto-sidebar=1)  - Shows Cloud ELement Details 

Whenever SUI  will show any specific App and Data Service Details, it will call the service explorer from remote grafana and render in iframe.

---

### **AWS Cloud Element Explorer** -- 

The purpose of this service is to render the explorer for every App and Data Services in cloud. This service is a remote grafana that has a inbuilt ***Cloud Datasource*** . This service will have explorer for all cloud elements (WAF / APIGw / CDN / S3 / Route53 / VPC -> EKS/ ECS / EC2 / RDS / Dynamo /.. ).

This remote grafana instance will either keep grafana Apps or Appkube Views for cloud elements(to be decided)

For any App or Data services , SUI will call the remote grafana url with parameters and display the results in iframe in phase1.

Phase2 , SUI will do only api calls and render directly in UI via proxy grafana API's server.
---

### **Cluster Explorer** -- This will  show  details of all the individual Cluster elements 
(products / services). 

![alt](./images/CloudElements/cluster-explorer.jpg)

---


### **Service Explorer** -- This will  show  details of all the individual cloud elements 
(WAF / APIGw / CDN / S3 / Route53 / VPC -> EKS/ ECS / EC2 / RDS / Dynamo /.. ). 

![alt](./images/CloudElements/ServiceExplorer.jpg)


![alt](./images/CloudElements/cloud-element4.jpg)

Theere would be two aproaches to display the services belong to cluster or cloud managed.

---


### **AWS-API-Server** -- For collecting all the elements data , AWS Cloud Explorer will call the api server.
---

### **Appkube-Catalogue** --  This service will have all the published dashboards / tools etc 

---

### **Appkube-cmdb** -- This service  will have the App / Data services details along with their topology details
---

# Process Flow


SUI will call the cmdb api's to show every cloud accounts details. The information will be shown as below:

![alt](./images/home.jpg)

CMDB will have API for all the Data.

When a user clicks on any AWS account id , SUI will call the CMDB provided Api's and will draw the topology of every product enclaves,i.e  the elements(clusters/ firewall / Load Balancers / Nodes / Databases / Other services)  inside the product enclaves as follows:

Screens for any accountId navigation :

![alt](./images/CloudElements/cloud-element1.jpg)

![alt](./images/CloudElements/cloud-element2.jpg)

![alt](./images/CloudElements/cloud-element3.jpg)

SUI will call the API's as follows:

https://cmdb.synectiks.net/getAllCloudElements/? LandingZone=3534545454 

https://cmdb.synectiks.net/getProductEnclaves/? LandingZone=3534545454

https://cmdb.synectiks.net/getCloudElementsInProductEnclave/? LandingZone=3534545454 && ProductEnclave= 435454

https://cmdb.synectiks.net/getClusterElements/? LandingZone=3534545454 && ProductEnclave= 435454 && ClusterId=657667

## Cluster Explorer Navigation

***what to do for the clusters inside product enclaves ??***


![alt](./images/CloudElements/cluster-explorer.jpg)


When a user will navigate till any cluster inside the product enclave,and click on them , it will open the **Cluster explorer** from the reomte grafana.

The detail design of the cluster explorer will be published in figma.

### ***How do we do it technically ??***


From SUI, whenever we will navigate to any individual cluster and click for detail, SUI  will call the grafana instance  of the cluster as follows:

https://cluster1.synectiks.net/ 

SUI will collect the cluster URL from cmdb (CMDB has the cluster URL for each cluster)

Later on(phase 2) we will address this with proxy.

## Cluster Managed Services Explorer Navigation


***what to do for the App and Data services inside product enclaves that comes from cloud itself (cloud managed)??***



![alt](./images/CloudElements/ServiceExplorer.jpg)


![alt](./images/CloudElements/cloud-element4.jpg)



When a user will navigate till any App and Data Services of the the product modules ,and click on them , it will open the **service explorer** from the reomte grafana inside the cluster.

### ***How do we do it technically ??***

SUI will call the service explorer as follows:

https://cluster1.synectiks.net/rds-explorer/? DEPT=HR && PROD=HRMS && ENV=PROD && MODULE=Admission Service=RDS-postgresql

Then the corresponding grafana App (rds-explorer) will be served as headless UI and it will be renedered inside the iframe in SUI.

## Cloud Managed Services Explorer Navigation

## ***what to do for the App and Data services that are managed by cloud itself??***


![alt](./images/CloudElements/ServiceExplorer.jpg)


![alt](./images/CloudElements/cloud-element4.jpg)


When a user will navigate till any App and Data Services of the the product modules ,and click on them , it will open the **service explorer** from the reomte grafana.

### ***How do we do it technically ??***

SUI will call the service explorer as follows:

https://grafana.synectiks.net/rds-explorer/? DEPT=HR && PROD=HRMS && ENV=PROD && MODULE=Admission Service=RDS-postgresql

Then the corresponding grafana App (rds-explorer) will be served as headless UI and it will be renedered inside the iframe in SUI.

# How to implement Service Explorer 
## ***Proposed approach1***

* SUI-> <br>
    * -> CloudServiceExplorer { its a app (that has many dashboards with variable) inside the grafana (which is maintained for all cloud accounts)}<br>
    * -> ClusterExplorer (Its a diffrent grafana URL for every cluster)<br>
    * -> ClusterServiceExplorer -- It will be a App (that has many dashboards with variable) inside that cluster grafana url )

## How to develop Service Explorer in this aproach

We will write grafana App for each explorer. Each App will be created with a mix of different dashboards and each dashboards will have the following variables:

PRODUCT / ENVIRONMENT / MODULE / SERVICE

SUI will call the element explorer (say RDS explorer)

http://grafana.synectiks.net/a/rds-explorer-app?orgId=1&var-product=HRMS&var-env=production&var-module=admission&var-service=rds-postgresql&from=now-24h&to=now

SUI can know from the elementData , what sort of explorer need to be called , say for Ec2 machine , ec2-explorer,
for RDS-potgresql database, it will call rds-postgresql explorer.

### How the backend of DataSource will be implemented? 
<br>

---
### How developers will create the dashboards that will be used in element explorer App? 
<br>

---

![alt](./images/CloudElements/datasource1.jpg)

Developers can select the product , environment , module , App/Data service details in datasource query.

The datasource frontend ui can call the CMDB api to know the element type( Node / RDS / Dynamo etc) and its 
unique id.

The Datasouce UI should fire the Metrics/ Log / Trace / Api  queries with parameters (product , environment , module , App/Data service) 

The Datasource backend will fetch the query response as follows:

* For the (product , environment , module , App/Data service) combination , get the landing zone (Aws Account Id) for the service.
* For the landing zone , get credentials(role arn) etc from vault service and initailize the sdk credential.
* For EC2 / RDS  etc where we query with unique instance id , we wil find the element type and id from the CMDB api.
    * In case of Lambda , where for a particular App service, we run multiple lambdas , we will find the list of
    lambda functions for the (product , environment , module , App service) combination and issue the query for those lambdas.
* Then we will fire the query with the dimension being equated with instanceId or RDS id or Dynamo table etc





## ***Proposed approach2***

* SUI-> <br>
    * -> CloudServiceExplorer { its a view mainatined for each sepcific element (that has many dashboards with instanceId) inside the grafana (which is maintained for all cloud accounts)}<br>
    * -> ClusterExplorer (Its a diffrent grafana URL for every cluster)<br>
    * -> ClusterServiceExplorer -- It will be a view (that has many dashboards with instanceId) inside that cluster grafana url )


awsexplorer will maintain views for every account and each of App and Data Services, like as below:

aws-112234344-explorer

![alt](./images/CloudElements/cloud-element2.jpg)

For App and Data Services explorer --

aws-112234344-prod-env-app1-explorer

aws-112234344-prod-env-data1-explorer

![alt](./images/CloudElements/cloud-element4.jpg)

There would be many views that will be retained inside awsexplorer database , say we have HRMS and LOGISTIC prducts and for their PROD env , 

HR-PROD-BusinessService1-Data1  and LOGISTICS-PROD-BusinessService1-Data2, two RDS instance is there.

awsexplorer will maintian two views as follows:

aws-112234344-HR-PROD-BusinessService1-Data1-explorer

aws-112234344-LOGISTICS-PROD-BusinessService1-Data2-explorer

Each of the views will be a composure of few dashboards, say d1/d2/d3/d4.

This dasboards will have variables - say var1.

For RDS / Dynamo , this variables will be the ARN of the databases.

Whenever inside CMDB , we will add any product and its services , proxy API will create this view inside 
awsexplorer database with their variable being associated with the ARN of the RDS/Dynamo etc.

So , 
aws-112234344-HR-PROD-BusinessService1-Data1-explorer ( var1=ARN1)
aws-112234344-LOGISTICS-PROD-BusinessService1-Data2-explorer(var1=ARN2 )

So from SUI , wehn we click enable monitoring for any services , SUI will call a API to awsexplorer 

/createView/ ? Accid=112234344 & PROD=HR & ENV=PROD & ELEMENT=RDS & ARN=arn1 & logLocation="some path"

awsexplorer will implement the createView algorithm as follows:

        Get the view template ( view is the composure of diffrent dashboards that collect metric / log data)
        for the AccId , get the right datasource and replace the datasource in the view
        replace the ARN for metric data collection
        replace the log location for log data collection
        store the new view in database with proper naming convention, and update the view table.

For those cloud elements , where there is no single ARN , say for s3 and lambdas, where for a app and data 
service, multiple S3 bucket or multiple lambdas will be there , SUI , will call createView api as follows

/createViewWithoutArn/ ? Accid=112234344 & PROD=HR & ENV=PROD & ELEMENT=LAMBDA

awsexplorer will implement the createView algorithm as follows:

            Get the template view 
            for the AccId , get the right datasource and replace the datasource in the view
            Set the variable for PROD and ENV
            The query that gets fired to datasource will have prod and env and business service
            store the new view in database with proper naming convention, and update the view table.

Inside AWS datasource we need to implement this kind of queries where corresponding to product/env / business service , we will get list of s3 buckets or lambda API's.


# Required Api's


|API | Description | Input | Output |
|:---|:---|:---|:---|
|/enableMonitoring/{elementId} | enable the monitoring for that cloud element| elementId in CMDB |  return success or failure code|
|/enableAlerts/{elementId} | enable the monitoring for that cloud element | elementId in CMDB |  return success or failure |
|/createInput/{accountId}/?inputType=cloudWatchAPi & dataType = metrics | create the api based metric type input for a accounId | aws accountId |  return success or failure |
|/createInput/{accountId}/?inputType=cloudWatchAPi & dataType = log | create the api based log type input for a accounId | aws accountId |  return success or failure |
|/createInput/{accountId}/?inputType=promethus & dataType = metric | create the api based log type input for a accounId | aws accountId |  return success or failure |
|/setLogLocation/{elementId}/?ElementType=EC2 | set the log location for the EC2 machine | EC2 machine id |  return success or failure |
|/setLogLocation/{elementId}/?ElementType=RDS | set the log location for the RDS db | RDS id |  return success or failure |
|/elementExplorer? ElementType=EC2 & PROD= HRMS & ENV =PROD & SERVICE = Admission & Ec2Id= 53545 | Open the EC2 explorer for a specific EC2 element | EC2 id or ARN |  return success or failure |
|/elementExplorer? ElementType=RDS & PROD= HRMS & ENV =PROD & SERVICE = Admission & rdsId= 55646 | Open the RDS explorer for a specific RDS element | EC2 id or ARN |  return success or failure |

# Api's Algorithm
## enableMonitoring
We could take two aproaches --
1. Create dynamic views for every element and store inside the cmdb database, the underlying algo is :

    - From Catalogues filter all the Dashbooards available for that element
    - Check the dashboards (Performance / Availability...) with available Inputs and if there are matching inputs(datasources), import those dashboards by replacing the DS and ARN.

2. Create grafana plugin App for every element and call them with ARN and log location as follows:

/elementsExplorer?   Accid=112234344 & ElementType=EC2 & PROD= HRMS & ENV =PROD & SERVICE = Admission & Ec2Id= 53545 & logLocation="some path"

/elementsExplorer/ ? Accid=112234344 & ElementType=RDS & PROD=HRMS & ENV=PROD & SERVICE = Admission  & ARN=arn1 & logLocation="some path"

Every elmentExplorer App plugin will have the variables 
    Var accId , Var ElementType , Var PRODUCT , Var ENV , Var Service , Var Arn , Var Loglocation


# Required Code Changes

**UI Changes**
 - headless view API
 - when we call /elementsExplorer/ ? Accid=112234344 & ElementType=RDS & PROD=HRMS & ENV=PROD & SERVICE = Admission  & ARN=arn1 & logLocation="some path", plugin variables gets set properly

 - element explorer App plugin that opens with the variable being set from the URL parameters
 - A datasource plugin  that takes AccId / Product / ENV / SVC and return metric / log / api data

 - Write explorer for WAF / APiGw / RDS / DYNAMO / S3 / Lambda....

 **Dashboard.js** - Futuristics

**API Changes**

    - datasource plugin  API implementation ( Accid / Prod / Env / SVC / ElementType / Arn or NoArn / Query ) -- (AwsApi / Aws-metric / Aws-logs)

# CMDB Population 

We wil add product/env/modules/ (App & Data) services metadata inside Appkube platform from the UI (manually) where the product and Env is not created from the Appkube automation Engine. Every product/env  will have association with Departments. Unless a full fledge UI is developed and we have the capability to add environments / modules / app & Data services inside modules, we can have them imported through a JSON     


# Different Table Structure

## Cloud_Element 
This table will hold all the discovered cloud elements for any specific AWS account.

|Column Name | Description | type | Format-Example|
|:---|:---|:---|:---|
|id | id of the element | int |unique id |
|elementType | Type of the service it belongs | string | EC2/EKS/ECS/LAMBDA/RDS....|
|cloudIdentity | The ARN or any unique identity for the resource|jsonb | [JSONlink](./jsons/Cloud_Element/identity.json)|
|hardwareLocation| Which account and which VPC does it belong | jsonb |[JSONlink](./jsons/Cloud_Element/hardwareLocation.json)|
|hostedServices| Which business and common services it hosts | jsonb |[JSONlink](./jsons/Cloud_Element/hostedServices.json)|
|SLA| Its calculated SLA scores | jsonb |[JSONlink](./jsons/Cloud_Element/sla.json)|
|COST| Its calculated Cost| jsonb |[JSONlink](./jsons/Cloud_Element/cost.json)|
|VIEW| All the Dashboards that is associated with the element | jsonb |[JSONlink](./jsons/Cloud_Element/view.json)|
|CONFIG| Element Config Json  | jsonb |[JSONlink](./jsons/Cloud_Element/config.json)|
|COMPLIANCE| ELement Compliance Json | jsonb |[JSONlink](./jsons/Cloud_Element/compliance.json)|

## Snapshot of Cloud_Element  table

|id |elementType | cloudIdentity | hardwareLocation | hostedServices | SLA | Cost | VIEW | CONFIG | COMPLIANCE|
|:---|:---|:---|:---|:---|:---|:---|:---|:---|:---|
|1234 | EKS |[JSONlink](./jsons/Cloud_Element/identity.json) | [JSONlink](./jsons/Cloud_Element/hardwareLocation.json) |[JSONlink](./jsons/Cloud_Element/hostedServices.json)|SLAJSON|COSTJSON | VIEWJSON | CONFIGJSON |COMPLIANCEJSON|
|1235 | ECS |[JSONlink](./jsons/Cloud_Element/identity.json) | [JSONlink](./jsons/Cloud_Element/hardwareLocation.json) |[JSONlink](./jsons/Cloud_Element/hostedServices.json)|SLAJSON|COSTJSON | VIEWJSON | CONFIGJSON |COMPLIANCEJSON|
|1236 | EC2 |[JSONlink](./jsons/Cloud_Element/identity.json)| [JSONlink](./jsons/Cloud_Element/hardwareLocation.json) |[JSONlink](./jsons/Cloud_Element/hostedServices.json)|SLAJSON|COSTJSON | VIEWJSON | CONFIGJSON |COMPLIANCEJSON|
|1237 | LAMBDA |[JSONlink](./jsons/Cloud_Element/identity.json)| [JSONlink](./jsons/Cloud_Element/hardwareLocation.json) |[JSONlink](./jsons/Cloud_Element/hostedServices.json)|SLAJSON|COSTJSON | VIEWJSON | CONFIGJSON |COMPLIANCEJSON|
|1238 | RDS |[JSONlink](./jsons/Cloud_Element/identity.json)| [JSONlink](./jsons/Cloud_Element/hardwareLocation.json) |[JSONlink](./jsons/Cloud_Element/hostedServices.json)|SLAJSON|COSTJSON | VIEWJSON | CONFIGJSON |COMPLIANCEJSON|
|1239 | DYNAMO |[JSONlink](./jsons/Cloud_Element/identity.json)| [JSONlink](./jsons/Cloud_Element/hardwareLocation.json) |[JSONlink](./jsons/Cloud_Element/hostedServices.json)|SLAJSON|COSTJSON | VIEWJSON | CONFIGJSON |COMPLIANCEJSON|
|1241 | ApiGw |[JSONlink](./jsons/Cloud_Element/identity.json)| [JSONlink](./jsons/Cloud_Element/hardwareLocation.json) |[JSONlink](./jsons/Cloud_Element/hostedServices.json)|SLAJSON|COSTJSON | VIEWJSON | CONFIGJSON |COMPLIANCEJSON|
|1242 | NLB |[JSONlink](./jsons/Cloud_Element/identity.json)| [JSONlink](./jsons/Cloud_Element/hardwareLocation.json) |[JSONlink](./jsons/Cloud_Element/hostedServices.json)|SLAJSON|COSTJSON | VIEWJSON | CONFIGJSON |COMPLIANCEJSON|
|1243 | OpenSearch |[JSONlink](./jsons/Cloud_Element/identity.json)| [JSONlink](./jsons/Cloud_Element/hardwareLocation.json) |[JSONlink](./jsons/Cloud_Element/hostedServices.json)|SLAJSON|COSTJSON | VIEWJSON | CONFIGJSON |COMPLIANCEJSON|
|1244 | CacheDB |[JSONlink](./jsons/Cloud_Element/identity.json)| [JSONlink](./jsons/Cloud_Element/hardwareLocation.json) |[JSONlink](./jsons/Cloud_Element/hostedServices.json)|SLAJSON|COSTJSON | VIEWJSON | CONFIGJSON |COMPLIANCEJSON|


## Micro_Services
This table will hold all the discovered cloud elements for any specific AWS account.

|Column Name | Description | type | Format-Example|
|:---|:---|:---|:---|
|id | id of the service | int |unique id |
|name | name of the service | string |unique |
|department | department it belongs | string |unique |
|product | product of the service | string |unique |
|environment | environment of the service | string |unique |
|serviceType | Type of the service it belongs | string | Business ,Common  |
|serviceTopology| how its composed of Gw/LB/AppLayer/DataLayer| jsonb |[JSONlink](./jsons/Micro_Service/serviceTopology.json)|
|SLA| Its calculated SLA scores | jsonb |[JSONlink](./jsons/Cloud_Element/sla.json)|
|COST| Its calculated Cost| jsonb |[JSONlink](./jsons/Cloud_Element/cost.json)|
|VIEW| All the Dashboards that is associated with the element | jsonb |[JSONlink](./jsons/Cloud_Element/view.json)|
|CONFIG| Element Config Json  | jsonb |[JSONlink](./jsons/Cloud_Element/config.json)|
|COMPLIANCE| ELement Compliance Json | jsonb |[JSONlink](./jsons/Cloud_Element/compliance.json)|


## Snapshot of Micro_Services table

|id |Name| Department |  Product | Environment | serviceType |serviceTopology| SLA | Cost | VIEW | CONFIG | COMPLIANCE|
|:---|:---|:---|:---|:---|:---|:---|:---|:---|:---|:---|:---|
|1255 | Admission |HR |HRMS|PROD| BUSINESS|[JSONlink](./jsons/Micro_Service/serviceTopology.json) |SLAJSON|COSTJSON | VIEWJSON | CONFIGJSON |COMPLIANCEJSON|


## organization
This table will hold organization information.

|Column Name | Description | type | Format-Example|
|:---|:---|:---|:---|
|id | organization id | int |unique id |
|name | organization name | string |unique |



## Snapshot of organization table

|id |name| 
|:---|:---|
|1 | Synectiks |



## department
This table will hold department information.

|Column Name | Description | type | Format-Example|
|:---|:---|:---|:---|
|id | department id | int |unique id |
|name | department name | string |unique |
|organization_id | forign key organization table | int | integer id |

## Snapshot of department table

|id |name|organization_id|  
|:---|:---|:---|
|1 | Human Resource | 1|
|2 | IT Networking | 1|



## cloud_environment
This table will hold cloud wise account and its cross role information.

|Column Name | Description | type | Format-Example|
|:---|:---|:---|:---|
|id | cloud environment id | int |unique id |
|department_id | forign key organization table | int |integer |
|description | description | string |  |
|account_id | cloud(AWS/AZURE/GCP ect.) account id | string |  |
|display_name | name given to any account | string |  |
|role_arn | cross account role arn | string |  |
|external_id | external id of an account | string |  |
|cloud | cloud name (AWS/AZURE/GCP) | string |  |

## Snapshot of cloud_environment table

|id |department_id|description|account_id|display_name|role_arn|external_id|cloud|  
|:---|:---|:---|:---|:---|:---|:---|:---|
|1 | 1 | AWS account for for Human Resource department|7869574624|HRD Account|arn:aws:iam::7869574624:role/CrossAccount|abcdqwe|AWS|


## cloud_element_summary
This table will have summary of an account.

|Column Name | Description | type | Format-Example|
|:---|:---|:---|:---|
|id | cloud element id | int |unique id |
|cloud_environment_id | forign key cloud_environment table | int |integer |
|summary_json | JSON having compelte summanry of an account | jsonb |  |

## Snapshot of cloud_element_summary table

|id |cloud_environment_id|summary_json|  
|:---|:---|:---|
|1 | 1 | [JSONlink](./jsons/Cloud_Element_Summary/summary.json)|


## deployment_environment
This table will have summary of an account.

|Column Name | Description | type | Format-Example|
|:---|:---|:---|:---|
|id | environment id | int |unique id |
|name | environment name | string |unique |


## Snapshot of deployment_environment table

|id |name|  
|:---|:---|
|1 | DEV |
|2 | TEST |
|3 | STAGE |
|4 | PROD |



## catalogue
This table will hold dev/sec/ops dashboards and datasource master datas.

|Column Name | Description | type | Format-Example|
|:---|:---|:---|:---|
|id | catalogue id | int |unique id |
|details | dev/sec/ops datasource and dashboards master data | jsonb | |


## Snapshot of catalogue table

|id |details|  
|:---|:---|
|1 | [JSONlink](./jsons/Catalogue/catalogue.json) |


# CMDB API End Points
## Base API : http://localhost:6057/api
## [CURD APIs](./md-files/cmd-curd-apis.md) 

## **Please give endpoints for the following Queries --**


For every Organization --

    - Its department , products , products environment , associated landing Zones and Product Enclaves 
    - Department wise product and services Cost & SLA's 
    - Landing Zone Wise Costs 

For every landing Zones --

    - How Many AWS services Elements it has - Product Enclaves/ Global / Gateway Services / Clusters / Serverles / Nodes / Databases / Datalake services
    - How many products / envs it hosts?
    - How many Micro-Services its hosts , which product / env they belong
    - Its available ApiGateways/LB's/ Product Enclaves / Clusters/ Service Mesh / Cloud Managed Services (With filters of Data(Cache / SQL / NOSQL ..) & DataLake  Services) 

For every Product Enclaves --

    - How many clusters ? 
    - Each cluster hosted App & Data Layer , which microservices they belong?

For every Product --

    - How many environment it has ?
    - List the MicroServices for that Env
    - List the Api Gateway / LB for that Env

For every Microservices --

    - What are their Gateway / Lb / App / Data Layer
    - Find the Cost & SLA's of the Microservice
    - Find the Cost & SLA's of their Gateway / Lb / App / Data Layer
    - Find the Cost & SLA's  trending of the Microservice
    - Find the Cost & SLA's trending of their Gateway / Lb / App / Data Layer
    - Find the compliance of its Gateway / Lb / App / Data Layer

Find the Cost(daily/weekly/monthly) of the entire organization

    - Department Wise Cost
    - Product Wise Cost 
    - Environment Wise Cost
    - Department/ Product  wise Costs
    - Department/ Product / Environemnt wise Costs
    - Department/ Product / Environemnt / Microservice Wise Costs

Find the SLA's (daily/weekly/monthly) of the entire organization

    - Department/ Product / Environemnt wise CostsSLA's
    - Department/ Product / Environemnt / Microservice Wise SlA's
    - Department/ Product / Environemnt / Microservice/ (Gw/LB/App/Data layer) Wise SlA's
  

## Query API End Points


#### **1. organization wise**

| description | method | end point | Request | Response | 
|:---|:---|:---|:---|:---|
|departments|GET | /organizations/{orgId}/departments | | [Response JSON](./jsons/Department/all.json)|
|products|GET | /organizations/{orgId}/products | | |
|landing Zones |GET | /organizations/{orgId}/landing-zone | | |
|product enclaves|GET | /organizations/{orgId}/product-enclave | | |
|resource count|GET | /organizations/{orgId}/cloud-environments/count | | |
|resource summary|GET | /organizations/{orgId}/cloud-environments/summary | | |
|services |GET | /organizations/{orgId}/services | | |

#### **2. organization and department wise**
| description | method | end point | Request | Response | 
|:---|:---|:---|:---|:---|
|products|GET | /organizations/{orgId}/departments/{depId}/products | | |
|landing-zone |GET | /organizations/{orgId}/departments/{depId}/landing-zone | | | 
|product enclave |GET | /organizations/{orgId}/departments/{depId}/product-enclave | | |
|services |GET | /organizations/{orgId}/departments/{depId}/services | | |

#### **3. organization and cloud wise landing Zones**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|GET | /organizations/{orgId}/cloud/{cloudName}/landing-zone | | | organization and cloud wise landing-zones|

#### **4. organization, department and cloud wise landing Zones**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|GET | /organizations/{orgId}/departments/{depId}/cloud/{cloudName}/landing-zone | | | organization, department and cloud wise landing-zones|

#### **5. organization and environment wise**
##### **products**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|GET | /organizations/{orgId}/environments/{env}/products | | | Get list of products of given organization id and deployment environment name |

#### **6. organization, department and environment wise**
##### **products**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|GET | /organizations/{orgId}/departments/{depId}/environments/{env}/products | | | organization, department and deployment environment wise products |
	

#### **7. organization and landing zone wise**
##### **product enclaves**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|GET | /organizations/{orgId}/landing-zone/{landingZone}/product-enclave | | | Get list of product-enclaves of given organization id and landing-zone |

#### **8. organization, department and landing zone wise**
##### **product enclaves**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|GET | /organizations/{orgId}/departments/{depId}/landing-zone/{landingZone}/product-enclave | | | Get list of product-enclaves of given organization id, department id and landing-zone |

		
#### **department wise product, services cost & SLA's**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|GET | /organizations/{orgId}/cloud/{cloudName}/services | | | Get list of services of given organization id and cloud name|
|GET | /organizations/{orgId}/departments/{depId}/cloud/{cloudName}/services | | | Get list of services of given organization id, department id and cloud name |
|GET | /organizations/{orgId}/departments/{depId}/landing-zone/{landingZone}/cloud/{cloudName}/services | | | Get list of services of given organization id, department id, landing-zone and cloud name|
|GET | /organizations/{orgId}/products/associated-services | | | Get list of services of every product of given organization |
|GET | /organizations/{orgId}/products/{product}/services | | | Get list of services of given organization id and product |
|GET | /organizations/{orgId}/departments/{depId}/products/{product}/services | | | Get list of services of given organization id, department id and product|
|GET | /organizations/{orgId}/departments/{depId}/products/associated-services | | | Get list of services of every product of given organization id and department id|
|GET | /organizations/{orgId}/cloud/{cloudName}/products/{product}/services | | | Get list of services of given organization id, cloud name and product|
|GET | /organizations/{orgId}/departments/{depId}/cloud/{cloudName}/products/{product}/services | | | Get list of services of given organization id, department id, cloud name and product|
		
		
#### **landing zone wise costs**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|GET | /organizations/{orgId}/landing-zone/{landingZone}/costs | | | Get list of costs of given organization id and landing-zone |
|GET | /organizations/{orgId}/landing-zone/{landingZone}/cloud/{cloudName}/costs | | | Get list of costs of given organization id, landing-zone and cloud name|
|GET | /organizations/{orgId}/landing-zone/{landingZone}/products/{product}/costs | | | Get list of costs of given organization id, landing-zone and product|
|GET | /organizations/{orgId}/departments/{depId}/landing-zone/{landingZone}/products/{product}/costs | | | Get list of costs of given organization id, department id, landing-zone and product|
|GET | /organizations/{orgId}/landing-zone/{landingZone}/cloud/{cloudName}/products/{product}/costs | | | Get list of costs of given organization id, landing-zone, cloud name and product|
|GET | /organizations/{orgId}/departments/{depId}/landing-zone/{landingZone}/cloud/{cloudName}/products/{product}/costs | | | Get list of costs of given organization id, department id, landing-zone, cloud name and product|


### **For every landing Zone**
#### **How Many AWS services Elements it has - Product Enclaves/ Global / Gateway Services / Clusters / Serverles / Nodes / Databases / Datalake services**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|GET | /organizations/{orgId}/landing-zone/cloud-elements | | | Get list of cloud elements for each landing zone of given organization id |
|GET | /organizations/{orgId}/landing-zone/{landingZone}/cloud-elements | | | Get list of cloud elements of given organization id and landing zone|
|GET | /organizations/{orgId}/landing-zone/{landingZone}/cloud-elements/{cloud-element-type} | | | Get list of cloud elements of given organization id, landing zone and cloud element type|
|GET | /organizations/{orgId}/departments/{depId}/landing-zone/{landingZone}/cloud-elements | | | Get list of cloud elements of given organization id, department id and landing-zone |
|GET | /organizations/{orgId}/departments/{depId}/landing-zone/{landingZone}/cloud-elements/{cloud-element-type} | | | Get list of cloud elements of given organization id, department id, landing-zone and cloud element type |

#### **How many products**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|GET | /organizations/{orgId}/landing-zone/{landingZone}/products | | | Get list of products of given organization id and landing-zone |
|GET | /organizations/{orgId}/departments/{depId}/landing-zone/{landingZone}/products | | | Get list of products of given organization id, department id and landing-zone |
|GET | /organizations/{orgId}/landing-zone/{landingZone}/environment/{envId}/products | | | Get list of products of given organization id, landing-zone and deployment environment |
|GET | /organizations/{orgId}/departments/{depId}/landing-zone/{landingZone}/environment/{envId}/products | | | Get list of products of given organization id, department id, landing-zone and deployment environment |

#### **product belongs to how many envs?**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|GET | /organizations/{orgId}/landing-zone/{landingZone}/products/associated-environments | | | Get list of associated environment, every product belongs to of given organization id and landing-zone |
|GET | /organizations/{orgId}/departments/{depId}/landing-zone/{landingZone}/products/associated-environments | | | Get list of associated environment, every product belongs to of given organization id, department id and landing-zone |
|GET | /organizations/{orgId}/landing-zone/{landingZone}/products{productName}/associated-environments | | | Get list of associated environment, a product belongs to of given organization id and landing-zone |
|GET | /organizations/{orgId}/departments/{depId}/landing-zone/{landingZone}/products{productName}/associated-environments | | | Get list of associated environment, a product belongs to of given organization id, department id and landing-zone |	
	
#### **How many Micro-Services its hosts -, which product / env they belong**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|GET | /organizations/{orgId}/landing-zone/{landingZone}/services | | | Get list of services of given organization id and landing-zone |
|GET | /organizations/{orgId}/departments/{depId}/landing-zone/{landingZone}/services | | | Get list of services of given organization id, department id and landing-zone |
|GET | /organizations/{orgId}/landing-zone/{landingZone}/services/associated-product | | | Get list of associated products of every service of given organization id and landing-zone |
|GET | /organizations/{orgId}/departments/{depId}/landing-zone/{landingZone}/services/associated-product | | | Get list of associated product of every service of given organization id, department id and landing-zone |
|GET | /organizations/{orgId}/landing-zone/{landingZone}/services/{serviceName}/associated-product | | | Get list of associated products of given service, organization id and landing-zone |
|GET | /organizations/{orgId}/departments/{depId}/landing-zone/{landingZone}/services/{serviceName}/associated-product | | | Get list of associated products of given service, organization id, department id and landing-zone |
|GET | /organizations/{orgId}/landing-zone/{landingZone}/services/associated-product/associated-environment | | | Get list of associated environment of every associated products of every service of given organization id and landing-zone |
|GET | /organizations/{orgId}/departments/{depId}/landing-zone/{landingZone}/services/associated-product/associated-environment | | | Get list of associated environment of every associated product of every service of given organization id, department id and landing-zone |
|GET | /organizations/{orgId}/landing-zone/{landingZone}/services/{serviceName}/associated-product/associated-environment | | | Get list associated environment of every associated product of given service, organization id and landing-zone |
|GET | /organizations/{orgId}/departments/{depId}/landing-zone/{landingZone}/services/{serviceName}/associated-product/associated-environment | | | Get list associated environment of every associated product of given service, organization id, department id and landing-zone |


#### **Its available ApiGateways/LB's/ Product Enclaves / Clusters/ Service Mesh / Cloud Managed Services (With filters of Data(Cache / SQL / NOSQL ..) & DataLake  Services)**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|GET | /organizations/{orgId}/landing-zone/{landingZone}/cloud-element-type/{cloud-element-type}/services | | | Get list of services of given organization id, landing zone and cloud element type|
|GET | /organizations/{orgId}/landing-zone/{landingZone}/cloud-element-type/{cloud-element-type}/{cloudElementId}/services | | | Get list of services of given organization id, landing zone, cloud element type and cloud element id|
|GET | /organizations/{orgId}/departments/{depId}/landing-zone/{landingZone}/cloud-element-type/{cloud-element-type}/{cloudElementId}/services | | | Get list of services of given organization id, department id, landing zone, cloud element type and cloud element id|


### **For every Product Enclaves**
#### **How many clusters ?**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|GET | /organizations/{orgId}/product-enclave/associated-clusters | | | Get list of associated clusters for each product-enclaves of given organization id |
|GET | /organizations/{orgId}/product-enclave/{productEnclave}/clusters | | | Get list of clusters for given organization id and product-enclave |
|GET | /organizations/{orgId}/departments/{depId}/product-enclave/associated-clusters | | | Get list of associated clusters for each product-enclave of given organization id and department id |
|GET | /organizations/{orgId}/departments/{depId}/product-enclave/{productEnclave}/clusters | | | Get list of clusters for given organization id, department id and product-enclave|
|GET | /organizations/{orgId}/landing-zone/{landingZone}/product-enclave/associated-clusters | | | Get list of associated clusters for each product-enclave of given organization id and landing-zone |
|GET | /organizations/{orgId}/landing-zone/{landingZone}/product-enclave/{productEnclave}/clusters | | | Get list of clusters of given organization id, landing-zone and product enclave|
|GET | /organizations/{orgId}/departments/{depId}/landing-zone/{landingZone}/product-enclave/associated-clusters | | | Get list of associated cluster for each product-enclaves of given organization id, department id and landing-zone |
|GET | /organizations/{orgId}/departments/{depId}/landing-zone/{landingZone}/product-enclave/{productEnclave}/associated-clusters | | | Get list of cluster of given organization id, department id, landing-zone and product enclave|

#### **Each cluster hosted App & Data Layer , which microservices they belong?**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|GET | /organizations/{orgId}/product-enclave/{productEnclave}/clusters/{clusterId}/services | | | Get list of services for given organization id, product-enclave and cluster id |
|GET | /organizations/{orgId}/departments/{depId}/product-enclave/{productEnclave}/clusters/{clusterId}/services | | | Get list of services for given organization id, department id, product-enclave and cluster id |
|GET | /organizations/{orgId}/departments/{depId}/landing-zone/{landingZone}/product-enclave/{productEnclave}/clusters/{clusterId}/services | | | Get list of services for given organization id, department id, landing zone, product-enclave and cluster id |
|GET | /organizations/{orgId}/product-enclave/{productEnclave}/clusters/{clusterId}/{serviceType}/services | | | Get list of services for given organization id, product-enclave, cluster id and service type|
|GET | /organizations/{orgId}/departments/{depId}/product-enclave/{productEnclave}/clusters/{clusterId}/{serviceType}/services | | | Get list of services for given organization id, department id, product-enclave, cluster id and service type|
|GET | /organizations/{orgId}/departments/{depId}/landing-zone/{landingZone}/product-enclave/{productEnclave}/clusters/{clusterId}/{serviceType}/services | | | Get list of services for given organization id, department id, landing zone, product-enclave, cluster id and service type |


### **For every Product**
#### **List the MicroServices for that Env**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|GET | /organizations/{orgId}/products/{product}/environments/{env}/services | | | Get list of services of given organization id, product and deployment environment|
|GET | /organizations/{orgId}/departments/{depId}/products/{product}/environments/{env}/services | | | Get list of services of given organization id, department id, product and deployment environment|
|GET | /organizations/{orgId}/landing-zone/{landingZone}/products/{product}/environments/{env}/services | | | Get list of services of given organization id, landing-zone, product and deployment environment|
|GET | /organizations/{orgId}/departments/{depId}/landing-zone/{landingZone}/products/{product}/environments/{env}/services | | | Get list of services of given organization id, department id, landing-zone, product and deployment environment|
|GET | /organizations/{orgId}/cloud/{cloudName}/products/{product}/environments/{env}/services | | | Get list of services of given organization id, cloud name, product and deployment environment|
|GET | /organizations/{orgId}/departments/{depId}/cloud/{cloudName}/products/{product}/environments/{env}/services | | | Get list of services of given organization id, department id, cloud name, product and deployment environment|
|GET | /organizations/{orgId}/landing-zone/{landingZone}/cloud/{cloudName}/products/{product}/environments/{env}/services | | | Get list of services of given organization id, landing-zone, cloud name, product and deployment environment|
|GET | /organizations/{orgId}/departments/{depId}/landing-zone/{landingZone}/cloud/{cloudName}/products/{product}/environments/{env}/services | | | Get list of services of given organization id, department id, landing-zone, cloud name, product and deployment environment|

#### **List the Api Gateway / LB for that Env**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|GET | /organizations/{orgId}/products/{product}/environments/{env}/cloud-elements | | | Get list of cloud elements of given organization id, product and deployment environment|
|GET | /organizations/{orgId}/products/{product}/environments/{env}/{cloudElementType}/cloud-elements | | | Get list of cloud elements of given organization id, product, deployment environment and cloud element type|

### **For every Microservice**
#### **What are their Gateway / Lb / App / Data Layer**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|GET | /organizations/{orgId}/departments/{depId}/landing-zone/{landingZone}/services/{serviceName}/associated-cloud-elements | | | Get list of associated cloud elements given organization id, department id, landing zone and service name|

#### **Find the Cost & SLA's of the Microservice**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|GET | /organizations/{orgId}/departments/{depId}/landing-zone/{landingZone}/services/{serviceName}/cost-and-sla | | | Get COST and SLA of given organization id, department id, landing zone and service name|

#### **Find the Cost & SLA's of their Gateway / Lb / App / Data Layer**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|GET | /organizations/{orgId}/departments/{depId}/landing-zone/{landingZone}/services/{serviceName}/{cloudElementType}/cost-and-sla | | | Get COST and SLA of given organization id, department id, landing zone, service name and cloud element type|
|GET | /organizations/{orgId}/departments/{depId}/landing-zone/{landingZone}/services/{serviceName}/{serviceType}/cost-and-sla | | | Get COST and SLA of given organization id, department id, landing zone, service name and service type|


#### **What are their Gateway / Lb / App / Data Layer**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|GET | /organizations/{orgId}/departments/{depId}/landing-zone/{landingZone}/services/{serviceName}/cloud-element-type | | | Get list of cloud elements (API Gateway/LB etc.) of given organization id, department id, landing zone, service name|
|GET | /organizations/{orgId}/departments/{depId}/landing-zone/{landingZone}/services/{serviceName}/{serviceType} | | | Get list of service types (App layer/Data layer) of given organization id, department id, landing zone, service name and service type|


#### **Find the Cost & SLA's  trending of the Microservice**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|GET | /organizations/{orgId}/departments/{depId}/landing-zone/{landingZone}/services/{serviceName}/trend/{frequency} | | | Get cost trend of a microservice of given organization id, department id, landing zone, service name|

#### **Find the Cost & SLA's trending of their Gateway / Lb / App / Data Layer**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|GET | /organizations/{orgId}/departments/{depId}/landing-zone/{landingZone}/services/{serviceName}/{cloudElementType}/trend/{frequency} | | | Get cost trend of cloud element type (ApiGateways/LB) of a microservice of given organization id, department id, landing zone, service name|
|GET | /organizations/{orgId}/departments/{depId}/landing-zone/{landingZone}/services/{serviceName}/{serviceType}/trend/{frequency} | | | Get cost trend of service type (App/Data layer) of a microservice of given organization id, department id, landing zone, service name|


#### **Find the compliance of its Gateway / Lb / App / Data Layer**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|GET | /organizations/{orgId}/departments/{depId}/landing-zone/{landingZone}/services/{serviceName}/{cloudElementType}/compliance/{rules} | | | Get compliance of cloud element type (ApiGateways/LB) of a microservice of given organization id, department id, landing zone, service name and rules|
|GET | /organizations/{orgId}/departments/{depId}/landing-zone/{landingZone}/services/{serviceName}/{serviceType}/compliance/{rules} | | | Get compliance of service type (App/Data layer) of a microservice of given organization id, department id, landing zone, service name and rules|



### **Find the Cost(daily/weekly/monthly) of the entire organization**
#### **Department Wise Cost**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|GET | /organizations/{orgId}/departments/{depId}/cost/{frequency} | | | Get cost for given organization id, department id and frequency|

#### **Product Wise Cost**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|GET | /organizations/{orgId}/products/{product}/cost/{frequency} | | | Get cost for given organization id, product and frequency|
		
#### **Environment Wise Cost**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|GET | /organizations/{orgId}/environments/{env}/cost/{frequency} | | | Get cost for given organization id, environment and frequency|
		
#### **Department/ Product  wise Costs**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|GET | /organizations/{orgId}/departments/{depId}/products/{product}/cost/{frequency} | | | Get cost for given organization id, department id, product and frequency|

#### **Department/ Product / Environemnt wise Costs**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|GET | /organizations/{orgId}/departments/{depId}/products/{product}/environments/{env}/cost/{frequency} | | | Get cost for given organization id, department id, product, environment and frequency|

#### **Department/ Product / Environemnt / Microservice Wise Costs**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|GET | /organizations/{orgId}/departments/{depId}/products/{product}/environments/{env}/services/{serviceName}/cost/{frequency} | | | Get cost for given organization id, department id, product, environment, service name and frequency|
		

### **Find the SLA's (daily/weekly/monthly) of the entire organization**
#### **Department/ Product / Environemnt wise CostsSLA's**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|GET | /organizations/{orgId}/departments/{depId}/products/{product}/environments/{env}/sla/{frequency} | | | Get SLA for given organization id, department id, product, environment and frequency|
		
#### **Department/ Product / Environemnt / Microservice Wise SlA's**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|GET | /organizations/{orgId}/departments/{depId}/products/{product}/environments/{env}/services/{serviceName}/sla/{frequency} | | | Get SLA for given organization id, department id, product, environment, service name and frequency|
		
#### **Department/ Product / Environemnt / Microservice/ (Gw/LB/App/Data layer) Wise SlA's	**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|GET | /organizations/{orgId}/departments/{depId}/products/{product}/environments/{env}/services/{serviceName}/{cloudElementType}/sla/{frequency} | | | Get SLA for given organization id, department id, product, environment, service name, cloud element type and frequency|
|GET | /organizations/{orgId}/departments/{depId}/products/{product}/environments/{env}/services/{serviceName}/{serviceType}/sla/{frequency} | | | Get SLA for given organization id, department id, product, environment, service name, service type and frequency|




### **Please verify entity checklist -**

- Organization
- Department 
- Product
- Environment
- MicroServices (GW/LB/App/Data Layers)
- Cloud Element (cluster/ cloud-managed elements / global elements / gateway elements)
- Landing Zones
- Product Enclaves
- Artifact Catalogue
- Artifacts
- Library
- Automation-Jobs
- ToolChains
- Service Cost & Quaity Trend 
- Process Cost & Quality Trend

## **Please verify the metadata checklists :**

App_Services_Type -- 
Like NodesJs / Golang / Java / Python Api Server / Lambda / StepFunction / Batch

Data_Services_Type
SQLDB (Mysql/Mssql/Postgresql) , CacheDB(Redis/Memcache) , SearchDB(Opensearch/ ElasticSearch) , NOSQL DB (Mongo/ Dynamo ), GRAPH DB , METRIC DB , LOG DB , Object Store(S3) , Github 

Data stored as follows:

Mysql    	SQLDB
Mssql    	SQLDB
Postgresql 	SQLDB
Redis		CacheDB
MemCache 	CacheDB
HazzleCast      CacheDB
ElasticSearch   SearchDB
OpenSearch      SearchDB
Dynamo		NOSQL DB
Mongo		NOSQL DB


Master ENVTYPE -- DEV/TEST/STAGE/PROD




## **Please give endpoints for the following Provisioning --**

For every cloudelement

    - Set the log / Trace Location
    - Set the cost & SLA 
    - enable the monitoring
    - enable the alerts

For every Microservices 

    - Set/update the topology (GW/LB/APP/DATA)
    - Set the cost & SLA 
    - Set the dependent Microservices

For every organization / department 

    - Add landing Zone

For every Landing Zone ---

    - Add product enclave (WAF / Gateway / LB / VPC/ Few Clusters )
    - Add cluster in product enclave
    - Add WAF/Gateway/LB/product enclave / Cluster for a particular product env
    - Add product(specifying its env) in product enclave
    - Add Business Service in a product
    - Add Common Service in a product
    - Run Audits on Product
    - Run Audits on Landing Zone
    - Run Audits on Product Enclave
    - Run Audits on Cluster


## Provisioned End Points

### **For every cloudelement**
#### **Set the log / Trace Location**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|POST | /organizations/{orgId}/departments/{depId}/landing-zone/{landingZone}/cloud-elements/{cloudElementId}/log-location | | | create log location of a given cloud element|
|PATCH | /organizations/{orgId}/departments/{depId}/landing-zone/{landingZone}/cloud-elements/{cloudElementId}/log-location | | | update log location of a given cloud element|

#### **Set the cost & SLA**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|POST | /organizations/{orgId}/departments/{depId}/landing-zone/{landingZone}/cloud-elements/{cloudElementId}/cost/{frequency} | | | create cost of a given cloud element|
|PATCH | /organizations/{orgId}/departments/{depId}/landing-zone/{landingZone}/cloud-elements/{cloudElementId}/cost/{frequency} | | | update cost of a given cloud element|
|POST | /organizations/{orgId}/departments/{depId}/landing-zone/{landingZone}/cloud-elements/{cloudElementId}/sla/{frequency} | | | create SLA of a given cloud element|
|PATCH | /organizations/{orgId}/departments/{depId}/landing-zone/{landingZone}/cloud-elements/{cloudElementId}/sla/{frequency} | | | update SLA of a given cloud element|

#### **enable the monitoring**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|POST | /organizations/{orgId}/departments/{depId}/landing-zone/{landingZone}/cloud-elements/{cloudElementId}/enable-monitoring | | | enable monitoring of a given cloud element|

#### **enable the alerts**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|POST | /organizations/{orgId}/departments/{depId}/landing-zone/{landingZone}/cloud-elements/{cloudElementId}/enable-alert | | | enable alerts of a given cloud element|


### **For every Microservices**
#### **Set/update the topology (GW/LB/APP/DATA)**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|PATCH | /services/{id}/topoloy | | | update topology of a given service id|

#### **Set the cost & SLA**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|POST | /services/{id}/cost | | | create cost of a given service id|
|PATCH | /services/{id}/cost | | | update cost of a given service id|
|POST | /services/{id}/sla | | | create SLA of a given service id|
|PATCH | /services/{id}/sla | | | update SLA of a given service id|
|POST | /services/{id}/cost-sla | | | create cost and SLA of a given service id|
|PATCH | /services/{id}/cost-sla | | | update cost and SLA of a given service id|


#### **Set the dependent Microservices**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|POST | /services/{id}/dependent-service | | | create dependent microservice of a given service id|
|PATCH | /services/{id}/dependent-service | | | update dependent microservice of a given service id|
		
### **For every Landing Zone**
#### **Add product enclave (WAF / Gateway / LB / VPC/ Few Clusters )**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|POST | /landing-zone/{landingZone}/product-enclave | | | create product enclave under a landing zone|

#### **Add cluster in product enclave**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|POST | /landing-zone/{landingZone}/product-enclave/{productEnclave}/cluster | | | create a cluster under a product enclave|

#### **Add WAF/Gateway/LB/product enclave / Cluster for a particular product env**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|POST | /landing-zone/{landingZone}/products/{product}/environments/{env}/cloud-element/{cloudElementId} | | | create a hardware location and hosted services of a cloud-element |
|PATCH | /landing-zone/{landingZone}/products/{product}/environments/{env}/cloud-element/{cloudElementId} | | | update a hardware location and hosted services of a cloud-element |	
|POST | /landing-zone/{landingZone}/products/{product}/environments/{env}/services/{id} | | | create a topology of a service |
|PATCH | /landing-zone/{landingZone}/products/{product}/environments/{env}/services/{id} | | | update a topology of a service |

#### **Add product(specifying its env) in product enclave**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|POST | /landing-zone/{landingZone}/environments/{env}/product-enclave/{productEnclave}/products/{product} | | | create a product in a given landing zone and product-enclave |
		
#### **Add Business Service in a product**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|POST | /landing-zone/{landingZone}/products/{product}/services/{serviceNature} | | | create a business service in a given landing zone and product |

#### **Add Common Service in a product**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|POST | /landing-zone/{landingZone}/products/{product}/services/{serviceNature} | | | create a common service in a given landing zone and product |

#### **Run Audits on Product**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|GET | /landing-zone/{landingZone}/products/{product}/audit | | | run audit on a given landing zone and product |

#### **Run Audits on Landing Zone**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|GET | /landing-zone/{landingZone}/audit | | | run audit on a given landing zone |

#### **Run Audits on Product Enclave**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|GET | /landing-zone/{landingZone}/product-enclave/{productEnclave}/audit | | | run audit on a given landing zone and product enclave |

#### **Run Audits on Cluster**
| method | end point | Request | Response | Description | 
|:---|:---|:---|:---|:---|
|GET | /landing-zone/{landingZone}/cluster/{clusterId}/audit | | | run audit on a given landing zone and cluster |
