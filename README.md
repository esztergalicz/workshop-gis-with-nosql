# GIS with NoSQL Workshop Tutorial

## Intorduction

The aim of this hands-on session is to gain first experience with MongoDB, create basic geo-queries and visualize documents from MongoDB on a webmap (in this case OpenLayers). We will use data about pubs in Helsinki, collected from social media (Facebook, Foursquare and Twitter) and OSM.

> If you want to look up how the data was collected you can follow these links:    

> [Facebook](https://github.com/Merithios/DjangoMap/blob/master/Facebook/facebook_skript.py)    
> [Foursquare](https://github.com/Merithios/DjangoMap/tree/master/Foursquare)    
> [Twitter](https://github.com/Merithios/DjangoMap/blob/master/Tweets/tweets.py)    
> [OSM](http://rosslawley.co.uk/the-most-popular-pub-name/)    

So here is your task:
####Find the best pub near the conference location!

## Part I
### 1. Fire up MongoDB
1. Start MongoDB by running `mongod.exe` from `C:\MongoDB\Server\3.2\bin\` (or click on the shortcut on the desktop)
2. To check what‘s in the database we will work with the mongo console    
Start a cmd from `C:\MongoDB\Server\3.2\bin\` (Shift+right-click in the folder --> Open command window here)
& type   
``` mongo ```    
you can ignore the warnings    
3. To list all databases type    
     ``` show dbs ```   
4. To switch to a database type    
     ``` use pubs ```   
5. To show all collections in the database type  
     ``` show collections ```
6. To show a document from the collection (e.g. 'osm') type  
``` db.osm.findOne() ```
7. Let’s take a look at all 4 collections! (Ignore 'allpubs' for now.) Try to find the coordinates of the location in each document.         
     ``` db.facebook.findOne() ```  
     ``` db.foursquare.findOne()   ```   
     ``` db.twitter.findOne()     ```
    
See how they all have a different structure?     
To be able to easily work with them, they were all converted to GeoJSON format and loaded in the ‘allpubs’ collection.      
For example a Facebook document transformed to GeoJSON:
![GeoJSON transformation](https://raw.githubusercontent.com/geszter/workshop-gis-with-nosql/master/images/geojson.png)     

If you are interested how this is done, you can look it up in [/python/](https://github.com/geszter/workshop-gis-with-nosql/tree/master/python), you will need both `create_geojson.py` & `GeoTransformer.py`    

Converting to GeoJSON has another upside: this way a 2dspehere index could be added which enables the geo-queries.    
This is how a geospatial index is added (e.g. for the 'facebook' collection):    
 ``` db.facebook.createIndex( { "geo" : "2dsphere" } ) ```      
 This should be the output:    
 ```JSON
 {
  "createdCollectionAutomatically" : false,
  "numIndexesBefore" : 1,
  "numIndexesAfter" : 2,
  "ok" : 1
}
```

### 2. Django
To be able to visualize the features from MongoDB on a map in the browser, we are using the django framework.    
1.	Start the django project 
go to `C:\django\map` and open a cmd there
type    
```python manage.py runserver ```    
The last line of the output should be    
  ```Quit the server with CTRL-BREAK.```     
  Do not quit the server or close the window.    
2.	Call localhost:8000  in a browser    
Now we have a map with only the conference location.    
3.	Let‘s explore the django logic:    
  1.	Open `C:\django\map`
  2.	The html file is in `map\mapsite\templates\mapsite\header.html`
  3.	The javascript file for the OpenLayers map is in `map\mapsite\static\js`
  4.	Css & images are also in the static folder
  5.	The connection to the database is made in the `map\mapsite\views.py` file     
We will now edit the `views.py` and write some queries to load the features from the database    

### 3. Visualizing MongoDB documents on the map
#####1. Task: Visualize all pubs from the 'allpubs' collection
Open `views.py` (either with PyCharm (this might be slow) or with Notepad++)
If you are using Notepad++ please replace the tabs by 4 spaces:
>Settings --> Preferences --> Tab Settings --> Tab size: 4    
> - [x] **Replace by space**    
> (If you work locally and use Notepad++ often, then please do not forget to change this back after the workshop!)

During the following steps please make sure that the identation is correct! Either copy the spaces at the beginning of the lines too or type 4 spaces before inserting. You can check whether there is a problem in the commander window where manage.py runs. If it still ends with the line  ```Quit the server with CTRL-BREAK.```   then everything is Ok. Otherwise it will tell you in which line the error is.

1.a) Go to the section `#Reading the data from MongoDB` and copy the following:     
```Python
    Data_ptr_allpubs = db.allpubs.find({}, {'geometry.coordinates': 1})
```    
This is a pointer to the collection.     
The first {} section says load all documents (as it is left empty), this is where we will write the conditions later.     
The second {} defines which fields of the document should be returned. `1`means the returned documents are sorted in ascending order. `-1` would mean descending order.   
1.b) Now go to the section `#Transformation of the single documents to a JSON-List`    
   Here we will convert the dictionary to a list of JSON documents    
```Python
    docs_allpubs =[]
    for doc in Data_ptr_allpubs:
        doc_j = json.dumps(doc, default=json_util.default)
        docs_allpubs.append(doc_j)
```
1.c) Now we will specify what to send to our html file in the `Shows the header.html and sends the JSON-List as 'data_'` section. Insert `'data_allpubs': docs_allpubs` in the curly brackets :    
```Python
    return render(request, 'mapsite/header.html', { 'data_allpubs': docs_allpubs})
```
Save the file and reload the map in the browser. Hover over blue OpenLayers icon in the upper right corner and switch on the 'all pubs' layer. Now a lot of black markers should have appeared. If you zoom out, you will see that there are markers all over the country. We are however only interested in pubs near the conference location so let’s make a query which finds only the nearest pubs.    

#####2. Task: Visualize only the pubs which are in walking distance to our location    
2.a) In the previous task at `Data_ptr_allpubs` we have left the first {} empty. This is where our query goes:
```Python
'geometry': {
                '$near' : {
                    '$geometry': {
                        'type': "Point" ,
                        'coordinates':  coord
                    } , 
                '$maxDistance': 500 
                }
            }
```
So the whole query looks like this:    
```Python
    Data_ptr_allpubs = db.allpubs.find({
            'geometry': {
                '$near' : {
                    '$geometry': {
                        'type': "Point" ,
                        'coordinates':  coord
                    } , 
                '$maxDistance': 500 
                }
            }
        }, 
        {'geometry.coordinates': 1}
    )
```
Save the file and refresh the map. Now the markers should appear only within Helsinki.

#####3. Task: Create 4 layers for the 4 document types (Twitter, Facebook, Foursquare & OSM)    
3.a) For this we have to analyze the data: which fields are unique in each collection?    
3.b) For each layer we will use this unique field to query based on its existence:   
```Python
    #Foursquare
    Data_ptr_fs = db.allpubs.find(
        {
            'properties.rating': { 
                    '$exists': True
            }
        }, 
        {
            'geometry.coordinates': 1,
            'properties.name': 1
        }
    )
```
This returns all documents from the 'allpubs' collection, which have a 'properties.rating' field. Ergo only the Foursquare documents.

