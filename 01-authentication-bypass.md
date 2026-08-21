## Lab

SQL injection vulnerability allowing login bypass

## Difficulty

Apprentice

## Objective

The objective of this lab is to bypass the login functionality
using SQL injection and gain access to the administrator account.

## Testing

The application contains a login panel.

I intercepted the login request using Burp Suite and tested the
username parameter for SQL injection.

I used the following payload in the authorized lab:

`administrator' OR 1=1--`

The single quote (`'`) can terminate the existing string in the
SQL query. The `OR 1=1` condition is always true, and `--` is used
to comment out the remaining part of the query.

As a result, the application's authentication logic was bypassed
and I was able to log in as the administrator.

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
