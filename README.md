# nosql-challenge: Evaluation of food quality ratings data for food critics

![mongoDB](https://github.com/dspataru/nosql-challenge/assets/61765352/8e604b7b-0a15-4277-a902-064a6e19ba14)


## Table of Contents
* [Background](https://github.com/dspataru/nosql-challenge/tree/main#background)
* [Database and Jupyter Notebook Set Up](https://github.com/dspataru/nosql-challenge/tree/main#database-and-jupyter-notebook-set-up)
* [Update the Database](https://github.com/dspataru/nosql-challenge/tree/main#update-the-database)
* [Exploratory Analysis](https://github.com/dspataru/nosql-challenge/tree/main#exploratory-analysis)

## Background

MongoDB is a popular noSQL database that uses a document-oriented model as opposed to a table-based relational model such as SQL. The data in MongoDB is stored in a binary format which allows it to be parsed much more quickly. In MongoDB, the code defines the schema which allows for a flexible schema where the syntax is easier to read and understand. On the flip side, MongoDB can have lots of duplicate data, the relationships are not well-defined, and it is difficult to create joins.

For this project, we import a JSON file of food establishment ratings data to a MongoDB database and conduct analysis to help a food magazine company, Eat, Safe, Love, make decisions about their future articles. The UK Food Standards Agency evaluates various establishments across the United Kingdom, and gives them a food hygiene rating. The PyMongo library was used to interface with the Mongo databse using Python, and aggregation pipelines were used to retrieve data.

![food](https://github.com/dspataru/nosql-challenge/assets/61765352/af9141f1-ccfe-48a6-99ff-45cf0f424515)

This project is broken up into two Jupyter Notebook:
1. [NoSQL_setup_starter](https://github.com/dspataru/nosql-challenge/blob/main/NoSQL_setup_starter.ipynb): This notebook covers the first two sections of this repository, the database and jupyter notebook set up, as well as the steps that were taken to update the database before performing an exploratory analysis on the establishments.
2. [NoSQL_analysis_starter](https://github.com/dspataru/nosql-challenge/blob/main/NoSQL_analysis_starter.ipynb): This notebook contains the code to answer the following questions:
* Which establishments have a hygiene score equal to 20?
* Which establishments in London have a RatingValue greater than or equal to 4?
* What are the top 5 establishments with a RatingValue of 5, sorted by lowest hygiene score, nearest to the new restaurant added, "Penang Flavours"?
* How many establishments in each Local Authority area have a hygiene score of 0? Sort the results from highest to lowest, and print out the top ten local authority areas.

#### Key Words
noSQL, Python, mongodb, pymongo, aggregation pipeline, exploratory analysis, Mongo Client, pretty print, json, Jupyter Notebook, aggregation, documents, database, collections

## Database and Jupyter Notebook Set Up

The first step to be able to analyze the data is import the data that is provided as a [json file](https://github.com/dspataru/nosql-challenge/blob/main/Resources/establishments.json) from the Terminal using the following command `mongoimport --type json -d uk_food -c establishments --drop --jsonArray establishments.json`. We call the database 'uk_food' and the collection 'establishments'. MongoClient was imported from the pymongo library so that an instance of MongoClient could be created and the database could be accessed. The Pretty Print library was also imported to display the data throughout the set up and analysis process. After the command to load the database is run, you are able to see the database was imported properly by running the command `mongo.list_database_names()`. Below is an example of what one of the documents in the establishments collection looks like:

```python
{'AddressLine1': 'The Pines Garden',
 'AddressLine2': 'Beach Road',
 'AddressLine3': 'St Margarets Bay',
 'AddressLine4': 'Kent',
 'BusinessName': 'The Tea Room',
 'BusinessType': 'Restaurant/Cafe/Canteen',
 'BusinessTypeID': 1,
 'ChangesByServerID': 0,
 'Distance': 4587.362402580997,
 'FHRSID': 551803,
 'LocalAuthorityBusinessID': 'PI/000070948',
 'LocalAuthorityCode': '182',
 'LocalAuthorityEmailAddress': 'publicprotection@dover.gov.uk',
 'LocalAuthorityName': 'Dover',
 'LocalAuthorityWebSite': 'http://www.dover.gov.uk/',
 'NewRatingPending': False,
 'Phone': '',
 'PostCode': 'CT15 6DZ',
 'RatingDate': '2021-08-17T00:00:00',
 'RatingKey': 'fhrs_5_en-gb',
 'RatingValue': '5',
 'RightToReply': '',
 'SchemeType': 'FHRS',
 '_id': ObjectId('64f900a300a8382bc2f49c53'),
 'geocode': {'latitude': '51.148133', 'longitude': '1.383298'},
 'links': [{'href': 'https://api.ratings.food.gov.uk/establishments/551803',
            'rel': 'self'}],
 'meta': {'dataSource': None,
          'extractDate': '0001-01-01T00:00:00',
          'itemCount': 0,
          'pageNumber': 0,
          'pageSize': 0,
          'returncode': None,
          'totalCount': 0,
          'totalPages': 0},
 'scores': {'ConfidenceInManagement': 0, 'Hygiene': 0, 'Structural': 0}}
```

## Update the Database

The second part of this project involves updating the database to incude a new restaurant that the magazine editors wanted to include before performing an analysis on the establishments. The new restaurant is a halal restaurant in Greenwich that has not been rated yet. Below is the dictionary that was created for the new restaurant data:

```python
new_restaurant = {
    "BusinessName":"Penang Flavours",
    "BusinessType":"Restaurant/Cafe/Canteen",
    "BusinessTypeID":"",
    "AddressLine1":"Penang Flavours",
    "AddressLine2":"146A Plumstead Rd",
    "AddressLine3":"London",
    "AddressLine4":"",
    "PostCode":"SE18 7DY",
    "Phone":"",
    "LocalAuthorityCode":"511",
    "LocalAuthorityName":"Greenwich",
    "LocalAuthorityWebSite":"http://www.royalgreenwich.gov.uk",
    "LocalAuthorityEmailAddress":"health@royalgreenwich.gov.uk",
    "scores":{
        "Hygiene":"",
        "Structural":"",
        "ConfidenceInManagement":""
    },
    "SchemeType":"FHRS",
    "geocode":{
        "longitude":"0.08384000",
        "latitude":"51.49014200"
    },
    "RightToReply":"",
    "Distance":4623.9723280747176,
    "NewRatingPending":True
}
```
The new restaurant data was inserted into the establishments collection by using the `insert_one()` method. A value for the BusinessTypeID field is missing from new restaurant data. Since it is classified as a Restaurant/Cafe/Canteen, we can update the business type by finding the list of establishments with BusinessType = Restaurant/Cafe/Canteen, and returning the BusinessTypeID and BusinessType fields. We find that the BusinessTypeID for the Restaurant/Cafe/Canteen establishments is 1. To update the new restaurant data, the following command is used: `establishments.update_one(new_restaurant, [{'$set' : {'BusinessTypeID' : 1}}])`.

Since the magazine is not interested in any in any establishments in Dover, we checked how many documents contain the Dover Local Authority using the `count_documents()` method to find that the number of documents that have LocalAuthorityName as Dover is 994. We then remove any establishments within the Dover Local Authority from the database by using the `delete_many()` method, and check the number of documents to ensure they were deleted.

The last step we take in transforming the data is by updating some of the number values that are stored as strings when they should be stored as numbers. 
1. Converting the latitude and longitude to decimal numbers:
```python
lat_long_update_query = {
    '$set': {
        'geocode.latitude': {
            '$toDouble': '$geocode.latitude'
        },
        'geocode.longitude': {
            '$toDouble': '$geocode.longitude'
        }
    }
}

lat_long_update_result = establishments.update_many({}, [lat_long_update_query])
```
3. Converting the RatingValue to integer numbers:
```python
RatingValue_update_query = {
    '$set': {
        'RatingValue': {
            '$toInt': '$RatingValue'
        }
    }
}

RatingValue_update_result = establishments.update_many({}, [RatingValue_update_query])
```
We can see that the coordinates and rating value are now numbers.
```python
{'_id': ObjectId('64f900a300a8382bc2f49f3a'),
 'FHRSID': 647177,
 'ChangesByServerID': 0,
 'LocalAuthorityBusinessID': 'PI/000041489',
 'BusinessName': 'Wear Bay Bowls Club',
 'BusinessType': 'Pub/bar/nightclub',
 'BusinessTypeID': 7843,
 'AddressLine1': 'Wear Bay Road',
 'AddressLine2': 'Folkestone',
 'AddressLine3': 'Kent',
 'AddressLine4': '',
 'PostCode': 'CT19 6PY',
 'Phone': '',
 'RatingValue': 4,
 'RatingKey': 'fhrs_4_en-gb',
 'RatingDate': '2014-03-31T00:00:00',
 'LocalAuthorityCode': '188',
 'LocalAuthorityName': 'Folkestone and Hythe',
 'LocalAuthorityWebSite': 'http://www.folkestone-hythe.gov.uk',
 'LocalAuthorityEmailAddress': 'foodteam@folkestone-hythe.gov.uk',
 'scores': {'Hygiene': 5, 'Structural': 5, 'ConfidenceInManagement': 10},
 'SchemeType': 'FHRS',
 'geocode': {'longitude': 1.196408, 'latitude': 51.086058},
 'RightToReply': '',
 'Distance': 4591.821311183521,
 'NewRatingPending': False,
 'meta': {'dataSource': None,
  'extractDate': '0001-01-01T00:00:00',
  'itemCount': 0,
  'returncode': None,
  'totalCount': 0,
  'totalPages': 0,
  'pageSize': 0,
  'pageNumber': 0},
 'links': [{'rel': 'self',
   'href': 'https://api.ratings.food.gov.uk/establishments/647177'}]}
```

## Exploratory Analysis

The food magazine company has specific questions they are trying to answer which will help them find locations they would like to visit and places they would want to avoid. There are some considerations to be made while exploring the dataset:
* RatingValue refers to the overall rating decided by the Food Authority and ranges from 1-5. The higher the value, the better the rating.
* The scores for Hygiene, Structural, and ConfidenceInManagement work in reverse. This means, the higher the value, the worse the establishment is in these areas.

The following questions were answered to help the food magazine company make their decisions:
1. Which establishments have a hygiene score equal to 20?
2. Which establishments in London have a RatingValue greater than or equal to 4?
3. What are the top 5 establishments with a RatingValue of 5, sorted by lowest hygiene score, nearest to the new restaurant added, "Penang Flavours"?
4. How many establishments in each Local Authority area have a hygiene score of 0? Sort the results from highest to lowest, and print out the top ten local authority areas.

For each question, count_documents is used to display the number of documents contained in the result, yhe first document in the results is displayed using pprint, and the results are then converted into a Pandas DataFrame.

For the first question, the following query was used:
```python
hygiene_query = {
    'scores.Hygiene': {
        '$eq': 20
    }
}
```
There are 41 documents in the establishments collection with a hygiene score of 20.

For the second questions, the following query was used:
```python
RatingValue_query = {
    'LocalAuthorityName': {
        '$regex': 'London'
    },
    'RatingValue': {
        '$gte': 4
    }
}
```
There are 33 documents with London as the Local Authority and RatingValue greater than or equal to 4.

For the third question, we searched within 0.01 degree on either side of the latitude and longitude of the newly added restaurant "Penang Flavours". The rating value must be equal to 5, the highest rating you can get, and the data was sorted in descending order by hygiene score using the following code:
```python
degree_search = 0.01
latitude = establishments.find_one({'BusinessName': 'Penang Flavours'})['geocode']['latitude']
longitude = establishments.find_one({'BusinessName': 'Penang Flavours'})['geocode']['longitude']

query = {
    'RatingValue': {
        '$eq': 5
    },
    'geocode.latitude': {
        '$gte': latitude - degree_search, '$lte': latitude + degree_search 
    },
    'geocode.longitude': {
        '$gte': longitude - degree_search, '$lte': longitude + degree_search
    }
}

sort = [('scores.Hygiene', -1)]

limit = 5
```

To answer the last question, a pipeline was created that (1) Matches establishments with a hygiene score of 0, (2) Groups the matches by Local Authority Name, and (3) Sorts the matches from highest to lowest.
```python
match_query = {
    '$match': {
        'scores.Hygiene': {
            '$eq': 0
        }
    }
}

group_query = {
    '$group': {
        '_id': '$LocalAuthorityName',
        'count': { '$sum': 1 }
    }
}

sort_values = {
    '$sort': {
        'count': -1
    }
}

hygiene0_pipeline = [match_query, group_query, sort_values]
```
The number of documents in the results is 55, and below are the top 10 results.
```python
[{'_id': 'Thanet', 'count': 1130},
 {'_id': 'Greenwich', 'count': 882},
 {'_id': 'Maidstone', 'count': 713},
 {'_id': 'Newham', 'count': 711},
 {'_id': 'Swale', 'count': 686},
 {'_id': 'Chelmsford', 'count': 680},
 {'_id': 'Medway', 'count': 672},
 {'_id': 'Bexley', 'count': 607},
 {'_id': 'Southend-On-Sea', 'count': 586},
 {'_id': 'Tendring', 'count': 542}]
```
These results were converted into a python DataFrame. The first 10 rows of the DataFrame can be seen below:
![Q4](https://github.com/dspataru/nosql-challenge/assets/61765352/7b9988d5-ebd9-43bc-a413-6ec157224e30)
