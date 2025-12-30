---
title: "DIY Web App Lab"
icon: octicons/browser-16
draft: false
#date:
#  created: 2023-06-14
#  updated: 2025-12-29
categories:
  - web
  - diy
  - pentesting
  - webapp
  - ai
  - lab
---

# Build Your Own Web Application Testing Environment

How to deploy a web application testing environment, to:

- Have better visibility and understanding behind what's happening server side
- Gain muscle memory from interacting with individual components locally
- Demonstrate building solutions to problems with help from ChatGPT

Why you would do this:

- To better understand the systems and processes versus using abstraction technologies
- To have the ability to modify application logic and environment configuration dynamically
- To examine each component's behavior with analysis tools (wireshark, tcpdump, auditd, Sysmon)
- To make the deployment steps repeatable

What this guide covers:

- Deploying a LAMP stack manually
- Deploying a LAMP stack with Docker Compose
- Deploying a LAMP stack with Ansible
- Configuring Apache, MySQL, and PHP
- Setup and configure basic logging for Apache, MySQL, and PHP
- Working with ChatGPT to construct your own vulnerable web application code (*quickly*)
- Working with ChatGPT to modify and fix the vulnerable code
- Verifying output from ChatGPT
- Reviewing considerations from an offensive, administrative, and developer perspective

!!! note "Source Code"

    The source files are available in the project's GitHub public archive: [straysheep-dev/cheatsheets/diy-webapp-pentesting-lab](https://github.com/straysheep-dev/cheatsheets/tree/main/diy-webapp-pentesting-lab)

<!-- more -->

---


