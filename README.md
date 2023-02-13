# ELT-data-pipeline
A data pipeline using Singer (an open-sourced ETL) for MySQL DB to CSV and PostgreSQL


### Install

#### MySQL Tap

```bash
$ virtualenv venv
$ pip install tap-mysql
```
```bash
{
  "host": "localhost",
  "port": "3306",
  "user": "<username>",
  "password": "<password>"
}
```

```bash
tap-mysql --config tap_mysql_config.json --discover > catalog.json
```

A JSON schema description of each table is produced and stored in the file “catalog.json”. In the file, a source table directly corresponds to a Singer stream. Creation of this file was necessary so that tap’s discovery mode file could be modified. 

In the catalog.json file, add metadata in the catalog for field selection and replication method. As the dataset tables which we want to move are “census” and “state_fact”, only respected tap_stream’s are marked to replicate using FULL_TABLE along with the required replication key. Similarly, the fields of these two tables are also selected as true for the pipeline. Following picture depict these two selections

After updating the catalog.json file, run MySQL Tap in sync mode. This is done so that the tap can read the catalog and find for tables and fields marked as selected (in the previous step)
```bash
tap-mysql --config tap_mysql_config.json --discover > catalog.json
```


#### CSV Target

```bash
$ virtualenv venv
$ pip install target-csv
```
##### Edit target's config file (target_csv_config.json) to include necessary credentials or parameters.
```bash
{
  "delimiter": "\t",
  "quotechar": "'",
  "destination_path": ""
}
```

Run the Singer Tap and pipe the output to the CSV Target. The final state by CSV target was saved for future runs in state.json
```bash
tap-mysql --config tap_mysql_config.json --catalog catalog.json & target-csv --config target_csv_config.json >> state.json
```


Extract the last line of state.json and save it into state.json.tmp. And then rename the newly created state.json.tmp as state.json
```bash
Get-Content state.json | Select-Object Last 1 | Set-Content state.json.tmp | move state.json.tmp state.json
```

The state.json file after the above command run


#### PostgreSQL Target

```bash
$ virtualenv venv
$ pip install target-csv
```
Need to install PostgreSQL database. (Reference: https://www.postgresql.org/download/ Links to an external site.).

##### Edit target's config file (target_postgres_config.json) to include necessary credentials or parameters.
```bash
{
  "postgres_host": "localhost",
  "postgres_port": 5432,
  "postgres_database": "my_analytics",
  "postgres_username": "<username>",
  "postgres_password": "<password>",
  "postgres_schema": "mytapname"
}
```
