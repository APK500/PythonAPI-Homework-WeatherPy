
# Temperature Approaching the Equator




```python
# Temperature VS Latitude:  
# We can see that the observed temperature (in Celsius) 
# approaches the max of about 37-39 degrees(C) near the equator-- or the 0 latitude point.  As latitude increases (IE, we go North) we begin to see a drop in temp.


# Humidity (%) VS Latitude:  The more humid cities tend to group between -20 degrees and +80 degrees latitude.
# This suggests that humidity isn't necessarily highest near the equator.
# High humidity levels can persist anywhere.  IE, the equator isn't neccesarily more humid.

# Cloudiness VS Latitude:   Cloud cover appears much more random and not particularly tied to
# latitude.  There is a small negative correlation that can be seen:  It appears that from -20 to +40
# degrees in latitude, cloudiness tends to have smaller values.  IE, the equator
# may be somewhat less cloudy than at higher latitudes.  

# Windspeed VS Latitide:  Windspeed seems to carry an even smaller correlation to latitiude.
# It appears that most cities chosen at random tend to have a windspeed of between 4-10 mph, on average.
# Wind speeds exceeding 10mph seem very infrequent--regardless of their position to the equator.  
```


```python
# IMPORT OUR DEPENDENCIES:  

#To create our randomly-selected coordinates:
import random
import requests
import numpy as np


#To hold our data and create dataframes:
import pandas as pd


#Our API keys, and citipy (newly installed for project), to import the city weather-data.
from config import api_key
from citipy import citipy 


#And to plot our data:
import matplotlib.pyplot as plt
import matplotlib


#Last, for any formating of plots (they may be needed):
import seaborn
```


```python
# GETTING A RANDOM SET OF COORDINATES TO USE FOR CALLING ON CITY WEATHER: 

# First we define the Latitude & Longitude Zones and use numpy to select 
#..coordinates at random, given the earth's ranges:


latitude_zone = np.arange(-90,90,15)
longitude_zone = np.arange(-180,180,15)

```


```python
# DATAFRAME TO HOLD OUR COORDINATES:

# Create our list/Pandas DataFrame & calling it "cities" which will hold the coordinates to their city.


cities_df = pd.DataFrame()

cities_df["Latitude"] = ""
cities_df["Longitude"] = ""

```


```python
# COORDINATE SELECTION!

# First, using a coordinate systen we will assign "X" for latitude, and "Y" for long.
# For both latitude "X" & longitude "Y", we randomly select 500 unique coordinates:
# For the random samples we collect, we will assign "lats" for X and "lons" for y.
# We then will create/append the lists, "lat_samples" and "lon_samples", to use in dataframes.

for x in latitude_zone:
    for y in longitude_zone:
        x_values = list(np.arange(x,x+15,0.01))
        y_values = list(np.arange(y,y+15,0.01))
        lats = random.sample(x_values,500)
        lons = random.sample(y_values,500)
        lat_samples = [(x+dec_lat) for dec_lat in lats]
        lon_samples = [y+dec_lon for dec_lon in lons]
        cities_df = cities_df.append(pd.DataFrame.from_dict({"Latitude":lat_samples,
                                       "Longitude":lon_samples}))

# We then place the coordinates into our "cities" dataframe that was created above.        
cities_df = cities_df.reset_index(drop=True)


```


```python
# USING CITIPY MODULE TO TIE COORDINATES TO A CORRESPONDING/NEARBY CITY:

cities_df["Closest City name"] = ""
cities_df["Closest Country code"] = ""
for index,row in cities_df.iterrows():
    city = citipy.nearest_city(row["Latitude"],row["Longitude"])
    cities_df.set_value(index,"Closest City name",city.city_name)
    cities_df.set_value(index,"Closest Country code",city.country_code)
```

    C:\Users\Adam Knapp\AppData\Local\conda\conda\envs\PythonData\lib\site-packages\ipykernel_launcher.py:7: FutureWarning: set_value is deprecated and will be removed in a future release. Please use .at[] or .iat[] accessors instead
      import sys
    C:\Users\Adam Knapp\AppData\Local\conda\conda\envs\PythonData\lib\site-packages\ipykernel_launcher.py:8: FutureWarning: set_value is deprecated and will be removed in a future release. Please use .at[] or .iat[] accessors instead
      
    


```python
# CLEANING THE DATAFRAME: ELIMINATE COORDINATE-SETS THAT DON'T YIELD NEARBY CITIES:


# First we create a new data frame that eliminates coordinates that aren't near any city:
# ..Calling it "clean_cities":
filtered_cities_df = cities_df.drop(['Latitude', 'Longitude'],axis=1)
filtered_cities_df

# Next we filter for any possible duplicates (cities that come up twice)
filtered_cities_df = filtered_cities_df.drop_duplicates()


```


```python
# CREATING OUR SET OF CITIES WE WILL MAKE AN API CALL WITH

# Creation of our random sample set of cities from our "clean" data frame (above).
# Now we use a sample size of 500 in order to return their weather data.
# ** We will call this group of 500, "selected_cities".

selected_cities = filtered_cities_df.sample(500)

selected_cities = selected_cities.reset_index(drop=True)


```


```python
# USING API CALLS TO GATHER WEATHER INFO ON OUR SELECTED CITIES:

# We use Openweathermap to make our API CALLS:
# Set up format for the calls:
base_url = "http://api.openweathermap.org/data/2.5/weather"

app_id = api_key

params = { "appid" :app_id,"units":"metric" }

```


