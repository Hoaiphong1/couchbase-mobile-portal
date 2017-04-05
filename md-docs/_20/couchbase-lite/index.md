---
id: api-guide
title: API Guide
permalink: guides/couchbase-lite/index.html
---

## Getting Started

<block class="swift" />

- Download the latest [developer build](../../whatsnew.html).
- Drag **CouchbaseLiteSwift.framework** from your Finder to the Xcode navigator.
- Click on Project > General > Embedded Binary > Add **CouchbaseLiteSwift.framework**.
- Import the framework and start using it in your project.

	```swift
	import CouchbaseLiteSwift
	...
	```

API references for the Swift SDK are available [here](http://docs.couchbase.com/mobile/2.0/couchbase-lite-swift/db003/).

<block class="objc" />

- Download the latest [developer build](../../whatsnew.html).
- Drag **CouchbaseLite.framework** from your Finder to the Xcode navigator.
- Import the framework and start using it in your project.

	```swift
	#include <CouchbaseLite/CouchbaseLite.h>
	...
	```

API references for the Objective-C SDK are available [here](http://docs.couchbase.com/mobile/2.0/couchbase-lite-objc/db003/index.html).

<block class="csharp" />

- Add `http://mobile.nuget.couchbase.com/nuget/Developer/` to your Nuget package sources and expect a new build approximately every 2 weeks!

When a support assembly is required, your app must call the relevant `Activate()` function inside of the class that is included in the assembly (there is only one public class in each support assembly).  For example, UWP looks like `Couchbase.Lite.Support.UWP.Activate()`.  Currently the support assemblies provide dependency injected mechanisms for default directory logic, and platform specific logging (i.e. Android will log to logcat with correct log levels and tags.  No more "mono-stdout" at always info level).

API references for Couchbase Lite .NET are available [here](http://docs.couchbase.com/mobile/2.0/couchbase-lite-net/db003/).

<block class="all" />

## Databases

### Creating Databases

As the top-level entity in the API, new databases can be created using the {% st Database|CBLDatabase|DatabaseFactory|DatabaseFactory %} class by passing in a name, options, or both. The following example creates a database using the {% st Database(name: String)|initWithName:|Create(string name)|Create(string name) %} method.

<block class="swift" />

```swift
do {
  let database = try Database(name: "my-database")
} catch let error as NSError {
  NSLog("Cannot open the database: %@", error);
}
```

<block class="objc" />

```c
NSError *error;
CBLDatabase* database = [[CBLDatabase alloc] initWithName:@"my-database" error:&error];
if (!database) {
	NSLog(@"Cannot open the database: %@", error);
}
```

<block class="csharp" />

```csharp
var database = DatabaseFactory.Create("my-database");
```

<block class="all" />

Just as before, the database will be created in a default location. Alternatively, the {% st Database(name: String options: DatabaseOptions?)|-initWithName:options:error:|Create(string name, DatabaseOptions options)|d %} method can be used to provide specific options (directory to create the database, whether it is read-only etc.)

You can instantiate multiple {% st Databases|CBLDatabases|IDatabases|IDatabases %} with the same name and directory; these will all share the same storage. Do this if you will be calling Couchbase Lite from multiple threads or dispatch queues, since Couchbase Lite objects are not thread-safe and can only be called from one thread/queue. Otherwise, for use on a single thread/queue, it's more efficient to use a single instance.

### Transactions / batch operations

As before, if you're making multiple changes to a database at once, it's *much* faster to group them together. (Otherwise each individual change incurs overhead, from flushing writes to the filesystem to ensure durability.) In 2.0 we've renamed the method, to {% st -inBatch:do:|-inBatch:do:|InBatch()|-inBatch:do: %}, to emphasize that Couchbase Lite does not offer transactional guarantees, and that the purpose of the method is to optimize batch operations rather than to enable ACID transactions.

At the *local* level this operation is still transactional: no other {% st CBLDatabase|CBLDatabase|IDatabase|IDatabase %} instances, including ones managed by the replicator or HTTP listener, can make changes during the execution of the block, and other instances will not see partial changes. But Couchbase Mobile is a *distributed* system, and due to the way replication works, there's no guarantee that Sync Gateway or other devices will receive your changes all at once.

Again, the behavior of the method hasn't changed, just its name.

## Documents

In Couchbase Lite, a document's body takes the form of a JSON object — a collection of key/value pairs where the values can be different types of data such as numbers, strings, arrays or even nested objects. Every document is identified by a document ID, which can be automatically generated (as a UUID) or determined by the application; the only constraints are that it must be unique within the database, and it can't be changed. There are two methods in the API to create a new document:

- The {% st document(withID: String)|documentWithID:|CreateDocument()|document(withID: String) %} method can be used to create a document with a specific ID defined by the application.
- The {% st document()|document:|GetDocument(string id)|-document %} method can be used to let the database generate a random document ID.

[//]: # (TODO: Since this identifier must be unique, you may want to check if a document with this ID already exists in the database using the {% st a|b|c|d %} method.)

The following code example creates a document and persists it to the database.

<block class="swift" />

```swift
let document = database.document()
do {
	try document.save()
} catch let error {
	print(error.localizedDescription)
}
```

<block class="objc" />

```objective-c
CBLDocument* document = [database document];
[document save:&error];
if (error) {
	NSLog(@"Cannot save document %@", error);
}
```

<block class="csharp" />

```csharp
var document = database.CreateDocument();
document.Save();
```

<block class="all" />

### Mutability

The biggest change is that {% st Document|CBLDocument|IDocument|IDocument %} properties are now mutable. Instead of having to make a mutable copy of the properties dictionary, update it, and then save it back to the document, you can now modify individual properties in place and then save.

<block class="swift" />

```swift
doc.properties = [
	"type": "user",
	"admin": false,
	"address": [
		"street": "1 park street",
		"zip": 123456
	]
]
do {
	try document.save()
	print("document type :: \(document["type"] as String?)")
} catch let error {
	print(error.localizedDescription)
}
```

<block class="objc" />

```objective-c
document.properties = @{
	 @"type": @"user",
	 @"admin": @FALSE,
	 @"address": @{
			@"street": @"1 park street",
			@"zip": @123456
	 }
};
[document save:&error];
NSLog(@"document type :: %@", document[@"type"]);
if (error) {
	NSLog(@"Cannot save document %@", error);
}
```

<block class="csharp" />

```csharp
document.Properties = new Dictionary<string, object>
{
	["type"] = "user",
	["admin"] = false,
	["address"] = new Dictionary<string, object>
	{
		["street"] = "1 park street",
		["zip"] = 123456
	}
};
document.Save();
Console.WriteLine($"document type :: ${document.Get("type")}");
```

<block class="all" />

This does create the possibility of confusion, since the document's in-memory state may not match what's in the database. Unsaved changes are not visible to other {% st Database|CBLDatabase|IDatabase|IDatabase %} instances (i.e. other threads that may have other instances), or to queries.

### Typed Accessors

The {% st Document|CBLDocument|IDocument|IDocument %} class now offers a set of property accessors for various scalar types, including boolean, integers, floating-point and strings. These accessors take care of converting to/from JSON encoding, and make sure you get the type you're expecting: for example, {% st let name: String = doc["name"]|-stringForKey:|GetString()|GetString() %} returns either a {% st String|NSString|string|string %} or {% st nil|nil|null|null %}, so you can't get an unexpected object class and crash trying to use it as a string. (Even if the property in the document has an incompatible type, the accessor returns {% st nil|nil|null|null %}.)

<block class="all" />

In addition, as a convenience we offer {% st Date|NSDate|DateTimeOffset|d %} accessors. Dates are a common data type, but JSON doesn't natively support them, so the convention is to store them as strings in ISO-8601 format. The following example sets the date on the `createdAt` property and reads it from the document using the {% st subscript|-dateForKey:|GetDate|GetDate %} accessor method.

<block class="swift" />

```swift
document["createdAt"] = Date()
do {
	try document.save()
} catch let error {
	print(error.localizedDescription)
}
print("createdAt value :: \(document["createdAt"] as Date?)")
```

<block class="objc" />

```objective-c
[document setObject:[NSDate date] forKey:@"createdAt"];
[document save:&error];
if (error) {
	NSLog(@"Cannot save document %@", error);
}
NSLog(@"createdAt value :: %@", [document dateForKey:@"createdAt"]);
```

<block class="csharp" />

```csharp
document.Set("createdAt", DateTimeOffset.UtcNow);
document.Save();
Console.WriteLine($"createdAt value :: ${document.GetDate("createdAt")}");
```

<block class="all" />

### Subdocuments

A subdocument is a nested document with its own set of named properties. In JSON terms it's a nested object. This isn't a new feature of the document model; it's just that we're exposing it in a more structured form. In Couchbase Lite 1.x you would see a nested object as a nested {% st Dictionary|NSDictionary|IDictionary|IDictionary %}. In 2.0 we expose it as a {% st Subdocument|CBLSubdocument|ISubdocument|ISubdocument %} object instead.

<block class="swift" />

```swift
let address: Subdocument? = document["address"]
address?["city"] = "galaxy city"
do {
	try document.save()
} catch let error {
	print(error.localizedDescription)
}
print("address properties :: \((document["address"] as Subdocument?)?.properties)")
```

<block class="objc" />

```objective-c
CBLSubdocument* address = [document subdocumentForKey:@"address"];
[address setObject:@"galaxy city" forKey:@"city"];
[document save:&error];
if (error) {
	NSLog(@"address properties :: %@", [[document subdocumentForKey:@"address"] properties]);
}
```

<block class="csharp" />

```csharp
var address = document.GetSubdocument("address")["city"] = "galaxy city";
document.Save();
Console.WriteLine($"address properties :: ${document.GetSubdocument("address").Properties}");
```

<block class="all" />

{% st Subdocument|CBLSubdocument|ISubdocument|ISubdocument %}, like {% st Document|CBLDocument|IDocument|IDocument %}, inherits from {% st Properties|CBLProperties|IPropertyContainer|IPropertyContainer %}. That means it has the same set of type-specific accessors discussed in the previous section. Like {% st Document|CBLDocument|IDocument|IDocument %}, it's mutable, so you can make changes in-place. The difference is that a subdocument doesn't have its own ID. It's not a first-class entity in the database, it's just a nested object within the document's JSON. It can't be saved individually; changes are persisted when you save its document.

### Blobs

We've renamed "attachments" to "blobs", for clarity. The new behavior should be clearer too: a {% st Blob|CBLBlob|IBlob|IBlob %} is now a normal object that can appear in a document as a property value, either at the top level or in a subdocument. In other words, there's no special API for creating or accessing attachments; you just instantiate an {% st Blob|CBLBlob|IBlob|IBlob %} and set it as the value of a property, and then later you can get the property value, which will be a {% st Blob|CBLBlob|IBlob|IBlob %} object. The following code example adds a blob to the document under the `avatar` property.

<block class="swift" />

```swift
let image = UIImage(named: "avatar.jpg")
let imageData = UIImageJPEGRepresentation(image!, 1)

let blob = Blob(contentType: "image/jpg", data: imageData!)
document["avatar"] = blob
do {
	try document.save()
} catch let error {
	print(error.localizedDescription)
}
print("document properties :: \(document.properties)")
```

<block class="objc" />

```objective-c
UIImage *image = [UIImage imageNamed:@"avatar.jpg"];
NSData *data = UIImageJPEGRepresentation(image, 1);

CBLBlob *blob = [[CBLBlob alloc] initWithContentType:@"image/jpg" data:data];
document[@"avatar"] = blob;
if (error) {
	NSLog(@"Cannot save document %@", error);
}
NSLog(@"document properties :: %@", [document properties]);
```

<block class="csharp" />

```csharp
var data = Encoding.UTF8.GetBytes("12345");
var blob = BlobFactory.Create("image/jpg", data);
document["avatar"] = blob;
document.Save();
Console.WriteLine($"document properties :: ${document["avatar"]}");
```

<block class="all" />

{% st Blob|CBLBlob|IBlob|IBlob %} itself has a simple API that lets you access the contents as in-memory data (an {% st Data|NSData|byte[]|byte[] %} object) or as a {% st InputStream|NSInputStream|Stream|Stream %}. It also supports an optional `type` property that by convention stores the MIME type of the contents. Unlike {% st CBLAttachment|CBLAttachment|Attachment|Attachment %}, blobs don't have names; if you need to associate a name you can put it in another document property, or make the filename be the property name (e.g. {% st doc["thumbnail.jpg"] = imageBlob|[doc setObject: imageBlob forKey: @"thumbnail.jpg"]|doc.Set("thumbnail.jpg", imageBlob)|doc.Set("thumbnail.jpg", imageBlob) %})

> **Note:** A blob is stored in the document's raw JSON as an object with a property `"_cbltype":"blob"`. It also has properties such as `"digest"` (a SHA-1 digest of the data), `"length"` (the length in bytes), and optionally `"type"` (the MIME type.) As always, the data is not stored in the document, but in a separate content-addressable store, indexed by the digest.

### Conflict Handling

We're approaching conflict handling differently, and more directly. Instead of requiring application code to go out of its way to find conflicts and look up the revisions involved, Couchbase Lite will detect the conflict (while saving a document, or during replication) and invoke an app-defined conflict-resolver handler. The conflict resolver is given "my" document properties, "their" document properties, and (if available) the properties of the common ancestor revision.

* When saving a {% st CBLDocument|CBLDocument|IDocument|IDocument %}, "my" properties will be the in-memory properties of the object, and "their" properties will be one ones already saved in the database (by some other application thread, or by the replicator.)
* During replication, "my" properties will be the ones in the local database, and "their" properties will be the ones coming from the server.

The resolver is responsible for returning the resulting properties that should be saved. There are of course a lot of ways to do this. By the time 2.0 is released we want to include some resolver implementations for common algorithms (like the popular "last writer wins" that just returns "my" properties.) The resolver can also give up by returning {% st nil|nil|null|null %}, in which case the save fails with a "conflict" error. This can be appropriate if the merge needs to be done interactively or by user intervention.

A resolver can be specified either at the database or the document level. If a document doesn't have its own, the database's resolver will be used. If the database doesn't have one either (the default situation), a default algorithm is used that picks the revision with the larger number of changes in its history.

## Queries

Database queries have changed significantly. Instead of the map/reduce algorithm used in 1.x, they're now based on expressions, of the form "return ____ from documents where \_\_\_\_, ordered by \_\_\_\_", with semantics based on Couchbase Server's N1QL query language. If you've used {% tx Core Data|Core Data|LINQ|LINQ %}, or other query APIs based on SQL, you'll find this familiar.

### Cross Platform Query API

The Query API provides a simple way to construct a query statement from a set of API methods. There will be two API styles (builder and chainable) implemented based on what makes sense for each platform.

<block class="csharp swift" />

We are still designing the cross-platform query API; it will appear in a future preview release (likely DB004).

<block class="objc" />

In the current Developer Build, a builder API has been implemented. You can call one of the select methods in the `CBLQuery` class to build up your query statement.

For example, the `SELECT * FROM type='user' AND admin='false'` statement can be written with the builder API as follows.

```c
CBLQuery* query = [CBLQuery select:[CBLQuerySelect all]
                              from:[CBLQueryDataSource database:database]
                             where:[
                                    [[CBLQueryExpression property:@"type"] equalTo:@"user"]
                                    and: [[CBLQueryExpression property:@"admin"] equalTo:@FALSE]]];

NSEnumerator* rows = [query run:&error];
for (CBLQueryRow *row in rows) {
	NSLog(@"doc ID :: %@", row.documentID);
}
```

The query can be executed by calling the `-run:` method which will return an `NSEnumerator` instance (enumerator of `CBLQueryRow` objects). As of the current Developer Build, joins are not available yet but will be supported in a future release.

There are several parts to specifying a query:

1. What document criteria to match (corresponding to the “`WHERE …`” clause in N1QL or SQL)
2. What properties (JSON or derived) of the documents to return (“`SELECT …`”)
3. What criteria to group rows together by (“`GROUP BY …`”)
4. Which grouped rows to include (“`HAVING …`”)
3. The sort order (“`ORDER BY …`”)

These all have defaults:

* If you don't specify criteria, all documents are returned
* If you don't specify properties to return, you just get the document ID and sequence number
* If you don't specify grouping, rows are not grouped
* If you don't specify what groups to include, all are included
* If you don't specify a sort order, the order is undefined

#### Parameters

The list of available expressions can be found on the API reference of the [CBLQueryExpression](http://docs.couchbase.com/mobile/2.0/couchbase-lite-objc/db003/Classes/CBLQueryExpression.html) class.

[//]: # (TODO: #### Return Values)

[//]: # (TODO: #### Aggregation and Grouping)

<block class="objc swift" />

### NSPredicate API

The current Developer Build also supports the [NSPredicate](https://developer.apple.com/reference/foundation/nspredicate) query API. The {% st database.createQuery(where: NSPredicate?, groupBy: [Expression]?, having: Predicate?, returning: [Expression]?, distinct: Bool, orderBy: [SortDescriptor]?)|[database createQueryWhere:]|c|d %} method can be used to create an `NSPredicate` query.

Similarly to Core Data, we support the same Core Foundation classes:

1. Document criteria are expressed as an NSPredicate
2. The sort order is an array of NSSortDescriptors
3. The properties to return is an array of NSExpressions.

For convenience, you can provide these as NSStrings: document criteria will be interpreted as NSPredicate format strings, properties to return as NSExpression format strings, and sort orders as key-paths (optionally prefixed with “-” to indicate descending order.)

A CBLPredicateQuery object can be created by calling -createQueryWhere: method on CBLDatabase. After creating a query you can set additional attributes like grouping and ordering before running it.

#### Parameters

A query can have placeholder parameters that are filled in when it's run. This makes the query more flexible, and it improves performance since the query only has to be compiled once (see below.)

Parameters are specified in the usual way when constructing the NSPredicate. In the string-based syntax they're written as “`$`”-prefixed identifiers, like “`$MinPrice`”. (The “`$`” is not considered part of the parameter name.) If constructing the predicate as an object tree, you call `+[NSExpression expressionForVariable:]`.

The compiled CBLPredicateQuery has a property `parameters` , an NSDictionary that maps parameter names (minus the “`$`”!) to values. The values need to be JSON-compatible types. All parameters specified in the query need to be given values via the `parameters` property before running the query, otherwise you'll get an error.

#### Return Values

As in 1.x, running a CBLQuery returns an enumeration of CBLQueryRow objects. Each row's `documentID` property gives the ID of the associated document, and its `document` property loads the document object (at the cost of an extra database lookup.) But a query row can also return values directly, which is often faster than having to load the whole document.

To return values directly from query rows, set the query object's `returning:` property to an array of NSExpressions (or strings that parse to NSExpressions.) It's common to use key-paths, to return document properties directly, but you can add logic or computation.

To access the values returned by a CBLQueryRow, call any of the methods `-objectAtIndex:`, `integerAtIndex:`, etc., where the index corresponds to the index in the query's `returning:` array. Use the most appropriate method for the type of value returned; the numeric/boolean accessors are more efficient, as well as more convenient, because they avoid allocating NSNumber objects. `-stringAtIndex:` will return nil if the value is not a string (avoiding the possibility of an exception), and `-dateAtIndex:` additionally converts an ISO-8601 date string into an NSDate for you.

#### Aggregation and Grouping

If the return values of a query include calls to aggregate functions like `count()`, `min()` or `max()`, all of its rows will be combined together into one, with the aggregate functions operating on their parameters from all the rows.

If you set the query's `groupBy` property, all rows that have the same values of the expressions given in that property will be grouped together. In this case, aggregate functions will operate on the rows in a group, not all the rows of the query.

### Query Performance

Queries have to be parsed and compiled into an optimized form for the underlying database to execute. This doesn't take long, but it's best to create a {% st Query|CBLQuery|c|d %} once and then reuse it, instead of recreating it every time. (Of course, only reuse a CBLQuery on the same thread/queue you created it on!)

Expression-based queries have different performance-vs-flexibility trade offs than map/reduce queries. Map functions can be unintuitive to design, and an individual map function isn't very flexible (all you can control is the range of keys.) But any map/reduce query will be fast because, by definition, it's just a single traversal of an index.

On the other hand, expression-based queries are easier to design and more flexible, but there's no guarantee of performance. In fact, by default *all* queries will be unoptimized, because they have to make a linear scan of the entire database, testing every document against the criteria! In a small database you might not notice, but as the database grows, query time will increase linearly. So how do you make a query faster? By creating any necessary indexes.

### Indexing

A query can only be fast if there's a pre-existing database index it can search to narrow down the set of documents to examine. On the other hand, every index has to be updated whenever a document is updated, so too many indexes can hurt performance. Thus, good performance depends on designing and creating the *right* indexes to go along with your queries.

To create an index, call {% st createIndex(expressions: [Expression])|-createIndexOn:error:|CreateIndex()|d %} passing an array of one or more {% st String|NSString|string|d %}s. These are most often key-paths, but they don't have to be. If there are multiple expressions, the first one will be the primary key, the second the secondary key, etc.

<block class="objc" />

### Full-Text Search

To run a full-text search (FTS) query, you must have created a full-text index on the expression being matched. Unlike regular queries, the index is not optional. The index's (single) expression should be the property name you wish to search on. The index type must also be {% st kCBLFullTextIndex|kCBLFullTextIndex|kCBLFullTextIndex|kCBLFullTextIndex %}. The following code example inserts three documents of type `task` and creates an FTS index on the `name` property.

```c
// Insert documents
NSArray *tasks = @[@"buy groceries", @"play chess", @"book travels", @"buy museum tickets"];
for (NSString* task in tasks) {
	CBLDocument* doc = [database document];
	doc.properties = @{@"type": @"task", @"name": task};
	[doc save:&error];
	if (error) {
		NSLog(@"Cannot save document %@", error);
	}
}

// Create index
[database createIndexOn:@[@"name"] type:kCBLFullTextIndex options:NULL error:&error];
if (error) {
	NSLog(@"Cannot create index %@", error);
}
```

With the index created, an FTS query on the property that is being indexed can be constructed and ran. The full-text search criteria is defined as a {% st CBLQueryExpression|CBLQueryExpression|CBLQueryExpression|CBLQueryExpression %}. The left-hand side is usually a document property, but can be any expression producing a string. The right-hand side is the pattern to match: usually a word or a space-separated list of words, but it can be a more powerful [FTS4 search expression](https://www.sqlite.org/fts3.html#full_text_index_queries). The following code example matches all documents that contain the word 'buy' in the value of the `name` property.

```c
CBLQueryExpression* where = [[CBLQueryExpression property:@"name"] match:@"'buy'"];
CBLQuery *ftsQuery = [CBLQuery select:[CBLQuerySelect all]
                                 from:[CBLQueryDataSource database:database]
                                where:where];

NSEnumerator* ftsQueryResult = [ftsQuery run:&error];
for (CBLFullTextQueryRow *row in ftsQueryResult) {
	NSLog(@"document properties :: %@", row.document.properties);
}
```

When you run a full-text query, the resulting rows are instances of {% st CBLFullTextQueryRow|CBLFullTextQueryRow|CBLFullTextQueryRow|CBLFullTextQueryRow %}. This class has additional methods that let you access the full string that was matched, and the character range(s) in that string where the match(es) occur.

It's very common to sort full-text results in descending order of relevance. This can be a very difficult heuristic to define, but Couchbase Lite comes with a fairly simple ranking function you can use. In the `orderBy:` array, use a string of the form `rank(X)`, where `X` is the property or expression being searched, to represent the ranking of the result. Since higher rankings are better, you'll probably want to reverse the order by prefixing the string with a `-`.

<block class="all" />

### Under The Hood

For the time being, the Objective-C NSPredicate query API also allows you to compose queries using the underlying [JSON-based query syntax](https://github.com/couchbase/couchbase-lite-core/wiki/JSON-Query-Schema) recognized by LiteCore. This can be useful as a workaround if you run into limitations or bugs in the NSPredicate/NSExpression-based API. (But if so, please report the issue to us so we can fix it.)

> Disclaimer: This low-level query syntax is not part of Couchbase Lite's public API. We will probably remove this
 access to it before the final release of Couchbase Lite 2.0. By then the public API should be robust enough to handle your needs.

Using the JSON query syntax is very simple: just construct a JSON object tree out of Foundation objects, in accordance with the [spec](https://github.com/couchbase/couchbase-lite-core/wiki/JSON-Query-Schema), then pass the top level NSArray or NSDictionary to the CBLDatabase method that creates a query or creates/deletes an index:

* `-createQueryWhere:` — The `query` parameter can be a JSON NSArray (interpreted as a WHERE clause), or NSDictionary (interpreted as an entire SELECT query.)
* `-createIndexOn:error:` — Any item of the `expressions` array can be a JSON NSArray (interpreted as an expression to index.)
* `-createIndexOn:type:options:error:` — Same as above.
* `-deleteIndexOn:type:error:` — Same as above.

**Troubleshooting:** If LiteCore doesn't like your JSON, the call will return with an error. More usefully, LiteCore will log an error message to the console, so check that. (For internal reasons these messages don't propagate all the way up to the NSError yet.) If you're still stuck, it may help to set an Xcode breakpoint on all C++ exceptions; this will get hit when the parser gives up, and the stack backtrace _might_ give a clue. A common mistake is to pass an expression where an _array of_ expressions is expected; this is easy to do since expressions themselves are arrays. For example, `returning: @[@".", @"x"]` won't work; instead use `returning: @[@[@".", @"x"]]`.