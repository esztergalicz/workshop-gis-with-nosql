# GIS with NoSQL Workshop Tutorial

## Intorduction

The aim of this hands-on session is to gain first experience with MongoDB, create basic geo-queries and visualize documents from MongoDB on a webmap (in this case OpenLayers). We will use data about pubs in Helsinki, collected from social media (Facebook, Foursquare and Twitter) and OSM.

So here is your task:
######Find the best pub near the conference locaiton!

## Part I
### 1. Fire up MongoDB
1. Start MongoDB by running mongod.exe from `C:\MongoDB\Server\3.2\bin\` (or click on the shortcut on the desktop)
2. To check what‘s in the database we will work with the mongo console    
Start a cmd (shortcut on desktop to cmd) from `C:\MongoDB\Server\3.2\bin\` 
& type   
``` >mongo ```    
you can ignore the warnings    
3. To list all databases type    
     ``` >show dbs ```   
4. To switch to a database type    
     ``` >use pubs ```   
5. To show all collections in the database type  
     ``` >show collections ```
6. To show a document from the collection (e.g. 'osm') type  
``` >db.osm.findOne() ```
7. Let’s take a look at all 4 collections! (Ignore 'allpubs' for now.) Try to find the coordinates of the location in each document.         
     ``` >db.facebook.findOne() ```  
     ``` >db.foursquare.findOne()   ```   
     ``` >db.twitter.findOne()     ```
    
See how they all have a different structure?     
To be able to easily work with them, they were all converted to GeoJSON format and loaded in the ‘allpubs’ collection.     
If you are interested how this is done, you can look it up in `Downloads\python\create_geojson.py` & `GeoTransformer.py`    

Converting to GeoJSON has other pros too: this way a 2dspehere index could be added which enables the geo-queries.    
This is how a geospatial index is added (e.g. for the 'osm' collection):    
 ``` >db.osm.createIndex( { "location" : "2dsphere" } ) ```      
 This should be the output:    
 ```JSON
 {
  "createdCollectionAutomatically" : false,
  "numIndexesBefore" : 1,
  "numIndexesAfter" : 2,
  "ok" : 1
}
```

