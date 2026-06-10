## Here will discuss about Advance Database Concepts

1. Function in SQL
2. Procedure in SQL
3. Trigger in SQL
4. Indexing in SQL

Bonus

5. Views in SQL


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