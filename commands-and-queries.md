# Links
[MongoDB](https://www.mongodb.com/try/download/community)

[Mongo Database Tools](https://www.mongodb.com/try/download/tools)

[Mongo Query Operator Documentation](https://www.mongodb.com/docs/manual/reference/operator/query/)

[Mongo Aggregation Operator Documentation](https://www.mongodb.com/docs/manual/reference/operator/aggregation-pipeline/)

[Mongo BSON Types](https://www.mongodb.com/docs/v4.4/reference/operator/query/type/#available-types)

# Commands : 
`./mongod --dbpath ../data`

`./mongoimport --file acds-data-1.json --collection people --db acds --jsonArray`

`./mongoexport --collection people --db acds --out people-data-with-coordinates.json`

See : "Example of combining separate lat/long properties into an array suitable for a geo index"


# Query Operator Examples
$eq : 

`db.people.find({ "id": "3d8a29f5-2936-4ac9-9eb1-4af4b02c6b29"})`

`db.people.find({ "id": {$eq: "3d8a29f5-2936-4ac9-9eb1-4af4b02c6b29"}})`

$ne : `db.people.find({ "gender": {$ne: "Male"}})`

$exists : `db.people.find({ id: { $exists: true } })`

$gt / $lt :

`db.people.find({ age: { $gt: 40 }})`

`db.people.find({ age: { $lt: 21 }})`

$in / $nin : 

`db.people.find({ "address.state": { $in: ['AR', 'TX', 'MO', 'TN'] }})`

`db.people.find({ "address.state": { $nin: ['WY', 'KY', 'ND', 'MN'] }})`

$and : 

`db.people.find({ $and: [{"gender": { $ne: "Male" }}, { hobbies: "hiking" }]})`

$or : 

`db.people.find({ $or: [{"address.state": "AR"}, {"address.state": "TN"}]})`

$not : `db.people.find({ age: { $not: { $lt: 21 }}})`

$type : `db.people.find({ age: { $type: "int" }})`

$size : `db.people.find({ hobbies: { $size: 2 }})`

$all : `db.people.find({ hobbies: { $all: ["camping", "hiking"] }})`


# Aggregation Operator Examples
    

# Index Examples
`db.people.createIndex({keys, options})`

Special Types : 
1. Geospatial
2. Text
3. Hashed

Options : 
1. Unique
2. Partial
3. Sparse
4. TTL

# Basic Query Practice : 
Simple Queries : 
    People who like a certain color
    People who have a certain hobby
    People who have multiple hobbies
    People that are a certain age
    People that are in an age range
    Documents created within a date range
    Documents updated within a date range
    People updated by a certain user
    People who own a particular make/model vehicle
    People who own more than X number of vehicles
    Number of pets (0, 1, or more than X)
    People in an age range with a certain hobby

# Geo Index/Queries 


**Important** : Coordinates must be listed in longitude, latitude order!

Geo Index : `db.people.createIndex({coords: "2dsphere"})`

Example of combining separate lat/long properties into an array suitable for a geo index : 
`db.getCollection('people').updateMany({}, [{"$set": {"coords": ["$longitude", "$latitude"]}}])`

Geo Queries : 

Arkansas Polygon Coordinates : 
36.526802, -94.654032 northwest
36.472899, -90.019774 northeast
33.033187, -91.022933 southeast
32.942091, -94.114652 southwest

Little Rock Coordinates : 
34.747459, -92.329080

Find people (approximately) in Arkansas via Polygon :

db.getCollection('people').find({
     coords: {
       $geoWithin: {
          $polygon: 
             [ [ -94.654032, 36.526802 ], [ -90.019774, 36.472899 ], 
              [ -91.022933, 33.033187 ], [ -94.114652, 32.942091 ] ] 
       }
     }
   });


Find people in a circle around Little Rock (with ""units"") : 

db.getCollection('people').find({
     coords: {
       $geoWithin: {
          $center: 
             [ [-92.329080, 34.747459], 2] 
       }
     }
   })


Find people in a 100 mile circle around Little Rock (with radian units)
db.getCollection('people').find({
     coords: {
       $geoWithin: {
          $centerSphere: 
             [ [-92.329080, 34.747459], 100 / 3963.2] 
       }
     }
   })


A better way : 

db.getCollection('people').find({
     coords: {
       $nearSphere: {
          $geometry: {
              type: "Point",
              coordinates: [-92.329080, 34.747459]
          },
          $minDistance: 0,
          $maxDistance: 15000
       }
     }
   })




Aggregates : 

Be sure to cover : 
    $match
    $addFields
    $count
    $group
    $limit / $skip
    $lookup
    $project
    $set / $unset
    $sort
    $unwind


    
    

Maybe : 
    $graphLookup
    $out
    $sample
    

Queries : 
    Number of people in a state
    List of unique email addresses
    Average/min/max age of people who enjoy a particular hobby
    Most popular hobby
    Most popular favorite color
    unwind example?



GEO : https://www.mongodb.com/docs/manual/reference/operator/query/#geospatial
    $geoNear : https://www.mongodb.com/docs/manual/reference/operator/aggregation/geoNear/#mongodb-pipeline-pipe.-geoNear

    Distance between points
    Two closest people
    Two furthest away people
    People closest to a specified location (and what that distance is)
