# Yelp Dataset

This dataset is a small portion of Yelp's businesses, reviews, and user data. It was originally put together for the Yelp Dataset Challenge which is a chance for students to conduct research or analysis on Yelp's data and share their discoveries. In the dataset you'll find information about businesses across 11 metropolitan areas in two countries.

More information can be found at [Yelp](https://www.yelp.com/dataset/documentation/main).  
Some dataset examples can be found on their [GitHub](https://github.com/Yelp/dataset-examples) page.

Currently, the metropolitan areas centered on Montreal, Calgary, Toronto, Pittsburgh, Charlotte, Urbana-Champaign, Phoenix, Las Vegas, Madison, and Cleveland, are included in the dataset.  

Summary for contents of each JSON file:  

**business.json**  
Contains business data including location data, attributes, and categories

**review.json**  
Contains full review text data including the user_id that wrote the review and the business_id the review is written for.

**user.json**  
User data including the user's friend mapping and all the metadata associated with the user.

**checkin.json**  
Checkins on a business.

**tip.json**  
Tips written by a user on a business. Tips are shorter than reviews and tend to convey quick suggestions.

**photo.json**  
Contains photo data including the caption and classification (one of "food", "drink", "menu", "inside" or "outside").


# MongoDB 

This notebook is using the Yelp Dataset from [Kaggle](https://www.kaggle.com/yelp-dataset/yelp-dataset) which is first being downloaded as JSON files and then being inserted into a MongoDB database on my local hard drive. This step could be skipped and the json files can be analysed directly using

```pandas.read_json(JSON_file, lines=True, nrows=int)``` since these files are large is it recommended to use nrows.

This is a great chance to upload these files into MongoDB since it uses JSON-like documents.  

Below is a shortened list of SQL to MongoDB Mapping chart from the offical [docs](https://docs.mongodb.com/manual/reference/sql-comparison/)  

| SQL Terms/Concepts|MongoDB Terms/Concepts|
|:----------:|:-------------:|
| database |  database |
| table |    collection   |
| row | document or BSON object |
| column | filed |
| index | index |
| table joins | $lookup |

We start by import libraries and then setting up MongoDB database below using [pymongo](https://pymongo.readthedocs.io/en/stable/) driver

# Import Libraries


```python
import json
import math
from pymongo import MongoClient, ASCENDING
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
from mpl_toolkits.basemap import Basemap
from collections import Counter
```

# JSON data
Download data from [Kaggle](https://www.kaggle.com/yelp-dataset/yelp-dataset) and read JSON files into memory using json library.


```python
data_path0 = 'yelp-data/yelp_academic_dataset_business.json'
data_path1 = 'yelp-data/yelp_academic_dataset_checkin.json'
data_path2 = 'yelp-data/yelp_academic_dataset_review.json'
data_path3 = 'yelp-data/yelp_academic_dataset_tip.json'
data_path4 = 'yelp-data/yelp_academic_dataset_user.json'
```


```python
data_paths = [data_path0, data_path1, data_path2, data_path3, data_path4]
json_data = [[],[],[],[],[]]

for i, file in enumerate(data_paths,0):
    with open(file) as f:
        for line in f:
            json_data[i].append(json.loads(line))
```


```python
#pd.read_json(data_path0, lines=True, nrows=5)
```

# MongoDB

## Creating Collections and inserting documents

Create Yelp Database below, insert new collection called business and using `insert_many()` method the `json_data[0]` above is inserted into this collection.


```python
client = MongoClient('mongodb://localhost:27017/')
db = client['Yelp']
business = db['business']
business.insert_many(json_data[0])
```




    <pymongo.results.InsertManyResult at 0x5c2660700>




```python
business.find_one()
```




    {'_id': ObjectId('5f73c43926a5b6392883c222'),
     'business_id': 'f9NumwFMBDn751xgFiRbNA',
     'name': 'The Range At Lake Norman',
     'address': '10913 Bailey Rd',
     'city': 'Cornelius',
     'state': 'NC',
     'postal_code': '28031',
     'latitude': 35.4627242,
     'longitude': -80.8526119,
     'stars': 3.5,
     'review_count': 36,
     'is_open': 1,
     'attributes': {'BusinessAcceptsCreditCards': 'True',
      'BikeParking': 'True',
      'GoodForKids': 'False',
      'BusinessParking': "{'garage': False, 'street': False, 'validated': False, 'lot': True, 'valet': False}",
      'ByAppointmentOnly': 'False',
      'RestaurantsPriceRange2': '3'},
     'categories': 'Active Life, Gun/Rifle Ranges, Guns & Ammo, Shopping',
     'hours': {'Monday': '10:0-18:0',
      'Tuesday': '11:0-20:0',
      'Wednesday': '10:0-18:0',
      'Thursday': '11:0-20:0',
      'Friday': '11:0-20:0',
      'Saturday': '11:0-20:0',
      'Sunday': '13:0-18:0'}}



---

Repeat creating collections with other JSON data

In the end there will be 5 collections.


```python
checkin = db['checkin']
checkin.insert_many(json_data[1])
```




    <pymongo.results.InsertManyResult at 0x5c3a3a240>




```python
checkin.find_one()
```




    {'_id': ObjectId('5f73c97226a5b6392886f413'),
     'business_id': '--1UhMGODdWsrMastO9DZw',
     'date': '2016-04-26 19:49:16, 2016-08-30 18:36:57, 2016-10-15 02:45:18, 2016-11-18 01:54:50, 2017-04-20 18:39:06, 2017-05-03 17:58:02, 2019-03-19 22:04:48'}



---

The review JSON file is 6.33 GBs in size and the `insert_many()` function will not be good approach due to memory limitations and looping through the data and inserting batches will be better.


```python
review = db['review']
```


```python
batches = 100 #number of batches
batch_length = math.ceil(len(json_data[2])/batches) #number of items in each batch
tmp_ = 0 #keep track for number of items in a batch

for i in range(batches):
    data = json_data[2][tmp_:tmp_+batch_length]
    review.insert_many(data)
    tmp_ += batch_length

review.find_one()
```




    {'_id': ObjectId('5f73c9a626a5b6392889a066'),
     'review_id': 'xQY8N_XvtGbearJ5X4QryQ',
     'user_id': 'OwjRMXRC0KyPrIlcjaXeFQ',
     'business_id': '-MhfebM0QIsKt87iDN-FNw',
     'stars': 2.0,
     'useful': 5,
     'funny': 0,
     'cool': 0,
     'text': 'As someone who has worked with many museums, I was eager to visit this gallery on my most recent trip to Las Vegas. When I saw they would be showing infamous eggs of the House of Faberge from the Virginia Museum of Fine Arts (VMFA), I knew I had to go!\n\nTucked away near the gelateria and the garden, the Gallery is pretty much hidden from view. It\'s what real estate agents would call "cozy" or "charming" - basically any euphemism for small.\n\nThat being said, you can still see wonderful art at a gallery of any size, so why the two *s you ask? Let me tell you:\n\n* pricing for this, while relatively inexpensive for a Las Vegas attraction, is completely over the top. For the space and the amount of art you can fit in there, it is a bit much.\n* it\'s not kid friendly at all. Seriously, don\'t bring them.\n* the security is not trained properly for the show. When the curating and design teams collaborate for exhibitions, there is a definite flow. That means visitors should view the art in a certain sequence, whether it be by historical period or cultural significance (this is how audio guides are usually developed). When I arrived in the gallery I could not tell where to start, and security was certainly not helpful. I was told to "just look around" and "do whatever." \n\nAt such a *fine* institution, I find the lack of knowledge and respect for the art appalling.',
     'date': '2015-04-15 05:21:16'}



---

The collection 'tip' below will be created and then the JSON data will be inserted into it.


```python
tip = db['tip']
tip.insert_many(json_data[3])
tip.find_one()
```




    {'_id': ObjectId('5f73de0226a5b639280404e8'),
     'user_id': 'hf27xTME3EiCp6NL6VtWZQ',
     'business_id': 'UYX5zL_Xj9WEc_Wp-FrqHw',
     'text': 'Here for a quick mtg',
     'date': '2013-11-26 18:20:08',
     'compliment_count': 0}



Not unlike the JSON file above, the user JSON file is 3.27 GBs in size and the `insert_many()` function below will take some time, and our solution will be the same.


```python
user = db['user'] #create collection
batches = 100
batch_length = math.ceil(len(json_data[4])/batches) #number of items in each batch
tmp_ = 0 #keep track for number of items in a batch

for i in range(batches):
    data = json_data[4][tmp_:tmp_+batch_length]
    user.insert_many(data)
    tmp_ += batch_length

user.find_one()
```




    {'_id': ObjectId('5f73e31226a5b63928182c21'),
     'user_id': 'ntlvfPzc8eglqvk92iDIAw',
     'name': 'Rafael',
     'review_count': 553,
     'yelping_since': '2007-07-06 03:27:11',
     'useful': 628,
     'funny': 225,
     'cool': 227,
     'elite': '',
     'friends': 'oeMvJh94PiGQnx_6GlndPQ, wm1z1PaJKvHgSDRKfwhfDg, IkRib6Xs91PPW7pon7VVig, A8Aq8f0-XvLBcyMk2GJdJQ, eEZM1kogR7eL4GOBZyPvBA, e1o1LN7ez5ckCpQeAab4iw, _HrJVzFaRFUhPva8cwBjpQ, pZeGZGzX-ROT_D5lam5uNg, 0S6EI51ej5J7dgYz3-O0lA, woDt8raW-AorxQM_tIE2eA, hWUnSE5gKXNe7bDc8uAG9A, c_3LDSO2RHwZ94_Q6j_O7w, -uv1wDiaplY6eXXS0VwQiA, QFjqxXn3acDC7hckFGUKMg, ErOqapICmHPTN8YobZIcfQ, mJLRvqLOKhqEdkgt9iEaCQ, VKX7jlScJSA-ja5hYRw12Q, ijIC9w5PRcj3dWVlanjZeg, CIZGlEw-Bp0rmkP8M6yQ9Q, OC6fT5WZ8EU7tEVJ3bzPBQ, UZSDGTDpycDzrlfUlyw2dQ, deL6e_z9xqZTIODKqnvRXQ, 5mG2ENw2PylIWElqHSMGqg, Uh5Kug2fvDd51RYmsNZkGg, 4dI4uoShugD9z84fYupelQ, EQpFHqGT9Tk6YSwORTtwpg, o4EGL2-ICGmRJzJ3GxB-vw, s8gK7sdVzJcYKcPv2dkZXw, vOYVZgb_GVe-kdtjQwSUHw, wBbjgHsrKr7BsPBrQwJf2w, p59u2EC_qcmCmLeX1jCi5Q, VSAZI1eHDrOPRWMK4Q2DIQ, efMfeI_dkhpeGykaRJqxfQ, x6qYcQ8_i0mMDzSLsFCbZg, K_zSmtNGw1fu-vmxyTVfCQ, 5IM6YPQCK-NABkXmHhlRGQ, U_w8ZMD26vnkeeS1sD7s4Q, AbfS_oXF8H6HJb5jFqhrLw, hbcjX4_D4KIfonNnwrH-cg, UKf66_MPz0zHCP70mF6p1g, hK2gYbxZRTqcqlSiQQcrtQ, 2Q45w_Twx_T9dXqlE16xtQ, BwRn8qcKSeA77HLaOTbfiQ, jouOn4VS_DtFPtMR2w8VDA, ESteyJabbfvqas6CEDs3pQ',
     'fans': 14,
     'average_stars': 3.57,
     'compliment_hot': 3,
     'compliment_more': 2,
     'compliment_profile': 1,
     'compliment_cute': 0,
     'compliment_list': 1,
     'compliment_note': 11,
     'compliment_plain': 15,
     'compliment_cool': 22,
     'compliment_funny': 22,
     'compliment_writer': 10,
     'compliment_photos': 0}




```python
db.command("dbstats")
```




    {'db': 'Yelp',
     'collections': 3,
     'views': 0,
     'objects': 8405702,
     'avgObjSize': 852.5322018315662,
     'dataSize': 7166131634.0,
     'storageSize': 4791033856.0,
     'numExtents': 0,
     'indexes': 3,
     'indexSize': 84541440.0,
     'scaleFactor': 1.0,
     'fsUsedSize': 460941709312.0,
     'fsTotalSize': 500068036608.0,
     'ok': 1.0}




```python
tmp = db.command( { 'collStats': 'business', 'scale': 1024000 } )
tmp['size'],tmp['count']
```




    (156, 209393)




```python
tmp = db.command( { 'collStats': 'checkin', 'scale': 1024000 } )
tmp['size'],tmp['count']
```




    (442, 175187)




```python
tmp = db.command( { 'collStats': 'review', 'scale': 1024000 } )
tmp['size'],tmp['count']
```




    (6398, 8021122)




```python
tmp = db.command( { 'collStats': 'tip', 'scale': 1024000 } )
tmp['size'],tmp['count']
```




    (289, 1320761)




```python
tmp = db.command( { 'collStats': 'user', 'scale': 1024000 } )
tmp['size'],tmp['count']
```




    (3272, 1968703)



## MongoDB Initialization


```python
client = MongoClient('mongodb://localhost:27017/')
db = client['Yelp']
business = db['business']
checkin = db['checkin']
review = db['review']
tip = db['tip']
user = db['user']
```


```python
pipeline = [
    {'$lookup':{'from' : 'tip',
                'localField' : 'business_id',
                'foreignField' : 'business_id',
                'as' : 'buis_tip'}},
    {'$replaceRoot':{'newRoot':{'$mergeObjects':[{'$arrayElemAt':["$buis_tip",0]}, "$$ROOT"]}}},
    {'$project': {'buis_tip':0, 'compliment_count':0, 'address':0, 'postal_code':0, 'latitude':0, 'longitude':0,'attributes':0, 'categories':0, 'hours':0}}
]

#x = business.aggregate(pipeline)
#df_agg = pd.DataFrame(list(x))
```

We will use the JSON data directly from files from now on since they are large and loading them into dataframes, from a mongoDB database, takes a long time using `pd.DataFrame(list(x))`


```python
df = pd.read_json(data_path0, lines=True)
```


```python
df.head(2)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>business_id</th>
      <th>name</th>
      <th>address</th>
      <th>city</th>
      <th>state</th>
      <th>postal_code</th>
      <th>latitude</th>
      <th>longitude</th>
      <th>stars</th>
      <th>review_count</th>
      <th>is_open</th>
      <th>attributes</th>
      <th>categories</th>
      <th>hours</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>f9NumwFMBDn751xgFiRbNA</td>
      <td>The Range At Lake Norman</td>
      <td>10913 Bailey Rd</td>
      <td>Cornelius</td>
      <td>NC</td>
      <td>28031</td>
      <td>35.462724</td>
      <td>-80.852612</td>
      <td>3.5</td>
      <td>36</td>
      <td>1</td>
      <td>{'BusinessAcceptsCreditCards': 'True', 'BikePa...</td>
      <td>Active Life, Gun/Rifle Ranges, Guns &amp; Ammo, Sh...</td>
      <td>{'Monday': '10:0-18:0', 'Tuesday': '11:0-20:0'...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Yzvjg0SayhoZgCljUJRF9Q</td>
      <td>Carlos Santo, NMD</td>
      <td>8880 E Via Linda, Ste 107</td>
      <td>Scottsdale</td>
      <td>AZ</td>
      <td>85258</td>
      <td>33.569404</td>
      <td>-111.890264</td>
      <td>5.0</td>
      <td>4</td>
      <td>1</td>
      <td>{'GoodForKids': 'True', 'ByAppointmentOnly': '...</td>
      <td>Health &amp; Medical, Fitness &amp; Instruction, Yoga,...</td>
      <td>None</td>
    </tr>
  </tbody>
</table>
</div>



# EDA & Visualizations

Lets take a look where most of the businesses are located in this data set by plotting them onto a globe.


```python
plt.figure(figsize=(20,10));
map = Basemap(projection='ortho',lat_0=25,lon_0=-100,resolution='l')
map.bluemarble()
# draw coastlines, country boundaries, fill continents.
map.drawcoastlines(linewidth=0.25)
map.drawcountries(linewidth=0.25)
map.fillcontinents(color='green',lake_color='blue')
# draw the edge of the map projection region (the projection limb)
map.drawmapboundary(fill_color='blue')
long_lat = map(df['longitude'].tolist(),df['latitude'].tolist())
map.scatter(long_lat[0], long_lat[1], s=3, c="orange", lw=3, alpha=1, zorder=5)
plt.title("World-wide Yelp Reviews")
plt.show();
```

    Clipping input data to the valid range for imshow with RGB data ([0..1] for floats or [0..255] for integers).



![png](img/output_42_1.png)



```python
print(df['longitude'].min())
print(df['longitude'].max())
```

    -158.0255252123
    -72.80655


We note from negative values for longitude that all the businesses in the yelp database are found in North America, in particular Canada and the United States. 

Lets take a closer look at Greater Toronto Area (GTA) which is the most populous metropolitan area in Canada.

## Greater Toronto Area (GTA)


```python
lon_min, lon_max = -80, -78.8
lat_min, lat_max = 43.2, 44.2

TOR = ((df["longitude"]>lon_min) &(df["longitude"]<lon_max)) &\
            ((df["latitude"]>lat_min) & (df["latitude"]<lat_max))

TOR_business = df[TOR].copy()
```


```python
plt.figure(figsize=(20,10));
map = Basemap(projection='merc',llcrnrlat=lat_min,urcrnrlat=lat_max,llcrnrlon=lon_min,urcrnrlon=lon_max, resolution='f')
#map.bluemarble()
# country boundaries, fill continents.
map.drawcountries(linewidth=0.25)
map.fillcontinents(color='black',lake_color='blue')
# draw the edge of the map projection region (the projection limb)
map.drawmapboundary(fill_color='blue')
TOR_ = map(TOR_business['longitude'].tolist(),TOR_business['latitude'].tolist())
map.scatter(TOR_[0], TOR_[1], s=3, c="orange", lw=3, alpha=1, zorder=5)
plt.title("World-wide Yelp Reviews")
plt.show();
```


![png](img/output_47_0.png)



```python
TOR_business.sort_values(by='review_count', ascending=False).head(5)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>business_id</th>
      <th>name</th>
      <th>address</th>
      <th>city</th>
      <th>state</th>
      <th>postal_code</th>
      <th>latitude</th>
      <th>longitude</th>
      <th>stars</th>
      <th>review_count</th>
      <th>is_open</th>
      <th>attributes</th>
      <th>categories</th>
      <th>hours</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>65694</th>
      <td>r_BrIgzYcwo1NAuG9dLbpg</td>
      <td>Pai Northern Thai Kitchen</td>
      <td>18 Duncan Street</td>
      <td>Toronto</td>
      <td>ON</td>
      <td>M5H 3G8</td>
      <td>43.647866</td>
      <td>-79.388685</td>
      <td>4.5</td>
      <td>2758</td>
      <td>1</td>
      <td>{'RestaurantsTableService': 'True', 'BikeParki...</td>
      <td>Restaurants, Thai, Specialty Food, Food, Ethni...</td>
      <td>{'Monday': '11:30-22:0', 'Tuesday': '11:30-22:...</td>
    </tr>
    <tr>
      <th>183740</th>
      <td>RtUvSWO_UZ8V3Wpj0n077w</td>
      <td>KINKA IZAKAYA ORIGINAL</td>
      <td>398 Church St</td>
      <td>Toronto</td>
      <td>ON</td>
      <td>M5B 2A2</td>
      <td>43.660430</td>
      <td>-79.378927</td>
      <td>4.0</td>
      <td>1592</td>
      <td>1</td>
      <td>{'RestaurantsAttire': 'u'casual'', 'BusinessPa...</td>
      <td>Restaurants, Tapas/Small Plates, Japanese, Bar...</td>
      <td>{'Monday': '17:0-0:0', 'Tuesday': '17:0-0:0', ...</td>
    </tr>
    <tr>
      <th>143305</th>
      <td>aLcFhMe6DDJ430zelCpd2A</td>
      <td>Khao San Road</td>
      <td>11 Charlotte Street</td>
      <td>Toronto</td>
      <td>ON</td>
      <td>M5V 2H5</td>
      <td>43.646411</td>
      <td>-79.393480</td>
      <td>4.0</td>
      <td>1542</td>
      <td>1</td>
      <td>{'WiFi': 'u'no'', 'RestaurantsTakeOut': 'True'...</td>
      <td>Thai, Restaurants</td>
      <td>{'Monday': '17:0-22:0', 'Tuesday': '17:0-22:0'...</td>
    </tr>
    <tr>
      <th>133959</th>
      <td>iGEvDk6hsizigmXhDKs2Vg</td>
      <td>Seven Lives Tacos Y Mariscos</td>
      <td>69 Kensington Avenue</td>
      <td>Toronto</td>
      <td>ON</td>
      <td>M5T 2K2</td>
      <td>43.654341</td>
      <td>-79.400480</td>
      <td>4.5</td>
      <td>1285</td>
      <td>1</td>
      <td>{'RestaurantsGoodForGroups': 'False', 'Alcohol...</td>
      <td>Restaurants, Seafood, Mexican</td>
      <td>{'Monday': '11:0-19:0', 'Tuesday': '11:0-19:0'...</td>
    </tr>
    <tr>
      <th>172038</th>
      <td>N93EYZy9R0sdlEvubu94ig</td>
      <td>Banh Mi Boys</td>
      <td>392 Queen Street W</td>
      <td>Toronto</td>
      <td>ON</td>
      <td>M5V 2A9</td>
      <td>43.648827</td>
      <td>-79.396970</td>
      <td>4.5</td>
      <td>1097</td>
      <td>1</td>
      <td>{'Alcohol': 'u'none'', 'BikeParking': 'True', ...</td>
      <td>Sandwiches, Restaurants, Food, Vietnamese, Asi...</td>
      <td>{'Monday': '11:0-22:0', 'Tuesday': '11:0-22:0'...</td>
    </tr>
  </tbody>
</table>
</div>




```python
TOR_business['review_count'].describe()
```




    count    36636.000000
    mean        24.111830
    std         54.647122
    min          3.000000
    25%          4.000000
    50%          8.000000
    75%         21.000000
    max       2758.000000
    Name: review_count, dtype: float64




```python
TOR_business['review_count'].hist();
```


![png](img/output_50_0.png)


The 'review_count' column is the number of reviews a business has received and we see in the histogram above that there are some business with thousands of reviews but 75% of the businesses in this dataset only have 21 reviews or less. 

The histogram below only shows business in Toronto will less than 100 reviews.


```python
t_ = TOR_business[TOR_business['review_count']<100]['review_count'].hist()
```


![png](img/output_52_0.png)



```python
fig, ax = plt.subplots(figsize=(20,10))
ax.spines['top'].set_visible(False)
ax.spines['right'].set_visible(False)
ax.tick_params(bottom=False, left=True)

ax.set_axisbelow(True)
ax.yaxis.grid(True)
ax.xaxis.grid(False)
plt.title('Top 10 Type of Businesses in Toronto')
plt.ylabel('Number of Business with category listed')
plt.xticks(np.arange(10), TOR_business['categories'].value_counts().head(10), rotation=45)

bars = ax.bar(
    x=np.arange(10),
    height=TOR_business['categories'].value_counts().head(10),
    color='teal',
    tick_label=TOR_business['categories'].value_counts().head(10).index
)

for bar in bars:
    plt.text(
      bar.get_x() + bar.get_width() / 2,
      bar.get_height() + 4,
      round(bar.get_height(), 1),
      horizontalalignment='center',
      color='teal',
      weight='bold',
      size=15
    )
```


![png](img/output_53_0.png)


We can see in the figure above the top ten buisnesses (by count of each category) are counted twice. For example the number one most common business is 'Coffee & Tea, Food' and the second more common is 'Food, Coffee & Tea'. This needs to be addressed by combining the categories which have the text in their descriptions mixed up.


```python
TOR_business['categories'].value_counts().head(10)
```




    Coffee & Tea, Food            306
    Food, Coffee & Tea            303
    Restaurants, Chinese          300
    Chinese, Restaurants          280
    Hair Salons, Beauty & Spas    244
    Beauty & Spas, Hair Salons    243
    Pizza, Restaurants            203
    Restaurants, Pizza            199
    Nail Salons, Beauty & Spas    173
    Grocery, Food                 169
    Name: categories, dtype: int64




```python
len(TOR_business['categories'].unique())
```




    19661




```python
TOR_business.loc[TOR_business['categories'] == 'Food, Coffee & Tea', 'categories'] = 'Coffee & Tea, Food'
TOR_business.loc[TOR_business['categories'] == 'Chinese, Restaurants', 'categories'] = 'Restaurants, Chinese'
TOR_business.loc[TOR_business['categories'] == 'Beauty & Spas, Hair Salons', 'categories'] = 'Hair Salons, Beauty & Spas'
TOR_business.loc[TOR_business['categories'] == 'Restaurants, Pizza', 'categories'] = 'Pizza, Restaurants'
TOR_business.loc[TOR_business['categories'] == 'Beauty & Spas, Nail Salons', 'categories'] = 'Nail Salons, Beauty & Spas'
TOR_business.loc[TOR_business['categories'] == 'Restaurants, Italian', 'categories'] = 'Italian, Restaurants'
TOR_business.loc[TOR_business['categories'] == 'Food, Grocery', 'categories'] = 'Grocery, Food'
TOR_business.loc[TOR_business['categories'] == 'Food, Bakeries', 'categories'] = 'Bakeries, Food'
TOR_business.loc[TOR_business['categories'] == 'Indian, Restaurants', 'categories'] = 'Restaurants, Indian'
TOR_business.loc[TOR_business['categories'] == 'Restaurants, Vietnamese', 'categories'] = 'Vietnamese, Restaurants'
TOR_business.loc[TOR_business['categories'] == 'Restaurants, Japanese', 'categories'] = 'Japanese, Restaurants'
TOR_business.loc[TOR_business['categories'] == 'Thai, Restaurants', 'categories'] = 'Restaurants, Thai'
```


```python
TOR_business['categories'].value_counts().head(10)
```




    Coffee & Tea, Food            609
    Restaurants, Chinese          580
    Hair Salons, Beauty & Spas    487
    Pizza, Restaurants            402
    Nail Salons, Beauty & Spas    340
    Grocery, Food                 323
    Italian, Restaurants          282
    Bakeries, Food                256
    Restaurants, Indian           239
    Japanese, Restaurants         212
    Name: categories, dtype: int64



---

Now we can plot the top 10 categories of businesses in Toronto:


```python
fig, ax = plt.subplots(figsize=(20,10))
ax.spines['top'].set_visible(False)
ax.spines['right'].set_visible(False)
ax.tick_params(bottom=False, left=True)

ax.set_axisbelow(True)
ax.yaxis.grid(True)
ax.xaxis.grid(False)
plt.title('Top 10 Type of Businesses in Toronto')
plt.ylabel('Number of Business with category listed')
plt.xticks(np.arange(10), TOR_business['categories'].value_counts().head(10), rotation=45)

bars = ax.bar(
    x=np.arange(10),
    height=TOR_business['categories'].value_counts().head(10),
    color='teal',
    tick_label=TOR_business['categories'].value_counts().head(10).index
)

for bar in bars:
    plt.text(
      bar.get_x() + bar.get_width() / 2,
      bar.get_height() + 4,
      round(bar.get_height(), 1),
      horizontalalignment='center',
      color='teal',
      weight='bold',
      size=15
    )
```


![png](img/output_61_0.png)


Below is a table for the average stars (1-5) for the top ten business categories above


```python
top_ten_list = TOR_business['categories'].value_counts().head(10).index.tolist()
TOR_business.groupby('categories').agg(['mean']).loc[top_ten_list,:].sort_values(by=('stars','mean'),ascending=False)['stars']
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>mean</th>
    </tr>
    <tr>
      <th>categories</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bakeries, Food</th>
      <td>3.705078</td>
    </tr>
    <tr>
      <th>Italian, Restaurants</th>
      <td>3.489362</td>
    </tr>
    <tr>
      <th>Hair Salons, Beauty &amp; Spas</th>
      <td>3.434292</td>
    </tr>
    <tr>
      <th>Coffee &amp; Tea, Food</th>
      <td>3.428571</td>
    </tr>
    <tr>
      <th>Restaurants, Indian</th>
      <td>3.424686</td>
    </tr>
    <tr>
      <th>Japanese, Restaurants</th>
      <td>3.318396</td>
    </tr>
    <tr>
      <th>Restaurants, Chinese</th>
      <td>3.192241</td>
    </tr>
    <tr>
      <th>Pizza, Restaurants</th>
      <td>3.136816</td>
    </tr>
    <tr>
      <th>Grocery, Food</th>
      <td>3.082043</td>
    </tr>
    <tr>
      <th>Nail Salons, Beauty &amp; Spas</th>
      <td>2.867647</td>
    </tr>
  </tbody>
</table>
</div>



Another approach to solving the problem above is to instead to split the strings by `,` (comma) in each row of the category column and using the `Counter()` function from the `collections` library, which we then increase the count for each string encountered.


```python
Tor_catlist = TOR_business['categories'].tolist()
```


```python
c = Counter()
for n in Tor_catlist:
    if n is not None:
        cat_list = n.split(', ')
        for cat in cat_list:
            c[cat] += 1
    else:
        c['N/A'] += 1
```


```python
c.most_common(10)
```




    [('Restaurants', 16227),
     ('Food', 7979),
     ('Shopping', 5596),
     ('Beauty & Spas', 3610),
     ('Nightlife', 2743),
     ('Coffee & Tea', 2456),
     ('Bars', 2452),
     ('Health & Medical', 1891),
     ('Chinese', 1817),
     ('Event Planning & Services', 1795)]




```python
TORcat = pd.DataFrame.from_dict(c, orient='index')
```


```python
TORcat.sort_values(by=0,ascending=False).head(10)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Restaurants</th>
      <td>16227</td>
    </tr>
    <tr>
      <th>Food</th>
      <td>7979</td>
    </tr>
    <tr>
      <th>Shopping</th>
      <td>5596</td>
    </tr>
    <tr>
      <th>Beauty &amp; Spas</th>
      <td>3610</td>
    </tr>
    <tr>
      <th>Nightlife</th>
      <td>2743</td>
    </tr>
    <tr>
      <th>Coffee &amp; Tea</th>
      <td>2456</td>
    </tr>
    <tr>
      <th>Bars</th>
      <td>2452</td>
    </tr>
    <tr>
      <th>Health &amp; Medical</th>
      <td>1891</td>
    </tr>
    <tr>
      <th>Chinese</th>
      <td>1817</td>
    </tr>
    <tr>
      <th>Event Planning &amp; Services</th>
      <td>1795</td>
    </tr>
  </tbody>
</table>
</div>




```python
tmp_ = TORcat.sort_values(by=0,ascending=False).head(10)

fig, ax = plt.subplots(figsize=(20,10))
ax.spines['top'].set_visible(False)
ax.spines['right'].set_visible(False)
ax.tick_params(bottom=False, left=True)

ax.set_axisbelow(True)
ax.yaxis.grid(True)
ax.xaxis.grid(False)
plt.title('Top 10 Type of Businesses in Toronto')
plt.ylabel('Number of Business with category listed')
plt.xticks(np.arange(10), tmp_.value_counts(), rotation=45)

bars = ax.bar(
    x=np.arange(10),
    height=tmp_[0].values,
    color='teal',
    tick_label=tmp_.index
)

for bar in bars:
    plt.text(
      bar.get_x() + bar.get_width() / 2,
      bar.get_height() + 100,
      round(bar.get_height(), 1),
      horizontalalignment='center',
      color='teal',
      weight='bold',
      size=15
    )
```


![png](img/output_70_0.png)


## More Top 10 Business Categories Analysis for GTA

The top ten businesses, by number of a businesses in a unique category, have their average rating shown above in descending order.


```python
plt.figure(figsize=(12,6))
TOR_business['stars'].hist(bins=17)
plt.title('Histogram of 1-5 Star Reviews on Yelp within Toronto')
plt.xlabel('Stars')
plt.ylabel('Number of reviews');
```


![png](img/output_73_0.png)



```python
top_ten_list = TOR_business['categories'].value_counts().head(10).index.tolist()
TOR_business.groupby('categories').agg(['mean']).loc[top_ten_list,:].sort_values(by=('stars','mean'),ascending=False)['stars']
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>mean</th>
    </tr>
    <tr>
      <th>categories</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bakeries, Food</th>
      <td>3.705078</td>
    </tr>
    <tr>
      <th>Italian, Restaurants</th>
      <td>3.489362</td>
    </tr>
    <tr>
      <th>Hair Salons, Beauty &amp; Spas</th>
      <td>3.434292</td>
    </tr>
    <tr>
      <th>Coffee &amp; Tea, Food</th>
      <td>3.428571</td>
    </tr>
    <tr>
      <th>Restaurants, Indian</th>
      <td>3.424686</td>
    </tr>
    <tr>
      <th>Japanese, Restaurants</th>
      <td>3.318396</td>
    </tr>
    <tr>
      <th>Restaurants, Chinese</th>
      <td>3.192241</td>
    </tr>
    <tr>
      <th>Pizza, Restaurants</th>
      <td>3.136816</td>
    </tr>
    <tr>
      <th>Grocery, Food</th>
      <td>3.082043</td>
    </tr>
    <tr>
      <th>Nail Salons, Beauty &amp; Spas</th>
      <td>2.867647</td>
    </tr>
  </tbody>
</table>
</div>




```python
plt.figure(figsize=(12,6))
TOR_business[TOR_business['categories'] == 'Bakeries, Food']['stars'].plot.kde();
plt.title('KDE of Reviews for Bakeries within Toronto')
plt.xlabel('Number of Stars')
plt.ylabel('Number of reviews');
```


![png](img/output_75_0.png)



```python
TOR_business[TOR_business['categories'] == 'Bakeries, Food'][['stars']].describe()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>stars</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>256.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>3.705078</td>
    </tr>
    <tr>
      <th>std</th>
      <td>0.779601</td>
    </tr>
    <tr>
      <th>min</th>
      <td>1.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>3.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>4.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>4.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>5.000000</td>
    </tr>
  </tbody>
</table>
</div>




```python
plt.figure(figsize=(12,6))
TOR_business[TOR_business['categories'] == 'Italian, Restaurants']['stars'].plot.kde();
plt.title('KDE of Reviews for Italian Restaurants within Toronto')
plt.xlabel('Number of Stars')
plt.ylabel('Number of reviews');
```


![png](img/output_77_0.png)



```python
TOR_business[TOR_business['categories'] == 'Italian, Restaurants'][['stars']].describe()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>stars</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>282.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>3.489362</td>
    </tr>
    <tr>
      <th>std</th>
      <td>0.678779</td>
    </tr>
    <tr>
      <th>min</th>
      <td>1.500000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>3.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>3.500000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>4.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>5.000000</td>
    </tr>
  </tbody>
</table>
</div>




```python
plt.figure(figsize=(12,6))
TOR_business[TOR_business['categories'] == 'Nail Salons, Beauty & Spas']['stars'].plot.kde();
plt.title('KDE of Reviews for Nail Salons, Beauty & Spas within Toronto')
plt.xlabel('Number of Stars')
plt.ylabel('Number of reviews');
```


![png](img/output_79_0.png)



```python
TOR_business[TOR_business['categories'] == 'Nail Salons, Beauty & Spas'][['stars']].describe()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>stars</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>340.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>2.867647</td>
    </tr>
    <tr>
      <th>std</th>
      <td>0.916972</td>
    </tr>
    <tr>
      <th>min</th>
      <td>1.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>2.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>3.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>3.500000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>5.000000</td>
    </tr>
  </tbody>
</table>
</div>



## Review.json


```python
df_ = pd.read_json(data_path2, lines=True)
TOR_review = df_[TOR].copy()
```


```python
TOR_review.head(2)
```


```python

```
