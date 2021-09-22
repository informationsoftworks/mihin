
# Using the MiHIN Mapping table results

The results of the distillation process can be used to select the associated
mappings by following the three steps listed here.

## Step 1: Create database tables in your database

Create tables in the database containing your ADT messages, similar to the code below:
```sql
create table ref.facility (
	oid	text,
	facility	text,
	primary key (oid)
);

create table ref.facility_mapping (
	oid	text,
	name	text,
	id	text,
	primary key (oid, name)
);

create table ref.mappings (
	id		text,
	value		text,
	description	text,
	mihin_value	text,
	mihin_description text,
	primary key (id, value)
);

-- Optional, but convenient:
create or replace view ref.mihin_mapping as
select *
from ref.facility_mapping
join ref.mappings using (id)
;
```

## Step 2: Create batch file to update data from GitHub

Create a batch file that will truncate and reload the Mapping Table data
from the GitHub "raw" urls.  Here's an example using "wget" to retrieve the
data and load directly into a PostgreSQL database.  This can then be scheduled
to execute on  a regular schedule, or at the beginning of a database update process.

```bash
#!/bin/bash
psql -c 'truncate table ref.facility' cdw_prod
wget -q -O - 'https://raw.githubusercontent.com/informationsoftworks/mihin/master/mapping_tables/target/facility.csv' \
	| psql -c '\copy ref.facility from stdin csv header' cdw_prod

psql -c 'truncate table ref.facility_mapping' cdw_prod
wget -q -O - 'https://raw.githubusercontent.com/informationsoftworks/mihin/master/mapping_tables/target/facility_mapping.csv' \
	| psql -c '\copy ref.facility_mapping from stdin csv header' cdw_prod

psql -c 'truncate table ref.mappings' cdw_prod;
wget -q -O - 'https://raw.githubusercontent.com/informationsoftworks/mihin/master/mapping_tables/target/mappings.csv' \
	| psql -c '\copy ref.mappings from stdin csv header' cdw_prod
```

## Step 3: Use loaded data in reporting queries

Using the data to look up coded values is dependent on the specific data
model and database, but here is an example with an "ADT" table:

### Table "ADT"

patient_id | facility_oid | patient_class
----------- | ------- | ------------------
12345 | 2.16.840.1.113883.3.801.4.5 | E
54321 | 2.16.840.1.113883.3.4376.1100 | O
44444 | 2.16.840.1.113883.3.801.4.5 | O
88888 | 2.16.840.1.113883.3.4376.1100 | P
99999 | 2.16.840.1.113883.3.4376.1100 | O
11111 | 2.16.840.1.113883.3.4220.4.10005 | OV
12222 | 2.16.840.1.113883.3.3238.50.958 | N
13333 | 2.16.840.1.113883.3.4220.4.10005 | O
14444 | 2.16.840.1.113883.3.4220.4.10005 | I
87654 | 2.16.840.1.113883.3.4220.4.10001 | SA

The decoded value for "patient_class" (PV1-2) can be retrieved from the mapping
tables by joining with the "value" column, and selecting the "PV1-2" HL7 field name:

```sql
select adt.patient_id,
	adt.patient_class,
	mihin_mapping.description,
	mihin_mapping.mihin_value,
	mihin_mapping.mihin_description
from adt
left join ref.mihin_mapping
	on adt.facility_oid = mihin_mapping.oid
	and adt.patient_class = mihin_mapping.value
	and mihin_mapping.name = 'PV1-2'
;
```

### Query Results

patient_id | patient_class | description | mihin_value | mihin_description
---------- | ------------- | ----------- | ----------- | -----------------
12345 | E | | E | Emergency
54321 | O | | O | Outpatient
44444 | O | | O | Outpatient
88888 | P | | P | Preadmit
99999 | O | | O | Outpatient
11111 | OV | | O | Outpatient
12222 | N | | NB | Newborn
13333 | O | | O | Outpatient
14444 | I | | I | Inpatient
87654 | SA | | SA | Surgery Admit

