# Postgres Query Optimization + Diagnostic Tooling



PostgreSQL is a database management system that utilizes SQL and includes features like foreign keys, updatable views, and various query optimization tools. This article will outline some of the query optimization techniques in Postgres using diagnostic tools.
<br />
## Scope:
Query optimization involves identifying the most efficient way to execute a query, considering the structure of the database and available indexes. The Postgres query optimizer can perform this task. Proper optimization techniques can help reduce costs and enhance environmental sustainability by using resources more efficiently.
Postgres Query optimiser:
The PostgreSQL optimizer generates a plan for each query it receives, and the EXPLAIN command allows one to view these plans. The EXPLAIN ANALYZE command can be used to evaluate the accuracy of these plans.
 &nbsp; 
 > For Example:
 
 &nbsp;&nbsp; `` EXPLAIN ANALYZE SELECT * FROM customer; ``
 
  > Execution Plan of this Query is: <br>
&nbsp;&nbsp;![](.gitbook/assets/images/Picture1.png)
 
## Postgres Query optimization and performance tuning methods:
In this section, you will learn a few optimization techniques along with examples and query execution plans.
### Use of Indexes:
An index serves as an additional access structure that enables the rapid location and retrieval of specific data. In the absence of an index, a query would need to search the entire table to find the desired information. By indexing columns commonly used in WHERE or JOIN clauses, PostgreSQL can swiftly locate the needed data.

For instance, the following picture depicts the query execution plan without any indexes.

&nbsp;&nbsp; ![](.gitbook/assets/images/Picture2.png)

<br>The execution plan displayed in this image depicts the impact of adding an index on the "address_id" column. The addition of the index has resulted in a noticeable improvement in both Planning Time and Execution Time.

&nbsp;&nbsp; ![](.gitbook/assets/images/Picture3.png)

### Optimize SELECT Clause:
Efficiency issues in queries can be addressed by using specific column names instead of the ‘*’ argument in SELECT statements. Additionally, it is important to execute the HAVING clause after filtering the data with the SELECT statement, as SELECT acts as a filter. To illustrate, replacing ‘*’ with Payment_id in Group by can improve the query performance. You can see the change in execution plans in both the queries.
 &nbsp;&nbsp;   ![](.gitbook/assets/images/Picture4.png)   
 <br>
 &nbsp;&nbsp; ![](.gitbook/assets/images/Picture5.png)

