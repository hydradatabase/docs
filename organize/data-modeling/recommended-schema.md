# Recommended Schema

## Star Schema

Star schema is a logical arrangement of tables in a multidimensional database such that the entity relationship diagram resembles a star shape. It is a fundamental approach that is widely used to develop or build a data warehouse. It requires classifying model tables as either dimension or fact.

Fact tables record measurements or metrics for a specific event. A fact table contains dimension key columns that relate to dimension tables and numeric measure columns. Dimension tables describe business entities - the things you model. Entities can include products, people, places, and concepts including time itself. A dimension table contains a key column (or columns) that acts as a unique identifier and other descriptive columns. Fact tables are best as columnar tables, while dimension tables may be best as row tables due to their size and rate of updates.

The following diagram shows the star schema that we are going to model for the tutorial:

![https://hydras.io/assets/blog/2022-10-19/star-schema-6902bc4b4b608aad3844912e49028d73a2a99e4d629fbe0cf018f68b60d9a37d.png](https://hydras.io/assets/blog/2022-10-19/star-schema-6902bc4b4b608aad3844912e49028d73a2a99e4d629fbe0cf018f68b60d9a37d.png)

****

## Hydra Schema

A Hydra schema is conceptually identical to star schema with Fact tables record measurements / metics and Dimension tables are the entities that are modeled, such as people and places. The key difference of the Hydra schema is dimension tables may or may not be local within the data warehouse and can exist as foreign tables. Hydra schemas allow for rapidly updated data sets to avoid data duplication in the warehouse, but available for analysis instantly. To optimize for performance, foreign dimension tables will be no more than (XYZ# of rows).

##
