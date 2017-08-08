---
layout: notebook
title: ""
---
# The Boston Light Swim temperature analysis with Python

In the past we demonstrated how to perform a CSW catalog search with [`OWSLib`](https://ioos.github.io/notebooks_demos//notebooks/2016-12-19-exploring_csw),
and how to obtain near real-time data with [`pyoos`](https://ioos.github.io/notebooks_demos//notebooks/2016-10-12-fetching_data).
In this notebook we will use both to find all observations and model data around the Boston Harbor to access the sea water temperature.


This workflow is part of an example to advise swimmers of the annual [Boston lighthouse swim](http://bostonlightswim.org/) of the Boston Harbor water temperature conditions prior to the race. For more information regarding the workflow presented here see [Signell, Richard P.; Fernandes, Filipe; Wilcox, Kyle.   2016. "Dynamic Reusable Workflows for Ocean Science." *J. Mar. Sci. Eng.* 4, no. 4: 68](http://dx.doi.org/10.3390/jmse4040068).

(This notebook uses a custom `ioos_tools` module that needs to be added to the path separately. We recommend cloning the [repository](https://github.com/ioos/notebooks_demos) on GitHub which already includes the most update version of `ioos_tools`.)

<div class="prompt input_prompt">
In&nbsp;[1]:
</div>

```python
import os
import sys
import warnings

ioos_tools = os.path.join(os.path.pardir)
sys.path.append(ioos_tools)

# Suppresing warnings for a "pretty output."
warnings.simplefilter('ignore')
```

This notebook is quite big and complex,
so to help us keep things organized we'll define a cell with the most important options and switches.

Below we can define the date,
bounding box, phenomena `SOS` and `CF` names and units,
and the catalogs we will search.

<div class="prompt input_prompt">
In&nbsp;[2]:
</div>

```python
%%writefile config.yaml

# Specify a YYYY-MM-DD hh:mm:ss date or integer day offset.
# If both start and stop are offsets they will be computed relative to datetime.today() at midnight.
# Use the dates commented below to reproduce the last Boston Light Swim event forecast.
date:
    start: -5 # 2016-8-16 00:00:00
    stop: +4 # 2016-8-29 00:00:00

run_name: 'latest'

# Boston harbor.
region:
    bbox: [-71.3, 42.03, -70.57, 42.63]
    crs: 'urn:ogc:def:crs:OGC:1.3:CRS84'

sos_name: 'sea_water_temperature'

cf_names:
    - sea_water_temperature
    - sea_surface_temperature
    - sea_water_potential_temperature
    - equivalent_potential_temperature
    - sea_water_conservative_temperature
    - pseudo_equivalent_potential_temperature

units: 'celsius'

catalogs:
    - https://data.ioos.us/csw
    - https://gamone.whoi.edu/csw
```
<div class="output_area"><div class="prompt"></div>
<pre>
    Overwriting config.yaml

</pre>
</div>
We'll print some of the search configuration options along the way to keep track of them.

<div class="prompt input_prompt">
In&nbsp;[3]:
</div>

```python
import shutil
from datetime import datetime
from ioos_tools.ioos import parse_config

config = parse_config('config.yaml')

# Saves downloaded data into a temporary directory.
save_dir = os.path.abspath(config['run_name'])
if os.path.exists(save_dir):
    shutil.rmtree(save_dir)
os.makedirs(save_dir)

fmt = '{:*^64}'.format
print(fmt('Saving data inside directory {}'.format(save_dir)))
print(fmt(' Run information '))
print('Run date: {:%Y-%m-%d %H:%M:%S}'.format(datetime.utcnow()))
print('Start: {:%Y-%m-%d %H:%M:%S}'.format(config['date']['start']))
print('Stop: {:%Y-%m-%d %H:%M:%S}'.format(config['date']['stop']))
print('Bounding box: {0:3.2f}, {1:3.2f},'
      '{2:3.2f}, {3:3.2f}'.format(*config['region']['bbox']))
```
<div class="output_area"><div class="prompt"></div>
<pre>
    Saving data inside directory /home/filipe/IOOS/notebooks_demos/notebooks/latest
    *********************** Run information ************************
    Run date: 2017-07-03 20:12:03
    Start: 2017-06-28 00:00:00
    Stop: 2017-07-07 00:00:00
    Bounding box: -71.30, 42.03,-70.57, 42.63

</pre>
</div>
We already created an `OWSLib.fes` filter [before](https://ioos.github.io/notebooks_demos//notebooks/2016-12-19-exploring_csw).
The main difference here is that we do not want the atmosphere model data,
so we are filtering out all the `GRIB-2` data format.

<div class="prompt input_prompt">
In&nbsp;[4]:
</div>

```python
def make_filter(config):
    from owslib import fes
    from ioos_tools.ioos import fes_date_filter
    kw = dict(wildCard='*', escapeChar='\\',
              singleChar='?', propertyname='apiso:AnyText')

    or_filt = fes.Or([fes.PropertyIsLike(literal=('*%s*' % val), **kw)
                      for val in config['cf_names']])

    not_filt = fes.Not([fes.PropertyIsLike(literal='GRIB-2', **kw)])

    begin, end = fes_date_filter(config['date']['start'],
                                 config['date']['stop'])
    bbox_crs = fes.BBox(config['region']['bbox'],
                        crs=config['region']['crs'])
    filter_list = [fes.And([bbox_crs, begin, end, or_filt, not_filt])]
    return filter_list


filter_list = make_filter(config)
```

In the cell below we ask the catalog for all the returns that match the filter and have an OPeNDAP endpoint.

<div class="prompt input_prompt">
In&nbsp;[5]:
</div>

```python
from ioos_tools.ioos import service_urls, get_csw_records
from owslib.csw import CatalogueServiceWeb


dap_urls = []
print(fmt(' Catalog information '))
for endpoint in config['catalogs']:
    print('URL: {}'.format(endpoint))
    try:
        csw = CatalogueServiceWeb(endpoint, timeout=120)
    except Exception as e:
        print('{}'.format(e))
        continue
    csw = get_csw_records(csw, filter_list, esn='full')
    OPeNDAP = service_urls(csw.records, identifier='OPeNDAP:OPeNDAP')
    odp = service_urls(csw.records, identifier='urn:x-esri:specification:ServiceType:odp:url')
    dap = OPeNDAP + odp
    dap_urls.extend(dap)

    print('Number of datasets available: {}'.format(len(csw.records.keys())))

    for rec, item in csw.records.items():
        print('{}'.format(item.title))
    if dap:
        print(fmt(' DAP '))
        for url in dap:
            print('{}.html'.format(url))
    print('\n')

# Get only unique endpoints.
dap_urls = list(set(dap_urls))
```
<div class="output_area"><div class="prompt"></div>
<pre>
    ********************* Catalog information **********************
    URL: https://data.ioos.us/csw
    Number of datasets available: 17
    HYbrid Coordinate Ocean Model (HYCOM): Global
    NECOFS (FVCOM) - Scituate - Latest Forecast
    NECOFS GOM3 (FVCOM) - Northeast US - Latest Forecast
    NECOFS Massachusetts (FVCOM) - Boston - Latest Forecast
    NECOFS Massachusetts (FVCOM) - Massachusetts Coastal - Latest Forecast
    NERACOOS Gulf of Maine Ocean Array: Realtime Buoy Observations: A01 Massachusetts Bay: A01 ACCELEROMETER Massachusetts Bay
    NOAA Coral Reef Watch Operational Daily Near-Real-Time Global 5-km Satellite Coral Bleaching Monitoring Products
    COAWST Modeling System: USEast: ROMS-WRF-SWAN coupled model (aka CNAPS)
    Coupled Northwest Atlantic Prediction System (CNAPS)
    Directional wave and sea surface temperature measurements collected in situ by Datawell Mark 3 directional buoy located near ASTORIA CANYON, OR from 2016/03/30 17:00:00 to 2017/07/03 18:11:47.
    Directional wave and sea surface temperature measurements collected in situ by Datawell Mark 3 directional buoy located near CLATSOP SPIT, OR from 2016/10/12 17:00:00 to 2017/07/03 18:00:53.
    Directional wave and sea surface temperature measurements collected in situ by Datawell Mark 3 directional buoy located near LAKESIDE, OR from 2017/03/31 23:00:00 to 2017/07/03 17:30:42.
    Directional wave and sea surface temperature measurements collected in situ by Datawell Mark 3 directional buoy located near LOWER COOK INLET, AK from 2016/12/16 00:00:00 to 2017/07/03 18:09:44.
    Directional wave and sea surface temperature measurements collected in situ by Datawell Mark 3 directional buoy located near OCEAN STATION PAPA from 2015/01/01 01:00:00 to 2017/07/03 17:40:54.
    Directional wave and sea surface temperature measurements collected in situ by Datawell Mark 3 directional buoy located near POST-RECOVERY GRAYS HARBOR from 2017/06/29 13:00:00 to 2017/07/03 17:53:36.
    Directional wave and sea surface temperature measurements collected in situ by Datawell Mark 3 directional buoy located near SCRIPPS NEARSHORE, CA from 2015/01/07 23:00:00 to 2017/07/03 18:00:18.
    G1SST, 1km blended SST
    ***************************** DAP ******************************
    http://oos.soest.hawaii.edu/thredds/dodsC/hioos/satellite/dhw_5km.html
    http://oos.soest.hawaii.edu/thredds/dodsC/pacioos/hycom/global.html
    http://thredds.cdip.ucsd.edu/thredds/dodsC/cdip/realtime/162p1_rt.nc.html
    http://thredds.cdip.ucsd.edu/thredds/dodsC/cdip/realtime/166p1_rt.nc.html
    http://thredds.cdip.ucsd.edu/thredds/dodsC/cdip/realtime/179p1_rt.nc.html
    http://thredds.cdip.ucsd.edu/thredds/dodsC/cdip/realtime/201p1_rt.nc.html
    http://thredds.cdip.ucsd.edu/thredds/dodsC/cdip/realtime/204p1_rt.nc.html
    http://thredds.cdip.ucsd.edu/thredds/dodsC/cdip/realtime/231p1_rt.nc.html
    http://thredds.cdip.ucsd.edu/thredds/dodsC/cdip/realtime/997p1_rt.nc.html
    http://thredds.secoora.org/thredds/dodsC/G1_SST_GLOBAL.nc.html
    http://thredds.secoora.org/thredds/dodsC/SECOORA_NCSU_CNAPS.nc.html
    http://www.neracoos.org/thredds/dodsC/UMO/DSG/SOS/A01/Accelerometer/HistoricRealtime/Agg.ncml.html
    http://www.smast.umassd.edu:8080/thredds/dodsC/FVCOM/NECOFS/Forecasts/NECOFS_FVCOM_OCEAN_BOSTON_FORECAST.nc.html
    http://www.smast.umassd.edu:8080/thredds/dodsC/FVCOM/NECOFS/Forecasts/NECOFS_FVCOM_OCEAN_MASSBAY_FORECAST.nc.html
    http://www.smast.umassd.edu:8080/thredds/dodsC/FVCOM/NECOFS/Forecasts/NECOFS_FVCOM_OCEAN_SCITUATE_FORECAST.nc.html
    http://www.smast.umassd.edu:8080/thredds/dodsC/FVCOM/NECOFS/Forecasts/NECOFS_GOM3_FORECAST.nc.html
    
    
    URL: https://gamone.whoi.edu/csw
    Number of datasets available: 2
    COAWST Forecast System : USGS : US East Coast and Gulf of Mexico (Experimental)
    COAWST Modeling System: USEast: ROMS-WRF-SWAN coupled model (aka CNAPS)
    ***************************** DAP ******************************
    http://geoport-dev.whoi.edu/thredds/dodsC/coawst_4/use/fmrc/coawst_4_use_best.ncd.html
    
    

</pre>
</div>
We found some models, and observations from NERACOOS there.
However, we do know that there are some buoys from NDBC and CO-OPS available too.
Also, those NERACOOS observations seem to be from a [CTD](http://www.neracoos.org/thredds/dodsC/UMO/DSG/SOS/A01/CTD1m/HistoricRealtime/Agg.ncml.html) mounted at 65 meters below the sea surface. Rendering them useless from our purpose.

So let's use the catalog only for the models by filtering the observations with `is_station` below.
And we'll rely `CO-OPS` and `NDBC` services for the observations.

<div class="prompt input_prompt">
In&nbsp;[6]:
</div>

```python
from ioos_tools.ioos import is_station

# Filter out some station endpoints.
non_stations = []
for url in dap_urls:
    try:
        if not is_station(url):
            non_stations.append(url)
    except (RuntimeError, OSError, IOError) as e:
        print('Could not access URL {}. {!r}'.format(url, e))

dap_urls = non_stations

print(fmt(' Filtered DAP '))
for url in dap_urls:
    print('{}.html'.format(url))
```
<div class="output_area"><div class="prompt"></div>
<pre>
    ************************* Filtered DAP *************************
    http://thredds.secoora.org/thredds/dodsC/G1_SST_GLOBAL.nc.html
    http://geoport-dev.whoi.edu/thredds/dodsC/coawst_4/use/fmrc/coawst_4_use_best.ncd.html
    http://www.smast.umassd.edu:8080/thredds/dodsC/FVCOM/NECOFS/Forecasts/NECOFS_GOM3_FORECAST.nc.html
    http://www.smast.umassd.edu:8080/thredds/dodsC/FVCOM/NECOFS/Forecasts/NECOFS_FVCOM_OCEAN_BOSTON_FORECAST.nc.html
    http://oos.soest.hawaii.edu/thredds/dodsC/pacioos/hycom/global.html
    http://oos.soest.hawaii.edu/thredds/dodsC/hioos/satellite/dhw_5km.html
    http://thredds.secoora.org/thredds/dodsC/SECOORA_NCSU_CNAPS.nc.html
    http://www.smast.umassd.edu:8080/thredds/dodsC/FVCOM/NECOFS/Forecasts/NECOFS_FVCOM_OCEAN_MASSBAY_FORECAST.nc.html
    http://www.smast.umassd.edu:8080/thredds/dodsC/FVCOM/NECOFS/Forecasts/NECOFS_FVCOM_OCEAN_SCITUATE_FORECAST.nc.html

</pre>
</div>
Now we can use `pyoos` collectors for `NdbcSos`,

<div class="prompt input_prompt">
In&nbsp;[7]:
</div>

```python
from pyoos.collectors.ndbc.ndbc_sos import NdbcSos

collector_ndbc = NdbcSos()

collector_ndbc.set_bbox(config['region']['bbox'])
collector_ndbc.end_time = config['date']['stop']
collector_ndbc.start_time = config['date']['start']
collector_ndbc.variables = [config['sos_name']]

ofrs = collector_ndbc.server.offerings
title = collector_ndbc.server.identification.title
print(fmt(' NDBC Collector offerings '))
print('{}: {} offerings'.format(title, len(ofrs)))
```
<div class="output_area"><div class="prompt"></div>
<pre>
    ******************* NDBC Collector offerings *******************
    National Data Buoy Center SOS: 988 offerings

</pre>
</div>
<div class="prompt input_prompt">
In&nbsp;[8]:
</div>

```python
import pandas as pd
from ioos_tools.ioos import collector2table

ndbc = collector2table(collector=collector_ndbc,
                       config=config,
                       col='sea_water_temperature (C)')

if ndbc:
    data = dict(
        station_name=[s._metadata.get('station_name') for s in ndbc],
        station_code=[s._metadata.get('station_code') for s in ndbc],
        sensor=[s._metadata.get('sensor') for s in ndbc],
        lon=[s._metadata.get('lon') for s in ndbc],
        lat=[s._metadata.get('lat') for s in ndbc],
        depth=[s._metadata.get('depth') for s in ndbc],
    )

table = pd.DataFrame(data).set_index('station_code')
table
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>depth</th>
      <th>lat</th>
      <th>lon</th>
      <th>sensor</th>
      <th>station_name</th>
    </tr>
    <tr>
      <th>station_code</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>44013</th>
      <td>0.6</td>
      <td>42.346</td>
      <td>-70.651</td>
      <td>urn:ioos:sensor:wmo:44013::watertemp1</td>
      <td>BOSTON 16 NM East of Boston, MA</td>
    </tr>
  </tbody>
</table>
</div>



and `CoopsSos`.

<div class="prompt input_prompt">
In&nbsp;[9]:
</div>

```python
from pyoos.collectors.coops.coops_sos import CoopsSos

collector_coops = CoopsSos()

collector_coops.set_bbox(config['region']['bbox'])
collector_coops.end_time = config['date']['stop']
collector_coops.start_time = config['date']['start']
collector_coops.variables = [config['sos_name']]

ofrs = collector_coops.server.offerings
title = collector_coops.server.identification.title
print(fmt(' Collector offerings '))
print('{}: {} offerings'.format(title, len(ofrs)))
```
<div class="output_area"><div class="prompt"></div>
<pre>
    ********************* Collector offerings **********************
    NOAA.NOS.CO-OPS SOS: 1181 offerings

</pre>
</div>
<div class="prompt input_prompt">
In&nbsp;[10]:
</div>

```python
coops = collector2table(collector=collector_coops,
                        config=config,
                        col='sea_water_temperature (C)')

if coops:
    data = dict(
        station_name=[s._metadata.get('station_name') for s in coops],
        station_code=[s._metadata.get('station_code') for s in coops],
        sensor=[s._metadata.get('sensor') for s in coops],
        lon=[s._metadata.get('lon') for s in coops],
        lat=[s._metadata.get('lat') for s in coops],
        depth=[s._metadata.get('depth') for s in coops],
    )

table = pd.DataFrame(data).set_index('station_code')
table
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>depth</th>
      <th>lat</th>
      <th>lon</th>
      <th>sensor</th>
      <th>station_name</th>
    </tr>
    <tr>
      <th>station_code</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>44013</th>
      <td>0.6</td>
      <td>42.346</td>
      <td>-70.651</td>
      <td>urn:ioos:sensor:wmo:44013::watertemp1</td>
      <td>BOSTON 16 NM East of Boston, MA</td>
    </tr>
  </tbody>
</table>
</div>



We will join all the observations into an uniform series, interpolated to 1-hour interval, for the model-data comparison.

This step is necessary because the observations can be 7 or 10 minutes resolution,
while the models can be 30 to 60 minutes.

<div class="prompt input_prompt">
In&nbsp;[11]:
</div>

```python
data = ndbc + coops

index = pd.date_range(start=config['date']['start'].replace(tzinfo=None),
                      end=config['date']['stop'].replace(tzinfo=None),
                      freq='1H')

# Preserve metadata with `reindex`.
observations = []
for series in data:
    _metadata = series._metadata
    obs = series.reindex(index=index, limit=1, method='nearest')
    obs._metadata = _metadata
    observations.append(obs)
```

In this next cell we will save the data for quicker access later.

<div class="prompt input_prompt">
In&nbsp;[12]:
</div>

```python
import iris
from ioos_tools.tardis import series2cube

attr = dict(
    featureType='timeSeries',
    Conventions='CF-1.6',
    standard_name_vocabulary='CF-1.6',
    cdm_data_type='Station',
    comment='Data from http://opendap.co-ops.nos.noaa.gov'
)


cubes = iris.cube.CubeList(
    [series2cube(obs, attr=attr) for obs in observations]
)

outfile = os.path.join(save_dir, 'OBS_DATA.nc')
iris.save(cubes, outfile)
```

Taking a quick look at the observations:

<div class="prompt input_prompt">
In&nbsp;[13]:
</div>

```python
%matplotlib inline

ax = pd.concat(data).plot(figsize=(11, 2.25))
```


![png](data:image/png;base64,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
6P8BHosFFHtrf94AAAAASUVORK5CYII=
)


Now it is time to loop the models we found above,

<div class="prompt input_prompt">
In&nbsp;[14]:
</div>

```python
from iris.exceptions import (CoordinateNotFoundError, ConstraintMismatchError,
                             MergeError)
from ioos_tools.ioos import get_model_name
from ioos_tools.tardis import quick_load_cubes, proc_cube, is_model, get_surface

print(fmt(' Models '))
cubes = dict()
for k, url in enumerate(dap_urls):
    print('\n[Reading url {}/{}]: {}'.format(k+1, len(dap_urls), url))
    try:
        cube = quick_load_cubes(url, config['cf_names'],
                                callback=None, strict=True)
        if is_model(cube):
            cube = proc_cube(cube,
                             bbox=config['region']['bbox'],
                             time=(config['date']['start'],
                                   config['date']['stop']),
                             units=config['units'])
        else:
            print('[Not model data]: {}'.format(url))
            continue
        cube = get_surface(cube)
        mod_name = get_model_name(url)
        cubes.update({mod_name: cube})
    except (RuntimeError, ValueError,
            ConstraintMismatchError, CoordinateNotFoundError,
            IndexError) as e:
        print('Cannot get cube for: {}\n{}'.format(url, e))
```
<div class="output_area"><div class="prompt"></div>
<pre>
    **************************** Models ****************************
    
    [Reading url 1/9]: http://thredds.secoora.org/thredds/dodsC/G1_SST_GLOBAL.nc
    
    [Reading url 2/9]: http://geoport-dev.whoi.edu/thredds/dodsC/coawst_4/use/fmrc/coawst_4_use_best.ncd
    
    [Reading url 3/9]: http://www.smast.umassd.edu:8080/thredds/dodsC/FVCOM/NECOFS/Forecasts/NECOFS_GOM3_FORECAST.nc
    
    [Reading url 4/9]: http://www.smast.umassd.edu:8080/thredds/dodsC/FVCOM/NECOFS/Forecasts/NECOFS_FVCOM_OCEAN_BOSTON_FORECAST.nc
    
    [Reading url 5/9]: http://oos.soest.hawaii.edu/thredds/dodsC/pacioos/hycom/global
    
    [Reading url 6/9]: http://oos.soest.hawaii.edu/thredds/dodsC/hioos/satellite/dhw_5km
    
    [Reading url 7/9]: http://thredds.secoora.org/thredds/dodsC/SECOORA_NCSU_CNAPS.nc
    
    [Reading url 8/9]: http://www.smast.umassd.edu:8080/thredds/dodsC/FVCOM/NECOFS/Forecasts/NECOFS_FVCOM_OCEAN_MASSBAY_FORECAST.nc
    
    [Reading url 9/9]: http://www.smast.umassd.edu:8080/thredds/dodsC/FVCOM/NECOFS/Forecasts/NECOFS_FVCOM_OCEAN_SCITUATE_FORECAST.nc

</pre>
</div>
Next, we will match them with the nearest observed time-series. The `max_dist=0.08` is in degrees, that is roughly 8 kilometers.

<div class="prompt input_prompt">
In&nbsp;[15]:
</div>

```python
import iris
from iris.pandas import as_series
from ioos_tools.tardis import (make_tree, get_nearest_water,
                               add_station, ensure_timeseries, remove_ssh)

for mod_name, cube in cubes.items():
    fname = '{}.nc'.format(mod_name)
    fname = os.path.join(save_dir, fname)
    print(fmt(' Downloading to file {} '.format(fname)))
    try:
        tree, lon, lat = make_tree(cube)
    except CoordinateNotFoundError as e:
        print('Cannot make KDTree for: {}'.format(mod_name))
        continue
    # Get model series at observed locations.
    raw_series = dict()
    for obs in observations:
        obs = obs._metadata
        station = obs['station_code']
        try:
            kw = dict(k=10, max_dist=0.08, min_var=0.01)
            args = cube, tree, obs['lon'], obs['lat']
            try:
                series, dist, idx = get_nearest_water(*args, **kw)
            except RuntimeError as e:
                print('Cannot download {!r}.\n{}'.format(cube, e))
                series = None
        except ValueError as e:
            status = 'No Data'
            print('[{}] {}'.format(status, obs['station_name']))
            continue
        if not series:
            status = 'Land   '
        else:
            raw_series.update({station: series})
            series = as_series(series)
            status = 'Water  '
        print('[{}] {}'.format(status, obs['station_name']))
    if raw_series:  # Save cube.
        for station, cube in raw_series.items():
            cube = add_station(cube, station)
            cube = remove_ssh(cube)
        try:
            cube = iris.cube.CubeList(raw_series.values()).merge_cube()
        except MergeError as e:
            print(e)
        ensure_timeseries(cube)
        try:
            iris.save(cube, fname)
        except AttributeError:
            # FIXME: we should patch the bad attribute instead of removing everything.
            cube.attributes = {}
            iris.save(cube, fname)
        del cube
    print('Finished processing [{}]'.format(mod_name))
```
<div class="output_area"><div class="prompt"></div>
<pre>
     Downloading to file /home/filipe/IOOS/notebooks_demos/notebooks/latest/G1_SST_GLOBAL.nc 
    [Water  ] BOSTON 16 NM East of Boston, MA
    Finished processing [G1_SST_GLOBAL]
     Downloading to file /home/filipe/IOOS/notebooks_demos/notebooks/latest/fmrc-coawst_4_use_best.nc 
    [Water  ] BOSTON 16 NM East of Boston, MA
    Finished processing [fmrc-coawst_4_use_best]
     Downloading to file /home/filipe/IOOS/notebooks_demos/notebooks/latest/FVCOM_Forecasts-NECOFS_GOM3_FORECAST.nc 
    [Water  ] BOSTON 16 NM East of Boston, MA
    Finished processing [FVCOM_Forecasts-NECOFS_GOM3_FORECAST]
     Downloading to file /home/filipe/IOOS/notebooks_demos/notebooks/latest/Forecasts-NECOFS_FVCOM_OCEAN_BOSTON_FORECAST.nc 
    [No Data] BOSTON 16 NM East of Boston, MA
    Finished processing [Forecasts-NECOFS_FVCOM_OCEAN_BOSTON_FORECAST]
     Downloading to file /home/filipe/IOOS/notebooks_demos/notebooks/latest/pacioos_hycom-global.nc 
    [Water  ] BOSTON 16 NM East of Boston, MA
    Finished processing [pacioos_hycom-global]
     Downloading to file /home/filipe/IOOS/notebooks_demos/notebooks/latest/hioos_satellite-dhw_5km.nc 
    [Water  ] BOSTON 16 NM East of Boston, MA
    Finished processing [hioos_satellite-dhw_5km]
     Downloading to file /home/filipe/IOOS/notebooks_demos/notebooks/latest/SECOORA_NCSU_CNAPS.nc 
    [Water  ] BOSTON 16 NM East of Boston, MA
    Finished processing [SECOORA_NCSU_CNAPS]
     Downloading to file /home/filipe/IOOS/notebooks_demos/notebooks/latest/Forecasts-NECOFS_FVCOM_OCEAN_MASSBAY_FORECAST.nc 
    [Water  ] BOSTON 16 NM East of Boston, MA
    Finished processing [Forecasts-NECOFS_FVCOM_OCEAN_MASSBAY_FORECAST]
     Downloading to file /home/filipe/IOOS/notebooks_demos/notebooks/latest/Forecasts-NECOFS_FVCOM_OCEAN_SCITUATE_FORECAST.nc 
    [No Data] BOSTON 16 NM East of Boston, MA
    Finished processing [Forecasts-NECOFS_FVCOM_OCEAN_SCITUATE_FORECAST]

</pre>
</div>
Now it is possible to compute some simple comparison metrics. First we'll calculate the model mean bias:

$$ \text{MB} = \mathbf{\overline{m}} - \mathbf{\overline{o}}$$

<div class="prompt input_prompt">
In&nbsp;[16]:
</div>

```python
from ioos_tools.ioos import stations_keys


def rename_cols(df, config):
    cols = stations_keys(config, key='station_name')
    return df.rename(columns=cols)
```

<div class="prompt input_prompt">
In&nbsp;[17]:
</div>

```python
from ioos_tools.ioos import load_ncs
from ioos_tools.skill_score import mean_bias, apply_skill

dfs = load_ncs(config)

df = apply_skill(dfs, mean_bias, remove_mean=False, filter_tides=False)
skill_score = dict(mean_bias=df.to_dict())

# Filter out stations with no valid comparison.
df.dropna(how='all', axis=1, inplace=True)
df = df.applymap('{:.2f}'.format).replace('nan', '--')
```

And the root mean squared rrror of the deviations from the mean:
$$ \text{CRMS} = \sqrt{\left(\mathbf{m'} - \mathbf{o'}\right)^2}$$

where: $\mathbf{m'} = \mathbf{m} - \mathbf{\overline{m}}$ and $\mathbf{o'} = \mathbf{o} - \mathbf{\overline{o}}$

<div class="prompt input_prompt">
In&nbsp;[18]:
</div>

```python
from ioos_tools.skill_score import rmse

dfs = load_ncs(config)

df = apply_skill(dfs, rmse, remove_mean=True, filter_tides=False)
skill_score['rmse'] = df.to_dict()

# Filter out stations with no valid comparison.
df.dropna(how='all', axis=1, inplace=True)
df = df.applymap('{:.2f}'.format).replace('nan', '--')
```

The next 2 cells make the scores "pretty" for plotting.

<div class="prompt input_prompt">
In&nbsp;[19]:
</div>

```python
import pandas as pd

# Stringfy keys.
for key in skill_score.keys():
    skill_score[key] = {str(k): v for k, v in skill_score[key].items()}

mean_bias = pd.DataFrame.from_dict(skill_score['mean_bias'])
mean_bias = mean_bias.applymap('{:.2f}'.format).replace('nan', '--')

skill_score = pd.DataFrame.from_dict(skill_score['rmse'])
skill_score = skill_score.applymap('{:.2f}'.format).replace('nan', '--')
```

<div class="prompt input_prompt">
In&nbsp;[20]:
</div>

```python
from ioos_tools.ioos import make_map

bbox = config['region']['bbox']
units = config['units']
run_name = config['run_name']

kw = dict(zoom_start=11, line=True, states=False,
          secoora_stations=False, layers=False)
m = make_map(bbox, **kw)
```

The cells from `[20]` to `[25]` create a [`folium`](https://github.com/python-visualization/folium) map with [`bokeh`](http://bokeh.pydata.org/en/latest/) for the time-series at the observed points.

Note that we did mark the nearest model cell location used in the comparison.

<div class="prompt input_prompt">
In&nbsp;[21]:
</div>

```python
all_obs = stations_keys(config)

from glob import glob
from operator import itemgetter

import iris
import folium
from folium.plugins import MarkerCluster

iris.FUTURE.netcdf_promote = True

big_list = []
for fname in glob(os.path.join(save_dir, '*.nc')):
    if 'OBS_DATA' in fname:
        continue
    cube = iris.load_cube(fname)
    model = os.path.split(fname)[1].split('-')[-1].split('.')[0]
    lons = cube.coord(axis='X').points
    lats = cube.coord(axis='Y').points
    stations = cube.coord('station_code').points
    models = [model]*lons.size
    lista = zip(models, lons.tolist(), lats.tolist(), stations.tolist())
    big_list.extend(lista)

big_list.sort(key=itemgetter(3))
df = pd.DataFrame(big_list, columns=['name', 'lon', 'lat', 'station'])
df.set_index('station', drop=True, inplace=True)
groups = df.groupby(df.index)


locations, popups = [], []
for station, info in groups:
    sta_name = all_obs[station]
    for lat, lon, name in zip(info.lat, info.lon, info.name):
        locations.append([lat, lon])
        popups.append('[{}]: {}'.format(name, sta_name))

MarkerCluster(locations=locations, popups=popups).add_to(m)
```




    <folium.plugins.marker_cluster.MarkerCluster at 0x7f0a715d09e8>



Here we use a dictionary with some models we expect to find so we can create a better legend for the plots. If any new models are found, we will use its filename in the legend as a default until we can go back and add a short name to our library.

<div class="prompt input_prompt">
In&nbsp;[22]:
</div>

```python
titles = {
    'coawst_4_use_best': 'COAWST_4',
    'global': 'HYCOM',
    'NECOFS_GOM3_FORECAST': 'NECOFS_GOM3',
    'NECOFS_FVCOM_OCEAN_MASSBAY_FORECAST': 'NECOFS_MassBay',
    'OBS_DATA': 'Observations'
}
```

<div class="prompt input_prompt">
In&nbsp;[23]:
</div>

```python
from bokeh.resources import CDN
from bokeh.plotting import figure
from bokeh.embed import file_html
from bokeh.models import HoverTool
from itertools import cycle
from bokeh.palettes import Spectral6

from folium import IFrame

# Plot defaults.
colors = Spectral6
colorcycler = cycle(colors)
tools = 'pan,box_zoom,reset'
width, height = 750, 250


def make_plot(df, station):
    p = figure(toolbar_location='above',
               x_axis_type='datetime',
               width=width,
               height=height,
               tools=tools,
               title=str(station))
    for column, series in df.iteritems():
        series.dropna(inplace=True)
        if not series.empty:
            line = p.line(
                x=series.index,
                y=series.values,
                legend='%s' % titles.get(column, column),
                line_color=next(colorcycler),
                line_width=5,
                line_cap='round',
                line_join='round'
            )
            if 'OBS_DATA' not in column:
                bias = mean_bias[str(station)][column]
                skill = skill_score[str(station)][column]
            else:
                skill = bias = 'NA'
            p.add_tools(HoverTool(tooltips=[('Name', '%s' % column),
                                            ('Bias', bias),
                                            ('Skill', skill)],
                                  renderers=[line]))
    return p


def make_marker(p, station):
    lons = stations_keys(config, key='lon')
    lats = stations_keys(config, key='lat')

    lon, lat = lons[station], lats[station]
    html = file_html(p, CDN, station)
    iframe = IFrame(html, width=width+40, height=height+80)

    popup = folium.Popup(iframe, max_width=2650)
    icon = folium.Icon(color='green', icon='stats')
    marker = folium.Marker(location=[lat, lon],
                           popup=popup,
                           icon=icon)
    return marker
```

<div class="prompt input_prompt">
In&nbsp;[24]:
</div>

```python
dfs = load_ncs(config)

for station in dfs:
    sta_name = all_obs[station]
    df = dfs[station]
    if df.empty:
        continue
    p = make_plot(df, station)
    maker = make_marker(p, station)
    maker.add_to(m)

m
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><iframe src="data:text/html;charset=utf-8;base64,PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgPHNjcmlwdD5MX1BSRUZFUl9DQU5WQVMgPSBmYWxzZTsgTF9OT19UT1VDSCA9IGZhbHNlOyBMX0RJU0FCTEVfM0QgPSBmYWxzZTs8L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL3VucGtnLmNvbS9sZWFmbGV0QDEuMS4wL2Rpc3QvbGVhZmxldC5qcyI+PC9zY3JpcHQ+CiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9hamF4Lmdvb2dsZWFwaXMuY29tL2FqYXgvbGlicy9qcXVlcnkvMS4xMS4xL2pxdWVyeS5taW4uanMiPjwvc2NyaXB0PgogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vYm9vdHN0cmFwLzMuMi4wL2pzL2Jvb3RzdHJhcC5taW4uanMiPjwvc2NyaXB0PgogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY2RuanMuY2xvdWRmbGFyZS5jb20vYWpheC9saWJzL0xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLzIuMC4yL2xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9sZWFmbGV0Lm1hcmtlcmNsdXN0ZXIvMS4wLjAvbGVhZmxldC5tYXJrZXJjbHVzdGVyLXNyYy5qcyI+PC9zY3JpcHQ+CiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9jZG5qcy5jbG91ZGZsYXJlLmNvbS9hamF4L2xpYnMvbGVhZmxldC5tYXJrZXJjbHVzdGVyLzEuMC4wL2xlYWZsZXQubWFya2VyY2x1c3Rlci5qcyI+PC9zY3JpcHQ+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vdW5wa2cuY29tL2xlYWZsZXRAMS4xLjAvZGlzdC9sZWFmbGV0LmNzcyIgLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9ib290c3RyYXAvMy4yLjAvY3NzL2Jvb3RzdHJhcC5taW4uY3NzIiAvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLXRoZW1lLm1pbi5jc3MiIC8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vZm9udC1hd2Vzb21lLzQuNi4zL2Nzcy9mb250LWF3ZXNvbWUubWluLmNzcyIgLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9jZG5qcy5jbG91ZGZsYXJlLmNvbS9hamF4L2xpYnMvTGVhZmxldC5hd2Vzb21lLW1hcmtlcnMvMi4wLjIvbGVhZmxldC5hd2Vzb21lLW1hcmtlcnMuY3NzIiAvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9sZWFmbGV0Lm1hcmtlcmNsdXN0ZXIvMS4wLjAvTWFya2VyQ2x1c3Rlci5EZWZhdWx0LmNzcyIgLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9jZG5qcy5jbG91ZGZsYXJlLmNvbS9hamF4L2xpYnMvbGVhZmxldC5tYXJrZXJjbHVzdGVyLzEuMC4wL01hcmtlckNsdXN0ZXIuY3NzIiAvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL3Jhd2dpdC5jb20vcHl0aG9uLXZpc3VhbGl6YXRpb24vZm9saXVtL21hc3Rlci9mb2xpdW0vdGVtcGxhdGVzL2xlYWZsZXQuYXdlc29tZS5yb3RhdGUuY3NzIiAvPgogICAgPHN0eWxlPmh0bWwsIGJvZHkge3dpZHRoOiAxMDAlO2hlaWdodDogMTAwJTttYXJnaW46IDA7cGFkZGluZzogMDt9PC9zdHlsZT4KICAgIDxzdHlsZT4jbWFwIHtwb3NpdGlvbjphYnNvbHV0ZTt0b3A6MDtib3R0b206MDtyaWdodDowO2xlZnQ6MDt9PC9zdHlsZT4KICAgIAogICAgICAgICAgICA8c3R5bGU+ICNtYXBfMTUwMjNmYTIyMjUyNGE2YWJlMTY5MmU4ZDlmMWRmMmIgewogICAgICAgICAgICAgICAgcG9zaXRpb24gOiByZWxhdGl2ZTsKICAgICAgICAgICAgICAgIHdpZHRoIDogMTAwLjAlOwogICAgICAgICAgICAgICAgaGVpZ2h0OiAxMDAuMCU7CiAgICAgICAgICAgICAgICBsZWZ0OiAwLjAlOwogICAgICAgICAgICAgICAgdG9wOiAwLjAlOwogICAgICAgICAgICAgICAgfQogICAgICAgICAgICA8L3N0eWxlPgogICAgICAgIAogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY2RuanMuY2xvdWRmbGFyZS5jb20vYWpheC9saWJzL2xlYWZsZXQubWFya2VyY2x1c3Rlci8xLjAuMC9sZWFmbGV0Lm1hcmtlcmNsdXN0ZXIuanMiPjwvc2NyaXB0PgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9sZWFmbGV0Lm1hcmtlcmNsdXN0ZXIvMS4wLjAvTWFya2VyQ2x1c3Rlci5jc3MiIC8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuanMuY2xvdWRmbGFyZS5jb20vYWpheC9saWJzL2xlYWZsZXQubWFya2VyY2x1c3Rlci8xLjAuMC9NYXJrZXJDbHVzdGVyLkRlZmF1bHQuY3NzIiAvPgo8L2hlYWQ+Cjxib2R5PiAgICAKICAgIAogICAgICAgICAgICA8ZGl2IGNsYXNzPSJmb2xpdW0tbWFwIiBpZD0ibWFwXzE1MDIzZmEyMjI1MjRhNmFiZTE2OTJlOGQ5ZjFkZjJiIiA+PC9kaXY+CiAgICAgICAgCjwvYm9keT4KPHNjcmlwdD4gICAgCiAgICAKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIHNvdXRoV2VzdCA9IEwubGF0TG5nKC05MCwgLTE4MCk7CiAgICAgICAgICAgICAgICB2YXIgbm9ydGhFYXN0ID0gTC5sYXRMbmcoOTAsIDE4MCk7CiAgICAgICAgICAgICAgICB2YXIgYm91bmRzID0gTC5sYXRMbmdCb3VuZHMoc291dGhXZXN0LCBub3J0aEVhc3QpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIHZhciBtYXBfMTUwMjNmYTIyMjUyNGE2YWJlMTY5MmU4ZDlmMWRmMmIgPSBMLm1hcCgKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICdtYXBfMTUwMjNmYTIyMjUyNGE2YWJlMTY5MmU4ZDlmMWRmMmInLAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAge2NlbnRlcjogWzQyLjMzLC03MC45MzVdLAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgem9vbTogMTEsCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBtYXhCb3VuZHM6IGJvdW5kcywKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIGxheWVyczogW10sCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICB3b3JsZENvcHlKdW1wOiBmYWxzZSwKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIGNyczogTC5DUlMuRVBTRzM4NTcKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgfSk7CiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciB0aWxlX2xheWVyX2UyZjllM2Y5NjI4YjQ0MmJiYjJjNGRjODE3NGUxZTQwID0gTC50aWxlTGF5ZXIoCiAgICAgICAgICAgICAgICAnaHR0cHM6Ly97c30udGlsZS5vcGVuc3RyZWV0bWFwLm9yZy97en0ve3h9L3t5fS5wbmcnLAogICAgICAgICAgICAgICAgewogICAgICAgICAgICAgICAgICAgIG1heFpvb206IDE4LAogICAgICAgICAgICAgICAgICAgIG1pblpvb206IDEsCiAgICAgICAgICAgICAgICAgICAgY29udGludW91c1dvcmxkOiBmYWxzZSwKICAgICAgICAgICAgICAgICAgICBub1dyYXA6IGZhbHNlLAogICAgICAgICAgICAgICAgICAgIGF0dHJpYnV0aW9uOiAnRGF0YSBieSA8YSBocmVmPSJodHRwOi8vb3BlbnN0cmVldG1hcC5vcmciPk9wZW5TdHJlZXRNYXA8L2E+LCB1bmRlciA8YSBocmVmPSJodHRwOi8vd3d3Lm9wZW5zdHJlZXRtYXAub3JnL2NvcHlyaWdodCI+T0RiTDwvYT4uJywKICAgICAgICAgICAgICAgICAgICBkZXRlY3RSZXRpbmE6IGZhbHNlLAogICAgICAgICAgICAgICAgICAgIHN1YmRvbWFpbnM6ICdhYmMnCiAgICAgICAgICAgICAgICAgICAgfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfMTUwMjNmYTIyMjUyNGE2YWJlMTY5MmU4ZDlmMWRmMmIpOwoKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFjcm9fZWxlbWVudF8wZWY4ODIzMmYxNmY0ZTc2OGViMDE1NjdkZTFlMWMwNiA9IEwudGlsZUxheWVyLndtcygKICAgICAgICAgICAgICAgICdodHRwOi8vaGZybmV0LnVjc2QuZWR1L3RocmVkZHMvd21zL0hGUk5ldC9VU0VHQy82a20vaG91cmx5L1JUVicsCiAgICAgICAgICAgICAgICB7CiAgICAgICAgICAgICAgICAgICAgbGF5ZXJzOiAnc3VyZmFjZV9zZWFfd2F0ZXJfdmVsb2NpdHknLAogICAgICAgICAgICAgICAgICAgIHN0eWxlczogJycsCiAgICAgICAgICAgICAgICAgICAgZm9ybWF0OiAnaW1hZ2UvcG5nJywKICAgICAgICAgICAgICAgICAgICB0cmFuc3BhcmVudDogdHJ1ZSwKICAgICAgICAgICAgICAgICAgICB2ZXJzaW9uOiAnMS4xLjEnLAogICAgICAgICAgICAgICAgICAgICBhdHRyaWJ1dGlvbjogJ0hGUk5ldCcKICAgICAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF8xNTAyM2ZhMjIyNTI0YTZhYmUxNjkyZThkOWYxZGYyYik7CgogICAgICAgIAogICAgCiAgICAgICAgICAgICAgICB2YXIgcG9seV9saW5lX2FkYmI3YzVkYWU0NjRiNTlhNmYzMmQ0N2MwYjE3YTU4ID0gTC5wb2x5bGluZSgKICAgICAgICAgICAgICAgICAgICBbWzQyLjAzMDAwMDAwMDAwMDAwMSwgLTcxLjI5OTk5OTk5OTk5OTk5N10sIFs0Mi4wMzAwMDAwMDAwMDAwMDEsIC03MC41Njk5OTk5OTk5OTk5OTNdLCBbNDIuNjMwMDAwMDAwMDAwMDAzLCAtNzAuNTY5OTk5OTk5OTk5OTkzXSwgWzQyLjYzMDAwMDAwMDAwMDAwMywgLTcxLjI5OTk5OTk5OTk5OTk5N10sIFs0Mi4wMzAwMDAwMDAwMDAwMDEsIC03MS4yOTk5OTk5OTk5OTk5OTddXSwKICAgICAgICAgICAgICAgICAgICB7CiAgICAgICAgICAgICAgICAgICAgICAgIGNvbG9yOiAnI0ZGMDAwMCcsCiAgICAgICAgICAgICAgICAgICAgICAgIHdlaWdodDogMiwKICAgICAgICAgICAgICAgICAgICAgICAgb3BhY2l0eTogMC45LAogICAgICAgICAgICAgICAgICAgICAgICB9KTsKICAgICAgICAgICAgICAgIG1hcF8xNTAyM2ZhMjIyNTI0YTZhYmUxNjkyZThkOWYxZGYyYi5hZGRMYXllcihwb2x5X2xpbmVfYWRiYjdjNWRhZTQ2NGI1OWE2ZjMyZDQ3YzBiMTdhNTgpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgbGF5ZXJfY29udHJvbF9lNzBiOWUzZGM4NGE0MzA2YTQ0Nzk0MjBkYzY1ZTY4NSA9IHsKICAgICAgICAgICAgICAgIGJhc2VfbGF5ZXJzIDogeyAib3BlbnN0cmVldG1hcCIgOiB0aWxlX2xheWVyX2UyZjllM2Y5NjI4YjQ0MmJiYjJjNGRjODE3NGUxZTQwLCB9LAogICAgICAgICAgICAgICAgb3ZlcmxheXMgOiB7ICJIRiBSYWRhciIgOiBtYWNyb19lbGVtZW50XzBlZjg4MjMyZjE2ZjRlNzY4ZWIwMTU2N2RlMWUxYzA2LCB9CiAgICAgICAgICAgICAgICB9OwogICAgICAgICAgICBMLmNvbnRyb2wubGF5ZXJzKAogICAgICAgICAgICAgICAgbGF5ZXJfY29udHJvbF9lNzBiOWUzZGM4NGE0MzA2YTQ0Nzk0MjBkYzY1ZTY4NS5iYXNlX2xheWVycywKICAgICAgICAgICAgICAgIGxheWVyX2NvbnRyb2xfZTcwYjllM2RjODRhNDMwNmE0NDc5NDIwZGM2NWU2ODUub3ZlcmxheXMsCiAgICAgICAgICAgICAgICB7cG9zaXRpb246ICd0b3ByaWdodCcsCiAgICAgICAgICAgICAgICAgY29sbGFwc2VkOiB0cnVlLAogICAgICAgICAgICAgICAgIGF1dG9aSW5kZXg6IHRydWUKICAgICAgICAgICAgICAgIH0pLmFkZFRvKG1hcF8xNTAyM2ZhMjIyNTI0YTZhYmUxNjkyZThkOWYxZGYyYik7CiAgICAgICAgCiAgICAKICAgICAgICAgICAgICAgIHZhciBtYXJrZXJfY2x1c3Rlcl8zYWMwODYyYzA1Mzg0MzgzYmUyYjFhODljMzAwMjA2ZCA9IEwubWFya2VyQ2x1c3Rlckdyb3VwKCk7CiAgICAgICAgICAgICAgICBtYXBfMTUwMjNmYTIyMjUyNGE2YWJlMTY5MmU4ZDlmMWRmMmIuYWRkTGF5ZXIobWFya2VyX2NsdXN0ZXJfM2FjMDg2MmMwNTM4NDM4M2JlMmIxYTg5YzMwMDIwNmQpOwogICAgICAgICAgICAKICAgIAoKICAgICAgICAgICAgdmFyIG1hcmtlcl84YzE0NGQ2NDZkY2E0NTQzODhkYTE0NjVjNTMyN2Q5ZCA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzQyLjMyNTAwNDU3NzYsLTcwLjY3NDk5NTQyMjRdLAogICAgICAgICAgICAgICAgewogICAgICAgICAgICAgICAgICAgIGljb246IG5ldyBMLkljb24uRGVmYXVsdCgpCiAgICAgICAgICAgICAgICAgICAgfQogICAgICAgICAgICAgICAgKQogICAgICAgICAgICAgICAgLmFkZFRvKG1hcmtlcl9jbHVzdGVyXzNhYzA4NjJjMDUzODQzODNiZTJiMWE4OWMzMDAyMDZkKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzA4YjA2Y2VhMWYzZDRmOTZiNGU5ZjI3MjVmZjgxODI0ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzU1ZjQ4ZWI2MWE4ODQ4ZWVhYWY2MzY2MTdlMmY0ODc5ID0gJCgnPGRpdiBpZD0iaHRtbF81NWY0OGViNjFhODg0OGVlYWFmNjM2NjE3ZTJmNDg3OSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+W2Rod181a21dOiBCT1NUT04gMTYgTk0gRWFzdCBvZiBCb3N0b24sIE1BPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF8wOGIwNmNlYTFmM2Q0Zjk2YjRlOWYyNzI1ZmY4MTgyNC5zZXRDb250ZW50KGh0bWxfNTVmNDhlYjYxYTg4NDhlZWFhZjYzNjYxN2UyZjQ4NzkpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIG1hcmtlcl84YzE0NGQ2NDZkY2E0NTQzODhkYTE0NjVjNTMyN2Q5ZC5iaW5kUG9wdXAocG9wdXBfMDhiMDZjZWExZjNkNGY5NmI0ZTlmMjcyNWZmODE4MjQpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKCiAgICAgICAgICAgIHZhciBtYXJrZXJfYjc1M2M0YzllZGM1NDBhYTlkYzQzZjE4NDAxY2ZlYmEgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFs0Mi4zNDUwMDEyMjA3LC03MC42NTQ5OTg3NzkzXSwKICAgICAgICAgICAgICAgIHsKICAgICAgICAgICAgICAgICAgICBpY29uOiBuZXcgTC5JY29uLkRlZmF1bHQoKQogICAgICAgICAgICAgICAgICAgIH0KICAgICAgICAgICAgICAgICkKICAgICAgICAgICAgICAgIC5hZGRUbyhtYXJrZXJfY2x1c3Rlcl8zYWMwODYyYzA1Mzg0MzgzYmUyYjFhODljMzAwMjA2ZCk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF83YzU0NzAyOWI5MGY0OWNkODA3ZTM0YzRkNzBkNTAwYSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF81ZGIzYmM1YjQ3YzM0NmY5OWMwNDE4YjdiMzgyNmQ3OCA9ICQoJzxkaXYgaWQ9Imh0bWxfNWRiM2JjNWI0N2MzNDZmOTljMDQxOGI3YjM4MjZkNzgiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPltHMV9TU1RfR0xPQkFMXTogQk9TVE9OIDE2IE5NIEVhc3Qgb2YgQm9zdG9uLCBNQTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfN2M1NDcwMjliOTBmNDljZDgwN2UzNGM0ZDcwZDUwMGEuc2V0Q29udGVudChodG1sXzVkYjNiYzViNDdjMzQ2Zjk5YzA0MThiN2IzODI2ZDc4KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBtYXJrZXJfYjc1M2M0YzllZGM1NDBhYTlkYzQzZjE4NDAxY2ZlYmEuYmluZFBvcHVwKHBvcHVwXzdjNTQ3MDI5YjkwZjQ5Y2Q4MDdlMzRjNGQ3MGQ1MDBhKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCgogICAgICAgICAgICB2YXIgbWFya2VyXzBjZjliMjFlZDZkYTQ5NTRiOTlmMGY5ZTBkZjA3YzFjID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbNDIuMzI0OCwtNzAuNjM5OTddLAogICAgICAgICAgICAgICAgewogICAgICAgICAgICAgICAgICAgIGljb246IG5ldyBMLkljb24uRGVmYXVsdCgpCiAgICAgICAgICAgICAgICAgICAgfQogICAgICAgICAgICAgICAgKQogICAgICAgICAgICAgICAgLmFkZFRvKG1hcmtlcl9jbHVzdGVyXzNhYzA4NjJjMDUzODQzODNiZTJiMWE4OWMzMDAyMDZkKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2UwZDFmYmM4YWVjNDRkOTA5MjE0ZTI1MTExZjZjZjdjID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sX2U0MzI3NjdhYzJkYjQzNDJhNjc1MDgwNGQ0ZjFjODEzID0gJCgnPGRpdiBpZD0iaHRtbF9lNDMyNzY3YWMyZGI0MzQyYTY3NTA4MDRkNGYxYzgxMyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+W2dsb2JhbF06IEJPU1RPTiAxNiBOTSBFYXN0IG9mIEJvc3RvbiwgTUE8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2UwZDFmYmM4YWVjNDRkOTA5MjE0ZTI1MTExZjZjZjdjLnNldENvbnRlbnQoaHRtbF9lNDMyNzY3YWMyZGI0MzQyYTY3NTA4MDRkNGYxYzgxMyk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgbWFya2VyXzBjZjliMjFlZDZkYTQ5NTRiOTlmMGY5ZTBkZjA3YzFjLmJpbmRQb3B1cChwb3B1cF9lMGQxZmJjOGFlYzQ0ZDkwOTIxNGUyNTExMWY2Y2Y3Yyk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAoKICAgICAgICAgICAgdmFyIG1hcmtlcl9jNmFkMDZlMjgzOTE0NzcwODIxMjg2YjE3NjA3MDY1ZCA9IEwubWFya2VyKAogICAgICAgICAgICAgICAgWzQyLjM0MTMyNzY2NzIsLTcwLjY0ODMxNTQyOTddLAogICAgICAgICAgICAgICAgewogICAgICAgICAgICAgICAgICAgIGljb246IG5ldyBMLkljb24uRGVmYXVsdCgpCiAgICAgICAgICAgICAgICAgICAgfQogICAgICAgICAgICAgICAgKQogICAgICAgICAgICAgICAgLmFkZFRvKG1hcmtlcl9jbHVzdGVyXzNhYzA4NjJjMDUzODQzODNiZTJiMWE4OWMzMDAyMDZkKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2U0MTNjZGU4OWEzZDRlNTdiNDQxNDQ4OGZjZTE5M2Q5ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzA5NTU5MjY0ODk2YzRiMmU4Yjk1OTM5N2QzOTUzYjZiID0gJCgnPGRpdiBpZD0iaHRtbF8wOTU1OTI2NDg5NmM0YjJlOGI5NTkzOTdkMzk1M2I2YiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+W05FQ09GU19GVkNPTV9PQ0VBTl9NQVNTQkFZX0ZPUkVDQVNUXTogQk9TVE9OIDE2IE5NIEVhc3Qgb2YgQm9zdG9uLCBNQTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfZTQxM2NkZTg5YTNkNGU1N2I0NDE0NDg4ZmNlMTkzZDkuc2V0Q29udGVudChodG1sXzA5NTU5MjY0ODk2YzRiMmU4Yjk1OTM5N2QzOTUzYjZiKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBtYXJrZXJfYzZhZDA2ZTI4MzkxNDc3MDgyMTI4NmIxNzYwNzA2NWQuYmluZFBvcHVwKHBvcHVwX2U0MTNjZGU4OWEzZDRlNTdiNDQxNDQ4OGZjZTE5M2Q5KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCgogICAgICAgICAgICB2YXIgbWFya2VyXzRhMGM4OTgwOGQxMjQ2MWFiZTg4MmM2MmFkZjg2NjliID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbNDIuMzQzNzg0MzMyMywtNzAuNjUyNDU4MTkwOV0sCiAgICAgICAgICAgICAgICB7CiAgICAgICAgICAgICAgICAgICAgaWNvbjogbmV3IEwuSWNvbi5EZWZhdWx0KCkKICAgICAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICAgICApCiAgICAgICAgICAgICAgICAuYWRkVG8obWFya2VyX2NsdXN0ZXJfM2FjMDg2MmMwNTM4NDM4M2JlMmIxYTg5YzMwMDIwNmQpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfZjY3NmRkNmUwZTk0NGI4ODhjNGVkMzNmNzk1MjlhNzMgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfY2QwYjg3NmI5NjdlNGI2Zjk3Yjk3M2I2ZmNlZjFmNDUgPSAkKCc8ZGl2IGlkPSJodG1sX2NkMGI4NzZiOTY3ZTRiNmY5N2I5NzNiNmZjZWYxZjQ1IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5bTkVDT0ZTX0dPTTNfRk9SRUNBU1RdOiBCT1NUT04gMTYgTk0gRWFzdCBvZiBCb3N0b24sIE1BPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9mNjc2ZGQ2ZTBlOTQ0Yjg4OGM0ZWQzM2Y3OTUyOWE3My5zZXRDb250ZW50KGh0bWxfY2QwYjg3NmI5NjdlNGI2Zjk3Yjk3M2I2ZmNlZjFmNDUpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIG1hcmtlcl80YTBjODk4MDhkMTI0NjFhYmU4ODJjNjJhZGY4NjY5Yi5iaW5kUG9wdXAocG9wdXBfZjY3NmRkNmUwZTk0NGI4ODhjNGVkMzNmNzk1MjlhNzMpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKCiAgICAgICAgICAgIHZhciBtYXJrZXJfMzQzOWNjM2Y4MmYwNDc1OTg2NGIyMWJjNDEyY2MyZmMgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFs0Mi4zNTM3MDI4MzA0LC03MC42NDE0MDI3MTQ4XSwKICAgICAgICAgICAgICAgIHsKICAgICAgICAgICAgICAgICAgICBpY29uOiBuZXcgTC5JY29uLkRlZmF1bHQoKQogICAgICAgICAgICAgICAgICAgIH0KICAgICAgICAgICAgICAgICkKICAgICAgICAgICAgICAgIC5hZGRUbyhtYXJrZXJfY2x1c3Rlcl8zYWMwODYyYzA1Mzg0MzgzYmUyYjFhODljMzAwMjA2ZCk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF81OTRhOTFkMGRkYmM0MjI1YjhlOWQzOTdjMzkyZWMxMCA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9kMzMzOTAzNjg3NGM0OGFmODBjYjYyYTIzMGM5NzQxOCA9ICQoJzxkaXYgaWQ9Imh0bWxfZDMzMzkwMzY4NzRjNDhhZjgwY2I2MmEyMzBjOTc0MTgiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPltTRUNPT1JBX05DU1VfQ05BUFNdOiBCT1NUT04gMTYgTk0gRWFzdCBvZiBCb3N0b24sIE1BPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF81OTRhOTFkMGRkYmM0MjI1YjhlOWQzOTdjMzkyZWMxMC5zZXRDb250ZW50KGh0bWxfZDMzMzkwMzY4NzRjNDhhZjgwY2I2MmEyMzBjOTc0MTgpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIG1hcmtlcl8zNDM5Y2MzZjgyZjA0NzU5ODY0YjIxYmM0MTJjYzJmYy5iaW5kUG9wdXAocG9wdXBfNTk0YTkxZDBkZGJjNDIyNWI4ZTlkMzk3YzM5MmVjMTApOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKCiAgICAgICAgICAgIHZhciBtYXJrZXJfNTVkOTc4ZmNjYWMxNDM4M2E0MmI2NDIxOGQwMjhiOTUgPSBMLm1hcmtlcigKICAgICAgICAgICAgICAgIFs0Mi4zNDg5NzIxMjYsLTcwLjY1NTEwNjEzOTRdLAogICAgICAgICAgICAgICAgewogICAgICAgICAgICAgICAgICAgIGljb246IG5ldyBMLkljb24uRGVmYXVsdCgpCiAgICAgICAgICAgICAgICAgICAgfQogICAgICAgICAgICAgICAgKQogICAgICAgICAgICAgICAgLmFkZFRvKG1hcmtlcl9jbHVzdGVyXzNhYzA4NjJjMDUzODQzODNiZTJiMWE4OWMzMDAyMDZkKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzk1ZDcxNWE3ZDk5ODRiODI4NTdmYzVmYmZiYmIxODNkID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzQ4MjJiZGVhNjIzZjQ0OTg5NjhkYTQ1N2JhNTY4MjNkID0gJCgnPGRpdiBpZD0iaHRtbF80ODIyYmRlYTYyM2Y0NDk4OTY4ZGE0NTdiYTU2ODIzZCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+W2NvYXdzdF80X3VzZV9iZXN0XTogQk9TVE9OIDE2IE5NIEVhc3Qgb2YgQm9zdG9uLCBNQTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfOTVkNzE1YTdkOTk4NGI4Mjg1N2ZjNWZiZmJiYjE4M2Quc2V0Q29udGVudChodG1sXzQ4MjJiZGVhNjIzZjQ0OTg5NjhkYTQ1N2JhNTY4MjNkKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBtYXJrZXJfNTVkOTc4ZmNjYWMxNDM4M2E0MmI2NDIxOGQwMjhiOTUuYmluZFBvcHVwKHBvcHVwXzk1ZDcxNWE3ZDk5ODRiODI4NTdmYzVmYmZiYmIxODNkKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCgogICAgICAgICAgICB2YXIgbWFya2VyXzY1NTRjYTlmODQ2MjQ5NmRiOGQwMjE0MzQ2YmVlMWZkID0gTC5tYXJrZXIoCiAgICAgICAgICAgICAgICBbNDIuMzQ2LC03MC42NTFdLAogICAgICAgICAgICAgICAgewogICAgICAgICAgICAgICAgICAgIGljb246IG5ldyBMLkljb24uRGVmYXVsdCgpCiAgICAgICAgICAgICAgICAgICAgfQogICAgICAgICAgICAgICAgKQogICAgICAgICAgICAgICAgLmFkZFRvKG1hcF8xNTAyM2ZhMjIyNTI0YTZhYmUxNjkyZThkOWYxZGYyYik7CiAgICAgICAgICAgIAogICAgCgogICAgICAgICAgICAgICAgdmFyIGljb25fYzBhZmMxY2YzNTgwNGY3NDhmNjMyYjQ5OGU2MTk5MmYgPSBMLkF3ZXNvbWVNYXJrZXJzLmljb24oewogICAgICAgICAgICAgICAgICAgIGljb246ICdzdGF0cycsCiAgICAgICAgICAgICAgICAgICAgaWNvbkNvbG9yOiAnd2hpdGUnLAogICAgICAgICAgICAgICAgICAgIG1hcmtlckNvbG9yOiAnZ3JlZW4nLAogICAgICAgICAgICAgICAgICAgIHByZWZpeDogJ2dseXBoaWNvbicsCiAgICAgICAgICAgICAgICAgICAgZXh0cmFDbGFzc2VzOiAnZmEtcm90YXRlLTAnCiAgICAgICAgICAgICAgICAgICAgfSk7CiAgICAgICAgICAgICAgICBtYXJrZXJfNjU1NGNhOWY4NDYyNDk2ZGI4ZDAyMTQzNDZiZWUxZmQuc2V0SWNvbihpY29uX2MwYWZjMWNmMzU4MDRmNzQ4ZjYzMmI0OThlNjE5OTJmKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzg0MmI4OGJkY2M5YzQxMjRhYTk1ZWViNzk5MTQyNTFlID0gTC5wb3B1cCh7bWF4V2lkdGg6ICcyNjUwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaV9mcmFtZV8wOTI4NzFkMGZmNTY0OWRhOTRlYWFjZTQ1NWQyODk1NCA9ICQoJzxpZnJhbWUgc3JjPSJkYXRhOnRleHQvaHRtbDtjaGFyc2V0PXV0Zi04O2Jhc2U2NCxDaUFnSUNBS1BDRkVUME5VV1ZCRklHaDBiV3crQ2p4b2RHMXNJR3hoYm1jOUltVnVJajRLSUNBZ0lEeG9aV0ZrUGdvZ0lDQWdJQ0FnSUR4dFpYUmhJR05vWVhKelpYUTlJblYwWmkwNElqNEtJQ0FnSUNBZ0lDQThkR2wwYkdVK05EUXdNVE04TDNScGRHeGxQZ29nSUNBZ0lDQWdJQW84YkdsdWF5QnlaV3c5SW5OMGVXeGxjMmhsWlhRaUlHaHlaV1k5SW1oMGRIQnpPaTh2WTJSdUxuQjVaR0YwWVM1dmNtY3ZZbTlyWldndmNtVnNaV0Z6WlM5aWIydGxhQzB3TGpFeUxqWXViV2x1TG1OemN5SWdkSGx3WlQwaWRHVjRkQzlqYzNNaUlDOCtDaUFnSUNBZ0lDQWdDanh6WTNKcGNIUWdkSGx3WlQwaWRHVjRkQzlxWVhaaGMyTnlhWEIwSWlCemNtTTlJbWgwZEhCek9pOHZZMlJ1TG5CNVpHRjBZUzV2Y21jdlltOXJaV2d2Y21Wc1pXRnpaUzlpYjJ0bGFDMHdMakV5TGpZdWJXbHVMbXB6SWo0OEwzTmpjbWx3ZEQ0S1BITmpjbWx3ZENCMGVYQmxQU0owWlhoMEwycGhkbUZ6WTNKcGNIUWlQZ29nSUNBZ1FtOXJaV2d1YzJWMFgyeHZaMTlzWlhabGJDZ2lhVzVtYnlJcE93bzhMM05qY21sd2RENEtJQ0FnSUNBZ0lDQThjM1I1YkdVK0NpQWdJQ0FnSUNBZ0lDQm9kRzFzSUhzS0lDQWdJQ0FnSUNBZ0lDQWdkMmxrZEdnNklERXdNQ1U3Q2lBZ0lDQWdJQ0FnSUNBZ0lHaGxhV2RvZERvZ01UQXdKVHNLSUNBZ0lDQWdJQ0FnSUgwS0lDQWdJQ0FnSUNBZ0lHSnZaSGtnZXdvZ0lDQWdJQ0FnSUNBZ0lDQjNhV1IwYURvZ09UQWxPd29nSUNBZ0lDQWdJQ0FnSUNCb1pXbG5hSFE2SURFd01DVTdDaUFnSUNBZ0lDQWdJQ0FnSUcxaGNtZHBiam9nWVhWMGJ6c0tJQ0FnSUNBZ0lDQWdJSDBLSUNBZ0lDQWdJQ0E4TDNOMGVXeGxQZ29nSUNBZ1BDOW9aV0ZrUGdvZ0lDQWdQR0p2WkhrK0NpQWdJQ0FnSUNBZ0NpQWdJQ0FnSUNBZ1BHUnBkaUJqYkdGemN6MGlZbXN0Y205dmRDSStDaUFnSUNBZ0lDQWdJQ0FnSUR4a2FYWWdZMnhoYzNNOUltSnJMWEJzYjNSa2FYWWlJR2xrUFNJeE9EaGlNRGd4T0MweE9XRmlMVFF6WXpndFlXVXlaQzFoT1dFNFkyRXhPR00wTXpNaVBqd3ZaR2wyUGdvZ0lDQWdJQ0FnSUR3dlpHbDJQZ29nSUNBZ0lDQWdJQW9nSUNBZ0lDQWdJRHh6WTNKcGNIUWdkSGx3WlQwaWRHVjRkQzlxWVhaaGMyTnlhWEIwSWo0S0lDQWdJQ0FnSUNBZ0lDQWdLR1oxYm1OMGFXOXVLQ2tnZXdvZ0lDQWdJQ0FnSUNBZ2RtRnlJR1p1SUQwZ1puVnVZM1JwYjI0b0tTQjdDaUFnSUNBZ0lDQWdJQ0FnSUVKdmEyVm9Mbk5oWm1Wc2VTaG1kVzVqZEdsdmJpZ3BJSHNLSUNBZ0lDQWdJQ0FnSUNBZ0lDQjJZWElnWkc5amMxOXFjMjl1SUQwZ2V5SmtZVEUxTTJVMVpTMHhOVGsyTFRSa04yUXRPVGRqTkMweVlUSXlOV0U1WkRnM09XVWlPbnNpY205dmRITWlPbnNpY21WbVpYSmxibU5sY3lJNlczc2lZWFIwY21saWRYUmxjeUk2ZXlKaVpXeHZkeUk2VzNzaWFXUWlPaUpsTWpWaE0yTmtNeTFoWXpCaUxUUTFZbVF0T0RnM05TMDVOR05tTVRBMllUQTJORElpTENKMGVYQmxJam9pUkdGMFpYUnBiV1ZCZUdsekluMWRMQ0pzWldaMElqcGJleUpwWkNJNkltSXhZVGN5WW1Nd0xXTmxNalV0TkRSbE5DMWlNMkV4TFROallURTFZemc0WlRneE5TSXNJblI1Y0dVaU9pSk1hVzVsWVhKQmVHbHpJbjFkTENKd2JHOTBYMmhsYVdkb2RDSTZNalV3TENKd2JHOTBYM2RwWkhSb0lqbzNOVEFzSW5KbGJtUmxjbVZ5Y3lJNlczc2lhV1FpT2lKbE1qVmhNMk5rTXkxaFl6QmlMVFExWW1RdE9EZzNOUzA1TkdObU1UQTJZVEEyTkRJaUxDSjBlWEJsSWpvaVJHRjBaWFJwYldWQmVHbHpJbjBzZXlKcFpDSTZJakppTTJZelpESTNMVFl6WlRRdE5HVmtZaTA0TkdOa0xXRTNabVF6TWpNeVlXRTJOU0lzSW5SNWNHVWlPaUpIY21sa0luMHNleUpwWkNJNkltSXhZVGN5WW1Nd0xXTmxNalV0TkRSbE5DMWlNMkV4TFROallURTFZemc0WlRneE5TSXNJblI1Y0dVaU9pSk1hVzVsWVhKQmVHbHpJbjBzZXlKcFpDSTZJak0zTURobE1qbGpMVGhoTWpRdE5EZ3paQzA1TjJReExXWmpNMlEwWXpZd05tUmlPU0lzSW5SNWNHVWlPaUpIY21sa0luMHNleUpwWkNJNkltTmtNREUzWm1FeExUTXhNVGd0TkRnNE5DMWhPVEprTFRBM1lqWTRaR1F4WVdNeVpTSXNJblI1Y0dVaU9pSkNiM2hCYm01dmRHRjBhVzl1SW4wc2V5SnBaQ0k2SW1VMk5EY3pZbUl3TFRSbE5HSXROREppWkMxaU9HRTFMVGRtTlRBd1ltRmxNV0V6TlNJc0luUjVjR1VpT2lKTVpXZGxibVFpZlN4N0ltbGtJam9pTWpCaVl6QmlPREl0WXpWak5TMDBaR1V3TFRoa1ptTXRPVGczTkRSaVl6bGxNak0wSWl3aWRIbHdaU0k2SWtkc2VYQm9VbVZ1WkdWeVpYSWlmU3g3SW1sa0lqb2lOalJtWW1aalpEY3ROMlU0WmkwMFlUQmpMVGs1TjJNdE56QmxNMlk1TkdVek1tWmlJaXdpZEhsd1pTSTZJa2RzZVhCb1VtVnVaR1Z5WlhJaWZTeDdJbWxrSWpvaU4yTXlaR1EyTTJFdFlUa3lZUzAwWXpBd0xXRmpNVGN0TUdVMFpqZzBZbUkzWmpjMUlpd2lkSGx3WlNJNklrZHNlWEJvVW1WdVpHVnlaWElpZlN4N0ltbGtJam9pWlRrd1l6WmxNRFl0TkRreU5DMDBOREV4TFdFek5qUXROVFF4WWprMlpUQTFZVEJqSWl3aWRIbHdaU0k2SWtkc2VYQm9VbVZ1WkdWeVpYSWlmU3g3SW1sa0lqb2lZVEE0WmpsaU1XVXRZekExWkMwME5HSmlMV0kwT1RjdFlUYzNabVZsTlRkaE9UUmxJaXdpZEhsd1pTSTZJa2RzZVhCb1VtVnVaR1Z5WlhJaWZTeDdJbWxrSWpvaU9EbGpOalU1TmpBdFpqaGlPQzAwWkRobUxUaGhNbVl0TWprMU9EZzROR0kyWVRjeElpd2lkSGx3WlNJNklrZHNlWEJvVW1WdVpHVnlaWElpZlN4N0ltbGtJam9pT1RSak5tSmtPREl0WmpJM1ppMDBOemMwTFRrMVlUQXRPR013TjJKak1HWTVZelUxSWl3aWRIbHdaU0k2SWtkc2VYQm9VbVZ1WkdWeVpYSWlmU3g3SW1sa0lqb2lNRFk0TW1VNE5Ua3RaakEzTkMwME5EWTRMV0V6WXpJdE5EZzFaV0l5TldJellqRmtJaXdpZEhsd1pTSTZJa2RzZVhCb1VtVnVaR1Z5WlhJaWZWMHNJblJwZEd4bElqcDdJbWxrSWpvaVlUWmlNekU0TkRNdFl6ZzJNeTAwTXpNekxXRXdNVEl0WldWak5XUTNZVEF3WlRnNUlpd2lkSGx3WlNJNklsUnBkR3hsSW4wc0luUnZiMnhmWlhabGJuUnpJanA3SW1sa0lqb2lNR1F3T0RZM09XVXROMll6TkMwME5qSmpMVGxrTmpFdFpqSmhOak5oTlRka1lXWTBJaXdpZEhsd1pTSTZJbFJ2YjJ4RmRtVnVkSE1pZlN3aWRHOXZiR0poY2lJNmV5SnBaQ0k2SWpFMFlUTmxZMkV5TFdFM01qQXROREl3TlMxaE1qRTRMVFpqT0RVMk16YzNZalF6TXlJc0luUjVjR1VpT2lKVWIyOXNZbUZ5SW4wc0luUnZiMnhpWVhKZmJHOWpZWFJwYjI0aU9pSmhZbTkyWlNJc0luaGZjbUZ1WjJVaU9uc2lhV1FpT2lKak56a3pNV1V4TVMwMk5XRTBMVFF5TUdZdE9USmpaaTFpTmpBeFlXSTRNalkyTmpVaUxDSjBlWEJsSWpvaVJHRjBZVkpoYm1kbE1XUWlmU3dpZUY5elkyRnNaU0k2ZXlKcFpDSTZJamhrWldJMk1XSTNMV05tWTJZdE5HWmpOUzFpTUdVM0xUVmpORGMzWWpSbVpEZGpaU0lzSW5SNWNHVWlPaUpNYVc1bFlYSlRZMkZzWlNKOUxDSjVYM0poYm1kbElqcDdJbWxrSWpvaU16STVNR00xTW1NdE1HSTJaUzAwWkdSakxXRTFORGN0WmprMk5qY3dZelpsWldJMklpd2lkSGx3WlNJNklrUmhkR0ZTWVc1blpURmtJbjBzSW5sZmMyTmhiR1VpT25zaWFXUWlPaUl4WldFMU56Y3lNUzFpTnpFNUxUUTJNV010T1RoaU5pMHdPRFEzTWpoaFltSXdabUlpTENKMGVYQmxJam9pVEdsdVpXRnlVMk5oYkdVaWZYMHNJbWxrSWpvaVlqVTJZalUwTm1FdFpEUmpaQzAwWWpBMExXRm1Zakl0WlRBd09XSTBNMlJtWldSaklpd2ljM1ZpZEhsd1pTSTZJa1pwWjNWeVpTSXNJblI1Y0dVaU9pSlFiRzkwSW4wc2V5SmhkSFJ5YVdKMWRHVnpJanA3SW1KaGMyVWlPall3TENKdFlXNTBhWE56WVhNaU9sc3hMRElzTlN3eE1Dd3hOU3d5TUN3ek1GMHNJbTFoZUY5cGJuUmxjblpoYkNJNk1UZ3dNREF3TUM0d0xDSnRhVzVmYVc1MFpYSjJZV3dpT2pFd01EQXVNQ3dpYm5WdFgyMXBibTl5WDNScFkydHpJam93ZlN3aWFXUWlPaUptWVRneE5tUTVaaTB4TldSaUxUUXhaak10T0daaU55MWlNelk1WXpCaE4ySTVOR1lpTENKMGVYQmxJam9pUVdSaGNIUnBkbVZVYVdOclpYSWlmU3g3SW1GMGRISnBZblYwWlhNaU9uc2laR0YwWVY5emIzVnlZMlVpT25zaWFXUWlPaUpqTm1RNE5HUTRPUzA0TUdWakxUUXhOREF0T1dKak1pMW1OR0kwTlRJMlpEazNZekVpTENKMGVYQmxJam9pUTI5c2RXMXVSR0YwWVZOdmRYSmpaU0o5TENKbmJIbHdhQ0k2ZXlKcFpDSTZJbUkyTlRkaU9ESXpMV0kzTnpJdE5HWTBNeTA1WXpRMUxXWXpNR1V5WkdVeFpHRTFOaUlzSW5SNWNHVWlPaUpNYVc1bEluMHNJbWh2ZG1WeVgyZHNlWEJvSWpwdWRXeHNMQ0p0ZFhSbFpGOW5iSGx3YUNJNmJuVnNiQ3dpYm05dWMyVnNaV04wYVc5dVgyZHNlWEJvSWpwN0ltbGtJam9pTWpGbE5qQTJZall0TWpWaE5DMDBZbVppTFdKaE4ySXROVFV4T1RrNVpEUXdZVGcwSWl3aWRIbHdaU0k2SWt4cGJtVWlmU3dpYzJWc1pXTjBhVzl1WDJkc2VYQm9JanB1ZFd4c2ZTd2lhV1FpT2lJMk5HWmlabU5rTnkwM1pUaG1MVFJoTUdNdE9UazNZeTAzTUdVelpqazBaVE15Wm1JaUxDSjBlWEJsSWpvaVIyeDVjR2hTWlc1a1pYSmxjaUo5TEhzaVlYUjBjbWxpZFhSbGN5STZleUp0WVhoZmFXNTBaWEoyWVd3aU9qVXdNQzR3TENKdWRXMWZiV2x1YjNKZmRHbGphM01pT2pCOUxDSnBaQ0k2SWpkallUSXpabUZpTFRWa1pHSXROR1l5TkMwNVpEUmlMV1UzTVRKaU1UaG1ZbUkzTXlJc0luUjVjR1VpT2lKQlpHRndkR2wyWlZScFkydGxjaUo5TEhzaVlYUjBjbWxpZFhSbGN5STZleUpzWVdKbGJDSTZleUoyWVd4MVpTSTZJazVGUTA5R1UxOUhUMDB6SW4wc0luSmxibVJsY21WeWN5STZXM3NpYVdRaU9pSTNZekprWkRZellTMWhPVEpoTFRSak1EQXRZV014Tnkwd1pUUm1PRFJpWWpkbU56VWlMQ0owZVhCbElqb2lSMng1Y0doU1pXNWtaWEpsY2lKOVhYMHNJbWxrSWpvaVltSTNOR0UxWlRZdE9HSmhZeTAwTkRKbExUaGhOak10TURVd1pqQTROVFE0WTJKaElpd2lkSGx3WlNJNklreGxaMlZ1WkVsMFpXMGlmU3g3SW1GMGRISnBZblYwWlhNaU9uc2lZV04wYVhabFgyUnlZV2NpT2lKaGRYUnZJaXdpWVdOMGFYWmxYMmx1YzNCbFkzUWlPaUpoZFhSdklpd2lZV04wYVhabFgzTmpjbTlzYkNJNkltRjFkRzhpTENKaFkzUnBkbVZmZEdGd0lqb2lZWFYwYnlJc0luUnZiMnh6SWpwYmV5SnBaQ0k2SW1VNVltSTVNemRrTFdFNU1qZ3RORFl5T1MxaE1tVTNMVEV5WVdZek1UQTJOV1ZsTkNJc0luUjVjR1VpT2lKUVlXNVViMjlzSW4wc2V5SnBaQ0k2SWpJeFlqUXhPVFkwTFRSbVltTXRORFV4WkMxaVl6RmlMV0ZtWVRReU5qWmtNRE00T0NJc0luUjVjR1VpT2lKQ2IzaGFiMjl0Vkc5dmJDSjlMSHNpYVdRaU9pSmtZalJtTUdNNE5pMHpZakl6TFRReU1EY3RZV1V4TUMweVpEUmpPV1ZrTW1VNE5tVWlMQ0owZVhCbElqb2lVbVZ6WlhSVWIyOXNJbjBzZXlKcFpDSTZJbU15TVRJM056RmxMVFJsWW1JdE5HTTVOUzFoWWpCa0xXVTJPVFF6TVRrek9ESTBPQ0lzSW5SNWNHVWlPaUpJYjNabGNsUnZiMndpZlN4N0ltbGtJam9pWm1GbU0yVmpZMlF0TVRFeU1DMDBPRFEzTFRrd05qY3ROelJsTnpkbU9EQTNOVFUxSWl3aWRIbHdaU0k2SWtodmRtVnlWRzl2YkNKOUxIc2lhV1FpT2lJNU9XUmlNek14Tmkwek56ZGtMVFJtTXpVdFlUUmhZUzFpTmpObE16TTFNMk13TlRBaUxDSjBlWEJsSWpvaVNHOTJaWEpVYjI5c0luMHNleUpwWkNJNkltTXhZakk0TXpKaUxUY3dOREF0TkRrd1l5MWlPR1l5TFRrME1HRTBOalUzTUdJeU5pSXNJblI1Y0dVaU9pSkliM1psY2xSdmIyd2lmU3g3SW1sa0lqb2lZV1U0WWpsak5EQXRZMlppWmkwMFl6UXlMVGhrTkRRdE56YzRORFJpWldFelpqUm1JaXdpZEhsd1pTSTZJa2h2ZG1WeVZHOXZiQ0o5TEhzaWFXUWlPaUkxTnpRd09EUXpOQzA0WXpVMkxUUTFOakF0WWpSaU9TMDNOelZoTjJSaFlqVXhPVFVpTENKMGVYQmxJam9pU0c5MlpYSlViMjlzSW4wc2V5SnBaQ0k2SWpJd1pqazVOamcyTFRCaFlqY3RORFV4TVMwNE5qVmpMVGc0TURoaU5qWm1NV0ZrTlNJc0luUjVjR1VpT2lKSWIzWmxjbFJ2YjJ3aWZTeDdJbWxrSWpvaVlqRTFOMk00TXpNdE4ySmlOeTAwWVRsa0xXRXdOamt0TURZd1pXUTJNamczTVRKaklpd2lkSGx3WlNJNklraHZkbVZ5Vkc5dmJDSjlYWDBzSW1sa0lqb2lNVFJoTTJWallUSXRZVGN5TUMwME1qQTFMV0V5TVRndE5tTTROVFl6TnpkaU5ETXpJaXdpZEhsd1pTSTZJbFJ2YjJ4aVlYSWlmU3g3SW1GMGRISnBZblYwWlhNaU9uc2laR0YwWVY5emIzVnlZMlVpT25zaWFXUWlPaUpqTURNeE56VXdZaTFsTlRBeExUUXdNV1V0T1RGaE1TMDRORFU0TTJSaU56bGhPR01pTENKMGVYQmxJam9pUTI5c2RXMXVSR0YwWVZOdmRYSmpaU0o5TENKbmJIbHdhQ0k2ZXlKcFpDSTZJbVl6WW1RMU9UTmxMVFEzTldJdE5HRmxNUzFpWXpBNExURTJaak0xWXprek5HRmtNaUlzSW5SNWNHVWlPaUpNYVc1bEluMHNJbWh2ZG1WeVgyZHNlWEJvSWpwdWRXeHNMQ0p0ZFhSbFpGOW5iSGx3YUNJNmJuVnNiQ3dpYm05dWMyVnNaV04wYVc5dVgyZHNlWEJvSWpwN0ltbGtJam9pTnpJNFpXWXpNelV0WkRBMk15MDBORFl4TFdFMFptSXROMlF6WkdVMU1XSTVObU5rSWl3aWRIbHdaU0k2SWt4cGJtVWlmU3dpYzJWc1pXTjBhVzl1WDJkc2VYQm9JanB1ZFd4c2ZTd2lhV1FpT2lJM1l6SmtaRFl6WVMxaE9USmhMVFJqTURBdFlXTXhOeTB3WlRSbU9EUmlZamRtTnpVaUxDSjBlWEJsSWpvaVIyeDVjR2hTWlc1a1pYSmxjaUo5TEhzaVlYUjBjbWxpZFhSbGN5STZleUpzYVc1bFgyRnNjR2hoSWpwN0luWmhiSFZsSWpvd0xqRjlMQ0pzYVc1bFgyTmhjQ0k2SW5KdmRXNWtJaXdpYkdsdVpWOWpiMnh2Y2lJNmV5SjJZV3gxWlNJNklpTXhaamMzWWpRaWZTd2liR2x1WlY5cWIybHVJam9pY205MWJtUWlMQ0pzYVc1bFgzZHBaSFJvSWpwN0luWmhiSFZsSWpvMWZTd2llQ0k2ZXlKbWFXVnNaQ0k2SW5naWZTd2llU0k2ZXlKbWFXVnNaQ0k2SW5raWZYMHNJbWxrSWpvaU56STRaV1l6TXpVdFpEQTJNeTAwTkRZeExXRTBabUl0TjJRelpHVTFNV0k1Tm1Oa0lpd2lkSGx3WlNJNklreHBibVVpZlN4N0ltRjBkSEpwWW5WMFpYTWlPbnNpYkdsdVpWOWpZWEFpT2lKeWIzVnVaQ0lzSW14cGJtVmZZMjlzYjNJaU9uc2lkbUZzZFdVaU9pSWpaVFptTlRrNEluMHNJbXhwYm1WZmFtOXBiaUk2SW5KdmRXNWtJaXdpYkdsdVpWOTNhV1IwYUNJNmV5SjJZV3gxWlNJNk5YMHNJbmdpT25zaVptbGxiR1FpT2lKNEluMHNJbmtpT25zaVptbGxiR1FpT2lKNUluMTlMQ0pwWkNJNkltWXpZbVExT1RObExUUTNOV0l0TkdGbE1TMWlZekE0TFRFMlpqTTFZemt6TkdGa01pSXNJblI1Y0dVaU9pSk1hVzVsSW4wc2V5SmhkSFJ5YVdKMWRHVnpJanA3SW14aFltVnNJanA3SW5aaGJIVmxJam9pVGtWRFQwWlRYMDFoYzNOQ1lYa2lmU3dpY21WdVpHVnlaWEp6SWpwYmV5SnBaQ0k2SWpZMFptSm1ZMlEzTFRkbE9HWXROR0V3WXkwNU9UZGpMVGN3WlRObU9UUmxNekptWWlJc0luUjVjR1VpT2lKSGJIbHdhRkpsYm1SbGNtVnlJbjFkZlN3aWFXUWlPaUk0WXpoa05Ea3pZUzAzWVRkbUxUUmpOR1V0T0RKbFlpMHdZbVJoTW1ZM09Ua3pNREFpTENKMGVYQmxJam9pVEdWblpXNWtTWFJsYlNKOUxIc2lZWFIwY21saWRYUmxjeUk2ZXlKdmRtVnliR0Y1SWpwN0ltbGtJam9pWTJRd01UZG1ZVEV0TXpFeE9DMDBPRGcwTFdFNU1tUXRNRGRpTmpoa1pERmhZekpsSWl3aWRIbHdaU0k2SWtKdmVFRnVibTkwWVhScGIyNGlmU3dpY0d4dmRDSTZleUpwWkNJNkltSTFObUkxTkRaaExXUTBZMlF0TkdJd05DMWhabUl5TFdVd01EbGlORE5rWm1Wa1l5SXNJbk4xWW5SNWNHVWlPaUpHYVdkMWNtVWlMQ0owZVhCbElqb2lVR3h2ZENKOWZTd2lhV1FpT2lJeU1XSTBNVGsyTkMwMFptSmpMVFExTVdRdFltTXhZaTFoWm1FME1qWTJaREF6T0RnaUxDSjBlWEJsSWpvaVFtOTRXbTl2YlZSdmIyd2lmU3g3SW1GMGRISnBZblYwWlhNaU9uc2liR0ZpWld3aU9uc2lkbUZzZFdVaU9pSklXVU5QVFNKOUxDSnlaVzVrWlhKbGNuTWlPbHQ3SW1sa0lqb2lNRFk0TW1VNE5Ua3RaakEzTkMwME5EWTRMV0V6WXpJdE5EZzFaV0l5TldJellqRmtJaXdpZEhsd1pTSTZJa2RzZVhCb1VtVnVaR1Z5WlhJaWZWMTlMQ0pwWkNJNklqWmxaV1ZoWWpnMUxUQmhNVEV0TkRZMllpMDVNMkl6TFdZNU9Ua3habVpsTkdRNU9TSXNJblI1Y0dVaU9pSk1aV2RsYm1SSmRHVnRJbjBzZXlKaGRIUnlhV0oxZEdWeklqcDdJbU5oYkd4aVlXTnJJanB1ZFd4c0xDSmpiMngxYlc1ZmJtRnRaWE1pT2xzaWVDSXNJbmtpWFN3aVpHRjBZU0k2ZXlKNElqcDdJbDlmYm1SaGNuSmhlVjlmSWpvaVFVRkNRWGRYVkZCa1ZVbEJRVU5uZDJGTk9URlJaMEZCUlVvNWNub3pWa05CUVVRMFJGY3ZVR1JWU1VGQlQwSTRZM001TVZGblFVRjVUM1F4ZWpOV1EwRkJRM2RYYm01UVpGVkpRVUZLYWtwbVRUa3hVV2RCUVdkRWFVRjZNMVpEUVVGQ2IzQTBVRkJrVlVsQlFVWkJWMmc0T1RGUlowRkJUMGxYUzNvelZrTkJRVUZuT1VrelVHUlZTVUZCUVdocWEyTTVNVkZuUVVFNFRrZFZlak5XUTBGQlJGbFJTbXBRWkZWSlFVRk5RM1p0T0RreFVXZEJRWEZDTm1aNk0xWkRRVUZEVVdwaFRGQmtWVWxCUVVocU9IQmpPVEZSWjBGQldVZDFjSG96VmtOQlFVSkpNbkY2VUdSVlNVRkJSRUpLYzAwNU1WRm5RVUZIVEdsNmVqTldRMEZCUVVGS04yWlFaRlZKUVVGUGFWWjFjemt4VVdkQlFUQkJVeXQ2TTFaRFFVRkROR000U0ZCa1ZVbEJRVXRFYVhoTk9URlJaMEZCYVVaSVNYb3pWa05CUVVKM2QwMTJVR1JWU1VGQlJtZDJlamc1TVZGblFVRlJTamRUZWpOV1EwRkJRVzlFWkdKUVpGVkpRVUZDUWpneVl6a3hVV2RCUVN0UGNtTjZNMVpEUVVGRVoxZGxSRkJrVlVsQlFVMXFTVFE0T1RGUlowRkJjMFJtYm5velZrTkJRVU5aY0hWeVVHUlZTVUZCU1VGV04zTTVNVkZuUVVGaFNWUjRlak5XUTBGQlFsRTRMMVJRWkZWSlFVRkVhR2tyVFRreFVXZEJRVWxPU0RkNk0xWkRRVUZCU1ZGUUwxQmtWVWxCUVZCRGRVRjBRakZSWjBGQk1rSXdSekJJVmtOQlFVUkJha0Z1VVdSVlNVRkJTMm8zUkU1Q01WRm5RVUZyUjI5Uk1FaFdRMEZCUWpReVVsQlJaRlZKUVVGSFFrbEdPVUl4VVdkQlFWTk1ZMkV3U0ZaRFFVRkJkMHBvTjFGa1ZVbEJRVUpwVmtsa1FqRlJaMEZCUVVGUmJEQklWa05CUVVSdlkybHFVV1JWU1VGQlRrUm9TemxDTVZGblFVRjFSa0YyTUVoV1EwRkJRMmQyZWt4UlpGVkpRVUZKWjNWT2RFSXhVV2RCUVdOS01EVXdTRlpEUVVGQ1dVUkVNMUZrVlVsQlFVVkNOMUZPUWpGUlowRkJTMDl3UkRCSVZrTkJRVUZSVjFWbVVXUlZTVUZCVUdwSVUzUkNNVkZuUVVFMFJGcFBNRWhXUTBGQlJFbHdWa2hSWkZWSlFVRk1RVlZXWkVJeFVXZEJRVzFKVGxrd1NGWkRRVUZEUVRoc2RsRmtWVWxCUVVkb2FGZzVRakZSWjBGQlZVNUNhVEJJVmtOQlFVRTBVREppVVdSVlNVRkJRME4xWVdSQ01WRm5RVUZEUWpGME1FaFdRMEZCUkhkcE0wUlJaRlZKUVVGT2FqWmpPVUl4VVdkQlFYZEhiRE13U0ZaRFFVRkRiekpJY2xGa1ZVbEJRVXBDU0daMFFqRlJaMEZCWlV4aFFqQklWa05CUVVKblNsbFlVV1JWU1VGQlJXbFZhVTVDTVZGblFVRk5RVTlOTUVoV1EwRkJRVmxqYnk5UlpGVkpRVUZCUkdocmRFSXhVV2RCUVRaRksxY3dTRlpEUVVGRVVYWndibEZrVlVsQlFVeG5kRzVrUWpGUlowRkJiMHA1WnpCSVZrTkJRVU5KUXpaVVVXUlZTVUZCU0VJMmNEbENNVkZuUVVGWFQyMXhNRWhXUTBGQlFrRlhTemRSWkZWSlFVRkRha2h6WkVJeFVXZEJRVVZFWVRFd1NGWkRRVUZFTkhCTWFsRmtWVWxCUVU5QlZIWk9RakZSWjBGQmVVbExMekJJVmtOQlFVTjNPR05NVVdSVlNVRkJTbWhuZUhSQ01WRm5RVUZuVFM5S01FaFdRMEZCUW05UWN6TlJaRlZKUVVGR1EzUXdUa0l4VVdkQlFVOUNlbFV3U0ZaRFFVRkJaMms1WmxGa1ZVbEJRVUZxTmpKMFFqRlJaMEZCT0VkcVpUQklWa05CUVVSWk1TdElVV1JWU1VGQlRVSkhOV1JDTVZGblFVRnhURmh2TUVoV1EwRkJRMUZLVDNwUlpGVkpRVUZJYVZRM09VSXhVV2RCUVZsQlRIb3dTRlpEUVVGQ1NXTm1ZbEZrVlVsQlFVUkVaeXRrUWpGUlowRkJSMFV2T1RCSVZrTkJRVUZCZG1kRVVtUlZTVUZCVDJkelFrNUdNVkZuUVVFd1NuTklNRmhXUTBGQlF6UkRaM1pTWkZWSlFVRkxRalZFZEVZeFVXZEJRV2xQWjFJd1dGWkRRVUZDZDFaNFdGSmtWVWxCUVVacVIwZE9SakZSWjBGQlVVUlZZekJZVmtOQlFVRnZjRUl2VW1SVlNVRkJRa0ZVU1RsR01WRm5RVUVyU1VWdE1GaFdRMEZCUkdjNFEyNVNaRlZKUVVGTmFHWk1aRVl4VVdkQlFYTk5OSGN3V0ZaRFFVRkRXVkJVVkZKa1ZVbEJRVWxEYzA0NVJqRlJaMEZCWVVKek56QllWa05CUVVKUmFXbzNVbVJWU1VGQlJHbzFVV1JHTVZGblFVRkpSMmhHTUZoV1EwRkJRVWt4TUdwU1pGVkpRVUZRUWtaVVRrWXhVV2RCUVRKTVVsQXdXRlpESWl3aVpIUjVjR1VpT2lKbWJHOWhkRFkwSWl3aWMyaGhjR1VpT2xzeE5EUmRmU3dpZVNJNmV5SmZYMjVrWVhKeVlYbGZYeUk2SWtGQlFVRkpRMHh0VERCQlFVRkJSRUZHVVdOM1VVRkJRVUZKUVdGSGVrSkJRVUZCUVVsQ09IWk5SVUZCUVVGRVFYWlNVWGRSUVVGQlFVOURORGxET1VGQlFVRkJTVkJoTDB3d1FVRkJRVU5uU21GdmRsRkJRVUZCUlVKV2JFTTVRVUZCUVVGM1NWSXJUREJCUVVGQlJHZHZXRlYyVVVGQlFVRlBReXRpUXpsQlFVRkJRVUZPZUdwTU1FRkJRVUZEUVdWSlkzWlJRVUZCUVU5QlZYRjVPVUZCUVVGQldVeElUMHd3UVVGQlFVSkJhSGR2ZDFGQlFVRkJUME14VEZSQ1FVRkJRVUZuVDFKUlRVVkJRVUZCUTJjeFZrbDNVVUZCUVVGTFJFZFdSRUpCUVVGQlFYZE1aRmROUlVGQlFVRkRaM1pGZDNkUlFVRkJRVWRFUWxGcVFrRkJRVUZCVVUxWk5FMUZRVUZCUVVSblUydFJkMUZCUVVGQlIwUlFWSHBDUVVGQlFVRkJSbEppVFVWQlFVRkJRa0V4Vm1kM1VVRkJRVUZMUWxkV2FrSkJRVUZCUVRST1pGUk5SVUZCUVVGQ1FVd3dkM2RSUVVGQlFVdERSMUpFUWtGQlFVRkJRVTQwT0UxRlFVRkJRVVJCVDNwdmQxRkJRVUZCU1VOYVRucENRVUZCUVVGUlVHTXdUVVZCUVVGQlFVRkRiRWwzVVVGQlFVRlBRV05pZWtKQlFVRkJRVzlESzAxTlJVRkJRVUZCUVU4MVFYZFJRVUZCUVVWQ1IyeEVRa0ZCUVVGQmIwWkhXVTFGUVVGQlFVSm5ZVE52ZDFGQlFVRkJSVU5HV0VSQ1FVRkJRVUZCU2pnclRVVkJRVUZCUTJkSWFWbDNVVUZCUVVGSFEyVkVWRUpCUVVGQlFVRkVlbkZNTUVGQlFVRkNRVzFCWTNkUlFVRkJRVWxCVTBkcVFrRkJRVUZCZDBsM2MwMUZRVUZCUVVSbllYbG5kMUZCUVVGQlQwSkxTa1JDUVVGQlFVRkJRMjluVFVWQlFVRkJRbWRUZUUxM1VVRkJRVUZOUW5OQ2FrSkJRVUZCUVVsQ2VucE1NRUZCUVVGRFoyMVBUWFpSUVVGQlFVTkJWakZET1VGQlFVRkJiMHBJUlV3d1FVRkJRVUZuUVZCSmRsRkJRVUZCUjBNelJIcENRVUZCUVVGdlJ6UnRUVVZCUVVGQlFrRkJSakIzVVVGQlFVRkJRMU5yZWtKQlFVRkJRVzlEVUV0TlJVRkJRVUZFWjJWbFRYZFJRVUZCUVVORVVTOUVRa0ZCUVVGQldVTlpWMDFWUVVGQlFVRkJOVUZ6ZUZGQlFVRkJTVU5vUVZSR1FVRkJRVUZKUmk4elRVVkJRVUZCUVdka1RWRjNVVUZCUVVGQlEwcHJWRUpCUVVGQlFVRktOV1ZOUlVGQlFVRkVaM0pyYTNkUlFVRkJRVTFETDA1RVFrRkJRVUZCYjA1QlprMUZRVUZCUVVKbldFRnpkMUZCUVVGQlJVUlJOMU01UVVGQlFVRjNUMlpGVERCQlFVRkJRV2RvUzI5MlVVRkJRVUZMUVdkclF6bEJRVUZCUVVGTU1URk1NRUZCUVVGRVoxZzJWWFpSUVVGQlFVdEJRekZUT1VGQlFVRkJkMFpKUTAxRlFVRkJRVVJCZFVSTmQxRkJRVUZCUzBGbFdsUkNRVUZCUVVGdlNWTlhUVVZCUVVGQlFtZFBiMmQzVVVGQlFVRkRSSGRsVkVKQlFVRkJRVFJMVm5KTlJVRkJRVUZDWnpCV1VYZFJRVUZCUVUxRU9GQlVRa0ZCUVVGQlVVTm5iazFGUVVGQlFVRkJNbll3ZGxGQlFVRkJSMEpxY2xNNVFVRkJRVUUwVDNoalREQkJRVUZCUWtFM2FWbDJVVUZCUVVGTlJIWTRRelZCUVVGQlFVbFFSelpNYTBGQlFVRkRRVzl4WjNWUlFVRkJRVTlDVkd4cE5VRkJRVUZCVVVGWFJVeHJRVUZCUVVSQlkyODRkVkZCUVVGQlJVUm5iV2sxUVVGQlFVRjNSVEp0VEd0QlFVRkJRa0ZpVDJkMVVVRkJRVUZMUTB0TGFUbEJRVUZCUVVsTGJITk1NRUZCUVVGQlFVWktUWFpSUVVGQlFVMUNLM1ZUT1VGQlFVRkJiMDl1Wmt3d1FVRkJRVU5CY2s5SmRsRkJRVUZCUlVKMk5WTTVRVUZCUVVGSlJFeHZUREJCUVVGQlFtZHZUbXQyVVVGQlFVRk5RVTk1ZVRsQlFVRkJRVUZJTWpoTU1FRkJRVUZDWjNkd1ozWlJRVUZCUVUxQlNHUlRPVUZCUVVGQlNVVXhVa3d3UVVGQlFVTm5kbWcwZGxGQlFVRkJSVUYzTjBNMVFVRkJRVUYzUzBjMVRHdEJRVUZCUTJkTlNXTjFVVUZCUVVGTFF5OVdRelZCUVVGQlFXZEZOR2xNYTBGQlFVRkJRV1pTWTNWUlFVRkJRVWREY2tSRE5VRkJRVUZCTkU1clFreHJRVUZCUVVOQk5GVnpkVkZCUVVGQlEwUndiRk0xUVVGQlFVRjNVRVJtVEd0QlFVRkJSR2QzTUhOMlVVRkJRVUZQUTFkMGVUbEJRVUZCUVVGTVZWSk5SVUZCUVVGRVp6SnBWWGRSUVVGQlFVdEJRVTlxUWtGQlFVRkJaME5hVDAxRlFVRkJRVU5CU21zMGQxRkJRVUZCU1VGdFZHcENRU0lzSW1SMGVYQmxJam9pWm14dllYUTJOQ0lzSW5Ob1lYQmxJanBiTVRRMFhYMTlmU3dpYVdRaU9pSmpNRE14TnpVd1lpMWxOVEF4TFRRd01XVXRPVEZoTVMwNE5EVTRNMlJpTnpsaE9HTWlMQ0owZVhCbElqb2lRMjlzZFcxdVJHRjBZVk52ZFhKalpTSjlMSHNpWVhSMGNtbGlkWFJsY3lJNmV5SnNhVzVsWDJOaGNDSTZJbkp2ZFc1a0lpd2liR2x1WlY5amIyeHZjaUk2ZXlKMllXeDFaU0k2SWlNek1qZzRZbVFpZlN3aWJHbHVaVjlxYjJsdUlqb2ljbTkxYm1RaUxDSnNhVzVsWDNkcFpIUm9JanA3SW5aaGJIVmxJam8xZlN3aWVDSTZleUptYVdWc1pDSTZJbmdpZlN3aWVTSTZleUptYVdWc1pDSTZJbmtpZlgwc0ltbGtJam9pTURZd1lUVTNZakl0TURGbE5DMDBZamszTFRrMFptVXRNbVpoWVdKaVl6azFZamN5SWl3aWRIbHdaU0k2SWt4cGJtVWlmU3g3SW1GMGRISnBZblYwWlhNaU9uc2laR0Y1Y3lJNld6RXNNaXd6TERRc05TdzJMRGNzT0N3NUxERXdMREV4TERFeUxERXpMREUwTERFMUxERTJMREUzTERFNExERTVMREl3TERJeExESXlMREl6TERJMExESTFMREkyTERJM0xESTRMREk1TERNd0xETXhYWDBzSW1sa0lqb2lZbUZrWWpRM1pqTXRaV0ZpTWkwME5UVmlMV0pqWm1FdE9XTXdZemxqWTJKbU16ZG1JaXdpZEhsd1pTSTZJa1JoZVhOVWFXTnJaWElpZlN4N0ltRjBkSEpwWW5WMFpYTWlPbnNpWTJGc2JHSmhZMnNpT201MWJHd3NJbkJzYjNRaU9uc2lhV1FpT2lKaU5UWmlOVFEyWVMxa05HTmtMVFJpTURRdFlXWmlNaTFsTURBNVlqUXpaR1psWkdNaUxDSnpkV0owZVhCbElqb2lSbWxuZFhKbElpd2lkSGx3WlNJNklsQnNiM1FpZlN3aWNtVnVaR1Z5WlhKeklqcGJleUpwWkNJNklqZGpNbVJrTmpOaExXRTVNbUV0TkdNd01DMWhZekUzTFRCbE5HWTROR0ppTjJZM05TSXNJblI1Y0dVaU9pSkhiSGx3YUZKbGJtUmxjbVZ5SW4xZExDSjBiMjlzZEdsd2N5STZXMXNpVG1GdFpTSXNJazVGUTA5R1UxOUhUMDB6WDBaUFVrVkRRVk5VSWwwc1d5SkNhV0Z6SWl3aUxUQXVPVFlpWFN4YklsTnJhV3hzSWl3aU1TNHlOQ0pkWFgwc0ltbGtJam9pT1Rsa1lqTXpNVFl0TXpjM1pDMDBaak0xTFdFMFlXRXRZall6WlRNek5UTmpNRFV3SWl3aWRIbHdaU0k2SWtodmRtVnlWRzl2YkNKOUxIc2lZWFIwY21saWRYUmxjeUk2ZTMwc0ltbGtJam9pTldOa1ptUTJOVFl0TURnM05DMDBPVFl5TFRsbU1qZ3RNR1JtWVRZM05qWTFNRE0zSWl3aWRIbHdaU0k2SWtKaGMybGpWR2xqYTBadmNtMWhkSFJsY2lKOUxIc2lZWFIwY21saWRYUmxjeUk2ZXlKc2FXNWxYMkZzY0doaElqcDdJblpoYkhWbElqb3dMakY5TENKc2FXNWxYMk5oY0NJNkluSnZkVzVrSWl3aWJHbHVaVjlqYjJ4dmNpSTZleUoyWVd4MVpTSTZJaU14WmpjM1lqUWlmU3dpYkdsdVpWOXFiMmx1SWpvaWNtOTFibVFpTENKc2FXNWxYM2RwWkhSb0lqcDdJblpoYkhWbElqbzFmU3dpZUNJNmV5Sm1hV1ZzWkNJNkluZ2lmU3dpZVNJNmV5Sm1hV1ZzWkNJNklua2lmWDBzSW1sa0lqb2lNMk0wTWpFeVpUWXRNbUUxWmkwMFlqbGpMVGsxWXprdE16a3paamM0WXpGaE16QTNJaXdpZEhsd1pTSTZJa3hwYm1VaWZTeDdJbUYwZEhKcFluVjBaWE1pT25zaWNHeHZkQ0k2ZXlKcFpDSTZJbUkxTm1JMU5EWmhMV1EwWTJRdE5HSXdOQzFoWm1JeUxXVXdNRGxpTkROa1ptVmtZeUlzSW5OMVluUjVjR1VpT2lKR2FXZDFjbVVpTENKMGVYQmxJam9pVUd4dmRDSjlmU3dpYVdRaU9pSmxPV0ppT1RNM1pDMWhPVEk0TFRRMk1qa3RZVEpsTnkweE1tRm1NekV3TmpWbFpUUWlMQ0owZVhCbElqb2lVR0Z1Vkc5dmJDSjlMSHNpWVhSMGNtbGlkWFJsY3lJNmUzMHNJbWxrSWpvaU1ETTROVGRrTlRFdE9EZGxNUzAwWmpkakxUazVZalV0TVRZMk56a3lNamRsWWpNMElpd2lkSGx3WlNJNklrUmhkR1YwYVcxbFZHbGphMFp2Y20xaGRIUmxjaUo5TEhzaVlYUjBjbWxpZFhSbGN5STZleUprWVhSaFgzTnZkWEpqWlNJNmV5SnBaQ0k2SWpFeFpqVTBaRGMxTFRSaU1ETXRORFJtWmkxaU9XUXlMVFEwWlRVd016TXpZbUV3WkNJc0luUjVjR1VpT2lKRGIyeDFiVzVFWVhSaFUyOTFjbU5sSW4wc0ltZHNlWEJvSWpwN0ltbGtJam9pTURZd1lUVTNZakl0TURGbE5DMDBZamszTFRrMFptVXRNbVpoWVdKaVl6azFZamN5SWl3aWRIbHdaU0k2SWt4cGJtVWlmU3dpYUc5MlpYSmZaMng1Y0dnaU9tNTFiR3dzSW0xMWRHVmtYMmRzZVhCb0lqcHVkV3hzTENKdWIyNXpaV3hsWTNScGIyNWZaMng1Y0dnaU9uc2lhV1FpT2lJell6UXlNVEpsTmkweVlUVm1MVFJpT1dNdE9UVmpPUzB6T1RObU56aGpNV0V6TURjaUxDSjBlWEJsSWpvaVRHbHVaU0o5TENKelpXeGxZM1JwYjI1ZloyeDVjR2dpT201MWJHeDlMQ0pwWkNJNklqSXdZbU13WWpneUxXTTFZelV0TkdSbE1DMDRaR1pqTFRrNE56UTBZbU01WlRJek5DSXNJblI1Y0dVaU9pSkhiSGx3YUZKbGJtUmxjbVZ5SW4wc2V5SmhkSFJ5YVdKMWRHVnpJanA3SW1OaGJHeGlZV05ySWpwdWRXeHNMQ0pqYjJ4MWJXNWZibUZ0WlhNaU9sc2llQ0lzSW5raVhTd2laR0YwWVNJNmV5SjRJanA3SWw5ZmJtUmhjbkpoZVY5Zklqb2lRVUZFUVRsaUwwOWtWVWxCUVV0b2EzYzROVEZSWjBGQmEwNVFSM3B1VmtOQlFVSTBVWE55VDJSVlNVRkJSME40ZW1NMU1WRm5RVUZUUTBSU2VtNVdRMEZCUVhkcU9WUlBaRlZKUVVGQ2Fpc3hPRFV4VVdkQlFVRkhNMko2YmxaRFFVRkViekk1TjA5a1ZVbEJRVTVDU3pSek5URlJaMEZCZFV4dWJIcHVWa05CUVVOblMwOXVUMlJWU1VGQlNXbFlOMDAxTVZGblFVRmpRV0ozZW01V1EwRkJRbGxrWmxCUFpGVkpRVUZGUkdzNWN6VXhVV2RCUVV0R1VEWjZibFpEUVVGQlVYZDJNMDlrVlVsQlFWQm5kMEZqT1RGUlowRkJORW80UlhvelZrTkJRVVJKUkdkcVVHUlZTVUZCVEVJNVF6ZzVNVkZuUVVGdFQzZFBlak5XUTBGQlEwRlhlRXhRWkZWSlFVRkhha3RHWXpreFVXZEJRVlZFYTFwNk0xWkRRVUZCTkhGQ2VsQmtWVWxCUVVOQldFbE5PVEZSWjBGQlEwbFphbm96VmtOQlFVUjNPVU5pVUdSVlNVRkJUbWhxUzNNNU1WRm5RVUYzVGtsMGVqTldRMEZCUTI5UlZFaFFaRlZKUVVGS1EzZE9UVGt4VVdkQlFXVkNPRFI2TTFaRFFVRkNaMnBxZGxCa1ZVbEJRVVZxT1ZCek9URlJaMEZCVFVkNFEzb3pWa05CUVVGWk1qQllVR1JWU1VGQlFVSkxVMk01TVZGblFVRTJUR2hOZWpOV1EwRkJSRkZLTVVSUVpGVkpRVUZNYVZkVk9Ea3hVV2RCUVc5QlZsaDZNMVpEUVVGRFNXUkdjbEJrVlVsQlFVaEVhbGhqT1RGUlowRkJWMFpLYUhvelZrTkJRVUpCZDFkVVVHUlZTVUZCUTJkM1lVMDVNVkZuUVVGRlNqbHllak5XUTBGQlJEUkVWeTlRWkZWSlFVRlBRamhqY3preFVXZEJRWGxQZERGNk0xWkRRVUZEZDFkdWJsQmtWVWxCUVVwcVNtWk5PVEZSWjBGQlowUnBRWG96VmtOQlFVSnZjRFJRVUdSVlNVRkJSa0ZYYURnNU1WRm5RVUZQU1ZkTGVqTldRMEZCUVdjNVNUTlFaRlZKUVVGQmFHcHJZemt4VVdkQlFUaE9SMVY2TTFaRFFVRkVXVkZLYWxCa1ZVbEJRVTFEZG0wNE9URlJaMEZCY1VJMlpub3pWa05CUVVOUmFtRk1VR1JWU1VGQlNHbzRjR001TVZGblFVRlpSM1Z3ZWpOV1EwRkJRa2t5Y1hwUVpGVkpRVUZFUWtwelRUa3hVV2RCUVVkTWFYcDZNMVpEUVVGQlFVbzNabEJrVlVsQlFVOXBWblZ6T1RGUlowRkJNRUZUSzNvelZrTkJRVU0wWXpoSVVHUlZTVUZCUzBScGVFMDVNVkZuUVVGcFJraEplak5XUTBGQlFuZDNUWFpRWkZWSlFVRkdaM1o2T0RreFVXZEJRVkZLTjFONk0xWkRRVUZCYjBSa1lsQmtWVWxCUVVKQ09ESmpPVEZSWjBGQkswOXlZM296VmtOQlFVUm5WMlZFVUdSVlNVRkJUV3BKTkRnNU1WRm5RVUZ6UkdadWVqTldRMEZCUTFsd2RYSlFaRlZKUVVGSlFWWTNjemt4VVdkQlFXRkpWSGg2TTFaRFFVRkNVVGd2VkZCa1ZVbEJRVVJvYVN0Tk9URlJaMEZCU1U1SU4zb3pWa05CUVVGSlVWQXZVR1JWU1VGQlVFTjFRWFJDTVZGblFVRXlRakJITUVoV1EwRkJSRUZxUVc1UlpGVkpRVUZMYWpkRVRrSXhVV2RCUVd0SGIxRXdTRlpEUVVGQ05ESlNVRkZrVlVsQlFVZENTVVk1UWpGUlowRkJVMHhqWVRCSVZrTkJRVUYzU21nM1VXUlZTVUZCUW1sV1NXUkNNVkZuUVVGQlFWRnNNRWhXUTBGQlJHOWphV3BSWkZWSlFVRk9SR2hMT1VJeFVXZEJRWFZHUVhZd1NGWkRRVUZEWjNaNlRGRmtWVWxCUVVsbmRVNTBRakZSWjBGQlkwb3dOVEJJVmtOQlFVSlpSRVF6VVdSVlNVRkJSVUkzVVU1Q01WRm5RVUZMVDNCRU1FaFdRMEZCUVZGWFZXWlJaRlZKUVVGUWFraFRkRUl4VVdkQlFUUkVXazh3U0ZaRFFVRkVTWEJXU0ZGa1ZVbEJRVXhCVlZaa1FqRlJaMEZCYlVsT1dUQklWa05CUVVOQk9HeDJVV1JWU1VGQlIyaG9XRGxDTVZGblFVRlZUa0pwTUVoV1EwRkJRVFJRTW1KUlpGVkpRVUZEUTNWaFpFSXhVV2RCUVVOQ01YUXdTRlpEUVVGRWQya3pSRkZrVlVsQlFVNXFObU01UWpGUlowRkJkMGRzTXpCSVZrTkJRVU52TWtoeVVXUlZTVUZCU2tKSVpuUkNNVkZuUVVGbFRHRkNNRWhXUTBGQlFtZEtXVmhSWkZWSlFVRkZhVlZwVGtJeFVXZEJRVTFCVDAwd1NGWkRRVUZCV1dOdkwxRmtWVWxCUVVGRWFHdDBRakZSWjBGQk5rVXJWekJJVmtOQlFVUlJkbkJ1VVdSVlNVRkJUR2QwYm1SQ01WRm5QVDBpTENKa2RIbHdaU0k2SW1ac2IyRjBOalFpTENKemFHRndaU0k2V3pFME1GMTlMQ0o1SWpwN0lsOWZibVJoY25KaGVWOWZJam9pVFhwTmVrMTZUWHBNTUVGNlRYcE5lazE2VFhaUlJFMTZUWHBOZWsxNU9VRkJRVUZCUVVGQlFVd3dRVUZCUVVGQlFVRkJkbEZCUVVGQlFVRkJRVU01UVhwamVrMTZUWHBOVEd0RFlXMWFiVnB0V210MVVVZGFiVnB0V20xYWFUVkJiWEJ0V20xYWJWcE1NRU5oYlZwdFdtMWFhM1pSVFROTmVrMTZUWHBETlVGNlkzcE5lazE2VFV4clJFNTZUWHBOZWsxM2RGRkJRVUZCUVVGQlFVTTFRVnB0V20xYWJWcHRUR3RCUVVGQlFVRkJRVUYyVVUwelRYcE5lazE2UXpsQldtMWFiVnB0V20xTlJVSnRXbTFhYlZwdFdYaFJSMXB0V20xYWJWcHFSa0ZhYlZwdFdtMWFiVTFWUVVGQlFVRkJRVWxCZUZGS2NWcHRXbTFhYlZSR1FVRkJRVUZCUVVOQlRWVkNiVnB0V20xYWJWbDRVVWRhYlZwdFdtMDFha0pCVFhwTmVrMTZUM3BOUlVST2VrMTZUWHBGZDNkUlRUTk5lazE2VFhwRE9VRkJRVUZCUVVGQlFVd3dSRTU2VFhwTmVrMTNkVkZLY1ZwdFdtMWFiVk0xUVhwamVrMTZUWHBOVEd0QmVrMTZUWHBOZWsxMlVVZGFiVnB0V20xYWFUbEJiWEJ0V20xYWJWcE1NRVJPZWsxNlRYcE5kM1pSUVVGQlFVRkJRVUZFUWtGTmVrMTZUWHBOZWsxRlFVRkJRVUZCUVVsQmQxRkVUWHBOZWsxNmMzcENRWHBqZWsxNlRYcE5UVVZDYlZwdFdtMWFkVmwzVVVGQlFVRkJRVUZCUkVaQmVtTjZUWHBOZWsxTlJVUk9lazE2VFhwTmQzZFJRVUZCUVVGQlFXZEVRa0ZOZWsxNlRYcE5lazFGUW0xYWJWcHRXbTFaZGxGSFdtMWFiVnB0V21rNVFVMTZUWHBOZWsxNlREQkVUbnBOZWsxNlRYZDFVVXB4V20xYWJWcHRVelZCYlhCdFdtMWFiVnBNYTBGQlFVRkJRVUZCUVhWUlRUTk5lazE2VFhwRE1VRmFiVnB0V20xYWJVeFZRbTFhYlZwdFdtMVpkRkZIV20xYWJWcHRXbWt4UVcxd2JWcHRXbTFhVEZWRVRucE5lazE2VFhkMFVVMHpUWHBOZWsxNlF6RkJlbU42VFhwTmVrMU1WVVJPZWsxNlRYcE5kM1JSUVVGQlFVRkJRVUZETlVGTmVrMTZUWHBOZWt4clFVRkJRVUZCUVVGQmRsRkVUWHBOZWsxNlRYazVRVTE2VFhwTmVrMTZUREJCZWsxNlRYcE5lazEzVVVweFdtMWFiVnBIVkVKQmJYQnRXbTFhYTFwTlJVTmhiVnB0V20xU2EzZFJRVUZCUVVGQlFVRkVRa0Y2WTNwTmVrMTZUVXd3UTJGdFdtMWFiVnByZGxGSFdtMWFiVnB0V21rNVFVRkJRVUZCUVVGQlRVVkJRVUZCUVVGQlFVRjNVVTB6VFhwTmVrMTZRemxCV20xYWJWcHRXbTFNTUVGNlRYcE5lazE2VFhaUlJFMTZUWHBOZWsxNU9VRk5lazE2VFhwTmVrd3dRbTFhYlZwdFdtMVpkbEZLY1ZwdFdtMWFSMVJDUVZwdFdtMWFiVnB0VFVWQlFVRkJRVUZCU1VGM1VVUk5lazE2VFhwemVrSkJiWEJ0V20xYWJWcE5SVU5oYlZwdFdtMWFhM2RSUjFwdFdtMWFiVnBxUWtGNlkzcE5lazE0VFUxRlJFNTZUWHBOZWsxM2RsRk5NMDE2VFhwTmVrTTVRWHBqZWsxNlRYcE5UREJEWVcxYWJWcHRXbXQyVVVweFdtMWFiVnB0VXpsQlRYcE5lazE2VFhwTU1FTmhiVnB0V20xYWEzVlJSMXB0V20xYWJWcHBOVUZOZWsxNlRYcE5la3hyUVhwTmVrMTZUWHBOZFZGRVRYcE5lazE2VFhrMVFWcHRXbTFhYlZwdFRHdERZVzFhYlZwdFdtdDFVVXB4V20xYWJWcHRVelZCV20xYWJWcHRXbTFNTUVOaGJWcHRXbTFTYTNkUlNuRmFiVnB0V2tkVVFrRmFiVnB0V20xYWJVMUZRMkZ0V20xYWJWcHJkMUZCUVVGQlFVRkJaMFJHUVUxNlRYcE5lazk2VFZWQmVrMTZUWHBOTjAxNFVVRkJRVUZCUVVGblJFcEJiWEJ0V20xYWJWcE5hMEY2VFhwTmVrMDNUWGxSUVVGQlFVRkJRVUZFVGtGQlFVRkJRVUZCUVUwd1JFNTZUWHBOZWsxM2VWRk5NMDE2VFhwTmVrUktRVTE2VFhwTmVrOTZUV3RCZWsxNlRYcE5OMDE1VVVSTmVrMTZUWHB6ZWtwQlRYcE5lazE2VDNwTmEwUk9lazE2VFhwTmQzbFJUVE5OZWsxNlRYcEVTa0ZOZWsxNlRYcFBlazFyUTJGdFdtMWFiVnByZVZGS2NWcHRXbTFhYlZSS1FVRkJRVUZCUVVOQlRXdERZVzFhYlZwdFdtdDVVVWRhYlZwdFdtMWFha3BCV20xYWJWcHRZbTFOVlVGQlFVRkJRVUZKUVhoUlRUTk5lazE2VFZSRVJrRmFiVnB0V20xYWJVMVZSRTU2VFhwTmVrMTNlRkZCUFQwaUxDSmtkSGx3WlNJNkltWnNiMkYwTmpRaUxDSnphR0Z3WlNJNld6RTBNRjE5Zlgwc0ltbGtJam9pTUdOaVpEbG1OVFV0WW1Zd09TMDBaR1ZrTFRnd05tSXROMlprTURoak1XSTFZamd5SWl3aWRIbHdaU0k2SWtOdmJIVnRia1JoZEdGVGIzVnlZMlVpZlN4N0ltRjBkSEpwWW5WMFpYTWlPbnNpYkdsdVpWOWpZWEFpT2lKeWIzVnVaQ0lzSW14cGJtVmZZMjlzYjNJaU9uc2lkbUZzZFdVaU9pSWpabVZsTURoaUluMHNJbXhwYm1WZmFtOXBiaUk2SW5KdmRXNWtJaXdpYkdsdVpWOTNhV1IwYUNJNmV5SjJZV3gxWlNJNk5YMHNJbmdpT25zaVptbGxiR1FpT2lKNEluMHNJbmtpT25zaVptbGxiR1FpT2lKNUluMTlMQ0pwWkNJNklqVTRaREE0WXpGbUxUWXhaREF0TkRBd1pDMWlNMkl4TFdVeE16QTFaV05oWVRFME9TSXNJblI1Y0dVaU9pSk1hVzVsSW4wc2V5SmhkSFJ5YVdKMWRHVnpJanA3SW14cGJtVmZZV3h3YUdFaU9uc2lkbUZzZFdVaU9qQXVNWDBzSW14cGJtVmZZMkZ3SWpvaWNtOTFibVFpTENKc2FXNWxYMk52Ykc5eUlqcDdJblpoYkhWbElqb2lJekZtTnpkaU5DSjlMQ0pzYVc1bFgycHZhVzRpT2lKeWIzVnVaQ0lzSW14cGJtVmZkMmxrZEdnaU9uc2lkbUZzZFdVaU9qVjlMQ0o0SWpwN0ltWnBaV3hrSWpvaWVDSjlMQ0o1SWpwN0ltWnBaV3hrSWpvaWVTSjlmU3dpYVdRaU9pSTNNR1ZqT0RNMFlTMHhZelExTFRRd1lUQXRZVEkxWXkxak16ZzNaVFJpWm1ZME9UZ2lMQ0owZVhCbElqb2lUR2x1WlNKOUxIc2lZWFIwY21saWRYUmxjeUk2ZXlKa1lYUmhYM052ZFhKalpTSTZleUpwWkNJNklqQmpZbVE1WmpVMUxXSm1NRGt0TkdSbFpDMDRNRFppTFRkbVpEQTRZekZpTldJNE1pSXNJblI1Y0dVaU9pSkRiMngxYlc1RVlYUmhVMjkxY21ObEluMHNJbWRzZVhCb0lqcDdJbWxrSWpvaU5UaGtNRGhqTVdZdE5qRmtNQzAwTURCa0xXSXpZakV0WlRFek1EVmxZMkZoTVRRNUlpd2lkSGx3WlNJNklreHBibVVpZlN3aWFHOTJaWEpmWjJ4NWNHZ2lPbTUxYkd3c0ltMTFkR1ZrWDJkc2VYQm9JanB1ZFd4c0xDSnViMjV6Wld4bFkzUnBiMjVmWjJ4NWNHZ2lPbnNpYVdRaU9pSTNNR1ZqT0RNMFlTMHhZelExTFRRd1lUQXRZVEkxWXkxak16ZzNaVFJpWm1ZME9UZ2lMQ0owZVhCbElqb2lUR2x1WlNKOUxDSnpaV3hsWTNScGIyNWZaMng1Y0dnaU9tNTFiR3g5TENKcFpDSTZJbVU1TUdNMlpUQTJMVFE1TWpRdE5EUXhNUzFoTXpZMExUVTBNV0k1Tm1Vd05XRXdZeUlzSW5SNWNHVWlPaUpIYkhsd2FGSmxibVJsY21WeUluMHNleUpoZEhSeWFXSjFkR1Z6SWpwN0ltUmhlWE1pT2xzeExERTFYWDBzSW1sa0lqb2lNV1E1WldZelpHSXRZbUkyTWkwME5tUmpMVGswWm1NdE1qSmlOMkptWVRCbU1EaGtJaXdpZEhsd1pTSTZJa1JoZVhOVWFXTnJaWElpZlN4N0ltRjBkSEpwWW5WMFpYTWlPbnQ5TENKcFpDSTZJakZsWVRVM056SXhMV0kzTVRrdE5EWXhZeTA1T0dJMkxUQTRORGN5T0dGaVlqQm1ZaUlzSW5SNWNHVWlPaUpNYVc1bFlYSlRZMkZzWlNKOUxIc2lZWFIwY21saWRYUmxjeUk2ZXlKa1lYbHpJanBiTVN3NExERTFMREl5WFgwc0ltbGtJam9pT1dSa04yRXdOVGd0TkRjMU9DMDBOalZqTFdJM00yRXRZalk1TjJRd09EWXlaV0V5SWl3aWRIbHdaU0k2SWtSaGVYTlVhV05yWlhJaWZTeDdJbUYwZEhKcFluVjBaWE1pT25zaVkyRnNiR0poWTJzaU9tNTFiR3dzSW1OdmJIVnRibDl1WVcxbGN5STZXeUo0SWl3aWVTSmRMQ0prWVhSaElqcDdJbmdpT25zaVgxOXVaR0Z5Y21GNVgxOGlPaUpCUVVSQk9XSXZUMlJWU1VGQlMyaHJkemcxTVZGblFVRnJUbEJIZW01V1EwRkJRMEZYZUV4UVpGVkpRVUZIYWt0R1l6a3hVV2RCUVZWRWExcDZNMVpEUVVGQ1FYZFhWRkJrVlVsQlFVTm5kMkZOT1RGUlowRkJSVW81Y25velZrTWlMQ0prZEhsd1pTSTZJbVpzYjJGME5qUWlMQ0p6YUdGd1pTSTZXemxkZlN3aWVTSTZleUpmWDI1a1lYSnlZWGxmWHlJNklrRkJRVUZSUjJadFRHdEJRVUZCUkVGa1VHOTFVVUZCUVVGRlEwTkVhVGxCUVVGQlFXOU9WbXBOUlVGQlFVRkVRVWxHYjNkUlFVRkJRVUZDYzFWRVFrRkJRVUZCVVUxUU1VeHJRVUZCUVVKQmR5OVZkVkZCUVVGQlJVUkVPVk0xUVNJc0ltUjBlWEJsSWpvaVpteHZZWFEyTkNJc0luTm9ZWEJsSWpwYk9WMTlmWDBzSW1sa0lqb2lNVEZtTlRSa056VXROR0l3TXkwME5HWm1MV0k1WkRJdE5EUmxOVEF6TXpOaVlUQmtJaXdpZEhsd1pTSTZJa052YkhWdGJrUmhkR0ZUYjNWeVkyVWlmU3g3SW1GMGRISnBZblYwWlhNaU9uc2laR2x0Wlc1emFXOXVJam94TENKd2JHOTBJanA3SW1sa0lqb2lZalUyWWpVME5tRXRaRFJqWkMwMFlqQTBMV0ZtWWpJdFpUQXdPV0kwTTJSbVpXUmpJaXdpYzNWaWRIbHdaU0k2SWtacFozVnlaU0lzSW5SNWNHVWlPaUpRYkc5MEluMHNJblJwWTJ0bGNpSTZleUpwWkNJNklqbGtOV1F3WkdVMkxXTXhaVE10TkRNNU1TMWhNbUl3TFRGa09UTmhPV1UzTmpSak5DSXNJblI1Y0dVaU9pSkNZWE5wWTFScFkydGxjaUo5ZlN3aWFXUWlPaUl6TnpBNFpUSTVZeTA0WVRJMExUUTRNMlF0T1Rka01TMW1Zek5rTkdNMk1EWmtZamtpTENKMGVYQmxJam9pUjNKcFpDSjlMSHNpWVhSMGNtbGlkWFJsY3lJNmV5SmtZWGx6SWpwYk1TdzBMRGNzTVRBc01UTXNNVFlzTVRrc01qSXNNalVzTWpoZGZTd2lhV1FpT2lJM01qWXpZbVk0WWkxbU9EWTFMVFEzTUdJdE9UWmtNUzFsTWpabU1UbGpNV1ExTWpnaUxDSjBlWEJsSWpvaVJHRjVjMVJwWTJ0bGNpSjlMSHNpWVhSMGNtbGlkWFJsY3lJNmV5SnNZV0psYkNJNmV5SjJZV3gxWlNJNklrOWljMlZ5ZG1GMGFXOXVjeUo5TENKeVpXNWtaWEpsY25NaU9sdDdJbWxrSWpvaVpUa3dZelpsTURZdE5Ea3lOQzAwTkRFeExXRXpOalF0TlRReFlqazJaVEExWVRCaklpd2lkSGx3WlNJNklrZHNlWEJvVW1WdVpHVnlaWElpZlYxOUxDSnBaQ0k2SWpjMFpUSXhNV1UxTFRFME9Ea3ROREZrTlMxaVlqa3dMVE0wWkdFMU16TmlZelprWkNJc0luUjVjR1VpT2lKTVpXZGxibVJKZEdWdEluMHNleUpoZEhSeWFXSjFkR1Z6SWpwN0luQnNiM1FpT25zaWFXUWlPaUppTlRaaU5UUTJZUzFrTkdOa0xUUmlNRFF0WVdaaU1pMWxNREE1WWpRelpHWmxaR01pTENKemRXSjBlWEJsSWpvaVJtbG5kWEpsSWl3aWRIbHdaU0k2SWxCc2IzUWlmWDBzSW1sa0lqb2laR0kwWmpCak9EWXRNMkl5TXkwME1qQTNMV0ZsTVRBdE1tUTBZemxsWkRKbE9EWmxJaXdpZEhsd1pTSTZJbEpsYzJWMFZHOXZiQ0o5TEhzaVlYUjBjbWxpZFhSbGN5STZleUp1ZFcxZmJXbHViM0pmZEdsamEzTWlPalY5TENKcFpDSTZJakk1WWpVNE56Wm1MV05rWmpZdE5EWXhNUzFpTjJFekxXUTFZVFU0WkdRM1pUbGxNQ0lzSW5SNWNHVWlPaUpFWVhSbGRHbHRaVlJwWTJ0bGNpSjlMSHNpWVhSMGNtbGlkWFJsY3lJNmUzMHNJbWxrSWpvaU9HUmxZall4WWpjdFkyWmpaaTAwWm1NMUxXSXdaVGN0TldNME56ZGlOR1prTjJObElpd2lkSGx3WlNJNklreHBibVZoY2xOallXeGxJbjBzZXlKaGRIUnlhV0oxZEdWeklqcDdJbU5oYkd4aVlXTnJJanB1ZFd4c2ZTd2lhV1FpT2lJek1qa3dZelV5WXkwd1lqWmxMVFJrWkdNdFlUVTBOeTFtT1RZMk56QmpObVZsWWpZaUxDSjBlWEJsSWpvaVJHRjBZVkpoYm1kbE1XUWlmU3g3SW1GMGRISnBZblYwWlhNaU9udDlMQ0pwWkNJNklqQmtNRGcyTnpsbExUZG1NelF0TkRZeVl5MDVaRFl4TFdZeVlUWXpZVFUzWkdGbU5DSXNJblI1Y0dVaU9pSlViMjlzUlhabGJuUnpJbjBzZXlKaGRIUnlhV0oxZEdWeklqcDdJbVp2Y20xaGRIUmxjaUk2ZXlKcFpDSTZJakF6T0RVM1pEVXhMVGczWlRFdE5HWTNZeTA1T1dJMUxURTJOamM1TWpJM1pXSXpOQ0lzSW5SNWNHVWlPaUpFWVhSbGRHbHRaVlJwWTJ0R2IzSnRZWFIwWlhJaWZTd2ljR3h2ZENJNmV5SnBaQ0k2SW1JMU5tSTFORFpoTFdRMFkyUXROR0l3TkMxaFptSXlMV1V3TURsaU5ETmtabVZrWXlJc0luTjFZblI1Y0dVaU9pSkdhV2QxY21VaUxDSjBlWEJsSWpvaVVHeHZkQ0o5TENKMGFXTnJaWElpT25zaWFXUWlPaUl5T1dJMU9EYzJaaTFqWkdZMkxUUTJNVEV0WWpkaE15MWtOV0UxT0dSa04yVTVaVEFpTENKMGVYQmxJam9pUkdGMFpYUnBiV1ZVYVdOclpYSWlmWDBzSW1sa0lqb2laVEkxWVROalpETXRZV013WWkwME5XSmtMVGc0TnpVdE9UUmpaakV3Tm1Fd05qUXlJaXdpZEhsd1pTSTZJa1JoZEdWMGFXMWxRWGhwY3lKOUxIc2lZWFIwY21saWRYUmxjeUk2ZXlKaWIzUjBiMjFmZFc1cGRITWlPaUp6WTNKbFpXNGlMQ0ptYVd4c1gyRnNjR2hoSWpwN0luWmhiSFZsSWpvd0xqVjlMQ0ptYVd4c1gyTnZiRzl5SWpwN0luWmhiSFZsSWpvaWJHbG5hSFJuY21WNUluMHNJbXhsWm5SZmRXNXBkSE1pT2lKelkzSmxaVzRpTENKc1pYWmxiQ0k2SW05MlpYSnNZWGtpTENKc2FXNWxYMkZzY0doaElqcDdJblpoYkhWbElqb3hMakI5TENKc2FXNWxYMk52Ykc5eUlqcDdJblpoYkhWbElqb2lZbXhoWTJzaWZTd2liR2x1WlY5a1lYTm9JanBiTkN3MFhTd2liR2x1WlY5M2FXUjBhQ0k2ZXlKMllXeDFaU0k2TW4wc0luQnNiM1FpT201MWJHd3NJbkpsYm1SbGNsOXRiMlJsSWpvaVkzTnpJaXdpY21sbmFIUmZkVzVwZEhNaU9pSnpZM0psWlc0aUxDSjBiM0JmZFc1cGRITWlPaUp6WTNKbFpXNGlmU3dpYVdRaU9pSmpaREF4TjJaaE1TMHpNVEU0TFRRNE9EUXRZVGt5WkMwd04ySTJPR1JrTVdGak1tVWlMQ0owZVhCbElqb2lRbTk0UVc1dWIzUmhkR2x2YmlKOUxIc2lZWFIwY21saWRYUmxjeUk2ZXlKaVlYTmxJam95TkN3aWJXRnVkR2x6YzJGeklqcGJNU3d5TERRc05pdzRMREV5WFN3aWJXRjRYMmx1ZEdWeWRtRnNJam8wTXpJd01EQXdNQzR3TENKdGFXNWZhVzUwWlhKMllXd2lPak0yTURBd01EQXVNQ3dpYm5WdFgyMXBibTl5WDNScFkydHpJam93ZlN3aWFXUWlPaUpsT1dJMU1EVXdaUzAxWWpBekxUUTVOak10T1RoaFl5MDRZVE5rTlRsaU9XRTROVEFpTENKMGVYQmxJam9pUVdSaGNIUnBkbVZVYVdOclpYSWlmU3g3SW1GMGRISnBZblYwWlhNaU9uc2ljR3h2ZENJNmJuVnNiQ3dpZEdWNGRDSTZJalEwTURFekluMHNJbWxrSWpvaVlUWmlNekU0TkRNdFl6ZzJNeTAwTXpNekxXRXdNVEl0WldWak5XUTNZVEF3WlRnNUlpd2lkSGx3WlNJNklsUnBkR3hsSW4wc2V5SmhkSFJ5YVdKMWRHVnpJanA3SW1OaGJHeGlZV05ySWpwdWRXeHNMQ0p3Ykc5MElqcDdJbWxrSWpvaVlqVTJZalUwTm1FdFpEUmpaQzAwWWpBMExXRm1Zakl0WlRBd09XSTBNMlJtWldSaklpd2ljM1ZpZEhsd1pTSTZJa1pwWjNWeVpTSXNJblI1Y0dVaU9pSlFiRzkwSW4wc0luSmxibVJsY21WeWN5STZXM3NpYVdRaU9pSXdOamd5WlRnMU9TMW1NRGMwTFRRME5qZ3RZVE5qTWkwME9EVmxZakkxWWpOaU1XUWlMQ0owZVhCbElqb2lSMng1Y0doU1pXNWtaWEpsY2lKOVhTd2lkRzl2YkhScGNITWlPbHRiSWs1aGJXVWlMQ0puYkc5aVlXd2lYU3hiSWtKcFlYTWlMQ0l0TUM0Mk1DSmRMRnNpVTJ0cGJHd2lMQ0l3TGpjMUlsMWRmU3dpYVdRaU9pSmlNVFUzWXpnek15MDNZbUkzTFRSaE9XUXRZVEEyT1Mwd05qQmxaRFl5T0RjeE1tTWlMQ0owZVhCbElqb2lTRzkyWlhKVWIyOXNJbjBzZXlKaGRIUnlhV0oxZEdWeklqcDdJbTF2Ym5Sb2N5STZXekFzTVN3eUxETXNOQ3cxTERZc055dzRMRGtzTVRBc01URmRmU3dpYVdRaU9pSTFNR0ptT1Rka05DMDFPVEpqTFRRMVkySXRPREF6TlMwNFpXRTFPR0ZsWXpnMFlUVWlMQ0owZVhCbElqb2lUVzl1ZEdoelZHbGphMlZ5SW4wc2V5SmhkSFJ5YVdKMWRHVnpJanA3SW1SaGRHRmZjMjkxY21ObElqcDdJbWxrSWpvaVpqUXpZak0xTVRZdFlqSXdOaTAwTW1ZM0xUZ3lPR1V0TURBd05ETmxaamhrWVRBMUlpd2lkSGx3WlNJNklrTnZiSFZ0YmtSaGRHRlRiM1Z5WTJVaWZTd2laMng1Y0dnaU9uc2lhV1FpT2lJME1HSXlNVFJrT1MxalpXRTNMVFJqTkRjdFlqUTBNaTFqWkRRM01qYzVNVGMwTldJaUxDSjBlWEJsSWpvaVRHbHVaU0o5TENKb2IzWmxjbDluYkhsd2FDSTZiblZzYkN3aWJYVjBaV1JmWjJ4NWNHZ2lPbTUxYkd3c0ltNXZibk5sYkdWamRHbHZibDluYkhsd2FDSTZleUpwWkNJNklqSmlNRFl5TXpkbExUZG1aVGt0TkdOaFlpMDRZV0V5TFRVNE9HTTJNakZqT1dFMU5TSXNJblI1Y0dVaU9pSk1hVzVsSW4wc0luTmxiR1ZqZEdsdmJsOW5iSGx3YUNJNmJuVnNiSDBzSW1sa0lqb2lNRFk0TW1VNE5Ua3RaakEzTkMwME5EWTRMV0V6WXpJdE5EZzFaV0l5TldJellqRmtJaXdpZEhsd1pTSTZJa2RzZVhCb1VtVnVaR1Z5WlhJaWZTeDdJbUYwZEhKcFluVjBaWE1pT25zaWJXOXVkR2h6SWpwYk1DdzBMRGhkZlN3aWFXUWlPaUk0TkRoaVpHVmxZeTFoT1dJM0xUUTRaakF0T1RBd015MDVOakl4Wm1ObU5HUTVOR1lpTENKMGVYQmxJam9pVFc5dWRHaHpWR2xqYTJWeUluMHNleUpoZEhSeWFXSjFkR1Z6SWpwN0lteHBibVZmWVd4d2FHRWlPbnNpZG1Gc2RXVWlPakF1TVgwc0lteHBibVZmWTJGd0lqb2ljbTkxYm1RaUxDSnNhVzVsWDJOdmJHOXlJanA3SW5aaGJIVmxJam9pSXpGbU56ZGlOQ0o5TENKc2FXNWxYMnB2YVc0aU9pSnliM1Z1WkNJc0lteHBibVZmZDJsa2RHZ2lPbnNpZG1Gc2RXVWlPalY5TENKNElqcDdJbVpwWld4a0lqb2llQ0o5TENKNUlqcDdJbVpwWld4a0lqb2llU0o5ZlN3aWFXUWlPaUl5WWpBMk1qTTNaUzAzWm1VNUxUUmpZV0l0T0dGaE1pMDFPRGhqTmpJeFl6bGhOVFVpTENKMGVYQmxJam9pVEdsdVpTSjlMSHNpWVhSMGNtbGlkWFJsY3lJNmV5SnNhVzVsWDJOaGNDSTZJbkp2ZFc1a0lpd2liR2x1WlY5amIyeHZjaUk2ZXlKMllXeDFaU0k2SWlNNU9XUTFPVFFpZlN3aWJHbHVaVjlxYjJsdUlqb2ljbTkxYm1RaUxDSnNhVzVsWDNkcFpIUm9JanA3SW5aaGJIVmxJam8xZlN3aWVDSTZleUptYVdWc1pDSTZJbmdpZlN3aWVTSTZleUptYVdWc1pDSTZJbmtpZlgwc0ltbGtJam9pWWpZMU4ySTRNak10WWpjM01pMDBaalF6TFRsak5EVXRaak13WlRKa1pURmtZVFUySWl3aWRIbHdaU0k2SWt4cGJtVWlmU3g3SW1GMGRISnBZblYwWlhNaU9uc2lZMkZzYkdKaFkyc2lPbTUxYkd3c0luQnNiM1FpT25zaWFXUWlPaUppTlRaaU5UUTJZUzFrTkdOa0xUUmlNRFF0WVdaaU1pMWxNREE1WWpRelpHWmxaR01pTENKemRXSjBlWEJsSWpvaVJtbG5kWEpsSWl3aWRIbHdaU0k2SWxCc2IzUWlmU3dpY21WdVpHVnlaWEp6SWpwYmV5SnBaQ0k2SWpnNVl6WTFPVFl3TFdZNFlqZ3ROR1E0WmkwNFlUSm1MVEk1TlRnNE9EUmlObUUzTVNJc0luUjVjR1VpT2lKSGJIbHdhRkpsYm1SbGNtVnlJbjFkTENKMGIyOXNkR2x3Y3lJNlcxc2lUbUZ0WlNJc0ltTnZZWGR6ZEY4MFgzVnpaVjlpWlhOMElsMHNXeUpDYVdGeklpd2lNQzQyTnlKZExGc2lVMnRwYkd3aUxDSXdMamN6SWwxZGZTd2lhV1FpT2lJMU56UXdPRFF6TkMwNFl6VTJMVFExTmpBdFlqUmlPUzAzTnpWaE4yUmhZalV4T1RVaUxDSjBlWEJsSWpvaVNHOTJaWEpVYjI5c0luMHNleUpoZEhSeWFXSjFkR1Z6SWpwN0ltTmhiR3hpWVdOcklqcHVkV3hzZlN3aWFXUWlPaUpqTnprek1XVXhNUzAyTldFMExUUXlNR1l0T1RKalppMWlOakF4WVdJNE1qWTJOalVpTENKMGVYQmxJam9pUkdGMFlWSmhibWRsTVdRaWZTeDdJbUYwZEhKcFluVjBaWE1pT25zaWJHRmlaV3dpT25zaWRtRnNkV1VpT2lKRFQwRlhVMVJmTkNKOUxDSnlaVzVrWlhKbGNuTWlPbHQ3SW1sa0lqb2lPRGxqTmpVNU5qQXRaamhpT0MwMFpEaG1MVGhoTW1ZdE1qazFPRGc0TkdJMllUY3hJaXdpZEhsd1pTSTZJa2RzZVhCb1VtVnVaR1Z5WlhJaWZWMTlMQ0pwWkNJNklqQTRaalkxTW1NNUxUQXdNRGd0TkRRNU1TMWlOVFJtTFRrMVlXVXhaREZrT1RabE5TSXNJblI1Y0dVaU9pSk1aV2RsYm1SSmRHVnRJbjBzZXlKaGRIUnlhV0oxZEdWeklqcDdJbTF2Ym5Sb2N5STZXekFzTWl3MExEWXNPQ3d4TUYxOUxDSnBaQ0k2SW1Oak1HVm1PVEpsTFROaU9XVXROR1JrTkMxaFpXTTNMVEU1Tm1VME5tUXdNemt5TXlJc0luUjVjR1VpT2lKTmIyNTBhSE5VYVdOclpYSWlmU3g3SW1GMGRISnBZblYwWlhNaU9uc2lZMkZzYkdKaFkyc2lPbTUxYkd3c0luQnNiM1FpT25zaWFXUWlPaUppTlRaaU5UUTJZUzFrTkdOa0xUUmlNRFF0WVdaaU1pMWxNREE1WWpRelpHWmxaR01pTENKemRXSjBlWEJsSWpvaVJtbG5kWEpsSWl3aWRIbHdaU0k2SWxCc2IzUWlmU3dpY21WdVpHVnlaWEp6SWpwYmV5SnBaQ0k2SWpJd1ltTXdZamd5TFdNMVl6VXROR1JsTUMwNFpHWmpMVGs0TnpRMFltTTVaVEl6TkNJc0luUjVjR1VpT2lKSGJIbHdhRkpsYm1SbGNtVnlJbjFkTENKMGIyOXNkR2x3Y3lJNlcxc2lUbUZ0WlNJc0lrY3hYMU5UVkY5SFRFOUNRVXdpWFN4YklrSnBZWE1pTENJdE1DNDBOeUpkTEZzaVUydHBiR3dpTENJd0xqTTVJbDFkZlN3aWFXUWlPaUpqTWpFeU56Y3haUzAwWldKaUxUUmpPVFV0WVdJd1pDMWxOamswTXpFNU16Z3lORGdpTENKMGVYQmxJam9pU0c5MlpYSlViMjlzSW4wc2V5SmhkSFJ5YVdKMWRHVnpJanA3SW1OaGJHeGlZV05ySWpwdWRXeHNMQ0pqYjJ4MWJXNWZibUZ0WlhNaU9sc2llQ0lzSW5raVhTd2laR0YwWVNJNmV5SjRJanA3SWw5ZmJtUmhjbkpoZVY5Zklqb2lRVUZEWjB0UGJrOWtWVWxCUVVscFdEZE5OVEZSWjBGQlkwRmlkM3B1VmtNaUxDSmtkSGx3WlNJNkltWnNiMkYwTmpRaUxDSnphR0Z3WlNJNld6TmRmU3dpZVNJNmV5SmZYMjVrWVhKeVlYbGZYeUk2SWtGQlFVRTBUREZ3VEd0QlFVRkJSR2QyVjJ0MVVVRkJRVUZQUXpsaFV6VkJJaXdpWkhSNWNHVWlPaUptYkc5aGREWTBJaXdpYzJoaGNHVWlPbHN6WFgxOWZTd2lhV1FpT2lKbE4yWTJabUZtTWkxak1UWmtMVFEwTjJJdFlXSTNNaTFqTldWak1HSTNNVGRpTldVaUxDSjBlWEJsSWpvaVEyOXNkVzF1UkdGMFlWTnZkWEpqWlNKOUxIc2lZWFIwY21saWRYUmxjeUk2ZXlKc2FXNWxYMk5oY0NJNkluSnZkVzVrSWl3aWJHbHVaVjlqYjJ4dmNpSTZleUoyWVd4MVpTSTZJaU16TWpnNFltUWlmU3dpYkdsdVpWOXFiMmx1SWpvaWNtOTFibVFpTENKc2FXNWxYM2RwWkhSb0lqcDdJblpoYkhWbElqbzFmU3dpZUNJNmV5Sm1hV1ZzWkNJNkluZ2lmU3dpZVNJNmV5Sm1hV1ZzWkNJNklua2lmWDBzSW1sa0lqb2laV1F3WlRjM1pXRXRaR1kxTkMwMFpXWTVMVGxpTUdNdE16YzBaVFZrWVRJeU9ESTRJaXdpZEhsd1pTSTZJa3hwYm1VaWZTeDdJbUYwZEhKcFluVjBaWE1pT250OUxDSnBaQ0k2SWpZellqVTVZVFk0TFRjeU1HWXROR0ZtTmkwNFpEUTBMVFEyTjJRME5tTXlObVprTXlJc0luUjVjR1VpT2lKWlpXRnljMVJwWTJ0bGNpSjlMSHNpWVhSMGNtbGlkWFJsY3lJNmV5SnNhVzVsWDJGc2NHaGhJanA3SW5aaGJIVmxJam93TGpGOUxDSnNhVzVsWDJOaGNDSTZJbkp2ZFc1a0lpd2liR2x1WlY5amIyeHZjaUk2ZXlKMllXeDFaU0k2SWlNeFpqYzNZalFpZlN3aWJHbHVaVjlxYjJsdUlqb2ljbTkxYm1RaUxDSnNhVzVsWDNkcFpIUm9JanA3SW5aaGJIVmxJam8xZlN3aWVDSTZleUptYVdWc1pDSTZJbmdpZlN3aWVTSTZleUptYVdWc1pDSTZJbmtpZlgwc0ltbGtJam9pTnpNek1EZG1NRFV0WVRrMFlTMDBaVGszTFdFMVpHVXRNekF3TVdVNE1EbG1PRFV5SWl3aWRIbHdaU0k2SWt4cGJtVWlmU3g3SW1GMGRISnBZblYwWlhNaU9uc2liVzl1ZEdoeklqcGJNQ3cyWFgwc0ltbGtJam9pWVdNM1pHUmtNbUV0T1daaE5DMDBZemRpTFRnMk9XRXRaVEUxWTJWbU5UWmhPVGcySWl3aWRIbHdaU0k2SWsxdmJuUm9jMVJwWTJ0bGNpSjlMSHNpWVhSMGNtbGlkWFJsY3lJNmV5SmtZWFJoWDNOdmRYSmpaU0k2ZXlKcFpDSTZJbVUzWmpabVlXWXlMV014Tm1RdE5EUTNZaTFoWWpjeUxXTTFaV013WWpjeE4ySTFaU0lzSW5SNWNHVWlPaUpEYjJ4MWJXNUVZWFJoVTI5MWNtTmxJbjBzSW1kc2VYQm9JanA3SW1sa0lqb2laV1F3WlRjM1pXRXRaR1kxTkMwMFpXWTVMVGxpTUdNdE16YzBaVFZrWVRJeU9ESTRJaXdpZEhsd1pTSTZJa3hwYm1VaWZTd2lhRzkyWlhKZloyeDVjR2dpT201MWJHd3NJbTExZEdWa1gyZHNlWEJvSWpwdWRXeHNMQ0p1YjI1elpXeGxZM1JwYjI1ZloyeDVjR2dpT25zaWFXUWlPaUkzTXpNd04yWXdOUzFoT1RSaExUUmxPVGN0WVRWa1pTMHpNREF4WlRnd09XWTROVElpTENKMGVYQmxJam9pVEdsdVpTSjlMQ0p6Wld4bFkzUnBiMjVmWjJ4NWNHZ2lPbTUxYkd4OUxDSnBaQ0k2SWprMFl6WmlaRGd5TFdZeU4yWXRORGMzTkMwNU5XRXdMVGhqTURkaVl6Qm1PV00xTlNJc0luUjVjR1VpT2lKSGJIbHdhRkpsYm1SbGNtVnlJbjBzZXlKaGRIUnlhV0oxZEdWeklqcDdJbU5oYkd4aVlXTnJJanB1ZFd4c0xDSmpiMngxYlc1ZmJtRnRaWE1pT2xzaWVDSXNJbmtpWFN3aVpHRjBZU0k2ZXlKNElqcDdJbDlmYm1SaGNuSmhlVjlmSWpvaVFVRkNRWGRYVkZCa1ZVbEJRVU5uZDJGTk9URlJaMEZCUlVvNWNub3pWa05CUVVRMFJGY3ZVR1JWU1VGQlQwSTRZM001TVZGblFVRjVUM1F4ZWpOV1EwRkJRM2RYYm01UVpGVkpRVUZLYWtwbVRUa3hVV2RCUVdkRWFVRjZNMVpEUVVGQ2IzQTBVRkJrVlVsQlFVWkJWMmc0T1RGUlowRkJUMGxYUzNvelZrTkJRVUZuT1VrelVHUlZTVUZCUVdocWEyTTVNVkZuUVVFNFRrZFZlak5XUTBGQlJGbFJTbXBRWkZWSlFVRk5RM1p0T0RreFVXZEJRWEZDTm1aNk0xWkRRVUZEVVdwaFRGQmtWVWxCUVVocU9IQmpPVEZSWjBGQldVZDFjSG96VmtOQlFVSkpNbkY2VUdSVlNVRkJSRUpLYzAwNU1WRm5RVUZIVEdsNmVqTldRMEZCUVVGS04yWlFaRlZKUVVGUGFWWjFjemt4VVdkQlFUQkJVeXQ2TTFaRFFVRkROR000U0ZCa1ZVbEJRVXRFYVhoTk9URlJaMEZCYVVaSVNYb3pWa05CUVVKM2QwMTJVR1JWU1VGQlJtZDJlamc1TVZGblFVRlJTamRUZWpOV1EwRkJRVzlFWkdKUVpGVkpRVUZDUWpneVl6a3hVV2RCUVN0UGNtTjZNMVpEUVVGRVoxZGxSRkJrVlVsQlFVMXFTVFE0T1RGUlowRkJjMFJtYm5velZrTkJRVU5aY0hWeVVHUlZTVUZCU1VGV04zTTVNVkZuUVVGaFNWUjRlak5XUTBGQlFsRTRMMVJRWkZWSlFVRkVhR2tyVFRreFVXZEJRVWxPU0RkNk0xWkRRVUZCU1ZGUUwxQmtWVWxCUVZCRGRVRjBRakZSWjBGQk1rSXdSekJJVmtOQlFVUkJha0Z1VVdSVlNVRkJTMm8zUkU1Q01WRm5RVUZyUjI5Uk1FaFdRMEZCUWpReVVsQlJaRlZKUVVGSFFrbEdPVUl4VVdkQlFWTk1ZMkV3U0ZaRFFVRkJkMHBvTjFGa1ZVbEJRVUpwVmtsa1FqRlJaMEZCUVVGUmJEQklWa05CUVVSdlkybHFVV1JWU1VGQlRrUm9TemxDTVZGblFVRjFSa0YyTUVoV1EwRkJRMmQyZWt4UlpGVkpRVUZKWjNWT2RFSXhVV2RCUVdOS01EVXdTRlpEUVVGQ1dVUkVNMUZrVlVsQlFVVkNOMUZPUWpGUlowRkJTMDl3UkRCSVZrTkJRVUZSVjFWbVVXUlZTVUZCVUdwSVUzUkNNVkZuUVVFMFJGcFBNRWhXUTBGQlJFbHdWa2hSWkZWSlFVRk1RVlZXWkVJeFVXZEJRVzFKVGxrd1NGWkRRVUZEUVRoc2RsRmtWVWxCUVVkb2FGZzVRakZSWjBGQlZVNUNhVEJJVmtOQlFVRTBVREppVVdSVlNVRkJRME4xWVdSQ01WRm5RVUZEUWpGME1FaFdRMEZCUkhkcE0wUlJaRlZKUVVGT2FqWmpPVUl4VVdkQlFYZEhiRE13U0ZaRFFVRkRiekpJY2xGa1ZVbEJRVXBDU0daMFFqRlJaMEZCWlV4aFFqQklWa05CUVVKblNsbFlVV1JWU1VGQlJXbFZhVTVDTVZGblFVRk5RVTlOTUVoV1EwRkJRVmxqYnk5UlpGVkpRVUZCUkdocmRFSXhVV2RCUVRaRksxY3dTRlpEUVVGRVVYWndibEZrVlVsQlFVeG5kRzVrUWpGUlowRkJiMHA1WnpCSVZrTkJRVU5KUXpaVVVXUlZTVUZCU0VJMmNEbENNVkZuUVVGWFQyMXhNRWhXUTBGQlFrRlhTemRSWkZWSlFVRkRha2h6WkVJeFVXZEJRVVZFWVRFd1NGWkRRVUZFTkhCTWFsRmtWVWxCUVU5QlZIWk9RakZSWjBGQmVVbExMekJJVmtOQlFVTjNPR05NVVdSVlNVRkJTbWhuZUhSQ01WRm5RVUZuVFM5S01FaFdRMEZCUW05UWN6TlJaRlZKUVVGR1EzUXdUa0l4VVdkQlFVOUNlbFV3U0ZaRFFVRkJaMms1WmxGa1ZVbEJRVUZxTmpKMFFqRlJaMEZCT0VkcVpUQklWa05CUVVSWk1TdElVV1JWU1VGQlRVSkhOV1JDTVZGblFVRnhURmh2TUVoV1EwRkJRMUZLVDNwUlpGVkpRVUZJYVZRM09VSXhVV2RCUVZsQlRIb3dTRlpEUVVGQ1NXTm1ZbEZrVlVsQlFVUkVaeXRrUWpGUlowRkJSMFV2T1RCSVZrTkJRVUZCZG1kRVVtUlZTVUZCVDJkelFrNUdNVkZuUVVFd1NuTklNRmhXUTBGQlF6UkRaM1pTWkZWSlFVRkxRalZFZEVZeFVXZEJRV2xQWjFJd1dGWkRRVUZDZDFaNFdGSmtWVWxCUVVacVIwZE9SakZSWjBGQlVVUlZZekJZVmtOQlFVRnZjRUl2VW1SVlNVRkJRa0ZVU1RsR01WRm5RVUVyU1VWdE1GaFdRMEZCUkdjNFEyNVNaRlZKUVVGTmFHWk1aRVl4VVdkQlFYTk5OSGN3V0ZaRFFVRkRXVkJVVkZKa1ZVbEJRVWxEYzA0NVJqRlJaMEZCWVVKek56QllWa05CUVVKUmFXbzNVbVJWU1VGQlJHbzFVV1JHTVZGblFVRkpSMmhHTUZoV1EwRkJRVWt4TUdwU1pGVkpRVUZRUWtaVVRrWXhVV2RCUVRKTVVsQXdXRlpESWl3aVpIUjVjR1VpT2lKbWJHOWhkRFkwSWl3aWMyaGhjR1VpT2xzeE5EUmRmU3dpZVNJNmV5SmZYMjVrWVhKeVlYbGZYeUk2SWtGQlFVRjNRVUkyUzFWQlFVRkJRV2N4TlZGeVVVRkJRVUZIUTNSeWVURkJRVUZCUVhkSlVFdE1NRUZCUVVGRFFVVTRjM1pSUVVGQlFVTkRhbmw1T1VGQlFVRkJORVJNVFV3d1FVRkJRVUZCTTJOTmRsRkJRVUZCUlVOSWRYazVRVUZCUVVGWlJFZDZUREJCUVVGQlFVRlNUVTEyVVVGQlFVRkpRbGN3ZVRsQlFVRkJRVWxIYm1wTU1FRkJRVUZEWjFKUVkzWlJRVUZCUVVGRFVVSlVRa0ZCUVVGQmQwZ3dVRTFGUVVGQlFVRm5WVUUwZDFGQlFVRkJTMEZwUkZSQ1FVRkJRVUZCVUZWTVRVVkJRVUZCUkVFd1VXTjNVVUZCUVVGSFEzVkJla0pCUVVGQlFVbENZaTlNTUVGQlFVRkJRWGhuU1hkUlFVRkJRVUZCUWtKcVFrRkJRVUZCTkVSelNrMUZRVUZCUVVSQmFsWk5kMUZCUVVGQlRVUm1ibFJDUVVGQlFVRnZSRWh2VFVWQlFVRkJRa0ZpYzFsM1VVRkJRVUZQUTNGd1JFSkJRVUZCUVdkUFpVTk5SVUZCUVVGRVoxbHRTWGRSUVVGQlFVZEVaVkZVUWtGQlFVRkJkMFpyYUUxRlFVRkJRVUpuWlVKQmQxRkJRVUZCUTBGMUwzazVRVUZCUVVGWlIzWmtUREJCUVVGQlEwRTJaWGQyVVVGQlFVRkxRbTR2UXpsQlFVRkJRVFJRU1VaTlJVRkJRVUZFWjAxQ1NYZFJRVUZCUVUxQ2RVaHFRa0ZCUVVGQmQwdDNjVTFGUVVGQlFVTm5aM2xyZDFGQlFVRkJTVUpoUzBSQ1FVRkJRVUZaUkVWdVRVVkJRVUZCUVVFdlFUUjNVVUZCUVVGRFEwNDNVemxCUVVGQlFWRkRTemxNTUVGQlFVRkNRV1ZEYzNkUlFVRkJRVWRDWm1WRVFrRkJRVUZCWjBWaVJrMUZRVUZCUVVKQmVFeEZkMUZCUVVGQlQwSkNibXBDUVVGQlFVRnZUQ3RMVFVWQlFVRkJSR2RhU0VGM1VVRkJRVUZGUVV0V2FrSkJRVUZCUVdkTE9EZE5SVUZCUVVGRFoycERaM2RSUVVGQlFVdENjRVpVUWtGQlFVRkJkMFZaUTAxRlFVRkJRVUpCWjNkWmQxRkJRVUZCVDBNdlEycENRVUZCUVVGWlVIZFBUVVZCUVVGQlEyZDFRamgzVVVGQlFVRkJRakZOUkVKQlFVRkJRVkZFUmtKTlJVRkJRVUZFWjFkcU1IZFJRVUZCUVVkRFJVOVVRa0ZCUVVGQlFVczBNVTFGUVVGQlFVTm5ja05OZDFGQlFVRkJSVU55UlZSQ1FVRkJRVUUwUmxBdlREQkJRVUZCUkdjNGNVbDJVVUZCUVVGUFExSlNhVGxCUVVGQlFUUkVSSEZNYTBGQlFVRkRaeXR5WjNWUlFVRkJRVWxFUldoNU5VRkJRVUZCVVVrMVYweHJRVUZCUVVSQlYzbFpkVkZCUVVGQlJVRndPV2t4UVVGQlFVRjNVR0pHVEZWQlFVRkJSRUZyTlRoMFVVRkJRVUZQUVhkbFV6RkJRVUZCUVRSTk1WTk1WVUZCUVVGRFoxaEdhM1JSUVVGQlFVZEVjbGg1TVVGQlFVRkJTVWh3YlV4VlFVRkJRVVJuYTI5SmRGRkJRVUZCVFVOeWJta3hRVUZCUVVGblRWTTJURlZCUVVGQlFrRkRObTkwVVVGQlFVRkRRbE50VXpGQlFVRkJRVFJLYVVsTVZVRkJRVUZDUVdsR01IUlJRVUZCUVVsQ00wMXBNVUZCUVVGQk5FZFpTRXhWUVVGQlFVUkJhVGxyYzFGQlFVRkJUVU4zY1hsNFFVRkJRVUZ2VGxZNVRFVkJRVUZCUTBGM1ZtOXpVVUZCUVVGSFEzUk9lWGhCUVVGQlFWRkthMVZNUlVGQlFVRkRRV0oxYzNKUlFVRkJRVTFDUkhkcGRFRkJRVUZCUVVKdFdrc3dRVUZCUVVKQmFXOUZjbEZCUVVGQlMwUTNZVk4wUVVGQlFVRTBSM2hUU3pCQlFVRkJRa0ZYV0VWeVVVRkJRVUZMUWtaclEzUkJRVUZCUVVGRVMzWkxNRUZCUVVGRFowUmtZM0pSUVVGQlFVZEVjQzlwZEVGQlFVRkJRVTFWYlV4RlFVRkJRVU5uY0VNd2MxRkJRVUZCUlVORlRrTjRRVUZCUVVFMFIwMDNURVZCUVVGQlFtZEdVbXR6VVVGQlFVRk5SRWM1YVhSQlFVRkJRVkZJYWxWTE1FRkJRVUZFWjJZM2MzSlJRVUZCUVVkRFNHOXBkRUZCUVVGQlFVa3JTa3N3UVVGQlFVUkJTMGhKY2xGQlFVRkJTVVJEVjJsMFFVRkJRVUZSUm5oRVN6QkJRVUZCUVVGcWVVbHlVVUZCUVVGTlJFSkJVM1JCUVVGQlFXZFFWR2RMYTBGQlFVRkJRWFJ6UlhGUlFVRkJRVXRDTTI5cGNFRkJRVUZCU1VSdFJFdHJRVUZCUVVSbmJWcEJjVkZCUVVGQlRVUTJibE53UVVGQlFVRm5SblZ5UzJ0QlFVRkJRMEY1T1dOeFVVRkJRVUZKUVRkQ1EzUkJRVUZCUVdkTGMzZExNRUZCUVVGRVowRkZSWEpSUVVGQlFVVkNWMVZUZEVGQlFVRkJiMHQwYUVzd1FVRkJRVU5uY1RKRmNsRkJRVUZCUzBOeVdWTjBRU0lzSW1SMGVYQmxJam9pWm14dllYUTJOQ0lzSW5Ob1lYQmxJanBiTVRRMFhYMTlmU3dpYVdRaU9pSmpObVE0TkdRNE9TMDRNR1ZqTFRReE5EQXRPV0pqTWkxbU5HSTBOVEkyWkRrM1l6RWlMQ0owZVhCbElqb2lRMjlzZFcxdVJHRjBZVk52ZFhKalpTSjlMSHNpWVhSMGNtbGlkWFJsY3lJNmV5SnBkR1Z0Y3lJNlczc2lhV1FpT2lKbU9EZ3pOR1F5TlMwd1pqTmhMVFJtWldFdE9UVTBNQzB6TXpaak16WXpabUpoWVRBaUxDSjBlWEJsSWpvaVRHVm5aVzVrU1hSbGJTSjlMSHNpYVdRaU9pSTRZemhrTkRrellTMDNZVGRtTFRSak5HVXRPREpsWWkwd1ltUmhNbVkzT1Rrek1EQWlMQ0owZVhCbElqb2lUR1ZuWlc1a1NYUmxiU0o5TEhzaWFXUWlPaUppWWpjMFlUVmxOaTA0WW1GakxUUTBNbVV0T0dFMk15MHdOVEJtTURnMU5EaGpZbUVpTENKMGVYQmxJam9pVEdWblpXNWtTWFJsYlNKOUxIc2lhV1FpT2lJM05HVXlNVEZsTlMweE5EZzVMVFF4WkRVdFltSTVNQzB6TkdSaE5UTXpZbU0yWkdRaUxDSjBlWEJsSWpvaVRHVm5aVzVrU1hSbGJTSjlMSHNpYVdRaU9pSmhOek5oWXpJMk1DMDVOemhpTFRSa1lUVXRPVE5oTmkweVlqUTJOakEzTmpaaVpERWlMQ0owZVhCbElqb2lUR1ZuWlc1a1NYUmxiU0o5TEhzaWFXUWlPaUl3T0dZMk5USmpPUzB3TURBNExUUTBPVEV0WWpVMFppMDVOV0ZsTVdReFpEazJaVFVpTENKMGVYQmxJam9pVEdWblpXNWtTWFJsYlNKOUxIc2lhV1FpT2lJelltVXhOelF3TUMwMU16UTJMVFJqWldNdFlqUmhZeTAzTlRJNU0yTmpORFE0TmpRaUxDSjBlWEJsSWpvaVRHVm5aVzVrU1hSbGJTSjlMSHNpYVdRaU9pSTJaV1ZsWVdJNE5TMHdZVEV4TFRRMk5tSXRPVE5pTXkxbU9UazVNV1ptWlRSa09Ua2lMQ0owZVhCbElqb2lUR1ZuWlc1a1NYUmxiU0o5WFN3aWNHeHZkQ0k2ZXlKcFpDSTZJbUkxTm1JMU5EWmhMV1EwWTJRdE5HSXdOQzFoWm1JeUxXVXdNRGxpTkROa1ptVmtZeUlzSW5OMVluUjVjR1VpT2lKR2FXZDFjbVVpTENKMGVYQmxJam9pVUd4dmRDSjlmU3dpYVdRaU9pSmxOalEzTTJKaU1DMDBaVFJpTFRReVltUXRZamhoTlMwM1pqVXdNR0poWlRGaE16VWlMQ0owZVhCbElqb2lUR1ZuWlc1a0luMHNleUpoZEhSeWFXSjFkR1Z6SWpwN0ltUmhkR0ZmYzI5MWNtTmxJanA3SW1sa0lqb2lOVEZoTkRReE16QXRaVEkzTmkwME9USmlMV0ZpT0RNdE9XVmxZekE1TlRBeU9HWmtJaXdpZEhsd1pTSTZJa052YkhWdGJrUmhkR0ZUYjNWeVkyVWlmU3dpWjJ4NWNHZ2lPbnNpYVdRaU9pSTFNVFl4T0RJM09TMHpORGt6TFRRek9ESXRPREppTlMwNE9EYzNZbVF4TjJSbE9EWWlMQ0owZVhCbElqb2lUR2x1WlNKOUxDSm9iM1psY2w5bmJIbHdhQ0k2Ym5Wc2JDd2liWFYwWldSZloyeDVjR2dpT201MWJHd3NJbTV2Ym5ObGJHVmpkR2x2Ymw5bmJIbHdhQ0k2ZXlKcFpDSTZJamd5WVdRNVlUTXpMVFJsWlRNdE5HRTROQzA1WVdGaExUaGhPV0ZrWkRaak16RTRPQ0lzSW5SNWNHVWlPaUpNYVc1bEluMHNJbk5sYkdWamRHbHZibDluYkhsd2FDSTZiblZzYkgwc0ltbGtJam9pT0Rsak5qVTVOakF0WmpoaU9DMDBaRGhtTFRoaE1tWXRNamsxT0RnNE5HSTJZVGN4SWl3aWRIbHdaU0k2SWtkc2VYQm9VbVZ1WkdWeVpYSWlmU3g3SW1GMGRISnBZblYwWlhNaU9uc2liR2x1WlY5allYQWlPaUp5YjNWdVpDSXNJbXhwYm1WZlkyOXNiM0lpT25zaWRtRnNkV1VpT2lJalpEVXpaVFJtSW4wc0lteHBibVZmYW05cGJpSTZJbkp2ZFc1a0lpd2liR2x1WlY5M2FXUjBhQ0k2ZXlKMllXeDFaU0k2Tlgwc0luZ2lPbnNpWm1sbGJHUWlPaUo0SW4wc0lua2lPbnNpWm1sbGJHUWlPaUo1SW4xOUxDSnBaQ0k2SWpVeE5qRTRNamM1TFRNME9UTXRORE00TWkwNE1tSTFMVGc0TnpkaVpERTNaR1U0TmlJc0luUjVjR1VpT2lKTWFXNWxJbjBzZXlKaGRIUnlhV0oxZEdWeklqcDdJbXhoWW1Wc0lqcDdJblpoYkhWbElqb2lSekZmVTFOVVgwZE1UMEpCVENKOUxDSnlaVzVrWlhKbGNuTWlPbHQ3SW1sa0lqb2lNakJpWXpCaU9ESXRZelZqTlMwMFpHVXdMVGhrWm1NdE9UZzNORFJpWXpsbE1qTTBJaXdpZEhsd1pTSTZJa2RzZVhCb1VtVnVaR1Z5WlhJaWZWMTlMQ0pwWkNJNkltWTRPRE0wWkRJMUxUQm1NMkV0TkdabFlTMDVOVFF3TFRNek5tTXpOak5tWW1GaE1DSXNJblI1Y0dVaU9pSk1aV2RsYm1SSmRHVnRJbjBzZXlKaGRIUnlhV0oxZEdWeklqcDdJbXhwYm1WZllXeHdhR0VpT25zaWRtRnNkV1VpT2pBdU1YMHNJbXhwYm1WZlkyRndJam9pY205MWJtUWlMQ0pzYVc1bFgyTnZiRzl5SWpwN0luWmhiSFZsSWpvaUl6Rm1OemRpTkNKOUxDSnNhVzVsWDJwdmFXNGlPaUp5YjNWdVpDSXNJbXhwYm1WZmQybGtkR2dpT25zaWRtRnNkV1VpT2pWOUxDSjRJanA3SW1acFpXeGtJam9pZUNKOUxDSjVJanA3SW1acFpXeGtJam9pZVNKOWZTd2lhV1FpT2lJNE1tRmtPV0V6TXkwMFpXVXpMVFJoT0RRdE9XRmhZUzA0WVRsaFpHUTJZek14T0RnaUxDSjBlWEJsSWpvaVRHbHVaU0o5TEhzaVlYUjBjbWxpZFhSbGN5STZlMzBzSW1sa0lqb2lPV1ExWkRCa1pUWXRZekZsTXkwME16a3hMV0V5WWpBdE1XUTVNMkU1WlRjMk5HTTBJaXdpZEhsd1pTSTZJa0poYzJsalZHbGphMlZ5SW4wc2V5SmhkSFJ5YVdKMWRHVnpJanA3SW14aFltVnNJanA3SW5aaGJIVmxJam9pWkdoM1h6VnJiU0o5TENKeVpXNWtaWEpsY25NaU9sdDdJbWxrSWpvaU9UUmpObUprT0RJdFpqSTNaaTAwTnpjMExUazFZVEF0T0dNd04ySmpNR1k1WXpVMUlpd2lkSGx3WlNJNklrZHNlWEJvVW1WdVpHVnlaWElpZlYxOUxDSnBaQ0k2SWpOaVpURTNOREF3TFRVek5EWXROR05sWXkxaU5HRmpMVGMxTWprelkyTTBORGcyTkNJc0luUjVjR1VpT2lKTVpXZGxibVJKZEdWdEluMHNleUpoZEhSeWFXSjFkR1Z6SWpwN0ltTmhiR3hpWVdOcklqcHVkV3hzTENKamIyeDFiVzVmYm1GdFpYTWlPbHNpZUNJc0lua2lYU3dpWkdGMFlTSTZleUo0SWpwN0lsOWZibVJoY25KaGVWOWZJam9pUVVGRVFUbGlMMDlrVlVsQlFVdG9hM2M0TlRGUlowRkJhMDVRUjNwdVZrTkJRVUkwVVhOeVQyUlZTVUZCUjBONGVtTTFNVkZuUVVGVFEwUlNlbTVXUTBGQlFYZHFPVlJQWkZWSlFVRkNhaXN4T0RVeFVXZEJRVUZITTJKNmJsWkRRVUZFYnpJNU4wOWtWVWxCUVU1Q1N6UnpOVEZSWjBGQmRVeHViSHB1VmtOQlFVTm5TMDl1VDJSVlNVRkJTV2xZTjAwMU1WRm5RVUZqUVdKM2VtNVdRMEZCUWxsa1psQlBaRlZKUVVGRlJHczVjelV4VVdkQlFVdEdVRFo2YmxaRFFVRkJVWGQyTTA5a1ZVbEJRVkJuZDBGak9URlJaMEZCTkVvNFJYb3pWa05CUVVSSlJHZHFVR1JWU1VGQlRFSTVRemc1TVZGblFVRnRUM2RQZWpOV1EwRkJRMEZYZUV4UVpGVkpRVUZIYWt0R1l6a3hVV2RCUVZWRWExcDZNMVpEUVVGQk5IRkNlbEJrVlVsQlFVTkJXRWxOT1RGUlowRkJRMGxaYW5velZrTkJRVVIzT1VOaVVHUlZTVUZCVG1ocVMzTTVNVkZuUVVGM1RrbDBlak5XUTBGQlEyOVJWRWhRWkZWSlFVRktRM2RPVFRreFVXZEJRV1ZDT0RSNk0xWkRRVUZDWjJwcWRsQmtWVWxCUVVWcU9WQnpPVEZSWjBGQlRVZDRRM296VmtOQlFVRlpNakJZVUdSVlNVRkJRVUpMVTJNNU1WRm5RVUUyVEdoTmVqTldRMEZCUkZGS01VUlFaRlZKUVVGTWFWZFZPRGt4VVdkQlFXOUJWbGg2TTFaRFFVRkRTV1JHY2xCa1ZVbEJRVWhFYWxoak9URlJaMEZCVjBaS2FIb3pWa05CUVVKQmQxZFVVR1JWU1VGQlEyZDNZVTA1TVZGblFVRkZTamx5ZWpOV1EwRkJSRFJFVnk5UVpGVkpRVUZQUWpoamN6a3hVV2RCUVhsUGRERjZNMVpEUVVGRGQxZHVibEJrVlVsQlFVcHFTbVpOT1RGUlowRkJaMFJwUVhvelZrTkJRVUp2Y0RSUVVHUlZTVUZCUmtGWGFEZzVNVkZuUVVGUFNWZExlak5XUTBGQlFXYzVTVE5RWkZWSlFVRkJhR3ByWXpreFVXZEJRVGhPUjFWNk0xWkRRVUZFV1ZGS2FsQmtWVWxCUVUxRGRtMDRPVEZSWjBGQmNVSTJabm96VmtOQlFVTlJhbUZNVUdSVlNVRkJTR280Y0dNNU1WRm5RVUZaUjNWd2VqTldRMEZCUWtreWNYcFFaRlZKUVVGRVFrcHpUVGt4VVdkQlFVZE1hWHA2TTFaRFFVRkJRVW8zWmxCa1ZVbEJRVTlwVm5Wek9URlJaMEZCTUVGVEszb3pWa05CUVVNMFl6aElVR1JWU1VGQlMwUnBlRTA1TVZGblFVRnBSa2hKZWpOV1EwRkJRbmQzVFhaUVpGVkpRVUZHWjNaNk9Ea3hVV2RCUVZGS04xTjZNMVpEUVVGQmIwUmtZbEJrVlVsQlFVSkNPREpqT1RGUlowRkJLMDl5WTNvelZrTkJRVVJuVjJWRVVHUlZTVUZCVFdwSk5EZzVNVkZuUVVGelJHWnVlak5XUTBGQlExbHdkWEpRWkZWSlFVRkpRVlkzY3preFVXZEJRV0ZKVkhoNk0xWkRRVUZDVVRndlZGQmtWVWxCUVVSb2FTdE5PVEZSWjBGQlNVNUlOM296VmtOQlFVRkpVVkF2VUdSVlNVRkJVRU4xUVhSQ01WRm5RVUV5UWpCSE1FaFdRMEZCUkVGcVFXNVJaRlZKUVVGTGFqZEVUa0l4VVdkQlFXdEhiMUV3U0ZaRFFVRkNOREpTVUZGa1ZVbEJRVWRDU1VZNVFqRlJaMEZCVTB4allUQklWa05CUVVGM1NtZzNVV1JWU1VGQlFtbFdTV1JDTVZGblFVRkJRVkZzTUVoV1EwRkJSRzlqYVdwUlpGVkpRVUZPUkdoTE9VSXhVV2RCUVhWR1FYWXdTRlpEUVVGRFozWjZURkZrVlVsQlFVbG5kVTUwUWpGUlowRkJZMG93TlRCSVZrTkJRVUpaUkVRelVXUlZTVUZCUlVJM1VVNUNNVkZuUVVGTFQzQkVNRWhXUTBGQlFWRlhWV1pSWkZWSlFVRlFha2hUZEVJeFVXZEJRVFJFV2s4d1NGWkRRVUZFU1hCV1NGRmtWVWxCUVV4QlZWWmtRakZSWjBGQmJVbE9XVEJJVmtOQlFVTkJPR3gyVVdSVlNVRkJSMmhvV0RsQ01WRm5QVDBpTENKa2RIbHdaU0k2SW1ac2IyRjBOalFpTENKemFHRndaU0k2V3pFeU1sMTlMQ0o1SWpwN0lsOWZibVJoY25KaGVWOWZJam9pUVVGQlFWRk1VbUZOUlVGQlFVRkVaMDVXUVhkUlFVRkJRVTFCWVZCRVFrRkJRVUZCU1VSemRVMUZRVUZCUVVOQmQwTlJkMUZCUVVGQlMwRmtTR3BDUVVGQlFVRkpSRGhaVFVWQlFVRkJRV2M1ZUVsM1VVRkJRVUZMUWtWRWFrSkJRVUZCUVdkSldVcE5SVUZCUVVGQloyMVJVWGRSUVVGQlFVVkNhaTk1T1VGQlFVRkJRVU42TVV3d1FVRkJRVU5uUkdaTmRsRkJRVUZCUTBOWUwyazVRVUZCUVVGSlRtZE5UVVZCUVVGQlJFRm9hRFIzVVVGQlFVRkhSR3ROVkVKQlFVRkJRVmxLYUVwTlJVRkJRVUZDWnpjeVNYZFJRVUZCUVVGRGMyVkVRa0ZCUVVGQmIwRlhURTFGUVVGQlFVSm5ibHBuZDFGQlFVRkJUVVJwYjFSQ1FVRkJRVUZaU0dGMFRVVkJRVUZCUW1kc1lXdDNVVUZCUVVGTFFUVnVla0pCUVVGQlFXZERTMUpOUlVGQlFVRkVRV00wWTNkUlFVRkJRVWxDV0dkRVFrRkJRVUZCZDBzNU0wMUZRVUZCUVVGblNUSTBkMUZCUVVGQlQwRkhXbFJDUVVGQlFVRjNTVFZqVFVWQlFVRkJRMmR4VmxGM1VVRkJRVUZOUTJSVWFrSkJRVUZCUVdkRGFFMU5SVUZCUVVGQlFYZEZOSGRSUVVGQlFVdERPRlpVUWtGQlFVRkJiMDlPWjAxRlFVRkJRVVJCY0RJNGQxRkJRVUZCU1VKTVoycENRVUZCUVVGWlRGTllUVVZCUVVGQlJFRlBjWE4zVVVGQlFVRkhRa1YxYWtKQlFVRkJRV2RCZFM5TlJVRkJRVUZFUVVKeVZYZFJRVUZCUVVWRFNYQnFRa0ZCUVVGQloweFhXVTFGUVVGQlFVUm5jMWxqZDFGQlFVRkJSVU55V1hwQ1FVRkJRVUZ2UW5oWlRVVkJRVUZCUWtFM1ZYZDNVVUZCUVVGUFJISlFla0pCUVVGQlFYZExOSHBOUlVGQlFVRkVaMkpUYTNkUlFVRkJRVWxEVWtsVVFrRkJRVUZCZDBoUllVMUZRVUZCUVVSblJGSlZkMUZCUVVGQlMwSk9SV3BDUVVGQlFVRkJRM05TVFVWQlFVRkJRa0ZUYUVsM1VVRkJRVUZGUkZKR2FrSkJRVUZCUVRSTlkyWk5SVUZCUVVGQlozWlRjM2RSUVVGQlFVZEVPVTU2UWtGQlFVRkJRVVZDUTAxRlFVRkJRVUpuWVZWdmQxRkJRVUZCVDBKa1ZXcENRVUZCUVVGblRHUllUVVZCUVVGQlJFRTBNVlYzVVVGQlFVRlBSSFZXYWtKQlFVRkJRV2RGT1daTlJVRkJRVUZCUVUxSFFYZFJRVUZCUVVORVZWaEVRa0ZCUVVGQlFVbE9XVTFGUVVGQlFVUm5Oa1pKZDFGQlFVRkJUMFJIVkZSQ1FVRkJRVUYzUjA1S1RVVkJRVUZCUkdkWlZWVjNVVUZCUVVGTlFUUlJWRUpCUVVGQlFWRkdkemxOUlVGQlFVRkJRV1JFYjNkUlFVRkJRVWxETUU5RVFrRkJRVUZCVVVsek0wMUZRVUZCUVVOblpFUnJkMUZCUVVGQlQwSjFVVVJDUVVGQlFVRkpSR3hPVFVWQlFVRkJSRUY0Vm5OM1VVRkJRVUZMUVVWaFZFSkJRVUZCUVVGQ1NqSk5SVUZCUVVGQ1ozb3pPSGRSUVVGQlFVTkVVMmhVUWtGQlFVRkJTVTE1U0UxRlFVRkJRVUpuZFVnNGQxRkJRVUZCVDBGVFlWUkNRVUZCUVVGUlNuUllUVVZCUVVGQlFXY3ZSa0YzVVVGQlFVRkpSRFJTVkVKQlFVRkJRVWxIVVRWTlJVRkJRVUZFWjJKNWQzZFJRVUZCUVVWQlZFbHFRa0ZCUVVGQlowaGpZVTFGUVVGQlFVTkJiWGhWZDFGQlFVRkJRVUpDUldwQ1FVRkJRVUZSVDI5UVRVVkJRVUZCUkVFeGR6UjNVVUZCUVVGTlJHNUVha0pCUVVGQlFUUkVPRkJOUlVGQlFVRkJRVzlTU1hkUlFVRkJRVWxFWTBoRVFrRkJRVUZCVVV0amVrMUZRVUZCUVVSbmRGWlJkMUZCUVVGQlJVRTNaV3BDUVVGQlFVRlpRM0ZwVFVWQlFVRkJRVUZ5T0dOM1VVRkJRVUZEUWpVMlZFSkJRVUZCUVZsUVVVdE5WVUZCUVVGRVoycERaM2hSUVVGQlFVTkRSbEJFUmtGQlFVRkJTVWxWT0UxVlFVRkJRVUZuYUZSM2VGRkJQVDBpTENKa2RIbHdaU0k2SW1ac2IyRjBOalFpTENKemFHRndaU0k2V3pFeU1sMTlmWDBzSW1sa0lqb2lOVEZoTkRReE16QXRaVEkzTmkwME9USmlMV0ZpT0RNdE9XVmxZekE1TlRBeU9HWmtJaXdpZEhsd1pTSTZJa052YkhWdGJrUmhkR0ZUYjNWeVkyVWlmU3g3SW1GMGRISnBZblYwWlhNaU9uc2liR2x1WlY5aGJIQm9ZU0k2ZXlKMllXeDFaU0k2TUM0eGZTd2liR2x1WlY5allYQWlPaUp5YjNWdVpDSXNJbXhwYm1WZlkyOXNiM0lpT25zaWRtRnNkV1VpT2lJak1XWTNOMkkwSW4wc0lteHBibVZmYW05cGJpSTZJbkp2ZFc1a0lpd2liR2x1WlY5M2FXUjBhQ0k2ZXlKMllXeDFaU0k2Tlgwc0luZ2lPbnNpWm1sbGJHUWlPaUo0SW4wc0lua2lPbnNpWm1sbGJHUWlPaUo1SW4xOUxDSnBaQ0k2SWpJeFpUWXdObUkyTFRJMVlUUXROR0ptWWkxaVlUZGlMVFUxTVRrNU9XUTBNR0U0TkNJc0luUjVjR1VpT2lKTWFXNWxJbjBzZXlKaGRIUnlhV0oxZEdWeklqcDdJbU5oYkd4aVlXTnJJanB1ZFd4c0xDSndiRzkwSWpwN0ltbGtJam9pWWpVMllqVTBObUV0WkRSalpDMDBZakEwTFdGbVlqSXRaVEF3T1dJME0yUm1aV1JqSWl3aWMzVmlkSGx3WlNJNklrWnBaM1Z5WlNJc0luUjVjR1VpT2lKUWJHOTBJbjBzSW5KbGJtUmxjbVZ5Y3lJNlczc2lhV1FpT2lKaE1EaG1PV0l4WlMxak1EVmtMVFEwWW1JdFlqUTVOeTFoTnpkbVpXVTFOMkU1TkdVaUxDSjBlWEJsSWpvaVIyeDVjR2hTWlc1a1pYSmxjaUo5WFN3aWRHOXZiSFJwY0hNaU9sdGJJazVoYldVaUxDSlRSVU5QVDFKQlgwNURVMVZmUTA1QlVGTWlYU3hiSWtKcFlYTWlMQ0l4TGpFM0lsMHNXeUpUYTJsc2JDSXNJakF1T0RJaVhWMTlMQ0pwWkNJNkltRmxPR0k1WXpRd0xXTm1ZbVl0TkdNME1pMDRaRFEwTFRjM09EUTBZbVZoTTJZMFppSXNJblI1Y0dVaU9pSkliM1psY2xSdmIyd2lmU3g3SW1GMGRISnBZblYwWlhNaU9uc2lZMkZzYkdKaFkyc2lPbTUxYkd3c0luQnNiM1FpT25zaWFXUWlPaUppTlRaaU5UUTJZUzFrTkdOa0xUUmlNRFF0WVdaaU1pMWxNREE1WWpRelpHWmxaR01pTENKemRXSjBlWEJsSWpvaVJtbG5kWEpsSWl3aWRIbHdaU0k2SWxCc2IzUWlmU3dpY21WdVpHVnlaWEp6SWpwYmV5SnBaQ0k2SWprMFl6WmlaRGd5TFdZeU4yWXRORGMzTkMwNU5XRXdMVGhqTURkaVl6Qm1PV00xTlNJc0luUjVjR1VpT2lKSGJIbHdhRkpsYm1SbGNtVnlJbjFkTENKMGIyOXNkR2x3Y3lJNlcxc2lUbUZ0WlNJc0ltUm9kMTgxYTIwaVhTeGJJa0pwWVhNaUxDSXdMakkwSWwwc1d5SlRhMmxzYkNJc0lqQXVNaklpWFYxOUxDSnBaQ0k2SWpJd1pqazVOamcyTFRCaFlqY3RORFV4TVMwNE5qVmpMVGc0TURoaU5qWm1NV0ZrTlNJc0luUjVjR1VpT2lKSWIzWmxjbFJ2YjJ3aWZTeDdJbUYwZEhKcFluVjBaWE1pT25zaVkyRnNiR0poWTJzaU9tNTFiR3dzSW5Cc2IzUWlPbnNpYVdRaU9pSmlOVFppTlRRMllTMWtOR05rTFRSaU1EUXRZV1ppTWkxbE1EQTVZalF6WkdabFpHTWlMQ0p6ZFdKMGVYQmxJam9pUm1sbmRYSmxJaXdpZEhsd1pTSTZJbEJzYjNRaWZTd2ljbVZ1WkdWeVpYSnpJanBiZXlKcFpDSTZJalkwWm1KbVkyUTNMVGRsT0dZdE5HRXdZeTA1T1RkakxUY3daVE5tT1RSbE16Sm1ZaUlzSW5SNWNHVWlPaUpIYkhsd2FGSmxibVJsY21WeUluMWRMQ0owYjI5c2RHbHdjeUk2VzFzaVRtRnRaU0lzSWs1RlEwOUdVMTlHVmtOUFRWOVBRMFZCVGw5TlFWTlRRa0ZaWDBaUFVrVkRRVk5VSWwwc1d5SkNhV0Z6SWl3aUxURXVNemdpWFN4YklsTnJhV3hzSWl3aU1TNDNOQ0pkWFgwc0ltbGtJam9pWm1GbU0yVmpZMlF0TVRFeU1DMDBPRFEzTFRrd05qY3ROelJsTnpkbU9EQTNOVFUxSWl3aWRIbHdaU0k2SWtodmRtVnlWRzl2YkNKOUxIc2lZWFIwY21saWRYUmxjeUk2ZXlKallXeHNZbUZqYXlJNmJuVnNiQ3dpWTI5c2RXMXVYMjVoYldWeklqcGJJbmdpTENKNUlsMHNJbVJoZEdFaU9uc2llQ0k2ZXlKZlgyNWtZWEp5WVhsZlh5STZJa0ZCUkVFNVlpOVBaRlZKUVVGTGFHdDNPRFV4VVdkQlFXdE9VRWQ2YmxaRFFVRkRRVmQ0VEZCa1ZVbEJRVWRxUzBaak9URlJaMEZCVlVScldub3pWa05CUVVKQmQxZFVVR1JWU1VGQlEyZDNZVTA1TVZGblFVRkZTamx5ZWpOV1EwRkJRVUZLTjJaUVpGVkpRVUZQYVZaMWN6a3hVV2RCUVRCQlV5dDZNMVpEUVVGRVFXcEJibEZrVlVsQlFVdHFOMFJPUWpGUlowRkJhMGR2VVRCSVZrTkJRVU5CT0d4MlVXUlZTVUZCUjJob1dEbENNVkZuUVVGVlRrSnBNRWhXUTBGQlFrRlhTemRSWkZWSlFVRkRha2h6WkVJeFVXZEJRVVZFWVRFd1NGWkRRVUZCUVhablJGSmtWVWxCUVU5bmMwSk9SakZSWjBGQk1FcHpTREJZVmtOQlFVUkJTVEZRVW1SVlNVRkJTMmxUVm5SR01WRm5RVUZyUVVaaE1GaFdReUlzSW1SMGVYQmxJam9pWm14dllYUTJOQ0lzSW5Ob1lYQmxJanBiTWpkZGZTd2llU0k2ZXlKZlgyNWtZWEp5WVhsZlh5STZJa0ZCUVVGSlJsRmFUVVZCUVVGQlFXZE1lR04zVVVGQlFVRkRRVXRHVkVKQlFVRkJRVzlNYWt4TU1FRkJRVUZCWjFkMFRYWlJRVUZCUVUxRU56SnBPVUZCUVVGQmQwYzFRazFGUVVGQlFVTm5OMm93ZDFGQlFVRkJSMEoxVDJwQ1FVRkJRVUZCVGxoaFREQkJRVUZCUVVFMWRXOTJVVUZCUVVGUFJESXJhVGxCUVVGQlFVRkVZWFZOUlVGQlFVRkJRVzl6UlhkUlFVRkJRVUZCVHpGVVFrRkJRVUZCYjBaaFFVMXJRVUZCUVVGblZEUmplVkZCUVVGQlMwSklhbXBLUVVGQlFVRlJTMGx1VFRCQlFVRkJRVUZaVTJkNlVVRkJRVUZOUVdaTFZFNUJRVUZCUVRSSlVUVk5NRUZCUVVGQ1ozRkVZM3BSUVVGQlFVOUVURTVVVGtGQlFVRkJORTVuVFUwd1FVRkJRVVJuTWtGM2VsRkJRVUZCVDBSWlJFUk9RU0lzSW1SMGVYQmxJam9pWm14dllYUTJOQ0lzSW5Ob1lYQmxJanBiTWpkZGZYMTlMQ0pwWkNJNkltWTBNMkl6TlRFMkxXSXlNRFl0TkRKbU55MDRNamhsTFRBd01EUXpaV1k0WkdFd05TSXNJblI1Y0dVaU9pSkRiMngxYlc1RVlYUmhVMjkxY21ObEluMHNleUpoZEhSeWFXSjFkR1Z6SWpwN0lteGhZbVZzSWpwN0luWmhiSFZsSWpvaVUwVkRUMDlTUVY5T1ExTlZYME5PUVZCVEluMHNJbkpsYm1SbGNtVnljeUk2VzNzaWFXUWlPaUpoTURobU9XSXhaUzFqTURWa0xUUTBZbUl0WWpRNU55MWhOemRtWldVMU4yRTVOR1VpTENKMGVYQmxJam9pUjJ4NWNHaFNaVzVrWlhKbGNpSjlYWDBzSW1sa0lqb2lZVGN6WVdNeU5qQXRPVGM0WWkwMFpHRTFMVGt6WVRZdE1tSTBOall3TnpZMlltUXhJaXdpZEhsd1pTSTZJa3hsWjJWdVpFbDBaVzBpZlN4N0ltRjBkSEpwWW5WMFpYTWlPbnNpYkdsdVpWOWpZWEFpT2lKeWIzVnVaQ0lzSW14cGJtVmZZMjlzYjNJaU9uc2lkbUZzZFdVaU9pSWpabU00WkRVNUluMHNJbXhwYm1WZmFtOXBiaUk2SW5KdmRXNWtJaXdpYkdsdVpWOTNhV1IwYUNJNmV5SjJZV3gxWlNJNk5YMHNJbmdpT25zaVptbGxiR1FpT2lKNEluMHNJbmtpT25zaVptbGxiR1FpT2lKNUluMTlMQ0pwWkNJNklqVXdaVFk0TURjd0xUSmpOemt0TkRFME5pMWlZakl4TFRSbU0ySXdaR1U0TmpjeU1pSXNJblI1Y0dVaU9pSk1hVzVsSW4wc2V5SmhkSFJ5YVdKMWRHVnpJanA3SW5Cc2IzUWlPbnNpYVdRaU9pSmlOVFppTlRRMllTMWtOR05rTFRSaU1EUXRZV1ppTWkxbE1EQTVZalF6WkdabFpHTWlMQ0p6ZFdKMGVYQmxJam9pUm1sbmRYSmxJaXdpZEhsd1pTSTZJbEJzYjNRaWZTd2lkR2xqYTJWeUlqcDdJbWxrSWpvaU1qbGlOVGczTm1ZdFkyUm1OaTAwTmpFeExXSTNZVE10WkRWaE5UaGtaRGRsT1dVd0lpd2lkSGx3WlNJNklrUmhkR1YwYVcxbFZHbGphMlZ5SW4xOUxDSnBaQ0k2SWpKaU0yWXpaREkzTFRZelpUUXROR1ZrWWkwNE5HTmtMV0UzWm1Rek1qTXlZV0UyTlNJc0luUjVjR1VpT2lKSGNtbGtJbjBzZXlKaGRIUnlhV0oxZEdWeklqcDdJbXhwYm1WZllXeHdhR0VpT25zaWRtRnNkV1VpT2pBdU1YMHNJbXhwYm1WZlkyRndJam9pY205MWJtUWlMQ0pzYVc1bFgyTnZiRzl5SWpwN0luWmhiSFZsSWpvaUl6Rm1OemRpTkNKOUxDSnNhVzVsWDJwdmFXNGlPaUp5YjNWdVpDSXNJbXhwYm1WZmQybGtkR2dpT25zaWRtRnNkV1VpT2pWOUxDSjRJanA3SW1acFpXeGtJam9pZUNKOUxDSjVJanA3SW1acFpXeGtJam9pZVNKOWZTd2lhV1FpT2lKbU16WXhaV1F4TlMxak1qTXlMVFF5WVRrdE9HVmhPQzFpWlRBM016bGtZbUZpWWpRaUxDSjBlWEJsSWpvaVRHbHVaU0o5TEhzaVlYUjBjbWxpZFhSbGN5STZleUptYjNKdFlYUjBaWElpT25zaWFXUWlPaUkxWTJSbVpEWTFOaTB3T0RjMExUUTVOakl0T1dZeU9DMHdaR1poTmpjMk5qVXdNemNpTENKMGVYQmxJam9pUW1GemFXTlVhV05yUm05eWJXRjBkR1Z5SW4wc0luQnNiM1FpT25zaWFXUWlPaUppTlRaaU5UUTJZUzFrTkdOa0xUUmlNRFF0WVdaaU1pMWxNREE1WWpRelpHWmxaR01pTENKemRXSjBlWEJsSWpvaVJtbG5kWEpsSWl3aWRIbHdaU0k2SWxCc2IzUWlmU3dpZEdsamEyVnlJanA3SW1sa0lqb2lPV1ExWkRCa1pUWXRZekZsTXkwME16a3hMV0V5WWpBdE1XUTVNMkU1WlRjMk5HTTBJaXdpZEhsd1pTSTZJa0poYzJsalZHbGphMlZ5SW4xOUxDSnBaQ0k2SW1JeFlUY3lZbU13TFdObE1qVXRORFJsTkMxaU0yRXhMVE5qWVRFMVl6ZzRaVGd4TlNJc0luUjVjR1VpT2lKTWFXNWxZWEpCZUdsekluMHNleUpoZEhSeWFXSjFkR1Z6SWpwN0ltUmhkR0ZmYzI5MWNtTmxJanA3SW1sa0lqb2lZamc1WkRrMFpXWXRPVGxsWkMwME5UVXhMV0V3TjJJdE5URmpOR0prWWpnd1lqa3pJaXdpZEhsd1pTSTZJa052YkhWdGJrUmhkR0ZUYjNWeVkyVWlmU3dpWjJ4NWNHZ2lPbnNpYVdRaU9pSTFNR1UyT0RBM01DMHlZemM1TFRReE5EWXRZbUl5TVMwMFpqTmlNR1JsT0RZM01qSWlMQ0owZVhCbElqb2lUR2x1WlNKOUxDSm9iM1psY2w5bmJIbHdhQ0k2Ym5Wc2JDd2liWFYwWldSZloyeDVjR2dpT201MWJHd3NJbTV2Ym5ObGJHVmpkR2x2Ymw5bmJIbHdhQ0k2ZXlKcFpDSTZJbVl6TmpGbFpERTFMV015TXpJdE5ESmhPUzA0WldFNExXSmxNRGN6T1dSaVlXSmlOQ0lzSW5SNWNHVWlPaUpNYVc1bEluMHNJbk5sYkdWamRHbHZibDluYkhsd2FDSTZiblZzYkgwc0ltbGtJam9pWVRBNFpqbGlNV1V0WXpBMVpDMDBOR0ppTFdJME9UY3RZVGMzWm1WbE5UZGhPVFJsSWl3aWRIbHdaU0k2SWtkc2VYQm9VbVZ1WkdWeVpYSWlmU3g3SW1GMGRISnBZblYwWlhNaU9uc2lZMkZzYkdKaFkyc2lPbTUxYkd3c0ltTnZiSFZ0Ymw5dVlXMWxjeUk2V3lKNElpd2llU0pkTENKa1lYUmhJanA3SW5naU9uc2lYMTl1WkdGeWNtRjVYMThpT2lKQlFVUkJPV0l2VDJSVlNVRkJTMmhyZHpnMU1WRm5RVUZyVGxCSGVtNVdRMEZCUWpSUmMzSlBaRlZKUVVGSFEzaDZZelV4VVdkQlFWTkRSRko2YmxaRFFVRkJkMm81VkU5a1ZVbEJRVUpxS3pFNE5URlJaMEZCUVVjellucHVWa05CUVVSdk1qazNUMlJWU1VGQlRrSkxOSE0xTVZGblFVRjFURzVzZW01V1EwRkJRMmRMVDI1UFpGVkpRVUZKYVZnM1RUVXhVV2RCUVdOQlluZDZibFpEUVVGQ1dXUm1VRTlrVlVsQlFVVkVhemx6TlRGUlowRkJTMFpRTm5wdVZrTkJRVUZSZDNZelQyUlZTVUZCVUdkM1FXTTVNVkZuUVVFMFNqaEZlak5XUTBGQlJFbEVaMnBRWkZWSlFVRk1RamxET0RreFVXZEJRVzFQZDA5Nk0xWkRRVUZEUVZkNFRGQmtWVWxCUVVkcVMwWmpPVEZSWjBGQlZVUnJXbm96VmtOQlFVRTBjVUo2VUdSVlNVRkJRMEZZU1UwNU1WRm5RVUZEU1ZscWVqTldRMEZCUkhjNVEySlFaRlZKUVVGT2FHcExjemt4VVdkQlFYZE9TWFI2TTFaRFFVRkRiMUZVU0ZCa1ZVbEJRVXBEZDA1Tk9URlJaMEZCWlVJNE5Ib3pWa05CUVVKbmFtcDJVR1JWU1VGQlJXbzVVSE01TVZGblFVRk5SM2hEZWpOV1EwRkJRVmt5TUZoUVpGVkpRVUZCUWt0VFl6a3hVV2RCUVRaTWFFMTZNMVpEUVVGRVVVb3hSRkJrVlVsQlFVeHBWMVU0T1RGUlowRkJiMEZXV0hvelZrTkJRVU5KWkVaeVVHUlZTVUZCU0VScVdHTTVNVkZuUVVGWFJrcG9lak5XUTBGQlFrRjNWMVJRWkZWSlFVRkRaM2RoVFRreFVXZEJRVVZLT1hKNk0xWkRRVUZFTkVSWEwxQmtWVWxCUVU5Q09HTnpPVEZSWjBGQmVVOTBNWG96VmtOQlFVTjNWMjV1VUdSVlNVRkJTbXBLWmswNU1WRm5RVUZuUkdsQmVqTldRMEZCUW05d05GQlFaRlZKUVVGR1FWZG9PRGt4VVdkQlFVOUpWMHQ2TTFaRFFVRkJaemxKTTFCa1ZVbEJRVUZvYW10ak9URlJaMEZCT0U1SFZYb3pWa05CUVVSWlVVcHFVR1JWU1VGQlRVTjJiVGc1TVZGblFVRnhRalptZWpOV1EwRkJRMUZxWVV4UVpGVkpRVUZJYWpod1l6a3hVV2RCUVZsSGRYQjZNMVpEUVVGQ1NUSnhlbEJrVlVsQlFVUkNTbk5OT1RGUlowRkJSMHhwZW5velZrTkJRVUZCU2pkbVVHUlZTVUZCVDJsV2RYTTVNVkZuUVVFd1FWTXJlak5XUTBGQlF6UmpPRWhRWkZWSlFVRkxSR2w0VFRreFVXZEJRV2xHU0VsNk0xWkRRVUZDZDNkTmRsQmtWVWxCUVVabmRubzRPVEZSWjBGQlVVbzNVM296VmtOQlFVRnZSR1JpVUdSVlNVRkJRa0k0TW1NNU1WRm5RVUVyVDNKamVqTldRMEZCUkdkWFpVUlFaRlZKUVVGTmFrazBPRGt4VVdkQlFYTkVabTU2TTFaRFFVRkRXWEIxY2xCa1ZVbEJRVWxCVmpkek9URlJaMEZCWVVsVWVIb3pWa05CUVVKUk9DOVVVR1JWU1VGQlJHaHBLMDA1TVZGblFVRkpUa2czZWpOV1EwRkJRVWxSVUM5UVpGVkpRVUZRUTNWQmRFSXhVV2RCUVRKQ01FY3dTRlpEUVVGRVFXcEJibEZrVlVsQlFVdHFOMFJPUWpGUlowRkJhMGR2VVRCSVZrTkJRVUkwTWxKUVVXUlZTVUZCUjBKSlJqbENNVkZuUVVGVFRHTmhNRWhXUTBGQlFYZEthRGRSWkZWSlFVRkNhVlpKWkVJeFVXZEJRVUZCVVd3d1NGWkRRVUZFYjJOcGFsRmtWVWxCUVU1RWFFczVRakZSWjBGQmRVWkJkakJJVmtOQlFVTm5kbnBNVVdSVlNVRkJTV2QxVG5SQ01WRm5RVUZqU2pBMU1FaFdRMEZCUWxsRVJETlJaRlZKUVVGRlFqZFJUa0l4VVdkQlFVdFBjRVF3U0ZaRFFVRkJVVmRWWmxGa1ZVbEJRVkJxU0ZOMFFqRlJaMEZCTkVSYVR6QklWa05CUVVSSmNGWklVV1JWU1VGQlRFRlZWbVJDTVZGblFVRnRTVTVaTUVoV1EwRkJRMEU0YkhaUlpGVkpRVUZIYUdoWU9VSXhVV2RCUVZWT1Fta3dTRlpEUVVGQk5GQXlZbEZrVlVsQlFVTkRkV0ZrUWpGUlowRkJRMEl4ZERCSVZrTkJRVVIzYVRORVVXUlZTVUZCVG1vMll6bENNVkZuUVVGM1Iyd3pNRWhXUTBGQlEyOHlTSEpSWkZWSlFVRktRa2htZEVJeFVXZEJRV1ZNWVVJd1NGWkRRVUZDWjBwWldGRmtWVWxCUVVWcFZXbE9RakZSWjBGQlRVRlBUVEJJVmtOQlFVRlpZMjh2VVdSVlNVRkJRVVJvYTNSQ01WRm5RVUUyUlN0WE1FaFdRMEZCUkZGMmNHNVJaRlZKUVVGTVozUnVaRUl4VVdkQlFXOUtlV2N3U0ZaRFFVRkRTVU0yVkZGa1ZVbEJRVWhDTm5BNVFqRlJaMEZCVjA5dGNUQklWa05CUVVKQlYwczNVV1JWU1VGQlEycEljMlJDTVZGblFVRkZSR0V4TUVoV1EwRkJSRFJ3VEdwUlpGVkpRVUZQUVZSMlRrSXhVV2RCUVhsSlN5OHdTRlpEUVVGRGR6aGpURkZrVlVsQlFVcG9aM2gwUWpGUlowRkJaMDB2U2pCSVZrTkJRVUp2VUhNelVXUlZTVUZCUmtOME1FNUNNVkZuUVVGUFFucFZNRWhXUTBGQlFXZHBPV1pSWkZWSlFVRkJhall5ZEVJeFVXZEJRVGhIYW1Vd1NGWkRRVUZFV1RFclNGRmtWVWxCUVUxQ1J6VmtRakZSWjBGQmNVeFliekJJVmtOQlFVTlJTazk2VVdSVlNVRkJTR2xVTnpsQ01WRm5RVUZaUVV4Nk1FaFdRMEZCUWtsalptSlJaRlZKUVVGRVJHY3JaRUl4VVdkQlFVZEZMemt3U0ZaRElpd2laSFI1Y0dVaU9pSm1iRzloZERZMElpd2ljMmhoY0dVaU9sc3hOamhkZlN3aWVTSTZleUpmWDI1a1lYSnlZWGxmWHlJNklrRkJRVUZKU2pKUVRVVkJRVUZCUkdkU1NGRjNVVUZCUVVGTFJITlhSRUpCUVVGQlFWbEtVVGxOUlVGQlFVRkNaM1ZUZDNkUlFVRkJRVVZFWlVkNlFrRkJRVUZCVVVGTlRFMUZRVUZCUVVGblNIWkpkbEZCUVVGQlQwRXhlbWs1UVVGQlFVRm5SVEp4VERCQlFVRkJRMmM0Y1hOMlVVRkJRVUZOUTFoeVV6bEJRVUZCUVRSRWVYWk1NRUZCUVVGQloyOW5iM2RSUVVGQlFVMURiRkJVUWtGQlFVRkJXVXRzZDAxRlFVRkJRVU5CSzJKSmQxRkJRVUZCUzBKS09WUkNRVUZCUVVGM1Ntc3pUVlZCUVVGQlFXYzRiRWw0VVVGQlFVRkxRa3RpYWtaQlFVRkJRVUZMVDBwTlZVRkJRVUZFWjJFMVozaFJRVUZCUVV0Qk1IQjZSa0ZCUVVGQloxQXlNVTFWUVVGQlFVTkJObTkzZUZGQlFVRkJTVVJZV1hwR1FVRkJRVUZuVFZFMlRWVkJRVUZCUkVGVmVEaDRVVUZCUVVGQlJHcEJla1pCUVVGQlFWRklURzlOUlVGQlFVRkVRVmhPWTNkUlFVRkJRVWRDU0hocVFrRkJRVUZCTkVSSE1VMUZRVUZCUVVKQmNuSlJkMUZCUVVGQlMwRnhkRVJDUVVGQlFVRkJTMlY2VFVWQlFVRkJSRUZLVG1kM1VVRkJRVUZIUTJrdlJFSkJRVUZCUVVsRFFXaE5WVUZCUVVGQ1FXNHlXWGhSUVVGQlFVbEJaWEpFUmtGQlFVRkJiMG96ZUUxVlFVRkJRVUZuVkU0MGVGRkJRVUZCVFVRMmVXcEdRVUZCUVVGUlMyMHpUVlZCUVVGQlJFRTViM040VVVGQlFVRkhRa1ZaUkVaQlFVRkJRVFJLUlRCTlZVRkJRVUZDWjJSNVVYaFJRVUZCUVVGQ1pFWkVSa0ZCUVVGQlowVkpSVTFWUVVGQlFVTm5SWFpWZDFGQlFVRkJTMFJwTlZSQ1FVRkJRVUYzVEV4WFRVVkJRVUZCUVdjelRWbDNVVUZCUVVGSlFVWjBla0pCUVVGQlFUUkRObTVOUlVGQlFVRkJRVEExTUhkUlFVRkJRVUZDTTJ4RVFrRkJRVUZCU1VKMVRFMUZRVUZCUVVKQk1uSkJkMUZCUVVGQlJVTmFNV3BDUVVGQlFVRlpSbW80VFVWQlFVRkJSRUZhYVhONFVVRkJRVUZGUWpGWGFrWkJRVUZCUVc5SlQwcE5WVUZCUVVGQloyWklUWGhSUVVGQlFVdENNRmhVUmtGQlFVRkJTVWN4U0UxVlFVRkJRVUpuUjBSM2VGRkJRVUZCVFVSRVRVUkdRVUZCUVVGQlJ6aHNUVlZCUVVGQlFVRnFRMFY0VVVGQlFVRlBRMjlJVkVaQlFVRkJRVFJOVlZwTlZVRkJRVUZEUVd4Q1dYaFJRVUZCUVVOQ2FrVjZSa0ZCUVVGQmQwUkZVVTFWUVVGQlFVTkJZV2RaZUZGQlFVRkJRME5xTDBSQ1FVRkJRVUUwVG5aNVRVVkJRVUZCUW1jclpsbDNVVUZCUVVGUFFWY3Jla0pCUVVGQlFWbEVWQzlOUlVGQlFVRkVRVWN3WjNoUlFVRkJRVVZCUkd0VVJrRkJRVUZCYjA5eVdrMVZRVUZCUVVSbmVWQkJlRkZCUVVGQlFVTnVRbnBLUVVGQlFVRlJTVlZsVFd0QlFVRkJRVUV6VURSNFVVRkJRVUZOUVhremVrWkJRVUZCUVdkSmJTOU5WVUZCUVVGQlFVVk1aM2hSUVVGQlFVZERWM05FUmtGQlFVRkJORUo1Y0UxVlFVRkJRVU5CVmtwemVGRkJRVUZCUVVOTmFsUkdRVUZCUVVGdlRVNHZUVlZCUVVGQlFXYzBiWGQ0VVVGQlFVRkxRVUZYYWtaQlFVRkJRVWxDT1VoTlZVRkJRVUZFUVdoNmEzaFJRVUZCUVVsRWQwdDZSa0ZCUVVGQlNVWnJaVTFWUVVGQlFVTm5WSGhSZUZGQlFVRkJSVUpIUTJwR1FVRkJRVUYzUkhkQlRWVkJRVUZCUVVGSVJGVjRVVUZCUVVGSFJEZGhWRVpCUVVGQlFXOU9jV1ZOVlVGQlFVRkJaemhRTkhoUlFVRkJRVWxCUmxoNlNrRkJRVUZCUVVKMUwwMXJRVUZCUVVOQlZGQjNlVkZCUVVGQlFVSXJUMVJPUVVGQlFVRm5Temt5VFRCQlFVRkJSR2RLZWsxNlVVRkJRVUZIUTJjM2VrcEJRVUZCUVhkQ2FYTk5hMEZCUVVGQ1FVRllTWGxSUVVGQlFVOUVjRTU2U2tGQlFVRkJXVTVNT1UxVlFVRkJRVVJuVWs5emVGRkJRVUZCUjBNek1rUkdRVUZCUVVFMFEyNUhUVlZCUVVGQlEyZHBOM040VVVGQlFVRkpSSFJ6UkVaQlFVRkJRVkZGSzIxTlZVRkJRVUZEWjJNMlFYaFJRVUZCUVVORFdXMXFSa0ZCUVVGQloweDVWVTFWUVVGQlFVUm5lRGhGZUZGQlFVRkJRMFJVTjJwR1FVRkJRVUZuVGpSaVRXdEJRVUZCUkdkUlNVVjVVVUZCUVVGSFEybzFha3BCUVVGQlFYZEJWazFOTUVGQlFVRkRRVzB5TkhwUlFVRkJRVVZCZUd0VVRrRkJRVUZCUVUxbGVrMHdRVUZCUVVGbk5USjNlbEZCUVVGQlEwRklTbXBPUVVGQlFVRlJRMlptVFd0QlFVRkJRMmRGVERCNVVVRkJRVUZEUkRadGFrcEJRVUZCUVdkUFRqUk5hMEZCUVVGQ1oxQnRkM2xSUVVGQlFVZERXbGg2U2tGQlFVRkJVVkJTVTAxclFVRkJRVUpCU0RGemVWRkJRVUZCUlVKTFdYcEtRVUZCUVVGUlNGWnlUV3RCUVVGQlEwRXdXRzk1VVVGQlFVRlBRWFJwYWtwQlFVRkJRVWxKY1ZwTmEwRkJRVUZDWnpGak5IbFJRVUZCUVUxQlowSkVUa0ZCUVVGQlFVZDNOVTB3UVVGQlFVRkJiMWxGZWxGQlFVRkJRMFJYZVZST1FVRkJRVUZKUVhOVFRrVkJRVUZCUTBGdFJFMHdVVUZCUVVGQlFXMVdWRkpCUVVGQlFWbE1UakpPUlVGQlFVRkNaM016V1RCUlFVRkJRVWREZW1ScVVrRWlMQ0prZEhsd1pTSTZJbVpzYjJGME5qUWlMQ0p6YUdGd1pTSTZXekUyT0YxOWZYMHNJbWxrSWpvaVlqZzVaRGswWldZdE9UbGxaQzAwTlRVeExXRXdOMkl0TlRGak5HSmtZamd3WWpreklpd2lkSGx3WlNJNklrTnZiSFZ0YmtSaGRHRlRiM1Z5WTJVaWZTeDdJbUYwZEhKcFluVjBaWE1pT25zaWJHbHVaVjlqWVhBaU9pSnliM1Z1WkNJc0lteHBibVZmWTI5c2IzSWlPbnNpZG1Gc2RXVWlPaUlqT1Rsa05UazBJbjBzSW14cGJtVmZhbTlwYmlJNkluSnZkVzVrSWl3aWJHbHVaVjkzYVdSMGFDSTZleUoyWVd4MVpTSTZOWDBzSW5naU9uc2labWxsYkdRaU9pSjRJbjBzSW5raU9uc2labWxsYkdRaU9pSjVJbjE5TENKcFpDSTZJalF3WWpJeE5HUTVMV05sWVRjdE5HTTBOeTFpTkRReUxXTmtORGN5TnpreE56UTFZaUlzSW5SNWNHVWlPaUpNYVc1bEluMHNleUpoZEhSeWFXSjFkR1Z6SWpwN0ltTmhiR3hpWVdOcklqcHVkV3hzTENKd2JHOTBJanA3SW1sa0lqb2lZalUyWWpVME5tRXRaRFJqWkMwMFlqQTBMV0ZtWWpJdFpUQXdPV0kwTTJSbVpXUmpJaXdpYzNWaWRIbHdaU0k2SWtacFozVnlaU0lzSW5SNWNHVWlPaUpRYkc5MEluMHNJbkpsYm1SbGNtVnljeUk2VzNzaWFXUWlPaUpsT1RCak5tVXdOaTAwT1RJMExUUTBNVEV0WVRNMk5DMDFOREZpT1RabE1EVmhNR01pTENKMGVYQmxJam9pUjJ4NWNHaFNaVzVrWlhKbGNpSjlYU3dpZEc5dmJIUnBjSE1pT2x0YklrNWhiV1VpTENKUFFsTmZSRUZVUVNKZExGc2lRbWxoY3lJc0lrNUJJbDBzV3lKVGEybHNiQ0lzSWs1QklsMWRmU3dpYVdRaU9pSmpNV0l5T0RNeVlpMDNNRFF3TFRRNU1HTXRZamhtTWkwNU5EQmhORFkxTnpCaU1qWWlMQ0owZVhCbElqb2lTRzkyWlhKVWIyOXNJbjFkTENKeWIyOTBYMmxrY3lJNld5SmlOVFppTlRRMllTMWtOR05rTFRSaU1EUXRZV1ppTWkxbE1EQTVZalF6WkdabFpHTWlYWDBzSW5ScGRHeGxJam9pUW05clpXZ2dRWEJ3YkdsallYUnBiMjRpTENKMlpYSnphVzl1SWpvaU1DNHhNaTQySW4xOU93b2dJQ0FnSUNBZ0lDQWdJQ0FnSUhaaGNpQnlaVzVrWlhKZmFYUmxiWE1nUFNCYmV5SmtiMk5wWkNJNkltUmhNVFV6WlRWbExURTFPVFl0TkdRM1pDMDVOMk0wTFRKaE1qSTFZVGxrT0RjNVpTSXNJbVZzWlcxbGJuUnBaQ0k2SWpFNE9HSXdPREU0TFRFNVlXSXRORE5qT0MxaFpUSmtMV0U1WVRoallURTRZelF6TXlJc0ltMXZaR1ZzYVdRaU9pSmlOVFppTlRRMllTMWtOR05rTFRSaU1EUXRZV1ppTWkxbE1EQTVZalF6WkdabFpHTWlmVjA3Q2lBZ0lDQWdJQ0FnSUNBZ0lDQWdDaUFnSUNBZ0lDQWdJQ0FnSUNBZ1FtOXJaV2d1WlcxaVpXUXVaVzFpWldSZmFYUmxiWE1vWkc5amMxOXFjMjl1TENCeVpXNWtaWEpmYVhSbGJYTXBPd29nSUNBZ0lDQWdJQ0FnSUNCOUtUc0tJQ0FnSUNBZ0lDQWdJSDA3Q2lBZ0lDQWdJQ0FnSUNCcFppQW9aRzlqZFcxbGJuUXVjbVZoWkhsVGRHRjBaU0FoUFNBaWJHOWhaR2x1WnlJcElHWnVLQ2s3Q2lBZ0lDQWdJQ0FnSUNCbGJITmxJR1J2WTNWdFpXNTBMbUZrWkVWMlpXNTBUR2x6ZEdWdVpYSW9Ja1JQVFVOdmJuUmxiblJNYjJGa1pXUWlMQ0JtYmlrN0NpQWdJQ0FnSUNBZ2ZTa29LVHNLSUNBZ0lDQWdJQ0FLSUNBZ0lDQWdJQ0E4TDNOamNtbHdkRDRLSUNBZ0lEd3ZZbTlrZVQ0S1BDOW9kRzFzUGc9PSIgd2lkdGg9Ijc5MCIgc3R5bGU9ImJvcmRlcjpub25lICFpbXBvcnRhbnQ7IiBoZWlnaHQ9IjMzMCI+PC9pZnJhbWU+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF84NDJiODhiZGNjOWM0MTI0YWE5NWVlYjc5OTE0MjUxZS5zZXRDb250ZW50KGlfZnJhbWVfMDkyODcxZDBmZjU2NDlkYTk0ZWFhY2U0NTVkMjg5NTQpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIG1hcmtlcl82NTU0Y2E5Zjg0NjI0OTZkYjhkMDIxNDM0NmJlZTFmZC5iaW5kUG9wdXAocG9wdXBfODQyYjg4YmRjYzljNDEyNGFhOTVlZWI3OTkxNDI1MWUpOwoKICAgICAgICAgICAgCiAgICAgICAgCjwvc2NyaXB0Pg==" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



Now we can navigate the map and click on the markers to explorer our findings.

The green markers locate the observations locations. They pop-up an interactive plot with the time-series and scores for the models (hover over the lines to se the scores). The blue markers indicate the nearest model grid point found for the comparison.
<br>
Right click and choose Save link as... to
[download](https://raw.githubusercontent.com/ioos/notebooks_demos/master/notebooks/2016-12-22-boston_light_swim.ipynb)
this notebook, or click [here](https://beta.mybinder.org/v2/gh/ioos/notebooks_demos/master?filepath=notebooks/2016-12-22-boston_light_swim.ipynb) to run a live instance of this notebook.