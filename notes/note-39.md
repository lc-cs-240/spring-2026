# Note 39

## Prepared statements

Below is a quote from Wikipedia's entry about [prepared
statements](https://en.wikipedia.org/wiki/Prepared_statement):

A *prepared statement* is a feature where the database pre-compiles SQL code and
stores the results, separating it from data.

Benefits of prepared statements are:

* Efficiency, because they can be used repeatedly without re-compiling
* Security, by reducing or eliminating SQL injection attacks

## Activity

Convert one of your PHP queries to use prepared statements. You may find the
reference below useful.

## Reference

* https://www.php.net/manual/en/mysqli-stmt.prepare.php
