# Airbnb Dataset - Mongodb

>Course project of **`Big Data Management And Modeling`**  course - [MDSAA-DS](www.novaims.unl.pt/MDSAA-DS) - Spring 2023

## Background

Congratulations on joining the AirBNB Business Intelligence department as part of the new Data Engineering and Management team! The organization is undergoing restructuring to address issues identified by the previous team, including slow query performance, lack of experience in applying patterns, absence of indexes, and data collection mistakes.

As the new team, you are expected to optimize the database schema and data model, leveraging tools such as embedding, linking, indexes, and patterns, to improve query performance and streamline data collection. You should also consider the relevancy of returned data, clean up errors and duplication, and ensure correct and relevant information is provided to all departments.

Good practices in our company include using capitalized field names, ensuring queries function on the most updated version of the database, and documenting all database transformation choices with reasoning and expected results.

## Requirements

Review the data and adjust the schema for the typical use case of showing property listing information to customers, optimizing the structure for this use case by considering what information should be returned and creating new collections/documents as needed.
Clean up data errors and inappropriate duplication.
Answer specific questions from different departments, ranging from standard difficulty questions on rewarding hosts, identifying common bed types, finding the longest review, assessing security deposit differences, and calculating average duration of stay per property type and city, to advanced difficulty questions on optimizing query performance, identifying popular neighborhoods, and finding unique amenities.
Documentation

Provide detailed documentation on all database transformation choices, including the applied transformation, reasoning behind it, and expected results.
Ensure all queries are designed to function on the most updated version of the database.
Follow good practices of using capitalized field names and providing relevant and accurate information to all departments.

## Note

Remember to consider the business use cases, query performance, and relevancy of data returned in each question, and use appropriate tools and techniques to optimize the database schema and data model for improved performance and efficiency.
1. to streamline the data collection system;
2. to clean up the data and optimise what will be returned for each use case;
3. to use correct patterns to improve speed of typical queries;
4. to ensure that all departments receive correct and relevant information from the database;
5. to provide the data model schema to inform other departments of the changes.

Good practices in our company include:
1. all newly created fields have CAPITALIZED names;
2. all new queries function on the most updated version of the database (if you make multiple changes all your queries still work after the final changes);
3. for some of the queries you will likely want to change the database schema in some way. Each database transformation choice is properly documented using format:
    * `TRANSFORMATION APPLIED: <NAME and DESCRIPTION OF TRANSFORMATION>`
    * `REASONING BEHIND: <WHY YOU DID IT>`
    * `EXPECTED RESULT: <WHAT IS THE (EXPECTED) RESULT OF RUNNING QUERIES GIVEN THIS TRANSFORMATION>`
    
## Setup

Once you have Docker up and running please perform the following steps:


**1. Run application**

Please execute the following command to run application containers:
    	
```bash
docker-compose up --detach
```
The Mongodb container will be listening on port `27017` on your machine,

**2. Connect to Mongodb**

