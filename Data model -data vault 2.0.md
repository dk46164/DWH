
# DataWareHouse Design , Developement & Implementation

A DataWareHouse developement  goes through  broadly three steps: 

* Data Source modelling (Data Vault 2.0).
* Modelling Metadata architecture.
* Developing data pipeline (ELT/ETL).

We will go over each steps in more details & see  the best practices.










###  Data Source Modelling
There are variety of  data Modelling techniques that are practiced like star schema,
Snowflake schema, Data Vault .And for this project we are going to explore Data Vault 2.0
Since this has been widely used across industries. 

Key Components of Data Vault modeling are  **HUB**,**LINK**,**SATLELLITE** tables & the classification has been 
done on the fact that each data Source have two sets of attributes,     
***Business Key*** : Any attributes that has Business meaning e.g. Branch code it can used to point specify branch 
within Bank ,employee number  for given organisation we are having set of all employee code
,policy number for the Insurance company,it can be used to track which set of policy issued by them.    
***Non Business Key(descriptive keys)*** :Sets of attributes that holds descriptive information about
the Business keys e.g. from previous examples of Business keys ,let's start with 
Branch code ,For each Branch code  we can have branch address,customer's name  belonging to given branch,
Now pick employee number ,So for any employee id we have attached/have name,contact number,
Spouse Name ..... more ,let pick policy number for each policy we can have premium amount,
re-insurer name,contact number.....more list goes on.

So now we have clear understanding on types of attributes.We will try to use same logic to uderstand 
the significance of HUB,LINK,SATLELLITE tables.

First start with ****HUB****:

| HUB_EMPLOYEE | Description |
| --- | --- |
| employee_id(BK) | All the  unique employee id ,BK(Business Key)|
| employee_id(HK) | Hash value of employee_id(for hash calculation you can use md5,sha256) | 
| RECORD_SOURCE | As name suggest,Source of specific record  |
| LOAD_DATE | Load date of particular RECORD_SOURCE | 

This is the typical HUB table looks like & particularly two columns *LOAD_DATE*,*RECORD_SOURCE*
will be the part of any data vault table be it staging ,HUB,LINK,SATLELLITE,REF.These two columns
will help in data audit & data lineage process.

***LINK:***

  LNK_EMPLOYEE | Description |
| --- | --- |
| ANY_BUISNES_KEY_COLUMN(HK) | All the  unique employee id ,BK(Business Key)|
| employee_id(HK) | Hash value of employee_id(for hash calculation you can use md5,sha256) | 
| ANY_BUISNES_KEY_COLUMN(HK)+employee_id(HK) | Combined hash key of all the Business key | 
| RECORD_SOURCE | As name suggest,Source of specific record  |
| LOAD_DATE | Load date of particular RECORD_SOURCE | 

Because of lnk table we got feature unique to data vault 

**Business Rule Changes** – today the rule may be 1 to 1, in the future the business may dictate that in fact, 1 customer can be linked to more than 1 Invoice, or maybe 1 invoice can
be sent to M customers. When the business changes; this structure keeps the model clean and allows history to be preserved while the new rules are put in place.  

**Misplaced Data** – Too frequently in existing data modeling architectures there is data associated with tables that have composite keys – but doesn’t belong there. In
this modeling architecture it becomes easier to spot modeling errors and correct them – in other words put the data where it belongs, directly with the related key. 

**Granularity Control** – keeping the granularity of information together is important. Making sure that the information or data elements are placed where they belong, and that the
aggregations are controlled by the granularity of the key. Adding another key to a Link entity is very similar to adding another dimension to a fact table. It can change the granularity of the
satellites associated with the Link. For Links without Satellites it becomes very easy to make the changes.

Now moving to next table type **SAT**:

| SAT_LNK_EMPLOYEE | Description |
| --- | --- |
| LNK_HASH_KEY(HK) | All the  unique employee id ,HK(Business Key)|
| Non_BUSINESS_KEY_HASH(HD) | Combined hash key of all the Business key  | -->  
| Non_BUSINESS_KEY  | Descriptive attributes  | 
| RECORD_SOURCE | As name suggest,Source of specific record  |
| LOAD_DATE | Load date of particular RECORD_SOURCE | 

-->Don't Include business key while calculating HashDiff for SAT table  
** If SAT doesn’t have any lnk table then it will not have  LNK_HASH_KEY.So depending on the requirements we can have 
these many types:


| SAT TYPES | Description |  
| --- | --- |
| SAT  | NORMAL SAT TABLE WITH NO LNK TABLES|
| SAT_LNK | SAT TABLE WITH LNK TABLE,WHERE ALL TRANSACTIONS BETWEEN  BUSINESS KEY & NON BUSINESS KEY | 

Satellite table can hold sanpshot data /transactional data that can be valid over specific period of time,which
can be achieved by adding start date ,end_Date & many more functionality can be achieved.

Another  Table type that is also used frequently **REF** reference table,which is just like a lookup table.     
Let's understand with example post office.
In post office of have certain set of pincode on which it operates .So instead of going for HUB,LNK,SAT ,we can
store them in reference table whcih furthur can be used to implement data quality check like
if process get pincode which is not present in reference table ,then process should stop loading data from
furthur.            


For more Details ,Please checkout this course       
[https://www.udemy.com/course/modeling-data-warehouse-with-data-vault-for-beginners/]   
[https://www.analytics.today/blog/when-should-i-use-data-vault]

Please Ignore any typos error 😀.





