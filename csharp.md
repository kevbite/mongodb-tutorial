# MongoDB Tutorial

This tutorial is meant to be a beginners guide to using MongoDB.

## Get Started

The fastest way to connect to a MongoDB instance is to use Docker.

### Installing Docker Desktop

Follow the follow installation guides to get Docker Desktop installed locally:

[Docker Desktop](https://www.docker.com/products/docker-desktop/)

### Playing with MongoDB with C#

#### Installing the .NET SDK

Before we get started, you'll need to have the .NET SDK installed, which can be installed via the download site:

[Download .NET SDK](https://dotnet.microsoft.com/en-us/download)

#### Project Setup

Let's first create a directory where we'll add our C# project.

```bash
mkdir C:\dev\MongoDBCSharp
cd C:\dev\MongoDBCSharp
```

We'll then create a new console app using the `dotnet new` command within the .NET CLI.

```bash
dotnet new console

The template "Console App" was created successfully.

Processing post-creation actions...
Restoring C:\dev\MongoDBCSharp\MongoDBCSharp.csproj:
  Determining projects to restore...
  Restored C:\dev\MongoDBCSharp\MongoDBCSharp.csproj (in 98 ms).
Restore succeeded.
```

We'll then need to add a dependacy from NuGet so that we can connect to a MongoDB instance. Again this can be done via the CLI.

```bash
dotnet add package MongoDB.Driver
```

We'll also add a second dependacy which will help us output our results to the console called `Dumpify`.

```bash
dotnet add package Dumpify
```

#### Database Interaction

To do the next section we'll need to start up a local MongoDB instance, this again can be done via docker with the offical `mongo` image.

```bash
docker run -p 27017:27017 mongo:latest
```

To interact with the database, the C# driver gives us a few objects to interact with the database.

##### MongoClient

The `MongoClient` is the client for interacting to any MongoDB instance.

```csharp
var client = new MongoClient();
```
##### IMongoDatabase

The `IMongoDatabase` is the object for interacting to a single database within a MongoDB instance.

```csharp
var database = client.GetDatabase("test");
```

##### IMongoCollection<T>

The `IMongoCollection<T>` is the object used for interacting to a single collection within a MongoDB instance.
You'll notice this is a generic method, meaning that we can use standard [C# POCOs](https://en.wikipedia.org/wiki/Plain_old_CLR_object), but for the time being we'll use a BsonDocument which is a representation of BSON Document.

```csharp
var collection = database.GetCollection<BsonDocument>("vehicles");
```

### Querying Collections

Within a database you'll find collections, collections are somewhat similar to tables within a relational database, so you might have a collection to store `vehicles` and another one to store `drivers`.

When accessing the collection everything is created dynamically when required within MongoDB so we are not required to upfront declare a collection, we can just use it.

So let's just query a `vehicles` and a `drivers` collection within the `test` database.

Can your `Program.cs` file to the following code:

```csharp

using Dumpify;
using MongoDB.Bson;
using MongoDB.Driver;

var client = new MongoClient();

// Get the test database
var database = client.GetDatabase("test");

// Get the vehicles and drivers collection
var vehicles = database.GetCollection<BsonDocument>("vehicles");
var drivers = database.GetCollection<BsonDocument>("drivers");

// Find items in our collection
var results1 = await vehicles.Find(Builders<BsonDocument>.Filter.Empty).ToListAsync();
var results2 = await drivers.Find(Builders<BsonDocument>.Filter.Empty).ToListAsync();

// Output Results
results1.Dump();
results2.Dump();
```

You'll notice that there is no error because there is no collection, this is one of MongoDB benefits of how easy it is to get going without any complex setup! But as you may have notice nothing is returned, this is because we've got nothing in these collections yet!

MongoDB has one of the best documentation sites out there with pretty much all the information you'll ever need. We'll refer back to the documentation in some of these sections but feel free to take a browse: [The MongoDB Manual](https://docs.mongodb.com/manual/)

## Be a Developer

We're going to go though some basic developer operations that you might need to execute while running MongoDB.

### Inserting data

To insert data in to MongoDB we can execute 2 functions on our collection `InsertOneAsync()` and `InsertManyAsync()`. 

To insert a single document in to our vehicles collection call the `InsertOneAsync` function passing in an object:

```csharp
var bsonDocument = new BsonDocument
{
    ["type"] = "Car",
    ["year"] = "01",
    ["make"] = "Honda",
    ["model"] = "Civic",
    ["registrationNumber"] = "CU57ABC",
    ["colour"] = "red"
};
await vehicles.InsertOneAsync(bsonDocument);
bsonDocument.ToJson().Dump();
```
Now we can run our program using `dotnet run`.
```bash
dotnet run

"{ "_id" : ObjectId("64539b557936d6b555b7a354"), "type" : "Car", "year" : "01", "make" : "Honda", "model" : "Civic",
"registrationNumber" : "CU57ABC", "colour" : "red" }"
```

As you'll notice that our `BsonDocument` document now has an extra property of `_id` which was set by the driver once the insert completed.
We can also explicity set an `_id` of our choice and it'll use that, this can be of any support type too.

```csharp
var bsonDocument = new BsonDocument
{
    ["_id"] = 1,
    ["type"] = "Car",
    ["year"] = "01",
    ["make"] = "Honda",
    ["model"] = "Civic",
    ["registrationNumber"] = "CU57ABC",
    ["colour"] = "red"
};
await vehicles.InsertOneAsync(bsonDocument);
bsonDocument.ToJson().Dump();
```
```bash
dotnet run

"{ "_id" : 1, "type" : "Car", "year" : "01", "make" : "Honda", "model" : "Civic", "registrationNumber" : "CU57ABC",
"colour" : "red" }"
```

We can also insert documents that have completely different properties, for our `vehicles` collection let's insert in some bicycles!

```csharp
var bicycle = new BsonDocument
{
    ["type"] = "Bicycle",
    ["bicycleType"] = "Road Bike",
    ["year"] = 2019,
    ["make"] = "Giant",
    ["model"] = "Contend 1",
    ["colour"] = "red"
};
await vehicles.InsertOneAsync(bicycle);
```

This can be very beneficial when building applications that are required to store data of different structures.

We can also insert many documents at once using the `insertMany()` function, let's just insert a bunch of vehicles in to our database:

```csharp
var input = new[]
{
    new BsonDocument
    {
        ["type"] = "Car", ["year"] = 2018, ["make"] = "Honda", ["model"] = "Civic", ["registrationNumber"] = "CE57ABC", ["colour"] = "blue"
    },
    new BsonDocument
    {
        ["type"] = "Car", ["year"] = 2018, ["make"] = "Honda", ["model"] = "Civic", ["registrationNumber"] = "EU57ABC", ["colour"] = "blue"
    },
    new BsonDocument
    {
        ["type"] = "Car", ["year"] = 2019, ["make"] = "Honda", ["model"] = "Jazz", ["registrationNumber"] = "BD57ABC", ["colour"] = "white"
    },
    new BsonDocument
    {
        ["type"] = "Car", ["year"] = 2019, ["make"] = "Peugeot", ["model"] = "208", ["registrationNumber"] = "BD57DEF", ["colour"] = "yellow"
    },
    new BsonDocument
    {
        ["type"] = "Car", ["year"] = 2019, ["make"] = "Renault", ["model"] = "Clio", ["registrationNumber"] = "TT57DEF", ["colour"] = "red"
    },
    new BsonDocument
    {
        ["type"] = "Bicycle", ["bicycleType"] = "Road Bike", ["year"] = 2019, ["make"] = "Liv", ["model"] = "Avail 3", ["colour"] = "blue"
    },
    new BsonDocument
    {
        ["type"] = "Bicycle", ["bicycleType"] = "Road Bike", ["year"] = 2019, ["make"] = "Specialized", ["model"] = "Allex E5", ["colour"] = "red"
    },
    new BsonDocument
    {
        ["type"] = "Bicycle", ["bicycleType"] = "Mountain Bike", ["year"] = 2015, ["make"] = "Specialized", ["model"] = "Rockhopper Expert", ["colour"] = "sliver"
    },
};

await vehicles.InsertManyAsync(input);
```

### Querying data

To query data inside MongoDB we can call the `find` function on the collection and pass in a query. The query is a way to declare what we want to find. Let's start by finding our vehicle that we inserted in the previous section with the `_id`.

We'll pass in the an object with an `_id` set to our id to find:

```csharp
var idToFind = 1;
var result = await vehicles.Find(Builders<BsonDocument>.Filter.Eq(x => x["_id"], idToFind)).SingleAsync();
result.ToJson().Dump();
```
```bash
dotnet run
"{ "_id" : 1, "type" : "Car", "year" : "01", "make" : "Honda", "model" : "Civic", "registrationNumber" : "CU57ABC",
"colour" : "red" }"
```

As you can see we're using `Builders<BsonDocument>.Filter` this allows us to build up complex filter queries.

The above query is using the [`$eq` Operator](https://docs.mongodb.com/manual/reference/operator/query/eq/#op._S_eq), this can alternatively be written like the follow query:

MongoDB has a vast number of query operators which allows you to filter documents in many different ways, the full list of the operators can be found on their [documentation](https://docs.mongodb.com/manual/reference/operator/query/), which are all reflected on the `Builders<BsonDocument>.Filter` as factory methods.

Let's now change this around to query for all bicycles:

```csharp
var filter = Builders<BsonDocument>.Filter.Eq(x => x["type"], "Bicycle");
var result = await vehicles.Find(filter).ToListAsync();
result.ToJson(new JsonWriterSettings { Indent = true }).Dump();
```
```bash
dotnet run
"[{
    "_id" : ObjectId("64539e2e851f98b8186fe829"),
    "type" : "Bicycle",
    "bicycleType" : "Road Bike",
    "year" : 2019,
    "make" : "Liv",
    "model" : "Avail 3",
    "colour" : "blue"
  }, {
    "_id" : ObjectId("64539e2e851f98b8186fe82a"),
    "type" : "Bicycle",
    "bicycleType" : "Road Bike",
    "year" : 2019,
    "make" : "Specialized",
    "model" : "Allex E5",
    "colour" : "red"
  }, {
    "_id" : ObjectId("64539e2e851f98b8186fe82b"),
    "type" : "Bicycle",
    "bicycleType" : "Mountain Bike",
    "year" : 2015,
    "make" : "Specialized",
    "model" : "Rockhopper Expert",
    "colour" : "sliver"
  }]"
```
We may also want to use a [less than operator](https://docs.mongodb.com/manual/reference/operator/query/lt/#op._S_lt) to only return bicycles that are less than the year of 2019.

```javascript
var filter = Builders<BsonDocument>.Filter.Eq(x => x["type"], "Bicycle")
             & Builders<BsonDocument>.Filter.Lt(x => x["year"], 2019);
var result = await vehicles.Find(filter).ToListAsync();
result.ToJson(new JsonWriterSettings { Indent = true }).Dump();
```
```bash
dotnet run
"[{
    "_id" : ObjectId("64539e2e851f98b8186fe82b"),
    "type" : "Bicycle",
    "bicycleType" : "Mountain Bike",
    "year" : 2015,
    "make" : "Specialized",
    "model" : "Rockhopper Expert",
    "colour" : "sliver"
  }]"
```
Then we can also extend our query to return all vehicles that are of the colour of yellow as well. We can achieve this by using the [or operator]():

```csharp
var bicyclesLessThan2019Filter = Builders<BsonDocument>.Filter.Eq(x => x["type"], "Bicycle")
                       & Builders<BsonDocument>.Filter.Lt(x => x["year"], 2019);
var filter = bicyclesLessThan2019Filter | Builders<BsonDocument>.Filter.Eq(x => x["colour"], "yellow");
var result = await vehicles.Find(filter).ToListAsync();
result.ToJson(new JsonWriterSettings { Indent = true }).Dump();
```
```bash
dotnet run
"[{
    "_id" : ObjectId("64539e2e851f98b8186fe827"),
    "type" : "Car",
    "year" : 2019,
    "make" : "Peugeot",
    "model" : "208",
    "registrationNumber" : "BD57DEF",
    "colour" : "yellow"
  }, {
    "_id" : ObjectId("64539e2e851f98b8186fe82b"),
    "type" : "Bicycle",
    "bicycleType" : "Mountain Bike",
    "year" : 2015,
    "make" : "Specialized",
    "model" : "Rockhopper Expert",
    "colour" : "sliver"
  }]"
```
Now try build up some of your own queries to query our vehicles collection.

### Updating data

Updating data is similar to query it, we'll use a function called `UpdateOneAsync` on the collection.

We first use the same queries used in finding the data but then we specify an update object on how we want to update the document. These update objects are built up of [update operators](https://docs.mongodb.com/manual/reference/operator/update/).

Let's change the colour of our vehicle with the `_id` of `1` to `'orange'`.

```javascript
var filter = Builders<BsonDocument>.Filter.Eq(x => x["_id"], 1);
var update = Builders<BsonDocument>.Update.Set(x => x["colour"], "orange");

var updateResult = await vehicles.UpdateOneAsync(filter, update);
updateResult.Dump();

var doc = await vehicles.Find(filter).SingleAsync();
doc.ToJson().Dump();
```
```bash
dotnet run
            Acknowledged
┌──────────────────────────┬───────┐
│ Name                     │ Value │
├──────────────────────────┼───────┤
│ IsAcknowledged           │ True  │
│ IsModifiedCountAvailable │ True  │
│ MatchedCount             │ 1     │
│ ModifiedCount            │ 1     │
│ UpsertedId               │ null  │
└──────────────────────────┴───────┘

"{ "_id" : 1, "type" : "Car", "year" : "01", "make" : "Honda", "model" : "Civic", "registrationNumber" : "CU57ABC",
"colour" : "orange" }"
```
Perfect, now we've got an orange car!

Feel free to make up some of your own updates to the documents.


## Querying parkrun Data

See [README.md](README.md)
