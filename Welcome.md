What is Data Warehousing?
- Technology that allow us to gather, store and present data in form suitable for human exploration.
- Multiple data formats at different locations.
- Data warehouse is subject oriented,integrated,time-variant and non volatile collection of data to support the decision making process of an enterprise

DBMS VS DATA WAREHOUSE
dbms:
supports day to day operations
transaction update is cruicial
heavy concurrency 
Normalized design is essential
each transaction only access small amount of data
focus on present
use of atomic data
changin incomplete data

Data warehouse
supports information analysis
mostly retreivals less updates
light concurrency
normalised design is appropriate
most analysis targets large amounts of data
focus on past,present and future
use of aggregate data
static historic data

ROLAP vs MOLAP:
Enhanced to handle dbms like stuff
molap:
uses different storage mechanism optimised for data warehousing


ARCHITECTURE OF DATA WAREHOUSE:
Data Sources: Repository of data that is integrated from multiple data sources, which may be present in different geographical locations.

ETL(Extract Transform Load):
To ensure periodic extraction of fresh data fromthe data sources.
Data cleaning is part of extraction, Transformed to match the schema and format of data warehouse.
Transformed Data loaded into the data warehouse.

Metadata Repository:
Data warehouse administrator manages the entire system by using these tools.
Contains schema of warehouse and administrative tools to monitor /control various aspects of data warehouse.

Data Marts:
Subset of what would be in a data warehouse of an enterprise.
Consists of data related to single department
Implemented because they are much cheaper than building an entire data warehouse
Proof of concept

Physical Storage:
Due to the huge amount of storage ,may be distributed into multiple servers.
Metadata is replicated on each server

Olap Server:
The servers require OLAP queries or data mining requests from front end tools and process them by accessing data in physical locations.
Architect decides of number of processing units/servers.

Front-End Tools:
Interface between analyst and the data warehouse. Analyst should be able to pose OLAP queries  or data mining tasks and visualise the reports

What is multidimensional model:
Each dimension of array responds to dimension of warehouse and values stored in each cell correspond to measures of warehouse

Drilling Down:While seeing a particular view, analyst may be interested in seeing other neighbour views, more details and may request for a more detailed view
Rolling Up:
Owner may be interested in further aggregating along some dimension.

Data Warehousing Data Characteristics:
1) Subject Oriented Data:
	1) Data is organised according to subject instead of application

2) Integrated Data:
	1) data warehouse is usually constructed by integrating multiple hetrogenous sources such as relational databases ,flat files and oltp files.
	2) can happen in various dimensions like consistent naming conventions ,consistent measurement of variables,consistent encoding structures,consistent physical attributes of data. 
	3) In data warehousing ,integration is done at data staging level.
3) Time-variate data:
	1) data is stored in data warehouse to provide historical perspective . Every key structure in data warehouse contains, implicitly or explicitly an element of time.
	2) Data warehouse represents flow of data through time 
	3) Usually contains data that is 5-10 years old
4) NON Volatile datA
	1) data that has been written into data warehouse cannot be overwritten but extended. Thus data are not updated or edited in any way once they enter the data warehouse, but are freshly loaded,refreshed and accedessed for queries.


In depth data warehouse components
1) Data Source
	1) divided into 4 broad categories
	2) Production data :
	3) Internal data: user keep their private spreadsheet,documents ,customer profiles and sometimes even departmental databases. This is internal data parts of which could be useful in data warehouse
	4) Archived data: Operational systems are primarily intended to run the current business. In every operational system, old data is periodically taken and stored in archive files.
	5) External data: Most executives depend on data from external sources for high percentage of information they use.
2) Physical data sources
3) front end tools
4) Metadata repository and administrative tools

What is data mart?
logical subset of data warehouse which is highly focused set of information that is designed in the same way as data warehouse but implemented to address specific need of a defined set of users.

Into 2 categories:
a)subset data mart-top down : Enterprise of huge data warehouse is constructed and populated.Subset data marts are then created taking part of enterprice data warehouse to serve specific user groups of homogenous characteristicserprise data warehouse.
b)Incremental data mart - bottom up- Uses incremental data marts as building blocks of enterprise data warehouse.Individual incremental data marts are created and deployed.

What is extraction and what is full extraction vs incremental extraction

Data Transformation:
process checks data for reliability,consistency and validity
What are the three main approaches to load the data warehouse.
Initial Load-Loading all data for first time
Incremental Load-Loading incremental part of data from last time load  periodically
Full refresh - Deleting contents completely and loading afresh with all required data

Logical data modeling
First step towards building the warehouse. Dimensionlity modeling is logical modeling design technique used for data warehouse

- **Purpose and Usage**
    - **ER Modeling**
        - Designed for OLTP (Online Transaction Processing) systems
        - Focuses on operational database design
        - Optimized for data insertion, updating, and maintaining data integrity
    - **Dimensional Modeling**
        - Designed for OLAP (Online Analytical Processing) systems
        - Focuses on data warehouse design
        - Optimized for complex queries and data analysis
