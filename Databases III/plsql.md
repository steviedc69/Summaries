# Introduction

## Declarations

```sql
DECLARE
    fam_birthdate	DATE;
    fam_size		NUMBER(2) NOT NULL := 10;
	fam_location	VARCHAR2(13) := 'Florida';
	fam_population	INTEGER;
	fam_name		VARCHAR2(20) DEFAULT 'Roberts';
	fam_party_size	CONSTANT PLS_INTEGER := 20;
```

### Using declared variables

```sql
DECLARE -- declarations
	v_myname	VARCHAR2(20);
BEGIN
	DBMS_OUTPUT.PUT_LINE('My name is: '||v_myname);
	v_myname := 'John';
	DBMS_OUTPUT.PUT_LINE('My name is: '||v_myname);
END;
```

### Data types

|Datatype|Explanation|
|----|-----|
|`CHAR`|fixed-length chardata, default length 1|
|`VARCHAR2(max_length)`|variable-length chardata|
|`LONG`|chardata longer than `VARCHAR2`|
|`LONG RAW`|raw binary data|
|`NUMBER(p,s)`|number with precision p and scale s|
|`BINARY_INTEGER`|signed integer|
|`PLS_INTEGER`|faster than `NUMBER`|
|`BINARY_FLOAT`<br>`BINARY_DOUBLE`|floating point numbers|
|`DATE`|stores a date|
|`TIMESTAMP`|extends `DATE`, holds up to fractions of seconds|
|`TIMESTAMP WITH TIME ZONE`|extends `TIMESTAMP`, includes timezone|
|`TIMESTAMP WITH LOCAL TIME ZONE`|extends `TIMESTAMP WITH TIME ZONE`, normalizes to database time zone|
|`INTERVAL YEAR TO MONTH`|stores intervals of years and months|
|`INTERVAL DAY TO SECOND`|stores intervals of days up to seconds|
|`BOOLEAN`|tri-state value: `TRUE`, `FALSE` or `NULL`|

## Useful keywords

|Keyword|Explanation|
|----|-----|
|`INTEGER`|alias for `NUMBER(38,0)`|
|`SYSDATE`|current date|
|`NOT NULL`|declares a variable that cannot be empty, needs to be initialized|
|`CONSTANT`|declares an unchgangeable variable, needs to be initialized?|
|`%TYPE`|can be used to dynamicaly get the datatype of a column: `table.col%TYPE`|

## Character functions

```
ASCII LENGTH RPAD
CHR LOWER RTRIM
CONCAT LPAD SUBSTR
INITCAP LTRIM TRIM
INSTR REPLACE UPPER
```

## Number functions

```
ABS EXP ROUND
ACOS LN SIGN
ASIN LOG SIN
ATAN MOD TAN
COS POWER TRUNC
```

## Date functions

```
ADD_MONTHS MONTHS_BETWEEN
CURRENT_DATE ROUND
CURRENT_TIMESTAMP SYSDATE
LAST_DAY TRUNC
```

## Implicit conversions 

(Implicit means no explicit conversion needed)

|from/to|`DATE`|`LONG`|`NUMBER`|`PLS_INTEGER`|`VARCHAR`|
|-------|------|------|--------|-------------|---------|
|**`DATE`**			|	|YUP| 	| 	|YUP|
|**`LONG`**			|	|	|	|	|YUP|
|**`NUMBER`**		|	|YUP| 	|YUP|YUP|
|**`PLS_INTEGER`**	|	|YUP|YUP|	|YUP|
|**`VARCHAR2`**		|YUP|YUP|YUP|YUP|	|

### Some drawbacks

 * may be slower
 * making assumptions about future PL/SQL standards
 * depending on environment (for example date formats)
 * harder to read

## Explicit conversions

 * `TO_NUMBER()`
 * `ROWIDTONCHAR()`
 * `TO_CHAR()`
 * `HEXTORAW()`
 * `TO_CLOB()`
 * `RAWTOHEX()`
 * `CHARTOROWID()`
 * `RAWTONHEX()`
 * `ROWIDTOCHAR()`
 * `TO_DATE()`

## Order of operations