!!! warning "AI Usage"

    *This cheatsheet was generated with the help of GPT-3.*

    ChatGPT Version Used: [ChatGPT May 24 Version](https://help.openai.com/en/articles/6825453-chatgpt-release-notes)

    Everything produced by ChatGPT has been quoted below using:

    > markdown quotation syntax

    The following citation format was taken from:

    - [Citing ChatGPT APA Style](https://apastyle.apa.org/blog/how-to-cite-chatgpt)

    ```
    OpenAI. (2023). ChatGPT (May 24 version) [Large language model].
    https://chat.openai.com/chat
    ```


## Requirements

Any of the following will work, noting this guide uses Ubuntu 20.04 as it's lab environment:

- Local Linux server VM
- Local Linux desktop VM
- Cloud Linux server VM

Configuring proper firewall rules is not covered here, but you should not expose any of these services publicly. An easy way to achieve this is by using SSH public key auth and local port forwarding.

- You only need to expose SSH (tcp/22) on the testing VM
- Connect and send / receive data to the vulnerable web app safely over SSH

To port forward your `localhost:8080` to the `remote-localhost:80`:

```bash
ssh -p <port> -L 127.0.0.1:8080:127.0.0.1:80 user@remote-host
```

Browsing to 127.0.0.1:8080 on your local machine will be like browsing to 127.0.0.1:80 on the lab VM.


## LAMP Stack Manual Install + Setup

Install Apache2 + MySQL + PHP on Ubuntu 20.04:

- [Digital Ocean: Install Apache2](https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-ubuntu-20-04)
- [Digital Ocean: Install PHP](https://www.digitalocean.com/community/tutorials/how-to-install-php-8-1-and-set-up-a-local-development-environment-on-ubuntu-22-04)
- [Digital Ocean: Install MySQL](https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-20-04)
- [Digital Ocean: Install LAMP Stack](https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-ubuntu-20-04)


All of the commands to install LAMP manually:

```bash
sudo apt update

# Apache2
sudo apt install -y apache2

# PHP
sudo apt install -y php php-mysql

# MySQL
sudo apt install -y mysql-server
sudo systemctl start mysql.service

# Package Checks
systemctl status apache2.service
php -v
systemctl status mysql.service
```

That's all you'll need to start working with a LAMP stack.

We'll (briefly) look at each component just to get a sense of how it works and how to interact with it locally (from a normal administrator or developer's perspective).


## Apache

- [Apache HTTP Server Documentation](https://httpd.apache.org/docs/)

Apache automatically starts up and listens on port of all interfaces (`*:80`) by default.

You can confirm this with `ss`:

```bash
sudo ss -anp -A inet
```

The directories we'll be using are:

- `/var/www/html/` is where our web application code and content goes to be served
- `/var/log/apache2/` contains apache's system logs
- `/etc/apache2/` is where apache's configuration files exist


## PHP

- [PHP Manual](https://www.php.net/manual/en/)

With PHP, we'll see how to execute code locally first, since it's different than injecting commands remotely.

The following works in bash:

```bash
php -r "system('id');"

uid=1000(ubuntu) gid=1000(ubuntu) groups=1000(ubuntu)
```

While the following does not:

```bash
php -r "echo system(\$_GET['id']);"

PHP Notice:  Undefined index: id in Command line code on line 1
PHP Warning:  system(): Cannot execute a blank command in Command line code on line 1
```

From ChatGPT:

> The error you encountered indicates that the $_GET['id'] variable is not defined, which is expected because it is typically used in web applications to access parameters passed through the URL. When executing PHP code from the command line, you don't have access to the $_GET superglobal array.

- [PHP $_GET Reserved Variable](https://www.php.net/manual/en/reserved.variables.get.php)
- [Wordpress $_SERVER superglobal](https://make.wordpress.org/cli/handbook/guides/common-issues/#php-notice-undefined-index-on-_server-superglobal)
- [PHP Passing $_GET via the CLI](https://stackoverflow.com/questions/4186392/php-passing-get-in-the-linux-command-prompt)
- [How to pass variables to PHP Script via CLI](https://serverfault.com/questions/187025/how-to-pass-get-variables-to-a-php-script-via-the-command-line/187038#187038)


## MySQL

- [MySQL: Getting Started](https://dev.mysql.com/doc/mysql-getting-started/en/#mysql-getting-started-connecting)

Using MySQL (at least when installed via apt-get) will require running as root, or sudo. Simply specifying `-u root` won't be enough.

```bash
sudo mysql -u root -p
```

At any time to get the status of mysql, including your current user context use `status`:

```bash
mysql> status
--------------
mysql  Ver 8.0.33-0ubuntu0.20.04.2 for Linux on x86_64 ((Ubuntu))

Connection id:		30
Current database:	LOGIN_TEST
Current user:		new_user@localhost
SSL:			Not in use
Current pager:		stdout
...
```

When reading lengthy output from queries, you have a few options to make them more readable:

- Append `\G` to the query instead of `;` for vertical formatting
- Set the reader to `\P less -S` and scroll with left and right arrow keys, disable this with `\n`
- Copy the terminal output and paste it into a text editor

From the mysql man page:

`--vertical`, `-E` Print query output rows vertically (one line per column value). Without this option, you can specify vertical output for individual statements by terminating them with `\G`.


## Create a Test Database

Next we need a test database to serve as the backend for a test login page.

Create the test database:

> ```mysql
> CREATE DATABASE your_database_name;
> ```

Then move into that database as your current working database:

> ```mysql
> USE your_database_name;
> ```

Create a new table named "users" with two columns, username and password:

> ```mysql
> CREATE TABLE users (
>     id INT PRIMARY KEY AUTO_INCREMENT,
>     username VARCHAR(255) NOT NULL,
>     password VARCHAR(255) NOT NULL
> );
> ```

Insert data into your users table, as many username/password pairs as you'd like:

> ```mysql
> INSERT INTO users (username, password) VALUES ('testuser', 'testpassword');
> ```

Let's also insert an admin user with a complex password for the sake of testing:

```sql
INSERT INTO users (username, password) VALUES ('admin', 'WDt0qAOYZiOotpPa8sGjuBvQfQGTeo9H');
```


## Enumerating MySQL Data

Show all of your databases:

> ```mysql
> SHOW DATABASES;
> ```

Show all of a database's tables:

```mysql
SHOW TABLES FROM your_database;
```

Show all of a table's columns:

> ```sql
> SHOW COLUMNS FROM your_database.your_table;
> ```

Dump all of the data from a table named 'your_table':

> ```mysql
> SELECT * FROM your_datase.your_table;
> ```

For example to step through enumerating the general query log table if it's enabled:

| Query | Description |
| ----- | ----------- |
| `SHOW DATABASES;` | Show all available databases |
| `SHOW TABLES FROM mysql;` | List all of the tables in the mysql database |
| `SHOW COLUMNS FROM mysql.general_log;` | List all of the columns in the general_log table |
| `SELECT * FROM mysql.general_log;` | Dump all of the data from the general_log table |


## MySQL User Management

Before we move on, we should be sure our MySQL instance has a unique set of credentials defined by us that we can leverage in various php functions.


### Update Root's Password

To continue using the `root` user within MySQL, update it's password with:

> ```mysql
> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'new_password';
> ```

From now on, you can run `mysql -u root -p` as a normal user, and provide the `<new_password>` when prompted to access mysql databases.


### Create a Low-privileged MySQL User

You may not want to always authenticate to MySQL as root. For this we'll need a low privleged user with limited capabilities within mysql. ChatGPT once again provides a brief walkthrough for us.

> Creating a second MySQL user and password pair:
>
> Log in to MySQL as the root user using the new password you set:
>
> ```bash
> mysql -u root -p
> ```
>
> Once logged in, you can create a new user using the following command:
>
> ```sql
> CREATE USER 'new_user'@'localhost' IDENTIFIED WITH mysql_native_password BY 'new_password';
> ```
>
> Replace 'new_user' with the desired username and 'new_password' with the desired password.
>
> Grant appropriate privileges to the new user. For example, if you want to grant all privileges on a specific database, you can use the following command:
>
> ```sql
> GRANT ALL PRIVILEGES ON your_database.* TO 'new_user'@'localhost';
> ```
>
> Replace 'your_database' with the name of the database you want to grant access to.
>
> Finally, flush the privileges to apply the changes:
>
> ```sql
> FLUSH PRIVILEGES;
> ```

Note that you can specify databases and tables in the following ways when defining privileges:

| Syntax | Description |
| ------ | ----------- |
| `*.*` | All databases and all tables |
| `your_database.*` | All tables within `your_database` |
| `your_database.this_table` | Only `this_table` within `your_database` |

From here you'll have root's password updated and ready to use in php functions, and a separate user with either no privileges (if you didn't grant them any) or all privileges over a single database, in our case the `LOGIN_TEST` database (or whatever you named yours) would make the most sense.

When asked, ChatGPT confirmed using the `GRANT ALL PRIVILEGES` to a user over a single database does not open the server to running arbitrary OS commands:

> The privileges granted by GRANT ALL PRIVILEGES are limited to database-level and table-level operations within the MySQL server, such as SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, and other database-related actions.

This is exactly what we want. So we can limit access to mysql in the following ways:

- Create unique users for specific purposes
- Specify source host(s) a user can connect from (for example set root to only authenticate from localhost)
- Limit a user to a single database
- Limit a user to a single table within that database
- Limit the user's actions on databases and tables


### Enumerating MySQL Users

Let's take a moment to get a list of all users (from the built in mysql.user database):

> ```mysql
> SELECT User, Host FROM mysql.user;
> ```

If we want to review the privileges for any specific user:

> ```mysql
> SHOW GRANTS FOR 'new_user'@'localhost';
> ```


### OS Command Execution

The [sqlmap wiki details the requirements for these types of attacks](https://github.com/sqlmapproject/sqlmap/wiki/Usage#operating-system-takeover) via sql injection.

According to the wiki, command injection to OS takeover requires:

- MySQL / PostgreSQL: `sys_exec()` or `sys_eval()` UDF
	- This UDF is uploaded by sqlmap if the user can write to the underlying filesystem
- MSSQL: xp_cmdshell

This means there are two configuration settings that can open your server to command injection:

- User-Defined Functions (UDF's)
- Stored-Procedures

From ChatGPT, UDF's are not enabled by default:

> Configure MySQL to allow UDFs: By default, UDF support is disabled in MySQL for security reasons. You need to modify the MySQL configuration to enable UDF loading. This involves adding or modifying the plugin-load or plugin-dir configuration option to specify the location of the UDF library.

> Creating User-Defined Functions (UDFs) requires additional privileges beyond the "CREATE" privilege. By default, creating UDFs is limited to users with the SUPER privilege, which is typically reserved for the MySQL root user or administrators.

- [MySQL Docs: --allow-suspicious-udfs](https://dev.mysql.com/doc/refman/8.0/en/server-options.html#option_mysqld_allow-suspicious-udfs)
- [MySQL Docs: Loadable Function Security Precautions](https://dev.mysql.com/doc/extending-mysql/8.0/en/adding-loadable-function.html#loadable-function-security)

As for Stored-Procedures, they are available but would require a custom UDF to execute OS commands:

> To confirm that stored procedures cannot execute OS commands in your local MySQL instance, you can try creating a simple stored procedure that attempts to execute an OS command and observe the behavior.
>
> Here's an example of a stored procedure that attempts to execute an OS command using the sys_exec UDF (User-Defined Function), assuming you have the UDF installed:
>
> ```sql
> DELIMITER //
>
> CREATE PROCEDURE test_sp()
> BEGIN
>     DECLARE result INT;
>     SET result = sys_exec('ls -l');
>     SELECT result;
> END //
>
> DELIMITER ;
> ```
>
> In this example, the stored procedure attempts to execute the ls -l command using the sys_exec UDF. The result of the execution is stored in the result variable and then selected.
>
> However, please note that the sys_exec UDF is not a built-in function in MySQL, and it requires additional configuration and installation steps. By default, MySQL does not provide a built-in mechanism to execute OS commands.
>
> If you try to execute the above stored procedure and receive an error message indicating that the sys_exec function is not found or recognized, it confirms that stored procedures alone cannot execute OS commands in your MySQL instance without additional UDFs or custom configurations.


## Filesystem Write Privileges

So how can we confirm whether or not a user has filesystem write permissions? We'll need to review the `File_priv` privilege for each user.

On MySQL Ver 8.0.33-0ubuntu0.20.04.2, this privilege is found under the `mysql.user` database:

- [MySQL 8: File_priv user privilege](https://dev.mysql.com/doc/refman/8.0/en/privileges-provided.html#priv_file)

```sql
mysql> SELECT File_priv, user FROM mysql.user;
+-----------+------------------+
| File_priv | user             |
+-----------+------------------+
| Y         | debian-sys-maint |
| N         | mysql.infoschema |
| N         | mysql.session    |
| N         | mysql.sys        |
| N         | new_user         |
| Y         | root             |
+-----------+------------------+
6 rows in set (0.00 sec)
```

*The Y or N indicates whether or not the user has filesystem write privileges.*

If we're being this thorough, we should attempt to write a file via the mysql shell to test this. We'll use this statement from ChatGPT to write the string "test" to the file "/tmp/test.txt":

> ```sql
> SELECT 'test' INTO OUTFILE '/tmp/test.txt';
> ```

Attempting to do this from any account will alert the user that `--secure-file-priv` is set (apparently by default).

- [MySQL 8: sysvar_secure_file_priv](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_secure_file_priv)

*"If set to the name of a directory, the server limits import and export operations to work only with files in that directory. The directory must exist; the server does not create it."*

To check this variable, ChatGPT provides us the query to dump system variables for us to review:

> ```sql
> mysql> show variables like 'secure_file_priv';
> +------------------+-----------------------+
> | Variable_name    | Value                 |
> +------------------+-----------------------+
> | secure_file_priv | /var/lib/mysql-files/ |
> +------------------+-----------------------+
> 1 row in set (0.00 sec)
> ```

Let's try again, this time writing the file to `/var/lib/mysql-files/test.txt`:

```sql
mysql> SELECT 'test' INTO OUTFILE '/var/lib/mysql-files/test.txt';
Query OK, 1 row affected (0.00 sec)
```

Checking this directory (which requires root), we can confirm `test.txt` was written:

```bash
root@lamp:~# cd /var/lib/mysql-files/
root@lamp:/var/lib/mysql-files# ls -l
total 4
-rw-r----- 1 mysql mysql 5 May 26 01:53 test.txt
root@lamp:/var/lib/mysql-files# cat test.txt
test
```

Repeating this with our low privileged user, we can confirm it fails as expected:

```sql
mysql> SELECT 'test' INTO OUTFILE '/var/lib/mysql-files/test2.txt';
ERROR 1227 (42000): Access denied; you need (at least one of) the FILE privilege(s) for this operation
```


## SQL Scripting

Repeating the above automatically can be done by writing the same statements to a SQL script file. This file can be called from the mysql shell to run commands programmatically.

The following example contains the setup we walked through above:

```sql
-- init.sql setup script

-- Create the initial users database for testing

CREATE DATABASE LOGIN_TEST;

USE LOGIN_TEST;

CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(255) NOT NULL,
    password VARCHAR(255) NOT NULL
);

INSERT INTO users (username, password) VALUES ('testuser', 'testpassword');

INSERT INTO users (username, password) VALUES ('admin', 'WDt0qAOYZiOotpPa8sGjuBvQfQGTeo9H');


-- Create a low privileged user

CREATE USER 'new_user'@'localhost' IDENTIFIED WITH mysql_native_password BY 'new_password1';


-- Give new_user privileges to access and modify everything within our LOGIN_TEST database

GRANT ALL PRIVILEGES ON LOGIN_TEST.* TO 'new_user'@'localhost';
```

Execute a sql script like this:

```bash
mysql -u root -p < /path/to/script.sql
```

## Configuring MySQL

Your php configuration files will be in the following paths (Ubuntu 20.04):

- `/etc/php/7.4/cli/php.ini`
- `/etc/php/7.4/apache2/php.ini`

Each relates to the process of the folder name, so we'll be focusing on `apache2/php.ini`.

We'll need to ensure these extensions are enabled in `php.ini`:

- `extension=mysqli` allows mysql communication via php
- `extension=pdo_mysql` allows creating pdo instances with mysql


## Vulnerable SQLi PHP Login Page

The following PHP login page was generated by ChatGPT after a few iterations. It contains two sections of code relating to different types of SQL injection:

- Authentication Bypass
- Information Disclosure / Data Exfiltration

The following prompt was used to get started:

*"Hey ChatGPT, I have a LAMP stack running on an ubuntu 20.04 desktop. I need to create a simple login page in PHP that connects to the locally running MySQL instance. It needs to be vulnerable to SQL injection so I can understand web application security and log review. Could you help me write the code for the login page?"*

The php code below contains the original generation from ChatGPT, with the added changes from subsequent prompts of separating the "data returned" section into two code blocks to demonstrate two vulnerabilities using the same SQL injection, as well as a few comment changes to better match the same script shown later in this guide where we fix SQL injection.

> ```php
> <?php
> // Handle form submission
> if ($_SERVER["REQUEST_METHOD"] == "POST") {
>
>     // Retrieve the user input
>     $username = $_POST["username"];
>     $password = $_POST["password"];
>
>     // Connect to MySQL database
>     $conn = mysqli_connect("127.0.0.1", "root", "new_password2", "LOGIN_TEST");
>
>     // Check connection
>     if (!$conn) {
>         die("Connection failed: " . mysqli_connect_error());
>     }
>
>     // SQL injection vulnerability (DO NOT USE IN PRODUCTION)
>     $query = "SELECT * FROM users WHERE username = '$username' AND password = '$password'";
>
>     // Perform query
>     $result = mysqli_query($conn, $query);
>
>     // Process the results
>
>     // This block emulates authenticaiton bypass
>     if (mysqli_num_rows($result) == 1) {
>         echo "Login successful!";
>
>     // This block emulates information disclosure, allowing dumping of the usernames and passwords
> //    if ($result && mysqli_num_rows($result) > 0) {
> //        echo "Login successful!";
> //        while ($row = mysqli_fetch_assoc($result)) {
> //            // Print or process the retrieved data here
> //            echo "<br>";
> //            echo "Username: " . $row['username'] . "<br>";
> //            echo "Password: " . $row['password'] . "<br>";
> //            echo "<br>";
> //        }
>
>     } else {
>         echo "Invalid username or password.";
>     }
>
>     // Close the database connection
>     mysqli_close($conn);
> }
>
> ?>
>
> <!DOCTYPE html>
> <html>
> <head>
>     <title>Login Page</title>
> </head>
> <body>
>     <h2>Login</h2>
>     <form method="POST" action="">
>         <label for="username">Username:</label>
>         <input type="text" id="username" name="username" required><br><br>
>
>         <label for="password">Password:</label>
>         <input type="password" id="password" name="password" required><br><br>
>
>         <input type="submit" value="Login">
>     </form>
> </body>
> </html>
> ```

Next you'll want Apache to serve this file. ChatGPT once again provides suggested steps. `login.php` was changed to `login-vuln.php` for the sake of this guide:

> Create a new file called login.php or any other desired name:
>
> ```bash
> sudo nano login-vuln.php
> ```
>
> This will open the file in the nano text editor. You can replace nano with your preferred text editor.
>
> Copy the PHP code for the login page into the login-vuln.php file and save it.

...

> Note: If you make any changes to the PHP file, you might need to restart the Apache service for the changes to take effect. You can do this by running:
>
> ```bash
> sudo service apache2 restart
> ```


## Testing SQLi

Browse to `http://127.0.0.1:80/login-vuln.php`.


## Authentication Bypass

Enter `' or 1=1-- -` into the `Username:` field, and anything into the `Password:` field to perform the first sql injection attack, in this case authentication bypass.

This is because the line `if (mysqli_num_rows($result) > 0) {` will match any amount of returned rows and evaluate successfully. This is a pretty big logic flaw, so we can change this to match only a single row to be more precise like this:

> ```php
> if (mysqli_num_rows($result) == 1) {
> ```

*Be sure to run `sudo systemctl restart apache2` once you finish modifying any live webpages.*

Our payload will no longer work, but the application is *still vulnerable* to sql injection. We need a payload to match only a single row.

To do this, lets make an educated guess as to what usernames exist, and try to run sql injection to bypass authentication for that user.

Enter `' or username = 'admin'-- -` into the `Username:` field, and anything into the `Password:` field to perform the second sql injection attack, authentication bypass, to login as the user `admin`.


## Data Exfiltration

While the php code we've been using is vulnerable to sql injection resulting in authentication bypass, this is rather difficult to illustrate how dumping a database manually can work.

What we need is the web application to also print something back to us from the database.

So comment (`//`) these lines to look like:

> ```php
> //    if (mysqli_num_rows($result) == 1) {
> //        echo "Login successful!";
> ```

And uncomment these lines:

> ```php
>     if ($result && mysqli_num_rows($result) > 0) {
>         echo "Login successful!";
>         while ($row = mysqli_fetch_assoc($result)) {
>             // Print or process the retrieved data here
>             echo "<br>";
>             echo "Username: " . $row['username'] . "<br>";
>             echo "Password: " . $row['password'] . "<br>";
>             echo "<br>";
>         }
>
> ```

*Be sure to run `sudo systemctl restart apache2` once you finish modifying any live webpages.*

---

### curl

The following curl payload will perform the injection, dumping ALL usernames and passwords from the current database:

```bash
curl -X POST 'http://127.0.0.1:80/login-vuln.php' -d 'username=test%27 OR 1=1;-- -&password=test'

Login successful!<br>Username: testuser<br>Password: testpassword<br><br><br>Username: admin<br>Password: WDt0qAOYZiOotpPa8sGjuBvQfQGTeo9H<br><br>
```

Alternatively you can simply paste `' or 1=1-- -` into the username form via a web browser.


### sqlmap

You can use sqlmap to test all of this as well.

Download the latest from GitHub:

```bash
git clone --depth 1 https://github.com/sqlmapproject/sqlmap.git sqlmap-dev
```

Typically you'll want to be as surgical as possible with your sqlmap queries. This means making the minimal amount of requests and dumping precisely the data you're after. I've put together steps I've learned from various trainings, resources, and CTF's that reliably step through that.

Use the following steps after finding a vulnerable parameter to walk through and dump a database:

- [`--current-user`](https://github.com/sqlmapproject/sqlmap/wiki/Usage#session-user) obtains the current mysql user context
- [`--is-dba`](https://github.com/sqlmapproject/sqlmap/wiki/Usage#detect-whether-or-not-the-session-user-is-a-database-administrator) checks whether the user is a database admin
- [`--dbs`](https://github.com/sqlmapproject/sqlmap/wiki/Usage#list-database-management-systems-databases) enumerate databases
- [`-D <database> --tables`](https://github.com/sqlmapproject/sqlmap/wiki/Usage#enumerate-databases-tables) enumerate tables
- [`-D <database> -T <table> --columns`](https://github.com/sqlmapproject/sqlmap/wiki/Usage#enumerate-database-table-columns) enumerate columns
- [`-D <database> -T <table> -C <column1>,<column2> --dump`](https://github.com/sqlmapproject/sqlmap/wiki/Usage#dump-database-table-entries) dump the columns data

To run sqlmap against this LAMP stack:

```bash
python3 ./sqlmap.py -u 'http://127.0.0.1:80/login-vuln.php' --data='username=TEST&password=TEST' -p "username" --param-del="&" --method=POST -f --banner -v 3
```

## Logging SQLi

Now that we have a testing environment and have confirmed we can successfully perform sql injection, how could we review this from a defender's perspective? What would they see? What would they need to know? To do this, we need to enable logging for each component.


## Logging in Apache

Apache logs are enabled by default, and can be a great way to observe various web attacks, including but not limited to:

- [Brute Forcing](https://owasp.org/www-community/attacks/Brute_force_attack)
- [SQL injection](https://owasp.org/www-community/attacks/SQL_Injection)
- [Directory Busting](https://owasp.org/www-community/attacks/Forced_browsing)
- [Directory traversal](https://owasp.org/www-community/attacks/Path_Traversal)
- [Command injection](https://owasp.org/www-community/attacks/Command_Injection)
- [Log poisoning](https://owasp.org/www-community/attacks/Log_Injection)
- [XSS](https://owasp.org/www-community/attacks/xss/)

The logs we're interested in are located at (Ubuntu 20.04):

- `/var/log/apache2/access.log`
- `/var/log/apache2/error.log`

To get started, simply tail these logs while testing:

```bash
tail -f /var/log/apache2/access.log
```

You can pipe the `tail` command over to `grep` to pattern match on common payloads, non-standard user agents, or accessing directories blocked in a `robots.txt`. You could also setup a cron job to review the logs looking for common and open source payload strings, and send an alert via your preferred method if it finds any.


## Logging in MySQL

- [The General Query Log](https://dev.mysql.com/doc/refman/8.0/en/query-log.html)
- [Log Destinations](https://dev.mysql.com/doc/refman/8.0/en/log-destinations.html)
- [Log Table Benefits](https://dev.mysql.com/doc/refman/8.0/en/log-destinations.html#log-destinations-tables)

MySQL can be configured to write logs to a file, or into a log database to be accessible by mysql queries.

Cross referencing the documentation with ChatGPT, we can enable general query logging to capture all database connections, operations, and queries by appending the following to `my.cnf`:


### Logging to a File

- ✅ Logging to a file is outside of mysql (in case of crash or data corruption)
- ✅ Plain text logs can be read easily by other applications
- ✅ Files permissions can limit read access
- ✅ Rotating log files are handled by the system automatically
- ❌ No formatting (no csv, xml, etc)
- ❌ Reviewing file logs can be slower than using MySQL queries on a log table
- ❌ File logging can consume more disk space

> Determine the default configuration file location: Run the following command to find the default configuration file location:
>
> ```bash
> mysql --help | grep "Default options" -A 1
> Default options are read from the following files in the given order:
> /etc/my.cnf /etc/mysql/my.cnf ~/.my.cnf
> ```

This means we can write to any of the following:

- `/etc/my.cnf`
- `/etc/mysql/my.cnf`
- `~/.my.cnf`

Since `/etc/my.cnf` does not exist in Ubuntu 20.04, we'll start with `/etc/mysql/my.cnf`.

> ```bash
> sudo nano /etc/mysql/my.cnf
> ```

Appending these lines to the `my.cnf` configuration will enable general query logging to `/var/log/mysql/general.log`:

> ```conf
> [mysqld]
> log_output = FILE
> general_log = 1
> general_log_file = /var/log/mysql/general.log
> ```

Restart `mysql.service`:

> ```bash
> sudo systemctl restart mysql.service
> ```

---


### Rotating File Logs

In the default configuration (for Ubuntu 20.04), logrotate will:

- Write 1 log file per day
- Maintain 7 days of logs

Configuration path: `/etc/logrotate.d/mysql-server`

You could change this configuration to rotate every 30, 60, 90 days etc.

---


### Reading File Logs

To read or follow these logs you can simply tail them as root:

```bash
sudo tail -f /var/log/mysql/general.log
```

---


### Logging to a Table

- ✅ Logging to a table may be more efficient
- ✅ Formatting for csv, xml
- ✅ Can be read by MySQL users without OS access
- ✅ Can be exported for archival purposes
- ❌ No built in rotation mechanism
- ❌ Harder to parse with external tools
- ❌ Can potentially slow down the server under heavy strain

In MySQL 8 there's a built in logging database ready to use at `mysql.general_log`. All you need to do is enable (or add) database logging.

To log only to the `mysql.general_log` table:

```conf
[mysqld]
log_output = TABLE
general_log = 1
```

To log to both the table and a file:

```conf
[mysqld]
log_output = TABLE,FILE
general_log = 1
general_log_file = /var/log/mysql/general.log
```

Like before, restart `mysql.service`:

> ```bash
> sudo systemctl restart mysql.service
> ```

---


### Rotating Table Logs

There is no built in method or mechanism for easily rotating table logs. Attempting to modify mysql.general_log directly will result in the error `ERROR 1556 (HY000): You can't use locks with log tables.`. Instead there are two options:

- Create a separate log table under mysql to maintain and rotate logs
- Write logs to a file and rotate that file

Since we already have file logging available, we'll do this entirely within a table.

Edit `/etc/mysql/my.cnf` to add:

> ```conf
> event_scheduler = ON
> ```

Restart `mysql.service`:

> ```bash
> sudo systemctl restart mysql.service
> ```

Confirm the event scheduler is running with:

> ```sql
> mysql> SHOW VARIABLES LIKE 'event_scheduler';
> +-----------------+-------+
> | Variable_name   | Value |
> +-----------------+-------+
> | event_scheduler | ON    |
> +-----------------+-------+
> 1 row in set (0.01 sec)
> ```

*NOTE: be sure your mysql user has the EVENT privilege. If not, grant it as root with:*

> ```sql
> GRANT EVENT ON your_database.* TO 'username'@'localhost';
> ```

Create a new table to maintain our mysql logs, matching the structure of the existing `mysql.general_log` table we'll be pulling from:

> ```sql
> CREATE TABLE mysql.general_log_rotated LIKE mysql.general_log;
> ```

Next add the event itself. This example event will:

- Create the `rotate_logs_event` event within the `mysql` database
- Run every minute
- Copy every entry from `mysql.general_log` to `mysql.general_log_rotated` from up to one minute ago
- `TRUNCATE` (wipe) the mysql.general_log table
- `DELETE` log entries under mysql.general_log_rotated older than 5 minutes:
- `MINUTE` can be swapped for `HOUR`, `DAY`, `WEEK`, `MONTH`, etc.

> ```sql
> DELIMITER //
> CREATE EVENT rotate_logs_event
> ON SCHEDULE EVERY 1 MINUTE
> DO
> BEGIN
>   INSERT INTO mysql.general_log_rotated SELECT * FROM mysql.general_log WHERE event_time >= (NOW() - INTERVAL 1 MINUTE);
>   TRUNCATE TABLE mysql.general_log;
>   DELETE FROM mysql.general_log_rotated WHERE event_time < (NOW() - INTERVAL 5 MINUTE);
> END//
> DELIMITER ;
> ```

Essentially, every minute it's moving the latest default logs over to our (rotatable) custom log table, and maintaining the most recent 5 minutes worth of logs in that custom log table.

With the above event block as a template you can see how to:

- Write an event block with multiple statements
- Change the time interval for each component

---

Now we can dump all of the event's information from the `information_schema.events`:

> ```sql
> SELECT * FROM information_schema.events WHERE EVENT_NAME = 'rotate_logs_event';
> ```

To get a clearer view of some important values in the event:

```sql
SELECT EVENT_NAME, STATUS, INTERVAL_VALUE, INTERVAL_FIELD, EVENT_TYPE, LAST_EXECUTED FROM information_schema.events WHERE EVENT_NAME = 'rotate_logs_event';
```

To dump only the event configuration (the sql statement block):

```sql
SELECT EVENT_DEFINITION FROM information_schema.events WHERE EVENT_NAME = 'rotate_logs_event';
```

If you need to revise an event you must delete it and create it again:

- The `database_name` relates to the database where the event runs, not information_schema
- In this example, `mysql` is the database and `rotate_logs_event` is the name of the event we created.

> ```sql
> DROP EVENT database_name.event_name;
> DROP EVENT mysql.rotate_logs_event;
> ```

---


### Reading Table Logs

*Note that when reading logs you can print them in a vertical format, which may be helpful.*

From the mysql man page:

`--vertical`, `-E` Print query output rows vertically (one line per column value). Without this option, you can specify vertical output for individual statements by terminating them with `\G`.

To read or query these logs you'll need to decode the (mediumblob type) arguments into a human readable format. To do this, we could use one of two methods:

- `sys.format_statement(<column_name>)`
- `CAST(<column_name> AS CHAR)`

We'll use the CAST method as sys.format_statement truncates the columns. This query was modified slightly from ChatGPT's suggestion:

> ```sql
> mysql> SELECT event_time, user_host, CAST(argument AS CHAR) AS statement FROM mysql.general_log;
> ```

Now we can see all of the arguments. However this may not all be useful. If we want to see only connections to the database via the web application, we can filter based on sources addresses that arent' empty. This will:

- Return only input ingested by the web application
- Filter out most of our own local queries within the mysql shell

> ```sql
> mysql> SELECT event_time, user_host, CAST(argument AS CHAR) AS statement FROM mysql.general_log WHERE user_host NOT LIKE '% []';
> ```

This shows us any connection from the web application is processed and coming from the localhost address where the web app is running. It's obvious that correlating source IPs to each log entry will be a task all it's own.

We'll substitute `WHERE user_host NOT LIKE '% []'` for `WHERE user_host LIKE '% [127.0.0.1]'` now that we know `localhost` is what we want to match on.

Now, say we want to search for sql injection. Specifically any occurrances of `or 1=1`:

> ```sql
> mysql> SELECT event_time, user_host, CAST(argument AS CHAR) AS statement FROM mysql.general_log WHERE user_host LIKE '% [127.0.0.1]' AND argument LIKE '%or 1=1%';
> ```

If you injected that payload into the web app after table logging was enabled, you'll see it here in the results.

Next we can search for any sql statements containing the string `admin`:

> ```sql
> mysql> SELECT event_time, user_host, CAST(argument AS CHAR) AS statement FROM mysql.general_log WHERE user_host LIKE '% [127.0.0.1]' AND argument LIKE '%admin%';
> ```

`%` is a wild card in pattern matching with sql statements. Use this to build bulk queries to find sql injection or other malicious patterns in your sql logs.


### Additional Logging

You'll notice throughout this section that the general logging function will record any statement mysql runs, including user login credentials in plain text.

Some additional resources on logging:

- [Installing MySQL Enterprise Audit](https://dev.mysql.com/doc/refman/8.0/en/audit-log-installation.html)
- [Audit Log Encryption](https://dev.mysql.com/doc/refman/8.0/en/audit-log-logging-configuration.html#audit-log-file-encryption)
- [Passwords and Logging](https://dev.mysql.com/doc/refman/8.0/en/password-logging.html)


## Logging in PHP

- [snyk.io: Prevent PHP Code Injection](https://snyk.io/blog/prevent-php-code-injection/)

Unfortunately there's no easy way to do full logging of all PHP activity on the server. If you want something more robust here you'll need to leverage `auditd` or `Sysmon`. An example using auditd to do this is below.


### PHP Auditd Logging

Add this line to your custom rule, for example file under `/etc/audit/rules.d/40-your-custom.rules`:

```conf
-w /usr/bin/php -p x -k T1059_Command_Scripting_PHP
```

Search for entries with this:

```bash
sudo ausearch -ts today -i -l -x php | grep 'proctitle='
```

*Keep in mind this may generate a ridiculous number of log entries on a busy server. Plan to tune and test this for a while.*


### PHP Built In Logging

PHP's built in logging is able to record errors to a file. This is also important as it touches on server configurations that attackers abuse to display errors directly to webpages.

First lets make a directory for php error logging:

```bash
sudo mkdir -m 750 /var/log/php
sudo chown root:adm /var/log/php
```

You can use `php -r 'phpinfo();' | grep php.ini` or `php --ini` to check your current .ini paths for the command line (cli).

To confirm the ini path used by apache, write a file named `test.php` to `/var/www/html` with the following contents:

> ```php
> <?php
> phpinfo();
> ?>
> ```

Edit the ini file loaded by apache:

> ```bash
> sudo nano /etc/php/7.4/apache2/php.ini
> ```

Write the following lines:

> - `error_reporting = E_ALL`
> - `error_log = /var/log/php/error.log`
> - `display_errors = 0`

After any configuration changes, restart apache:

```bash
sudo systemctl restart apache2
```

These logs are written in plain text and can be read with `tail -f`.

---

### PHP Log Rotation

You will need to create your own rotation script under `/etc/logrotate.d` to handle this file.

This code block is modified from my [unbound-update-blocklist.sh](https://github.com/straysheep-dev/linux-configs/blob/main/dns/unbound-update-blocklist.sh) script:

```bash
LOGFILE='php'

# Create the logging configuration if it doesn't exist
if ! [ -e /etc/logrotate.d/"$LOGFILE" ]; then
	echo "# Logging configuration for $LOGFILE
/var/log/$LOGFILE {
	nocompress
	monthly
	rotate 4
	missingok
	notifempty
	su root root
	create 0644 root root
}" > /etc/logrotate.d/"$LOGFILE"
fi
```


## Fixing SQL Injection

- [OWASP: Using PHP with Data Objects](https://cheatsheetseries.owasp.org/cheatsheets/Query_Parameterization_Cheat_Sheet.html#using-php-with-php-data-objects)

When asked to revise our script from earlier, ChatGPT was able to provide a template creating a PDO (PHP Data Object) instance, and paramaterizing the queries.

- PDO is it's own module, separate from the mysqli module, so we connect to mysql using `new PDO()`
- Placeholders are created for input values using `:username` and `:password`
- The query is prepare for execution with `$pdo->prepare($query)`

From ChatGPT:

> In essence, the $pdo->prepare($query) line sets up the SQL query for execution, but it doesn't actually execute the query at that point. It allows you to reuse the prepared statement object with different parameter values or execute it multiple times efficiently.

- Input values are bound to variables with `$statement->bindParam(':username', $username);`

> In the context of SQL queries, placeholders are used as a way to include dynamic values in the query while avoiding SQL injection vulnerabilities. By using placeholders, you separate the SQL code from the actual values, and the database driver handles the proper escaping and handling of the values.

Lastly we can close the connection to the PDO with `$pdo = null;`. With that, we have our login.php script but using prepared statements:

> ```php
> <?php
> // Handle form submission
> if ($_SERVER["REQUEST_METHOD"] == "POST") {
>
>     // Retrieve the user input
>     $username = $_POST['username'];
>     $password = $_POST['password'];
>
>     // Create a new PDO instance (The PDO, or PHP Data Object module is different from the mysqli module)
>     try {
>         $pdo = new PDO('mysql:host=127.0.0.1;dbname=LOGIN_TEST', 'root', 'new_password2');
>
>     // Check connection
>     } catch (PDOException $e) {
>         // This echoes any errors to the web page, do not use in production
>         die("Connection failed: " . $e->getMessage());
>     }
>
>     // Prepare the SQL statement with placeholders
>     $query = "SELECT * FROM users WHERE username = :username AND password = :password";
>     $statement = $pdo->prepare($query);
>
>     // Bind the input values to the prepared statement parameters
>     $statement->bindParam(':username', $username);
>     $statement->bindParam(':password', $password);
>
>     // Execute the prepared statement
>     $statement->execute();
>
>     // Process the results
>
>     // Get the number of rows affected by the executed statement
>     $rowCount = $statement->rowCount();
>
>     // This block emulates authentication bypass
> //    if ($rowCount == 1) {
> //        echo "Login successful!";
>
>     // This block emulates information disclosure, allowing dumping of the usernames and passwords
>     if ($rowCount == 1) {
>         echo "Login successful!";
>         $rows = $statement->fetchAll(PDO::FETCH_ASSOC);
>         foreach ($rows as $row) {
>             // Print or process the retrieved data here
>             echo "<br>";
>             echo "Username: " . $row['username'] . "<br>";
>             echo "Password: " . $row['password'] . "<br>";
>             echo "<br>";
>         }
>     } else {
>         echo "Invalid username or password.";
>     }
>
>     // Close the database connection
>     $pdo = null;
> }
>
> ?>
>
> <!DOCTYPE html>
> <html>
> <head>
>     <title>Login Page</title>
> </head>
> <body>
>     <h2>Login</h2>
>     <form method="POST" action="">
>         <label for="username">Username:</label>
>         <input type="text" id="username" name="username" required><br><br>
>
>         <label for="password">Password:</label>
>         <input type="password" id="password" name="password" required><br><br>
>
>         <input type="submit" value="Login">
>     </form>
> </body>
> </html>
> ```

Let's test this once more with the same payload for authentication bypass:

```bash
curl -X POST http://127.0.0.1:80/login-safe.php -d 'username=%27 or username = %27admin%27-- -&password=test'

Invalid username or password.
```

Is this perfect? Probably not, but we now have a local instance of this web application running where we can modify the application logic on the fly to test. It's an exercise to the reader (and the point of this guide) to test various payloads against variations of the login page and understand what changes in the application code do to the security of the website.


## Vulnerable PHP Path Traversal Page

- [OWASP: Path Traversal](https://owasp.org/www-community/attacks/Path_Traversal)

We'll create two files, one that's vulnerable named `read-vuln.php`, and one that's patched named `read-safe.php`.


## Testing Path Traversal

This script shows the directory contents of the `/var/www/html` folder and allows the user to enter (or click on) a filename to read it's contents for debugging.

It takes `?file=<file>` as an argument.

read-vuln.php:

> ```php
> <?php
>     // Set the base directory where files are stored
>     $baseDir = '/var/www/html/';
>
>     // Get the list of files in the base directory
>     $files = scandir($baseDir);
>
>     // Remove '.' and '..' from the list
>     $files = array_diff($files, array('.', '..'));
>
>     // Get the requested file from the URL parameter
>     $file = $_GET['file'];
>
>     // Concatenate the base directory with the requested file
>     $filePath = $baseDir . $file;
>
>     // Read and output the contents of the file
>     $fileContents = file_get_contents($filePath);
> ?>
>
> <!DOCTYPE html>
> <html>
> <head>
>     <title>Directory Listing</title>
> </head>
> <body>
>     <h1>Directory Listing</h1>
>     <ul>
>         <?php foreach ($files as $file) : ?>
>             <li><a href="?file=<?= urlencode($file) ?>"><?= $file ?></a></li>
>         <?php endforeach; ?>
>     </ul>
>
>     <hr>
>
>     <h2>Read File</h2>
>     <p>Enter (or click on) a filename to review its contents.</p>
>     <form method="GET" action="">
>         <input type="text" name="file" placeholder="Enter file name">
>         <input type="submit" value="Read">
>     </form>
>
>     <pre><?= htmlentities($fileContents) ?></pre>
> </body>
> </html>
> ```

We can exploit this with the following curl command:

```bash
curl --path-as-is 'http://127.0.0.1:80/read-vuln.php?file=../../../../../etc/passwd'
```

Alternatively you can do this by entering `../../../../../etc/passwd` into the browser form.

---

## Fixing Path Traversal

Changing the code to the following will prevent path traversal.

read-safe.php:

> ```php
> <?php
>     // Specify the base directory where files are stored
>     $baseDir = '/var/www/html/';
>
>     // Get the list of files in the base directory
>     $files = scandir($baseDir);
>
>     // Remove '.' and '..' from the list
>     $files = array_diff($files, array('.', '..'));
>
>     // Handle the file parameter if provided
>     if (isset($_GET['file'])) {
>         // Get the requested file from the URL parameter
>         $file = $_GET['file'];
>
>         // Sanitize the file path to prevent directory traversal
>         $filePath = realpath($baseDir . $file);
>
>         // Check if the file exists and is within the base directory
>         if ($filePath !== false && strpos($filePath, $baseDir) === 0) {
>             // Read and output the contents of the file
>             $fileContents = file_get_contents($filePath);
>         } else {
>             // Invalid file path
>             $fileContents = 'Invalid file.';
>         }
>     } else {
>         // No file parameter provided
>         $fileContents = '';
>     }
> ?>
>
> <!DOCTYPE html>
> <html>
> <head>
>     <title>Directory Listing</title>
> </head>
> <body>
>     <h1>Directory Listing</h1>
>     <ul>
>         <?php foreach ($files as $file) : ?>
>             <li><a href="?file=<?= urlencode($file) ?>"><?= $file ?></a></li>
>         <?php endforeach; ?>
>     </ul>
>
>     <hr>
>
>     <h2>Read File</h2>
>     <p>Enter (or click on) a filename to review its contents.</p>
>     <form method="GET" action="">
>         <input type="text" name="file" placeholder="Enter file name">
>         <input type="submit" value="Read">
>     </form>
>
>     <pre><?= htmlentities($fileContents) ?></pre>
> </body>
> </html>
>
> ```

The example above checks to be sure the file uses the intended `/var/www/html` as it's base directory. This could be improved further by assigning an index to every file you intend to be readable by the user. In that case the script will only accept inputs in the form of an index ID that relates to a filename rather than attempting to sanitize arbitrary filenames.


## Docker LAMP

How to do everything we just did, but with Docker.

## Installing Docker

The installation commands here were taken directly from the Docker documentation for Ubuntu.

- [Installing Docker Using the Apt Repository](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository)

```bash
# Update the apt package index and install packages to allow apt to use a repository over HTTPS:
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg

# Add Docker’s official GPG key:
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Use the following command to set up the repository:
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine, containerd, and Docker Compose.
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```


## Separate Configuration Directories

You can use the same file paths on the host VM as you did for the real apache, php, and mysql application files. But for simplicity, why don't we copy those files into the current directory to use specifically for Docker.

```bash
cp -r /var/www .
mkdir php
cp /etc/php/7.4/apache2/php.ini ./php
```

This will be more useful later if we choose to use docker compose. But ultimately it changes the docker commands from these:

```bash
sudo docker run -d -p 80:80 --name apache-php-testing -v /var/www/html:/var/www/html:ro -v /etc/php/7.4/apache2/:/usr/local/etc/php:ro php:apache
sudo docker run -d -p 3306:3306 --name mysql-testing -e MYSQL_ROOT_PASSWORD=new_password2 mysql:latest
```

To these:

```bash
sudo docker run -d -p 80:80 --name apache-php-testing -v ./www/html:/var/www/html:ro -v ./php:/usr/local/etc/php:ro php:apache
sudo docker run -d -p 3306:3306 --name mysql-testing -e MYSQL_ROOT_PASSWORD=new_password2 mysql:latest
```


## Setting Up LAMP in Docker

***NOTE: Docker will insert iptables rules above your own firewall rules. This exposes your containers to every interface on the running VM unless you specify an interface, like `-p 127.0.0.1:3306:3306`. You should review any firewall configurations required and check all of your interfaces with nmap to understand your attack surface.***

The example commands below leave the containers exposed to every interface. This allows us to spin up another VM within the same subnet to access the applications as part of testing.

- [Docker Official Images](https://docs.docker.com/docker-hub/official_images/)
- [PHP Official Docker Hub Images](https://hub.docker.com/_/php)
- [MySQL Official Docker Hub Images](https://hub.docker.com/_/mysql)

Steps to get the LAMP components running:

```bash
sudo docker pull php:apache
sudo docker pull mysql:latest

sudo systemctl stop apache2
sudo systemctl stop mysql
sudo systemctl disable apache2
sudo systemctl disable mysql

sudo docker run -d -p 80:80 --name apache-php-testing -v ./www/html:/var/www/html:ro -v ./php:/usr/local/etc/php:ro php:apache
sudo docker run -d -p 3306:3306 --name mysql-testing -e MYSQL_ROOT_PASSWORD=new_password2 mysql:latest

sudo docker exec -it apache-php-testing docker-php-ext-install mysqli
sudo docker restart apache-php-testing
```

This will:

- Pull the offical MySQL and PHP + Apache container images to your local machine
- Stop any running processes from the packages installed via apt-get
- Run the docker containers detached as background processes
- Give the docker containers names
- Use the `docker-php-ext-install` tool built in to the php container to install the `mysqli` module for mysql functions
- Mount and map the php.ini files from the host path to the container's preferred path as read-only
- `php:apache` is mounting the hosts's `./php` directory to `/usr/local/etc/php` as read-only
- `php:apache` is mounting the host's `./www/html` (as read-only)
- `mysql:latest` is using the `MYSQL_ROOT_PASSWORD` specified for root
- `mysql:latest` is also using the host's `/etc/mysql/` dir (as read-only)

Next you should obtain shells within both containers:

```bash
sudo docker exec -it <name-of-the-container> bash
sudo docker exec -it apache-php-testing bash
sudo docker exec -it mysql-testing bash
```

This will let you look around the filesystems of each container. Most importantly, you can invoke `mysql -u root -p` from the mysql container. Provide the password you set on the command line with `MYSQL_ROOT_PASSWORD`, and you'll have database access as normal.

You can even connect to this database from the host machine, by specifying a target IP with `-h`, in this case localhost will work:

```bash
mysql -u root -p -h 127.0.0.1
```

Next steps:

- Configure the MySQL container's database(s) like we did for the host
- Confgire any additional logging like we did for the host
- Point the MySQL connection php script functions to the MySQL Docker container, by container name or IP address
	- `sudo docker network inspect <network>` to obtain container's IP address information
	- Localhost or 127.0.0.1 won't work anymore since the containers are separate
- Change the paths pointing to (and within) some configurations and logs
- Docker logging and monitoring with `sudo docker logs <container-name>` as well as application logging functions
- Repeat our tests against docker web applications and observe any differences


### Docker Essentials

[Download an image from the registry](https://docs.docker.com/engine/reference/commandline/pull/)

- `sudo docker pull <image-name>[:<tag>]`

[List container images](https://docs.docker.com/engine/reference/commandline/image_ls/)

- `sudo docker image ls`

[List container processes](https://docs.docker.com/engine/reference/commandline/ps/)

- `sudo docker ps`
- `sudo docker ps -a`

[Stop / start a running container](https://docs.docker.com/engine/reference/commandline/stop/)

- `sudo docker stop <container>`
- `sudo docker start <container>`
- `sudo docker restart <container>`

[Remove one or more containers (from the process list)](https://docs.docker.com/engine/reference/commandline/rm/)

- `sudo docker rm <container>`

[Remove one or more images (delete the image itself)](https://docs.docker.com/engine/reference/commandline/rmi/)

- `sudo docker rmi <image>`

[Run command in a container](https://docs.docker.com/engine/reference/commandline/exec/)

- `sudo docker exec [options] <container> <command> [args...]`
- `sudo docker exec php:7.4-apache cat /etc/passwd`

[Drop into a shell inside a running container](https://docs.docker.com/engine/reference/commandline/exec/)

- `sudo docker exec -it <container-name> bash`

[Execute commands within the container interactively](https://docs.docker.com/engine/reference/run/)

- `sudo docker run -it <container> <command>`

[Review Docker networking](https://docs.docker.com/engine/reference/commandline/network/)

> - `sudo docker network ls`
> - `sudo docker network inspect bridge`

[Show containers logs](https://docs.docker.com/engine/reference/commandline/logs/)

- `sudo docker logs <container>`
- This is incredibly useful for debugging issues with your containers

Configure Docker to start on boot

- `sudo systemctl enable docker.service`
- `sudo systemctl enable containerd.service`

Completely remove all docker container components, restart from scratch:

> - `sudo docker system prune -a`
- You would do this if building custom images and something goes wrong


## Debugging

*This section was written with the help of ChatGPT through a lot of trial and error. There are no direct quotes from GPT's output, but the responses and suggestions helped me pivot between each piece quickly, ultimately putting these exact steps together based on where the conversation led me.*

This section details an error I encountered while trying to connect the php-apache and mysql docker containers. The containers could "see" each other, confirmed by running `curl http://172.17.0.3:3306` from the php-apache container to the mysql container's docker IP address, but the `login.php` script wasn't connecting to the mysql container's database.

Tips:

- `sudo docker logs <container-name>` is incredibly helpful for debugging
- Creating a `phpinfo();` page to be served by apache will help you identify php configuration issues

What was happening:

- The php-apache container did not have the mysqli extension installed (this provides php the functionality to connect to mysql)
- This resulted in a 500 server error response
- Checking the docker logs with `sudo docker logs <php-apache-container-name>` revealed this was the case

```
PHP Fatal error:  Uncaught Error: Call to undefined function mysqli_connect() in /var/www/html/login.php
```

*This assumes you know `mysqli_connect()` requires the mysqli module to be installed. If not, googling mysqli_connect will reveal this.*

- The mysql container confirmed the mysqli module was not installed
- First by running a `find / -name "mysqli.so" -ls 2>/dev/null` from within the container's command line
- Then by querying `php -m` for a list of active modules
- Install the module with `sudo docker exec -it <mysql-container-name> docker-php-ext-install mysqli`
- You could also run `docker-php-ext-install mysqli` from within the container if you have an interactive shell there
- Setting up a `test.php` page with `<?php phpinfo(); ?>` and doing a Ctrl+f find for mysqli shows the configuration is active

With everything configured correctly, attempting the POST request again threw one more error:

```bash
curl -i -X POST http://127.0.0.1:80/login.php -d 'username=testuser&password=testpassword'

HTTP/1.1 200 OK
Date: Sun, 28 May 2023 23:42:56 GMT
Server: Apache/2.4.38 (Debian)
Vary: Accept-Encoding
Content-Length: 83
Content-Type: text/html; charset=UTF-8

Connection failed: The server requested authentication method unknown to the client
```

No more 500 response code! However we can see that the client and server are unable to communicate. According the ChatGPT, this is likely due to the mysqlnd module's available authentication methods. Recent versions of MySQL default to using `caching_sha2_password` as the `default_authentication_plugin`. This means we were using an outdated version of the php-apache container.

Checking the tags on PHP's docker registry shows us the latest default php:apache container (no version number) was released a few days ago.

Repeating the prior steps with an up to date container solves the problem. Be sure to:

- Change the name you assign to the new container, to something like php-apache-testing-2

If you're just stopping the previous php-apache container with `sudo docker stop <php-apache-container-name>` and not completely removing it from your currently available container processes with `sudo docker rm <php-apache-container-name>`, then:

- Until you remove it from the docker process list with `rm` it's still available to restart
- This is like a regular system process that has been stopped.
- Removing the container from the process list does not delete the container image
- You can spin the same container image up again without redownloading it

```bash
sudo docker stop apache-php-testing
sudo docker pull php:apache
sudo docker run -d -p 80:80 --name apache-php-testing2 -v /var/www/html:/var/www/html:ro -v /etc/php/7.4/apache2/:/usr/local/etc/php:ro php:apache
sudo docker exec -it apache-php-testing2 docker-php-ext-install mysqli
sudo docker restart apache-php-testing2
```

With all of that done, let's confirm we can still sql inject into the target:

```bash
curl -i -X POST http://127.0.0.1:80/login.php -d 'username=%27 or username=%27admin%27-- -&password=test'
HTTP/1.1 200 OK
Date: Mon, 29 May 2023 00:06:39 GMT
Server: Apache/2.4.56 (Debian)
Vary: Accept-Encoding
Content-Length: 471
Content-Type: text/html; charset=UTF-8

Login successful!
```

We now have the same technology stack and testing environment running in docker. Like before we have the flexibility of adding and modifying our own vulnerable code for testing purposes, with the unique ability of pinning each application to a specific version on the same VM. This will allow us to test current or older versions of the same application more easily.


## Docker Compose

This was a lot to figure out on the fly, but with ChatGPT helping us we can automate this further. Let's write everything we've done to files docker compose can read, as a way to "save" our work so we can easily come back to it later.

This will:

- Pull the latest php-apache and mysql containers
- Build the php-apache container with `docker-php-ext-install mysqli` executed automatically
- Build the mysql container with the `init.sql` script to automatically setup the mysql database and user
- Configure a Docker network subnet named `web-test-env` and assign static IPs to the containers
- Start both containers with all of the command line arguments we used above

*NOTE: Multiple `FROM` commands in a Dockerfile will cause errors during the build process. If you try to include `FROM` commands different docker images, the build will fail. Instead, this setup includes two different Dockerfiles that docker-compose.yml points to for each service, mysql and php-apache respectively.*

To test the `init.sql` script manually in the mysql container:

```bash
sudo docker exec -i <mysql-container-name> mysql -uroot -p<password> < <path_to_init.sql>
```

The purpose and benefit of tagging your custom built images, is so they don't overwrite the official ones from Docker Hub that you may or may not have (and may or may not want to keep) locally:

```bash
# docker image ls
REPOSITORY   TAG                  IMAGE ID       CREATED          SIZE
php          custom-local-image   27155ecfffc7   16 seconds ago   565MB
mysql        custom-local-image   7089f74c8fb8   16 seconds ago   565MB
mysql        latest               05db07cd74c0   4 days ago       565MB
php          apache               c7e8ea333e39   5 days ago       460MB
```

### Running Everything with Docker Compose

Write the `docker-compose.yml`, `Dockerfile-mysql`, `Dockerfile-php-apache`, and `init.sql` files below to your current working directory before executing `docker compose`.

To get everything running, this will write container activity to the foreground:

```bash
sudo docker compose build # To build the custom container images
sudo docker compose up # To spin up the network and configured containers
sudo docker compose up -d # To spin up everything in the background
```

You can check however you like to see if the containers are running correctly. For example:

```bash
# Confirm mysqli was installed
docker exec -it apache-php-testing-custom-image php -m | grep mysqli

# Confirm our database was built
docker exec -i mysql-testing-custom-image mysql -uroot -p -e 'SELECT * FROM LOGIN_TEST.users;'

# Confirm SQL injection works
curl -X POST http://127.0.0.1:80/login.php -d 'username=test%27or username=%27admin%27-- -&password=test'
```

When you're done, spin everything down with:

```bash
# Ctrl+c in the active window to stop the containers, then
sudo docker compose down # to remove them
```

### Docker Configuration Files

All of the configuration files are included in this repo. They were put together in pieces through conversation with ChatGPT, additional comments were added from me for context.

docker-compose.yml:

> ```yaml
> version: '3'
> networks:
>   web-test-env:
>     ipam:
>       config:
>         - subnet: 172.20.0.0/16
> services:
>   php-apache:
>     # Builds the custom php:apache image using the specified Dockerfile
>     build:
>       context: .
>       dockerfile: Dockerfile-php-apache
>     # "image" should have a custom name since the accompanying Dockerfile is pulling the
>     # container we want from the public registry and building our changes locally.
>     # If the Dockerfile fails to build our custom image, this line won't attempt to
>     # search and pull whatever image name we write for "image:" from the public registry.
>     # Further, if the image name isn't unique, it will overwrite the default container
>     # named php:apache if we have pulled it previously and it exists locally.
>     image: php:custom-local-image
>     container_name: apache-php-testing-custom-image
>     networks:
>       web-test-env:
>         ipv4_address: 172.20.0.2
>     ports:
>       - 80:80
>     volumes:
>       - ./www/html:/var/www/html:ro
>       - ./php:/usr/local/etc/php:ro
>
>   mysql:
>     # Builds the custom mysql image using the specified Dockerfile
>     # This allows us to run the init.sql script to execute mysql commands
>     build:
>       context: .
>       dockerfile: Dockerfile-mysql
>     image: mysql:custom-local-image
>     container_name: mysql-testing-custom-image
>     networks:
>       web-test-env:
>         ipv4_address: 172.20.0.3
>     ports:
>       - 3306:3306
>     # Any other mysql configurations should be handled by the special init.sql file
>     environment:
>       - MYSQL_ROOT_PASSWORD=new_password2
> ```

Dockerfile-php-apache:

> ```Dockerfile
> # This file must be in the same directory as docker-compose.yml
>
> # This file is started from docker-compose.yml's "build:" lines and parameters
> # It's used to build a custom container images
>
> # Pull the php:apache image from Docker Hub
> FROM php:apache
>
> # Run this inside of the container to install the mysqli module
> RUN docker-php-ext-install mysqli
>
> # Install PDO extension and MySQL driver
> RUN docker-php-ext-install pdo pdo_mysql
>
> # You can add additional instructions if needed
> ```

Dockerfile-mysql:

> ```Dockerfile
> # This file must be in the same directory as docker-compose.yml
> # Each container image must have its own Dockerfile
>
> # This file is started from docker-compose.yml's "build:" lines and parameters
> # It's used to build a custom container images
>
> # Pull the mysql:latest image from Docker Hub
> FROM mysql:latest
>
> # Set the MYSQL_ROOT_PASSWORD variable here or the build will fail
> ENV MYSQL_ROOT_PASSWORD=new_password2
>
> # Copy the SQL script to the docker-entrypoint-initdb.d directory
> # This is a special directory for the MySQL docker container
> # It will execute all mysql commands within the init.sql script there
> COPY ./init.sql /docker-entrypoint-initdb.d/
>
> # You can add additional instructions if needed
> ```

init.sql:

> ```sql
> -- init.sql setup script
>
> -- Create the initial users database for testing
>
> CREATE DATABASE LOGIN_TEST;
>
> USE LOGIN_TEST;
>
> CREATE TABLE users (
>     id INT PRIMARY KEY AUTO_INCREMENT,
>     username VARCHAR(255) NOT NULL,
>     password VARCHAR(255) NOT NULL
> );
>
> INSERT INTO users (username, password) VALUES ('testuser', 'testpassword');
>
> INSERT INTO users (username, password) VALUES ('admin', 'WDt0qAOYZiOotpPa8sGjuBvQfQGTeo9H');
>
>
> -- Create a low privileged user
>
> CREATE USER 'new_user'@'localhost' IDENTIFIED WITH mysql_native_password BY 'new_password1';
>
>
> -- Give new_user privileges to access and modify everything within our LOGIN_TEST database
>
> GRANT ALL PRIVILEGES ON LOGIN_TEST.* TO 'new_user'@'localhost';
> ```


## Ansible LAMP

How to do everything we just did, but with Ansible.


## Install Ansible

- [Ansible python installation guide](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#pip-install)

You will likely need to do this as root.

```bash
python3 -m pip install --user ansible

# Upgrade via pip
python3 -m pip install --upgrade --user ansible

# Confirm ansible version
ansible --version
python3 -m pip show ansible
```

- [Ansible distro installation guide](https://docs.ansible.com/ansible/latest/installation_guide/installation_distros.html)

Install and manage Ansible with apt-get via the Ansible PPA.

```bash
sudo apt update ; \
sudo apt install software-properties-common ; \
sudo add-apt-repository --yes --update ppa:ansible/ansible ; \
sudo apt install ansible

# Confirm ansible version
ansible --version
```


## Write the Playbook (apt)

With only a few modifications made to ChatGPT's example playbook below, we have a complete playbook to deploy these components via apt. You could run the playbook as-is below on a host with Ansible installed to spin up the LAMP stack.

> Create a playbook file that defines the tasks for deploying the LAMP stack. For example, create a file named lamp.yml with the following content:
>
> ```yaml
> ---
> - name: Install LAMP stack
>   hosts: localhost
>   become: true
>
>   tasks:
>     - name: Update apt cache
>       apt:
>         update_cache: yes
>       become: true
>
>     - name: Install Apache
>       apt:
>         name: apache2
>         state: present
>       become: true
>
>     - name: Install MySQL
>       apt:
>         name: mysql-server
>         state: present
>       become: true
>
>     - name: Install PHP and required modules
>       apt:
>         name: "{{ item }}"
>         state: present
>       become: true
>       loop:
>         - php
>         - php-mysql
>
>     - name: Restart MySQL
>       service:
>         name: mysql
>         state: restarted
>
>     - name: Restart Apache
>       service:
>         name: apache2
>         state: restarted
> ```
>
> Run the playbook using the following command:
>
> ```bash
> ansible-playbook lamp.yml
> ```


## Write the Playbook (docker)

You will need the Docker SDK for Python, no matter how you install Ansible.

- [Ansible Docker Requirements](https://docs.ansible.com/ansible/latest/collections/community/docker/docsite/scenario_guide.html#requirements)

```bash
python3 -m pip install docker
```

A complete Docker playbook is beyond the scope of this guide, but the following resources have the relevant information on writing one:

- [Docker Guide](https://docs.ansible.com/ansible/latest/collections/community/docker/docsite/scenario_guide.html)
- [Docker Image Module](https://docs.ansible.com/ansible/latest/collections/community/docker/docker_image_module.html)


Since you already have these components automated with Docker compose, you can easily replicate this without writing a complex playbook with:

```yaml
---
- name: Deploy Docker containers
  hosts: localhost
  become: true

  tasks:
    - name: Build containers
      command: docker compose build
    - name: Spin up containers
      command: docker compose up -d
```

`-d` is included to send the containers to the background when spun up with Ansible.

A similar playbook could be written for spinning down the containers.


## Extras

If you want to tie everything together with a default `index.html` page, you can use the example below as a base.

> ```html
> <!DOCTYPE html>
> <html>
> <head>
>   <title>Web Application Test Environment</title>
> </head>
> <body>
>   <h1>Web Application Test Environment</h1>
>   <p>Welcome to the Web Application Test Environment.</p>
>   <p>This environment is designed for testing and demonstrating various web vulnerabilities and security measures.</p>
>   <ul>
>     <li><a href="login-safe.php">Safe Login Page</a></li>
>     <li><a href="login-vuln.php">Vulnerable Login Page</a></li>
>     <li><a href="test.php">Test Page</a></li>
>     <li><a href="read-safe.php">Safe Read Page</a></li>
>     <li><a href="read-vuln.php">Vulnerable Read Page</a></li>
>   </ul>
> </body>
> </html>
> ```


## Closing Thoughts

This process could be applied to almost any technology stack. Leveraging ChatGPT doesn't replace a human (yet) but reduces the time required to learn and implement ideas. This will help you understand how these applications are deployed and work from a perspective you won't often see if you're mainly doing CTF's or working with abstraction layers that remove all of this from you such as single click deployments.