- **Structure**
    - **ER Modeling**
        - Uses entities and relationships
        - Highly normalized (typically 3NF)
        - Complex structure with many tables
    - **Dimensional Modeling**
        - Uses facts and dimensions
        - Deliberately denormalized
        - Simpler structure (star or snowflake schema)
- **Key Components**
    - **ER Modeling**
        - Entities
        - Attributes
        - Relationships
        - Cardinality
    - **Dimensional Modeling**
        - Fact tables (contain measures)
        - Dimension tables (contain descriptive attributes)
        - Star schema or Snowflake schema

Dimensional model basics:
Conceptually represented using schemas.
	-A fact is focus of interest about particular subject. Facts are used to store business information on which detailed analysis is carried out
	-Measures are continuos value(mostly numerical) - describe fact drom different point of view
	- Dimensions determine contextual background of facts
	- Granularity - related to fact table it concerns about what detail level the fact table has.
		- represents lowest information that will be stored in fact table
		- Based on 1) which dimensions will be included
		- The heirarchy of each dimension the information will be kept
	- Surrogate Key
		- keys generated from source system are called natural keys
		- keys generated from data warehouse is called surrogate keys
		- keys themselves have no meaning
		- acts as production key and very useful in warehouse design
		- Dimension tables and fact tables are joined based on surrogate keys in data warehouse environment.
		- Hence during extraction generated systematically in place of incoming database keys.
		- Added for each record when loaded into data warehouse

There are two types of tables in schema representation:
1) Dimensional Table
2) Fact Table

Dimension table contains details pertaining to the dimension, relatively static over time.
smaller in size 
Dimension table is usually smaller than fact table .It has primary key and other attributes which provide details about the dimension and also useful for querying.
Fact Table: Contains transactional type information. Data in fact table changes over time.
Foreign key in table for each of the dimension tables.
Key containing facts in fact table are measures.
-keys
-measures
-own attributes

schemas
star schema,
snowflake schema
star schema

dimensional modeling
1)fully additive measures-some measures can be aggregated correctly across all dimensions.Ex no of accounts
2) semi additive :additive across some dimensions.ex:account balance-aggregate over account but not 
3) Non additive:
4) average balance:


Fact less tables

Dimension table characteristics:
1) Primary key: Generally numerical.Key values from source system cannot be used as keys in data warehouse.
2) Heirarchy fields:
		for each level of dimension heirarchy dimension tables have field
3) Attribute fields
	1) additional information regarding members at one of the levels of the dimension's heirarchy

How to handle changes in dimesion tables
	1)Original data is replaced by new data
2) New data is added into dimensional table and 2 records are there for same information
3) Original table structure is modified to reflect change in the record

What is OLAP: 
Data warehouse is Repository where data is gathered from internal and external sources
Storing data in such a way that it supports efficient analytical processing
olap is standard data analysis technique for analysing large volumes of data .Transforms raw data to reflect real dimension of information.
Facilitates reporting,aggregating,trending ,summarising


Principal component is olap server :
sits between client and data warehouse storage.

What is OLAP Cube?
building block for olap analysis.
provides fastest answers for queries that aggregate large amounts of detail data
olap operations:
	slicing and dicing
	rollup and drill down
	drill within and drill acrooss
	pivot(rotate)


OLAP SERVER ARCHITECTURE
relational and multidimensional olap
rolap - Data is aggregated and stored in relational. DBMS called relational OLAP servers.Performs dynamic multidimensional analysis of data that is stored in relational database.
Molap: Data is stored in multi dimensional data bases. Special purpose servers that use non relational specialized storage stucture such as arrays.
	features are store and manage warehouse data
	array based storage structures
	direct access to array data structure

Features of olap tools
summarisation: Refers to degree of aggregation of information
	Feature is measure based on number of hierarchies,granularity of data and capability to swap between summarisation and detailed levels
Visualisation: Allows user ot create summary tables,charts and graphs interactively
	Feature depends on user's requirements , occurance of multidimensional tables, any kind of data that need to be carried out.
Navigation: Roll up and drill down are the most common operations in olap
	Capabilty of this feature can be accesed based on support for no of concurrent users
Dimensionality : It refers to the support for maximum number of dimensions, and the time for data refresh after re-definition
Query Processing and Performance: Refers to extracting data from multi dimensional databases by query engine and provides results in a more useful manner. Response time and query building capabilities are other factors that define performance.

Give question and answer style to answer these questions

Proove that four data warehouse characteristics provide better way of organising the data for decision support systems.
Explain the architecture of data warehouse and explain it's components.
Discuss various steps and approaches for data cleansing
explain star schema and snow flake schema
Wat are different storage design structures of a cube what are the difference between them?
Discuss various olap operations .Explain how query performance can be improved by cascading the operations.
Design data warehouse maily depends on queries for which you want to get answers.

What are shortcoming in er modeling for analytical application? How does dimension modeling concepts enhance er modeling
what is the effective way of modeling metadata in data warehouse
do comparitive study on various etl tools and olap tools
is it needed to have seperate data warehouse system in addition to the oltp system to analyse data
Design an efficient algorithm to store a star schema with n dimensional tables and fact table
study various cube storage for rolap,molap and holap servers and perform comparisions