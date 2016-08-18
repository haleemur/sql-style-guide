# SQL Style Guide


    "Elegance is not optional" 
          - Richard A. O'Keefe


**goal**: Let's not make reading SQL harder than it already is. This means keeping lines short, and trading aggressively compact code for regularity and ease of modification. All code, even analysts' SQL is read more often than it is written.

# Raison d'etre

SQL is often written by non programmers. Power Business Users, Analysts, Data Scientists & Project Owners often claim to know / understand SQL. And they're right. SQL is an extremely friendly language. Its easy to pick up due to its natural language like syntax, and its extremely concise & expressive. The diverse background of all the practitioners of SQL yield a very complex world where very few things are standardized. Just as there are a lot of really bad javascript & php out there, there are a lot of badly written sql queries in existence. 

Expert Programmers who normally work in their favourite language have to deal with SQL whenever they interact with databases. They bring the flavours of their favourite language into SQL. Good intentioned as this may be, it further pollutes SQL style.

A fairly common complaint is that 'SQL is easy to write, but hard to read later'. This is not true for good SQL.

### What is good SQL

The same reasons that make snippets of code in other languages good also hold for SQL

* meaningful variable names
* short(ish) lines
* good indentation and whitespace
* good use of line breaks and continuation
* useful comments
* efficient
* modularity


## Line Length

Keep it under 100 characters. Fewer than 80 characters is preferable, but not always possible. 
Greater than 100 should not be accepted. Try to break complex expressions into multiple lines.

Most complex expressions will have natural newline points at the (infix) operators.

### Good

    CASE 
      WHEN 
            users.email LIKE '%gmail.com' 
         OR users.email LIKE '%hotmail.com' 
         OR users.email LIKE '%yahoo.com' 
      THEN 'free-domain' 
      ELSE 'company-domain' 
    END domain_type 

### Bad

    CASE WHEN users.email LIKE '%gmail.com' OR users.email LIKE '%hotmail.com' OR users.email LIKE '%yahoo.com' 
    THEN 'free-domain' 
    ELSE 'company-domain' END domain_type 

Even though it is common to put all the conditions in the same line, it is much more readable if broken up into multiple lines, and the operands are aligned rather than the operator with the preceding operand.

### Good

    ,   orders.subtotal
      - orders.discount
      + orders.tax
      + orders.shipping AS payable
      
### Bad

    , orders.subtotal -
      orders.discount +
      orders.tax + 
      orders.shipping AS payable
      
### Ugly
    
    , orders.subtotal
      - orders.discount
      + orders.tax 
      + orders.shipping AS payable

### Also Ugly, but acceptable for very short expressions

    , orders.subtotal - orders.discount + orders.tax + orders.shipping AS payable
    
    
## Variables, Column & Table Names

### Usage in queries

* Alias long table names.
* Specify the table alias (or full name) when referring to columns. This avoids future name collisions when more joins are added, and makes diffs easier to read.
* Aliases are meant to be short & descriptive (3 ~ 5 characters is good, IMO).
* 1 letter aliases are to be avoided in committed code. *Might be okay for one off queries*
* `AS` is optional when declaring aliases or labels, because the alias **should** be the last word in a *fairly short* line. Be consistent with the internal guideline.

### Bad

    SELECT
        user_id
      , COUNT(id) bookings
    FROM reservation
    WHERE _dump IS NULL
      AND complete = TRUE
      AND created_at >= '2016-01-01'
      
Adding a join would likely confuse the database due to field name collisions. Its common for tables to have `id`, `user_id` and `created_at` column names.

### Good Sometimes

    SELECT
        r.user_id
      , COUNT(r.id) bookings

    FROM reservation r

    WHERE r._dump IS NULL
      AND r.complete = TRUE
      AND r.created_at >= '2016-01-01'

    GROUP BY 1

The alias is clear in this context, but joining to `cancelled_reservation` with the same aliasing convention yields the abbreviation `cr`. Joined again to `credit_card` may produce `cc`. At which point this convention starts to show its lackings.

### Good all the times

    SELECT
        books.user_id
      , COUNT(book.id) bookings

    FROM reservation books

    WHERE books._dump IS NULL
      AND books.complete = TRUE
      AND books.created_at >= '2016-01-01'

    GROUP BY 1

### Naming Database objects

* If there's a naming / aliasing convention set up, adhere to it. *e.g. If **every body** agrees that `u` is an unambiguous alias for `users` table, then its okay, I guess, even though it is contrary to the advice given above.*

* Avoid SQL reserved words, i.e. don't name a table or column `user`, or timestamp.

* Avoid spaces, or anything else that would require one to put surrounding `"` around table or column names.

* *I personally prefer* table & column names & their aliases to be `lower case` with `under_score` separating words. Some people use `PascalCase` or `camelCase`. Other conventions state that tables should be `PascalCase` while columns `camelCase`. Adhere to conventions your team has used in the past.

