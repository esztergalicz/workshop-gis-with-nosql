# GIS with NoSQL Workshop Tutorial

## Intorduction

The aim of this hands-on session is to gain first experience with MongoDB, create basic geo-queries and visualize documents from MongoDB on a webmap (in this case OpenLayers). We will use data about pubs in Helsinki, collected from social media (Facebook, Foursquare and Twitter) and OSM.

So here is your task:
####Find the best pub near the conference locaiton!

## Part I
### 1. Fire up MongoDB
1. Start MongoDB by running `mongod.exe` from `C:\MongoDB\Server\3.2\bin\` (or click on the shortcut on the desktop)
2. To check what‘s in the database we will work with the mongo console    
Start a cmd from `C:\MongoDB\Server\3.2\bin\` (Shift+right-click in the folder --> Open command window here)
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

Converting to GeoJSON has another upside: this way a 2dspehere index could be added which enables the geo-queries.    
This is how a geospatial index is added (e.g. for the 'facebook' collection):    
 ``` >db.facebook.createIndex( { "geo" : "2dsphere" } ) ```      
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
We will now edit the views.py and write some queries to load the features from the database    

### 3. Visualizing MongoDB documents on the map
#####1. Task: visualize all pubs from the 'allpubs' collection
Open views.py (either with pycharm or with notepad++)
If you are using Notepad++ please change the following setting first:
Settings --> Preferences --> Tab Settings --> Tab size: 4    
Replace by space
1. 