```python
# NOW enter the call data, url formatting, variables we want to collect &
# interate through for our "selected_cities" group:

def encrypt_key(input_url):
    return input_url[0:53]+"<YourKey>"+input_url[85:]

for index,row in selected_cities.iterrows():
    params["q"] =f'{row["Closest City name"]},{row["Closest Country code"]}'
    print(f"Retrieving weather information for {params['q']}")
    city_weather_resp = requests.get(base_url,params)
    print(encrypt_key(city_weather_resp.url))
    city_weather_resp  = city_weather_resp.json()
    selected_cities.set_value(index,"Latitude",city_weather_resp.get("coord",{}).get("lat"))
    selected_cities.set_value(index,"Longitude",city_weather_resp.get("coord",{}).get("lon"))
    selected_cities.set_value(index,"Temperature",city_weather_resp.get("main",{}).get("temp_max"))
    selected_cities.set_value(index,"Wind speed",city_weather_resp.get("wind",{}).get("speed"))
    selected_cities.set_value(index,"Humidity",city_weather_resp.get("main",{}).get("humidity"))
    selected_cities.set_value(index,"Cloudiness",city_weather_resp.get("clouds",{}).get("all"))
```

    Retrieving weather information for chakwal,pk
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=chakwal%2Cpk
    

    C:\Users\Adam Knapp\AppData\Local\conda\conda\envs\PythonData\lib\site-packages\ipykernel_launcher.py:13: FutureWarning: set_value is deprecated and will be removed in a future release. Please use .at[] or .iat[] accessors instead
      del sys.path[0]
    C:\Users\Adam Knapp\AppData\Local\conda\conda\envs\PythonData\lib\site-packages\ipykernel_launcher.py:14: FutureWarning: set_value is deprecated and will be removed in a future release. Please use .at[] or .iat[] accessors instead
      
    C:\Users\Adam Knapp\AppData\Local\conda\conda\envs\PythonData\lib\site-packages\ipykernel_launcher.py:15: FutureWarning: set_value is deprecated and will be removed in a future release. Please use .at[] or .iat[] accessors instead
      from ipykernel import kernelapp as app
    C:\Users\Adam Knapp\AppData\Local\conda\conda\envs\PythonData\lib\site-packages\ipykernel_launcher.py:16: FutureWarning: set_value is deprecated and will be removed in a future release. Please use .at[] or .iat[] accessors instead
      app.launch_new_instance()
    C:\Users\Adam Knapp\AppData\Local\conda\conda\envs\PythonData\lib\site-packages\ipykernel_launcher.py:17: FutureWarning: set_value is deprecated and will be removed in a future release. Please use .at[] or .iat[] accessors instead
    C:\Users\Adam Knapp\AppData\Local\conda\conda\envs\PythonData\lib\site-packages\ipykernel_launcher.py:18: FutureWarning: set_value is deprecated and will be removed in a future release. Please use .at[] or .iat[] accessors instead
    

    Retrieving weather information for nakasongola,ug
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=nakasongola%2Cug
    Retrieving weather information for barra velha,br
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=barra+velha%2Cbr
    Retrieving weather information for mosquera,co
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=mosquera%2Cco
    Retrieving weather information for trat,th
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=trat%2Cth
    Retrieving weather information for cay,tr
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=cay%2Ctr
    Retrieving weather information for mianyang,cn
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=mianyang%2Ccn
    Retrieving weather information for santa catalina,co
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=santa+catalina%2Cco
    Retrieving weather information for maniitsoq,gl
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=maniitsoq%2Cgl
    Retrieving weather information for aneho,tg
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=aneho%2Ctg
    Retrieving weather information for chkalovsk,tj
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=chkalovsk%2Ctj
    Retrieving weather information for laguna,br
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=laguna%2Cbr
    Retrieving weather information for unye,tr
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=unye%2Ctr
    Retrieving weather information for ararangua,br
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=ararangua%2Cbr
    Retrieving weather information for saint-philippe,re
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=saint-philippe%2Cre
    Retrieving weather information for ascension,mx
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=ascension%2Cmx
    Retrieving weather information for avezzano,it
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=avezzano%2Cit
    Retrieving weather information for shaki,ng
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=shaki%2Cng
    Retrieving weather information for bobcaygeon,ca
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=bobcaygeon%2Cca
    Retrieving weather information for buritizeiro,br
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=buritizeiro%2Cbr
    Retrieving weather information for igrim,ru
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=igrim%2Cru
    Retrieving weather information for lacdayan,ph
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=lacdayan%2Cph
    Retrieving weather information for juan de acosta,co
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=juan+de+acosta%2Cco
    Retrieving weather information for santiago,br
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=santiago%2Cbr
    Retrieving weather information for charleston,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=charleston%2Cus
    Retrieving weather information for olafsvik,is
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=olafsvik%2Cis
    Retrieving weather information for sar-e pul,af
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=sar-e+pul%2Caf
    Retrieving weather information for dondo,mz
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=dondo%2Cmz
    Retrieving weather information for tosya,tr
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=tosya%2Ctr
    Retrieving weather information for cambui,br
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=cambui%2Cbr
    Retrieving weather information for humaita,py
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=humaita%2Cpy
    Retrieving weather information for la paz,ph
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=la+paz%2Cph
    Retrieving weather information for waspan,ni
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=waspan%2Cni
    Retrieving weather information for manacor,es
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=manacor%2Ces
    Retrieving weather information for camacha,pt
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=camacha%2Cpt
    Retrieving weather information for geilo,no
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=geilo%2Cno
    Retrieving weather information for zhengjiatun,cn
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=zhengjiatun%2Ccn
    Retrieving weather information for kaduna,ng
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=kaduna%2Cng
    Retrieving weather information for qasigiannguit,gl
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=qasigiannguit%2Cgl
    Retrieving weather information for yuma,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=yuma%2Cus
    Retrieving weather information for panukulan,ph
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=panukulan%2Cph
    Retrieving weather information for matiao,ph
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=matiao%2Cph
    Retrieving weather information for bouza,ne
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=bouza%2Cne
    Retrieving weather information for spoleto,it
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=spoleto%2Cit
    Retrieving weather information for sisimiut,gl
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=sisimiut%2Cgl
    Retrieving weather information for ningan,cn
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=ningan%2Ccn
    Retrieving weather information for castelnaudary,fr
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=castelnaudary%2Cfr
    Retrieving weather information for sur,om
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=sur%2Com
    Retrieving weather information for salto del guaira,py
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=salto+del+guaira%2Cpy
    Retrieving weather information for samarkand,uz
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=samarkand%2Cuz
    Retrieving weather information for kostomuksha,ru
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=kostomuksha%2Cru
    Retrieving weather information for chepareria,ke
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=chepareria%2Cke
    Retrieving weather information for tayibe,il
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=tayibe%2Cil
    Retrieving weather information for castres,fr
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=castres%2Cfr
    Retrieving weather information for aflao,gh
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=aflao%2Cgh
    Retrieving weather information for boulder city,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=boulder+city%2Cus
    Retrieving weather information for la libertad,sv
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=la+libertad%2Csv
    Retrieving weather information for fernandopolis,br
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=fernandopolis%2Cbr
    Retrieving weather information for manado,id
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=manado%2Cid
    Retrieving weather information for inca,es
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=inca%2Ces
    Retrieving weather information for gympie,au
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=gympie%2Cau
    Retrieving weather information for bataipora,br
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=bataipora%2Cbr
    Retrieving weather information for pangnirtung,ca
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=pangnirtung%2Cca
    Retrieving weather information for medea,dz
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=medea%2Cdz
    Retrieving weather information for ureki,ge
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=ureki%2Cge
    Retrieving weather information for rikitea,pf
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=rikitea%2Cpf
    Retrieving weather information for balestrand,no
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=balestrand%2Cno
    Retrieving weather information for paytug,uz
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=paytug%2Cuz
    Retrieving weather information for umm ruwabah,sd
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=umm+ruwabah%2Csd
    Retrieving weather information for xghajra,mt
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=xghajra%2Cmt
    Retrieving weather information for pontiac,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=pontiac%2Cus
    Retrieving weather information for luziania,br
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=luziania%2Cbr
    Retrieving weather information for mint hill,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=mint+hill%2Cus
    Retrieving weather information for clemencia,co
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=clemencia%2Cco
    Retrieving weather information for toowoomba,au
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=toowoomba%2Cau
    Retrieving weather information for san pedro,ph
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=san+pedro%2Cph
    Retrieving weather information for surin,th
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=surin%2Cth
    Retrieving weather information for mora,cm
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=mora%2Ccm
    Retrieving weather information for piranhas,br
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=piranhas%2Cbr
    Retrieving weather information for show low,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=show+low%2Cus
    Retrieving weather information for burley,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=burley%2Cus
    Retrieving weather information for kayseri,tr
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=kayseri%2Ctr
    Retrieving weather information for pueblo rico,co
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=pueblo+rico%2Cco
    Retrieving weather information for nigde,tr
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=nigde%2Ctr
    Retrieving weather information for tir pol,af
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=tir+pol%2Caf
    Retrieving weather information for aklavik,ca
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=aklavik%2Cca
    Retrieving weather information for lukovetskiy,ru
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=lukovetskiy%2Cru
    Retrieving weather information for rosignol,gy
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=rosignol%2Cgy
    Retrieving weather information for qixia,cn
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=qixia%2Ccn
    Retrieving weather information for wheatley,ca
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=wheatley%2Cca
    Retrieving weather information for candoni,ph
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=candoni%2Cph
    Retrieving weather information for coronado,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=coronado%2Cus
    Retrieving weather information for kamaishi,jp
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=kamaishi%2Cjp
    Retrieving weather information for kirkenes,no
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=kirkenes%2Cno
    Retrieving weather information for marondera,zw
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=marondera%2Czw
    Retrieving weather information for ko samui,th
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=ko+samui%2Cth
    Retrieving weather information for takoradi,gh
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=takoradi%2Cgh
    Retrieving weather information for taos,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=taos%2Cus
    Retrieving weather information for mataura,pf
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=mataura%2Cpf
    Retrieving weather information for carrollton,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=carrollton%2Cus
    Retrieving weather information for shujaabad,pk
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=shujaabad%2Cpk
    Retrieving weather information for huangnihe,cn
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=huangnihe%2Ccn
    Retrieving weather information for sao joaquim,br
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=sao+joaquim%2Cbr
    Retrieving weather information for matameye,ne
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=matameye%2Cne
    Retrieving weather information for natagaima,co
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=natagaima%2Cco
    Retrieving weather information for andradina,br
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=andradina%2Cbr
    Retrieving weather information for boa vista,br
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=boa+vista%2Cbr
    Retrieving weather information for bloomingdale,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=bloomingdale%2Cus
    Retrieving weather information for juba,sd
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=juba%2Csd
    Retrieving weather information for manchester,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=manchester%2Cus
    Retrieving weather information for gotsu,jp
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=gotsu%2Cjp
    Retrieving weather information for sheltozero,ru
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=sheltozero%2Cru
    Retrieving weather information for doume,cm
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=doume%2Ccm
    Retrieving weather information for dosso,ne
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=dosso%2Cne
    Retrieving weather information for ikom,ng
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=ikom%2Cng
    Retrieving weather information for ankpa,ng
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=ankpa%2Cng
    Retrieving weather information for yanji,cn
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=yanji%2Ccn
    Retrieving weather information for candelaria,br
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=candelaria%2Cbr
    Retrieving weather information for wanlaweyn,so
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=wanlaweyn%2Cso
    Retrieving weather information for magan,ru
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=magan%2Cru
    Retrieving weather information for coachella,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=coachella%2Cus
    Retrieving weather information for mezen,ru
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=mezen%2Cru
    Retrieving weather information for turkistan,kz
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=turkistan%2Ckz
    Retrieving weather information for sabaudia,it
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=sabaudia%2Cit
    Retrieving weather information for yaviza,pa
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=yaviza%2Cpa
    Retrieving weather information for chifeng,cn
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=chifeng%2Ccn
    Retrieving weather information for bom despacho,br
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=bom+despacho%2Cbr
    Retrieving weather information for lundamo,no
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=lundamo%2Cno
    Retrieving weather information for cocobeach,ga
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=cocobeach%2Cga
    Retrieving weather information for puerto el triunfo,sv
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=puerto+el+triunfo%2Csv
    Retrieving weather information for yumen,cn
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=yumen%2Ccn
    Retrieving weather information for buenos aires,cr
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=buenos+aires%2Ccr
    Retrieving weather information for kantang,th
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=kantang%2Cth
    Retrieving weather information for selje,no
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=selje%2Cno
    Retrieving weather information for chesley,ca
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=chesley%2Cca
    Retrieving weather information for simbahan,ph
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=simbahan%2Cph
    Retrieving weather information for bertoua,cm
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=bertoua%2Ccm
    Retrieving weather information for klaeng,th
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=klaeng%2Cth
    Retrieving weather information for swabi,pk
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=swabi%2Cpk
    Retrieving weather information for maine-soroa,ne
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=maine-soroa%2Cne
    Retrieving weather information for bama,ng
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=bama%2Cng
    Retrieving weather information for nargana,pa
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=nargana%2Cpa
    Retrieving weather information for oranjemund,na
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=oranjemund%2Cna
    Retrieving weather information for big rapids,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=big+rapids%2Cus
    Retrieving weather information for saint simons,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=saint+simons%2Cus
    Retrieving weather information for barentu,er
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=barentu%2Cer
    Retrieving weather information for adana,tr
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=adana%2Ctr
    Retrieving weather information for shitanjing,cn
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=shitanjing%2Ccn
    Retrieving weather information for racine,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=racine%2Cus
    Retrieving weather information for cascavel,br
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=cascavel%2Cbr
    Retrieving weather information for ullal,in
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=ullal%2Cin
    Retrieving weather information for casay,ph
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=casay%2Cph
    Retrieving weather information for traverse city,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=traverse+city%2Cus
    Retrieving weather information for mallama,co
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=mallama%2Cco
    Retrieving weather information for monzon,es
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=monzon%2Ces
    Retrieving weather information for pietermaritzburg,za
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=pietermaritzburg%2Cza
    Retrieving weather information for danville,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=danville%2Cus
    Retrieving weather information for chaman,pk
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=chaman%2Cpk
    Retrieving weather information for khoy,ir
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=khoy%2Cir
    Retrieving weather information for gornopravdinsk,ru
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=gornopravdinsk%2Cru
    Retrieving weather information for cockeysville,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=cockeysville%2Cus
    Retrieving weather information for ribeira brava,pt
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=ribeira+brava%2Cpt
    Retrieving weather information for erbaa,tr
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=erbaa%2Ctr
    Retrieving weather information for sturgeon bay,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=sturgeon+bay%2Cus
    Retrieving weather information for teodoro sampaio,br
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=teodoro+sampaio%2Cbr
    Retrieving weather information for florianopolis,br
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=florianopolis%2Cbr
    Retrieving weather information for daura,ng
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=daura%2Cng
    Retrieving weather information for dawlatabad,af
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=dawlatabad%2Caf
    Retrieving weather information for palana,ru
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=palana%2Cru
    Retrieving weather information for blagnac,fr
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=blagnac%2Cfr
    Retrieving weather information for mardin,tr
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=mardin%2Ctr
    Retrieving weather information for qarqin,af
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=qarqin%2Caf
    Retrieving weather information for sakaiminato,jp
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=sakaiminato%2Cjp
    Retrieving weather information for limbe,cm
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=limbe%2Ccm
    Retrieving weather information for amot,no
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=amot%2Cno
    Retrieving weather information for benton harbor,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=benton+harbor%2Cus
    Retrieving weather information for lewisville,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=lewisville%2Cus
    Retrieving weather information for barra do bugres,br
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=barra+do+bugres%2Cbr
    Retrieving weather information for kieta,pg
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=kieta%2Cpg
    Retrieving weather information for senador canedo,br
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=senador+canedo%2Cbr
    Retrieving weather information for inndyr,no
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=inndyr%2Cno
    Retrieving weather information for san pedro de ycuamandiyu,py
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=san+pedro+de+ycuamandiyu%2Cpy
    Retrieving weather information for along,in
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=along%2Cin
    Retrieving weather information for samandag,tr
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=samandag%2Ctr
    Retrieving weather information for debre sina,et
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=debre+sina%2Cet
    Retrieving weather information for lipin bor,ru
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=lipin+bor%2Cru
    Retrieving weather information for linping,cn
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=linping%2Ccn
    Retrieving weather information for edgewater,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=edgewater%2Cus
    Retrieving weather information for haibowan,cn
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=haibowan%2Ccn
    Retrieving weather information for clifton,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=clifton%2Cus
    Retrieving weather information for marsh harbour,bs
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=marsh+harbour%2Cbs
    Retrieving weather information for eyrarbakki,is
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=eyrarbakki%2Cis
    Retrieving weather information for tunceli,tr
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=tunceli%2Ctr
    Retrieving weather information for marystown,ca
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=marystown%2Cca
    Retrieving weather information for paamiut,gl
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=paamiut%2Cgl
    Retrieving weather information for hakkari,tr
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=hakkari%2Ctr
    Retrieving weather information for silver city,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=silver+city%2Cus
    Retrieving weather information for lock haven,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=lock+haven%2Cus
    Retrieving weather information for pitanga,br
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=pitanga%2Cbr
    Retrieving weather information for napanee,ca
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=napanee%2Cca
    Retrieving weather information for douglas,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=douglas%2Cus
    Retrieving weather information for zhaocheng,cn
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=zhaocheng%2Ccn
    Retrieving weather information for piney green,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=piney+green%2Cus
    Retrieving weather information for eldoret,ke
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=eldoret%2Cke
    Retrieving weather information for surenavan,am
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=surenavan%2Cam
    Retrieving weather information for elbistan,tr
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=elbistan%2Ctr
    Retrieving weather information for nongan,cn
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=nongan%2Ccn
    Retrieving weather information for kodinsk,ru
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=kodinsk%2Cru
    Retrieving weather information for peruibe,br
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=peruibe%2Cbr
    Retrieving weather information for tyssedal,no
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=tyssedal%2Cno
    Retrieving weather information for ypsilanti,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=ypsilanti%2Cus
    Retrieving weather information for kapoeta,sd
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=kapoeta%2Csd
    Retrieving weather information for pisco,pe
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=pisco%2Cpe
    Retrieving weather information for juegang,cn
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=juegang%2Ccn
    Retrieving weather information for gore,et
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=gore%2Cet
    Retrieving weather information for sirnak,tr
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=sirnak%2Ctr
    Retrieving weather information for mountain home,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=mountain+home%2Cus
    Retrieving weather information for himora,et
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=himora%2Cet
    Retrieving weather information for qunduz,af
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=qunduz%2Caf
    Retrieving weather information for totness,sr
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=totness%2Csr
    Retrieving weather information for marratxi,es
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=marratxi%2Ces
    Retrieving weather information for batagay,ru
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=batagay%2Cru
    Retrieving weather information for anito,ph
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=anito%2Cph
    Retrieving weather information for katherine,au
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=katherine%2Cau
    Retrieving weather information for linden,gy
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=linden%2Cgy
    Retrieving weather information for luderitz,na
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=luderitz%2Cna
    Retrieving weather information for pattoki,pk
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=pattoki%2Cpk
    Retrieving weather information for yokadouma,cm
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=yokadouma%2Ccm
    Retrieving weather information for ordu,tr
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=ordu%2Ctr
    Retrieving weather information for formosa,ar
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=formosa%2Car
    Retrieving weather information for gravdal,no
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=gravdal%2Cno
    Retrieving weather information for somerset,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=somerset%2Cus
    Retrieving weather information for kloulklubed,pw
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=kloulklubed%2Cpw
    Retrieving weather information for cannes,fr
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=cannes%2Cfr
    Retrieving weather information for teruel,co
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=teruel%2Cco
    Retrieving weather information for fort payne,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=fort+payne%2Cus
    Retrieving weather information for quirinopolis,br
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=quirinopolis%2Cbr
    Retrieving weather information for bonita,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=bonita%2Cus
    Retrieving weather information for nabul,tn
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=nabul%2Ctn
    Retrieving weather information for corbelia,br
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=corbelia%2Cbr
    Retrieving weather information for bergerac,fr
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=bergerac%2Cfr
    Retrieving weather information for newport news,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=newport+news%2Cus
    Retrieving weather information for norton shores,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=norton+shores%2Cus
    Retrieving weather information for shakiso,et
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=shakiso%2Cet
    Retrieving weather information for talas,kg
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=talas%2Ckg
    Retrieving weather information for boa esperanca do sul,br
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=boa+esperanca+do+sul%2Cbr
    Retrieving weather information for djougou,bj
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=djougou%2Cbj
    Retrieving weather information for toktogul,kg
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=toktogul%2Ckg
    Retrieving weather information for looc,ph
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=looc%2Cph
    Retrieving weather information for magui,co
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=magui%2Cco
    Retrieving weather information for kingaroy,au
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=kingaroy%2Cau
    Retrieving weather information for jarjis,tn
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=jarjis%2Ctn
    Retrieving weather information for bocas del toro,pa
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=bocas+del+toro%2Cpa
    Retrieving weather information for urumqi,cn
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=urumqi%2Ccn
    Retrieving weather information for mutare,zw
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=mutare%2Czw
    Retrieving weather information for brokopondo,sr
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=brokopondo%2Csr
    Retrieving weather information for berlevag,no
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=berlevag%2Cno
    Retrieving weather information for ahumada,mx
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=ahumada%2Cmx
    Retrieving weather information for verkhniy fiagdon,ru
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=verkhniy+fiagdon%2Cru
    Retrieving weather information for ikorodu,ng
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=ikorodu%2Cng
    Retrieving weather information for santa marta,co
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=santa+marta%2Cco
    Retrieving weather information for cape coast,gh
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=cape+coast%2Cgh
    Retrieving weather information for butaritari,ki
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=butaritari%2Cki
    Retrieving weather information for inverell,au
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=inverell%2Cau
    Retrieving weather information for horizontina,br
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=horizontina%2Cbr
    Retrieving weather information for corn island,ni
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=corn+island%2Cni
    Retrieving weather information for glenwood springs,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=glenwood+springs%2Cus
    Retrieving weather information for lindas,no
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=lindas%2Cno
    Retrieving weather information for williamsburg,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=williamsburg%2Cus
    Retrieving weather information for midland,ca
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=midland%2Cca
    Retrieving weather information for tazmalt,dz
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=tazmalt%2Cdz
    Retrieving weather information for grove city,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=grove+city%2Cus
    Retrieving weather information for kumluca,tr
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=kumluca%2Ctr
    Retrieving weather information for kars,tr
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=kars%2Ctr
    Retrieving weather information for deh rawud,af
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=deh+rawud%2Caf
    Retrieving weather information for yovon,tj
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=yovon%2Ctj
    Retrieving weather information for xiongyue,cn
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=xiongyue%2Ccn
    Retrieving weather information for resistencia,ar
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=resistencia%2Car
    Retrieving weather information for ridgecrest,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=ridgecrest%2Cus
    Retrieving weather information for marang,my
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=marang%2Cmy
    Retrieving weather information for posadas,ar
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=posadas%2Car
    Retrieving weather information for nantai,cn
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=nantai%2Ccn
    Retrieving weather information for jiupu,cn
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=jiupu%2Ccn
    Retrieving weather information for warwick,au
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=warwick%2Cau
    Retrieving weather information for puerto quijarro,bo
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=puerto+quijarro%2Cbo
    Retrieving weather information for bocana de paiwas,ni
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=bocana+de+paiwas%2Cni
    Retrieving weather information for flagstaff,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=flagstaff%2Cus
    Retrieving weather information for progreso,mx
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=progreso%2Cmx
    Retrieving weather information for brekstad,no
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=brekstad%2Cno
    Retrieving weather information for catio,gw
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=catio%2Cgw
    Retrieving weather information for holetown,bb
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=holetown%2Cbb
    Retrieving weather information for yeletskiy,ru
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=yeletskiy%2Cru
    Retrieving weather information for sandane,no
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=sandane%2Cno
    Retrieving weather information for bakurianis andeziti,ge
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=bakurianis+andeziti%2Cge
    Retrieving weather information for panjab,af
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=panjab%2Caf
    Retrieving weather information for reconquista,ar
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=reconquista%2Car
    Retrieving weather information for jinxi,cn
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=jinxi%2Ccn
    Retrieving weather information for acurenam,gq
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=acurenam%2Cgq
    Retrieving weather information for corum,tr
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=corum%2Ctr
    Retrieving weather information for la vergne,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=la+vergne%2Cus
    Retrieving weather information for kalaswala,pk
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=kalaswala%2Cpk
    Retrieving weather information for mineiros,br
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=mineiros%2Cbr
    Retrieving weather information for abaete,br
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=abaete%2Cbr
    Retrieving weather information for dzhusaly,kz
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=dzhusaly%2Ckz
    Retrieving weather information for kaupanger,no
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=kaupanger%2Cno
    Retrieving weather information for bichena,et
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=bichena%2Cet
    Retrieving weather information for san matias,bo
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=san+matias%2Cbo
    Retrieving weather information for sudak,ua
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=sudak%2Cua
    Retrieving weather information for tabarqah,tn
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=tabarqah%2Ctn
    Retrieving weather information for manlleu,es
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=manlleu%2Ces
    Retrieving weather information for kholmogory,ru
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=kholmogory%2Cru
    Retrieving weather information for gigmoto,ph
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=gigmoto%2Cph
    Retrieving weather information for korla,cn
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=korla%2Ccn
    Retrieving weather information for toledo,br
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=toledo%2Cbr
    Retrieving weather information for pobe,bj
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=pobe%2Cbj
    Retrieving weather information for bandipur,in
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=bandipur%2Cin
    Retrieving weather information for ilorin,ng
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=ilorin%2Cng
    Retrieving weather information for tera,ne
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=tera%2Cne
    Retrieving weather information for alanya,tr
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=alanya%2Ctr
    Retrieving weather information for blenheim,ca
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=blenheim%2Cca
    Retrieving weather information for necochea,ar
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=necochea%2Car
    Retrieving weather information for taywarah,af
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=taywarah%2Caf
    Retrieving weather information for machico,pt
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=machico%2Cpt
    Retrieving weather information for coquimbo,cl
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=coquimbo%2Ccl
    Retrieving weather information for lantawan,ph
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=lantawan%2Cph
    Retrieving weather information for rabak,sd
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=rabak%2Csd
    Retrieving weather information for salug,ph
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=salug%2Cph
    Retrieving weather information for korem,et
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=korem%2Cet
    Retrieving weather information for san joaquin,bo
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=san+joaquin%2Cbo
    Retrieving weather information for solginskiy,ru
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=solginskiy%2Cru
    Retrieving weather information for vrangel,ru
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=vrangel%2Cru
    Retrieving weather information for katsina,ng
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=katsina%2Cng
    Retrieving weather information for iguape,br
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=iguape%2Cbr
    Retrieving weather information for tuscaloosa,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=tuscaloosa%2Cus
    Retrieving weather information for masvingo,zw
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=masvingo%2Czw
    Retrieving weather information for karoi,zw
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=karoi%2Czw
    Retrieving weather information for mokhsogollokh,ru
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=mokhsogollokh%2Cru
    Retrieving weather information for severo-kurilsk,ru
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=severo-kurilsk%2Cru
    Retrieving weather information for kourou,gf
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=kourou%2Cgf
    Retrieving weather information for nova olimpia,br
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=nova+olimpia%2Cbr
    Retrieving weather information for guiratinga,br
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=guiratinga%2Cbr
    Retrieving weather information for veraval,in
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=veraval%2Cin
    Retrieving weather information for sinop,tr
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=sinop%2Ctr
    Retrieving weather information for gunjur,gm
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=gunjur%2Cgm
    Retrieving weather information for kudahuvadhoo,mv
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=kudahuvadhoo%2Cmv
    Retrieving weather information for zhangye,cn
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=zhangye%2Ccn
    Retrieving weather information for eldorado,ar
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=eldorado%2Car
    Retrieving weather information for navoi,uz
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=navoi%2Cuz
    Retrieving weather information for robertsport,lr
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=robertsport%2Clr
    Retrieving weather information for bella vista,py
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=bella+vista%2Cpy
    Retrieving weather information for iracoubo,gf
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=iracoubo%2Cgf
    Retrieving weather information for ongandjera,na
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=ongandjera%2Cna
    Retrieving weather information for eydhafushi,mv
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=eydhafushi%2Cmv
    Retrieving weather information for keta,gh
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=keta%2Cgh
    Retrieving weather information for abu jubayhah,sd
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=abu+jubayhah%2Csd
    Retrieving weather information for chak jhumra,pk
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=chak+jhumra%2Cpk
    Retrieving weather information for sunndalsora,no
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=sunndalsora%2Cno
    Retrieving weather information for wajir,ke
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=wajir%2Cke
    Retrieving weather information for bayji,iq
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=bayji%2Ciq
    Retrieving weather information for morganton,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=morganton%2Cus
    Retrieving weather information for caramay,ph
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=caramay%2Cph
    Retrieving weather information for fernley,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=fernley%2Cus
    Retrieving weather information for ormond beach,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=ormond+beach%2Cus
    Retrieving weather information for vermilion,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=vermilion%2Cus
    Retrieving weather information for rapu-rapu,ph
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=rapu-rapu%2Cph
    Retrieving weather information for masyaf,sy
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=masyaf%2Csy
    Retrieving weather information for agirish,ru
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=agirish%2Cru
    Retrieving weather information for zile,tr
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=zile%2Ctr
    Retrieving weather information for phalombe,mw
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=phalombe%2Cmw
    Retrieving weather information for briceno,co
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=briceno%2Cco
    Retrieving weather information for staryy nadym,ru
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=staryy+nadym%2Cru
    Retrieving weather information for mackay,au
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=mackay%2Cau
    Retrieving weather information for linqiong,cn
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=linqiong%2Ccn
    Retrieving weather information for scarborough,tt
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=scarborough%2Ctt
    Retrieving weather information for rosarito,mx
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=rosarito%2Cmx
    Retrieving weather information for evinayong,gq
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=evinayong%2Cgq
    Retrieving weather information for chopinzinho,br
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=chopinzinho%2Cbr
    Retrieving weather information for grand-santi,gf
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=grand-santi%2Cgf
    Retrieving weather information for mahebourg,mu
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=mahebourg%2Cmu
    Retrieving weather information for port elizabeth,za
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=port+elizabeth%2Cza
    Retrieving weather information for monchegorsk,ru
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=monchegorsk%2Cru
    Retrieving weather information for terrasini,it
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=terrasini%2Cit
    Retrieving weather information for kampot,kh
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=kampot%2Ckh
    Retrieving weather information for boa esperanca,br
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=boa+esperanca%2Cbr
    Retrieving weather information for aflu,dz
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=aflu%2Cdz
    Retrieving weather information for tibiao,ph
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=tibiao%2Cph
    Retrieving weather information for charlottesville,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=charlottesville%2Cus
    Retrieving weather information for kirovsk,ru
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=kirovsk%2Cru
    Retrieving weather information for sorrento,it
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=sorrento%2Cit
    Retrieving weather information for joao pinheiro,br
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=joao+pinheiro%2Cbr
    Retrieving weather information for taquaritinga,br
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=taquaritinga%2Cbr
    Retrieving weather information for lorengau,pg
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=lorengau%2Cpg
    Retrieving weather information for potiskum,ng
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=potiskum%2Cng
    Retrieving weather information for messina,za
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=messina%2Cza
    Retrieving weather information for christiansburg,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=christiansburg%2Cus
    Retrieving weather information for jalingo,ng
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=jalingo%2Cng
    Retrieving weather information for amga,ru
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=amga%2Cru
    Retrieving weather information for sangar,ru
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=sangar%2Cru
    Retrieving weather information for ballitoville,za
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=ballitoville%2Cza
    Retrieving weather information for opico,sv
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=opico%2Csv
    Retrieving weather information for yushu,cn
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=yushu%2Ccn
    Retrieving weather information for port lincoln,au
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=port+lincoln%2Cau
    Retrieving weather information for syasstroy,ru
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=syasstroy%2Cru
    Retrieving weather information for skeldon,gy
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=skeldon%2Cgy
    Retrieving weather information for bafra,tr
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=bafra%2Ctr
    Retrieving weather information for terre haute,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=terre+haute%2Cus
    Retrieving weather information for oyem,ga
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=oyem%2Cga
    Retrieving weather information for hermanus,za
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=hermanus%2Cza
    Retrieving weather information for huntington,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=huntington%2Cus
    Retrieving weather information for twentynine palms,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=twentynine+palms%2Cus
    Retrieving weather information for nedjo,et
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=nedjo%2Cet
    Retrieving weather information for tignere,cm
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=tignere%2Ccm
    Retrieving weather information for kondinskoye,ru
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=kondinskoye%2Cru
    Retrieving weather information for casas grandes,mx
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=casas+grandes%2Cmx
    Retrieving weather information for dromolaxia,cy
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=dromolaxia%2Ccy
    Retrieving weather information for apt,fr
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=apt%2Cfr
    Retrieving weather information for bloomfield,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=bloomfield%2Cus
    Retrieving weather information for colon,co
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=colon%2Cco
    Retrieving weather information for taloqan,af
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=taloqan%2Caf
    Retrieving weather information for maceio,br
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=maceio%2Cbr
    Retrieving weather information for puerto escondido,co
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=puerto+escondido%2Cco
    Retrieving weather information for asayita,et
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=asayita%2Cet
    Retrieving weather information for mikuni,jp
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=mikuni%2Cjp
    Retrieving weather information for iralaya,hn
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=iralaya%2Chn
    Retrieving weather information for sapa,ph
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=sapa%2Cph
    Retrieving weather information for baculin,ph
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=baculin%2Cph
    Retrieving weather information for yemtsa,ru
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=yemtsa%2Cru
    Retrieving weather information for foumbot,cm
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=foumbot%2Ccm
    Retrieving weather information for pinega,ru
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=pinega%2Cru
    Retrieving weather information for ponta delgada,pt
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=ponta+delgada%2Cpt
    Retrieving weather information for duartina,br
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=duartina%2Cbr
    Retrieving weather information for lemesos,cy
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=lemesos%2Ccy
    Retrieving weather information for togitsu,jp
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=togitsu%2Cjp
    Retrieving weather information for santa barbara,co
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=santa+barbara%2Cco
    Retrieving weather information for changtu,cn
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=changtu%2Ccn
    Retrieving weather information for mulatupo,pa
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=mulatupo%2Cpa
    Retrieving weather information for shu,kz
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=shu%2Ckz
    Retrieving weather information for goldsboro,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=goldsboro%2Cus
    Retrieving weather information for rostaq,af
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=rostaq%2Caf
    Retrieving weather information for jiaonan,cn
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=jiaonan%2Ccn
    Retrieving weather information for molave,ph
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=molave%2Cph
    Retrieving weather information for gaya,ne
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=gaya%2Cne
    Retrieving weather information for stranda,no
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=stranda%2Cno
    Retrieving weather information for ossora,ru
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=ossora%2Cru
    Retrieving weather information for souillac,mu
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=souillac%2Cmu
    Retrieving weather information for bon air,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=bon+air%2Cus
    Retrieving weather information for kaitangata,nz
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=kaitangata%2Cnz
    Retrieving weather information for iles,co
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=iles%2Cco
    Retrieving weather information for bongaree,au
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=bongaree%2Cau
    Retrieving weather information for kapchorwa,ug
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=kapchorwa%2Cug
    Retrieving weather information for takob,tj
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=takob%2Ctj
    Retrieving weather information for ashland,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=ashland%2Cus
    Retrieving weather information for planeta rica,co
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=planeta+rica%2Cco
    Retrieving weather information for manger,no
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=manger%2Cno
    Retrieving weather information for laramie,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=laramie%2Cus
    Retrieving weather information for ontario,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=ontario%2Cus
    Retrieving weather information for rab,hr
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=rab%2Chr
    Retrieving weather information for huarmey,pe
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=huarmey%2Cpe
    Retrieving weather information for yafran,ly
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=yafran%2Cly
    Retrieving weather information for twin falls,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=twin+falls%2Cus
    Retrieving weather information for bartica,gy
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=bartica%2Cgy
    Retrieving weather information for safaqis,tn
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=safaqis%2Ctn
    Retrieving weather information for hithadhoo,mv
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=hithadhoo%2Cmv
    Retrieving weather information for paharpur,pk
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=paharpur%2Cpk
    Retrieving weather information for rawah,iq
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=rawah%2Ciq
    Retrieving weather information for fillan,no
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=fillan%2Cno
    Retrieving weather information for luba,gq
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=luba%2Cgq
    Retrieving weather information for phalaborwa,za
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=phalaborwa%2Cza
    Retrieving weather information for thohoyandou,za
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=thohoyandou%2Cza
    Retrieving weather information for northam,au
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=northam%2Cau
    Retrieving weather information for nampula,mz
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=nampula%2Cmz
    Retrieving weather information for camingawan,ph
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=camingawan%2Cph
    Retrieving weather information for tabialan,ph
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=tabialan%2Cph
    Retrieving weather information for fenelon falls,ca
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=fenelon+falls%2Cca
    Retrieving weather information for uri,in
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=uri%2Cin
    Retrieving weather information for oildale,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=oildale%2Cus
    Retrieving weather information for bristol,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=bristol%2Cus
    Retrieving weather information for akhtyrskiy,ru
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=akhtyrskiy%2Cru
    Retrieving weather information for dogondoutchi,ne
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=dogondoutchi%2Cne
    Retrieving weather information for nieuw nickerie,sr
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=nieuw+nickerie%2Csr
    Retrieving weather information for northampton,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=northampton%2Cus
    Retrieving weather information for vao,nc
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=vao%2Cnc
    Retrieving weather information for nicoya,cr
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=nicoya%2Ccr
    Retrieving weather information for winnemucca,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=winnemucca%2Cus
    Retrieving weather information for lofthus,no
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=lofthus%2Cno
    Retrieving weather information for lima,pe
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=lima%2Cpe
    Retrieving weather information for dois corregos,br
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=dois+corregos%2Cbr
    Retrieving weather information for monatele,cm
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=monatele%2Ccm
    Retrieving weather information for watertown,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=watertown%2Cus
    Retrieving weather information for hihya,eg
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=hihya%2Ceg
    Retrieving weather information for curuguaty,py
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=curuguaty%2Cpy
    Retrieving weather information for jacksonville beach,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=jacksonville+beach%2Cus
    Retrieving weather information for tadotsu,jp
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=tadotsu%2Cjp
    Retrieving weather information for doha,kw
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=doha%2Ckw
    Retrieving weather information for potsdam,us
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=potsdam%2Cus
    Retrieving weather information for pyaozerskiy,ru
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=pyaozerskiy%2Cru
    Retrieving weather information for buritama,br
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=buritama%2Cbr
    Retrieving weather information for lambare,py
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=lambare%2Cpy
    Retrieving weather information for gagra,ge
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=gagra%2Cge
    Retrieving weather information for jamestown,sh
    http://api.openweathermap.org/data/2.5/weather?appid=<YourKey>&units=metric&q=jamestown%2Csh
    