|Operator|Operation|
|--------|---------|
|``**``|Exponentiation
|``+, -``|Identity, negation
|``*, /``|Multiplication, division
|``+, -, ||``|Addition, subtraction, concatenation
|``=, <, >, <=, >=,`` <br>`` <>, !=, ~=, ^=, `` <br>``IS NULL, LIKE, BETWEEN, IN`` | Comparison
|``NOT``|Logical negation
|``AND``|Conjunction
|``OR``|Inclusion

## Nested code blocks

In PL/SQL, one is allowed to nest code blocks, a variables' scope is this block and all nested code blocks. When a variable name is used, the one with the smallest scope is used.

To access variables with the same name, but that are not visible due to scope, one can use the following code:

```sql
<<outer>>
DECLARE -- outer block
	v_myname	VARCHAR2(20);
BEGIN
	v_myname := 'John';
	DECLARE -- inner block
		v_myname	VARCHAR2(20);
	BEGIN
		v_myname := 'Will';
		DBMS_OUTPUT.PUT_LINE('My name is: '||v_myname); -- Will
		DBMS_OUTPUT.PUT_LINE('My name is: '||outer.v_myname); -- John
	END;
END;
```

## SQL statements

To retrieve data from a table, one can use a `SELECT` statement to put values into already declared variables. The query must return exactly one rown for this to work.

```sql
SELECT col1,col2 INTO v_col1, v_col2 FROM table (WHERE ...);
SELECT sum(col1) INTO v_sum_col1 FROM table (WHERE ...);
```

One can also use other SQL statements:

```sql
DELETE FROM table (WHERE ...);
INSERT INTO table (col1, col2, col3) VALUES ('param', 'values', 99)
UPDATE table SET col = newValue (WHERE ...)
```

**WARNING:** column names have smaller scope than variable names ergo `WHERE customer_id = customer_id` will not work (returns all rows).

## Cursors

### Implicit Cursors

Cursors are objects that carry the SQL statement. When an SQL statement is executed, one can use the implicit cursor "SQL" to gain information about that statement.

### Explicit cursors

In the `DECLARE` block, one can declare a cursor and use it in a loop. Here is a simple example:

```sql
DECLARE
	CURSOR cursorName IS
		SELECT col1, col2 FROM table (WHERE ...);
	v_col1	table.col1%TYPE;
	v_col2 	table.col2%TYPE;
BEGIN
	OPEN cursorName; -- executes the query
	LOOP
		FETCH cursorName INTO v_col1, v_col2; -- fetches a row
		EXIT WHEN cursorName%NOTFOUND;
		DBMS_OUTPUT.PUT_LINE(v_col1||' '||v_col2);
	END LOOP;
	CLOSE cursorName; -- required, clears memory
```

Variables can be defined to be of type `cursorName%ROWTYPE`, which is a rowobject. Fields in rows can be accessed through dotnotation. The above now becomes:

```sql
DECLARE
	CURSOR cursorName IS
		SELECT col1, col2 FROM table (WHERE ...);
	v_row	cursorName%ROWTYPE;
BEGIN
	OPEN cursorName; -- executes the query
	LOOP
		FETCH cursorName INTO v_row; -- fetches a row
		EXIT WHEN cursorName%NOTFOUND;
		DBMS_OUTPUT.PUT_LINE(v_row.col1||' '||v_row.col2);
	END LOOP;
	CLOSE cursorName; -- required, clears memory
```

### Cursor attributes

 * **`%FOUND:`** Boolean value that is TRUE if the query returned at least one row.
 * **`%NOTFOUND:`** Boolean value that is TRUE if the query returned no rows or there are no more rows to fetch.
 * **`%ROWCOUNT:`** An integer containing the number of rows affected or the amount of rows already fetched.
 * **`%ISOPEN:`** Evaluates whether the cursor is open.

**WARNING:** A cursor attribute cannot be used inside SQL statements.

### Cursor FOR loops

One can shorten code a lot by using the cursor `FOR` loop. The following 2 statements are logically identical.

