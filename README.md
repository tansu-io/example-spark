
This repository showcases examples of structured data published to schema-backed topics, instantly accessible as [Apache Iceberg tables](https://iceberg.apache.org) via [Apache Spark](https://spark.apache.org)

Prerequisites:
- **[docker](https://www.docker.com)**, using [compose.yaml](compose.yaml) running [Tansu](https://tansu.io), [Apache Spark](https://spark.apache.org), [MinIO](https://min.io) and an [Apache Iceberg REST Catalog](https://iceberg.apache.org/terms/#decoupling-using-the-rest-catalog)
- **[just](https://github.com/casey/just)**, a handy way to save and run project-specific commands

The [justfile](./justfile) contains recipes to run [MinIO](https://min.io), create the buckets, an Apache Iceberg REST catalog and [Apache Spark](https://spark.apache.org) with [Tansu](https://tansu.io).

Once you have the prerequisites installed, clone this repository and start everything up with:

```shell
git clone git@github.com:tansu-io/example-spark.git
cd example-spark
just up
```

Should result in:

```
 ✔ Network example-spark_default
 ✔ Volume "example-spark_minio"
 ✔ Container example-spark-minio-1
The cluster 'local' is ready
Added `local` successfully.
Bucket created successfully `local/tansu`.
Bucket created successfully `local/lake`.
 ✔ Container example-spark-minio-1
 ✔ Container example-spark-iceberg-catalog-1
 ✔ Container example-spark-spark-1
 ✔ Container example-spark-tansu-1
 ```

Done! You can now run the examples.

## Employee

Employee is a protocol buffer backed topic, with the following schema [employee.proto](schema/employee.proto):

```proto
syntax = 'proto3';

message Key {
  int32 id = 1;
}

message Value {
  string name = 1;
  string email = 2;
}
```

Sample employee data is in [employees.json](data/employees.json):

```json
[
  {
    "key": { "id": 12321 },
    "value": { "name": "Bob", "email": "bob@example.com" }
  },
  {
    "key": { "id": 32123 },
    "value": { "name": "Alice", "email": "alice@example.com" }
  }
]
```

Create the employee topic:

```bash
just employee-topic-create
```

Publish the sample data onto the employee topic:

```bash
just employee-produce
```

View the data in Spark SQL:

```bash
just spark-sql
```

Query:

```sql
select * from tansu.employee;
```

With output:

```text
spark-sql ()> select * from tansu.employee;
{"partition":0,"timestamp":2025-05-31 09:44:11.566,"year":2025,"month":5,"day":31}	{"id":12321}	{"name":"Bob","email":"bob@example.com"}
{"partition":0,"timestamp":2025-05-31 09:44:11.566,"year":2025,"month":5,"day":31}	{"id":32123}	{"name":"Alice","email":"alice@example.com"}
```

## Grade

Grade is a JSON schema backed topic, with the following schema [grade.json](schema/grade.json):

```json
{
  "type": "record",
  "name": "Grade",

  "fields": [
    { "name": "key", "type": "string", "pattern": "^\\d{3}-\\d{2}-\\d{4}$" },
    {
      "name": "value",
      "type": {
        "type": "record",
        "fields": [
          { "name": "first", "type": "string" },
          { "name": "last", "type": "string" },
          { "name": "test1", "type": "double" },
          { "name": "test2", "type": "double" },
          { "name": "test3", "type": "double" },
          { "name": "test4", "type": "double" },
          { "name": "final", "type": "double" },
          { "name": "grade", "type": "string" }
        ]
      }
    }
  ]
}
```

Sample grade data is in: [grades.json](data/grades.json):

```json
[
  {
    "key": "123-45-6789",
    "value": {
      "lastName": "Alfalfa",
      "firstName": "Aloysius",
      "test1": 40.0,
      "test2": 90.0,
      "test3": 100.0,
      "test4": 83.0,
      "final": 49.0,
      "grade": "D-"
    }
  },
  ...
]
```

Create the grade topic:

```bash
just grade-topic-create
```

Publish the sample data onto the grade topic:

```bash
just grade-produce
```

View the data in Spark SQL:

```bash
just spark-sql
```

Query:

```sql
select * from tansu.grade;
```

With output:

```text
spark-sql ()> select * from tansu.grade;
{"day":31,"month":5,"partition":0,"timestamp":"2025-05-31T09:48:27.733+00:00","year":2025}	123-45-6789	{"final":49.0,"first":"Aloysius","grade":"D-","last":"Alfalfa","test1":40.0,"test2":90.0,"test3":100.0,"test4":83.0}
{"day":31,"month":5,"partition":0,"timestamp":"2025-05-31T09:48:27.733+00:00","year":2025}	123-12-1234	{"final":48.0,"first":"University","grade":"D+","last":"Alfred","test1":41.0,"test2":97.0,"test3":96.0,"test4":97.0}
{"day":31,"month":5,"partition":0,"timestamp":"2025-05-31T09:48:27.733+00:00","year":2025}	567-89-0123	{"final":44.0,"first":"Gramma","grade":"C","last":"Gerty","test1":41.0,"test2":80.0,"test3":60.0,"test4":40.0}
{"day":31,"month":5,"partition":0,"timestamp":"2025-05-31T09:48:27.733+00:00","year":2025}	087-65-4321	{"final":47.0,"first":"Electric","grade":"B-","last":"Android","test1":42.0,"test2":23.0,"test3":36.0,"test4":45.0}
{"day":31,"month":5,"partition":0,"timestamp":"2025-05-31T09:48:27.733+00:00","year":2025}	456-78-9012	{"final":45.0,"first":"Fred","grade":"A-","last":"Bumpkin","test1":43.0,"test2":78.0,"test3":88.0,"test4":77.0}
{"day":31,"month":5,"partition":0,"timestamp":"2025-05-31T09:48:27.733+00:00","year":2025}	234-56-7890	{"final":46.0,"first":"Betty","grade":"C-","last":"Rubble","test1":44.0,"test2":90.0,"test3":80.0,"test4":90.0}
{"day":31,"month":5,"partition":0,"timestamp":"2025-05-31T09:48:27.733+00:00","year":2025}	345-67-8901	{"final":43.0,"first":"Cecil","grade":"F","last":"Noshow","test1":45.0,"test2":11.0,"test3":-1.0,"test4":4.0}
{"day":31,"month":5,"partition":0,"timestamp":"2025-05-31T09:48:27.733+00:00","year":2025}	632-79-9939	{"final":50.0,"first":"Bif","grade":"B+","last":"Buff","test1":46.0,"test2":20.0,"test3":30.0,"test4":40.0}
{"day":31,"month":5,"partition":0,"timestamp":"2025-05-31T09:48:27.733+00:00","year":2025}	223-45-6789	{"final":83.0,"first":"Andrew","grade":"A","last":"Airpump","test1":49.0,"test2":1.0,"test3":90.0,"test4":100.0}
{"day":31,"month":5,"partition":0,"timestamp":"2025-05-31T09:48:27.733+00:00","year":2025}	143-12-1234	{"final":97.0,"first":"Jim","grade":"A+","last":"Backus","test1":48.0,"test2":1.0,"test3":97.0,"test4":96.0}
{"day":31,"month":5,"partition":0,"timestamp":"2025-05-31T09:48:27.733+00:00","year":2025}	565-89-0123	{"final":40.0,"first":"Art","grade":"D+","last":"Carnivore","test1":44.0,"test2":1.0,"test3":80.0,"test4":60.0}
{"day":31,"month":5,"partition":0,"timestamp":"2025-05-31T09:48:27.733+00:00","year":2025}	087-75-4321	{"final":45.0,"first":"Jim","grade":"C+","last":"Dandy","test1":47.0,"test2":1.0,"test3":23.0,"test4":36.0}
{"day":31,"month":5,"partition":0,"timestamp":"2025-05-31T09:48:27.733+00:00","year":2025}	456-71-9012	{"final":77.0,"first":"Ima","grade":"B-","last":"Elephant","test1":45.0,"test2":1.0,"test3":78.0,"test4":88.0}
{"day":31,"month":5,"partition":0,"timestamp":"2025-05-31T09:48:27.733+00:00","year":2025}	234-56-2890	{"final":90.0,"first":"Benny","grade":"B-","last":"Franklin","test1":50.0,"test2":1.0,"test3":90.0,"test4":80.0}
{"day":31,"month":5,"partition":0,"timestamp":"2025-05-31T09:48:27.733+00:00","year":2025}	345-67-3901	{"final":4.0,"first":"Boy","grade":"B","last":"George","test1":40.0,"test2":1.0,"test3":11.0,"test4":-1.0}
{"day":31,"month":5,"partition":0,"timestamp":"2025-05-31T09:48:27.733+00:00","year":2025}	632-79-9439	{"final":40.0,"first":"Harvey","grade":"C","last":"Heffalump","test1":30.0,"test2":1.0,"test3":20.0,"test4":30.0}
```

## Observation

Observation is an Avro backed topic, with the following schema [observation.avsc](schema/observation.avsc):

```json
{
  "type": "record",
  "name": "observation",
  "fields": [
    { "name": "key", "type": "string", "logicalType": "uuid" },
    {
      "name": "value",
      "type": "record",
      "fields": [
        { "name": "amount", "type": "double" },
        { "name": "unit", "type": "enum", "symbols": ["CELSIUS", "MILLIBAR"] }
      ]
    }
  ]
}
```

Sample observation data, is in: [observations.json](data/observations.json):

```json
[
  {
    "key": "1E44D9C2-5E7A-443B-BF10-2B1E5FD72F15",
    "value": { "amount": 23.2, "unit": "CELSIUS" }
  },
  ...
]
```

Create the observation topic:

```bash
just observation-topic-create
```

Publish the sample data onto the observation topic:

```bash
just observation-produce
```

View the data in Spark SQL:

```bash
just spark-sql
```

Query:

```sql
select * from tansu.observation;
```

With output:

```text
spark-sql ()> select * from tansu.observation;
1e44d9c2-5e7a-443b-bf10-2b1e5fd72f15	{"amount":23.2,"unit":"CELSIUS"}	{"partition":0,"timestamp":2025-05-31 09:49:29.28,"year":2025,"month":5,"day":31}
1e44d9c2-5e7a-443b-bf10-2b1e5fd72f15	{"amount":1027.0,"unit":"MILLIBAR"}	{"partition":0,"timestamp":2025-05-31 09:49:29.28,"year":2025,"month":5,"day":31}
1e44d9c2-5e7a-443b-bf10-2b1e5fd72f15	{"amount":22.8,"unit":"CELSIUS"}	{"partition":0,"timestamp":2025-05-31 09:49:29.28,"year":2025,"month":5,"day":31}
1e44d9c2-5e7a-443b-bf10-2b1e5fd72f15	{"amount":1023.0,"unit":"MILLIBAR"}	{"partition":0,"timestamp":2025-05-31 09:49:29.28,"year":2025,"month":5,"day":31}
1e44d9c2-5e7a-443b-bf10-2b1e5fd72f15	{"amount":22.5,"unit":"CELSIUS"}	{"partition":0,"timestamp":2025-05-31 09:49:29.28,"year":2025,"month":5,"day":31}
1e44d9c2-5e7a-443b-bf10-2b1e5fd72f15	{"amount":1018.0,"unit":"MILLIBAR"}	{"partition":0,"timestamp":2025-05-31 09:49:29.28,"year":2025,"month":5,"day":31}
1e44d9c2-5e7a-443b-bf10-2b1e5fd72f15	{"amount":23.1,"unit":"CELSIUS"}	{"partition":0,"timestamp":2025-05-31 09:49:29.28,"year":2025,"month":5,"day":31}
1e44d9c2-5e7a-443b-bf10-2b1e5fd72f15	{"amount":1020.0,"unit":"MILLIBAR"}	{"partition":0,"timestamp":2025-05-31 09:49:29.28,"year":2025,"month":5,"day":31}
1e44d9c2-5e7a-443b-bf10-2b1e5fd72f15	{"amount":23.4,"unit":"CELSIUS"}	{"partition":0,"timestamp":2025-05-31 09:49:29.28,"year":2025,"month":5,"day":31}
1e44d9c2-5e7a-443b-bf10-2b1e5fd72f15	{"amount":1025.0,"unit":"MILLIBAR"}	{"partition":0,"timestamp":2025-05-31 09:49:29.28,"year":2025,"month":5,"day":31}
```

## Person

Person is a JSON schema backed topic, with the following schema [person.json](schema/person.json):

```json
{
  "title": "Person",
  "type": "object",
  "properties": {
    "key": {
      "type": "string",
      "pattern": "^\\d{3}-\\d{2}-\\d{4}$"
    },
    "value": {
      "type": "object",
      "properties": {
        "firstName": {
          "type": "string",
          "description": "The person's first name."
        },
        "lastName": {
          "type": "string",
          "description": "The person's last name."
        },
        "age": {
          "description": "Age in years which must be equal to or greater than zero.",
          "type": "integer",
          "minimum": 0
        }
      }
    }
  }
}
```

Sample person data, is in [persons.json](data/persons.json):

```json
[
  {
    "key": "123-45-6789",
    "value": { "lastName": "Alfalfa", "firstName": "Aloysius", "age": 21 }
  },
  ...
]
```

Create the person topic:

```bash
just person-topic-create
```

Publish the sample data onto the person topic:

```bash
just person-produce
```

View the data in Spark SQL:

```bash
just spark-sql
```

Query:

```sql
select * from tansu.person;
```

With output:

```text
spark-sql ()> select * from tansu.person;
{"day":31,"month":5,"partition":0,"timestamp":"2025-05-31T09:50:13.115+00:00","year":2025}	123-45-6789	{"age":21,"firstName":"Aloysius","lastName":"Alfalfa"}
{"day":31,"month":5,"partition":0,"timestamp":"2025-05-31T09:50:13.115+00:00","year":2025}	123-12-1234	{"age":52,"firstName":"University","lastName":"Alfred"}
{"day":31,"month":5,"partition":0,"timestamp":"2025-05-31T09:50:13.115+00:00","year":2025}	567-89-0123	{"age":35,"firstName":"Gamma","lastName":"Gerty"}
{"day":31,"month":5,"partition":0,"timestamp":"2025-05-31T09:50:13.115+00:00","year":2025}	087-65-4321	{"age":23,"firstName":"Electric","lastName":"Android"}
{"day":31,"month":5,"partition":0,"timestamp":"2025-05-31T09:50:13.115+00:00","year":2025}	456-78-9012	{"age":72,"firstName":"Fred","lastName":"Bumpkin"}
{"day":31,"month":5,"partition":0,"timestamp":"2025-05-31T09:50:13.115+00:00","year":2025}	234-56-7890	{"age":44,"firstName":"Betty","lastName":"Rubble"}
{"day":31,"month":5,"partition":0,"timestamp":"2025-05-31T09:50:13.115+00:00","year":2025}	345-67-8901	{"age":67,"firstName":"Cecil","lastName":"Noshow"}
{"day":31,"month":5,"partition":0,"timestamp":"2025-05-31T09:50:13.115+00:00","year":2025}	632-79-9939	{"age":38,"firstName":"Buff","lastName":"Bif"}
{"day":31,"month":5,"partition":0,"timestamp":"2025-05-31T09:50:13.115+00:00","year":2025}	223-45-6789	{"age":42,"firstName":"Andrew","lastName":"Airpump"}
{"day":31,"month":5,"partition":0,"timestamp":"2025-05-31T09:50:13.115+00:00","year":2025}	143-12-1234	{"age":63,"firstName":"Jim","lastName":"Backus"}
{"day":31,"month":5,"partition":0,"timestamp":"2025-05-31T09:50:13.115+00:00","year":2025}	565-89-0123	{"age":29,"firstName":"Art","lastName":"Carnivore"}
{"day":31,"month":5,"partition":0,"timestamp":"2025-05-31T09:50:13.115+00:00","year":2025}	087-75-4321	{"age":56,"firstName":"Jim","lastName":"Dandy"}
{"day":31,"month":5,"partition":0,"timestamp":"2025-05-31T09:50:13.115+00:00","year":2025}	456-71-9012	{"age":45,"firstName":"Ima","lastName":"Elephant"}
{"day":31,"month":5,"partition":0,"timestamp":"2025-05-31T09:50:13.115+00:00","year":2025}	234-56-2890	{"age":54,"firstName":"Benny","lastName":"Franklin"}
{"day":31,"month":5,"partition":0,"timestamp":"2025-05-31T09:50:13.115+00:00","year":2025}	345-67-3901	{"age":91,"firstName":"Boy","lastName":"George"}
{"day":31,"month":5,"partition":0,"timestamp":"2025-05-31T09:50:13.115+00:00","year":2025}	632-79-9439	{"age":17,"firstName":"Harvey","lastName":"Heffalump"}
```

## Search

Search is a protocol buffer backedd topic, with the following schema [search.proto](schema/search.proto):

```proto
syntax = 'proto3';

enum Corpus {
  CORPUS_UNSPECIFIED = 0;
  CORPUS_UNIVERSAL = 1;
  CORPUS_WEB = 2;
  CORPUS_IMAGES = 3;
  CORPUS_LOCAL = 4;
  CORPUS_NEWS = 5;
  CORPUS_PRODUCTS = 6;
  CORPUS_VIDEO = 7;
}

message Value {
  string query = 1;
  int32 page_number = 2;
  int32 results_per_page = 3;
  Corpus corpus = 4;
}
```

Sample search data, is in [searches.json](data/searches.json):

```json
[
  {
    "value": {
      "query": "abc/def",
      "page_number": 6,
      "results_per_page": 13,
      "corpus": "CORPUS_WEB"
    }
  }
]
```

Create the search topic:

```bash
just search-topic-create
```

Publish the sample data onto the search topic:

```bash
just search-produce
```

View the data in Spark SQL:

```bash
just spark-sql
```

Query:

```sql
select * from tansu.search;
```

With output:

```text
spark-sql ()> select * from tansu.search;
{"partition":0,"timestamp":2025-05-31 09:50:57.027,"year":2025,"month":5,"day":31}	{"query":"abc/def","page_number":6,"results_per_page":13,"corpus":2}
```

## Taxi

Taxi is a protocol buffer backed topic, with the following schema [taxi.proto](schema/taxi.proto):

```proto
syntax = 'proto3';

enum Flag {
    N = 0;
    Y = 1;
}

message Value {
  int64 vendor_id = 1;
  int64 trip_id = 2;
  float trip_distance = 3;
  double fare_amount = 4;
  Flag store_and_fwd = 5;
}
```

Sample trip data, is in [trips.json](data/trips.json):

```json
[
  {
    "value": {
      "vendor_id": 1,
      "trip_id": 1000371,
      "trip_distance": 1.8,
      "fare_amount": 15.32,
      "store_and_fwd": "N"
    }
  },
  ...
]
```

Create the taxi topic:

```bash
just taxi-topic-create
```

Publish the sample data onto the taxi topic:

```bash
just taxi-produce
```

View the data in Spark SQL:

```bash
just spark-sql
```

Query:

```sql
select * from tansu.taxi;
```

With output:

```text
spark-sql ()> select * from tansu.taxi;
{"partition":0,"timestamp":2025-05-31 09:51:50.985,"year":2025,"month":5,"day":31}	{"vendor_id":1,"trip_id":1000371,"trip_distance":1.8,"fare_amount":15.32,"store_and_fwd":0}
{"partition":0,"timestamp":2025-05-31 09:51:50.985,"year":2025,"month":5,"day":31}	{"vendor_id":2,"trip_id":1000372,"trip_distance":2.5,"fare_amount":22.15,"store_and_fwd":0}
{"partition":0,"timestamp":2025-05-31 09:51:50.985,"year":2025,"month":5,"day":31}	{"vendor_id":2,"trip_id":1000373,"trip_distance":0.9,"fare_amount":9.01,"store_and_fwd":0}
{"partition":0,"timestamp":2025-05-31 09:51:50.985,"year":2025,"month":5,"day":31}	{"vendor_id":1,"trip_id":1000374,"trip_distance":8.4,"fare_amount":42.13,"store_and_fwd":1}
```
