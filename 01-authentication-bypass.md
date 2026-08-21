

## SQL injection vulnerability allowing login bypass

## Difficulty

Apprentice

## Objective

The objective of this lab is to bypass the login functionality
using SQL injection and gain access to the administrator account.

## Testing

First, I tested the username field with a single quote (`'`) to check
how the application handled the input. The application returned a
`500 Internal Server Error`, which indicated that the input was
affecting the backend SQL query.


<img width="1852" height="620" alt="Screenshot (288)" src="https://github.com/user-attachments/assets/83ecb04b-cf4e-4613-bead-7b2239dcfd0f" />

 
 <img width="750" height="583" alt="Screenshot (289)" src="https://github.com/user-attachments/assets/ca12b932-4ecc-4aae-9538-1207adabb930" />


Next, I tested the SQL comment syntax (`--`) to determine whether
I could comment out the remaining part of the SQL query.

Finally, I used the following SQL injection payload in the username
field:

`administrator' OR 1=1--`

<img width="1842" height="648" alt="Screenshot (285)" src="https://github.com/user-attachments/assets/720d9829-5c1d-43e1-bae9-68e0aadff312" />


The single quote (`'`) terminates the existing string in the SQL
query. The `OR 1=1` condition is always true, and `--` comments out
the remaining part of the query.

The application accepted the input, allowing me to bypass the
authentication and access the administrator panel.


<img width="1816" height="588" alt="Screenshot (286)" src="https://github.com/user-attachments/assets/e206e601-024b-43ca-909a-3915cb9daa47" />

## Impact

If this vulnerability existed in a real application, an attacker
could potentially bypass authentication and gain unauthorized
access to user or administrative functionality.

Depending on the application's design, this could lead to
account compromise and unauthorized access to sensitive data.

## Remediation

- Use parameterized queries / prepared statements.
- Do not concatenate user input directly into SQL queries.
- Apply appropriate input validation.
- Use least-privilege database accounts.