In the second {} we have inserted the properties.name field too, so that it shows up in the popup for the marker.
Now paste all 4 layers into the `#Reading the data from MongoDB`section, after the `allpubs` layer:
```Python
    #Foursquare
    Data_ptr_fs = db.allpubs.find(
        {
            'properties.rating': { 
                    '$exists': True
            }
        }, 
        {
            'geometry.coordinates': 1,
            'properties.name': 1
        }
    )
    
    #Twitter
    Data_ptr_tw = db.allpubs.find(
        {
            'properties.user': {
                '$exists': True
            }
        }, 
        {
            'geometry.coordinates': 1,
            'properties.user': 1 
        }
    )
    
    #Facebook
    Data_ptr_fb = db.allpubs.find(
        {
            'properties.category_list': {
                '$exists': True
            }
        },
        {
            'geometry.coordinates': 1,
            'properties.name':1
        }
    )
    
    #OSM
    Data_ptr_osm = db.allpubs.find(
        {
            'properties.amenity': {
                '$exists': True
            }
        }, 
        {
            'geometry.coordinates': 1,
            'properties.name': 1
        }
    )
```
3.c) Paste the following code to the `#Transformation of the single documents to a JSON-List` section:
```Python
    #Transformation of the single documents to a JSON-List
    docs_tw = []
    for doc in Data_ptr_tw:
        doc_j = json.dumps(doc, default=json_util.default)
        docs_tw.append(doc_j)

    docs_fb =[]
    for doc in Data_ptr_fb:
            doc_j = json.dumps(doc, default=json_util.default)
            docs_fb.append(doc_j)
	
    docs_fs =[]
    for doc in Data_ptr_fs:
            doc_j = json.dumps(doc, default=json_util.default)
            docs_fs.append(doc_j)

			
    docs_osm =[]
    for doc in Data_ptr_osm:
            doc_j = json.dumps(doc, default=json_util.default)
            docs_osm.append(doc_j)
```
3.d) Add the 4 new layer to the last row, right before `'data_allpubs':docs_allpubs`:
`'data_tw': docs_tw, 'data_fb': docs_fb, 'data_fs' : docs_fs, 'data_osm': docs_osm, `
This should look like this:   
```Python
 return render(request, 'mapsite/header.html', { 'data_tw': docs_tw, 'data_fb': docs_fb, 'data_fs' : docs_fs, 'data_osm': docs_osm, 'data_allpubs':docs_allpubs})
 ```
 If you refresh the map, you should see 3 colours. To be able to look at the Twitter data, switch on the Twitter layer in the upper right corner.
 
