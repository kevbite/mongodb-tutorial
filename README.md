# MongoDB Tutorial

This tutorial is meant to be a beginners guide to using MongoDB.

## Get Started

The MongoDB Community Edition is free and available for Windows, Linux, and macOS.

### Installing MongoDB Community Edition

Follow the follow installation guides to get MongoDB installed locally:

- [Linux](https://docs.mongodb.com/manual/administration/install-on-linux/)
- [macOS](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-os-x/)
- [Windows](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-windows/)

## MongoDB Shell

### Running the MongoDB Shell

To connect to a database instance within MongoDB you can use the MongoDB shell. This is normally found in the `bin` directory of the mongodb install directory (`<mongodb installation dir>/bin`).

Once inside the bin directory type `mongo` this is defaulted to connect to the local instance of the database running on your machine. Once connected you'll get some information regarding the server that you're connected to:

```bash
C:\mongodb\bin>mongo
MongoDB shell version v4.0.4
connecting to: mongodb://127.0.0.1:27017
Implicit session: session { "id" : UUID("a7df4f06-c81b-44b8-bcb3-1cada87e0bb1") }
MongoDB server version: 4.0.4
```

### Playing with MongoDB Shell

The mongo shell is an interactive JavaScript interface to MongoDB. That means we can run any arbitrary JavaScript! This makes the shell very powerful as we can write complex scripts to interact with MongoDB without any other tooling.

Let's try writing some basic JavaScript, we'll start by printing `"Hello World"`.

```javascript
> print('Hello World')
Hello World
```

Now let's do some basic calculations:

```javascript
> 1 + 2 * 3
7
```

Let's try to use the Maths functions inside JavaScript, these will work as well:
```javascript
> Math.pow(10, 1000);
Infinity
```

We can also declare functions that we can reuse, below is [Fizz Buzz](https://en.wikipedia.org/wiki/Fizz_buzz)

```javascript
> const fizzBuzz = input => {
  if (input % 15 === 0) return 'FizzBuzz';
   if (input % 3 === 0) return 'Fizz';
   if (input % 5 === 0) return 'Buzz';
   return input;
 };
> fizzBuzz(2)
2
> fizzBuzz(30)
FizzBuzz
```

We can also use arrays which comes in handy when receiving lists of documents from the database.

```javascript
> [0, 1, 2, 3, 4, 5].map(fizzBuzz).join(', ');
FizzBuzz, 1, 2, Fizz, 4, Buzz
```

Feel free to try out more JavaScript commands inside the MongoDB Shell.

## Database Interaction

To interact with the database, the shell gives us a few global variable helpers. The main one we'll be using is `db`. This is how we'll interact with the current database we're connected to.

### Database

Let's start by just executing `db`:

```javascript
> db
test
```

This will return the database that we're connected to.
We can swap the database with another shell helper called `use`, if you've used any other database this might be familiar.

```javascript
> use my-test
switched to db my-test
> db
my-test
```

### Collections

Within a database you'll find collections, collections are somewhat similar to tables within a relational database, so you might have a collection to store `vehicles` and another one to store `drivers`.

These collections are accessed via the `db` global object, everything is created  dynamically when required within MongoDB so we are not required to upfront declare a collection, we can just use it.

So let's just query a `vehicles` and a `drivers` collection within the `test` database.

```javascript
// change to test database
> use test
switched to db test

// query our vehicles and drivers
> db.vehicles.find()
> db.drivers.find()
```

You'll notice that there is no error because there is no collection, this is one of MongoDB benefits of how easy it is to get going without any complex setup! But as you may have notice nothing is returned, this is because we've got nothing in these collections yet!

### Help!

Within MongoDB the easiest way to find out what you can do on a certain object is to execute the `help()` method on it. Try executing `help()` on our `db` and our `vehicles` collections and see what you see!


```javascript
> db.help()

> db.vehicles.help()

```

Also, MongoDB has one of the best documentation sites out there with pretty much all the information you'll ever need. We'll refer back to the documentation in some of these sections but feel free to take a browse: [The MongoDB Manual](https://docs.mongodb.com/manual/)

## Be a Developer

We're going to go though some basic developer operations that you might need to execute while running MongoDB.

### Inserting data

To insert data in to MongoDB we can execute 2 functions on our collection `insertOne()` and `insertMany()`. 

To insert a single document in to our vehicles collection call the `insertOne` function passing in an object:

```javascript
> db.vehicles.insertOne({type: 'Car', year: 2018, make: 'Honda', model: 'Civic', registrationNumber: 'CU57ABC', colour: 'red'})
{
  "acknowledged" : true,
  "insertedId" : ObjectId("5d17d3d455fa784abd28f481")
}
```

As you'll notice we get a `insertedId` return this is the server generated `_id` to reference our document because we didn't specify one, If we specify one it will use it.

```javascript
> db.vehicles.insertOne({ _id: ObjectId("5d17d5ef55fa784abd28f482"), type: 'Car', year: 2018, make: 'Honda', model: 'Civic', registrationNumber: 'CU57ABC', colour: 'red' })
{
  "acknowledged" : true,
  "insertedId" : ObjectId("5d17d5ef55fa784abd28f482")
}
```

We can also insert documents that have completely different properties, for our `vehicles` collection let's insert in some bicycles!

```javascript
> db.vehicles.insertOne({ type: 'Bicycle', bicycleType: 'Road Bike', year: 2019, make: 'Giant', model: 'Contend 1', colour: 'red' })
{
  "acknowledged" : true,
  "insertedId" : ObjectId("5d17d70a55fa784abd28f483")
}
```

This can be very beneficial when building applications that are required to store data of different structures.

We can also insert many documents at once using the `insertMany()` function, let's just insert a bunch of vehicles in to our database:

```javascript
> db.vehicles.insertMany([
   { type: 'Car', year: 2018, make: 'Honda', model: 'Civic', registrationNumber: 'CE57ABC', colour: 'blue' },
   { type: 'Car', year: 2018, make: 'Honda', model: 'Civic', registrationNumber: 'EU57ABC', colour: 'blue' },
   { type: 'Car', year: 2019, make: 'Honda', model: 'Jazz', registrationNumber: 'BD57ABC', colour: 'white' },
   { type: 'Car', year: 2019, make: 'Peugeot', model: '208', registrationNumber: 'BD57DEF', colour: 'yellow' },
   { type: 'Car', year: 2019, make: 'Renault', model: 'Clio', registrationNumber: 'TT57DEF', colour: 'red' },
   { type: 'Bicycle', bicycleType: 'Road Bike', year: 2019, make: 'Liv', model: 'Avail 3', colour: 'blue' },
   { type: 'Bicycle', bicycleType: 'Road Bike', year: 2019, make: 'Specialized', model: 'Allex E5', colour: 'red' },
   { type: 'Bicycle', bicycleType: 'Mountain Bike', year: 2015, make: 'Specialized', model: 'Rockhopper Expert', colour: 'sliver' }
 ]);
{
 "acknowledged" : true,
 "insertedIds" : [
   ObjectId("5d17dac755fa784abd28f484"),
   ObjectId("5d17dac755fa784abd28f485"),
   ObjectId("5d17dac755fa784abd28f486"),
   ObjectId("5d17dac755fa784abd28f487"),
   ObjectId("5d17dac755fa784abd28f488"),
   ObjectId("5d17dac755fa784abd28f489"),
   ObjectId("5d17dac755fa784abd28f48a"),
   ObjectId("5d17dac755fa784abd28f48b")
 ]
}
```

### Querying data

To query data inside MongoDB we can call the `find` function on the collection and pass in a query. The query is a way to declare what we want to find. Let's start by finding our vehicle that we inserted in the previous section with the `_id` of `ObjectId("5d17d5ef55fa784abd28f482")`.

We'll pass in the an object with an `_id` set to our id to find:

```javascript
> db.vehicles.find({ _id: ObjectId("5d17d5ef55fa784abd28f482") })
{ "_id" : ObjectId("5d17d5ef55fa784abd28f482"), "type" : "Car", "make" : "Honda", "model" : "Civic", "registrationNumber" : "CU57ABC", "colour" : "red" }
```

You may also want to create a separate variables in JavaScript that contain your query so you can use them again and again:
```javascript
> var query = { _id: ObjectId("5d17d5ef55fa784abd28f482") };
> db.vehicles.find(query);
{ "_id" : ObjectId("5d17d5ef55fa784abd28f482"), "type" : "Car", "make" : "Honda", "model" : "Civic", "registrationNumber" : "CU57ABC", "colour" : "red" }
```

> You can make the documents returned easier to read by adding a call to `pretty()` on the end of your find method:
>```javascript
> db.vehicles.find(query).pretty();
>```

The above query is using the short hand for the [`$eq` Operator](https://docs.mongodb.com/manual/reference/operator/query/eq/#op._S_eq), this can alternatively be written like the follow query:

```javascript
> var query = { _id: { $eq: ObjectId("5d17d5ef55fa784abd28f482") } };
```

MongoDB has a vast number of query operators which allows you to filter documents in many different ways, the full list of the operators can be found on their [documentation](https://docs.mongodb.com/manual/reference/operator/query/).

Let's now change this around to query for all bicycles:

```javascript
var query = { type: 'Bicycle' };
```

We may also want to use a [less than operator](https://docs.mongodb.com/manual/reference/operator/query/lt/#op._S_lt) to only return bicycles that are less than the year of 2019.

```javascript
var query = { type: 'Bicycle', year: { $lt: 2019 } };
```

Then we can also extend our query to return all vehicles that are of the colour of yellow as well. We can achieve this by using the [or operator]():

```javascript
var query = {
  $or: [
    { type: 'Bicycle', year: { $lt: 2019 }},
    { colour: 'yellow' }
  ]
};
```

Now try build up some of your own queries to query our vehicles collection.

### Updating data

Updating data is similar to query it, we'll use a function called `update` on the collection.

We first use the same queries used in finding the data but then we specify an update object on how we want to update the document. These update objects are built up of [update operators](https://docs.mongodb.com/manual/reference/operator/update/).

Let's change the colour of our vehicle with the `_id` of `ObjectId("5d17d5ef55fa784abd28f482")` to `'orange'`.

```javascript
> var query = { _id: ObjectId("5d17d5ef55fa784abd28f482") };
> var update = { $set: { 'colour': 'orange' } };
>
> db.vehicles.updateOne(query, update);
{ "acknowledged" : true, "matchedCount" : 1, "modifiedCount" : 1 }
>
> db.vehicles.find(query).pretty()
{
        "_id" : ObjectId("5d17d5ef55fa784abd28f482"),
        "type" : "Car",
        "make" : "Honda",
        "model" : "Civic",
        "registrationNumber" : "CU57ABC",
        "colour" : "orange"
}
```

Perfect, now we've got an orange car!

Feel free to make up some of your own updates to the documents.


## Querying parkrun Data

### Connect to the database

We'll need to drop out the current shell and create a new session to a database hosted in [MongoDB Atlas](https://www.mongodb.com/cloud/atlas). Atlas is a SaaS offering of MongoDB provided by MongoDB.

To connect to the database run the following command but replace the `<username>` and `<password>` with the given details.

```bash
mongo "mongodb+srv://cluster0-dxyc7.mongodb.net/parkrun-map" --username <username> --password <password>
```

Once connected run the following to see the structure of the parkrun data:

```javascript
> db.parkruns.findOne();
```

Now use the data inside mongodb with the knowledge you've got so far to query the parkrun dataset to find answers to the following questions.

### Questions

1. What is the name of the parkrun with the `_id` of `ObjectId("5c2a324534656f06e4bd21c6")`?

2. What is the location (lat/lon coordinates) of `Polokwane` parkrun in `South Africa`?

3. What parkrun name starts with the letter `Ö`?

4. Where outside of the UK has a parkrun with the name starting with `Vall`?

5. Which 2 parkruns have had attendance of over 995?

6. Which parkrun was cancelled due to a `Soap Box Race`?

7. How many parkruns have showers that people can use?

8. How many countries have parkruns?

9. Which country has the most amount of parkruns?

10. Which country has the least amount of parkruns?

11. Which country has the fastest average seconds ran for the whole country?

12. What is total events run for the whole of `Italy`?