* Stick with either plurals (e.g. `users`) or singular (e.g. `reservation`) for table names. The plural form indicates that the table is a collecion of many records, while the singular refers to the type of entity being stored. *Note that in this document I mix & match. This is because the examples are inspired by a database I have worked on that was designed / implemented by someone else. Don't be that person.*

* All Keys should have the common suffix. This helps identify relationships easily to later developers. `_id` (*or `Id`*) is a classic choice you can't go wrong with (e.g. `user_id`). Its like ordering chocolate ice cream. 

* Try to make your primary key names *guessable*. Both `users.id` and `users.user_id` are great candidates for the primary key of the `users` table. If you go with 1 approach, be consistent across all your tables. Obvious exclusions to this rule are association proxy tables or calendar tables, which may have composite keys that follow a natural name other than `entity_name_id`, e.g. the primay key for the calendar table might be `date`.


## Indentation & Blank Lines

Put a Blank Line between each clause, as in above example.

Indentation is useful to show where expressions belong. Proper indentation should also be employed to break down a long expression (> 100 characters) over multiple lines.


### Boolean operators

In SQL, we're interested in expressions, so all the *expressions* conjugated with a boolean operator 
(`OR` | `AND`) should align. 

### Good

        users.created_at > '2016-01-01'
    AND users.role = 'admin'

### Bad
    users.created_at > '2016-01-01'
    AND users.role = 'admin'

### Also Bad
    users.created_at > '2016-01-01'
    AND users.old    = 'admin'

This way it is easier to scan the expressions, and spot where expressions are conjugated using the `AND` and `OR` operators. 

`users.old = 'admin'` is an expression, it should be viewed in its entirety, so the extra space before the `=`
has no utility. And what will you do if the next expression has `users.organization_id`? All the filter expressions will need laborious additions of spaces. **This will screw up the diff. DON'T BE STUPID**

Use brackets whenever there are both `AND` and `OR` present in the conjugate. Not everyone remembers operator precedence.

### Good

        users.created_at > '2016-01-01'
    AND (   users.role = 'admin'
         OR users.role = 'moderator')
        
### Also Good

        users.created_at > '2016-01-01'
    AND (
            users.role = 'admin'
         OR users.role = 'moderator'
        )

### Bad

        users.created_at > '2016-01-01'
    AND (users.role = 'admin' OR users.role = 'moderator')

### REALLY Bad, omission changes the logic, as default operator precedence kicks in

        users.created_at > '2016-01-01'
    AND users.role = 'admin' 
     OR users.role = 'moderator'


### Comma Separated List

These show up in `SELECT`, `GROUP BY`, `LIMIT` clauses most often. The comma can also show up 
occassionally in the `FROM` clause, `WINDOW` expressions, and queries with multiple CTEs.

### Good

      users.id
    , users.email
    , book.created_at
    , book.checkin_at

### Bad

    users.id,
    users.email,
    book.created_at,
    book.checkin_at

Comma omission is one of the most common errors seen during interactive querying. The *comma-at-start* style makes it easy to spot missing commas, because all the commas are aligned and the space makes it stick out. 

If one needs to add a new field to `GOOD`, it causes a 1 line diff, But a 2 line diff to `BAD`. The extra diff is due to the `,` at the end of `book.checkin_at`. The same applies if we remove the last field `book.checkin_at` from the list. This
is also useful during interactive querying. Most queries are built iteratively as expressions are added, each time an expression is added / removed the programmer only needs to modify 1 line.


### Clause dependent indentation

Here, the indentation is used to show membership to a clause, and each clause sets its own level of indentation. 

**SELECT Block**: First field is indented 4 spaces, 2nd field onwards indented with 2 spaces

    SELECT
        users.id
      , users.email

### Also Good

    SELECT users.id
         , users.email
         
**FROM Block:** A new join on a new line, each join condition on a separate line, 
indented to align with the `JOIN`. 

    SELECT 
        users.email
      , book.created_at
      , book.checkin_at
      , inv.created_at invoice_at
    
    FROM reservation book
    JOIN users
      ON users.id = book.user_id
    LEFT JOIN invoice inv
           ON inv.reservation_id = book.id
          AND inv.is_paid = FALSE

    WHERE book.checkin_at IS NOT NULL

Notice the first `ON` aligns with `JOIN`, while the 2nd aligns with `LEFT JOIN`. In both cases, they're aligning with their respective clauses

The `FROM`, `JOIN`, `LEFT JOIN` are all left aligned. This block also left aligns
with `SELECT`. 

**WHERE Block:** With just 1 condition, place it on the same line as above, or put it indented (4 spaces) on the next line, if there are more conditions, put the expressions on separate lines below, and follow the advice on conjugating Boolean Operators.

### Good

    WHERE book.checkin_at IS NOT NULL

### Also Good

    WHERE book.checkin_at IS NOT NULL
      AND book.is_complete = TRUE

