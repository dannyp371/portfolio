# OData Quick Reference Guide

The OData protocol is a data access protocol built on core protocols like HTTP and commonly accepted methodologies like REST for the web. It allows you to query databases over the web and supports several formats. This guide will cover the basics of OData's URL querying convention; however, full documentation for OData can be [found here](odata.org/documentation). 

## URL String Query

OData uses URL strings to query your database. Each query has 3 major parts, the `service root URL`<sup>1</sup>, `resource path`<sup>2</sup>, and `query options`<sup>3</sup>. An example is shown below:

http://host:port/path/SampleService.svc<sup>1</sup>/Categories(1)/Products?<sup>2</sup>$top=2&$orderby=Name<sup>3</sup>

- **`Service root URL:`**<sup>1</sup> Identifies the root of an OData service. 
- **`Resource path:`**<sup>2</sup> The type of CRM record you want to access. Typically, this will refer to a table in your database.
- **`Query options:`**<sup>3</sup> The functions which define your query.

## Query Options

Since the `service root URL` and the `resource path` are dependent on your specific service and database, the rest of this document will cover options for querying your service and database via OData. There are several query options:

- **$select**: Allows you to stratify the data returned by selecting specific fields. 
- **$filter**: Allows you to stratify the data returned by selecting a field as well as an [operator](docs.oasis-open.org/odata/odata/v4.01/odata-v4.01-part2-url-conventions.html#_Toc31360957) and a value.
- **$expand**: Allows you to connect your `resource path` to futher fields. Works similarly to a `join` in SQL.
- **$orderby**: Sorts the data returned by a field name and direction.
- **$top and $skip**: Both are set equal to a numeric value, `X`. $top will return the top `X` values, and $skip will return all values except for the top `X` values.
- **$count**: A Boolean value that, when true, will count the number of matching resources in the response.
- **$search**: Returns data which matches a specified free-text string value.
- **$format**: Changes the format of the returned data. Options include, XML, JSON, and more.
- **$compute**: Allows you to create an arithmetic statement based on columns within the table you are referencing. You can create an arithmetic statement with the [arithmetic operators](docs.oasis-open.org/odata/odata/v4.01/odata-v4.01-part2-url-conventions.html#Toc31360970) in the OData documentation.
- **$index**: Allows clients to a perform a positional insert into a collection.
