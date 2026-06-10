## Here will discuss about Advance Database Concepts

1. Function in SQL
2. Procedure in SQL
3. Trigger in SQL
4. Indexing in SQL

Bonus

5. Views in SQL


### Function Syntax

```sql

-- Syntax
CREATE [OR REPLACE] FUNCTION function_name (arguments)
RETURNS return_data_type AS $$
DECLARE
    -- Variable declarations (optional)
    variable_name data_type;
BEGIN
    -- Function logic
    RETURN value;
END;
$$ LANGUAGE plpgsql;

```


Example Here:

```sql

-- Example: A function that calculates a 10% bonus for an employee
CREATE OR REPLACE FUNCTION calculate_bonus(emp_salary INT)
RETURNS NUMERIC AS $$
BEGIN
    RETURN emp_salary * 0.10;
END;
$$ LANGUAGE plpgsql;

-- call it:
SELECT empname, calculate_bonus(salary) FROM emp;




```

### Procedure Syntax

```sql

-- Syntax
CREATE [OR REPLACE] PROCEDURE procedure_name (arguments)
AS $$
DECLARE
    -- Variable declarations (optional)
BEGIN
    -- Procedure logic (Transaction control allowed here)
END;
$$ LANGUAGE plpgsql;


```

Example Here:

```sql

-- Example: A procedure to give a specific employee a raise
CREATE OR REPLACE PROCEDURE give_raise(target_emp TEXT, raise_amount INT)
AS $$
BEGIN
    UPDATE emp 
    SET salary = salary + raise_amount 
    WHERE empname = target_emp;
    
    COMMIT; -- Commits the transaction directly inside the procedure
END;
$$ LANGUAGE plpgsql;

-- call it:
CALL give_raise('Alice', 5000);




```


### Trigger Syntax

```sql

-- Syntax (Step 1: Create the Trigger Function)
CREATE OR REPLACE FUNCTION trigger_function_name()
RETURNS TRIGGER AS $$
BEGIN
    -- Logic using special variables: NEW, OLD, TG_OP

    RETURN NEW;   -- or OLD, or NULL depending on the operation
END;
$$ LANGUAGE plpgsql; 



-- Syntax (Step 2: Bind the Function to a Table)
CREATE TRIGGER trigger_name
{BEFORE | AFTER | INSTEAD OF} {INSERT | UPDATE | DELETE}
ON table_name
[FOR EACH ROW]
EXECUTE FUNCTION trigger_function_name();


```


Example Here:

```sql


-- Example: Prevent updating a salary to a lower value
CREATE OR REPLACE FUNCTION prevent_pay_cut()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.salary < OLD.salary THEN
        RAISE EXCEPTION 'Salaries cannot be decreased!';
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;


CREATE TRIGGER check_salary_drop
BEFORE UPDATE ON emp
FOR EACH ROW
EXECUTE FUNCTION prevent_pay_cut();



```


### Views Syntax

```sql

-- Syntax
CREATE [OR REPLACE] VIEW view_name AS
SELECT columns
FROM tables
WHERE conditions;


```

Example Here:

```sql

-- Example: A view showing only high-earning employees (above something or 80,000)
CREATE OR REPLACE VIEW high_earners AS
SELECT empname, salary, last_date
FROM emp
WHERE salary > 80000;

-- call it:
SELECT * FROM high_earners;




```



### Here is and Trigger Function Examples on emp tables

```sql

-- 1. create table

CREATE TABLE emp (
    empname           text,
    salary            integer,
    last_date         timestamp,
    last_user         text
);


-- 2. create function emp_stamp()

CREATE FUNCTION emp_stamp() RETURNS trigger AS $emp_stamp$

-- 3. Body main logic between Begin to end   
    BEGIN
        -- Check that empname and salary are given
        IF NEW.empname IS NULL THEN
            RAISE EXCEPTION 'empname cannot be null';
        END IF;
        IF NEW.salary IS NULL THEN
            RAISE EXCEPTION '% cannot have null salary', NEW.empname;
        END IF;

        -- Who works for us when they must pay for it?
        IF NEW.salary < 0 THEN
            RAISE EXCEPTION '% cannot have a negative salary', NEW.empname;
        END IF;

        -- Remember who changed the payroll when
        NEW.last_date := current_timestamp;
        NEW.last_user := current_user;
        RETURN NEW;
    END;

-- 4. telling what will be the language sql or plpgsql format
$emp_stamp$ LANGUAGE plpgsql;

-- 5. creating trigger emp_stamp and implementing when it will occur

CREATE TRIGGER emp_stamp BEFORE INSERT OR UPDATE ON emp
    FOR EACH ROW EXECUTE FUNCTION emp_stamp();


```