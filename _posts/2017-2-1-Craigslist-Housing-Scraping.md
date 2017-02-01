---
layout: default
title: "Craigslist Housing Scraping Experiment"
date: 2017-2-1
---

# Craigslist Apt Scraper
    The goal of this project is to build a quick tool that will automatically scrape housing data from Craigslist with a definted set of features.  This tool will have two purposes:
    1). Automatically update the user when new apartments matching the filters are found
    2). Add to a database of all craigslist housing to analyze longitudinal trends


```python
import pandas as pd
import os
import time
from craigslist import CraigslistHousing
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, Integer, String, DateTime, Float, Boolean
from sqlalchemy.orm import sessionmaker


```


```python
cl = CraigslistHousing(site='seattle', area='see', category='apa', filters={'max_price': 3000, 'min_price': 800)

#get_results generator that filters to add coordinates and limits the results the the newest 50
gen = cl.get_results(sort_by='newest', geotagged=True, limit=50)
t = []
while True:
    try:
        result = next(gen)
    except StopIteration:
        break
    except Exception:
        continue
    t.append(result)
df = pd.DataFrame(t) 


```

Right now I have a quick scraper built up from the [python-craigslist](https://github.com/juliomalegria/python-craigslist) package. This returns the *results* generator which I then loop through and print each individual result.  The results contain:

     datetime        time at which the listing was posted
     has_image       whether there are images attached
     where           user-defined value for where the listing is
     geotag          geotagged coordinates
     has_map         whether there's a map associated with the image
     name            user-defined name of the posting
     price           price of the listing
     url             URL of the full listing
     id              craigslist defined ID for the listing

I can further limit my search using the various fiilters within the python-craigslist package(shown below)


```python
CraigslistHousing.show_filters()
```

    Base filters:
    * posted_today = True/False
    * search_titles = True/False
    * has_image = True/False
    * query = ...
    Section specific filters:
    * min_price = ...
    * max_ft2 = ...
    * laundry_in_unit = True/False
    * bathrooms = ...
    * min_ft2 = ...
    * max_price = ...
    * bedrooms = ...
    * cats_ok = True/False
    * dogs_ok = True/False
    * no_smoking = True/False
    * private_bath = True/False
    * zip_code = ...
    * private_room = True/False
    * search_distance = ...
    

For now, I want to stick with 1-bedroom apartments.  Unfortunately, Craigslist only filters based on 1+ bedrooms, so I'll have to do some cleaning later on to remove extra hits. 


```python
cl = CraigslistHousing(site='seattle', area='see', category='apa', filters={'max_price': 2000, 'min_price': 800, 'bedrooms' : 1})

#get_results generator that filters to add coordinates and limits the results the the newest 50
gen = cl.get_results(sort_by='newest', geotagged=True, limit=50)
t = []
while True:
    try:
        result = next(gen)
    except StopIteration:
        break
    except Exception:
        continue
    t.append(result)
df = pd.DataFrame(t) 
df.head()


```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>datetime</th>
      <th>geotag</th>
      <th>has_image</th>
      <th>has_map</th>
      <th>id</th>
      <th>name</th>
      <th>price</th>
      <th>url</th>
      <th>where</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2017-02-01 10:46</td>
      <td>(47.556376, -122.386932)</td>
      <td>True</td>
      <td>True</td>
      <td>5985135361</td>
      <td>1 Bedroom With Den and Amazing Water Views</td>
      <td>$1943</td>
      <td>http://seattle.craigslist.org/see/apa/59851353...</td>
      <td>5020 California Ave SW Seattle, WA</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2017-02-01 10:45</td>
      <td>(47.395329, -122.300863)</td>
      <td>True</td>
      <td>True</td>
      <td>5971957026</td>
      <td>2 Bedroom Apartment in Des Moines</td>
      <td>$1219</td>
      <td>http://seattle.craigslist.org/see/apa/59719570...</td>
      <td>Des Moines</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2017-02-01 10:42</td>
      <td>(47.615191, -122.31126)</td>
      <td>True</td>
      <td>True</td>
      <td>5985127894</td>
      <td>Huge One bedroom, Dining space, 6 months free ...</td>
      <td>$1950</td>
      <td>http://seattle.craigslist.org/see/apa/59851278...</td>
      <td>Capitol Hill</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2017-02-01 10:42</td>
      <td>(47.680426, -122.32406)</td>
      <td>True</td>
      <td>True</td>
      <td>5985127842</td>
      <td>Open 1 bedroom priced to lease today - 1 month...</td>
      <td>$1760</td>
      <td>http://seattle.craigslist.org/see/apa/59851278...</td>
      <td>Green Lake</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2017-02-01 10:42</td>
      <td>(47.685651, -122.365776)</td>
      <td>True</td>
      <td>True</td>
      <td>5985127286</td>
      <td>North Ballard, Hardwood, Bright, Garage!</td>
      <td>$1700</td>
      <td>http://seattle.craigslist.org/see/apa/59851272...</td>
      <td>Ballard, Greenwood</td>
    </tr>
  </tbody>
</table>
</div>



Now that I have this data, there are 2 seperate pipelines.  
    1. Add to previously scraped craigslist data for future analysis
    2. Filter this data with user-defined filters


# Pathway #1

For now, I'm just going to save to a .csv file.  In the future, I'd like to use a SQL Database but for now the data is small enough to be managable with just a .csv.


```python
cl = CraigslistHousing(site='seattle', area='see', category='apa', filters={'max_price': 2000, 'min_price': 800, 'bedrooms': 1})

#get_results generator that filters to add coordinates and limits the results the the newest 2000
gen = cl.get_results(sort_by='newest', geotagged=True, limit=2000)
t = []
while True:
    try:
        result = next(gen)
    except StopIteration:
        break
    except Exception:
        continue
    t.append(result)
df = pd.DataFrame(t) 
date = time.strftime("%m%d%Y")
#Append to old file if I've already ran that day, otherwise create new csv
if os.path.isfile(date+'.csv'):
    df.to_csv(date+'.csv',mode = 'a',header=False)
else:
    df.to_csv(date+'.csv')
```

Now I have the data in a daily .csv file.  I want to play around with creating a pipeline for data analysis.


```python

```




    str






```python

```

    None
    


```python

```


```python

```

    <sqlalchemy.orm.session.Session object at 0x000000568FCB6198>
    


```python

```