|FOR|%ROWTYPE|
|-----|-----|
|`DECLARE`<br/>&nbsp;&nbsp;&nbsp;&nbsp;`CURSOR emp_cursor IS`<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`SELECT employee_id, last_name`<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`FROM employees`<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`WHERE department_id = 50;`<br/>`BEGIN`<br/>&nbsp;&nbsp;&nbsp;&nbsp;`FOR v_emp_record IN emp_cursor`<br/>&nbsp;&nbsp;&nbsp;&nbsp;`LOOP`<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`DBMS_OUTPUT.PUT_LINE(...);`<br/>&nbsp;&nbsp;&nbsp;&nbsp;`END LOOP;`<br/>`END`|`DECLARE`<br/>&nbsp;&nbsp;&nbsp;&nbsp;`CURSOR emp_cursor IS`<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`SELECT employee_id, last_name`<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`FROM employees`<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`WHERE department_id = 50;`<br/>&nbsp;&nbsp;&nbsp;&nbsp;`v_emp_record emp_cursor%ROWTYPE;`<br/>`BEGIN`<br/>&nbsp;&nbsp;&nbsp;&nbsp;`OPEN emp_cursor;`<br/>&nbsp;&nbsp;&nbsp;&nbsp;`LOOP`<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`FETCH emp_cursor INTO v_emp_record;`<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`EXIT WHEN emp_cursor%NOTFOUND;`<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`DBMS_OUTPUT.PUT_LINE(...);`<br/>&nbsp;&nbsp;&nbsp;&nbsp;`END LOOP;`<br/>&nbsp;&nbsp;&nbsp;&nbsp;`CLOSE emp_cursor;`<br/>`END;`<br/>|

One does not need to declare a cursor, but can insert the SQL statement inside a `FOR` declaration:

```sql
FOR v_emp_record IN SELECT employee_id, last_name FROM employees WHERE department_id = 50
LOOP
	DBMS_OUTPUT.PUT_LINE(...);
END LOOP;
```

### Cursor parameters



## Transaction control

A number of commands can be used to control transactions:

 * `COMMIT` is used to finalize the previous statement and permanently write them to the database.
 * `ROLLBACK` rolls the state of the db back to the last `COMMIT`.
 * `SAVEPOINT` is used so one can `ROLLBACK` in steps.

## If/else structure

```sql
IF condition THEN
	statements; -- only executes these if condition is TRUE, not if FALSE or NULL
(ELSIF condition THEN
	statements;)
(ELSE
	statements;)
END IF;
```

**WARNING:** `NULL` is an unknown value, not an empty one. `NULL = NULL` results in `NULL`, as both are unknown.

## Case structure

```sql
CASE v_var -- shorter notation for IF/ELSIF/ELSE with the same variable
	WHEN 'A' THEN 
		statements; -- same as IF v_var = 'A'
	WHEN 'B' THEN
		statements;
	ELSE
		statements;
END CASE;
```

A `CASE` statement can also be used in an assignment:

```sql
v_var :=
	CASE v_other_var
		WHEN 1 THEN 'One'
		WHEN 2 THEN 'Two'
		ELSE 'Many'
	END; -- note the lack of "CASE" here
```

Another use is the searching expression, which is a simplified form of `IF/ELSIF/ELSE`.

```sql
v_var :=
	CASE -- no selector here
		WHEN condition1 THEN 'One'
		WHEN condition2 THEN 'Two'
		ELSE 'Many'
	END; -- note the lack of "CASE" here
```

## Loop statement

A simple loop looks like this:

```sql
LOOP
	statements;
END LOOP;
```

In the statements must be at least one `EXIT` statement, which can be formed like this:

```sql
EXIT; -- to use inside an IF statement in the loop
EXIT WHEN condition; -- simplifies the IF
```

`WHILE` and `FOR` loops are also available:

```sql
WHILE condition LOOP
	statements;
END LOOP;
```

```sql
FOR counter IN (REVERSE) lower..upper LOOP -- counter is defined in the loop as an INTEGER
										   -- lower and upper are included in the loop 1..3 loops 3 times
	statements;
END LOOP;
```

Nested loops can be used, and the `EXIT` keyword exits the smallest loop. To exit multiple loops at the same time, labels can be given, identical to the scope labels.

```sql
DECLARE
	declarations;
BEGIN
	<<outer_loop>>
	LOOP -- outer loop
		<<inner_loop>>
		LOOP -- inner loop
			EXIT outer_loop (WHEN ...) -- Exits both loops
			EXIT WHEN v_inner_done;
		END LOOP;
		EXIT WHEN v_outer_done;
	END LOOP;
END;
```

