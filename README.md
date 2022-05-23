# SOLUTION - Part 1 - Extracting and Saving Data from Yelp API

## Obective

- For this CodeAlong, we will be working with the Yelp API. 
- You will use the the Yelp API to search your home town for a cuisine type of your choice.
- Next class, we will then use Plotly Express to create a map with the Mapbox API to visualize the results.
    
    

## Tools You Will Use
- Part 1:
    - Yelp API:
        - Getting Started: 
            - https://www.yelp.com/developers/documentation/v3/get_started

    - `YelpAPI` python package
        -  "YelpAPI": https://github.com/gfairchild/yelpapi
- Part 2:

    - Plotly Express: https://plotly.com/python/getting-started/
        - With Mapbox API: https://www.mapbox.com/
        - `px.scatter_mapbox` [Documentation](https://plotly.com/python/scattermapbox/): 




### Applying Code From
- Efficient API Calls Lesson Link: https://login.codingdojo.com/m/376/12529/88078


```python
# Standard Imports
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

# Additional Imports
import os, json, math, time
from yelpapi import YelpAPI
from tqdm.notebook import tqdm_notebook
```

## 1. Registering for Required APIs


- Yelp: https://www.yelp.com/developers/documentation/v3/get_started


> Check the official API documentation to know what arguments we can search for: https://www.yelp.com/developers/documentation/v3/business_search

### Load Credentials and Create Yelp API Object


```python
# Load API Credentials
with open('/Users/codingdojo/.secret/yelp_api.json') as f:   #use your path here!
    login = json.load(f)
login.keys()
```




    dict_keys(['client-id', 'api-key'])




```python
# Instantiate YelpAPI Variable
yelp_api = YelpAPI(login['api-key'], timeout_s=5.0)
```

### Define Search Terms and File Paths


```python
# set our API call parameters and filename before the first call
LOCATION = 'Baltimore, MD,21202'
TERM = 'Burgers'
```


```python
## Specify fodler for saving data
FOLDER = "Data/"
os.makedirs(FOLDER, exist_ok=True)

# Specifying JSON_FILE filename (can include a folder)
JSON_FILE = f"{FOLDER}{TERM}-{LOCATION.split(',')[0]}.json"
JSON_FILE
```




    'Data/Burgers-Baltimore.json'



### Check if Json File exists and Create it if it doesn't


```python
## Check if JSON_FILE exists
file_exists = os.path.isfile(JSON_FILE)

## If it does not exist: 
if file_exists == False:
    
    ## CREATE ANY NEEDED FOLDERS
    # Get the Folder Name only
    folder = os.path.dirname(JSON_FILE)
    
    ## If JSON_FILE included a folder:
    if len(folder)>0:
        # create the folder
        os.makedirs(folder,exist_ok=True)
        
        
    ## INFORM USER AND SAVE EMPTY LIST
    print(f"[i] {JSON_FILE} not found. Saving empty list to file.")
    
    
    ## save the first page of results
    with open(JSON_FILE,'w') as f:
        json.dump([],f)  
        
## If it exists, inform user
else:
    print(f"[i] {JSON_FILE} already exists.")
```

    [i] Data/Burgers-Baltimore.json not found. Saving empty list to file.


### Load JSON FIle and account for previous results


```python
## Load previous results and use len of results for offset
with open(JSON_FILE,'r') as f:
    previous_results = json.load(f)
    
## set offset based on previous results
n_results = len(previous_results)
print(f'- {n_results} previous results found.')
```

    - 0 previous results found.


### Make the first API call to get the first page of data

- We will use this first result to check:
    - how many total results there are?
    - Where is the actual data we want to save?
    - how many results do we get at a time?



```python
# use our yelp_api variable's search_query method to perform our API call
results = yelp_api.search_query(location=LOCATION,
                                term=TERM,
                               offset=n_results+1)
results.keys()
```




    dict_keys(['businesses', 'total', 'region'])




```python
## How many results total?
total_results = results['total']
total_results
```




    527



- Where is the actual data we want to save?


```python
results['businesses']
```




    [{'id': 'AQ56plNP56TIk3JswtoiMA',
      'alias': 'abbey-burger-bistro-baltimore-4',
      'name': 'Abbey Burger Bistro',
      'image_url': 'https://s3-media4.fl.yelpcdn.com/bphoto/_ZZ4nIpyKbeP_5xK5Zf6tQ/o.jpg',
      'is_closed': False,
      'url': 'https://www.yelp.com/biz/abbey-burger-bistro-baltimore-4?adjust_creative=j8tXJtmEvaNIK8mAYJ_C5A&utm_campaign=yelp_api_v3&utm_medium=api_v3_business_search&utm_source=j8tXJtmEvaNIK8mAYJ_C5A',
      'review_count': 916,
      'categories': [{'alias': 'sportsbars', 'title': 'Sports Bars'},
       {'alias': 'tradamerican', 'title': 'American (Traditional)'}],
      'rating': 4.0,
      'coordinates': {'latitude': 39.2771835822848,
       'longitude': -76.6129885794452},
      'transactions': ['delivery', 'pickup'],
      'price': '$$',
      'location': {'address1': '1041 Marshall St',
       'address2': '',
       'address3': '',
       'city': 'Baltimore',
       'zip_code': '21230',
       'country': 'US',
       'state': 'MD',
       'display_address': ['1041 Marshall St', 'Baltimore, MD 21230']},
      'phone': '+14434539698',
      'display_phone': '(443) 453-9698',
      'distance': 2161.212353352811},
     {'id': '3j6GTI0V2Jshw8e1c3uFoA',
      'alias': 'the-urban-burger-bar-baltimore',
      'name': 'The Urban Burger Bar',
      'image_url': 'https://s3-media4.fl.yelpcdn.com/bphoto/1zNJ5cfmhUAetoEkp53Rcg/o.jpg',
      'is_closed': False,
      'url': 'https://www.yelp.com/biz/the-urban-burger-bar-baltimore?adjust_creative=j8tXJtmEvaNIK8mAYJ_C5A&utm_campaign=yelp_api_v3&utm_medium=api_v3_business_search&utm_source=j8tXJtmEvaNIK8mAYJ_C5A',
      'review_count': 35,
      'categories': [{'alias': 'burgers', 'title': 'Burgers'}],
      'rating': 4.5,
      'coordinates': {'latitude': 39.32696, 'longitude': -76.63753},
      'transactions': ['delivery', 'pickup'],
      'price': '$$',
      'location': {'address1': '3300 Clipper Mill Rd',
       'address2': '',
       'address3': None,
       'city': 'Baltimore',
       'zip_code': '21211',
       'country': 'US',
       'state': 'MD',
       'display_address': ['3300 Clipper Mill Rd', 'Baltimore, MD 21211']},
      'phone': '',
      'display_phone': '',
      'distance': 4242.702639113149},
     {'id': 'QHVUhI8JBAcqGj1JufDcHw',
      'alias': 'wiley-gunters-baltimore',
      'name': 'Wiley Gunters',
      'image_url': 'https://s3-media3.fl.yelpcdn.com/bphoto/Lc-4k9vMnFUDY3R4fUet-A/o.jpg',
      'is_closed': False,
      'url': 'https://www.yelp.com/biz/wiley-gunters-baltimore?adjust_creative=j8tXJtmEvaNIK8mAYJ_C5A&utm_campaign=yelp_api_v3&utm_medium=api_v3_business_search&utm_source=j8tXJtmEvaNIK8mAYJ_C5A',
      'review_count': 136,
      'categories': [{'alias': 'burgers', 'title': 'Burgers'},
       {'alias': 'sportsbars', 'title': 'Sports Bars'},
       {'alias': 'beerbar', 'title': 'Beer Bar'}],
      'rating': 4.5,
      'coordinates': {'latitude': 39.27174, 'longitude': -76.60223},
      'transactions': ['delivery', 'pickup'],
      'price': '$$',
      'location': {'address1': '823 E Fort Ave',
       'address2': '',
       'address3': '',
       'city': 'Baltimore',
       'zip_code': '21230',
       'country': 'US',
       'state': 'MD',
       'display_address': ['823 E Fort Ave', 'Baltimore, MD 21230']},
      'phone': '+14106373699',
      'display_phone': '(410) 637-3699',
      'distance': 2754.9954665346213},
     {'id': 'DnhKT0A9OIgFdIhzNryJBw',
      'alias': 'abbey-burger-bistro-baltimore-6',
      'name': 'Abbey Burger Bistro',
      'image_url': 'https://s3-media3.fl.yelpcdn.com/bphoto/oq4-Iymnf8mgQ2WkYH4_ig/o.jpg',
      'is_closed': False,
      'url': 'https://www.yelp.com/biz/abbey-burger-bistro-baltimore-6?adjust_creative=j8tXJtmEvaNIK8mAYJ_C5A&utm_campaign=yelp_api_v3&utm_medium=api_v3_business_search&utm_source=j8tXJtmEvaNIK8mAYJ_C5A',
      'review_count': 268,
      'categories': [{'alias': 'tradamerican', 'title': 'American (Traditional)'},
       {'alias': 'burgers', 'title': 'Burgers'}],
      'rating': 4.0,
      'coordinates': {'latitude': 39.2822776123555,
       'longitude': -76.5927489101887},
      'transactions': ['delivery', 'pickup'],
      'price': '$$',
      'location': {'address1': '811 S Broadway',
       'address2': '',
       'address3': '',
       'city': 'Baltimore',
       'zip_code': '21231',
       'country': 'US',
       'state': 'MD',
       'display_address': ['811 S Broadway', 'Baltimore, MD 21231']},
      'phone': '+14105221428',
      'display_phone': '(410) 522-1428',
      'distance': 2009.2757033273615},
     {'id': 'jvKhto6__tCGfs2N27oWdg',
      'alias': 'the-outpost-american-tavern-baltimore-2',
      'name': 'The Outpost American Tavern',
      'image_url': 'https://s3-media2.fl.yelpcdn.com/bphoto/4bN4vAWZCpGJjO_Axo9QGg/o.jpg',
      'is_closed': False,
      'url': 'https://www.yelp.com/biz/the-outpost-american-tavern-baltimore-2?adjust_creative=j8tXJtmEvaNIK8mAYJ_C5A&utm_campaign=yelp_api_v3&utm_medium=api_v3_business_search&utm_source=j8tXJtmEvaNIK8mAYJ_C5A',
      'review_count': 184,
      'categories': [{'alias': 'bars', 'title': 'Bars'},
       {'alias': 'tradamerican', 'title': 'American (Traditional)'},
       {'alias': 'sandwiches', 'title': 'Sandwiches'}],
      'rating': 4.5,
      'coordinates': {'latitude': 39.2774761241692,
       'longitude': -76.6091799999978},
      'transactions': ['delivery', 'pickup'],
      'price': '$$',
      'location': {'address1': '1032 Riverside Ave',
       'address2': '',
       'address3': None,
       'city': 'Baltimore',
       'zip_code': '21230',
       'country': 'US',
       'state': 'MD',
       'display_address': ['1032 Riverside Ave', 'Baltimore, MD 21230']},
      'phone': '+14433889113',
      'display_phone': '(443) 388-9113',
      'distance': 2083.6576473630867},
     {'id': 'LkCSwCV-WX2eCnv5RUM3Kw',
      'alias': '311-west-madison-ave-craft-beer-and-wine-restaurant-baltimore',
      'name': '311 West Madison Ave Craft Beer & Wine Restaurant',
      'image_url': 'https://s3-media1.fl.yelpcdn.com/bphoto/g7er6AzyQ0_oF6FoFTo0MA/o.jpg',
      'is_closed': False,
      'url': 'https://www.yelp.com/biz/311-west-madison-ave-craft-beer-and-wine-restaurant-baltimore?adjust_creative=j8tXJtmEvaNIK8mAYJ_C5A&utm_campaign=yelp_api_v3&utm_medium=api_v3_business_search&utm_source=j8tXJtmEvaNIK8mAYJ_C5A',
      'review_count': 6,
      'categories': [{'alias': 'bars', 'title': 'Bars'},
       {'alias': 'burgers', 'title': 'Burgers'},
       {'alias': 'newamerican', 'title': 'American (New)'}],
      'rating': 4.0,
      'coordinates': {'latitude': 39.298216, 'longitude': -76.620656},
      'transactions': ['delivery', 'pickup'],
      'location': {'address1': '311 W Madison St',
       'address2': None,
       'address3': '',
       'city': 'Baltimore',
       'zip_code': '21201',
       'country': 'US',
       'state': 'MD',
       'display_address': ['311 W Madison St', 'Baltimore, MD 21201']},
      'phone': '+14439389109',
      'display_phone': '(443) 938-9109',
      'distance': 1141.1605104199282},
     {'id': '5CM8nuqd0od68GqQ-UGicQ',
      'alias': 'cookhouse-baltimore-2',
      'name': 'CookHouse',
      'image_url': 'https://s3-media2.fl.yelpcdn.com/bphoto/N-2JWjDaDqu_BKjGeXJz7Q/o.jpg',
      'is_closed': False,
      'url': 'https://www.yelp.com/biz/cookhouse-baltimore-2?adjust_creative=j8tXJtmEvaNIK8mAYJ_C5A&utm_campaign=yelp_api_v3&utm_medium=api_v3_business_search&utm_source=j8tXJtmEvaNIK8mAYJ_C5A',
      'review_count': 20,
      'categories': [{'alias': 'newamerican', 'title': 'American (New)'},
       {'alias': 'cocktailbars', 'title': 'Cocktail Bars'}],
      'rating': 5.0,
      'coordinates': {'latitude': 39.30715667, 'longitude': -76.62584833},
      'transactions': ['delivery', 'pickup'],
      'location': {'address1': '1501 Bolton St',
       'address2': None,
       'address3': '',
       'city': 'Baltimore',
       'zip_code': '21217',
       'country': 'US',
       'state': 'MD',
       'display_address': ['1501 Bolton St', 'Baltimore, MD 21217']},
      'phone': '+14102259964',
      'display_phone': '(410) 225-9964',
      'distance': 1990.07895603796},
     {'id': 'GOvVNYe2DwjGzwdVJIKw1Q',
      'alias': 'wicked-sisters-tavern-baltimore',
      'name': 'Wicked Sisters Tavern',
      'image_url': 'https://s3-media2.fl.yelpcdn.com/bphoto/XdSbMUXtU7HvY8ZgPkD8fQ/o.jpg',
      'is_closed': False,
      'url': 'https://www.yelp.com/biz/wicked-sisters-tavern-baltimore?adjust_creative=j8tXJtmEvaNIK8mAYJ_C5A&utm_campaign=yelp_api_v3&utm_medium=api_v3_business_search&utm_source=j8tXJtmEvaNIK8mAYJ_C5A',
      'review_count': 579,
      'categories': [{'alias': 'tradamerican', 'title': 'American (Traditional)'}],
      'rating': 4.0,
      'coordinates': {'latitude': 39.3350187678809,
       'longitude': -76.6363579077745},
      'transactions': ['delivery', 'pickup'],
      'price': '$$',
      'location': {'address1': '3845 Falls Rd',
       'address2': '',
       'address3': None,
       'city': 'Baltimore',
       'zip_code': '21211',
       'country': 'US',
       'state': 'MD',
       'display_address': ['3845 Falls Rd', 'Baltimore, MD 21211']},
      'phone': '+14108780884',
      'display_phone': '(410) 878-0884',
      'distance': 4974.443761753821},
     {'id': 'L_cDwPh8kuxkEP-VXjJF-A',
      'alias': 'shake-shack-baltimore',
      'name': 'Shake Shack',
      'image_url': 'https://s3-media2.fl.yelpcdn.com/bphoto/cORtQVNtWDwWHR2QgEreyw/o.jpg',
      'is_closed': False,
      'url': 'https://www.yelp.com/biz/shake-shack-baltimore?adjust_creative=j8tXJtmEvaNIK8mAYJ_C5A&utm_campaign=yelp_api_v3&utm_medium=api_v3_business_search&utm_source=j8tXJtmEvaNIK8mAYJ_C5A',
      'review_count': 520,
      'categories': [{'alias': 'newamerican', 'title': 'American (New)'},
       {'alias': 'burgers', 'title': 'Burgers'},
       {'alias': 'icecream', 'title': 'Ice Cream & Frozen Yogurt'}],
      'rating': 3.5,
      'coordinates': {'latitude': 39.2871081, 'longitude': -76.6092275},
      'transactions': ['delivery', 'pickup'],
      'price': '$$',
      'location': {'address1': '400 Pratt St',
       'address2': '',
       'address3': '',
       'city': 'Baltimore',
       'zip_code': '21202',
       'country': 'US',
       'state': 'MD',
       'display_address': ['400 Pratt St', 'Baltimore, MD 21202']},
      'phone': '+14439733629',
      'display_phone': '(443) 973-3629',
      'distance': 1017.4666413623578},
     {'id': '8xyl44uoylj8cxPnH741aQ',
      'alias': 'noisy-burger-baltimore',
      'name': 'Noisy Burger',
      'image_url': 'https://s3-media3.fl.yelpcdn.com/bphoto/Gb_hF0VJF30Ntl_OxJX4vA/o.jpg',
      'is_closed': False,
      'url': 'https://www.yelp.com/biz/noisy-burger-baltimore?adjust_creative=j8tXJtmEvaNIK8mAYJ_C5A&utm_campaign=yelp_api_v3&utm_medium=api_v3_business_search&utm_source=j8tXJtmEvaNIK8mAYJ_C5A',
      'review_count': 17,
      'categories': [{'alias': 'burgers', 'title': 'Burgers'}],
      'rating': 4.0,
      'coordinates': {'latitude': 39.3217276, 'longitude': -76.6222487576701},
      'transactions': ['delivery', 'pickup'],
      'location': {'address1': '301 W 29th St',
       'address2': None,
       'address3': '',
       'city': 'Baltimore',
       'zip_code': '21211',
       'country': 'US',
       'state': 'MD',
       'display_address': ['301 W 29th St', 'Baltimore, MD 21211']},
      'phone': '+14436811906',
      'display_phone': '(443) 681-1906',
      'distance': 3105.663543006425},
     {'id': 'lx6ZQjgZDvL0Rz9LmXtTKg',
      'alias': 'charmed-baltimore',
      'name': 'Charmed.',
      'image_url': 'https://s3-media2.fl.yelpcdn.com/bphoto/e9AV7X8oamvpklsugp8jCQ/o.jpg',
      'is_closed': False,
      'url': 'https://www.yelp.com/biz/charmed-baltimore?adjust_creative=j8tXJtmEvaNIK8mAYJ_C5A&utm_campaign=yelp_api_v3&utm_medium=api_v3_business_search&utm_source=j8tXJtmEvaNIK8mAYJ_C5A',
      'review_count': 72,
      'categories': [{'alias': 'newamerican', 'title': 'American (New)'},
       {'alias': 'cafes', 'title': 'Cafes'}],
      'rating': 5.0,
      'coordinates': {'latitude': 39.29951, 'longitude': -76.61334},
      'transactions': ['delivery', 'pickup'],
      'location': {'address1': '824 N Calvert St',
       'address2': None,
       'address3': '',
       'city': 'Baltimore',
       'zip_code': '21202',
       'country': 'US',
       'state': 'MD',
       'display_address': ['824 N Calvert St', 'Baltimore, MD 21202']},
      'phone': '+14438352803',
      'display_phone': '(443) 835-2803',
      'distance': 609.3291173464794},
     {'id': 'S3keMV3sQjujV-oA-Ymxog',
      'alias': 'between-2-buns-baltimore-2',
      'name': 'Between 2 Buns',
      'image_url': 'https://s3-media1.fl.yelpcdn.com/bphoto/wfTqwH5shtXJNIMBry-BBQ/o.jpg',
      'is_closed': False,
      'url': 'https://www.yelp.com/biz/between-2-buns-baltimore-2?adjust_creative=j8tXJtmEvaNIK8mAYJ_C5A&utm_campaign=yelp_api_v3&utm_medium=api_v3_business_search&utm_source=j8tXJtmEvaNIK8mAYJ_C5A',
      'review_count': 163,
      'categories': [{'alias': 'burgers', 'title': 'Burgers'},
       {'alias': 'hotdogs', 'title': 'Fast Food'}],
      'rating': 3.5,
      'coordinates': {'latitude': 39.29588539090596,
       'longitude': -76.61848222083206},
      'transactions': ['delivery'],
      'price': '$$',
      'location': {'address1': '520 Park Ave',
       'address2': '',
       'address3': 'Mount Vernon Marketplace',
       'city': 'Baltimore',
       'zip_code': '21201',
       'country': 'US',
       'state': 'MD',
       'display_address': ['520 Park Ave',
        'Mount Vernon Marketplace',
        'Baltimore, MD 21201']},
      'phone': '+16673033273',
      'display_phone': '(667) 303-3273',
      'distance': 931.3243810613322},
     {'id': 'yITYeW3JZS557DIXj9Ivhg',
      'alias': 'mick-o-sheas-baltimore',
      'name': "Mick O'Shea's",
      'image_url': 'https://s3-media1.fl.yelpcdn.com/bphoto/zPk7LsERGz8moGdq6G7k2Q/o.jpg',
      'is_closed': False,
      'url': 'https://www.yelp.com/biz/mick-o-sheas-baltimore?adjust_creative=j8tXJtmEvaNIK8mAYJ_C5A&utm_campaign=yelp_api_v3&utm_medium=api_v3_business_search&utm_source=j8tXJtmEvaNIK8mAYJ_C5A',
      'review_count': 291,
      'categories': [{'alias': 'pubs', 'title': 'Pubs'},
       {'alias': 'irish', 'title': 'Irish'},
       {'alias': 'burgers', 'title': 'Burgers'}],
      'rating': 4.0,
      'coordinates': {'latitude': 39.293462, 'longitude': -76.615586},
      'transactions': ['delivery', 'pickup'],
      'price': '$$',
      'location': {'address1': '328 N Charles St',
       'address2': '',
       'address3': '',
       'city': 'Baltimore',
       'zip_code': '21201',
       'country': 'US',
       'state': 'MD',
       'display_address': ['328 N Charles St', 'Baltimore, MD 21201']},
      'phone': '+14105397504',
      'display_phone': '(410) 539-7504',
      'distance': 745.4681839527719},
     {'id': '6PLtSs9Bi4QDSNBM_u4fNg',
      'alias': 'the-brewers-art-baltimore',
      'name': "The Brewer's Art",
      'image_url': 'https://s3-media1.fl.yelpcdn.com/bphoto/2pkJFgRcf3G_dYopNnaGKA/o.jpg',
      'is_closed': False,
      'url': 'https://www.yelp.com/biz/the-brewers-art-baltimore?adjust_creative=j8tXJtmEvaNIK8mAYJ_C5A&utm_campaign=yelp_api_v3&utm_medium=api_v3_business_search&utm_source=j8tXJtmEvaNIK8mAYJ_C5A',
      'review_count': 1041,
      'categories': [{'alias': 'breweries', 'title': 'Breweries'},
       {'alias': 'modern_european', 'title': 'Modern European'},
       {'alias': 'belgian', 'title': 'Belgian'}],
      'rating': 4.0,
      'coordinates': {'latitude': 39.302825295486, 'longitude': -76.616179918669},
      'transactions': ['delivery', 'pickup'],
      'price': '$$',
      'location': {'address1': '1106 N Charles St',
       'address2': '',
       'address3': '',
       'city': 'Baltimore',
       'zip_code': '21201',
       'country': 'US',
       'state': 'MD',
       'display_address': ['1106 N Charles St', 'Baltimore, MD 21201']},
      'phone': '+14105476925',
      'display_phone': '(410) 547-6925',
      'distance': 1040.6716769983216},
     {'id': 'WRz58h4VB9nadcyar4LGvQ',
      'alias': 'wet-city-baltimore',
      'name': 'Wet City',
      'image_url': 'https://s3-media4.fl.yelpcdn.com/bphoto/8L30rMvLw1oSa9l0bT4DEQ/o.jpg',
      'is_closed': False,
      'url': 'https://www.yelp.com/biz/wet-city-baltimore?adjust_creative=j8tXJtmEvaNIK8mAYJ_C5A&utm_campaign=yelp_api_v3&utm_medium=api_v3_business_search&utm_source=j8tXJtmEvaNIK8mAYJ_C5A',
      'review_count': 159,
      'categories': [{'alias': 'beerbar', 'title': 'Beer Bar'},
       {'alias': 'pubs', 'title': 'Pubs'},
       {'alias': 'gastropubs', 'title': 'Gastropubs'}],
      'rating': 4.0,
      'coordinates': {'latitude': 39.3016090393066,
       'longitude': -76.6189498901367},
      'transactions': ['delivery'],
      'price': '$$',
      'location': {'address1': '223 W Chase St',
       'address2': '',
       'address3': '',
       'city': 'Baltimore',
       'zip_code': '21201',
       'country': 'US',
       'state': 'MD',
       'display_address': ['223 W Chase St', 'Baltimore, MD 21201']},
      'phone': '+14438736699',
      'display_phone': '(443) 873-6699',
      'distance': 1143.8733798625783},
     {'id': 'O7EO2AHxQXRHL1fqodpg7Q',
      'alias': 'daily-special-authentic-mexican-grill-baltimore-3',
      'name': 'Daily Special authentic mexican grill',
      'image_url': 'https://s3-media2.fl.yelpcdn.com/bphoto/76IbRS-FW9XCx03SCCcBnQ/o.jpg',
      'is_closed': False,
      'url': 'https://www.yelp.com/biz/daily-special-authentic-mexican-grill-baltimore-3?adjust_creative=j8tXJtmEvaNIK8mAYJ_C5A&utm_campaign=yelp_api_v3&utm_medium=api_v3_business_search&utm_source=j8tXJtmEvaNIK8mAYJ_C5A',
      'review_count': 25,
      'categories': [{'alias': 'burgers', 'title': 'Burgers'},
       {'alias': 'juicebars', 'title': 'Juice Bars & Smoothies'},
       {'alias': 'mexican', 'title': 'Mexican'}],
      'rating': 4.5,
      'coordinates': {'latitude': 39.29133, 'longitude': -76.61497},
      'transactions': ['delivery', 'pickup', 'restaurant_reservation'],
      'location': {'address1': '201 N Charles St',
       'address2': None,
       'address3': '',
       'city': 'Baltimore',
       'zip_code': '21201',
       'country': 'US',
       'state': 'MD',
       'display_address': ['201 N Charles St', 'Baltimore, MD 21201']},
      'phone': '+14106895603',
      'display_phone': '(410) 689-5603',
      'distance': 818.3375947996358},
     {'id': 'Ulw0CIk-H7ZMQXRDrLhU1w',
      'alias': 'southside-burger-bar-baltimore',
      'name': 'Southside Burger Bar',
      'image_url': 'https://s3-media4.fl.yelpcdn.com/bphoto/ly2VayOqzRkYrviC4MHoig/o.jpg',
      'is_closed': False,
      'url': 'https://www.yelp.com/biz/southside-burger-bar-baltimore?adjust_creative=j8tXJtmEvaNIK8mAYJ_C5A&utm_campaign=yelp_api_v3&utm_medium=api_v3_business_search&utm_source=j8tXJtmEvaNIK8mAYJ_C5A',
      'review_count': 8,
      'categories': [{'alias': 'burgers', 'title': 'Burgers'},
       {'alias': 'hotdogs', 'title': 'Fast Food'}],
      'rating': 4.0,
      'coordinates': {'latitude': 39.276819, 'longitude': -76.613262},
      'transactions': ['delivery', 'pickup'],
      'location': {'address1': '1065 S Charles St',
       'address2': '',
       'address3': '',
       'city': 'Baltimore',
       'zip_code': '21230',
       'country': 'US',
       'state': 'MD',
       'display_address': ['1065 S Charles St', 'Baltimore, MD 21230']},
      'phone': '+14433888321',
      'display_phone': '(443) 388-8321',
      'distance': 2205.639692238642},
     {'id': 'TL34QOmDmMtJmsabvPLErg',
      'alias': 'annabel-lee-tavern-baltimore',
      'name': 'Annabel Lee Tavern',
      'image_url': 'https://s3-media2.fl.yelpcdn.com/bphoto/dh6k0b51-GEggohxYvJc3A/o.jpg',
      'is_closed': False,
      'url': 'https://www.yelp.com/biz/annabel-lee-tavern-baltimore?adjust_creative=j8tXJtmEvaNIK8mAYJ_C5A&utm_campaign=yelp_api_v3&utm_medium=api_v3_business_search&utm_source=j8tXJtmEvaNIK8mAYJ_C5A',
      'review_count': 610,
      'categories': [{'alias': 'pubs', 'title': 'Pubs'},
       {'alias': 'newamerican', 'title': 'American (New)'}],
      'rating': 4.5,
      'coordinates': {'latitude': 39.28538, 'longitude': -76.56961},
      'transactions': ['delivery', 'pickup', 'restaurant_reservation'],
      'price': '$$',
      'location': {'address1': '601 S Clinton St',
       'address2': '',
       'address3': '',
       'city': 'Baltimore',
       'zip_code': '21224',
       'country': 'US',
       'state': 'MD',
       'display_address': ['601 S Clinton St', 'Baltimore, MD 21224']},
      'phone': '+14105222929',
      'display_phone': '(410) 522-2929',
      'distance': 3485.0458208677787},
     {'id': 'UplfpgvI5BfuDjtolDrmsA',
      'alias': 'blackwall-hitch-baltimore',
      'name': 'Blackwall Hitch',
      'image_url': 'https://s3-media3.fl.yelpcdn.com/bphoto/YoB4qYhyi-MM3pz3AxCiKA/o.jpg',
      'is_closed': False,
      'url': 'https://www.yelp.com/biz/blackwall-hitch-baltimore?adjust_creative=j8tXJtmEvaNIK8mAYJ_C5A&utm_campaign=yelp_api_v3&utm_medium=api_v3_business_search&utm_source=j8tXJtmEvaNIK8mAYJ_C5A',
      'review_count': 545,
      'categories': [{'alias': 'newamerican', 'title': 'American (New)'},
       {'alias': 'seafood', 'title': 'Seafood'},
       {'alias': 'cocktailbars', 'title': 'Cocktail Bars'}],
      'rating': 4.0,
      'coordinates': {'latitude': 39.2871515492791,
       'longitude': -76.60605301975444},
      'transactions': [],
      'price': '$$',
      'location': {'address1': '700 E Pratt St',
       'address2': None,
       'address3': '',
       'city': 'Baltimore',
       'zip_code': '21202',
       'country': 'US',
       'state': 'MD',
       'display_address': ['700 E Pratt St', 'Baltimore, MD 21202']},
      'phone': '+14437597176',
      'display_phone': '(443) 759-7176',
      'distance': 1013.2755773246936},
     {'id': 't0EA_fXewpQWfzWMrpNjIw',
      'alias': 'peters-pour-house-baltimore',
      'name': "Peter's Pour House",
      'image_url': 'https://s3-media3.fl.yelpcdn.com/bphoto/A5kRmG6uiu3L1vStpcRK-g/o.jpg',
      'is_closed': False,
      'url': 'https://www.yelp.com/biz/peters-pour-house-baltimore?adjust_creative=j8tXJtmEvaNIK8mAYJ_C5A&utm_campaign=yelp_api_v3&utm_medium=api_v3_business_search&utm_source=j8tXJtmEvaNIK8mAYJ_C5A',
      'review_count': 231,
      'categories': [{'alias': 'tradamerican', 'title': 'American (Traditional)'},
       {'alias': 'pubs', 'title': 'Pubs'}],
      'rating': 4.0,
      'coordinates': {'latitude': 39.28841, 'longitude': -76.61299},
      'transactions': ['delivery', 'pickup'],
      'price': '$$',
      'location': {'address1': '111 Mercer St',
       'address2': None,
       'address3': '',
       'city': 'Baltimore',
       'zip_code': '21202',
       'country': 'US',
       'state': 'MD',
       'display_address': ['111 Mercer St', 'Baltimore, MD 21202']},
      'phone': '+14105395818',
      'display_phone': '(410) 539-5818',
      'distance': 978.5679088399228}]




```python
## How many did we get the details for?
results_per_page = len(results['businesses'])
results_per_page
```




    20



- Calculate how many pages of results needed to cover the total_results


```python
# Import additional packages for controlling our loop
import time, math
# Use math.ceil to round up for the total number of pages of results.
n_pages = math.ceil((results['total']-n_results)/ results_per_page)
n_pages
```




    27




```python
for i in tqdm_notebook( range(1,n_pages+1)):
    ## The block of code we want to TRY to run
    try:
        
        time.sleep(.2)
        
        ## Read in results in progress file and check the length
        with open(JSON_FILE, 'r') as f:
            previous_results = json.load(f)
            
        ## save number of results for to use as offset
        n_results = len(previous_results)
        
        
        ## use n_results as the OFFSET 
        results = yelp_api.search_query(location=LOCATION,
                                        term=TERM, 
                                        offset=n_results+1)

        ## append new results and save to file
        previous_results.extend(results['businesses'])

        with open(JSON_FILE,'w') as f:
            json.dump(previous_results,f)
            
    ## What to do if we get an error/exception.
    except Exception as e: # saving the error message so we can print it.
        print('[!] ERROR: ',e)
```


      0%|          | 0/27 [00:00<?, ?it/s]


## Open the Final JSON File with Pandas


```python
df = pd.read_json(JSON_FILE)
df
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
      <th>id</th>
      <th>alias</th>
      <th>name</th>
      <th>image_url</th>
      <th>is_closed</th>
      <th>url</th>
      <th>review_count</th>
      <th>categories</th>
      <th>rating</th>
      <th>coordinates</th>
      <th>transactions</th>
      <th>price</th>
      <th>location</th>
      <th>phone</th>
      <th>display_phone</th>
      <th>distance</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>AQ56plNP56TIk3JswtoiMA</td>
      <td>abbey-burger-bistro-baltimore-4</td>
      <td>Abbey Burger Bistro</td>
      <td>https://s3-media4.fl.yelpcdn.com/bphoto/_ZZ4nI...</td>
      <td>False</td>
      <td>https://www.yelp.com/biz/abbey-burger-bistro-b...</td>
      <td>916</td>
      <td>[{'alias': 'sportsbars', 'title': 'Sports Bars...</td>
      <td>4.0</td>
      <td>{'latitude': 39.2771835822848, 'longitude': -7...</td>
      <td>[delivery, pickup]</td>
      <td>$$</td>
      <td>{'address1': '1041 Marshall St', 'address2': '...</td>
      <td>+14434539698</td>
      <td>(443) 453-9698</td>
      <td>2161.212353</td>
    </tr>
    <tr>
      <th>1</th>
      <td>3j6GTI0V2Jshw8e1c3uFoA</td>
      <td>the-urban-burger-bar-baltimore</td>
      <td>The Urban Burger Bar</td>
      <td>https://s3-media4.fl.yelpcdn.com/bphoto/1zNJ5c...</td>
      <td>False</td>
      <td>https://www.yelp.com/biz/the-urban-burger-bar-...</td>
      <td>35</td>
      <td>[{'alias': 'burgers', 'title': 'Burgers'}]</td>
      <td>4.5</td>
      <td>{'latitude': 39.32696, 'longitude': -76.63753}</td>
      <td>[delivery, pickup]</td>
      <td>$$</td>
      <td>{'address1': '3300 Clipper Mill Rd', 'address2...</td>
      <td></td>
      <td></td>
      <td>4242.702639</td>
    </tr>
    <tr>
      <th>2</th>
      <td>QHVUhI8JBAcqGj1JufDcHw</td>
      <td>wiley-gunters-baltimore</td>
      <td>Wiley Gunters</td>
      <td>https://s3-media3.fl.yelpcdn.com/bphoto/Lc-4k9...</td>
      <td>False</td>
      <td>https://www.yelp.com/biz/wiley-gunters-baltimo...</td>
      <td>136</td>
      <td>[{'alias': 'burgers', 'title': 'Burgers'}, {'a...</td>
      <td>4.5</td>
      <td>{'latitude': 39.27174, 'longitude': -76.60223}</td>
      <td>[delivery, pickup]</td>
      <td>$$</td>
      <td>{'address1': '823 E Fort Ave', 'address2': '',...</td>
      <td>+14106373699</td>
      <td>(410) 637-3699</td>
      <td>2754.995467</td>
    </tr>
    <tr>
      <th>3</th>
      <td>DnhKT0A9OIgFdIhzNryJBw</td>
      <td>abbey-burger-bistro-baltimore-6</td>
      <td>Abbey Burger Bistro</td>
      <td>https://s3-media3.fl.yelpcdn.com/bphoto/oq4-Iy...</td>
      <td>False</td>
      <td>https://www.yelp.com/biz/abbey-burger-bistro-b...</td>
      <td>268</td>
      <td>[{'alias': 'tradamerican', 'title': 'American ...</td>
      <td>4.0</td>
      <td>{'latitude': 39.2822776123555, 'longitude': -7...</td>
      <td>[delivery, pickup]</td>
      <td>$$</td>
      <td>{'address1': '811 S Broadway', 'address2': '',...</td>
      <td>+14105221428</td>
      <td>(410) 522-1428</td>
      <td>2009.275703</td>
    </tr>
    <tr>
      <th>4</th>
      <td>jvKhto6__tCGfs2N27oWdg</td>
      <td>the-outpost-american-tavern-baltimore-2</td>
      <td>The Outpost American Tavern</td>
      <td>https://s3-media2.fl.yelpcdn.com/bphoto/4bN4vA...</td>
      <td>False</td>
      <td>https://www.yelp.com/biz/the-outpost-american-...</td>
      <td>184</td>
      <td>[{'alias': 'bars', 'title': 'Bars'}, {'alias':...</td>
      <td>4.5</td>
      <td>{'latitude': 39.2774761241692, 'longitude': -7...</td>
      <td>[delivery, pickup]</td>
      <td>$$</td>
      <td>{'address1': '1032 Riverside Ave', 'address2':...</td>
      <td>+14433889113</td>
      <td>(443) 388-9113</td>
      <td>2083.657647</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>521</th>
      <td>gXQL3Zmlv0aXzjhYPRt5tg</td>
      <td>mcdonalds-baltimore-57</td>
      <td>McDonald's</td>
      <td>https://s3-media2.fl.yelpcdn.com/bphoto/VAqcvo...</td>
      <td>False</td>
      <td>https://www.yelp.com/biz/mcdonalds-baltimore-5...</td>
      <td>6</td>
      <td>[{'alias': 'coffee', 'title': 'Coffee &amp; Tea'},...</td>
      <td>1.5</td>
      <td>{'latitude': 39.3093679758871, 'longitude': -7...</td>
      <td>[delivery]</td>
      <td>$</td>
      <td>{'address1': '4526 Erdman Ave', 'address2': No...</td>
      <td>+14104839723</td>
      <td>(410) 483-9723</td>
      <td>4193.333190</td>
    </tr>
    <tr>
      <th>522</th>
      <td>x6u_Xd5Zr6skcQsIUiUOng</td>
      <td>jazzys-baltimore</td>
      <td>Jazzy's</td>
      <td>https://s3-media1.fl.yelpcdn.com/bphoto/NcDv9-...</td>
      <td>False</td>
      <td>https://www.yelp.com/biz/jazzys-baltimore?adju...</td>
      <td>1</td>
      <td>[{'alias': 'burgers', 'title': 'Burgers'}, {'a...</td>
      <td>2.0</td>
      <td>{'latitude': 39.322087392211, 'longitude': -76...</td>
      <td>[delivery]</td>
      <td>NaN</td>
      <td>{'address1': '3320 Belair Rd', 'address2': '',...</td>
      <td>+14105636222</td>
      <td>(410) 563-6222</td>
      <td>4134.435594</td>
    </tr>
    <tr>
      <th>523</th>
      <td>rtlLrtUn35c9Y2eIcxHT8g</td>
      <td>kings-pizza-and-subs-baltimore-2</td>
      <td>King's Pizza &amp; Subs</td>
      <td>https://s3-media2.fl.yelpcdn.com/bphoto/9FEdeU...</td>
      <td>False</td>
      <td>https://www.yelp.com/biz/kings-pizza-and-subs-...</td>
      <td>75</td>
      <td>[{'alias': 'pizza', 'title': 'Pizza'}]</td>
      <td>2.5</td>
      <td>{'latitude': 39.33086, 'longitude': -76.63161}</td>
      <td>[pickup, delivery]</td>
      <td>$$</td>
      <td>{'address1': '907 W 36th St', 'address2': None...</td>
      <td>+14108893663</td>
      <td>(410) 889-3663</td>
      <td>4382.210159</td>
    </tr>
    <tr>
      <th>524</th>
      <td>sJOcdfLvAnnBy6Vo0ylANQ</td>
      <td>chris-seafood-baltimore</td>
      <td>Chris' Seafood</td>
      <td>https://s3-media1.fl.yelpcdn.com/bphoto/RpSPa-...</td>
      <td>False</td>
      <td>https://www.yelp.com/biz/chris-seafood-baltimo...</td>
      <td>54</td>
      <td>[{'alias': 'seafood', 'title': 'Seafood'}]</td>
      <td>3.5</td>
      <td>{'latitude': 39.2828178405762, 'longitude': -7...</td>
      <td>[pickup, delivery]</td>
      <td>$$</td>
      <td>{'address1': '801 S Montford Ave', 'address2':...</td>
      <td>+14106750117</td>
      <td>(410) 675-0117</td>
      <td>2628.043743</td>
    </tr>
    <tr>
      <th>525</th>
      <td>rMrTzM9EET0wszOng1eINA</td>
      <td>chelles-kitchen-baltimore</td>
      <td>Chelles kitchen</td>
      <td>https://s3-media1.fl.yelpcdn.com/bphoto/M4RMzz...</td>
      <td>False</td>
      <td>https://www.yelp.com/biz/chelles-kitchen-balti...</td>
      <td>11</td>
      <td>[{'alias': 'hotdogs', 'title': 'Fast Food'}, {...</td>
      <td>2.5</td>
      <td>{'latitude': 39.32332, 'longitude': -76.57185}</td>
      <td>[pickup, delivery]</td>
      <td>NaN</td>
      <td>{'address1': '3436 Belair Rd', 'address2': '',...</td>
      <td>+14103014414</td>
      <td>(410) 301-4414</td>
      <td>4311.772603</td>
    </tr>
  </tbody>
</table>
<p>526 rows × 16 columns</p>
</div>




```python
## convert the filename to a .csv.gz
csv_file = JSON_FILE.replace('.json','.csv.gz')
csv_file
```




    'Data/Burgers-Baltimore.csv.gz'




```python
## Save it as a compressed csv (to save space)
df.to_csv(csv_file, compression='gzip', index=False)
```

## Bonus: compare filesize with os module's `os.path.getsize`


```python
size_json = os.path.getsize(JSON_FILE)
size_csv_gz = os.path.getsize(JSON_FILE.replace('.json','.csv.gz'))

print(f'JSON FILE: {size_json:,} Bytes')
print(f'CSV.GZ FILE: {size_csv_gz:,} Bytes')

print(f'the csv.gz is {size_json/size_csv_gz} times smaller!')
```

    JSON FILE: 516,403 Bytes
    CSV.GZ FILE: 72,762 Bytes
    the csv.gz is 7.09715235974822 times smaller!


## Next Class: Processing the Results and Mapping 
