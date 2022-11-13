# dbreporting
A python script that allow to create different kind of db extract to generate different output.
One of the objective was to create a script that merge the content from different db but with same content.
Like 3 db from a monitoring tool to merge all infos in a simple output.

Actually only Mysql or derivative are supported.

## Configuration
Copy conf/config.sample.json to conf/config.json

Edit it and change the following variables to fit your setup.

Don't hesitate to validate the json via a jq query to ensure the json structure.

###  "sql_directory"
Sample value : ../sql
This variable define where the sql files are read in order to perform queries

### "template_directory"
Sample value : "../template"
This variable define where the template files are read in order to create output files.
Actually only excel output can use templates

### "output_directory"
Sample value : "../data"
This variable define where the results file are stored.

### smtp_relay
Define your smtp gateway if you need to send the report.

### datasources
Define your datasources in this object, 

Each datasource (DS) must have a unique name and must be composed of the following KV:
 - hostname
 - username
 - password
 - port
 - database

### queries
Define your queries in this object.

Each query must have a unique name and must be composed of the following KV / objects :

#### query_file
Name of the file to use in order to perform the DB query.

The SQL Query must be formatted as following one :
``` 
SELECT *
FROM family
WHERE 1 = 1
```
Where condition is optional but if you use it, please keep it in UPPERCASE, it's important for this script.

**And query mustn't contain LIMIT neither ORDER BY command it will be cover in example commands.**

#### sources
The datasource that will be used to perform the query, in case of multiple source, they will be merged in a single one.

#### fixfields
List of fields to cleanup if there is some utf8 / html code inside to be converted to standard text.

#### sort_fields
The field that will be used to perform a data ordering, actually only one field is supported

#### output_type

Choose the output file format of the report, actually the supported file are :
   - html
   - json
   - csv
   - xlsx (MS Excel)

#### excel_options
If you have selected a xlsx output format you have to set the following option 

   - template_file, if you have an excel template to use with logo or whatever
   - sheet_name, the sheet_name to create or update
   - startrow, the row where to start to insert data

#### output_filename

#### email
If you want to send an email with the report generated, set the following options:
   - to, recipient of the email
   - from, sender of the email
   - subject, mail subject
   - body, content of the mail message


## Usage
```
usage: main.py [-h] [--conf CONF] [--dryrun] [--debug] [--version] [--query QUERY] [--filter FILTER] [--limit_rows LIMIT_ROWS] [--limit_start LIMIT_START]

Sample Reporting tool

optional arguments:
  -h, --help            show this help message and exit
  --conf CONF           configuration file to use (Mandatory)
  --dryrun              do not create snow ticket just add a comment
  --debug               display more info
  --version             display software info
  --query QUERY         run a predefined query (Mandatory)
  --filter FILTER       add an extra sql filter(Optional)
  --limit_rows LIMIT_ROWS
                        limit the results returned (Optional)
  --limit_start LIMIT_START
                        return the results starting from(Optional but require --limit_rows)
```

Examples :
```
./main.py --debug --conf ../conf/config.json --query all_events_last_24h --limit_rows 2 --limit_start 2
```
Will: 
- run in debug
- use configuration ../conf/config.json
- use the query all_events_last_24h
- limit to 2 rows starting from row 2

```
./main.py --conf ../conf/config.json --query info --limit_rows 4
```

Will:
- use configuration ../conf/config.json
- use the query info
- limit to 4 rows


```
./main.py --conf ../conf/config.json --query info --limit_rows 4 --filter "field > 20 OR field2 < 30
```

Will:
- use configuration ../conf/config.json
- use the query info
- limit to 4 rows
- add a WHERE condition : WHERE field > 20 OR field2 < 30 

In case you query file already contain a WHERE (in uppercase) condition it will automatically use AND instead of WHERE.
