# SQL Style Guide

**goal**: Let's not make reading SQL harder than it already is. This 
means keeping lines short, and trading aggressively compact code for 
regularity and ease of modification.

## Line Length

Keep it under 100 characters. Fewer than 80 characters is preferable, but not 
always possible. Greater than 100 should not be accepted. Try to break complex
expressions into multiple lines. Most complex expressions will have natural 
newline points.

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


Even though it is common to put all the conditions in the same line, it is 
much more readable if broken up into multiple lines.

## Variables, Column & Table Names

Keep them short and descriptive. For one off code that is not being committed to a 
repository, the following is okay. 

### Good Sometimes

    SELECT
        r.user_id
      , COUNT(r.id) bookings

    FROM reservation r

    WHERE r._dump IS NULL
      AND r.complete = TRUE
      AND r.created_at >= '2016-01-01'

    GROUP BY 1

### Good all the times

    SELECT
        book.user_id
      , COUNT(book.id) bookings

    FROM reservation book

    WHERE book._dump IS NULL
      AND book.complete = TRUE
      AND book.created_at >= '2016-01-01'

    GROUP BY 1


If there's a naming / aliasing convention set up, use it. Generally speaking, table & column names should be `lower case` with `under_score` separating words. 1 letter aliases should really be avoided in committed code.


## Indentation & Blank Lines

Put a Blank Line between each clause, as above.

Indentation is useful to show where expressions belongs. Proper indentation should
also be employed to break down a long expression (> 100 characters) over multiple 
lines.


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

This is tricky, so read carefully. If one needs to add a new field to `GOOD`, it causes a 1 line diff, But
a 2 line diff to `BAD`. The same applies if we remove the last field `book.checkin_at` from the list. It is
very rare that a query is edited to remove the first field in the list. That's why this rule makes sense.


### Clause dependent indentation

Here, the indentation is used to show membership to a clause, and each clause sets 
its own level of indentation. 

**SELECT Block**: First field is indented 4 spaces, 2nd field onwards indented with 2 spaces

    SELECT
        users.id
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

Notice the first `ON` aligns with `JOIN`, while the 2nd aligns with `LEFT JOIN`. 
In both case, they're aligning with their respective clauses

The `FROM`, `JOIN`, `LEFT JOIN` are all left aligned. This block also left aligns
with `SELECT`. 

**WHERE BLOCK:** With just 1 condition, place it on the same line as above, or put it indented (4 spaces) on the next line, if there are more conditions, put the expressions on separate lines below, and follow the advice on conjugating Boolean Operators.

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

### Bad
 
    GROUP BY users.email, book.created_at, book.checkin_at, inv.created_at

**ORDER BY Block, LIMIT Block:** Same rules as `GROUP BY` Block

**HAVING BLOCK:** Same rules as `WHERE` clause. 

### Comments

Use comments as appropriate. Debuging / Code review is not possible if the reviewer does not know the intent. Big queries should add a blurb at the top explaining what it is meant to return. Committed SQL is often sitting embedded inside other
language. This puts a lot of context switching overload on the reviewer going through the file. Comments are super because
the reviewer can get the context more easily.

Always add a comment if:

* A custom user defined function is being used.
* An `anti-join` or `set-difference` is being used, as this spans both the `FROM` clause & `WHERE` clause.
  In this case, annotate the join, specifiying what condition *SHOULD NOT* be removed from the `WHERE` clause.
* *TODO: more examples of tricky situations*

### CTEs & Subqueries

Avoid Sub Queries at all costs. If you DB supports CTEs (`WITH` queries), use them, otherwise try to get by with
a temporary table.

Indent the `SELECT` query within a CTE. With multiple CTEs, the comma rules apply.

### GOOD

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

### GOOD

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

If a field accepts `NULL`, always wrap it in a COALESCE with sensible default before comparing. Avoid this, 
and you'll have to debug one of the trickiest bugs in SQL 

### WHY? Try running these queries

    SELECT NULL = TRUE;
    SELECT NULL != TRUE
    SELECT NULL != FALSE;
    
In particular, the practice of allowing `NULL` in a Boolean column should be frowned upon. Go yell at your application 
developer the next time you see one.