```python
# POST CALL-RETREIVING:  CLEAN UP DATA (When needed) AND EXPORT OUR DATA TO CSV:
selected_cities = selected_cities.dropna()

selected_cities.shape
selected_cities.to_csv("City_Weather_data_2.csv")
```


```python
#  PLOTTING:  FIRST WE SET OUR PROPERTIES FOR OUR SCATTER-PLOTS:

# Sert the time
#  PLOTTING:  FIRST WE SET OUR TIME PROPERTIES FOR OUR SCATTER-PLOTS:
# import datetime
# date = datetime.date.today()
import time
date = time.strftime("%m/%d/%Y")
# print(date)

```


```python
#  TEMPERATRUE VS LATITUDE

plt.scatter(selected_cities['Latitude'],selected_cities['Temperature'])
plt.title(f"Temperature (C) vs. Latitude {date}")
plt.xlabel("Latitude")
plt.ylabel("Temperature (C)")
plt.style.use('ggplot')
plt.savefig("Temperature.png")
plt.show()
```


![png](output_13_0.png)



```python
#  HUMIDITY VS LATITUDE

# plt.scatter(latitude,humidity)
plt.scatter(selected_cities['Latitude'], selected_cities['Humidity'])
plt.title(f"Humidity (%) vs. Latitude {date}")
plt.xlabel("Latitude")
plt.ylabel("Humidity (%)")
plt.style.use('ggplot')
plt.savefig("Humidity.png")
plt.show()
```


![png](output_14_0.png)



```python
#  "CLOUDINESS" VS LATITUDE

# plt.scatter(latitude,cloudy)
plt.scatter(selected_cities['Latitude'], selected_cities['Cloudiness'])
plt.title(f"Cloudiness (%) vs. Latitude {date}")
plt.xlabel("Latitude")
plt.ylabel("Cloudiness (%)")
plt.style.use('ggplot')
plt.savefig("Cloudiness.png")
plt.show()
```


![png](output_15_0.png)



```python
# Wind speed (mph) VS Latitude:

# plt.scatter(latitude,windspeed)
plt.scatter(selected_cities['Latitude'], selected_cities['Wind speed'])
plt.title(f"Wind speed(mph) vs. Latitude {date}")
plt.xlabel("Latitude")
plt.ylabel("Wind speed(mph)")
plt.style.use('ggplot')
plt.savefig("Wind speed.png")
plt.show()
```


![png](output_16_0.png)

