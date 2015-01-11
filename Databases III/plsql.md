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
	DBMS_OUTPUT.PUT_LINE('My name is: '||v_myname);
	DECLARE -- inner block
		v_myname	VARCHAR2(20);
	BEGIN
		v_myname := 'Will';
		DBMS_OUTPUT.PUT_LINE('My name is: '||v_myname);
		DBMS_OUTPUT.PUT_LINE('My name is: '||outer.v_myname);
	END;
END;
```

