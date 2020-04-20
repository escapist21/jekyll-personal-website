---
title: Plotting crowdsourced data with Google Sheets and Folium
description: How to create web maps with crowdsourced data using Flask and Google
  sheets
tags:
- python
- webmap
- sheets
- crowdsourced
- flask
- geopandas
layout: single
---

In this codethrough we will explore creating web maps in Python using the _Flask_ web framework and the excellent _folium_ library. To make it more interesting we will retrieve the data to be plotted from Google Sheets using the _gspread_ library.

## Setting up the virtual environment and installing necessary packages
* create a project folder : `mkdir flask_webmap`
* `cd` into the folder and set up a virtual environment
*  we will be using the virtualenv as our choice of virtual environment tool. Create an environment named venv by running `virtualenv venv` inside your project directory
*  activate the environment by running `source venv/bin/activate`
*  to install the required modules run `python -m pip install -U Flask folium gspread`

## Registering our app with Google to use the Google Sheets API
To access data from Google sheets we need to register our app with the Google Sheets API. To do so follow these steps

* Open this [Google console link ](https://console.developers.google.com/) from a web browser
* From the Google console dasboard click on New Project link found at the top bar
* Give your project a name and click on Create
* Click on the **Enable APIs and services** 
* From the new page that opens listing the various APIs available, select the Google Sheets API and enable it
* Once the API is enabled click on the **Create credentials** alert that appears
* Fill in the necessary details: Select **Web server** for the question **Where will you be calling the API from?**
* In the next section enter any name for the **Service account**. The role option could be left blank but select **JSON** as your key type and click on continue
* A new json file with your credentials will get downloaded
* Copy the json file to your project directory

## Setting up a minimal Flask app
* Create a new Python file by running `touch app.py`
* open the python file in your editor of choice ans create a minimal Flask app

```python
from flask import Flask


# instantiate a new Flask app
app = Flask(__name__)


@app.route('/')
def hello_world():
    return "Hellow World"
		

if __name__ == "__main__":
    app.run()
```
* in your terminal export the Flask app environment variable by running `export FLASK_APP=app`
* The default environment set to production. Change this to development by running `FLASK_ENV=development flask run`
* Run `flask run` to start the app
* The new app should now be available at *localhost:5000/*
* The page should print the "Hello World" message

## Creating a Google sheet with seed data
* Create a new google sheet
* For the purpose of this exercise a subset of the earthquake location data by USGS was saved in sheet. To view the original data visit this [link](https://www.kaggle.com/usgs/earthquake-database#database.csv). To download the google sheet click on this [link](https://docs.google.com/spreadsheets/d/1x2bVUzH5cnzcDJXnleGteT5zCEytczDAghcRe0Z9WuU/edit?usp=sharing)


## Accessing the Google sheet data
To access this data using the gspread library
* create new  python file: `touch read_sheet.py`
* And copy the following code

```python
import gspread
from oauth2client.service_account import ServiceAccountCredentials

# defining globals
scope = ['https://spreadsheets.google.com/feeds', 'https://www.googleapis.com/auth/drive']
creds = ServiceAccountCredentials.from_json_keyfile_name('NAME-OF-JSON-FILE-WITH-CREDENTIALS', scope)
client = gspread.authorize(creds)

# opening the google sheet
sheet = client.open_by_url(url="URL-OF-THE-GOOGLE-SHEET-WITH-THE-SEED-DATA")

# accessing the specific worksheet by index 
# if index is left blank the first sheet will be read
worksheet = sheet.get_worksheet(index=0)


# defining a function to read all records from the worksheet
def get_all_records(worksheet=worksheet):
    records = worksheet.get_all_records()

    return records
```
#### Understanding the code
1. First we define the globals for authorization. Replace the 'NAME-OF-JSON-FILE-WITH-CREDENTIALS'  with the name of the json file downloaded Google API console. 
2. Then we open the google sheet in which the data is stored using its URL
3. next we get the specific woksheet in which the data is stored using its index location. The first sheet is indexed '0'
4. Lastly we define a new function which gets all the records from that specific worksheet and returns it

## Testing the Google Sheet access
* Re-open the _app.py_ file and edit the code

```python
from flask import Flask, render_template, jsonify
import pprint
from read_sheet import get_all_records

# instantiate a new Flash app
app = Flask(__name__)


@app.route('/')
def earthquake_data():
    # storing all data in a variable records
    earthquakes = get_all_records()
    # creating empty list
    data = []

    # looping through the variable
    for earthquake in earthquakes:
        # appending to list
        data.append(
            {
                "Latitude": earthquake['Latitude'],
                "Longitude": earthquake['Longitude'],
                "Magnitude": earthquake['Magnitude']
            }
        )

    # returning the list in json format
    return jsonify(data)


if __name__ == "__main__":
    app.run()
```

* `flask run` from your terminal
* _localhost:5000_ should now return a json representation of the data

![list of earthquakes](/assets/images/earthquake_json.png)

## Plotting the map
Now that we have verified the access to the Google Sheets data, let us plot it on a map. 

* `touch plot_map.py` from your terminal
* open the _plot_map.py_ file and copy the following code

```python
import folium

def basemap_layer():
    map = folium.Map(min_zoom=2)

    return map

def earthquake_plot(map, earthquakes):
    for earthquake in earthquakes:
        folium.Marker(location=[earthquake['Latitude'],
                                earthquake['Longitude']],
                        popup=folium.Popup("<b>Magnitude: </b>{}".format(earthquake['Magnitude']))
                        ).add_to(map)
```

#### Understanding the code
1. first we define the basemap_layer function which plots a basic map using the `folium.Map()`. The map is set to use all default values except for the **min_zoom** attribute which is set to 2.
2. next we define a function to plot the earthquake locations using the `folium.Marker()`, and add it to the parent map generated in the previous step 

## Rendering the map from the Flask app
Open the `app.py` file and add this code

```python
from flask import Flask, render_template
from read_sheet import get_all_records
from plot_map import basemap_layer, earthquake_plot

# instantiate a new Flash app
app = Flask(__name__)


@app.route('/')
def earthquake_data():
    # storing all data in a variable records
    earthquakes = get_all_records()
   
	  # creating the map by calling the basemap_layer function
	  map = basemap_layer()
		
		# generating the earthquake locations and plotting it
		# by calling the earthquake_plot function
    earthquake_plot(map=map, earthquakes=earthquakes)
		
		# save the map in the templates folder
    map.save('templates/map.html')
   
	  # render the index.html
    return render_template('index.html')


if __name__ == "__main__":
    app.run()
```

#### Understanding the code
1. from the previously created _plot_map.py_ file we import the **basemap_layer** and **earthquake_plot** functions
2. inside the view, first we create the map by calling the `basemap_layer()` function
3. to the map generated in step 2 we add the location of the earthquake generated by calling the `earthquake_plot()` function. The map from step 2 and earthquakes data is supplied as arguments
4. the map is saved as _map.html_ in the _templates_ folder. Create the templates folder before hand
5. lastly, return the _index.html_

## Creating the _index.html_ file
* `mkdir templates` and `touch templates/index.html`
* open the _index.html_ and copy the following code

{% raw %}
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    {% include "map.html" %}
</body>
</html>
```
{% endraw %}

#### Understanding the code
1. We create the _index.html_ file within the _templates_ folder
2. In the body of the html boilerplate we include the _map.html_ generated from the flask view using the jinja templating

!['web_page_displaying_the_map'](/assets/images/web_map.png)
## How does this map help in plotting crowdsourced data?
The interesting fact about this application is that since the mapping function is called from within the Flask view, the data is called everytime the web map is refreshed in the browser. Thus if a new record is added to the Google sheet, it is immediately rendered by the application on page refresh.

Additionally, no database needs to be setup and new records added could be validated by using the Google Sheets cell validation functionality itself. Contributors to project only need to access the specific Google Sheet.

So let's try out this functionality. We all know about the 2015 Nepal earthquake. Let's update the data in the sheet. 

To find Nepal's coordinates visit [latlong.net](http://www.latlong.net) and type Nepal in the search box. Copy the Latitude and Longtidue values to the Google sheet. [Wikipedia](https://en.wikipedia.org/wiki/April_2015_Nepal_earthquake) also suggests that the Magnitude of the earthquake was 7.8. Populate the **Magnitude** column with this value.

Now return to the map view and refresh the page. You will find that a marker for the new location has been added to the map.

!['web_map_after_refresh'](/assets/images/web_map_nepal.png)

This codethrough focussed only on creating the map using data from Google Sheets, and lacks commonly found functionalities in other web maps. We will focus on that in another tutorial. Till then, Happy coding!