#####4. Task: Restrict the pubs to walking distance for the Foursquare layer as well.     
4.a) To be able to query multiple conditions we will need the $and operator. This is how it works:     
`$and: [{condition1}, {condition2}]`
So for example the query for Foursquare will look like this:     
```Python
        {
            '$and': [
                {
                    'properties.rating': {
                        '$exists': True
                    }
                },
                {
                    'geometry': {
                        '$near' : {
                            '$geometry': {
                                'type': "Point" ,
                                'coordinates':  coord
                            } , 
                            '$maxDistance': 500 
                        }
                    }
                }
            ]
        }
```
This is how the whole pointer looks like:
```Python

    Data_ptr_fs = db.allpubs.find(
        {
            '$and': [
                {
                    'properties.rating': {
                        '$exists': True
                    }
                },
                {
                    'geometry': {
                        '$near' : {
                            '$geometry': {
                                'type': "Point" ,
                                'coordinates':  coord
                            } , 
                            '$maxDistance': 500 
                        }
                    }
                }
            ]
        }, 
        {
            'geometry.coordinates': 1,
            'properties.name': 1,
            'properties.rating': 1,
            
        }
    )
```
4.b) Notice how we have inserted “properties.rating” in the list of fields as well. This is needed so the rating information appears in the popup.         
4.c) Save the file and refresh the map     
If you want to, you can write the walking distance query for all layers.     
Okay, so now we have a map with (Foursquare-)pubs within walking distance. If you click on the markers, you will see some information on the name of the pub, the source of the data and, in case of Foursquare, the rating as well. We can refine the query once more, so that only the best-rated pubs show (e.g. with better rating than 8). So within the list of conditions for the field 'properties.rating' we insert a new one:
```Python
                    'properties.rating': { 
                        '$exists': True,
                        '$gt': 8
                    }
```

The problem is, only Foursquare has information about the quality of the pubs. So in partII we will take a different approach: we will go ahead and assume that the more it was tweeted from a pub, the better it is. 

## Part II - Working with Twitter data

Below you can find the code for the query which counts the tweets in each pub. If you want to skip creating the code by yourself, you can just open count_tweets.py with PyCharm and run it or     
open a cmd where count_tweets.py is located and type    
`python count_tweets.py`    
This creates a new collection called 'smallarea', where each pub has a new field called 'tweet_count'. To make use of this new field skip to Task 2.     

#####1. Task: create a new collection with pubs within 500m and add a new field called 'tweet_count' which contains the number of tweets in a given pub.    
1.a) Open PyCharm or Notepad++ and create a new Python file. Let's call it select_smallarea.py    
In this first step we are going to create a new collection     
Import the libraries    

```Python
import json
import pymongo
```
Create a connection to MongoDB     
```Python
conn = pymongo.MongoClient()
```
Specify which databse you want to connect to. In our case it is the 'pubs' database    
```Python
db = conn.pubs
```
Write the query which finds all pubs within 500 meter to the conference location    
```Python
allpubs_near = db.allpubs.find({'geometry': { '$near' : { '$geometry': {'type': "Point" , 'coordinates':  [24.9495854, 60.1694509]} , '$maxDistance': 500 }}})
```
Insert each document the query found into a new collection called 'smallarea'
```Python
for pub in allpubs_near:
     db.smallarea.insert(pub)
```
Create the 2dsphere index for the new collection
```Python
db.smallarea.ensure_index([("geometry", pymongo.GEOSPHERE)])
```
Close the connection
```Python
conn.close()
```
Run the file and check in the mongo console whether the collection was created.     

1.b) Create a new Python file called tweets_count.py
Import the libraries and create the connection to MongoDB:     

```Python
import json
import pymongo

conn = pymongo.MongoClient()
db = conn.pubs
```
Now write a query which finds all documents in the smallarea collection which are not tweets (hence do not have the field 'user')     
```Python
pubs = db.smallarea.find({'properties.user': { '$exists': False }})
```
Now for each of these pubs count the number of tweets within 50m:
```Python
for pub in pubs:
    tweetcount = db.smallarea.count({'$and': [
        {'properties.user': { '$exists': True}},
        {"geometry": {"$near":
                            {"$geometry": {
                                "type": "Point" ,
                                "coordinates": [ pub['geometry']['coordinates'][0] , pub['geometry']['coordinates'][1]  ]
                                },
                             "$maxDistance": 50,
                             "$minDistance": 0
                            }
                        }
        }
    ]})
```
Then update each of these documents with a new field 'tweet_count' and as a value set number of tweets counted in the previous step:    
```Python
db.smallarea.update({'_id' : pub['_id']},{"$set": {'properties.tweet_count': tweetcount}}, upsert=False)
```
Close the connection:
```Python
conn.close()
```
#####2. Task: Switch back to views.py and add the new collection 'smallarea' after the existing layers     
```Python
Data_ptr_smallarea = db.smallarea.find({'properties.tweet_count': { '$exists': True}}, {'geometry.coordinates': 1, '_id': 0, 'properties.name': 1, 'properties.tweet_count': 1})
```
Convert the pointer to list of JSON documents:
```Python
docs_smallarea =[]
    for doc in Data_ptr_smallarea:
            doc_j = json.dumps(doc, default=json_util.default)
            docs_smallarea.append(doc_j)
```
Don't forget to add the smallarea documents to the last line:
```Python
'data_smallarea':docs_smallarea
```
