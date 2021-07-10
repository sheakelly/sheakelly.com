---
slug: /blog/storing-objects-with-prim
title: "Storing Objects with Prim"
description: A simple library that allows storage of .net object in a sql database
banner: Prim_Hemline.png
date: 2015-11-10
---

With the invention of object oriented technology and the use of relational databases developers have had to deal the impedance mismatch between object oriented class structures and relational database models. The result was the creation of the Object Relational Mapper (ORM) to abstract away this complexity. I have used a few ORMs over the years: hibernate, it's .Net equivalent nhibernate and Entity Framework to name a few. They a fairly simply to setup and use but come with a significant learning curve and are often associated with poor performance due to the generation of inefficient sql queries.

<!-- More -->
# Lightweight or Micro ORMs
Over the last couple of years there has been an explosion of Micro ORMs e.g. Dapper, Massive, OrmLite, Simple.Data etc. These libraries aim to reduce the complexity and learning curve of  full ORMs and give the developer more control over the queries sent to the server. This can lead to improved performance for SQL experts. My favourite of these at the moment is Dapper and the Dapper.Contrib package to make it easier to work with CRUD scenarios.

# And Then There Are Aggregates
The Aggregate pattern comes from Domain Driven Design and refers to a group of related objects that have a consistency boundary around them. All changes to the aggregate are made through the Aggregate root. Consider aggregate that consists of an `Album` that has a number of associated `Track` objects.

This is where libraries like Dapper fall down. *You need to manage the relationship between Album and its Tracks yourself*. It is definitely possible however requires a bit more work than if a fully fledged ORM was managing the relationship and generating the SQL. Consider the following code that adds a new Album entity and a Track entity that is related to it.

```csharp
using (var connection = OpenConnection())                      
{
  const string insertAblumSql = "INSERT INTO Album (Id, ArtistName, RecordLabel, ReleaseDate) " +
                            "VALUES (@ArtistName, @ReleaseDate, @RecordLabel) " +
                            "SELECT SCOPE_IDENTITY()";
  var albumId = connection.Query<int>(insertAblumSql, new {album.ArtistName, album.RecordLabel, album.ReleaseDate }).First();
  const string insertTrackSql = "INSERT INTO Tracks (AlbumId, Name, Duration) " +
                       "VALUES (@AlbumId, @Name, @Duration)";
  var result = connection.Execute(insertTrackSql, new { AlbumId = albumId, track.Name, track.Duration });        
}
```

This example is fairly straight forward but there is a fair amount of code to manage the relationship between an album and it tracks. Also this type of code soon explodes as more aggregates are added to your system. Your repository classes or data access layer grow quickly and can become a maintenance burden.

# Prim Library
[Prim](https://github.com/sheakelly/prim) is and open source .Net library that attempts to make it easy to store an object graph in a relational database. It is inspired by Dapper in that it is implemented as a set of extension methods to the IDbConnection interface that is part of the System.Data namespace. It allows you to save and retrieve an object given a connection to the database and manages the serialisation of the object using the popular Json.Net library. This is based on an approach I have used on a number of projects over the years.

# CRUD Operations
The follows examples show how to use the extension methods Prim provides

## Insert a New Document
We assume that a class called `Album` exist with a string property called `Id` to be used as the value of the primary key

```csharp
var album = ...
using(var connection = dbProviderFactory.CreateConnection())
{
  connection.InsertDocument(album); //Will fail if document already exists
}
```

## Retrieve and Update a Document
A Document can be retrieved by it's unique identifier and updated

```csharp
using(var connection = dbProviderFactory.CreateConnection())
{
  var album = connection.GetDocumentById<Album>(albumId);
  album.Artist.Name = "Neil Young";
  connection.UpdateDocument(album);
}
```

## Delete a Document
Delete a document by passing it to the `DeleteDocument` method

```csharp
var album = ...
using(var connection = dbConnectionFactory.CreateConnection())
{
  connection.DeleteDocument(album);
}
```

## Insert or Update (Upsert) a Document
Some databases support an `upsert` statement. This is emulated by providing an `UpsertDocument` method that internally calls `InsertDocument` or `UpdateDocument` based on the existence of the album.

```csharp
var album = ...
using(var connection = dbConnectionFactory.CreateConnection())
{
  connection.UpsertDocument(album); //This is insert or update the document based on if it exist or not
}
```

## Promoted Properties
Suppose we add a `Producer` property to our `Album` class and a Producer has a name. It will not be easy to query for Albums by the producer's name as it is nested within the json structure. We could issue a query within the json like this:

```sql
select * from Album where data like '%Name: Fred Peters%'
```
This could get inefficient fairly quickly as the size of the json increases. Also the `Name` property could match other properties within the documents e.g. the Album. To work around this and support querying Prim allows you to promote properties within the document. Promoted properties are stored an a separate column in the database:

```csharp
Configure.Promote<Album>(a => a.Producer.Name, "ProducerName");
```

The above example would result in a `ProducerName` column being pre populated with the name of the producer for all `InsertDocumet` and `UpdateDocument` calls. Now we query by producer name using standard SQL.

# Limitations
The objects are serialised into an `ntext` column using Newtonsoft.Json package. This means that you cannot query into the the document easily without some upfront thought i.e. using the promoted properties feature. In practice however the primary key is often enough to load the object. I have use a Guid value rather than an auto incrementing key in the easiest way to generate a unique primary key for a document.

# Other Options
There are a number of alternative solutions to this problem. The most obvious is to use a *real* document database such a [mongodb](http://www.mongodb.org) or [RavenDb](http://ravendb.net). However these are not simple products. They have their own learning curves and require the deployment of a new type of database server for IT Operations to manage. Also in some organisations SQL Server is ingrained and it can be very difficult to get another database product installed. Some relational databases like [PostgreSQL](http://www.postgresql.org) support json column types. This allows you to query and index within the structure of the document using extensions to SQL. If you like this approach and are stuck with SQL Server a colleague of mine has created a product called [Json Select](https://www.jsonselect.com/) that adds a json datatype into SQL Server. Pretty clever.

# Final Thoughts
If you prefer the simplicity of working with objects and want to store them with minimal fuss in a relational database using .Net you can simply install the Prim package from nuget using `Install-Package Prim` and kiss your relationship woes goodbye.
