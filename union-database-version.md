# SQL Injection — Database Version Disclosure (Oracle)

## Lab

**SQL injection attack, querying the database type and version on Oracle**

## Difficulty

**Practitioner**

## Objective

The objective of this lab is to exploit a SQL injection vulnerability in the product category filter and retrieve the database version information.

---

## Vulnerability

**SQL Injection (SQLi) — UNION-based Information Disclosure**

The application is vulnerable because user-controlled input in the `category` parameter is directly included in a SQL query without proper parameterization.

---

## Testing

The application contains a product category filter. I identified the `category` parameter as the input point for testing.

The original request was:

GET /filter?category=Pets HTTP/2

<img width="1390" height="234" alt="sql1" src="https://github.com/user-attachments/assets/7651b105-1650-46f8-bc47-0ec77c7957fd" />

I sent the request to Burp Suite Repeater so that I could modify the category parameter and observe the application's responses.

## Step 1 — Test for SQL Injection

First, I tested the category parameter with a single quote (') to check how the application handled the input.

The application returned a 500 Internal Server Error, which indicated that the input was affecting the backend SQL query.

<img width="1406" height="235" alt="sql2" src="https://github.com/user-attachments/assets/a93394c8-56d9-43be-b9ae-46debfaca2db" />

## Step 2 — Test SQL Comment Syntax

Next, I tested the SQL comment syntax (--) to determine whether I could comment out the remaining part of the SQL query.

<img width="1397" height="226" alt="sql4" src="https://github.com/user-attachments/assets/a6b6de44-00e4-4afc-8b14-c9d787372a84" />

## Step 3 — Identify the Number of Columns

I used the ORDER BY clause with different column numbers to determine how many columns were returned by the original query.

Test 1

Payload:
```
' ORDER BY 1--
```
The application returned:

HTTP/2 200 OK

<img width="1397" height="226" alt="sql4" src="https://github.com/user-attachments/assets/192fffa2-c4ef-4be4-be0e-6611d90761a0" />

Test 2

Payload:
```
' ORDER BY 2--
```
The application returned:

HTTP/2 200 OK

<img width="1397" height="196" alt="sql5" src="https://github.com/user-attachments/assets/d9ab0a89-3f82-4dd3-ab89-7b74ac3fbd8c" />

Test 3

Payload:
```
' ORDER BY 3--
```
The application returned:

HTTP/2 500 Internal Server Error

<img width="1406" height="248" alt="sql6" src="https://github.com/user-attachments/assets/92608b64-fc6d-4a07-b979-6a9679744d89" />

This indicates that the original query returns 2 columns, because ORDER BY 1 and ORDER BY 2 were accepted, while ORDER BY 3 caused an error.

## Step 4 — Retrieve the Oracle Database Version

The objective was to retrieve the database version.

I used the following payload in the category parameter:

```
' UNION SELECT BANNER,NULL FROM v$version--
```
<img width="1404" height="231" alt="sql7" src="https://github.com/user-attachments/assets/95ab0c4c-2fab-4c1f-81a1-61a4876a362e" />

Why This Payload Works
```
'closes the existing string value in the SQL query.

UNION SELECT adds the result of another query to the original query.

BANNER contains Oracle version information.

NULL is used for the second column because the original query returns two columns.

FROM v$version queries Oracle's version information.

-- comments out the remaining part of the original SQL statement.'
```
The response displayed Oracle database version information, including:

Oracle Database 11g Express Edition
Release 11.2.0.2.0 - 64bit Production

<img width="1408" height="640" alt="sql8" src="https://github.com/user-attachments/assets/ed1c0801-f5f0-4788-85e1-75e289150351" />

The lab was successfully solved after the database version string was displayed.

## Impact

Potential impacts include:

* Disclosure of the database type and version.
* Database fingerprinting.
* Assistance in selecting database-specific SQL injection techniques.
* Exposure of internal database information.
* Increased risk when combined with other SQL injection attacks.

The version disclosure itself is primarily an information disclosure, but the underlying SQL injection could potentially have a much greater impact depending on the application's database permissions and query context.

## Remediation

1. Use Parameterized Queries

Use prepared statements or parameterized queries instead of concatenating user input directly into SQL queries.
```
Example:

cursor.execute(
    "SELECT ... FROM products WHERE category = :category",
    {"category": category}
)
```
The exact implementation depends on the programming language and database driver.

2. Validate Input

Apply appropriate server-side validation to the category parameter.

Input validation should be treated as an additional security control and not as a replacement for parameterized queries.

3. Use Least-Privilege Database Accounts

The application's database account should have only the permissions required for the application to function.

4. Handle Database Errors Safely

Do not expose raw database errors or detailed SQL errors to users.

Return a generic error message to the user while logging detailed errors securely on the server.

5. Perform Regular Security Testing

Perform regular penetration testing, and secure code reviews to identify SQL injection vulnerabilities before deployment.