### Also Good

    WHERE 
          book.checkin_at IS NOT NULL
      AND book.is_complete = TRUE

### Bad

    WHERE book.checkin_at IS NOT NULL
    AND book.is_complete = TRUE
    
*... but you wouldn't make this mistake because you've read the section on Boolean Operators*

**GROUP BY Block:** If using field positions, comma separate the numbers on 1 line. 
If using field expressions or names, the Comma Separated List rules apply.

### Good

    GROUP BY 1, 2, 3, 4

### Good

    GROUP BY 
        users.email
      , book.created_at
      , book.checkin_at
      , inv.created_at 

### Also Good

    GROUP BY users.email
           , book.created_at
           , book.checkin_at
           , inv.created_at 

### Bad
 
    GROUP BY users.email, book.created_at, book.checkin_at, inv.created_at

**ORDER BY Block, LIMIT Block:** Same rules as `GROUP BY` Block

**HAVING Block:** Same rules as `WHERE` clause. 

### Comments

Use comments as appropriate. Debuging / Code review is not possible if the reviewer does not know the intent. Big queries should add a blurb at the top explaining what it is meant to return. Committed SQL is often sitting embedded inside other
language. This puts a lot of context switching overload on the reviewer going through the file. Comments are super because
the reviewer can get the context more easily.

Always add a comment if:

* A custom user defined function is being used.

* An `anti-join` or `set-difference` is being used, as this spans both the `FROM` clause & `WHERE` clause.
  In this case, annotate the join, specifiying what condition *SHOULD NOT* be removed from the `WHERE` clause.

* todo: more examples of tricky situations

### CTEs & Subqueries

Avoid Sub Queries at all costs. If you DB supports CTEs (`WITH` queries), use them, otherwise try to get by with
a temporary table.

Indent the `SELECT` query within a CTE. With multiple CTEs, the comma rules apply.

### Good

    WITH 
      first_books AS (
      SELECT
          books.user_id
        , MIN(book.created_at) book_at
    
      FROM reservation books
    
      GROUP BY 
          books.user_id
    )
    SELECT
        TO_CHAR(first_books.book_at, 'YYYY-MM') first_book_month
      , COUNT(*) bookers
    
    FROM first_book
    
    GROUP BY 
        TO_CHAR(first_books.book_at, 'YYYY-MM')

### Good

    WITH 
      first_books AS (
      SELECT
          book.user_id
        , MIN(books.created_at) book_at
        , 'first_book'::varchar event_type
      
      FROM reservation books
      
      GROUP BY 
          books.user_id
    )
    , first_sesions AS (
      SELECT 
          sess.user_id
        , MIN(sess.created_at) sess_at
        , 'first_session'::varchar event_type
      
      FROM sessions sess
      
      GORUP BY 
          sess.user_id
    )
    , month_books AS (
      SELECT
          TO_CHAR(first_book.book_at, 'YYYY-MM') event_month
        , event_type
        , COUNT(*) event_count
      
      FROM first_book
      
      GROUP BY 1, 2
    )
    , month_sess AS (
      SELECT
          TO_CHAR(first_book.book_at, 'YYYY-MM') event_month
        , event_type
        , COUNT(*) event_count
        
      FROM first_book
      
      GROUP BY 1, 2
    )
    SELECT * FROM month_books

    UNION ALL 
    
    SELECT * FROM month_sess


### Window Expressions

Define the window expression at the end of the query if your DB allows it. Postgresql does. 

Putting them inline in the `SELECT` clause decreases legibility, due to long lines. 


### NULL Values Caveat

If a field accepts `NULL`, always wrap it in a `COALESCE` function with sensible default before comparing. Avoid doing this, and you'll have to debug one of the trickiest bugs in SQL.

### WHY? Try running these queries

    SELECT NULL = TRUE; -- returns NULL
    
    SELECT NULL != TRUE; -- returns NULL
    
    SELECT NULL != FALSE; -- returns NULL
    
    SELECT NULL IS NULL IS NULL; -- returns FALSE
    
In particular, the practice of allowing `NULL` in a Boolean column should be frowned upon. Go yell at your application 
developer the next time you see one.

## Misc. comments

* Put all dimensions at the top of the `SELECT` list, and all aggregates (or windowed fields) at the bottom. 
  
  This makes it easy to evaluate what is doing the grouping and what is being aggregated, allows one to copy-paste into the `GROUP BY` clause, or build it easily by specifying position. *PROTIP: you can count the number of dimensions by subtracting the line numbers between the first & last dimension*

* Make every effort to keep queries short. Other people don't like reading your long code. Longer code leads to more errors. 

* It's great that you can write complex where clauses. You know what's better? Altering your schemas so that incomprehensible `(condition 1 AND condition 2 AND (condition 3 OR condition 4)) OR NOT (condition 5 AND condition 1)` expression is not needed.

* Don't repeat yourself. Convert frequently used CTEs into views.
