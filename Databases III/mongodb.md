# MongoDB
=========

## Exercise 5

importing `JSON` data into mongo:

    cd /var/lib/mongo
    mongoimport --db test --collection collectionName /location/of/JSON.json

### Aggregation functions

Counting all records: with "param" = "something"

    db.collectionName.count({"param": "something"})

Give a list of all unique elements in the database

    db.collectionName.distinct("param")