Now you can connect to Mongodb using your favorite client, for example [Studio3T](https://studio3t.com/download/) or [MongoDB Compass](https://www.mongodb.com/download-center/compass).

There you can create New Connection OR reconnect to existing Connection:

    2.A	In Studio3T create Connection:
            a.	Press Connection -> New Connection
            b.	Insert credentials into URI field: mongodb://AzureDiamond:hunter2@localhost:27017
            c.	Press Test Connection
            d.	Assign a name to the Connection (top empty line) -> Save
            e.	Press Connect

**3. Import Database**

You can download the database from [here](https://drive.google.com/file/d/12flwsC4W_gHiKSPVTWTdI41NoFVLMBG1/view?usp=share_link).

Back to Studio3T you can import the database:

    2.B	In Studio3T import the Database:
            a.	Press Import
            b.	Choose BSON mongodump archive 
            c.	Find your database path
            d.	Select All file formats and choose the database file ‘sampledata.archive’
            e.	Press Run: be patient while the database loads
            f.	Inspect the collections and documents

## Database Schema Changes:

>1. **TRANSFORMATION APPLIED:** Applying [Schema Versioning](https://www.mongodb.com/docs/v6.0/tutorial/model-data-for-schema-versioning/) Pattern.

- **REASONING BEHIND:** As our schema may require some changes on the structure we decided to implement schema versioning pattern to easily manage and track changes over time without disrupting existing data or applications.

- **METHODOLOGY:** Add a version field to each document in the collection with a default value of 1, indicating the initial schema version, then to be incremented when changes are added to identify it as the latest revision. This will allow the application to handle queries appropriately based on each document version, ensuring smooth transition to new schema changes.

- **EXPECTED RESULT:** A version field in each document in the collection with a default value of 1.

-----

>2. **TRANSFORMATION APPLIED:** Applying [One-to-Many Relationship With Document Reference](https://www.mongodb.com/docs/manual/tutorial/model-referenced-one-to-many-relationships-between-documents/). (`Host-Listings`)

- **REASONING BEHIND:** As we found out that a host can have multiple listings, we decided to implement one-to-many relationship with document reference to store the host's listings in a separate collection to avoid repetition of host data on each listing.

- **METHODOLOGY:** Build a pipeline to extract the host's listings from the listing collection and store them in a new collection with a reference to the host's document in the listing collection then removed the host's related data from the listing collection.

- **EXPECTED RESULT:** A new collection with hosts related info with a reference to the host's document in the listing collection.

----

>3. **TRANSFORMATION APPLIED:** Applying [One-to-Many Relationship With Document Reference](https://www.mongodb.com/docs/manual/tutorial/model-referenced-one-to-many-relationships-between-documents/). (`Listing-Reviews`)

- **REASONING BEHIND:** As we found out that a listing can have multiple reviews, we decided to implement one-to-many relationship with document reference to store the listing's reviews in a separate collection to avoid mutable, growing arrays of reviews.

- **METHODOLOGY:** Build a pipeline to extract the listing's reviews from the reviews embedded documents, find common elements in duplicated reviews arrays and merge them into a single array, store them in a new collection with a reference to the listing's document then removed reviews arrays, as well  have embedded reviews scores to apply computed pattern later.

- **EXPECTED RESULT:** A new collection with reviews related info with a reference to the listing's document. A new Embedded document with computed reviews scores.

----

>4. **TRANSFORMATION APPLIED:** Applying [One-to-Many Relationship With Document Reference](https://www.mongodb.com/docs/manual/tutorial/model-referenced-one-to-many-relationships-between-documents/). (`Listing-Transactions`)

- **REASONING BEHIND:** As we found out that a listing can have multiple transactions, we decided to implement one-to-many relationship with document reference to store the listing's transactions in a separate collection to avoid mutable, growing arrays of transactions.

- **METHODOLOGY:** Build a pipeline to extract the listing's transactions from the transactions embedded documents, store them in a new collection with a reference to the listing's document then removed transactions array.

- **EXPECTED RESULT:** A new collection with transactions related info with a reference to the listing's document.

----

>5. **TRANSFORMATION APPLIED:** Applying [Model One-to-Many Relationships with Document References](https://www.mongodb.com/docs/v6.0/tutorial/model-referenced-one-to-many-relationships-between-documents/). (`Listings-Amenities`)

- **REASONING BEHIND:** As we found out that a listing can have multiple amenities, we decided to implement many-to-many relationship with document reference to store the listing's amenities in a separate collection and saving the amenities' indexed names in the listing's document.

- **METHODOLOGY:** Build a pipeline to store all unique amenities in a new collection and keeping the amenities' indexed names in the listing's document.

- **EXPECTED RESULT:** A new collection with amenities related info with indexed names in the listing's document.

----

## Indexing

In order to improve the performance of the queries we decided to create the following indexes:

1. **Indexing on `schema_version` field in `listingsAndReviews_new` collection.**
2. **Indexing on `schema_version`, `host_id`, `host_listings_count`, `address.market`, `amenities`, `price`, `security_deposit`, `accommodates`, `property_type`, `REVIEWS_SCORES`, `maximum_nights`, `minimum_nights`, (`minimum_nights`, `maximum_nights`), (`price`, `security_deposit`), (`accommodates`, `address.market`, `price`, `security_deposit`)
 field in `listings` collection.**
3. **Indexing on `listing_id`, `reviewer_id`, `reviewer_name`, (`reviewer_id`, `comments`) fields in `reviews` collection.**
4. **Indexing on `schema_version`, `listing_id`, `host_is_superhost`, `host_listings_count` fields in `hosts` collection.**
5. **Indexing on `schema_version`, `transactions`, (`transactions.date`, `transactions.price`) fields in `transactions` collection.**
6. **Indexing on `schema_version`, `name` fields in `amenities` collection.**