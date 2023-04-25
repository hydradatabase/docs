# Postgres Query Optimization + Diagnostic Tooling



PostgreSQL is a database management system that utilizes SQL and includes features like foreign keys, updatable views, and various query optimization tools. This article will outline some of the query optimization techniques in Postgres using diagnostic tools.
<br />
## Scope
Query optimization involves identifying the most efficient way to execute a query, considering the structure of the database and available indexes. The Postgres query optimizer can perform this task. Proper optimization techniques can help reduce costs and enhance environmental sustainability by using resources more efficiently.
Postgres Query optimiser:
The PostgreSQL optimizer generates a plan for each query it receives, and the EXPLAIN command allows one to view these plans. The EXPLAIN ANALYZE command can be used to evaluate the accuracy of these plans.
 &nbsp; 
 > For Example:
 
 &nbsp;&nbsp; `` EXPLAIN ANALYZE SELECT * FROM customer; ``
 
  > Execution Plan of this Query is: <br>
&nbsp;&nbsp;![](/.gitbook/assets/images/Picture1.png)
 
## Postgres Query optimization and performance tuning methods
In this section, you will learn a few optimization techniques along with examples and query execution plans.
### Use of Indexes
An index serves as an additional access structure that enables the rapid location and retrieval of specific data. In the absence of an index, a query would need to search the entire table to find the desired information. By indexing columns commonly used in WHERE or JOIN clauses, PostgreSQL can swiftly locate the needed data.

For instance, the following picture depicts the query execution plan without any indexes.

&nbsp;&nbsp; ![](/.gitbook/assets/images/Picture2.png)

<br>The execution plan displayed in this image depicts the impact of adding an index on the "address_id" column. The addition of the index has resulted in a noticeable improvement in both Planning Time and Execution Time.

&nbsp;&nbsp; ![](/.gitbook/assets/images/Picture3.png)

### Optimize SELECT Clause
Efficiency issues in queries can be addressed by using specific column names instead of the ‘*’ argument in SELECT statements. Additionally, it is important to execute the HAVING clause after filtering the data with the SELECT statement, as SELECT acts as a filter. To illustrate, replacing ‘*’ with Payment_id in Group by can improve the query performance. You can see the change in execution plans in both the queries. <br><br>
 &nbsp;&nbsp;   ![](/.gitbook/assets/images/Picture4.png)   
 <br>
 
 &nbsp;&nbsp; ![](/.gitbook/assets/images/Picture5.png) <br>
 The performance difference is not significant, but according to the sources reviewed in the literature study, modifying the use of * is expected to enhance performance.
 
 ### Filtering Rows with Subqueries and Indexes:
 Filtering data before performing complex operations like GROUP BY or JOIN can enhance query performance by minimizing the amount of data that needs to be processed. Utilizing subqueries or other techniques to filter data is often effective in reducing query execution time and improving database efficiency. <br><br>
&nbsp;&nbsp;   ![](/.gitbook/assets/images/Picture6.png)   
 <br>
 
 &nbsp;&nbsp; ![](/.gitbook/assets/images/Picture7.png)
 ### WHERE Clause:
 To optimize the WHERE Clause, we can use OR instead of IN when there is no index on the filtering column. The IN statement matches a column value to a list of values, and technically, it should execute faster than OR. However, in some cases, using OR instead of IN may not improve performance significantly, but it is a way to test if the query runs faster. <br><br>
 &nbsp;&nbsp;   ![](/.gitbook/assets/images/Picture8.png)   
 <br>
 
 &nbsp;&nbsp; ![](/.gitbook/assets/images/Picture9.png)
 
### Optimize Joins:
To enhance query performance, optimizing joins is essential as they are expensive operations. Substituting joins with subqueries whenever possible is one way to optimize them. Additionally, using the JOIN ON syntax instead of the WHERE clause can help the optimizer produce better execution plans. Filtering records of large tables before joining them with other tables can also lead to substantial performance improvements.

These are some additional factors that should be taken into consideration when designing a Postgres query for improved performance.  
*	Temporary tables can slow down execution but can also avoid the need for ORDER BY operations.  
*	Some operations may prevent the query from using indexes, so it's important to understand these peculiarities.  
*	The order of tables in the FROM statement can affect JOIN ordering, particularly when joining more than five tables.  
*	Index-only scans are faster than full table scans, but index access can be slower when query selectivity is high.  
*	Views can result in inefficient queries.  
*	Use DISTINCT only when necessary.  
*	Minimize the use of subqueries, especially correlated subqueries.  
*	Long queries are not helped by indexes, but instead can be optimized by minimizing the number of full table scans and reducing the result size as soon as possible.  
*	Materialized views are useful for speeding up execution time if having fully up-to-date data is not a priority for the query.  

## Conclusion:
In conclusion, optimizing Postgres queries is crucial for improving performance and reducing costs. Using diagnostic tools like EXPLAIN and EXPLAIN ANALYZE can help identify potential issues. Techniques such as using indexes, filtering columns, and optimizing joins can significantly improve 