# NoSQL MongoDB database

Patrick Ausderau / Ilkka Kylmäniemi

## Study before the classes

- <https://youtu.be/VOLeKvNz-Zo>
- <https://youtu.be/ofme2o29ngU>

---

## NoSQL

- NoSQL = "non SQL", "non relational" or "not only SQL"
- Contains about anything that is not only relational databases
  - Document databases like MongoDB
  - Key-value storages like Redis and Memcached
  - Graph databases like OrientDB and Neo4j
  - Object databases like ObjectDB
  - Tabular databases like HBase and Bigtable

---

## MongoDB

- Document-oriented database (server code available at <https://github.com/mongodb/mongo>)
  - First released in February 2009
  - Current version v7.x

---

## Main features

- Ad hoc queries
  - MongoDB supports rich query operators (field equality, ranges, regex) and fine-grained projections
  - Note: Avoid server-side JavaScript ($where) in queries for performance and security; prefer operators and the aggregation pipeline
- Indexing

  - Indexing means that you can define indexes on any field in the document to improve query performance. For example, if you have a collection of users and you often query by email, you can create an index on the email field:

    ```javascript
    db.collection.createIndex({ email: 1 });
    ```

  - The number 1 means that the index is ascending. You can also create a descending index with -1. After creating the index, you can query the collection like this:

    ```javascript
    db.collection.find({ email: 'frank@metropolia.fi' });
    ```

  - Consider unique, compound, text, and TTL indexes as needed. To prevent duplicates, create a unique index:

    ```javascript
    db.collection.createIndex({ email: 1 }, { unique: true });
    ```

- High availability (replication)
  - Replication means that the data is copied to multiple servers. If one server goes down, the data is still available on the other servers. MongoDB can have multiple copies of data, which are called replicas. If the primary server goes down, one of the replicas is automatically promoted to the primary server. This is called failover.
- Supports sharding
  - The difference to replication is that sharding means that the data is split into multiple servers. This is called horizontal scaling. For example, if you have a collection of users and you have a lot of users, you can split the collection into multiple servers. This way you can handle more users. MongoDB automatically balances the data between the servers.
- File storage
  - For storing files larger than 16 MB, MongoDB provides GridFS. In most applications, prefer external object storage (e.g., S3) for static assets and keep references in MongoDB.
- Aggregation framework
  - Powerful, composable data processing via the aggregation pipeline. Preferred over legacy Map-Reduce (Map-Reduce is deprecated in modern MongoDB).

---

## Basic structure

- Server has multiple databases
- Database has multiple collections
  - Like tables in relational database
  - Usually contains many related documents
  - Do not enforce schema on documents
  - Document validation available via JSON Schema (validated on inserts and updates)
- Collection has multiple documents (rows)
  - JSON-like structure for collection of key-value pairs
  - Serialized and stored as BSON (Binary JSON)
- Document has multiple fields (columns)
  - Can have different fields from each other within the collection
  - Each document has an `_id` that must be unique within its collection. By default, MongoDB generates an ObjectId (12 bytes, shown as a 24-hex-character string). You can also set `_id` yourself (any type), but it must remain unique.

---

## Modeling data

- Modeling the data is different from relational databases
  - Storage is cheap compared to computing time
  - Duplicating data is acceptable
- Model data by user requirements
- Joins are possible on write instead of read. This is called embedding
  - Easier to read but harder to update
  - Use references for large data sets. This is similar to foreign keys in relational databases
- Optimize for most usual use case
- Example: Blog post documents store tags (strings) and comments (objects with text, author etc.) within the blog document itself (with author, text, date, etc.)

---

## MariaDB vs MongoDB

| **Criteria**      | **MariaDB (Relational Database)**                       | **MongoDB (NoSQL Database)**                             |
| ----------------- | ------------------------------------------------------- | -------------------------------------------------------- |
| **Data Type**     | Organized in tables (like spreadsheets).                | Organized in flexible documents (like folders of files). |
| **Best For**      | Structured, consistent data (e.g., customer databases). | Flexible, changing data (e.g., social media posts).      |
| **Schema**        | Fixed structure; data must fit into predefined tables.  | Flexible structure; data can vary from entry to entry.   |
| **Scalability**   | Grows by adding power to a single server.               | Grows by adding more servers.                            |
| **Speed**         | Fast for complex queries and transactions.              | Fast for large volumes of simple data.                   |
| **Relationships** | Great for connected data (e.g., sales and customers).   | Better for isolated or loosely connected data.           |
| **Examples**      | Banking systems, inventory management.                  | Real-time analytics, content management systems.         |
| **Development**   | Requires careful planning; changes can be difficult.    | Easy to adapt; changes are simple to make.               |

- **Use MariaDB** when your data is structured, consistent, and requires complex operations (like finance or inventory systems).
- **Use MongoDB** when your data is flexible, unstructured, and grows rapidly (like social media or big data applications).

---

### Assignment 1

- Create a free MongoDB Atlas cluster (video walkthrough: <https://youtu.be/YfyKoMNavs4>)
- Install
- Install MongoDB Database Tools (mongodump, mongorestore, mongoexport, mongoimport): <https://www.mongodb.com/docs/database-tools/>

  - add the path to your system PATH so you can run the tools from any terminal:

    - windows: search "environment variables" in start menu, edit system environment variables, Environment Variables -> select Path under System variables -> Edit -> New -> add the path to the bin folder of the tools

    - macOS/Linux: add `export PATH=<path-to-tools-bin>:$PATH` to your shell profile file (`~/.zshrc`):

    ```bash
    echo 'export PATH=<path-to-tools-bin>:$PATH' >> ~/.zshrc
    source ~/.zshrc
    ```

    - on older macOS versions, the profile file may be `~/.bash_profile` or `~/.bashrc` instead of `~/.zshrc`

- Restore from [this mongo dump](./mongodump.zip)
- Download the dump file and extract it
- with mongorestore:

  ```bash
  mongorestore --uri "mongodb+srv://<USERNAME>:<PASSWORD>@<HOST>" /path/to/dump
  ```

- Browse the database in MongoDB Atlas
- Database has two collections: `user` and `cat`
  - `user` contains user information
  - `cat` contains cat information, including name, owner, birthdate, weight and a GeoJSON location
- One cat has a placeholder image; replace it with a real image URL from Wikipedia: `https://upload.wikimedia.org/wikipedia/commons/thumb/1/15/Cat_August_2010-4.jpg/1920px-Cat_August_2010-4.jpg`

---

## MongoDB CRUD operations

- Create: <https://www.mongodb.com/docs/manual/tutorial/insert-documents/>
- Read: <https://www.mongodb.com/docs/manual/tutorial/query-documents/>
- Update: <https://www.mongodb.com/docs/manual/tutorial/update-documents/>
- Delete: <https://www.mongodb.com/docs/manual/tutorial/remove-documents/>

---

### Assignment 2

- Install MongoDB Shell: <https://www.mongodb.com/docs/mongodb-shell/> and add it to your system PATH like with the Database Tools above
- Connect to your Atlas cluster:

  ```bash
  mongosh "mongodb+srv://<USERNAME>:<PASSWORD>@<HOST>/animalsdb?retryWrites=true&w=majority"
  ```

- [Basic commands](https://www.mongodb.com/docs/mongodb-shell/run-commands/)
- Select database: `use cat-labs`, show all databases: `show dbs`
- [Perform CRUD operations](https://www.mongodb.com/docs/mongodb-shell/crud/)

1. Select all documents from the `cat` collection
2. Select all documents from the `user` collection

- [Query and Projection Operators](https://www.mongodb.com/docs/manual/reference/mql/query-predicates/#std-label-query-projection-operators-top)
- [Date](https://www.mongodb.com/docs/manual/reference/method/Date/#insert-and-return-isodate-objects)

1. Select all cats born before 2020
2. Select all cats born in 2023

- [Geospatial queries](https://www.mongodb.com/docs/manual/geospatial-queries/)

Prerequisite: ensure a 2dsphere index on the GeoJSON field (here we use `cat.location`). Run once:

```javascript
db.cat.createIndex({ location: '2dsphere' });
```

1. Find all cats within the following bounding region (approx. Africa). Use a Polygon with these corner coordinates (Longitude, Latitude):

   ```javascript
   db.cat.find({
     location: {
       $geoWithin: {
         $geometry: {
           type: 'Polygon',
           coordinates: [
             [
               [-20.134438978133744, 33.00029113824003],
               [-20.134438978133744, -33.50621356197972],
               [61.03523868865631, -33.50621356197972],
               [61.03523868865631, 33.00029113824003],
               [-20.134438978133744, 33.00029113824003],
             ],
           ],
         },
       },
     },
   });
   ```

2. Find all cats within 100 km of Helsinki:

   ```javascript
   db.cat.find({
     location: {
       $near: {
         $geometry: { type: 'Point', coordinates: [24.9384, 60.1695] },
         $maxDistance: 100000,
       },
     },
   });
   ```

3. Find all cats in France. Use [geojson.io](https://geojson.io/) to draw a polygon around France and use those coordinates in a `$geoWithin: { $geometry: { type: "Polygon", coordinates: [...] } }` query. Ensure the polygon is closed (first and last coordinates the same).

---

## MongoDB Aggregation

- [Aggregation](https://www.mongodb.com/docs/manual/aggregation/) operations process data records and return computed results
- Aggregation operations group values from multiple documents together and can perform a variety of operations on the grouped data to return a single result
- Example: Calculate the average age of cats

  ```javascript
  db.cat.aggregate([
    {
      $group: {
        _id: null,
        avgAgeMs: { $avg: { $subtract: [new Date(), '$birthdate'] } },
      },
    },
    {
      $project: {
        avgAgeYears: { $divide: ['$avgAgeMs', 1000 * 60 * 60 * 24 * 365.25] },
      },
    },
  ]);
  ```

- In the example above:
  1. First stage - `$group`
     - `_id: null`: Groups all documents into a single group.
     - `avgAgeMs`: Calculates the average age in milliseconds.
  2. Second stage - `$project`
     - `avgAgeYears`: Converts milliseconds to years by dividing by the number of milliseconds in a year.
- Example, get all cats and their owners. Show only the cat name and owner name:

  ```javascript
  db.cat.aggregate([
    {
      $lookup: {
        from: 'user',
        localField: 'owner',
        foreignField: '_id',
        as: 'owner_info',
      },
    },
    {
      $unwind: '$owner_info',
    },
    {
      $project: {
        _id: 1,
        cat_name: 1,
        owner_name: '$owner_info.name',
      },
    },
  ]);
  ```

---

## Assignment 3

- Select all cats from the `cat` collection and include owner data using the aggregation pipeline (docs: <https://www.mongodb.com/docs/manual/reference/operator/aggregation-pipeline/>):
  (docs: <https://www.mongodb.com/docs/manual/reference/operator/aggregation-pipeline/>)
  - Use `$lookup` to perform left outer join between `cat` and `owner` (as `owner_info`).
  - `$unwind` the `owner_info` array.
  - `$match` documents where `owner_info.name` is "John".
  - `$project` fields you want to show from cats and the joined owners.

---

## Submitting assignments

Submit the queries you used in MongoDB shell to Oma. Don't submit any files, just write the queries in the text field.

---

### Optional: Local installation

- Install Community Edition: <https://www.mongodb.com/docs/manual/installation/>
- Shell access: <https://www.mongodb.com/docs/mongodb-shell/> (optionally a UI like [Studio 3T](https://studio3t.com/download-studio3t-free))
- Enable authentication (local deployments): <https://blog.tericcabrel.com/enable-authentication-and-authorization-on-mongodb/>

---

## Sources

- [NoSQL (Wikipedia)](https://en.wikipedia.org/wiki/NoSQL)
- [MongoDB Docs](https://www.mongodb.com/docs/)

---
