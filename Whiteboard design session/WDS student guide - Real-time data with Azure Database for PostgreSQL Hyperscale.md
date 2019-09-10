![Microsoft Cloud Workshops](https://github.com/Microsoft/MCW-Template-Cloud-Workshop/raw/master/Media/ms-cloud-workshop.png 'Microsoft Cloud Workshops')

<div class="MCWHeader1">
Real-time data with Azure Database for PostgreSQL Hyperscale
</div>

<div class="MCWHeader2">
Whiteboard design session student guide
</div>

<div class="MCWHeader3">
September 2019
</div>

Information in this document, including URL and other Internet Web site references, is subject to change without notice. Unless otherwise noted, the example companies, organizations, products, domain names, e-mail addresses, logos, people, places, and events depicted herein are fictitious, and no association with any real company, organization, product, domain name, e-mail address, logo, person, place or event is intended or should be inferred. Complying with all applicable copyright laws is the responsibility of the user. Without limiting the rights under copyright, no part of this document may be reproduced, stored in or introduced into a retrieval system, or transmitted in any form or by any means (electronic, mechanical, photocopying, recording, or otherwise), or for any purpose, without the express written permission of Microsoft Corporation.

Microsoft may have patents, patent applications, trademarks, copyrights, or other intellectual property rights covering subject matter in this document. Except as expressly provided in any written license agreement from Microsoft, the furnishing of this document does not give you any license to these patents, trademarks, copyrights, or other intellectual property.

The names of manufacturers, products, or URLs are provided for informational purposes only and Microsoft makes no representations and warranties, either expressed, implied, or statutory, regarding these manufacturers or the use of the products with any Microsoft technologies. The inclusion of a manufacturer or product does not imply endorsement of Microsoft of the manufacturer or product. Links may be provided to third party sites. Such sites are not under the control of Microsoft and Microsoft is not responsible for the contents of any linked site or any link contained in a linked site, or any changes or updates to such sites. Microsoft is not responsible for webcasting or any other form of transmission received from any linked site. Microsoft is providing these links to you only as a convenience, and the inclusion of any link does not imply endorsement of Microsoft of the site or the products contained therein.

© 2019 Microsoft Corporation. All rights reserved.

Microsoft and the trademarks listed at <https://www.microsoft.com/en-us/legal/intellectualproperty/Trademarks/Usage/General.aspx> are trademarks of the Microsoft group of companies. All other trademarks are property of their respective owners.

**Contents**

<!-- TOC -->

- [Real-time data with Azure Database for PostgreSQL Hyperscale whiteboard design session student guide](#real-time-data-with-azure-database-for-postgresql-hyperscale-whiteboard-design-session-student-guide)
  - [Abstract and learning objectives](#abstract-and-learning-objectives)
  - [Step 1: Review the customer case study](#step-1-review-the-customer-case-study)
    - [Customer situation](#customer-situation)
    - [Customer needs](#customer-needs)
    - [Customer objections](#customer-objections)
    - [Infographic for common scenarios](#infographic-for-common-scenarios)
  - [Step 2: Design a proof of concept solution](#step-2-design-a-proof-of-concept-solution)
  - [Step 3: Present the solution](#step-3-present-the-solution)
  - [Wrap-up](#wrap-up)
  - [Additional references](#additional-references)

<!-- /TOC -->

# Real-time data with Azure Database for PostgreSQL Hyperscale whiteboard design session student guide

## Abstract and learning objectives

Wide World Importers has a host of online stores for various product offerings, including traditional product catalogs offered by their physical storefronts, to specialized categories like automotive and consumer technology products. This expansion has made it more challenging to analyze user clickstream data, online ad performance, and other marketing campaigns at scale, and to provide insights to the marketing team in real-time. Today they store and analyze user clickstream data, online ad performance, and other marketing campaigns to evaluate marketing effectiveness and customer reach.

At the end of this whiteboard design session, you will be better able to use advanced features of the managed PostgreSQL PaaS service on Azure to make your database more scalable and able to handle the rapid ingest of streaming data while simultaneously generating and serving pre-aggregated data for reports. You will design a resilient stream processing pipeline to ingest, process, and save real-time data and provide guidance on how to create complex reports containing advanced visualizations and use them to build a customizable dashboard.

## Step 1: Review the customer case study

**Outcome**

Analyze your customer's needs.

Timeframe: 15 minutes

Directions: With all participants in the session, the facilitator/SME presents an overview of the customer case study along with technical tips.

1.  Meet your table participants and trainer.

2.  Read all of the directions for steps 1-3 in the student guide.

3.  As a table team, review the following customer case study.

### Customer situation

Wide World Importers (WWI) is a traditional brick and mortar business with a long track record of success, generating profits through strong retail store sales of their unique offering of affordable products from around the world. They have a great training program for new employees, that focuses on connecting with their customers and providing great face-to-face customer service. This strong focus on customer relationships has helped set WWI apart from their competitors.

Over time, WWI modernized their business by expanding to online storefronts. During this expansion period, WWI experimented with various marketing tactics to drive online sales, from offering in-store shoppers special discounts online with promotional coupons after making a purchase, to running ad campaigns targeted toward customers based on demographics and shopping habits. These marketing campaigns proved successful, prompting WWI to invest more resources to these efforts and grow their marketing team.

Today, WWI has a host of online stores for various product offerings, from their traditional product catalogs offered by their physical storefronts, to specialized categories like automotive and consumer technology products. This expansion has made it more challenging to analyze user clickstream data, online ad performance, and other marketing campaigns at scale, and to provide insights to the marketing team in real-time.

Real-time marketing analysis is provided through interactive reports and dashboards on WWI's home-grown web platform, ReMarketable. This platform has served them well, but they are currently hindered by their inability to keep up with demand. ReMarketable's primary users are members of the marketing team, and the secondary users are shoppers on their various online platforms for whom website interaction behavior is being tracked. Other sources of data are fed from online ad data generated by ads run on social media platforms and email marketing campaigns. They use this type of data to evaluate ad effectiveness and customer reach, ultimately leading to sales conversions.

Their current challenges with ReMarketable are:

1. **Scale** - WWI is using a PostgreSQL database to store ReMarketable's data. Historical data is growing by over 2.9 GB rows of data per month. It is taking consistently longer to run complex queries. Queries that used to run in 3-5 seconds now take several minutes to complete. This is impacting their users' ability to evaluate up-to-date marketing and website use statistics. Instead of providing real-time reports for all users, they have to keep delaying report runs. They have scaled up their database, but this is becoming very expensive and they will soon hit a ceiling.

2. **Multi-tenancy** - The nature of their marketing and site usage data would benefit from multi-tenancy. Some storefronts generate considerably more data than others and have more marketing analysts that run reports on them than others. WWI believes sharding their database would help take the pressure off lower-volume data stores and also help them scale out. However, this will require re-engineering their database schema and client applications. In addition, sharding will require additional maintenance and increased complexity of aggregated views. These additional challenges and required resources are why they have not pursued this option yet.

3. **Process data while generating roll-ups** - Another byproduct of outgrowing their database is that WWI is having difficulty efficiently processing and ingesting streaming data, while at the same time generating pre-aggregated data for their dashboards. The Postgres engine is well-suited to handle multiple workloads simultaneously when the databases are properly configured, and you are able to appropriately scale up or scale out to multiple nodes. WWI does needs help optimizing their database to handle these demanding workloads at scale. They have looked moving to a non-relational database to speed up queries, but that option added too much complexity to manage multiple databases, losing the ability to wrap their operations inside of transactions, re-architect their application, and migrate their historical data. In addition, they rely on Postgres' ability to create complicated ways of representing and indexing their data, which is impossible to do with a column store. Their need for high transaction volume and a real-time data set ruled out a lot of off-the-shelf data warehouses, where they would need to create a lambda architecture to handle both speeds of feeds.

4. **Resilient stream processing** - WWI is processing their streaming data through a web-based cluster that balances HTTP requests in round-robin fashion. When a node is processing the data and writing it to Postgres, subsequent requests are handled by available nodes. However, if processing fails for any reason, they risk losing that data and have no way to pick up where it left off. They have tried creating their own poison queue to reprocess these failed messages, but if the failed node is unable to add the data to the queue, then it is lost. The WWI technical team is aware of existing products and services that can help improve their stream processing and add resiliency, but they currently lack the skills and bandwidth to implement a solution for these complex scenarios. They are interested to see how Azure can help them rapidly create a solution for resilient stream processing and reduce their technical debt.

5. **Advanced dashboards** - WWI creates canned reports that are displayed on their ReMarketable website. However, their developers spend a lot of time creating new reports, owing to advanced charts, graphs, and other visualizations that are usually included. They would like a way to more rapidly create reports and be able to display them on a dashboard that can be customized and show real-time updates.

### Customer needs

1. Scale our marketing PostgreSQL database to handle high data growth while reducing the amount of time to run complex queries.

2. Shard our database so we can scale out, based on our tenants and their load requirements. We need a way to do this that will reduce schema changes, maintenance, and complexity of aggregated views.

3. Need a way to efficiently ingest and process streaming data, while at the same time generating pre-aggregated data for our dashboards.

4. During our stream processing, we sometimes encounter errors and have a difficult time recovering and continuing where we left off. We need a more resilient stream processing solution to reduce errors and prevent lost data.

5. Would like a simple way to create powerful reports with a variety of visualizations. Ideally, this is something our analysts should be able to do against our live data sets.

### Customer objections

1. Does Azure offer a managed PostgreSQL database that can handle our scale requirements?

2. We are worried about the re-engineering effort involved in sharding our database, from modifying the schema to updating our applications to account for the changes.

3. Is there a way to migrate to PostgreSQL on Azure with minimal downtime?

4. We've looked at several PostgreSQL-based data platforms for adding enhancements like distributed data and scalability, but we are concerned about our existing applications being compatible and having access to the latest versions of PostgreSQL.

### Infographic for common scenarios

![Infographic for common scenarios that you can use for inspiration.](media/common-scenarios.png 'Infographic for common scenarios')

## Step 2: Design a proof of concept solution

**Outcome**

Design a solution and prepare to present the solution to the target customer audience in a 15-minute chalk-talk format.

Timeframe: 60 minutes

**Business needs**

Directions: With all participants at your table, answer the following questions and list the answers on a flip chart:

1.  Who should you present this solution to? Who is your target customer audience? Who are the decision makers?

2.  What customer business needs do you need to address with your solution?

**Design**

Directions: With all participants at your table, respond to the following questions on a flip chart:

_High-level architecture_

1. Without getting into the details (the following sections will address the particular details), diagram your initial vision for handling the top-level requirements for creating a real-time data processing pipeline that can ingest, process, and write streaming data to a highly scalable managed PostgreSQL database on Azure. The stream processor needs to be resilient by keeping track of where it left off and prevent lost data. Include a solution for creating advanced visualizations that can be shared or embedded in external websites and mobile devices.

_Scale_

1. How will you configure the managed PostgreSQL database so it can be scaled to meet demand? Think about options for scaling up and for scaling out.

2. WWI wants to be able to scale their database out, but they've already expressed concerns about how sharding tables to accomplish this adds a lot of complexity and maintenance overhead, as well as required code changes. How would you propose they shard tables that need to have data distributed amongst the nodes?

3. How can the clickstream time series event data be partitioned into 5-minute increments to avoid creating large indexes?

_Multi-tenancy_

1. The clickstream event data is multi-tenant by nature. Each tenant is denoted by a Tenant ID which is related to the source of the clickstream feed. How can WWI shard the raw event table by tenant across multiple nodes in a way that causes little impact to existing applications and maintenance overhead?

2. WWI has expressed a desire to create rollup tables that contain pre-aggregated data for efficient reporting. These rollup tables should also be sharded by Tenant ID. How can WWI shard these tables as well and what data type can be used to rapidly obtain distinct counts within a small margin of error in a highly scalable way across partitions?

_Process data while generating roll-ups_

1. Rollup tables enable faster queries for reporting and exploration, but oftentimes require compute-heavy work to periodically run in the background to populate these tables. How would you schedule these aggregates to run on a periodic basis?

2. Within your rollup functions that perform the background aggregations, how would you implement incremental aggregations to handle late, incoming, data while keeping track of which time periods have been aggregated already?

3. Incremental aggregations sacrifice the ability to handle all types of aggregates for ease of use when tracking what has already been aggregated when processing late data. What advanced aggregation options can you use to provide highly accurate approximation in this situation?

_Resilient stream processing_

1. WWI is currently using a Kafka cluster to ingest a high volume of streaming data from various clickstream sources across their tenants. Is there a managed option in Azure that supports Kafka?

2. WWI sometimes encounters errors while stream processing, and has a difficult time recovering and continuing where they left off if the stream has to stop for any reason. What would you recommend for a resilient stream processing solution to reduce errors and prevent lost data?

3. Could your chosen stream processing solution also provide the ability to conduct batch processing against large amounts of data while sharing much of the data processing and cleansing code, as well as code to write data to PostgreSQL?

_Advanced dashboards_

1. What would you recommend Wide World Importers use to create reports and dashboards with advanced visualizations, that can be created with an intuitive visual interface, easily shared with others, embedded in external websites or mobile devices?

2. How can WWI refresh the report's data on a regular basis and provide redundancy in their synchronization process? Does this synchronization process require any inbound ports to be opened up on the computer or servers on which it runs?

**Prepare**

Directions: With all participants at your table:

1.  Identify any customer needs that are not addressed with the proposed solution.

2.  Identify the benefits of your solution.

3.  Determine how you will respond to the customer's objections.

Prepare a 15-minute chalk-talk style presentation to the customer.

## Step 3: Present the solution

**Outcome**

Present a solution to the target customer audience in a 15-minute chalk-talk format.

Timeframe: 30 minutes

**Presentation**

Directions:

1.  Pair with another table.

2.  One table is the Microsoft team and the other table is the customer.

3.  The Microsoft team presents their proposed solution to the customer.

4.  The customer makes one of the objections from the list of objections.

5.  The Microsoft team responds to the objection.

6.  The customer team gives feedback to the Microsoft team.

7.  Tables switch roles and repeat Steps 2-6.

## Wrap-up

Timeframe: 15 minutes

Directions: Tables reconvene with the larger group to hear the facilitator/SME share the preferred solution for the case study.

## Additional references

|                                                         |                                                                                                    |
| ------------------------------------------------------- | :------------------------------------------------------------------------------------------------: |
| **Description**                                         |                                             **Links**                                              |
| Azure PostgreSQL documentation                          |                        <https://docs.microsoft.com/en-us/azure/postgresql/>                        |
| PostgreSQL time series data processing                  |  <https://docs.microsoft.com/en-us/azure/postgresql/tutorial-design-database-hyperscale-realtime>  |
| PostgreSQL distribution columns and multi-tenant apps   | <https://docs.microsoft.com/en-us/azure/postgresql/concepts-hyperscale-choose-distribution-column> |
| PostgreSQL HyperLogLog extension                        |                           <https://github.com/citusdata/postgresql-hll>                            |
| Azure Databricks documentation                          |         <https://docs.microsoft.com/en-us/azure/azure-databricks/what-is-azure-databricks>         |
| Azure Databricks Structured Streaming                   |          <https://docs.azuredatabricks.net/spark/latest/structured-streaming/index.html>           |
| Event Hubs for Apache Kafka                             |    <https://docs.microsoft.com/en-us/azure/event-hubs/event-hubs-for-kafka-ecosystem-overview>     |
| Power BI documentation                                  |                            <https://docs.microsoft.com/en-us/power-bi/>                            |
| High availability clusters for On-premises data gateway |       <https://docs.microsoft.com/en-us/power-bi/service-gateway-high-availability-clusters>       |
| Azure Database Migration Service                        |                                      <https://aka.ms/get-dms>                                      |
