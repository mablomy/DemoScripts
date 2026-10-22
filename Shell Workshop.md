# MySQL Shell Workshop — Demo Runbook

## Contents

* [00 — Environment / Prerequisites](#00--environment--prerequisites)
* [01 — Start Shell and Connect](#01--start-shell-and-connect)
* [02 — Shell Modes and Commands](#02--shell-modes-and-commands)
* [03 — Productivity Features](#03--productivity-features)
* [04 — Utilities Overview](#04--utilities-overview)
* [05 — Dump & Load](#05--dump--load)
* [06 — Binary Log Dump / Load](#06--binary-log-dump--load)
* [07 — Upgrade Checker](#07--upgrade-checker)
* [08 — AdminAPI](#08--adminapi)
* [09 — Diagnostics](#09--diagnostics)
* [10 — Scripting](#10--scripting)
* [11 — Custom Plugin](#11--custom-plugin)
* [12 — Community Plugins: Audit](#12--community-plugins-audit)
* [13 — Community Plugins: LeFred](#13--community-plugins-lefred)
* [99 — Cleanup](#99--cleanup)

---

# 00 — Environment / Prerequisites

MySQL Shell 26.7 installed on a dedicated machine or on one of the MySQL servers.

Download these files to `$HOME` on the server where you plan to use MySQL Shell:

* `mysql-shell-commercial-26.7.0-1.1.el8.x86_64.rpm`
* `world.sql`

### For modules 01–07 and 10–12

* 1 installed MySQL Instance 8.4

  * admin user with root privileges
  * admin user can connect from `%`
  * enable the following options in `/etc/my.cnf`:

    * `mysql_native_password=ON`
    * `max_length_for_sort_data=4096`
    * `performance_schema_show_processlist=ON`
    * `replica_parallel_type=DATABASE`
  * restart MySQL after changing `/etc/my.cnf`
  * create `bad_user`:

    ```sql
    CREATE USER bad_user IDENTIFIED WITH mysql_native_password BY '...';
    ```
* MySQL client installed
* MySQL Shell 26.7 installed

### For module 08

* 3 MySQL 8.4 instances

  * instances ready for InnoDB Cluster
  * admin user with the same password on all instances
  * MySQL client installed on all instances
  * MySQL Shell 26.7 installed on mysql2 and mysql3
  * mysql2 and mysql3 use the same `/etc/my.cnf` configuration as mysql1
  * `mysql_native_password=ON`
  * `max_length_for_sort_data=4096`
  * `performance_schema_show_processlist=ON`
  * `replica_parallel_type=DATABASE`
* Create `bad_user` on mysql1

### MySQL Shell workshop plugin

Save the following file as `init.py`:

```python
def info():
    session = shell.get_session()

    if session is None:
        print("No active MySQL connection.")
        return

    result = session.run_sql("""
        SELECT
            @@hostname AS hostname,
            @@port AS port,
            @@version AS version
    """).fetch_one()

    print("MySQL Shell Workshop Plugin")
    print("----------------------------")
    print("Hostname : {}".format(result[0]))
    print("Port     : {}".format(result[1]))
    print("Version  : {}".format(result[2]))


workshop = shell.create_extension_object()

shell.add_extension_object_member(
    workshop,
    "info",
    info,
    {
        "brief": "Display basic information about the connected MySQL server."
    }
)

shell.register_global(
    "workshop",
    workshop,
    {
        "brief": "MySQL Shell Workshop example plugin."
    }
)
```



> **NOTES:**  
> * The workshop works better with two Shell sessions, with one session opened on the primary MySQL server.
> * Server names are assumed to be mysql1, mysql2 and mysql3.

[↑ Back to Contents](#contents)

---

# 01 — Start Shell and Connect

## Prerequisites

None.

## Commands

### Install MySQL Shell

```bash
# Change to the directory containing the MySQL Shell RPM
cd $HOME

ls -l mysql-shell-*.rpm

sudo dnf install mysql-shell-*.rpm
```

### Compare the classic mysql client with MySQL Shell

```bash
mysql -uadmin -p -hmysql1
```

```sql
SHOW DATABASES;
```

```sql
\q
```

```bash
mysqlsh -uadmin -p -hmysql1
```

```sql
SHOW DATABASES;
```

```sql
\q
```

### Connect using a URI-like connection string

```bash
mysqlsh admin@mysql1
```

### Import the world database using batch mode

```bash
mysqlsh admin@mysql1 < world.sql
```

```bash
mysqlsh admin@mysql1
```

```sql
SHOW DATABASES;
```

### Display the current connection status

```text
\status
```

### Display MySQL Shell help

```text
\help
```

```text
\h
```

### Get help for a SQL statement

```text
\help SELECT
```

### Use TAB completion for SQL keywords and objects

```text
SHOW DA[TAB][TAB];

USE wo[TAB][TAB];

SHOW TABLES;

SELECT * FROM country[TAB][TAB]
```

### Refresh the SQL object cache

```text
\rehash
```

```sql
SELECT * FROM performance_schema.processlist;
```

```sql
SELECT ID,USER,HOST,COMMAND,STATE
FROM performance_schema.processlist;
```

### Repeat a query at regular intervals

```text
\watch query SELECT now(),ID,USER,HOST,COMMAND,STATE
FROM performance_schema.processlist;
```

```text
[CTRL + C]
```

### Change the refresh interval to one second

```text
\watch query --interval=1 SELECT now(),ID,USER,HOST,COMMAND,STATE
FROM performance_schema.processlist;
```

### Optional: open a second Shell session

```bash
mysqlsh admin@mysql1
```

Generate a long-running query:

```sql
SELECT SLEEP(60);
```

Check the connection from the first session.

```text
[CTRL + C]
```

```text
\q
```

## Expected observations

[↑ Back to Contents](#contents)

---

# 02 — Shell Modes and Commands

## Prerequisites

* MySQL Shell connected to the MySQL instance
* `world` database loaded

## Commands

### Switch to JavaScript mode

```text
\js
```

> Note that the prompt changes.

### Switch to Python mode

```text
\py
```

> Note that the prompt changes.

### Switch back to JavaScript mode

```text
\js
```

### Display the Shell help

```text
\h
```

Scroll down to the **GLOBAL OBJECTS** section.

We will focus on the `util` object.

### Discover the methods exposed by the `util` object

```text
util.[TAB][TAB]
```

## Expected observations

[↑ Back to Contents](#contents)

---

# 03 — Productivity Features

## Prerequisites

* MySQL Shell connected to the MySQL instance
* `world` database loaded

## Commands

Use the productivity features introduced in module 01 as required during the presentation.

```text
\status
```

```text
\help SELECT
```

```text
SHOW DA[TAB][TAB];
```

```text
USE wo[TAB][TAB];
```

```text
\rehash
```

```text
\watch query SELECT now(),ID,USER,HOST,COMMAND,STATE
FROM performance_schema.processlist;
```

## Expected observations

[↑ Back to Contents](#contents)

---

# 04 — Utilities Overview

## Prerequisites

* MySQL Shell connected in JavaScript mode

## Commands

### Display the available `util` methods

```text
util.[TAB][TAB]
```

### Display help for the dump utilities

```text
\h util.dumpInstance
```

```text
\h util.dumpSchemas
```

```text
\h util.loadDump
```

### Display help for binary log dump/load

```text
\h util.dumpBinlogs
```

```text
\h util.loadBinlogs
```

### Display help for diagnostics

```text
\h util.debug.collectDiagnostics
```

## Expected observations

[↑ Back to Contents](#contents)

---

# 05 — Dump & Load

## Prerequisites

* `world` database loaded
* Binary logging enabled
* **GTID enabled**
  * 
  * \sql SET GLOBAL gtid_mode=OFF_PERMISSIVE;
  * \sql SET GLOBAL gtid_mode=ON_PERMISSIVE;
  * \sql SET GLOBAL gtid_mode=ON;
* **`local_infile` enabled** \sql SET GLOBAL local_infile=true;

## Commands

### Create a full instance dump and preview the operation

The demo only restores the `world` schema later.

```text
\js
```

```javascript
util.dumpInstance('./dumpWorld',{'dryRun':true})
```

```javascript
util.dumpInstance('./dumpWorld')
```

### Inspect the generated dump

```bash
tree dumpWorld
```

### Destroy the `world` database

```text
\sql
```

```sql
DROP DATABASE world;
```

```sql
SHOW DATABASES;
```

### Preview the load operation

```text
\js
```

```javascript
util.loadDump('./dumpWorld',
{'includeSchemas':['world'],'dryRun':true})
```

### Restore only the `world` schema from the full instance dump

```javascript
util.loadDump('./dumpWorld',
{'includeSchemas':['world']})
```

### Verify that the `world` database has been restored

```text
\sql
```

```sql
SHOW TABLES FROM world;
```

## Expected observations

[↑ Back to Contents](#contents)

---

# 06 — Binary Log Dump / Load

## Prerequisites

* `world` database restored
* Binary logging enabled
* GTID enabled

## Commands

### Create changes after the instance dump

```text
\sql
```

```sql
CREATE TABLE world.workshop_log (
    id INT PRIMARY KEY,
    message VARCHAR(100)
);
```

```sql
INSERT INTO world.workshop_log VALUES
(1,'Created after the dump'),
(2,'This change exists only in the binary log'),
(3,'MySQL Shell can recover it');
```

```sql
SELECT * FROM world.workshop_log;
```

### Rotate the binary log

```sql
FLUSH BINARY LOGS;
```

### Preview the binary log dump

```text
\js
```

```javascript
util.dumpBinlogs('./binlogWorld',
{'since':'./dumpWorld','dryRun':true})
```

### Dump binary logs generated since the instance dump

```javascript
util.dumpBinlogs('./binlogWorld',
{'since':'./dumpWorld'})
```

### Inspect the generated binary log dump

```bash
tree binlogWorld
```

> The binary logs are intentionally not loaded in this workshop.
>
> Loading binary logs requires recovery-specific GTID handling, which is outside the scope of this introductory workshop.

## Expected observations

[↑ Back to Contents](#contents)

---

# 07 — Upgrade Checker

## Prerequisites

* MySQL 8.4 instance with the workshop configuration
* MySQL Shell 26.7

## Commands

### Run the Upgrade Checker with its default target

```text
\js
```

```javascript
util.checkForServerUpgrade()
```

### Specify the target MySQL Server version explicitly

```javascript
util.checkForServerUpgrade(
'root@127.0.0.1:3306',
{'targetVersion':'9.7.2','configPath':'/etc/my.cnf'}
)
```

> Discuss the findings and the error related to `bad_user`.

> Note that the report identifies the affected MySQL accounts.

## Expected observations

[↑ Back to Contents](#contents)

---

# 08 — AdminAPI

## Prerequisites

* 3 MySQL 8.4 instances ready for InnoDB Cluster
* admin user with sufficient privileges on all instances
* same `/etc/my.cnf` configuration on mysql1, mysql2 and mysql3
* MySQL Shell 26.7 connected to mysql1

## Commands

### Make sure we are using JavaScript mode

```text
\js
```

### Explore the AdminAPI object

```text
dba.[TAB][TAB]
```

### Check the configuration of the first instance

```javascript
dba.checkInstanceConfiguration('admin@mysql1')
```

### Let MySQL Shell fix the configuration

```javascript
dba.configureInstance('admin@mysql1')
```

### Apply the same configuration to the other instances

```javascript
dba.configureInstance('admin@mysql2')
```

```javascript
dba.configureInstance('admin@mysql3')
```

### Create the InnoDB Cluster

```javascript
cluster=dba.createCluster(
'mycluster',{'communicationStack':'MySQL'})
```

### Add the other instances

```javascript
cluster.addInstance('admin@mysql2')
```

```javascript
cluster.addInstance('admin@mysql3')
```

### Discover the methods available on the Cluster object

```text
cluster.[TAB][TAB]
```

```javascript
cluster.status()
```

### Optional: inspect only the topology information

```javascript
cluster.status().defaultReplicaSet.topology
```

### Simplify Cluster operations
```Shell
mysqlsh root@demo1 --js --cluster
```

```Javascript
cluster.execute("SELECT @@hostname", "all")
```


## Expected observations

[↑ Back to Contents](#contents)

---

# 09 — Diagnostics

## Prerequisites

* InnoDB Cluster created
* MySQL Shell connected to a cluster member

## Commands

### Display the diagnostic utility help

```text
\h util.debug.collectDiagnostics
```

### Collect diagnostic information from all cluster members

```javascript
util.debug.collectDiagnostics(
'./diagnostic2',
{allMembers:true,schemaStats:true,ignoreErrors:true}
)
```

### Inspect the generated archive

From the operating system:

```bash
unzip diagnostic2.zip
```

```bash
tree diagnostic2
```

## Expected observations

A single command collects diagnostic information from all cluster members.

The output is a ZIP archive containing diagnostic reports.

`schemaStats` adds schema size and object statistics.

`ignoreErrors` allows the collection to continue if individual diagnostic queries fail.

[↑ Back to Contents](#contents)

---

# 09b — Render Graphical EXPLAIN Plans

## Prerequisites
* Have the World database installed
* Have an option to display SVG images (download locally, or use imgcat to display in iTerm)

## Commands
```text
\h \h visual.explain
```

```text
\js
util.visual.explain(
"WITH european_g_countries AS
(SELECT co.Code, co.Name AS country, co.Population AS country_population, COUNT(cl.Language) AS language_count FROM world.country AS co
JOIN world.countrylanguage AS cl ON cl.CountryCode = co.Code WHERE co.Continent = 'Europe' AND co.Name LIKE 'G%'
GROUP BY co.Code, co.Name, co.Population),
ranked_cities AS
(SELECT eg.country, eg.country_population, eg.language_count, ci.Name AS city, ci.Population, ROW_NUMBER()
OVER ( PARTITION BY eg.Code ORDER BY ci.Population DESC) AS city_rank
FROM european_g_countries AS eg JOIN world.city AS ci ON ci.CountryCode = eg.Code)
SELECT country, language_count, ROUND(100.0 * SUM(Population) / NULLIF(MAX(country_population), 0), 2) AS pct_of_country,
GROUP_CONCAT(CONCAT(city, ' (', FORMAT(Population, 0), ')') ORDER BY Population DESC SEPARATOR ', ' ) AS three_largest_cities
FROM ranked_cities   WHERE city_rank <= 3 GROUP BY country, language_count ORDER BY pct_of_country DESC", "./explain.svg") 
```
Download the SVG and display

## Expected observations
Graphical Explain Plan produced
See also
```
util.visual.renderExplainPlan(plan, path[, kwargs])
```

[↑ Back to Contents](#contents)

---

# 10 — Scripting

## Prerequisites

* MySQL Shell connected to the MySQL instance

## Commands

### Switch to JavaScript mode

```text
\js
```

Run the prepared scripting example.

### Switch to Python mode

```text
\py
```

Run the prepared scripting example.

## Expected observations

[↑ Back to Contents](#contents)

---

# 11 — Custom Plugin

## Prerequisites

* MySQL Shell 26.7
* MySQL Shell connected to the MySQL instance

## Commands

### Exit MySQL Shell

```text
\q
```

### Create the plugin directory

```bash
mkdir -p ~/.mysqlsh/plugins/workshop
```

### Copy the prepared plugin

```bash
cp init.py ~/.mysqlsh/plugins/workshop/
```

### Start MySQL Shell again

```bash
mysqlsh admin@mysql1
```

### Switch to Python mode

```text
\py
```

### Check the new extension object

```text
workshop.[TAB][TAB]
```

### Run the plugin

```text
workshop.info()
```

### Test the same extension object from JavaScript mode

The plugin is implemented in Python, but the extension object is also available in JavaScript mode.

```text
\js
```

```text
workshop.info()
```

### Display the plugin help

```text
\help workshop.info
```

## Expected observations

[↑ Back to Contents](#contents)

---

# 12 — Community Plugins Audit

## Prerequisites

* MySQL Enterprise Audit installed and configured
* Audit log configured to use JSON format
* `bad_user` account already created

## Commands

### Exit MySQL Shell

```text
\q
```

### Download the community Audit Filter Wizard plugin

```bash
mkdir -p ~/.mysqlsh/plugins/audittool_plugin
```

```bash
cd ~/.mysqlsh/plugins/audittool_plugin
```

```bash
wget https://raw.githubusercontent.com/mfrankmysql/mysqlshellauditplugin/refs/heads/main/init.py
```

```bash
wget https://raw.githubusercontent.com/mfrankmysql/mysqlshellauditplugin/refs/heads/main/wizard.py
```

```bash
ls -l
```

```bash
cd
```

### Start MySQL Shell in Python mode

```bash
mysqlsh admin@mysql1 --py
```

### Explore the installed plugin

```text
audittool.[TAB][TAB]
```

```text
\h audittool
```

### Start the interactive wizard

```text
audittool.start.wizard()
```

### Create a connection audit filter for connect/disconnect events

Follow the wizard prompts and use:

```text
filter-login-logout
```

Assign `bad_user` to the filter.

### Show the resulting audit filter and account assignment

Use the wizard options:

```text
1 - Show audit filters
4 - Show users and filter
```

### Exit the wizard

```text
\q
```

### Generate a connection event with `bad_user`

```bash
mysql -ubad_user -p -hmysql1 -e "SELECT 1"
```

### Display the generated JSON audit events

The `audit.log` file is open, so its JSON array is not closed yet.

```bash
sudo sh -c 'cat /var/lib/mysql/audit.log; echo "]"' | jq .
```

### Optional: show only connection events

```bash
sudo sh -c 'cat /var/lib/mysql/audit.log; echo "]"' |
jq '.[] | select(.class == "connection") |
{timestamp, event, connection_id, user: .account.user, ip: .login.ip}'
```

## Expected observations

[↑ Back to Contents](#contents)

---

# 13 — Community Plugins LeFred

## Prerequisites
Clone LeFred's github
```bash
mkdir ~/.mysqlsh/LeFred
git clone https://github.com/lefred/mysqlshell-plugins.git ~/.mysqlsh/LeFred/
```
Copy the wanted plugins to the plugin directory and Restart Shell

```bash
cp -r .mysqlsh/LeFred/security .mysqlsh/plugins/
cp -r .mysqlsh/LeFred/user     .mysqlsh/plugins/
mysqlsh --js --cluster root@demo1
```

## Commands
```text
\h
user.create_user()
security.showPasswordExpireSoon()
security.showPasswordExpire()
```

[↑ Back to Contents](#contents)

---


# 99 — Cleanup

## Commands

### Remove workshop dump and plugin files as needed

```bash
rm -rf dumpWorld binlogWorld diagnostic2 \
~/.mysqlsh/plugins/workshop \
~/.mysqlsh/plugins/audittool_plugin
```

### Drop workshop objects if needed

```text
\sql
```

```sql
DROP TABLE IF EXISTS world.workshop_log;
```

## Expected observations
