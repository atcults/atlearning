---
sidebar_position: 1
---

# MongoDB

# With nodeJS

MongoDB & Node.js: Connecting & CRUD Operations (Part 1 of 4) - YouTube

<iframe width="560" height="315" src="https://www.youtube.com/embed/fbYExfeFsI0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

MongoDB & Node.js: Aggregation & Data Analysis (Part 2 of 4) - YouTube

<iframe width="560" height="315" src="https://www.youtube.com/embed/iz37fDe1XoM" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

MongoDB & Node.js: Create an ACID Transaction (Part 3 of 4) - YouTube

<iframe width="560" height="315" src="https://www.youtube.com/embed/bdS03tgD2QQ" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

MongoDB & Node.js: Change Streams & Triggers (Part 4 of 4) - YouTube

<iframe width="560" height="315" src="https://www.youtube.com/embed/9LA7_CSyZb8" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

# Dotnet

[Working with MongoDB in .NET (Part 2): Retrieving Documents with Filter Clause | Codementor](https://www.codementor.io/@pmbanugo/working-with-mongodb-in-net-2-retrieving-mrlbeanm5)

<!-- **Working with MongoDB in .NET (Part 2): Retrieving Documents with Filter Clause**

Published Dec 13, 2016Last updated Jan 19, 2017

![Mongo DB](/img/mongo-db.jpg)

In the previous part we went through some of the driver basics and how to insert documents to a collection. In this part of the series, we'll learn how to retrieve documents from the database.

Any document belongs to a collection, therefore all CRUD operations have the scope of a single collection. To retrieve documents from a collection, we can use the `Find`, `FindSync`, and  `FindAsync` functions.

# FindSync & FindAsync

`FindSync`, and  `FindAsync` both have two overloads with three parameters. Both `FindSync`, and  `FindAsync` are somewhat similar except that `FindSync` is synchronous and blocks until it's call is complete. `FindSync`  returns an  `IAsyncCursor` while `FindAsync` returns a task of `IAsyncCursor`.

# What is IAsyncCursor

MongoDB returns a query result in batches, and the batch size will not exceed the maximum size of a BSON document. As of version 3.2, the maximum size of a BSON document is 16 megabytes. The maximum document size helps ensure that a single document cannot use excessive amount of RAM or, during transmission, excessive amount of bandwidth. This constraint also applies when adding documents to a collection, but in order to store larger documents, MongoDB has made GridFS API as a provision. For most queries, the first batch returns 101 documents or just enough document to exceed 1 megabyte, and subsequent batches will be 4MB. It's possible to override the default batch size, and we can do this from the driver by setting `BatchSize` property of the `FindOptions`, which is passed in as a second parameter to any of the find methods. So basically, a cursor is a pointer to the result set of a query.

By default, the server will automatically close the cursor after 10 minutes of inactivity or if the client has exhausted the cursor. To override this behavior, you can specify the `noTimeout` flag in your query using `NoCursorTimeout` property of the `FindOptions` class. However, you should either close the cursor manually or exhaust the cursor.

The `IAsyncCursor` from the driver represents an asynchronous cursor. To access the documents, we need to manually iterate the cursor.

# Retrieve documents

Let's build our first read query to give us all the documents in the `students` collection in our database. Update the `MainAsync` method with the following:

```
static async Task MainAsync()
{

  var client = new MongoClient();

  IMongoDatabase db = client.GetDatabase("school");

  var collection = db.GetCollection<BsonDocument>("students");

  using (IAsyncCursor<BsonDocument> cursor = await collection.FindAsync(new BsonDocument()))
  {
    while (await cursor.MoveNextAsync())
    {
      IEnumerable<BsonDocument> batch = cursor.Current;
      foreach (BsonDocument document in batch)
      {
        Console.WriteLine(document);
        Console.WriteLine();
      }
    }
  }
}
```

The first overload to any of the find method takes 3 parameters; a `FilterDefinition` which is used to define the filter for the query, an optional `FindOptions` for specifying options for the query (e.g cursor timeout, batch size, etc), and an optional `cancellationToken`.

In the code above, we specified an empty filter definition by passing an empty `BsonDocument` to the method. Another way of writing this would be to use `FilterDefinition<BsonDocument>.Empty` to indicate an empty filter. With an empty filter, we're basically telling it to give us all the documents in a collection. Afterward, we iterate over the cursor to get documents in batches (`MoveNextAsync` in the while loop), and call `cursor.Current` to get documents in the current batch, which then gets printed out.

Running the code above should give us all the documents we already have in that collection

```
{ "_id" : ObjectId("58469c732adc9f5370e50c9c"), "FirstName" : "Gregor", "LastName" : "Felix", "Class" : "JSS 3", "Age" : 23, "Subjects" : ["English", "Mathematics", "Physics", "Biology"] }

{ "_id" : ObjectId("58469c732adc9f5370e50c9d"), "FirstName" : "Machiko", "LastName" : "Elkberg", "Class" : "JSS 3", "Age" : 23, "Subjects" : ["English", "Mathematics", "Spanish"] }

{ "_id" : ObjectId("58469c732adc9f5370e50c9e"), "FirstName" : "Julie", "LastName" : "Sandal", "Class" : "JSS 1", "Age" : 25, "Subjects" : ["English", "Mathematics", "Physics", "Chemistry"] }

{ "_id" : ObjectId("58469c732adc9f5370e50c9f"), "FirstName" : "Peter", "LastName" : "Cyborg", "Class" : "JSS 1", "Age" : 39, "Subjects" : ["English", "Mathematics", "Physics", "Chemistry"] }

{ "_id" : ObjectId("58469c732adc9f5370e50ca0"), "FirstName" : "James", "LastName" : "Cyborg", "Class" : "JSS 1", "Age" : 39, "Subjects" : ["English", "Mathematics", "Physics", "Chemistry"] }

```

We can see a resemblance of the documents we added in the **[previous post](https://www.codementor.io/@pmbanugo/working-with-mongodb-in-net-1-basics-g4frivcvz)** but with a new property `_id`. All collections have a unique primary index on this field, and if you don't supply one when creating documents, MongoDB supplies one by default. It's of type `ObjectId`, and is defined in the Bson spec.

To demonstrate `FindOptions`, I'll add an option that restricts the batch size to 2 which will then display what batch we're looping through in the console. Update your code with the following

```
FilterDefinition<BsonDocument> filter = FilterDefinition<BsonDocument>.Empty;
FindOptions<BsonDocument> options = new FindOptions<BsonDocument>
{
  BatchSize = 2,
  NoCursorTimeout = false
};

using (IAsyncCursor<BsonDocument> cursor = await collection.FindAsync(filter, options))
{

                var batch = 0;
                while (await cursor.MoveNextAsync())
                {
                    IEnumerable<BsonDocument> documents = cursor.Current;
                    batch++;

                    Console.WriteLine($"Batch: {batch}");

                    foreach (BsonDocument document in documents)
                    {
                        Console.WriteLine(document);
                        Console.WriteLine();
                    }
                }

                Console.WriteLine($"Total Batch: { batch}");
}

```
#### And run it to get the following result:

```
Batch: 1
{ "_id" : ObjectId("58469c732adc9f5370e50c9c"), "FirstName" : "Gregor", "LastName" : "Felix", "Class" : "JSS 3", "Age" : 23, "Subjects" : ["English", "Mathematics", "Physics", "Biology"] }

{ "_id" : ObjectId("58469c732adc9f5370e50c9d"), "FirstName" : "Machiko", "LastName" : "Elkberg", "Class" : "JSS 3", "Age" : 23, "Subjects" : ["English", "Mathematics", "Spanish"] }

Batch: 2
{ "_id" : ObjectId("58469c732adc9f5370e50c9e"), "FirstName" : "Julie", "LastName" : "Sandal", "Class" : "JSS 1", "Age" : 25, "Subjects" : ["English", "Mathematics", "Physics", "Chemistry"] }

{ "_id" : ObjectId("58469c732adc9f5370e50c9f"), "FirstName" : "Peter", "LastName" : "Cyborg", "Class" : "JSS 1", "Age" : 39, "Subjects" : ["English", "Mathematics", "Physics", "Chemistry"] }

Batch: 3
{ "_id" : ObjectId("58469c732adc9f5370e50ca0"), "FirstName" : "James", "LastName" : "Cyborg", "Class" : "JSS 1", "Age" : 39, "Subjects" : ["English", "Mathematics", "Physics", "Chemistry"] }

Total Batch: 3

```
We can also write this in a shorter, cleaner way by calling `ToListAsync` or `ForEachAsync` to get all the documents from the cursor and put them in memory. There are extension methods on the `IAsyncCursor` available to do this. Here are a few snippets:

```
collection.FindSync(filter).ToList();

await collection.FindSync(filter).ToListAsync();

await collection.FindSync(filter).ForEachAsync(doc => Console.WriteLine());

collection.FindSync(filter).FirstOrDefault();

collection.FindSync(filter).FirstOrDefault();

await collection.FindSync(filter).FirstOrDefaultAsync();

```
This looks neat and shorter, code-wise, but what it does is force all the documents to live in-memory. This might not be ideal in certain scenarios, and the cursor is helpful when the query result is huge and we can move the cursor by calling `MoveNextAsync` or `MoveNext`.

# Find

This method is similar to its counterpart except that it returns an `IFindFluent` interface. This is a fluent interface that gives us simple syntax to things like Count , Skip , Sort , and Limit (more on this ahead). From the `IFindFluent` we can also return a cursor (by calling `ToCursor` or `ToCursorAsync` on it) or a list (by calling `ToList` or `ToListAsync`). With the code below, we can get all the documents and print them to the console using the `Find` method

```
await collection.Find(FilterDefinition<BsonDocument>.Empty)
        .ForEachAsync(doc => Console.WriteLine(doc));

```
**Result**

```
{ "_id" : ObjectId("58469c732adc9f5370e50c9c"), "FirstName" : "Gregor", "LastName" : "Felix", "Class" : "JSS 3", "Age" : 23, "Subjects" : ["English", "Mathematics", "Physics", "Biology"] }
{ "_id" : ObjectId("58469c732adc9f5370e50c9d"), "FirstName" : "Machiko", "LastName" : "Elkberg", "Class" : "JSS 3", "Age" : 23, "Subjects" : ["English", "Mathematics", "Spanish"] }
{ "_id" : ObjectId("58469c732adc9f5370e50c9e"), "FirstName" : "Julie", "LastName" : "Sandal", "Class" : "JSS 1", "Age" : 25, "Subjects" : ["English", "Mathematics", "Physics", "Chemistry"] }
{ "_id" : ObjectId("58469c732adc9f5370e50c9f"), "FirstName" : "Peter", "LastName" : "Cyborg", "Class" : "JSS 1", "Age" : 39, "Subjects" : ["English", "Mathematics", "Physics", "Chemistry"] }
{ "_id" : ObjectId("58469c732adc9f5370e50ca0"), "FirstName" : "James", "LastName" : "Cyborg", "Class" : "JSS 1", "Age" : 39, "Subjects" : ["English", "Mathematics", "Physics", "Chemistry"] }

```
# Finding Specific documents

Most of the time we don't want to retrieve all documents, but rather specify a filter that returns documents matching that particular filter. Let's now look at ways of specifying filter to the query.

# Using BsonDocument or string

We can define a BsonDocument as a filter and the query will find documents matching the fields defined in the document. Add the following code to your method and run it to retrieve student whose name is "Peter"

```
var filter = new BsonDocument("FirstName", "Peter");

await collection.Find(filter)
         .ForEachAsync(document => Console.WriteLine(document));

```
And we get just one document

```
{ "_id" : ObjectId("58469c732adc9f5370e50c9f"), "FirstName" : "Peter", "LastName" : "Cyborg", "Class" : "JSS 1", "Age" : 39, "Subjects" : ["English", "Mathematics", "Physics", "Chemistry"] }

```
You could get a bit confused because these methods accept a `FilterDefinition` but we're giving it a BsonDocument and it doesn't complain. This happens because it gets implicitly converted and we can also do this from a string. To use a string, we need to define a valid JSON string specifying the filter. We can do the same thing above with a string using the code below and we would still get the same result:

```
var filter = "{ FirstName: 'Peter'}";

await collection.Find(filter)
    .ForEachAsync(document => Console.WriteLine(document));

```
We can also specify comparison or logical operator. An example would be getting students whose ages are 23. We can build the query as follows:

```
var filter = new BsonDocument("Age", new BsonDocument("$eq", 23));

```
or using a string

```
var filter = "{ Age: {'$eq': 23}}";

```

What we did was to add an identifier for an operator, which in our case was `$eq`. Check this **[page](https://www.mongodb.com/docs/v3.2/reference/operator/query/)** for a list of operators and what they do. Let's run our code with one of the code above and see that it will give us students whose ages are 23.

```
{ "_id" : ObjectId("58469c732adc9f5370e50c9c"), "FirstName" : "Gregor", "LastName" : "Felix", "Class" : "JSS 3", "Age" : 23, "Subjects" : ["English", "Mathematics", "Physics", "Biology"] }
{ "_id" : ObjectId("58469c732adc9f5370e50c9d"), "FirstName" : "Machiko", "LastName" : "Elkberg", "Class" : "JSS 3", "Age" : 23, "Subjects" : ["English", "Mathematics", "Spanish"] }

```
# Using the FilterDefinitionBuilder

You can use the `FilterDefinitionBuilder`, which is a builder for `FilterDefinition`. It provides a suite of methods to build up queries, and `Lt`, being one of them, specifies a less than comparison. So we can define a filter using the `FilterDefinitionBuilder` as follows:

```
var filter = new FilterDefinitionBuilder<BsonDocument>().Lt("age", 25);

```
Or use the overloaded method which accepts a LINQ expression:

```
var filter = new FilterDefinitionBuilder<Student>().Lt( student => student.Age, 25);

```
Also, you can use a static `Builders` class to build a filter definition, and this class also has static helper methods for building other things, like projection definition, sort definition, and a few others.

```
var filter = Builders<BsonDocument>.Filter.Lt("age", 25);
var filter = Builders<Student>.Filter.Lt(student => student.Age, 25);

```
The driver also overloaded 3 operators for the filter definition. The `and (&)`, `or (|)` and `not (!)` operators. For example, we want to get students whose ages are less than 25 and the first name is Peter, we can build up such query using the builder helper and the `&` overloaded operator as follows

```
var builder = Builders<BsonDocument>.Filter;
var filter = builder.Lt("Age", 40) & builder.Eq("FirstName", "Peter");

```
thus running and getting a single document

```
{ "_id" : ObjectId("58469c732adc9f5370e50c9f"), "FirstName" : "Peter", "LastName" : "Cyborg", "Class" : "JSS 1", "Age" : 39, "Subjects" : ["English", "Mathematics", "Physics", "Chemistry"] }

```
# LINQ Expression

The last part we haven't looked at is the overload of these methods that takes a LINQ expression and when we have a strongly typed object, we can build a filter query using LINQ expression. So let's say we want to get students whose ages are less than 25 and the first name is not Peter and print out their first and last names. We do that with the following code:

```
var collection = db.GetCollection<Student>("students");
await collection.Find(student => student.Age < 25 && student.FirstName != "Peter")
    .ForEachAsync(student => Console.WriteLine(student.FirstName + " " + student.LastName));

```

We change the collection type to `Student` and running the following code, we get an error on the console:

![](/img/mongo-db-1.jpg)

The error description says ***_Element 'id'*** does not match any field or property of type **Student** and this happens because it couldn't map the `_id` field from the database to any property of the Student type. This was automatically added when we created documented. Let's update the `Student` type by adding a property of type `ObjectId` defined in the MongoDB.Bson package.

```
class Student
{
    public ObjectId Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string Class { get; set; }
    public int Age { get; set; }
    public IEnumerable<string> Subjects { get; set; }
}

```
Modify the code to also print out the ID and run it:

```
await collection.Find(student => student.Age < 25 && student.FirstName != "Peter")
      .ForEachAsync(student => Console.WriteLine($"Id: {student.Id}, FirstName: {student.FirstName}, LastName: {student.LastName}"));

```
And it just works. So most often, you would want to use **[expression tree syntax](https://learn.microsoft.com/en-in/archive/blogs/charlie/expression-tree-basics)** to build your queries. And on occasions you want more granularity, you can use the other ways of doing it.

In the next tutorial, we'll see how to do projections, sort, skip, limit and sort.

# Links:

- [Part 1: Driver Basics & Inserting Documents](https://www.codementor.io/@pmbanugo/working-with-mongodb-in-net-1-basics-g4frivcvz)

- [Part 3: Skip, Sort, Limit & Projections](https://www.codementor.io/@pmbanugo/working-with-mongodb-in-net-part-3-skip-sort-limit-and-projections-oqfwncyka) -->

