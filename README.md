# SQL

Segment source for Postgresql Databases. Syncs your production postgressql databases with [Segment Objects API](https://github.com/segmentio/objects-go).


## Quick Start

### Build and Run
Prerequisites: [Go](https://golang.org/doc/install)

```bash
go get -u github.com/segment-sources/postgres
```

The first step is to initialize your schema. You can do so by running `postgres` with `--init` flag.
```bash
postgres --init --write-key=ab-200-1alx91kx --hostname=postgres-test.ksdg31bcms.us-west-2.rds.amazonaws.com --port=5432 --username=segment --password=cndgks8102baajls --database=segment -- sslmode=prefer
```
The init step will store the schema of possible tables that the source can sync in `schema.json`. The query will look for tables across all schemas excluding the ones without a `PRIMARY KEY`.

In the `schema.json` example below, our parser found the table `public.films` where `public` is the schema name and `films` the table name with a compound primary key and 6 columns. The values in the `primary_keys` list have to be present in the `columns` list. The `column` list is used to generate `SELECT` statements, you can filter out some fields that you don't want to sync with Segment by removing them from the list.
```json
{
	"public": { 
		"films": {
			"primary_keys": [
				"code",
				"title"
			],
			"columns": [
				"code",
				"title",
				"did",
				"date_prod",
				"kind",
				"len"
			]
		}
	}
}
```


Segment's Objects API requires a unique identifier in order to properly sync your tables, the `PRIMARY KEY` is used as the identifier. Your tables may also have multiple primary keys, in that case we'll concatenate the values in one string joined with underscores.


### Scan
```bash
sqlsource --write-key=ab-200-1alx91kx --driver=postgres --hostname=postgres-test.ksdg31bcms.us-west-2.rds.amazonaws.com --port=5432 --username=segment --password=cndgks8102baajls --database=segment --sslmode=prefer
```

Example Run:
```bash
INFO[0000] Scan started                                  schema=public table=films
DEBU[0000] Executing query: SELECT "code", "title", "did", "date_prod", "kind", "len" FROM "public"."films"
DEBU[0000] Received Row                                  row=map[did:1 date_prod:<nil> kind:<nil> len:<nil> code:1     title:title] schema=public table=films
INFO[0000] Scan finished                                 schema=public table=films
```

### Usage
```
Usage:
  sqlsource
    [--debug]
    [--init]
    [--concurrency=<c>]
    --write-key=<segment-write-key>
    --driver=<driver>
    --hostname=<hostname>
    --port=<port>
    --username=<username>
    --password=<password>
    --database=<database>
    [-- <extra-driver-options>...]
  sqlsource -h | --help
  sqlsource --version
  
Options:
  -h --help                   Show this screen
  --version                   Show version
  --write-key=<key>           Segment source write key
  --driver=<driver>           Choose the driver to use
  --concurrency=<c>           Number of concurrent table scans [default: 1]
  --hostname=<hostname>       Database instance hostname
  --port=<port>               Database instance port number
  --password=<password>       Database instance password
  --database=<database>       Database instance name
```

### Schema

