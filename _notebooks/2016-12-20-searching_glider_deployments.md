---
layout: notebook
title: ""
---
# Accessing glider data via the Glider DAC API with Python

IOOS provides an [`API`](https://en.wikipedia.org/wiki/Application_programming_interface) for getting information on all the glider deployments available in the [Glider DAC](https://gliders.ioos.us/index.html).

The raw JSON can be accessed at [https://data.ioos.us/gliders/providers/api/deployment](https://data.ioos.us/gliders/providers/api/deployment) and it is quite simple to parse it with Python.

First, lets check how many glider deployments exist in the Glider DAC.

<div class="prompt input_prompt">
In&nbsp;[1]:
</div>

```python
import requests

url = 'http://data.ioos.us/gliders/providers/api/deployment'

response = requests.get(url)

res = response.json()

print('Found {0} deployments!'.format(res['num_results']))
```
<div class="output_area"><div class="prompt"></div>
<pre>
    Found 192 deployments!

</pre>
</div>
And here is the JSON of the last deployment found in the list.

<div class="prompt input_prompt">
In&nbsp;[2]:
</div>

```python
deployments = res['results']

deployment = deployments[-1]

deployment
```




    {'archive_safe': True,
     'attribution': '',
     'checksum': '55e4ae8c590fd408c1246de3db32d93a',
     'completed': False,
     'created': 1482263525208,
     'dap': 'http://data.ioos.us/thredds/dodsC/deployments/nanoos-uw/SG187-20140625T1330/SG187-20140625T1330.nc3.nc',
     'deployment_date': 1403703000000,
     'deployment_dir': 'nanoos-uw/SG187-20140625T1330',
     'erddap': 'http://data.ioos.us/erddap/tabledap/SG187-20140625T1330.html',
     'estimated_deploy_date': None,
     'estimated_deploy_location': None,
     'glider_name': 'SG187',
     'id': '58598be598723c1805d6edd7',
     'iso': 'http://data.ioos.us/erddap/tabledap/SG187-20140625T1330.iso19115',
     'name': 'SG187-20140625T1330',
     'operator': 'nanoos-uw',
     'sos': 'http://data.ioos.us/thredds/sos/deployments/nanoos-uw/SG187-20140625T1330/SG187-20140625T1330.nc3.nc?service=SOS&request=GetCapabilities&AcceptVersions=1.0.0',
     'thredds': 'http://data.ioos.us/thredds/catalog/deployments/nanoos-uw/SG187-20140625T1330/catalog.html?dataset=deployments/nanoos-uw/SG187-20140625T1330/SG187-20140625T1330.nc3.nc',
     'updated': 1482278506388,
     'username': 'nanoos-uw',
     'wmo_id': None}



The `metadata` is very rich and informative. A quick way to get to the data is to read `dap` endpoint with `iris`.

<div class="prompt input_prompt">
In&nbsp;[3]:
</div>

```python
import iris


iris.FUTURE.netcdf_promote = True


# Get this specific glider because it looks cool ;-)
for deployment in deployments:
    if deployment['name'] == 'sp064-20161214T1913':
        url = deployment['dap']

cubes = iris.load_raw(url)

print(cubes)
```
<div class="output_area"><div class="prompt"></div>
<pre>
    0: northward_sea_water_velocity / (m s-1) (-- : 1; -- : 64)
    1: lat Variable Quality Flag / (1)     (-- : 1; -- : 64; -- : 216)
    2: latitude Variable Quality Flag / (1) (-- : 1; -- : 64; -- : 216)
    3: sea_water_density / (kg m-3)        (-- : 1; -- : 64; -- : 216)
    4: sea_water_electrical_conductivity / (S m-1) (-- : 1; -- : 64; -- : 216)
    5: longitude Variable Quality Flag / (1) (-- : 1; -- : 64; -- : 216)
    6: precise_lon Variable Quality Flag / (1) (-- : 1; -- : 64; -- : 216)
    7: Profile ID / (1)                    (-- : 1; -- : 64)
    8: longitude / (degrees)               (-- : 1; -- : 64; -- : 216)
    9: sea_water_pressure / (dbar)         (-- : 1; -- : 64; -- : 216)
    10: sea_water_temperature / (Celsius)   (-- : 1; -- : 64; -- : 216)
    11: WMO ID / (1)                        (-- : 1; -- : 64)
    12: latitude / (degrees)                (-- : 1; -- : 64; -- : 216)
    13: CTD Metadata / (1)                  (-- : 1; -- : 64; -- : 216)
    14: precise_time Variable Quality Flag / (1) (-- : 1; -- : 64; -- : 216)
    15: eastward_sea_water_velocity / (m s-1) (-- : 1; -- : 64)
    16: Platform Metadata / (1)             (-- : 1; -- : 64; -- : 216)
    17: Trajectory Name / (1)               (-- : 1; -- : 64)
    18: time / (seconds since 1970-01-01T00:00:00Z) (-- : 1; -- : 64; -- : 216)
    19: sea_water_practical_salinity / (1)  (-- : 1; -- : 64; -- : 216)
    20: lon_uv Variable Quality Flag / (1)  (-- : 1; -- : 64; -- : 216)
    21: time_uv Variable Quality Flag / (1) (-- : 1; -- : 64; -- : 216)
    22: u Variable Quality Flag / (1)       (-- : 1; -- : 64; -- : 216)
    23: v Variable Quality Flag / (1)       (-- : 1; -- : 64; -- : 216)
    24: lat_uv Variable Quality Flag / (1)  (-- : 1; -- : 64; -- : 216)

</pre>
</div><div class="warning" style="border:thin solid red">
    /home/filipe/miniconda3/envs/IOOS/lib/python3.5/site-
packages/iris/fileformats/cf.py:280: UserWarning: Missing CF-netCDF ancillary
data variable 'profile_time_qc', referenced by netCDF variable 'time'
      warnings.warn(message % (name, nc_var_name))
    /home/filipe/miniconda3/envs/IOOS/lib/python3.5/site-
packages/iris/fileformats/cf.py:280: UserWarning: Missing CF-netCDF ancillary
data variable 'profile_lat_qc', referenced by netCDF variable 'latitude'
      warnings.warn(message % (name, nc_var_name))
    /home/filipe/miniconda3/envs/IOOS/lib/python3.5/site-
packages/iris/fileformats/cf.py:280: UserWarning: Missing CF-netCDF ancillary
data variable 'profile_lon_qc', referenced by netCDF variable 'longitude'
      warnings.warn(message % (name, nc_var_name))
    /home/filipe/miniconda3/envs/IOOS/lib/python3.5/site-
packages/iris/fileformats/cf.py:280: UserWarning: Missing CF-netCDF ancillary
data variable 'lat_qc', referenced by netCDF variable 'precise_lat'
      warnings.warn(message % (name, nc_var_name))
    /home/filipe/miniconda3/envs/IOOS/lib/python3.5/site-
packages/iris/fileformats/cf.py:280: UserWarning: Missing CF-netCDF ancillary
data variable 'lon_qc', referenced by netCDF variable 'precise_lon'
      warnings.warn(message % (name, nc_var_name))
    /home/filipe/miniconda3/envs/IOOS/lib/python3.5/site-
packages/iris/fileformats/cf.py:1025: UserWarning: Ignoring variable 'lon_uv_qc'
referenced by variable 'lon_uv': Dimensions ('trajectory', 'profile', 'obs') do
not span ('trajectory', 'profile')
      warnings.warn(msg)
    /home/filipe/miniconda3/envs/IOOS/lib/python3.5/site-
packages/iris/fileformats/cf.py:1025: UserWarning: Ignoring variable 'u_qc'
referenced by variable 'u': Dimensions ('trajectory', 'profile', 'obs') do not
span ('trajectory', 'profile')
      warnings.warn(msg)
    /home/filipe/miniconda3/envs/IOOS/lib/python3.5/site-
packages/iris/fileformats/cf.py:1025: UserWarning: Ignoring variable 'v_qc'
referenced by variable 'v': Dimensions ('trajectory', 'profile', 'obs') do not
span ('trajectory', 'profile')
      warnings.warn(msg)
    /home/filipe/miniconda3/envs/IOOS/lib/python3.5/site-
packages/iris/fileformats/cf.py:1025: UserWarning: Ignoring variable
'time_uv_qc' referenced by variable 'time_uv': Dimensions ('trajectory',
'profile', 'obs') do not span ('trajectory', 'profile')
      warnings.warn(msg)
    /home/filipe/miniconda3/envs/IOOS/lib/python3.5/site-
packages/iris/fileformats/cf.py:1025: UserWarning: Ignoring variable 'lat_uv_qc'
referenced by variable 'lat_uv': Dimensions ('trajectory', 'profile', 'obs') do
not span ('trajectory', 'profile')
      warnings.warn(msg)

</div>
In order to plot, for example sea water temperature data, one must clean the data first for missing values

<div class="prompt input_prompt">
In&nbsp;[4]:
</div>

```python
import numpy as np
import numpy.ma as ma
import seawater as sw
import matplotlib.pyplot as plt
from mpl_toolkits.axes_grid1.inset_locator import inset_axes


def distance(x, y, units='km'):
    if ma.isMaskedArray(x):
        x = x.filled(fill_value=np.NaN)
    if ma.isMaskedArray(y):
        y = y.filled(fill_value=np.NaN)
    dist, pha = sw.dist(x, y, units=units)
    return np.r_[0, np.cumsum(dist)]


def apply_range(cube_coord):
    if isinstance(cube_coord, iris.cube.Cube):
        data = cube_coord.data.squeeze()
    elif isinstance(cube_coord, (iris.coords.AuxCoord, iris.coords.Coord)):
        data = cube_coord.points.squeeze()

    actual_range = cube_coord.attributes.get('actual_range')
    if actual_range is not None:
        vmin, vmax = actual_range
        data = ma.masked_outside(data, vmin, vmax)
    return data


def plot_glider(cube, cmap=plt.cm.viridis,
                figsize=(9, 3.75), track_inset=False):

    data = apply_range(cube)
    x = apply_range(cube.coord(axis='X'))
    y = apply_range(cube.coord(axis='Y'))
    z = apply_range(cube.coord(axis='Z'))
    t = cube.coord(axis='T')
    t = t.units.num2date(t.points.squeeze())

    fig, ax = plt.subplots(figsize=figsize)
    dist = distance(x, y)
    z = ma.abs(z)
    dist, _ = np.broadcast_arrays(dist[..., np.newaxis],
                                  z.filled(fill_value=np.NaN))
    dist, z = map(ma.masked_invalid, (dist, z))
    cs = ax.pcolor(dist, z, data, cmap=cmap, snap=True)
    kw = dict(orientation='horizontal', extend='both', shrink=0.65)
    cbar = fig.colorbar(cs, **kw)

    if track_inset:
        axin = inset_axes(
            ax, width=2, height=2, loc=4,
            bbox_to_anchor=(1.15, 0.35),
            bbox_transform=ax.figure.transFigure
        )
        axin.plot(x, y, 'k.')
        start, end = (x[0], y[0]), (x[-1], y[-1])
        kw = dict(marker='o', linestyle='none')
        axin.plot(*start, color='g', **kw)
        axin.plot(*end, color='r', **kw)
        axin.axis('off')

    ax.invert_yaxis()
    ax.invert_xaxis()
    ax.set_xlabel('Distance (km)')
    ax.set_ylabel('Depth (m)')
    return fig, ax, cbar
```

The functions above apply the `actual_range` metadata to the data, mask the invalid/bad values, and prepare the parameters for plotting.

The figure below shows the temperature slice (left), and glider track (right) with start and end points marked with green and red respectively.

Note: This glider was deployed off the west of the U.S.

<div class="prompt input_prompt">
In&nbsp;[5]:
</div>

```python
%matplotlib inline

temp = cubes.extract_strict('sea_water_temperature')

fig, ax, cbar = plot_glider(temp, cmap=plt.cm.viridis,
                            figsize=(9, 4.25), track_inset=True)
```


![png](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAA/QAAAFbCAYAAAB78MVwAAAABHNCSVQICAgIfAhkiAAAAAlwSFlz
AAAPYQAAD2EBqD+naQAAIABJREFUeJzsvXuwJcld5/fNOufce/t293T3vB8azQOBGEmD0AgMGFhw
KLxCy0NgbLBhY3FgLzb7gB3vYmIdeGFhww5jgzZYlrAhloANwF68MtYCBgkwCwKkMZLQoEWMRiPN
9PQ8etQ9/byvc05Vpf/IyqxfZv7yde7t6Vd+Izr6nnpkZVVlZeYnf7/8pZBSoqqqqqqqqqqqqqqq
qqqq6vpSc7UzUFVVVVVVVVVVVVVVVVVVVa4K9FVVVVVVVVVVVVVVVVVV16Eq0FdVVVVVVVVVVVVV
VVVVXYeqQF9VVVVVVVVVVVVVVVVVdR2qAn1VVVVVVVVVVVVVVVVV1XWoCvRVVVVVVVVVVVVVVVVV
VdehKtBXVVVVVVVVVVVVVVVVVV2HqkBfVVVVVVVVVVVVVVVVVXUdqgJ9VVVVVVVVVVVVVVVVVdV1
qAr0VVVVVVVVVVVVVVVVVVXXoW4ooBdC/G0hxLNCiF0hxIeFEF96tfNUVVVVVVVVVVVVVVVVVXUl
dMMAvRDi2wH8BIAfBvA2AE8CeL8Q4varmrGqqqqqqqqqqqqqqqqqqisgIaW82nk4EAkhPgzgCSnl
9w+/BYBTAH5KSvnjVzVzVVVVVVVVVVVVVVVVVVUHrBvCQi+EmAF4O4Df09ukGqn4XQBfcbXyVVVV
VVVVVVVVVVVVVVV1pXRDAD2A2wFMALzibH8FwN2vfXaqqqqqqqqqqqqqqqqqqq6splc7A1dYAgA7
p0AIcRuAdwJ4DsDea5inqqqqqqqqqqqqqiulDQAPAni/lPLVq5yXqqqqK6wbBejPAugA3OVsvxO+
1V7rnQB++Upmqqqqqqqqqqqqquoq6TsB/MrVzkRVVdWV1Q0B9FLKpRDiowDeAeDfACYo3jsA/FTg
tOcA4Jd+6ZfwyCOPAAB+9pm/t3IevucN/zS47+c/+3fZ7d/98D+LpvnhF7/V+r3Tr+HZxZ0Qos86
X+t3Tn0nAGAmOm/f177u/8hKIyadzy+/771m2+OPP473vOc95toAMEGPXz/7xQCA97ztB5Pp/upz
37PvvGl924M/mzzm107+Fyul/S0P/Avz9/tOfre3v4cAALy8OG5t/1uf/xPBNH/9+f88uO9X/ofn
8B3/3YPe9m98/S+wx//hC9/O5mdXrnnHvuv+fxm8bkr/76n/zNvWiHjQzVj5+7cv/KfZ184tx3/0
4n8S3f9z/+Q0/uYP3Y1DYpl9bVdvv/f/zjru6Zff5W37gnt+K+vcT7z8DXj0nt8AADz58jcCADo5
zqCaDHUEAEyh/n7zPb+Jv3z5r3lpPXLP/wMA+Jaf/WX82vd8p7f/23/il/Gv/r6/Xetvf/R/AQD8
87f/g2S+v+d7vwI/8MPHAACff/dvJ4/P1fOvvBMAcLGfmW36+YTEPYuY9HOi+ouXv74oDVdvvuc3
VzqP5n0yOKH9zz96AT/wj45nl6FSffb0Ow8knc2mLBDvdKivqG6/8/3Rcy6c+bro/rYgGLAE8MM/
cgn/+EduiR53SEyy02zRs9uP3+F/E/3Zb46m1dwerm9S5+ZoIdvsYzfuiH9zu2f87+UHf/g8/qd/
fCKZ9qE77G/FTcvdz+nr3/eLAIDffPd3JY8tVXv23ebv6e3vO9Dj27PvZo/ZOvPXcKkf38/20AYc
Ej0+80yL/+b7LgJDX7eqqurG1g0B9IN+EsAvDmD//wF4HMAmgF8IHL8HAI888ggee+wxAMBt0/AK
dykweezRx7xt/+MnVafrvjcds7bPh07nY2/xzwGA33lWDTB8/okNAMDlXv1/rjuCW+e3Y6NZohE9
3o8fwj98k9/JdPUXR9X1N5oRUtYGuD8FG0Df/fDHk+m5evXEBvbkDI89PN7P9vRJfObY38TDxzYx
GTovHRpMn78PR2YL88xj+vDho8V5AYClVB0rOoDxYfx9AMDfeuPvB8/72JEjK13v4/g+8/cDb1Zp
9EwntN87YZWjX8OPAQB+7NFf84798OHbgtdbP/IS7njE3//YG8dn+uuf/SLz9+cdPzTkyQ6Zsd2v
e2k8he81f3/HG54I5oHq//rM2wAADx/b9PZNEt9NrPydOu6nl5sOAMyE3xF9w4lD0XQ2j07whrcc
wgz+YEeuHnsoXLaffP5+8/cjt4/XWECV2cden/4uAGB5ch2PPaCO3Tup3mMI6GdCf3//kXXNUd+M
x17/PDbu/QP2uzx0J79d65bLdwIA/uHlf4n3f014YBMAjtzS4E2P6jx8EwDg0ftfiJ6T0qdfuBdv
vlPVqee6sUwvoQYav+yBZ/kTny99xz4cfeHt/jdUotz3rfWx518PwC47H9z5Anzt5qdw9JYGjzy6
BuDdeOvrT+0rX1qfPHWf+ftNd6z+TVAdbXiYDWnG1KXAN5i/7r7vJQDAqy+9zmx74O54XpcyPw89
gFtuEXj00Vn0uCMivzt1rldt8dHGHQT4JvPXbfeq76I/vZFILTLweW/q3LTmMn9w89A9fnnefvmB
8cc9/ns5dkuDL/6inLL1Ldavvbsn2LAGUdT+w/ecDKaw9vHfBYCs/keJ2tNvAO5VdUGDBs3d8fTp
8QAwjRyvj+WOufTSGs716vuYS4HF0MavoUcrzXdTp5RWVd0EumGAXkr5q8Oa8z8K5Xr/cQDvlFKe
OYj0e8l1KnxpiPfPbwxohqRBXouCPABc7g7hhb3jeN3GBQP1+noxsF/K4TXTPgxhuzUCvu/77Beb
v3PgXudZw+L/+Zm3W/sn6NGRi827KY7MFnj3H/0dvO+rfjqZ/irSAyb6shTsf+ZT/4H5Owb3q8oF
+cYJ4aDLEQX7//4TqiNCwf5iFwbPpZyw+ynEj/lZPe7lrzzzZQDCYK9BPqQUzHOi5e9qihuQ2Y8o
yGt1EObbyK1fXP3JyYf3lS8tBYqPr3RuL4Upz+/8g7+XhHpXnzj1upWh/tMv3Js85omTD4Wh/jqS
hnlXf7l9DwDgbDf223V5WxXsKci7mpA6rTvg72QVnX5RlYGZSNd1kyG/q/vfHIyWQz4u96ptWhMC
607+9QDFsWaE8uYA4xj3AS+Bg5AF8RnqAh4TExEuX5f7zrTxFOz1tTmwF52AnEg88PM/jpPf/d8W
5ZFTe/oN1u+c9+OecxCaS4G9oX/ZCIkO4gq+3aqqqmtRNwzQA4CU8mcA/MzVuHYJyFNLGpAH8oAC
uSfP3AfcAQvq9fVDUM8OJGTAfa72pAJoCvMC0oN5AGi7CbaWa1cU6nUn0wV7gIf7/YJ9DOLps+/Q
GG+FXLBfPU8H1/Fzwf5KgPzVlPs9drKxLNyrioN44GBAHjg4mN+vnj93Aq+/9XzSiwlQ3wodAND6
xCkFLyVgnwPzWtcz1IdAXmu7Xcdfbt+DrfYp/NudN+JrNz9l9j35/P3ZUJ8L8debJqR+7la4jw4S
cvh/ckADGJd71fU62gxeRFJiITusDQBL4X4paZvcYTbA60HC/SriBgRKQV5rGcJPGb7HM/06gLn6
kQv2vQAmB1OWKZjTd9FHWt9SmNfHd7IPdtZ3ZIOlbNBBYCkn2EB73bXBVVVV+9cNBfTXivqhETpI
kAcUCGztrltQDwCbE9WohaA+5RkQgvuUXOu81gQ9JIEWqnNbm5geUx2UI7NF/sUKpEF+fZhiMCfz
amNW+7UV+2quFd6FeCr9OwX2vQy7SnaywVbr7z9IiOd0I4C8+w0Gj9vHswxZ42m6Lsjv19KZe18p
vfFH34NP/SPfUv9Fj78Hf/4e3oLfLqYW1Mes9Je6Q1hggrUBUjiwz4H6EpjXeuLkQwAiLvjXoFIw
DwAv7dyCezcvoZUTY60HYMA+Za0PgXwI4vdbVkNQHIJt3TyVlPAUxJcM1+30Ap1U/9P5//uB+wu9
9rDaBWCDvZ9D259gnIikvqGZmFxRuL+SlnytVzr+3d81CV97r1/D6H6ZCfbyYKz0MZjPOYcqBus5
0jC/J6eYQGJPTtGI5TXhPVNVVfXaqQL9AamXdEQ9DvKuckBeX6PvbKgHkHTBN3CSw1uFbbe2zgOw
5sp/5TfyQW4WpzfRHtkBgCtmpd/r7bmO6yR2QMwdfyFX+xwmzkOjMBgaTKHWesAH+9ggzBu/7vXp
QZrXUFcK5Lk5/oebefb5q0DuV3/D8ZXPZfOQCfIHdb2rpXYxxc5yDUfW1PsJQf0XvOv1eHLvCN66
oeByTXbF1vpVYJ5KW+vpd+p+w9eCcmAeAC7sqfbi2Ne8GdvtOj5y8UEcno7fSS7YA3kQv1+X+xC4
pwA5Ne6chvjV6qnn2uP4975B4Ln2GB6cXjDb3eB+c+R7t13u3SlTIbAHGqds7rCAP157XcTn+l9r
+o+/eROvBqeY7QbP25FrplDkgr1ovw9SYGUr/Sog755H1ck+WS67QLyHSy+p+mF76IPpfiQAbIgW
ch/eX1VVVdefKtATxaxyqQ4fB1ihTrru1JeAvE5LSoG+wwj1QNIF3ww20Pp9nwzmWuc7aVvkv+ob
b2XPm+4InNvaxK1Hdq6Y6/28t4v1Xj8zAQE9qz15RTv9akGfjk3sTsdSTqwBHq1WTjAlAwiutR4Y
y0bM2v7Gdz24Mn50B9jIX2mLfO77KIH8lL7mm46nD0qos8AiD+T3411xUIMBzRLoGRaQAoi96r4T
aCYSZy8fxuati+h8+rve8Sa8sryIDyyP4a7ZRbx14xQL9QBvrU/BfAeR5R7+xMmH0JBXci3BfS7I
a+0tZrgAYOMr34aXdlRddO/mJQvsY274KYifQB6oy33AGBssZGc6VSjvmCwNtHNWexfiXVBaSonT
nWobDjMBM0N6an4v7nnnvXjKqWYo3APAXiD/O73fN/jknirHb9pQAf0u94dwtNmFB/YAlk6yRxOA
HwN6Cp62K//V07d9y2H87i4fkPZcHw5Ue6Y9ivtnw/LqvZoueMYMAofBflUrfS7ML2VndaxTMB8a
DKDn7colQuE3l3KKPTnFQk5xoVPDOxuixeUK9FVVN5Uq0BOdWxwO7rt1bTt67oyMkKdA3lyvAOTN
PG0pIAED9QCyXfAtyDwAuKfW+Wz1AovTmzh3NzA91kXd7ledX6zPc8EeGOGec8d/dblaVP0NpnO4
x1j7FUyrd+yC/ZWEiBDE6+2lYP5audY/u3eHt+2hjfwYl6qDnBYH065VrEQUDq8kyO9XbrkLhQ1I
fYWya9Cj91zvAR/q//Clh/FX7v2s+f0kYKAeiK8mkoL5Xg7PdbiRVSHUHdh9LQG/FOYBoO8FdvbW
0DQSF5x9925eKnLDpwMiV8rlPvg0A6/rT3YU2Pz7m8+YbRruaVocwAMwEA8An1mqOuXOyeXs/P7F
Vji2ANXdgTRf7f3VOk7P7ZVv7ppeAqaj5f7+6Tmzz3sPjQ3iLuC7osD4uW4HF4ao6LdmLh94tIm3
8zSgXSyIXey8J3fKy/2Tl16Hd93+CQAYwH4ByDjYi07lr9RKr+E6ZZVfyg5L2eGQc54rCvOxFRe0
dT48HUXi0mAYutBtYme476fm9+D5xWUAB7PaRVVV1bWvCvRE5xf5y2S5umv9UnAfB6YdmiKQ127k
3bLBZNYbqAfAuuBTqNevmYLnfuCes87nqJcNhFRWeu16/+reJm7b2GGt9DuMy3WJ6HPnrPYAcGy6
Mx7ThT+HWCeWg/clM5hwYXkIx2caMtPW+pCWfRoCQ+9kSfL6mfldAIDPW39FXTsB6quAvLYYAMDx
yU7kSF9nFnkDLCHI54D+tQBo/nuPg3wJJOm54DS9VH40KFOIP93egrunl8zvVeMAykUDrCEI9VTL
doI/fOlh3H10hJ8PLI/hrx79dwB4F3wgD+YB4LLcwFG9SpPoo1DvvqfQYMJrYb1fBeS1lsthmc5Z
x4J9ylq/QR5Dqcv93gpTfzYLC9pzc39gT8P9HRNinSZZ5yAeAC4O9dGyYHrV0xf963OabfDwdGrp
LzH6/I7vwfZKe4sCe0e3NlvOFsdVwAH8250ySiEeELgwtKuvdnl14WPrtgeAG5GeBrSbIFweYued
XtySlReqc3uH8FtnHwWAbLBv5gLdpiyy0ufAvPZ2WMoOZ/slbkE+zD8xP4p3Bq4JKOt8CPqXQwi+
S90GLveHMO9neHlxDFvdOs7snmPPqaqqujFVgZ7o5CV+3jcAINHecEAfAnm9rwTkdwe3Q7QNOsBA
PYCgC76Geg2XS0wxG1z5suA+Itc6nwoi1ssGHRqIDpBCYLqDpOv9xTa+ZniJQnB/saUhhlazPHHw
znkHPHf5Vjx4VDWyCux5a71KM/w8OXd+IA3x7pz0k7t2pzIX7Ev0kcsjfH7JUTsYWQrw//jFh6zf
Jw7v4E0FHvF3HMm3wl0pHSTIe2kXuNrr8r8nZ9hslFfMBy48ir96/BO4T7sOD31GLzBeojiIpYCE
gnpMOjOffnO28ILk9X2DZQucOn8cpy8fNdb6D+AtQRf8XJjvIPDJvfvwpo0XAUCBfcRa/y/O/BXz
99ff+iTunly09nOAv59giSHtB+Y7NGaubC7Ya1Gw59MOu9w/3yoL8/FJnhcM1SeXx9jt909d/wKl
j5wdgkze7u+jVnsqDuKB0Rvu5NxP7JFDL7FpbS/GqT8xuA/VZ2daf2DywlzPd+anplH92WKMHH9i
uo3HNp5zjrAB/3PdmA8F8iPEA8Dvb70peU2qt679ufU7GJGeEYV497yL/djubbflg/fndsb3+ltn
H8Ud65eB4VFrsF8bPCfPkPunVnoBQCYs9blWeQA42y9xqj2K1xfA/OfacOdyIhp0UuJku467mf2X
+97A/NnlUTy7exu2lhvYatdwcffamFJRVVX12qgCPdGFbR8g12cKfKOwD+ALj542f6dAXsNeEchT
EagHkHTBv0Qiot82U1MHlsOrnzVtGO4ZhazzSznFLDAvUcN8JwVEB4gJIDNc769U4LcQ3K/q4s/B
+5yZjnB2a5wP+ODRc0FrvcpL+D3kQOBSTj2AP9fZU0rO7vHzE68E2AM23ANpwN/d8efQ//H2mMaJ
w+r4Esh/LWTNoS8A+RJALxkIeLE9bqD9D7a+EF9z5CkAwDNbtwN4FH/j9j/GBH3Y5V6qzm8o0v1k
t0GHHhIN+klv5tPffhRekLzlcoLZ8GksWwSt9doFvwTml7LB09tjt/dNGy9GrfXPbo0w9Zt4q/n7
6299EgCyAH+/2i/MA4AcAEUbR12wd/URPGj9/rrDnyRphl3uNcS//9IXmW3vPvbR4nx/es6hSVg7
S3UPGuzvO3KRPe6u2bidg3gAeG5PWcs/N19telUM7r/g8Gn3cADA+aU/lW85zKvPAXt3YPv3tt4M
QME9AA/wZ2IcZNEgTyH+qS31/NcneXEElifiAL9D5uLPnHqOQjwFeAB4sT1s5sg/v8VX4hvTcB77
vrGgHgB+a26D/RvXXgYAA/aiH630YikgZwAy59KnrPIAcKo9is8s7sRXbvhlIQTzbl+BLlWnrfPb
ko8nsycbPDu/EwDwx68+jFtmc7Sywe5yhnlbu/dVVTeT6hdP1M7DADlfjo9KQz5VCAY5kNdz9TVc
A2mQ13Are0A0MFAPIOmC/4efVQ3E+voS/+EDtlXmttl2EO5D4qzzSznxgF5DqYb5nX4dogdEB2CS
73pfqrML1Um4fc11VfRF39tlZim4HB2Z+oMRrzLTN9qusaAeoGBvW+tXsdxecIDdBfitzr6/U5fi
JHwQYP+nZ3xg+dI7ngeQBnzZ28+AA3xghHwN+FpfnzZ+BRVzzU/Nr6dgftAgH9NHdh/Glxz6rLf9
w1tvwJcfUZbMT2/fabZfmm/gGdyO08eV673oAoHxJKKeO6IVI9SvjfPpuSB53VzN557NOstaT/UB
vMW44G9GIoe7MP9cexte3LGtvzFr/dZiHPAKwT0QBnygDPK5II9Hm73s87Wol8DHdh9AP58C034s
lw7Y74D/bjTYU6DnPBmeb49ZEH9qZxzc/sSGv0RjSs/s3FV0/O7CLpAvbh3Di1vHPLDfIKuZcBAP
jIOvW0vfIvzMNm9931v6g7Mbs6UF9wAP7gDw3K7vcr+r0xz+i4H9dhe3Xv/e1psN3APAl5I6QIO8
hngAuLBQz2barDZ9ZMcJpvfB3bGOf+fmi9Y+CvEvtur50EB3nx1gNFQH7kWgtOsF9OT4XLAXvVra
Xrvb57je51rlAeAzizvxyd37gFtsoI/BPLeaC9XJdt1bFUFHuO8g8NL8GJ7dug2X5hvYnC5xYb6B
y4t1MxBWVVV1c6gCPVE/9x9HfixcWzGQ3+tmaGUzBroDD/LUQq23i05AQhqoV9eKu+BrC858PsNv
PP0WAAruAViAT+GeU8w67zbI2iqvj9vp1/Hs/I6hAVXHSCHQLEbXey0N9avOd/70+aFjRpwqcuCe
6+Tl6NXGh/etpT840Lbj+3TB/otv1R2htFeCiYTvPPMUwLuWnr25KlO5YL+Kzm35z+b9W18IANY7
/9I7nvcA3wV6VyHAX1V0yoK/rNSo1FQBrtxeSZDXenFxAsDDAIAvI67IL+4dx4ehBvUuLTcM1Gsg
0a73QGAevYxHuhc9gFZAtAJ9Yj69XArQGNDaWq9d8LW1/gNQ9dS3HfsYe00O5j+y/RAuL9c9qAd8
az0AXNxR38fGml3LU7gHeOu91r0BN3EO3nVwN6p3Hvl37PkhuTD/5Pb9apC3bdC3zUpgzykG8c9f
HuuLTx8qg3MAOLfkY9WEQH/R8nWiC/b3rZ03+ziIB4CPn1VLIgqmMC+YaPSAPZgfEwfuAPDStu9S
3Q7z13cHot/FDIdmSxbsN5kB45g4a7yGeADYG+ILTCPB2KhiAA8Ar5ApFBf75619MYgHgJcX6tzF
Ctbktp0AU50331oPEG8U7ZDRj1Z6zdElAfJiVnkA+OTufXhx165/UjD/4tz3/tTB8PZkh225acWi
odrpZ3jyVdXPm7dT7LQzSCmwWE6xDHw3VVVVN6Yq0BOJLb8C5Jq8GOTngDwAnN4+ijceHeGdA3m6
7YK2+PZCzfsa8J1a60Mu+L0OftMBzUQdMx9g7jeefgsL9yGFrPN0cIK62AMwMP+py3cpKBgeqpgA
TYvR9X7S4baNssBpbB6HDpgBeyAL7i/OV7PQrzW+FfHlHd+lc7k3BcglppPegP3HoRplDfacx4cG
+GUABFMAH1Iu2L9t82RWelSLywxEDJ2ncxg7KRryLTnPINX/LAX8WDDHVwLzfHN0vBnL8GsB8lqf
vnwHcNQGGwA4RdxZX7p8i+ncSqmgXrveNz3QM89YyHhojckc6NaBZikgp+N8eg31dD696AXkEg7U
h631nEIw//zurej6BpeHgbmotR5jHQgAe4shcOiaX7vHrPffefuH2Dxy8P7U1j3etlygpyC/10/x
yfl9eHL7fpzZOwqxbCCHun+/YP9z577a/B2C+EU3tpNPXSoH+lK1i3g35cUt9Z5vX+Mh/uXdY3hl
WxV6HZuEW6P7Evj6f7k7ptVMVeHjFsrkwB0AtuaqPNLntjCDBGN528XMAD5VS+OpZFRxZ+cjPGuQ
b2Vj0tHBX3fbPC+wGMADI5QDI8ADaYgHgAvD4E7MEh/SYnsNODwMdkw7hKz1wAj2uv8hm/IAeUmr
PIAXd49Z9W0OzD+7PcZzcIPhXZZqcPlDlz4P3+Xkp4dEB2Fc67teYNFNsOwnaLsGXVfu5VdVVXX9
qgI9keAqwEzILwF5ANhZzJLW+AvEbfvFy6oRnGw36A6P+Eyt9SEXfGyReeJHWp1hAArwObgHALxt
/DPHOs+52AMwMH9m9zCaher8a9d7HSCPc71/+MhqDZK2fuyR4p0D95dWBPqWAbRLu0xavVBQDwAb
ysIwnXYs2OvgeZz2uLgKAD6XGRXe5Hvw8JhOB2tAAuxLgX67X2et7HqLBfuMhUQOHU6hLSiFgO8q
tRoDhaZPXI4sVZV4zGZtZO/6cZDfkftb1eHlrQEojgKnZiPcbM3XcQrqne4s1hTUA5gvZlhfWxrX
e9EhHBgvJqmgXjbCmk+vg+Tp+fSbMxV9WkM9MIK9hnrAgfo77UvFYH6rXcfuUlk6AeDych1PXbwT
R2c2fmmw74fyP3cgKhfuAeBPNvngVxy8c/VEjlyrPAAD889dPAF0AkLPp18B7EshfpcMhJxnvJNS
4qzjMfVDu6zBvl1MMWXejQvxAPDK9lErwKgG6YZxNw/NOZbD+aLp0Q91Ut9ODdxraXA31+r8voO+
futYTxtnCTnaFnGDDwCCcE+t8RzIn99W7+zwep7lPwbwAHBmMYJ7CcQDwKtDH2meGLThJFuhoB5Q
YE+s9QAP9pM50G7YrvdA3EpfYpU/tXUcF3fU88+F+TO7tmddJ3sTDO9ct44L3SZ2O/9lL9Hj04u7
h6kHqpxIKbDsJlguJ14Zq6qqurFVgZ6omTNWUa6PzUA+hfMUyANA201YiAdGkNcQD4zuoc1SAAPU
A7Ct9QEXfEFG4uWW/cr7Iy0L95xi1vmFnPIu9oCB+UU7VSAvh2m5JEAe53rfBlwgU5rrQFAbqsMy
aaSx2gNhuO8SLt4hvfxqnjVXdsLArAv2G8NAigb7uzfTEdrd58MGUIyoW9rn54J9SnpO4DN7d1mD
ZDqacAzyLemOSiMBF+6BJODnLKdIYYkG0nrmIhNSO1NfsmnPY88B+VOLsgn/bpof2X0Yi25ioP5Q
M3bW267B1nwdR9bnaLsGO4Orfdc1mC9m0EtHiy7sch+NdD/s0673bpA8PZ/+9qNQ77RRlnoAQWs9
Z6lPwfyim1hupi7Yh/KtwR7w4R5QgM/BPcCDO8DDe2ggLiZdPqlVHoCB+UU3UXAyfEEs2A9phcC+
FOJbamlmoFWLC8gH+EXpyCE14HJhj/co0vUFLZrtYuqB/cvE1fkF0nYuSN2vB3tn3AoGgQFCuRw+
kNn4Tincm+sEngW9fucsFaehazrtsOinHtgDwCuXVN9hjYndY0QetS57IZAH1DQCblCDUwzgAdvw
UALxALDsbOWSAAAgAElEQVQ1zPNeKSCtFJDDI1lsr2E56THbaKNu+KIbrPQtgKkdIA8Nb6Uvscpf
3DmErhfmnByY1+WeWue3+jmWsser/WG8sLgVn2MC2F7u+zG2khRYtso6P28n6NvGDIRVVVXdHKpA
T8R1ZnMh/xxppFIgD6iGnXWpxwjyGuKB0TrRyBHqAUSt9doF37pHpxMiXes9E4Mqxzp/vj1s1v2m
LvYADMx3UgASaBZAvwY1n20IkMe53odcy1PSK+XMSYdyPQPu+8hScbt74c54v8zMZy/GQZbhfwP2
gzacNX+pXIB3oaF0AER24wCQq+m098E+YrQGbJAHhrWFCby7XxJdLoiF/MUAfaSzGoR7wAP8kEIQ
Ty1R3IoXWs9w62dRDV7IJSD/p5ceih6b0tml6vBpqL9jY/Q8UUDRGguihpp+CCqlXe+b4Xt0A+MJ
GY90r61d2vVez6cH4AXJU1Q2vKcm7oJPlQPzpy4dx87OOjY3bYu8BnsX6uUA8mI61pOlcB+yunPw
rtuCHMVc7AEYmF8sp8Og2TBYBhXsQCxHsNcDvSGwL4V4qhC0A2FIc791PTVMSwP+eMJQlkhzpv/U
YA8AL8ziEA+MQM15CUwnPODSwSej2Wi15xSC+HZhP0fdRlOwB4CWxFLphm9hsZzileXRONgjDfKA
8nqYL/IGmGIADwCf2xn3v3wkH+KB0YqeipfCSdU7w7tpAaBhwN6x1veknnID5OnpjI6lPscqP19O
sbecoeuFMYrkwrw7130iGkCqYHhLOcXT23dhmwlwtycbvLQ4YcraYjHFcvhO++UEfbXQV1XdVKpA
T9Qw7SSzIhkL+boDlwPygHJ35iAeGEGezh3U1oBJB8jJAPVA1FpvxqkjzBOz3lv3l7DOn28P4/ap
sipTF3tgjOi7WCoLPQRG13snQB51vb91fRurSHfKG9JRd+F+MjwUCvcxaG93I50f1oWUAVRiSUuB
PQfnPsDbv0/vFi7FpN0NkQf2IbEgD+DZrduc+Sn2M4kBPkCmwBDPUA7utYU5FtgoB+Kf2RkHd1xL
GlUM9oHVQP7k5fiymDF9ZFcFwtudz3BofYlFN8Gzl0eLf98LA/W04yx7gR6j671laafSFvpAn3v8
hu359D3gBclrWoF+Sko/gXrAdsE3+c+F+d01dMsGOzuqPKbA3iz3RspGCdwDYas7B+9b87wYDzEX
+612DWe3DxuY77oGanERMqzbCPUxD2BPrfWAD/alEE/LUGQMNOjV0Ue+LcAHfDP1hlSRGu5pUU1B
PDC6r3fRjDvqtPcDub6Ge2K1z4H4sUzZH1MI7AHiSTWzr3MefD2kh/hDIA+o55BrFQ8BPFeeSyAe
GMtetwp8dgJ6iFw2o7Wegv2hI3OzDRia6s52vbcC5DHL2OVa5QEVl0P2AvMhMzkw33WNt1QdDYa3
1a4bUAfsCPd/ev4BLIZ+YjdXRpNlOwH2JsDiysRqqaqqujZVgZ6IW7GNqxI5yC8BeQBo51MW4oER
5KlLn7amNkvVidHe+lFrve48ENcr4UAPbdNd6z2Qb53/0OcexPQu1QmhLvadFKYDsrOzjs0BDrTr
vRsgDxDG9X7BPegcOR1XIAz3tNMfhfZY47juW2kk8yyFFBA9saQhDPac5c8F+HN7dkdrl1leKar5
kN66DqZIOsEZp7sgDyiYf3ZLzd8+u33Y7tR7jykO+GY7dR1k4F7qESsK9yiH+LN7R3B5WMrMHWAB
4M2ZLVUM5HPjNzxx0rfkn10ewfl2E4sd4vHTjJ39bogGvVhO0bUTTAZY6JcNmllvXO83ATR94N1H
It3rTrIusqJTdUkDeEHyRC/MwKkCexuWXGt9Ccwv51PI5TiQmQJ7Y3m19pbBfcjqzsHObgLoQ1Z5
YHSxB2DBfLucoOkF0NKaZIQcdLwbPjB+jqUQ35FnMZ2lawp3Dnjn1qWJcQ4zz5lum+p8jdtSEA+M
gwmchb5jW3tyYdqO6l3Eap+GeADOYEaPuMVe5dmMWKr/HLBfm7U4vzt+7653y3w5tUBe58nKV0Sf
Pc9PB+Lm9pdAPM2rO30hR3rQEACEHsgCLLDf3VJ1gAb7ZuhzCKQD5GnlWOXbAcxlLyA7gT3Z4cnF
8SyYXzBtDQ2Gt9PO2LZ9p5/h3N4m+rZR/cNefQNtO1FebNXlvqrqplIFeqIJ6ffJ4clwDnVcM1gC
8uqPhoV4YGzcZMCSoaEeyLPWm/3M/VDADw3Y51jnz14+jE9tji721CqvO9bdxTXVEdKNKBMgr2ml
cb2/Y3O1IHUw9zs+vxDc67wBALZXc1GTEwbeuQ6KBAzAJMDehXfAB/jzjqV4LaNzzeXRgDwZmIhZ
7UMgDyir/Nlt1ak7f3nTQBNAwFsrAfhsnhm496z2g0ohHgBevqjug3MB7TWIkuBZOZCfA/K7CS+I
kLR1/mNn7odsGwP17dpIGqozr6C+I9ND5FLZaEUjYSxYZB69DoynXe6Db2fYp13vm3YYcGz9IHlN
N5b5pkXQWk+hPhvmOwExb0b+Gv4Pgb2eDmJZXq0D0nAfsrpz8O66W1PlutgDsGC+bxtM6DSGlrpS
hN3wgRHsSyHemmLE1DkewDvgKJ0pSl4K7qOLvCdqtU9BvPp7SKvxS3MQb/VzoPUXY7XPgvgAuIbA
Hhj7AeN66GGwp6JWeQryY8JsVjyFgvJxy/mVQDwwDsLIzMEFqqYV5p0pxz8ykIXRDR+AAfsjg4eg
7NMB8rSVnrPKz5dTbA/fuLbKA8rrR/YCv7OjrOg5MK/Lpw6Gt9d3VjC87eWa1z7oCPeLdmJgXqvr
GjQLYfX7qqqqbnxVoKciDZy21k+G/yV5Uhzkl4C8+l+wEA/4IE87QHoua7Mc85Ky1jdWMFu7kqf3
4lrvQ9b5nX7ds87Pt9Zx5pbRxZ5a5buLquETy6HJHSz0Qz+TDZC3OL2JrROrrTMuBguQtDGevWcK
9+wqBznirPFzv4MiGxC38KGTPVzeBXsX3gEf4N2OuCjNvnax5sDecccH0iAPKKv8+csq7+3ODPYU
UzuDKcDnAuqx+wMu+TGIB2BAXkM8QDr/TCfWuGgT614fmc5aAvIlEYn1Ungf2X3YWOf32qnqVDKQ
psC9BzCxVsPQ++RsRAUa6X68yXh+FJjzrvfWfHo03nzvkLV+dMHPh3lIfS39/es0lDywN/c1Aps1
AGXfpfmLwn3I6s7Be7/0t3EgD4Rd7AFYMI9emOdu7sWy1qs7cd3wgRHsqbIgnh7PfCcuwLvg6Naz
0kFpD/Dd9wQ4QxZ+XjiIB8b7Y/MdqP/dOhrACPfknCyI18/XffStAKbSA/vhwuq/TLDX0wks93p3
UKUXVnyTmDhwB5j3jDKIB8Z6b5U59OjHtqvB+EhDbvhqI8wAJDIC5AHIsspDqnuQvYBcNljKKXqI
PJgnZWirn+OyhBUMb285s7yvgDHC/aKbmBuXnUDfN2QqUVVV1c2kCvREdDlxdwozdceftDbgA2Ug
DwDNzgT9oTHRKMSTkVYxND7ACPZJaz3p9TTeSjVh6z3AW+c72XjWedmKsFV+qa3S4xXl0LCGAuQB
ArtMIJgcNXuDNYJ0FLPgflWPagbeBdNBET253oRY0hiwd+Ed8AHecwtfddWzDLAHbJAHGPd6KKt8
O3Q+ZCdMYDsAwJr7gOOA3xB31p4cG5xrD1hwH4N4YAR5u/OvLUbhTpE1l5fpPH1qPkY+zwX51Lxi
TjoQ3sfO3I+9+QyiFZBTbXFXWi4nwO5ELWOJ3iqrom0gp71xvQcGb5l+tFoBUJ+LnkfPSZcfxvUe
cFxjqTVZTQxSxzjWen1ECczLVp03esfEwX505R6O6MVYNWTCfcjqzsF7zAWWc7Hn5ssDsGBetoIJ
5kqt9fo3sV4O3kEG7AshnsIqB3VuOfHKtvNtuU/FBfzGfU/kLHpuCuLVD+FvS0j0w8gzec6SDAIZ
pSAeGKeYmbrWkV4Wj2waYwgM7zAB9px7PZWBeabd4tQd4o/jXPZLIF4dM9zbCi73QsJMuRCOtR7g
3fBNHTVRB8YC5Gkr/f2v963ybTtBO3zjug3QMC96UQTzshPGOg8JnOvsYHiLbuL1D3WEeymFub5o
G3S98saoSF9VdfOpAj0VaUXdpioG+EAZyAMKZNtMiJ9sj9spwGOaZ6238u30IGLWe5Wmb503QXeI
dV7Mm6BVXnc4JzsNmh7oGxjX+2CAvFbizAV/qZYcTbiVCTLgflVx8M6qV26Cw5+etV6lpbZwbrAu
wLuN/EqBhawEh9xEXPGBsHs9oKzy2mLhyoL7BoDnru48R/JcmyWximfC/Vmy1E/MGj/+73f8Ocl4
kGkrYn0uyHe5KyUMOt0pq5G2zs93Z0OQKDXYqJcka5cTBfoD1DcLMX4Lw/rlJpTFYOYSnfqnwVzP
nQ9Fuhc9sdLDdr3vMXa2e7gR9NPW+iKYXzRq8GA2fEutUMnMebBvGIjw4L7R3yUP9yy4D8/WS5vZ
tjfECXFd7DmY1wBBYV6LXW4w4IavIUeDfRLi3XbK+k78w32At3/7Fno2194BlseON0kpDfEAzHvp
u/yuTzNMlRISgJDqe+Cs9iGIB8io+5BrvZ/UJUGPkKWuj4dBtxTYD20JC/I6b5L3IuMUmmvPbS+B
eHUP+jmsgKCSBMVzrPUA74Yv9Kcw9D1iAfK0lV57KGzP14JWeQxz58Xw98n57Zj30yyY12Vhq59j
T3Z4tT9mBcPbnc+8ukNHuN9bzMzUGbFUXgOyE2rFkoRnVVVV1Y2lCvREFHbdNa1TgF8C8sAwEux0
nkIQPyEgJKQN8DnWelqxu7HWYtb7kHUegGedn8yboFW+2W3UdYiVT7vehwLkyV6g21ltXrH2iOXA
HojA/Yrr0LPD4RzQduPmphWAjvjtWOsBPiibF1PBaeTbnbLPeaItC647ewDsU+71Ko+krHYi7Dbf
E8DXt+UAvtUh6YUpu7lwn2ONtyCAWFvZhQv0cQn7B41Ynwvybl2Q0ke3H8KldsNY5/v5ZHAjVVBv
4G85MVZruTsBOjHWQa1QXtgD1MuBf9nOoP52mVvX8M+53tP59A2G+kDXXzOQBHlrfQnMQwKTPQJ4
MzWfJwT2zcK3htq3LMc6IQD3oeiRHLxzHeyQiz3gB78zAEVg3h1MDIJ9yFoPBuJjAA/722A9S1zQ
S8zlTQK+9Ld73hVIQ7zKm+AvEpN1LA/23vW1AiA/TpOgeeMvOQ4C6/KbAntmMElb5fWzbJtsK24w
eB5zv0UQT9LOHhR3NbxnAWlZ6wHeDV/3N+TgdaTfg3a9l2skQN6wjB1nlXdd7M099ALoUAbznTDW
+ctSBRymwfAWOzOIXXVdL8L9zkxNa5Kq/9h1jfISaMXqUwirqqquS1WgJ7LmIjrAngL8EpBXCdoA
D0Qgnuarh1prHr61Xm8bDjPWehHodAI24LsQE7LOa5in1nnRg7XKWzA/3LdrpecC5GGClZdd0c+6
Jx77OXBPXbxL1DPjDsG0NOzo4zTYr9udMTYgotvxdb0tCoP6jV4meWAfda8nnUVBrC42YDj519fT
HbGFsD4sD6p1WnoebBOGeyBsjQ9BvOWCGunwi0QgPBqxPhfkRSHQA4OrvbbOY4D2qQQ6MVqOO2G2
i6HjZzrUvfouuqlaEcNMiXEC45lZsRJspHvXRT/keo/Wnu+dY60vgXlAf/sO1A95N+VyeM9jEFRy
PPeg4cA9ef+h+aqm3Cc61SEXe8APfmfKPYX5TngDLSXWenUP+QDv3RMHey7Au+k58OZOtQmBiDXg
zhwrUxAPIDiHPSXvkyfWYS67IYu8qSO5i/Bwr/NsPFgSYC8m482ZQQY9AKIBuhPRQUuq0PQELpBd
CcRb+SkZYDEngxRl21oPgHfD1671uo5yXO+9AHmz8Z7a5cSAvM47tbBrmBedKIJ5sRTGOs8Fw5Nd
433TvRQqzk4/wnyzGNzte4FmCc+LtKqq6sZWBfqAvAY3AfglIK/Tnzgu8UGIJ42dINcWjWOtByw3
fL2tIcGd+8g869jy2dQ6f7497FnnRQfeKu/kX//WUK9HzN0Aeex5mTKDCToKuvPuQnC/6oh2w2SU
S6tZAt3GcKwL9gNkGLDnXHZDzzNyzZjM+3HAHnDgfvgzx71edGL8BtoRzAAwyz0ygG9Z5Z2jPTcZ
2G75zoOh1kM61zkI8fT5xXqYCfdQaxmwTJBvAmWSUycb7HYz7LVTY51HLwZ3eiXpDqpo2CcDgpO5
QLcuTUAu25d8/JN+n6FcmvnyEdd7epxWylpfAvOarEZPIwfsnQEnMxXICn6fA/fc0bxSc4NDLvaA
H/zOdrEfYH4AM5OnHLB3gub1JQDv/uas72567vnud50KlqmJmWSMs9onId7dnik91cS/4GgdBpw+
QRDk9e/h3C5Uz9B3rZOwwV52E8iJ9MDeZIFY5SnIm/xkWsVDEei5sl0C8VZ+VpBX7qX/Plw3fNrf
4FzvpYAXII+zyrsu9sqCAuOVVgLzuoyEguHJTmDifGd7QxAn2apo9s1C1fN9D2ApzBSpqqqqm0cV
6ImirukJwC8BeWBolBd2JR2EeOkco9sXx1oP2G74ehu9r8ZduSkRSC3XOi+6cY11zypP7sNqhIFg
gDxgdIktlulMkevo0XhHFO5jAxox5XZKRDe6BKfAvmMiUBsF5neXPi/zfDS8WyP6gxWCgL3nXh+w
ypu5yZ1wYMlWCvD9E5yjPTcZ+/xSiLc6QLE59InoifYa0nkg3+yVFb4Ly03szWejdX74BnW90+ln
sVBLGPVrcnw++vsagk+iUcWw6ST6iTAWeuqamhpcM9DRUCvXsE3EoR4IW+tLYN7Mj82w1nN5B8rh
PvlcEnXDVqvciHKC36m8EotrJ1QgV9IO5IG9/lbV0UUA76THTi1wt0WeN7PbA3z+pvy8ywyIX2le
MX2+EbCnLlMhkPcHq+kgfqAwOe2ZBnvRKK8bF+zRkDICBdBuPwQFc6yDg1KJOBFZEL+qqz2G/NNy
b97NCPauG77pf0i7frNc74mVXk7GDovshbHKq+sLY5XX96X6QSiC+WZPGOs8FwxP7E6s8tZD4tX+
MBbtBKITpk8hJCClQLNoTP1dVVV186gCPVGsAkwBfhHID9u9jm0E4rX7FO0YY2Jb6wHeDT/mch+z
3muYd63zH/rcg751vlUgb/LOwLw7DzcaIK+PdCQScq03V7ph4wYKuI6+BQ4JsI8BZeh+/BUM4hqn
ZAzXolZ5xmqfAnlgmEJAO68kyRjcAz7gu6DhPZIE4JdDPElrHx1N6hmQC/KpZ+Pq6Qt3YL47WufF
ohkGxEaQB9T7UGkrqJ/sCVPm9IBfN7jeS6E7qrAC44kealWPBNibTqSA7daq6ybHBTTHWl8E83LM
r9oesdZH7kGrGO4T6XEKzZcHwMO8Lrsa5od7d9sBlxejbvgFAO+lwA1YcHV/ZnrM6aEr+4kXQHyJ
9ZJ9vmTgaLwOuX4S5LnfAbjXkfmH+jkG9vQ4yypPQF5vi/ULLKWmlRAVQ7yG4VU8J2j+G7/Mc274
kGN/I+R67wbIC7rYt8JY5fU96vNKYH6yGK3zXDC8ZrD+ay3R40x7CxbdRBlUSB/TBMRr4/2+qqqq
G08V6IlKwM8F/BKQ17/dQOaeJR4jyFvB61oCN8RaD4B1w7eayky4p3Kt82cvH/as802HcSm/AMw3
LdB0Vmye8fDhGB0gTwJejIFsBTq3nmVocjCNHmfZZ9OVfh5CYB8riyFrbrGLnTvwQTtdjNVeclYn
CvKA6qyQe6d/+1MfyrJbCvjFEE//js4/jJdLGrE+F+Rz5js+cfIhPjc9TMcQGGNH6HcyxnMYpkBo
i04HYDa63svJcGv9mK76X6IXoxsnF+neXMENkDekY74RMqBn8uCIWutLYJ5CPc3/CPb2e/OOc+8l
A+5lIp5Cik6D8+UBHuaHcqxhHsP7MIOYXN0XtdajCOA9cQOXHrQ66bvwlnqGjnGZ3QlYdVXwnRqw
LWhbuPnZDXxrvdV+J0CevY6VU5KWvS0G9gAJNGe51zv5ksh+Btrzzt/un98XQDwATHbdESh98rA5
4rjkvWNuMMtxwzcH6LZYYKzzhMqbGyBPBzyOudjr/DRz1f6VwLzoxqXquGB4zdKOd0CXrNMeBUIO
bUhP3m9pf6Cqquq6VgV6qlTHJ3ZqAcgDwwiqs8y6BUIMyJt0xFiBU2s9ANYN37qGuyEBtJx1fr61
7lnndQPnXY/AvOlY6E69DAfI69dWD1JnAKIA7FOK5YUzpnNwTd3+U2Afc8EWq05FCKXngD1AyjMz
lz7kXq/22elGAzImAN8NNhhboQFg3kMxxI/HT7fDz7g9HO8p2ctP5oH8pNC7Ymtv3bLOTxbCKqPN
+vg+RDfOIXW9OCjsA7Ai3XuBuHUnMVL8qKu+53pPQN9KF3450flaFMC8GILu6XLlAjtnrafH0WO5
+9Ki5XS/K0WG5ssDGNaYJ9+XtsoDBuZFNw6yADzYp6z1xW0dhTOuonM/xNT1EoBv7o28J+4LTEI8
SYw7NgiPZFTEmp/tuOFTgA2BfPagK1dH5YI9515PQX7IdLbLfQD8uUHIHIhvWjHG2+mcgquzZ9LT
f4TyxmyMueGTATBtSHBd790AeZaLfTtaI6iLvRo0HKcrlcA8iHWeDYYngcmuMBHu9ZJ129sb5p7E
YCyR3TDYEOiPVVVV3biqQB+S04AkG78CkNfpcbATAnnTKdZWZTGma/WfGDf80Oi386cH9x0a1jov
W+FZ57kG14P5oeNtrPQO1NNR82bhA12u6BJ4JWAfdy1NXC83b+Q9ctfUYB/xuDfLEYbSzpXpwDJ9
Ks5qn3Sv18cRK5gVFC+VHxeO3PcTyr97Xe53BsRTj5DYO43BPgB0R8aMZYN8Zue6G57Coh1gfrDO
m+tpN3rHlVVDveiIN5F+NzNSPxlLHvmb5s2xCAPD4BsZKKCdcOrWqs8f15tHFOoBFMH8dNuGlCTY
a/glrWAp3KcUCsCplZwvz7jY67zTKS0G5BmwL3PD9+UBXcCSbOR+t+53mbquA/hST5Eg53n35CgY
D4Oxpmfly3l41vxsXWZ67ng7PzorjfM7Kvc6CbBn3evJd6OPyZ9Dz2/n2qEuA+J1nqy8ul4cTpwS
9j33MPAeAnu3zGuXezMuOQwAUtd7DfU6QF6Oi736DeNhWALzoldL1dFgeO1iOgbDI3WmyrJasq5b
NmhonBqp8lqD4VVV3Zy6JoBeCPHVAH4AwNsB3APgm6WU/8Y55kcB/JcAjgP4YwDfK6V8huw/AeCn
AXwDVFX/XgDfL6Xczs5IZLQ3BfglIM+enwB52hGxXMWJtT7ohu9apDlLhPC7Zjv9OmudF/MGjWud
d5+PJNY/3fHW+XCs9CQLJkBeaMAjS5JkZwWwL1V+x3jMSwrsQ9AOhGHTnaOckgs8TMwpuyOVcq8n
IA/AC8zj5i/12Lw58e5+57dXrgshnuYv5pGRGmiiEesPCuRdLXZm6vkO1nkDdwLDkkXDu1oS9/du
jMEhJ8O7aZ2AdHRgUr8/MtDGRboPDYxZVnrAuLPqV5FlrV8B5kMeOB7Ym2XAyLGFcJ8C9tT7Tc6X
Z1zsAVjvm3pNcGCfml/vKg7wzm8OjLn6iX5/hfW6e08AfMj18sVAPLk2N/ffW7aTuz55eGx0+5y8
g5Qn6nWQsEKb9ioB9qx7PQF5QNVDue1WsC3iHpcD8QB8azzNh86DW+ScMugCPjAYBsyPMNRbWdXX
BflsXNd7GiCvxTjANMA8nU5BrfLmnB5FMC964EK3aQXD6+YTEwzP/Z52+tm4ZB2NUyOhItx3Ncp9
VdXNqGsC6AEcBvBxAD8PBeKWhBA/CODvAPguAM8C+CcA3i+EeERKqbvGvwLgLgDvALAG4BcA/G8A
/npuJljvwVAj4TRAq4A8HflOgTw9XzYEBIl13LKEAWMk/gwLCQf5nWxY6/xkWGfess478NQsYTU0
BtAHILCs9OBd7/c1v5003BjSNPdZ2MEFEnnJBXptAcgA+2hjHLIcFgK90NeEAhmrDDBW+5B7PQfy
wACWkWeTAnyRcGdOAX4pxFt5jeU7MRWETpc4aJAHgNPdMcjd6XAt8i5Mj3/M52SurmWgXo5QT13i
m+G8ph0i3ffjN6uhPmQOnewC7WHHSq8HA3RfWH/bZEBAIm2tL4J5Od6TzgOVB/Y60Cg5phjuE+8x
1al258vnuNir3zD1rrVyCAP20fn1KAR4cg11buTmQuAfausCMt8uccU38Gu9sDjEAyS/3EBEKvib
FxBvtNa7lw+CvL6Gfhd0ZwDu3eeTAnvOvZ6CPEDq8hwFyjBXthumnvWs8SDtnNP+mf1uFji3fwnz
feSCvfetyPFfyPU+x8Vep63ns5fAvOiAFxa3BoPhTXYFiYsk0UFg0U7MknW0j2gCIEbq66qbV0KI
uwD8awD3AngJwH8N4H8lv79VSvm5q5fDqv3omgB6KeVvA/htABBCcC3N9wP4MSnlrw/H/A0ArwD4
ZgC/KoR4BMA7AbxdSvlnwzF/F8BvCiH+gZTy9Kp5y4b8EpCPQFsQ5J3GUHcwk9b6YTuXV+u6zD31
w4mudV6PDLvWeZ1HA/NWQ8N0hKmFiWZNN3arNkgUbBiwN51z18oTs7QUAhhnLXcDxMXAPpqX0HMp
fF4UTKmThQX3tOwUgDxAplropBLP0AN8xxKeOt/jjlKIp+dGnmXMIgfYc6xzQb6krH90+yF1jmOd
bzqgH8qPhvlmMQyWgdRN0nGxJ4ORUgjV+XW+WdEjGOlel2XO9Z5a6V34pJ9+0FpfCPOihWnVUmDP
BuQjf2fB/X47ze58+UwXexODZIgbYLyyQPYP+YvNr7dPCvzOaD/YfRZURwAyAfhj9UzSmPoQnYR4
cgJXP/vLaJoLq3ScDI2Xs8Heyxd8yHbfhZU+uQbdkQv2rHu9C/IF5bakvYm51Vt9IDev7r0lAF+n
oSCd3OoAACAASURBVI/LBnuMdRCdTx9zvbdc7AerPDC2c3SKnzZmlMB80wJPb99lB8MbYF3X1bpN
WaLHpxd3qwj3nZrSRutoOuBQgb6K0b8G8FXD3w8D+BMAt5Dfvy6EWKAC/nWpawLoYxJCPATgbgC/
p7dJKS8JIZ4A8BUAfhXAlwM4r2F+0O9CVWlfBuB9B5ondmR62JcB8myaKZB3Oz0EBlPWem9uMtc6
Bir/XOu86H0Xe51PauUTvYRsRNJKL/fRIBkwDYE9tdaT36t2zll457YBbMeFA/tYXkLL0xVPHSCd
xhDc9yRwYxHIA2jmdp68Oe8pwHcs4aWAXwzxmR4hqXJC3/1BgrylpbCt84M01GuYN7DfO1Z56ZS7
Ic9yAi8wXtNK9FMEI93LZrTSW/fWk+tpzyLS0dbjefpv1lpfCPPWAEUK7PWynqE5wuTvENzvVyu5
2AMjzA+DZqYMMGAfm19vHzyeY2cydgPh3xTi6bfslnnP1TwA+Pb78K32SEA8TYurA4JTlhxvgBDY
86uduNbyYbtTRoEw3LvlNwn2jHu9C/JF07MC7597hlG3eoTbt8a5JzdpLnCtDp5L97tgHxpG0lNR
dNBI0y8Qw78e1vryOu8xqzxAjBkFMA8AW+26FQwPS5Wxya793qwI93raI/mORa/S5KZGVVVBgTrV
pvP7iwBsDH9XwL/OdM0DPRTMSyiLPNUrwz59jFXIpJSdEOIcOSYtruHKrBVXAXl7rqhzvtsBCKWR
sNYDSI5+6+NdaZjPss73BKADMK/vyzxm3Rkfd9uu9/tUCOyptR7gO2JeWpEOEOdTwnpf9Bh7pRGw
B8LQbqXnpFEKh5IcT28h5FJu4D4D5PX+GHSnAN/9XQr4pRCf7UqdKC9WxPpMkM8djOlkg91uptaX
J9b5yQImSFTTAVjAghfZqG8L/fiu3f0atik4GE8ZCtj093AvMSu9+mPcRgGFGXOzrfVANsyba+pn
mQB7w3nkWq813K/iYg+MMK8HZsy9cWDvgrzzOxfgU3Pj1fUDEG89NOccN8lAfqyBq+F/zmrvJmKd
F6vHA9+g562UtNhHQN69d5qfANy7+UiBvTto0JDI7MbwQKzbKRUNFNM2JQPkQ0oBPgBM6CB0LtiT
uktI1Qaaz0rXMdSjqBvz3jgridB7osYM0aMI5iGBnXZmBcPT7vbNEONEPzcd4X5vMTPfvnnODtgX
D/BX3VBi3Ou/dfj/YXLYDkYLPScX8N8L4KsPPLNVB6LrAehDcpy1Vz4mrkzILwJ5p5G3zg+BPPkd
AqGQtd4LWMdYIrnG/Xx7uMg6TzvhLMybZ+RY6Sfw3pRpYFeQsTZqC5xOM9SRDHSQqGLLinGdet56
QX5EwD4kN3/eOu6FzytkPQ/CfQHIA+XQ6pbrVKduVQt+COKzO0D76CjFnkmuh8CF5SaZnzps1B1V
pyzrZ2hZygED/j1gLHtsZ5tavrUGCz2tq7SVvjvE358VIM/ZRrOv/x47uwUw71ijs8GelrtCuO8S
LWhyYG4FF3udN/1c9HMw9yJssDevNQD2rOt0BrwDNsADYYi3BtecdNxkQ4O5NjAP6ZJt3SQD4mN1
ZO4ocgDsrRVBEiDPJhuAe1cpsOfc6ynI0zQOWt6gbALkPRh1NeTTbQv0uRO3DUiAvdoJu7IBxrqD
cb0Pudh7VnmM9VEJzEPCC4ZH157XcVCAMcL9YmemHo2EXQ6lnW7VTS3Xvf69UFD/XoyQ/19BxRrT
v2dQXs0h3S+E+CCqxf6a1PUA9Keh6qq7YFvp7wTwZ+SYO+lJQogJgBPwLfuWHn/8cRw7dgwA8Jkn
PwsAOPF5b8Otb3hMHZDbsSkBee6cBMjTBoCDen0OZ63PcXXmYKLEOu/efwjm9b2aR9EPU3YZ1/tV
5Xbkc8E+Bu2xDhn7PDO3hcDejfAOpAG+2KshZZFy0iwGefL+uQwmAd+5v1LAp3CZBfHkelwH0lx3
hbXHc0A+VsaeOPmQ+fvpC3dgsjuC7mRB0qNWXAqEwzU1xEtA/TFh3gOxiKpzFMGL4Zt33Tl1PuQk
HiBPNsM3Ti30CWt9CcxT5YA9zZ+59VK4T3x0SdfmVVzsQZ5LZ9enVtugwT4yv57+bynRxpn7W4aP
yQXpFOCb45hjrO86A+Kt78RR8LrcwA9NQ9d7dH/o2TKDnVx0e6u8aS53rh8Ce8693u0LiA5JTyP3
Ov6O8DnZID8oVN9GmwgykJUL9kJKFSdk+Las+fTAOC1Ou953yLfK9zCgXwLzTQ/sLWfhYHhyfI69
FCbCPYb58gLAhac+hgtP/Rm6Q3KYfw90i93Y06u68eW61987wLdrYTe/hRB3wgZ+F/BPAHhg+Lta
7K8xXfNAL6V8VghxGip6/Z8DgBDiFqhC9s+Hwz4E4LgQ4m1kHv07oOq6J2Lpv+c978Fjjyl4f/v3
vMc/gOuwZXZ2zL5Qo851AAIg77rnhzoZnLXeVWgwwNWHPvdg/tx5kqcYzJuGlFjpZT9uN/fdA/2K
I8wWUAL5YL/q9WLvPtcC64D9hEvzgPLLKeluikKQH35Tq7nV+Y94uXDX5vanAL8U4mPXsq4b3qXy
Qb67/YJ8SEKS+5NOfsUI8xpoabyIIeYdgBHw9T40sCPd0znHMauPtAEj5HpPg7S5YM9Z64tgnuaP
sUZ7YA/7WHOfOrkMuE8BfUqruNjr/d5cXn2a0zbE5tfbG0bxg77+zXrfZK41PKLQ98CNDdJtSYiH
/2ys6wbyQwdL2HOZzFj3EBm4567rB9Rz9ifAnus7uGVBu5tnqeA9xuo0tk5lngdVbGDVHRAE0mCv
8sZAfQ9rPr3UdSod3O7G++Ks8ipt9a8E5gFg0U3QLqYmGJ72qpjMCcxDYm/oxMh2nMsveuDEFzyG
E5//GLZfJzHdFli7BOy+8gKe+ZWfDD/Aqhtdrnv9S6kTXOBnAP9+2C767qBB1VXUNQH0QojDAN6A
sW17WAjxVgDnpJSnAPxTAD8khHgGwHMAfgzACxiC3UkpnxJCvB/Azwkhvhdq2bp/BuB/30+E+3CG
Mw8LgLxudFrayCRAnloW6Hq4Odb6FCiFtp29fLjIOu92/j33O6fDQ61wboA86kpWKrcDWwr2+1UQ
BmmHPSTtYcF1iA4a6APwEsp/CcgDqtNDO1I02RTcc9tLAb8U4mNpl4jzrjgIkO+GB7K1t642SMc6
T6VhnpQ5F+r139DePEKfJwz1N84zZCPdD51hDfWhAHkAMJFApzvZBOyD1voCmG9a+zfdz4G9eR4c
KcJ+riG4Tynp2ryKiz1GmDfb6LxgJ/+cGz7gH0fFRqVnyqvngRAaIDuAOtayautt9NIJiKcncvcc
elem7MSmajj5YyEeTHvOpeNeP2Owlf7m3Ou5Oie7jisZtFoV5Pc5mJ4L9iPEJ6Beu9739rOLWeVp
O1gC85DA7nyGbj4xwfCsFQJ6VU0s0ePV/jAW7RDhfil8Lzg9MNjC8/qruunkutd/a2kCDOB/EKOF
HsgYJKh67XRNAD2ALwHw+xi7ij8xbP9FAN8tpfxxIcQm1FyP4wA+COBdZA16APgOAD8NFd2+h5o/
8v2vTfZtpUA+aLlGBOShGqG+EbaVS/qNmjmeDAKkxHVm5lvr2dZ563ohmJekw0ms9GbuL7VKCeY5
FqoU7Fe+Tgn8kQ55PFFmk/s8XMAvhdCQJSwA9yUgz6ZbCvdu/goBvxTi7cTDu0qe80FZ5KlUh274
Ie3vzrjnku/HrkOGuoO+Vx3HYjifRrof3XjDke5nGsbl8PmGAuT1drm3wD5grS+BecsK78K6A/Z0
X2jpsyy4T1QeybJC6wPy/jTMcy72Ol3uXqw4AiQPnhs+YOrFXHhn7yl2XGhf4BkHxdC7e49uvkJ5
9OIHBM6PZicE9gmId/8OXddriyODENzxnHs913bsZ9AypOjAMFNec7+P4D5iWKDXD4G91f/goJ72
Peh8eth9nUYbNkBgnvRxSmAegBcMbzIfg+Hp61/ue5xpb8GiU+BPl6wz99eraz35U4/jYx/7GN7+
r6qF/mZVwL1+v9r3IEHVldM1AfRSyj9AwpNVSvkjAH4ksv8CgL9+oBkrUCigFOCDvBH5nQJ5c7xu
f2jHhDSOWRHbczsurSiaOx9qpGkDx1qTBhCwrPQS/vPKlduJzwT7g1Ss46bhJ3QcgDGvVAmAzw2q
xp0fdCpgOmg5IK+t5V0AfrLg3suM8zsB+CuXn9C5ERdl67DAezmoKRKLnRkODfNirUE0wFgpzTtx
Vt/Q+6SeIzpYgAz/Nsx3rdNzXdvp8yfWrZCVXufDi3WwBs8NX2slmB/yY6XHQKH5BummQrhPASkH
jlT7dbH3LzjmKeiG79aLK5Zn9tgciA8dk4L7CNgDcJYfyMxXhlIgzXs4pP8eN8avxw3QxPLDudeP
B43XzG0vSgLo5brVH8RgghWLpwDsWagneXXn02db5aV/XA7MA1DB8Li158lz2pPNuGRdP9YHbh/s
SgzUVFUBV2yQoOqAdE0A/Q2nFMgPsiL/pkBei0C9OgZJa32Oyz3bKZk3qoHJsM5zaXgNnLYU0AZV
W+kB20qPckD1VAj2qyrH6gKQ9zlEi4+Bfc47Cj7vXFErLb/ZlmOJioG8uR8aeT4A5CG45+4nCFwk
vVVldby5a7MnpdM7yFgHp7tjkLtTiCWG74fkQ4IyoapfAt+n+Q51/nXHkLH2SYFopHshJdALs749
JIJWerpsHi03nBu+yXMmzOu/LYXA3k7G22blJXJQEthT757WUTLTxZ7KqTO9NGGDj2utZ/OcAPjU
d+Llw/k75m4+HsSc674wN/9MXlJeTVnS76AA7IsgPlWPOQNPKbA3xwdA3vzOrC9LAFFyPyLPb1/w
KZlvOAH2ljUe+m8JCTFuJ4OTgiROrfIWlGuwd8p4LswDgOwEBFl7ngbD0/VbB4GXFiewvb2h6t1+
/Eevu6+B7KrrXtxydVcqEv1rea2qtCrQE4kuXBPKSaL1I5VqCuTH65G/UyBv9o0NkT6k1FrPNaLc
sWqOVqZ1nsmnzqAL89QaJQDAXcZOW+kPSplgf5Aj21HIb9Jgn5qHrU5M/E7lMXC/Ibing0hAAuSZ
TmVofekxQ0iujRyF/BU6MqFvMPvakTRLQD63vH90+6HxOk4n1XRW4cB8rPNO6wzdsQWsegRAMtI9
ABNUDxLWMnYc1OtLAwmwL4D5pgU6Z+lC95pW4XZAiWzytwcOygb2kKz7KXCxBzyroJ/pcVvQWk9+
myxFAJ473lII4vcB95x7vTVyxbnPB64X3RaSc2zo/pMQT/JlladUvehe39ns5YdLbx91ZJGY5x4D
easO45KLeAfQ+iob7C2Qz4N6A+XUKk9B3sm/jlmQC/MAINrRE8lde17/30u1ZF23bNB0IjyQUC30
N7u45equlFX9tbxWVUIV6ImiS1UlWsLJ0nbbiinUiMVAXh0AA8G04aKglTW3nmtQuEagL7DOSzsf
NA/uHD7XOmi5zmmLHMIMsrJSYL+iVnZ1j4F9zHVRp7PfzlmgY01l8Y8OupQF8n5v281vEvA5pTrv
OUkUQHwyrX3CwkoDV3oNemqdh/MdcVBAoF9L9E4eBijXke69MuJ0IM0+baUHxgB5E9/13oJ7Avac
G76+Zi7MZ8lyARnTczc5m/Ms94yS5UsPjpW62Gu4cOsebvCGbHOt9UA5wOcCfei7tAb5AumyHkrc
eXRAKpCPVB6ylQD7FMQH03LfV6hOyAR7MyhKq2AuD7l1X0kdFWlTPEtyzqUTZU0MF8wFe6v/lAn1
nFWe68PRJYNLYB4Sak78Uph4FtqtX3tBNUtgT07NknV0JSEReeZVN6W85epukGtVJVSBnihWGSaX
qsroIOwL5Kmk03ClrPWko6EDXnmXCEB+tnXeaVhYmKf/uw2ra6WfIPw89qsQ2B/gCALbuXOvGwB7
AFlR7oPpZ0qQQivJ+ltBuC8C+YzrpwCfE5d8xnk5EE+XI4vVBSVzSoNpOHnOAYxONtjtZgr+6Ldk
Wdkj6Uq702rO1QN1HVSLIGFFunc7jGyke7qfWOn1NgPycow4TcGes9YDQK9bqEyY37chshDu91tH
FbvYAzbMx2Axw1qvr0WVnNpDwTu3HEv+7+Az5r4/OhCVaDKD4p5NQt7AeALsgYznEBsUcvKm69re
BXXncG8wL/IxWPO/UyqAxCxrPJP2KgMs1uAUSIA7RMBeOlVmBtSn3OsB8MsEZ8K8dp+HHIPhWf2p
4e9XezUyKvVc+6GudZc0vmL9pqrrRcXL1V0n16pKqAJ9pvYz8nkQIG8FbiHn5FjrrcbSnautlzDi
IL9DtnVe9LZlIATzZk4YaUTNdp1gP3jurlg6LU+FmDI6PlnX49zrY2CbAnsAInYDISAtBXp6fAbc
Uw+WFMivEv9gJcAnefDSS0C8cKy6uaCWqgti0wZWAXmqC8tNCOoNRP5sFmR5piFtBYp24CfLtR4E
wgHT27Uj3UvIiQhGupdCQEg5Xkdbtgao11Z6CvWAA/YhN3wHfJKWeV0ehfVzNWWAZxIMExkodbHX
x1rLYoUuE4JHp10oAXgrXSRuPwDxOWkl3xvTtgXri9igR46c5xQE+5zBDL3JrDCQcfzwOxvsEyCv
EgnkldOq7WIOyDteRkWSdtNtrPUxsEc51LvvnyZGrfJW+gUwDwl/7XnmmzvT3jIuWUej7A9TRfW0
0IOM2VJ1Xeq1jERfo95fQ6pAfwV1kCCvz+HAPmWtj3bAHOu9l5Vc67x2x9XHcJZ5QLV0Q4Mrhn2m
ISVWejm4nK0qdq5iSFzHt+haXI8tI08hsAei934l3Oqy4N61BCAM8uy7L32+3PexgmXN/I4BvHO9
g7bQJ0E+0gl74uRD5u+nL9yByS79yGGsMkISq7ew75eFerpkU6+AoQOKIt2Hgjdyy9jJRnVYjdWd
gD3nhm8pA+a5tKNgX1I2VwXPhEpd7AEb5s25IpIX+t5oprl6h/kdAkyaBruv9OEkBgrY5EogHqQO
zs+Vl2YK7KMXoO8QDgiGBkGd9iIF9jkgnz3oDaYsxo6NudUz+6LQn5EvWpTHv8Ngb+q7EqgvAHl1
XRTBvPndjcHwIEldt1T5WcqJWbKOevBokE/Fn6m6OfRaRqKn1xoC5L1XCFED5F0lVaC/AuIaAAo+
pkPgNqgRkB8bJrmStd49lrmkZ70vsc5TALAC4ME5ljRWuuHirPTGDW0VkVa+COxXVawDRbcZS/xw
WgDsrWMZiZz5HRkKzVnNgvsUyDNpeble0SrjKZBOCcADkQ61m25BucwG+ZJOs/PdQP8pyabOPzY1
HcJ8wwykhyLdj1ZB3kovAStAHjCCOIVvzg1f79PnpGCeLl3nwhYH9hbjrgj3qWNzp8lku9gP57CD
qjmDFwzYFwF8KN3IsV6dEKh3kumkspSAeAAj0JYMDLptSAnYx7bDrnOScF8K9vo0if25tu/T5Z4L
Ouv2kULLlfahIJfDubaF3krS7ivpd2ZBeybUO8+Mc6+nAyRNCz+OgZtv0j/Sf3Nrz+v9ogdeWpwY
l6zTdbX0n9EV7etUVYVVA+RdZVWgJ9qPVc7t6JrtQ0+R3S+c/xEGebVt6LS6a6eSdDlrfbQnRNN3
dpVa54ULETQtvV8CxkpPjjWNqF7GTsTfR0waeA2A5oL9ipwchXc2f8MfIbBHHNoPzKXOuh7ZHIJ7
BghLIrp7LvXeAek0WIU6shnHHWSAPHMZ5j5SIJ96ft1wN1t76+r4DiaiPP32vPSk3Um1MzWePw6q
MZHuDUCoCkgMdYKVLgNt7jJ2k107DxbYD+e7YK+PyYF5LqhkCOxdBeHe3enuSpX7RJmKutgP57vz
5d22xHgmkHsMZouSD90W++3mmeznDk16n2TUO1x63DhKKKscxEfzlKEU2Jekz4F8CO6D/YUQ2Dv5
deNg6HNz25Gi6VMULmNWeLduCQB9Ujod+I9JFfWhL2B5JzFQL4fyx0C9TjHoXi/Uv9D0H3ebC/OT
PWnc6IXjmdiYPhPwp+cfMEvW6X6X6CvAV+FaWT6uBsi7yqpATxWxYIk+UWsyEK/O8/frBrgjDUMK
5HVa4/IrfkNFz7P62FxPiNvn3H6JdZ7mMwTz2hVfp+lZ6SXJt8Q+gN6+IRfsD7oBXKVDZ53ngj0O
ENpj1w90zLM62bkgz/b4+XMPDPBj10YY4mOeLHYC+VnIBfncsq7mUNp5MP38wKCivo4L9aKT47fh
ApCOdD8TvmWNdEj1d6y+K99KT5exM/fr1jMBN3xg/CayYJ4qAfZwngUim6PW+30OAgVd7Ie03fny
dsbGNNzBkSJrfQHAu8ezj7Ck7iqAe66+sq6fgPis+fYZCk6dyjwPsO/Ve62TAERngr0pr1zZIt9t
LqhnryABoIu41XN9Gp12qB2Jgb7Vlku/ONuQL/1zyXEAVFR5BuqLQZ5mwtnGwrz0g+HpAUzl7aiO
Obe3aZasCw4Avgb9hqprUteCdbwGyLvKqkBPFfPDS7mrBhprt7G1knTmtNLjaZpWgzgOGJvzUtZ6
OsAg3S5YzHpfYJ3nrHT6Hty5oSpgjISAMDBgAB8YrfQrylgsnBvTaZqG330UqzaGuec5785c1wF7
INHhvAKNdg7cc2WzJF0u/fHADMDfh0ohPnZ/qXxFO1oBkHenB4S02JnhkLMkkgpemf5uXKjXnjAj
GCpLEY10LwWAXoIGaRQ9TKR7c6/O988uY6cjMjMgxLnh62vp/SmYZwefQmAP+5nHAnBGrfcJpb6T
EMjrfUmYh72v2Fqfm6dQIqvAbOj0UBsa8Xixzu/5YyyR57Zfpby+op4K9J05hyXHiNwBJvd7SoC8
ue4VAPqYW71gvvfRQyVE9PE3Zb0D591K+I+GPXc4SU9NcKGem5seAnkTG4EZYAzBPEDWnmeMI5YV
3lmyzjoWCIN+1Y2ua8E6rgPk3Q/gBID7hRAfRJ1L/5qpAn2uEpPuQuATazStOa0pkB+2s5a2hLU+
eE04gB9wyc2yzvd+w+fB/NBo60691KYkDSYgjXAvTbCXUhkIGH6Xgj2bZndArSQFIPca1EK1iptv
aRbd4xnLudt3zE460pPi0ol1uLw0M7VvS3wo3ZJzDgjkAeB0dwxydwqhrVZyfJb6ezRlnHj8mEEz
OPWHgG2lp3kWMMHs1KABH+nejEM6Vnot41Wkv286cJMB9vp3DsxHvW/cOtX5GyvAvZt39vjS8kU6
5CzMB8DYhXog01ofy2MG8GfXU6F7EOzmQIWQuEbJfRS8FwN+iSkBJdewBjUcgi+KuUm/pYDnkzEE
wNmf+wxWnUPv1HnW8pKOh1fYWyAvkyGw95pbMt2PnhuDelcxkDffHp16kIB5gMC8HOvIZrDOQwL9
VGDRTsySdTqPrgfDQU0bq7rudNWt4zpA3gDxDwC4Zfi/zqV/jVSB/qBUMPLNuaABcZDXv60l3wAP
7FlrfSwvMes9yW+pdZ6FeQkYy7yGEfeeaKdknwxdCvYxaI91zEut1ZLtaRSkmdHRzstQIl2nQ5MC
aro/GnE3F/LdMhW5Hqd9Qfx+yh5zrRTI57y7j24/ZB0vAWOdN9uNt4cubMN/HNQDtpVe+pHu3bwJ
/b1jPMeANGOldwPk6bxY6TL1h+k0M+84BPN0gCNnCbNg59d5N1HAT7y3okEo6mIPZMO8lZ8Q2ANx
az1zDSvdwjpOXTh9DD18v3BvKXEfRe/F+XZLAvlx13aP9cbhQ+Wy1GvLAfdQ/yKlpmQwm3ryOCBv
1VPOewiWr8LlT902lh1X1g0LN68ewzm9/55zQL5ZDqkcGkfRUjAPGVh7ngbKFMCiG5esswwp1OuI
WO6rbmw58+bPAHgCwB24+svHXQveAjelKtAT7Sd6eNOVWPBtmssBeSraOacQrM4ZYDXUO4paVPwL
5Vrnvf0MzFuA1Q3QF1vGbtXX4TTY2WB/kA0h28FSG4UOshMD+4y8HLhr3Qp5AMZOGYX4fpYwC3qJ
lB+TAvxSiD+w5+kCMPYH8pbIGvTUOh/Lg9lEOs+WVaobvWHMd83MOeci3ZvOqoBZt16Ct9KbqTVO
zAhvkCCiGMzb9aB/3zFFLVsR6/2BxeNw5st7eQrBvLTbA+882G74rLW+FOJjAwshiOeOaZz3RI+n
7zIQ9M1LOOc+9LPlD+Xl1jspsOeu66TjQRg9L5A5p9sQ1gGB/CqKutUXfpPuOa7ccs+d5z4yy4vH
sdbTczTUA4Ugb2UwD+Yh7P6UtU+O9Z2UwluyzgV61xuj6oaWO2/+j6SUn3cV86N11b0FblZVoL/C
GhtRv5YtBXlAWdD6id0ZNocFrPVcNGkw53B5z7XOiw4QJFqvOk9ax4/3paz0SATIW7VhGhvmMrBf
Wczp3OCQm68ssGfOD+4vdLeLLZuYygtnjTcQD3idylR6KykB+EUWM+f41BJv8XRJJ/GAQL6TDXa7
GZpWjNboHpY1l1rV6XVo9HcMVlrdsVX7BUQnzSAbjXSvrXPG2uZEuldpqW06SBR1vecC5LlAlHLD
16Iw7wXZc8/LgPvg9ky4B7D/tZ9T8+Vj8CzHe+DgZhVrfe7gkJt/TsnP3YkdEoJ7mLJHTk3AfXAw
LxeKmWQ9jyGnHMfqE0/0e3QGBkLlzwzGpOrUTJDPbi8K6qqYWz0H8SXz8zmlLPwu2Hv5iVjruUeb
BfLDBXNhnvZ93LXnhQS6mYBsoCLc0z6Z1P2MkGtL1Q2ua9USrufS3wvgLICZEOIzqGvTX3FVoKc6
oMqQt8YP+yRzXAbI6/1S2svU5FjrtWjj6QafgnOuzl+udV7IIaAWCMyTDoWBEDk2pNr1nl3GTuwD
qvR1zU96AyII9qtaaEs9O3LAPtbfDHbECp+Xe7/eXGrnZ8ilPmaND3bSmfT3rX0AvDpg/LNfMX4D
YL+ffVvkiS4sNyGWdr5Mp90JxmTHYxAQGL9HDeJjIkzeBgCXQqhvlazyYQbzyMCCcbnX6THwEUVO
GQAAIABJREFUYDY5oCEbpmPuPH4X5q26koFxC5JCoAd+u+fpEQOfognP4evuB+bNpgDchILmibGC
9NKOKrI/+NUw7Z70R1ZZuOfS4JZqS0I8OWaVzzAF9islyrS/nqt3Zx9qOd9FBrBSU/tyl4orqbNi
bvVUxnrfr/5CqLU9G+xJfykG9pYLPrmnJMjTfOTCPEgfieTXrVe7ZaNmUslxf+g+q24KXZOWcD2X
HgCG+fRXO/r+TaMK9AekXIi3oKkA5LW0hR6iwFpPO/dDZyEJ97CjrtI8s9b5DiqaNUaYt+bghjrg
ASs9dXlbWQwcU6u9C/ZXXE7nKwT21rFsOvzO0sbcBxc7ARfwWYgHEnkl6Yd3sQdw91Pi4lwC8J72
USi4QHerdrSeOPmQ+fvpC3dgsjt0OMmUlpzOsIZ6E7SOdIS1lb5pe/Qz2JHuJ8P+QKT7caBOfcAx
Kz3rHqrT4tzwYa9Db8F8APxp/gDY3gmR461zA6BM0zX79tmBXmm+PIX5wVOCHl9qrb8iAO+cZ92b
sLdZZcKx2qfStqz2wt8P+KDk7C5WCOxD6ple1mQx/s0GEGXuhT2MsdqnQN477gAVGxizrPfuIPgK
46eig7fk64GAPTUuOPlmQT7hJZGCebPWPJzfUgXDM3VeL4BhyTp9vGzEFXmPVdemruF58yFdq14E
N6Qq0BPtx/2ra+yKPgzxkhwTtmy6adBzm04Yy0SOtd6zvNEkmwDcmwPs/ASt81EXezsfISs9XcZO
Otf2VNIBYDrHHNgf2HxYJAAuAfb7SrtAtFyyUwScbf2MrqsXT3sczBkPdIMuJgE/t+8Uem8lAA/k
DyAVwH7uu8r18qAu5tS6O1lKC4xs13IF1HoKjqRWKGJVV0EhSZ3kQAUX6d6CcjGwpbYq5ZZTArMu
2JtDXJgPdKJdq6AHjSt8O1HrfWHQLk/k+dHfJn3uWALz1uAqOSbXWg8A3Rqfp5CsNCMfsRszRUtH
AB+ttOR0zmofkwtKTB5Zy30BBAWfpd4/bLcim9PzmV6WmCeux70DB+zJJt5qr/e5v1NLxe1DWdZ4
64T9XIx4DuWCvTs4woE96RsAaWu8lbzjKZEL87ou1tOotLu9npIhekC245J11vQqJ/0D7MZUXXu6
VufNh3RNehHcqKpAT7SfBk44LVkI4tWx7GZvf+h8SGmC8FFrvfkNAsWptJ3OlD2K7ubD6UBK2FG2
SeeUhXk55st0/AF2GbuUhT4VkCh84nBN85P2/FZrClcG7ADYZ5/nZaTw8tYzjMN99Lo0FTPYI63f
AMbI6ya5OOBz759778HbLgT4XEtH6jEUuahmuJx2w41s7a2rcyhEUoDjQNeULX8FDNr5pd+rF+le
wgckUmbNN02ukW2ld+7DA/tE8Dv3fCuP5D5Za7Dz/eUo6fVRqJBVHvBh3h2YpR5Qkp7k3nsE6rnr
enl09scGgEMQb92MsM/l2hwraFxkQCWUz6D7PZOdlPSAR0Os6ta19GVCvalEeU3mxR1A4a7NXC8E
8to7YN/tTUI5IJ/0Mkq8b+PddiXB3nIp9PMhmbJdAvOS9Nn0cXqAlA5Eim5csg5Q77Ofwpvm8Oc/
+bifyaobRdebxZvOp79WvQhuGFWgPyD5aznzEB9TCuS5OV/UWu+64Y8NRBhaY9b7Euu82Qa7YTK/
5dCwk6j2cKz0YjjHXXKGVSwYE9d5d+U06HTbgYjrRIbyxYFFJC8h8Fx5kMM7NwPuA3nyQF76xxgl
AL9p/Wv3U79UZN93AuCtmA3R2050wTNoYZW5o4t2Mk5R0elo67zbs6fjiW6AOpoP/b2BdGj1d848
VzfSvRQqD3I6dkJDVnoK9UAe2JsgbiGYDwG2Yykz4r4dF5hKdFB1RgzwKMw7da3ZT5NZwVpvbY8A
vHes+w39/+y9ffRux1Ue9sz7u9fX0gIJCLLNh0m8wC4fgYCEiSEUCixwTLxoE9NgrTQEs5IVUtcQ
CVZoqB2+EweoTQEBbUrrWE20WmwgJOAPIJRGyJXBIsRddhCQYCi1DAYk2dKV7/297/SPc+acPXv2
ntkz55z343fPs9aVfu/5mNnna2ae/ezZo5F48eTufxGxZ5EVyikAUnJfIvERKp5bqGdHIhk2EilW
rleagpOzJfHJSREM4UBDP8aJ/NlTfRu9AKHPhtULfUFueUpL1RH5Jm2LRuy1CnLEXrXNpftD+1pD
5od206drz0f5Qq5vhiXrhjn0Tv9WVlwcsFB7iqNWvNl8+mcCeKNzbiD3a4K8ebESeoopg7MJ85qt
RJ52PnTplZJan7VTWLZFROhABiKOSJ2P5rWG4yOPsx/UvEDao9D7rYuXsSvAdF+1gRC7rvH4xhdA
Iu+ZUNwisUehk56JRGjZlTVyLyGryCcVyucO4NNWhLKsJN9SH0+6GE1ByKy9XHpPfCahXo7Il97p
a09exk1hAEy+tyghnWZTIPU7H11nomTxTPcCMeaZ7l0/v95jJO2hzK6pGJexo8tqmog9bftKZF7b
ppB7sX2YQu5bUCBznMyn7zCi8NvIhyIQe+kdqyHwybm8nTPcs4QnZYh9dAL7O3kdDO2iNqXDAnrO
TlLtlftGjxmOzfVN/J1QHFLFZfMKRH6ROfQWNR5EvS/4Na2Piar1OWKvIkPso8MENZ46SYf+ib+/
GTI/fONhTBUiHMn3Owgq0XLAvUK/j3ZqxaFBQ+0B4CkAv4rTUrz5dIE1Qd7MWAk9QW5APSkZF91H
PdhJR56eSInyuHZ73PHk1Hpqj3gNGfWe2lBS50UyP3Tk3Fvdq/K9nYONUoI8DZWdmIncN8I6j5YP
XiyEpsqOygEaf08lgq/dqyKRt9hfIviZeikkkg/EAy0gT+CBmMTP1RYM5VmIfKbOR7a3wl+9BEcy
UyfqvFo5+m/KYbPzQG+LP3PDtyqquIGwe9/52bRM970tkkqvk8iY2BcdbgYyT9sotTyFKB4DuW8h
88OhZHUDHqxBWb70nGtzx0TzhM0n6ZuyxB4YQtkTwmxsI6PcEmGJzYkjH1G1V+bFbwpRIUmixZyj
l25X+rSiIl/bt7Q6kA1Efs66pTnwnNhb61MjQQQ1nvY/Q/sTxmEWMs+n3fTtMU8AGo3DfD+u2rno
Hs2ZB2jFUSFR5r33p0aGT226wMlhJfQE1vl5tfudGoYWXLwKkQcGr20C0vHk1HrNxiK5j+wjA3hF
nafKXxRiT+qlxw8hcgN5lxPkqcjd78KgXiX3rQOXSlQR+8z5HFPNtxD88W+ByE81wEDwpXW/VUcA
D+nPEHggzpidexaltiD6wicS+YB3PPEcsX6qztP7EC27R75bKZnS8J3tfJfwjma6B+DOPdzG6Znu
d11hkUpP1X2HQaUfE/L56P8hRXly3y2qfG+L82l5YpkSSgSrkdxbiUtUJBnAiyH2pFy36+Z4hyXI
uFo/FGdU6zVopCb77loJNy+aEfuhvSSjlVIYu0Ti+XYrklUBaD3kXojE3YJCu0f7f3otGrkP7ViR
yC/R3ynfUZHIN0ZM5CJOhpw8SO9puXDZNkmNj/vF/rheeLGQ+dC+hHB758e156O2ejs6YOn22FG9
MvqLhFMNtVewJshbGCuhN6LW86mReK5UJ+exzjdZ15WrWZ6M1TS1XrLP6NWtUeelEPuhHJo4L3TE
nnANR45zXWfsMp1T1ubhfiskITqWlLnw+nXBrsjxgZTY030i1GyKdS9pdg6q9Ju+x0YiXwxbzMCc
pE55z2sI/JyInHZGIp973tvwYrI16E3qPLVDIvXBQRe+cyHTfbR6hZDpfnPdY3vFRSo9Np2nji9j
NxZdSeyDPQKGENXo+yEHTyD3ofzofPrcDMXlMEWVp07WIXP8dURqfahjOE1Q6zWoJD57krJdIL47
KUcDPTwQe+FcSbWfk8RTXLra/f/8pnGbqPBXEOSakPsNabd25EZq5L5E5JM54jOCv6/VinxVZW5Q
4zVizzPWp+ErZYgh9WBjIlLnUL6RzA+Oq5DBnjhpaXswhNuH73fjlrmvK44CPZl/GMAtZPMphtoH
rAnyFsZK6CkaFWEgVQw0Ep8jKTy8PjmfLNMirucMiGp9CVly7xENHDV13vmRZ2pkPg3LZqH3PEFe
bq5yZp7yUC8j9oCd3E+F2NGS+zPa4xJi3/2o76mndu5Fgl+hxmfnaSrJ1iyoeUaOXcBSBD6BRjxz
RL5wP69uL2Nz7qI2Ipk7T8qgA88hx0BPqp33QzjnUP3gDHSDWhQy3aNP0uS43YMCH0JbiUrPQu/p
VBp6bonYS9dGIZH55BiF3JuRab9nDXE1knlxmlMgvwqxz6n1kQm1JD733kbKJtncf4fDCnUZYs/L
ocdudohU+xoSX5UnoL/fgdgDI7mPiH1N5EamLeK2n9FQ/isGcl8g8uNJxg6jYZK2mciH776hbe7y
R/TjIYXYJ2H4ww5qQ74eicTTv0WRxjkzmd9sM2vPUzuuj9OTQkTOvqIKVxwEb0BM5oHTDLUHgCFB
Hok6eJtzbk2ONyNWQk9hHJyIuxn5tJL46FifdmzdAJ4ybrL2KlJiL6n1kYpReOJ80F4zd14LsR/P
Jcw1qHZ9WFwUet8nyMt38pmHxXi8NM1hrnnrQAORFu4ZAICvlKDVp7xPflNniGPkht8TybmTLS8a
2OjnmEl+JuTfsr2WwEeq2RSipjlGckS+UN+j12+Gu545SHk2/qxT+EIIvpTpnqq2lDR2SrtQpkOS
6Z6r9KGd4Anyen/BWFeJ2CsPIhpM078FkhudR9rS5D5YkOEFk+DZtdSQ+ci50v2PE/tc0rx9kHjx
UEbsAZnca+XzY2uU+LPMOvBq1eSZSKp9a1k1sJD7MpFvq9uCWiKfm85gAY96KxJ7HrW1o4MmrQ7l
76ENT0+sIfMA1LXnaZu2OUfUPuwuxduk615xmuhJ72cLuy5CmPqaHG8hrISeIJsIq3Qu/23osIvh
9UDUWVBvc47UB3vERFdMmcgSfD92QiV13m0BUJVAIfMjccdA6kPoPU+Qt8lkGt9lRp6i4p2Q+wyx
XwiDXYo6UArVnt8g9rOUnEkrRiKsufffSPITB4OR4FtRjJZRYI4mqCDy0r1+8D3PGf5++NHbcHa1
J73h22J2UAfG9ml8X0zqAfRqfUywfV8+zXQvJQuMMt0DSFR6h0Slp999dAsyxF7CMNAFdDIP4fec
avpcoI6UWlUewjaB2GfVenpOwcbhcGojD6QQyuJ9zOapfntYf3s7/s6F4ydoeJ7FJeQESP2opNq3
9iVmhz+zRyP34nlSG2RFRbRYbpocLYdHSCTL/obyzMum2og9vxaR4HcFkG3k7+ie5m0zk3m6L5B5
1p65XugYcoXsMIzJ9hZ5tmKfeAOAp7Ntj+NihKmvyfEWwkroCbKdR0k9tfQ7vAPwbICOPJEfVJwN
6bAMan1WBc6o91XqfD+3ll7PeFzh5vSh9iEkN/zODXQ2WYW+vzc0JDG6sJTYz43cu6ROcTCOm7QB
2WQfQAXBlwaLpRwArUkONftqCX5pv/VZFN+YBiJvCTkdBnmODYg9GeSFY/n0CNeR8OgZ0iXsBJUe
ACHMmUz3vepOVXqc9btYRAB1IERT9fl9Ue4yD7GP28ZSO8MLyx9uwsSPrjnEPmcHU9xzan2xLOh2
AUq4POtHuNmDKhv2Z4h9UsdEEp+dCqRAWsu8pRyKaF48mz4mTU2L9rOb4B0j90I7ps0xnxulZLva
VAdNod+cZxz3gsPfTOwF2+h5UZlGEh8lARTIfFd+Subd1g9t+xBuz775YTk7st07YON93Hcs8ExX
HATPZr+fAvDcCxKavibHWwgroafIDeJLa09LhL8wwCyG1wMpEQCiUFiTWh8NEJhN1O4kDwBGRc0j
q84PKn10bnr9obPiKr3ve6ghssD79HzHyEQJ1CmikPtoPnvjAD83X160hxEcqf69hM5pdTCVb9gs
kHiAEISCzZrSpJqXq1+wr5bgt65qUXw0MxL5bX/CB5+6Eh0vrW0tYQiLp7a5cR+fSy9muvfIZrof
EzmNKj3gxGXsumR6bjAl3JPh9mT8bCKZHwhL/QdDIxUOhpYQewrfkaSBEEnEnqj1wDj/Vis7R+D5
fn9FNCmP/gBO7AFEU60Gok/byMgQvQqJxNcZma9HVO0rlNIz+v0+LTaGE/y08gLBp/ty/ZAVrX2R
gciXkAvFP7sGNSGqRuyLqLw3ah8jkHm361ekEMj80LYpYwCq4o/CCnHGrrho+Ej2+9oFIfPAmhxv
MRyc0Dvn/j6AvwzgkwFcBfAAgG/23j9MjrkC4DUAvgrAFQBvAfBf0xfcOfdsAD8K4D8D8AEArwfw
33rvzT70rKpYaueNHUHWU66o8mI54apMaj0pN0kKRSpIws/I/4dOJD3NbeNs2xqZj8r2KakXQ+8j
S6Wbp3vUY8XVQO735d2m90Yh9y2oPV+fi193bjrgtzm/lib4/HWZK4FZeX6o8HeDIk9x7fxsWAEC
wDj304/fTrwKAfuW++8tkPhgW7Tc5NbDnYUQeyKfg5RNm4s+073befhLLlbpd2wZOzcm9AtJpiJi
H1cnkkiJzLcQ+QHh2shF7Zvcz6HKux2bi6x9FyViz2zQbKTf0aQmixF7IFbtB2JPl6vTyD0MJL4R
kX08+oDci5pl6+i3enaNvXOM4BfbyQLBl44B6tsgC3gel1Yib6ssdf5wcGK/uS6/GDVz+FUS74gD
oYLMI4x9MuH2Z9c7Ah/Iv9+MXtCWKL8VR48/RpwQ748PZcjcCMnxDm3HRcTBCT26B/uD6JZiuATg
HwF4q3PuU7z3YYba9wN4ETpPzuMA7gFJpOCc2wD4WXTenheg8/zcC+AagFdOMa46dNhYRrRdIvJA
qpxIfNaq1g8bWee/y/QGYVzfd5w5dd5vSIh8bpDtEYeUMdui0HsG8VlkejCR2DP7snP+jLDaGilb
A6mSyX2+Qu2a665FHfTW5H8Iv8kAdXNt/Ht3WXC4CIRfXIJHDN8kfxYIfu13GoWj5waipXIjL1Fc
dusg+tqTl3FTsElrT7SyidMskPrhHKLSu52HlOm+C7f3g/MgTK+hihJX6f0G0TJ2fCkmICb2wyVx
Yk8JPJ07L5D5kMHfCinXxkHJvZXMR4P9/vH256jE3qV/D8Reisyi1RUiYSzI3snISdTbRvcro5R9
OWDpdxUl8ONT1Bq/bd4eJgSft7Xs/PxUJoHI19436fjCp2El8kMURms+lPDNWIm9sj9nb/Z9JySe
LhNcQ+aDs5KuPR9FH21Ju82+fWCdQ3/R0CfE+yi2+f89hC1Lg2S8H9T6CxSJsHccnNB777+c/nbO
fQ2APwBwB4D7nXO3APhaAC/13v9Sf8zLALzbOfc53vu3A3ghOoX/i7z37wfwTufcqwC82jn3bd77
Wf3EMrEsH5McZAivV8ui+wtqfRZZ9X7sREzqfMVg2hJ6b0H2PgtKf1bdaBzAt4Q1ivPTJ0R5AA3O
eaU+693XSDy9j5IiYiX5sqdEMERx3JS+m1yI/qSQe7oe9kQiDwCPbG+Fv3oJ7jrZyNR5gL1TSVuD
5N6FUO1oLj1iYqlGcTiMme57Z0BRpU/mSAQ75DB8agv4IFcg863QVF8LuZ8UIdDXEmzIPj9h+zig
JwkPG4h9icAD8beikSKOqCorYRveCbnu4ISYihpSe0basC25qoTnzeRgKCaCKxF8xZBZHSBKWbVE
3ivvYKs9RWKvtMPZd1pxNPuNi0i8aI9A5gFEZB4YHZ6SOj/cm95Rurnez8O/DlX8WHHS4MvVXZRk
eBLWjPcz4uCEXsBHoGvSQojJHejs/IVwgPf+N5xzvwvgcwG8HZ0q/86ezAe8BcCPAPg0AL++tNEW
Al9U6kvneJ8lnTm13oRkznpnj0Wdt5DRyGFBBuxq6L3SkVrriU5nBKGUvGcqclMrRJ7KBzR76KT1
kHvbgFAj8XQAfibO9baRfNk4yTDjqTNlyS8NjGkSLyuRz5X5jieekxzH1ejNFvjlH/9GtYzP+Ruv
6b5ljO/fZtsR6UFR779r39sdMt1310Ecf4RIdpnue8W+oNJj45XpMEoYPvlDC7GfW0WfQu6n1Fej
ytPzAkKSNYnYZ0k9dEKgkficYqmS+NpvjdirJlJrIPct33xcv0zugZj4F5F5HhxiZJdWllBe0rZU
dnM1joAcuYzeIdYfu3O5ktAeiHYxR2RXcL9PIfbau5trp+N3XyfxVCnfnaVkfjju3EdkHhiT4Q3P
hlzH2XUfRVa5Hbpn7FZ1/qJBWa7u/RdYtV4z3s+IoyL0zjmHLrz+fu/9u/rNz0KXEOJxdvj7+n3h
mPcJ+8O+xQm91EkmHWGyVr0+GTEh8kZIan0ratT5aG6uVBbzPJtC75PtUsGFOqVTHfLEvhLmpEv9
83dkEHJI77o6n1Ah18nxComPB+PjjzA4kwchqS3uSmqH7IiRyrMTeHl6RPu347b2h1rKlbENF8HW
oB/WLC6cH/D2f3o3/vxXv6YjViHM9RzYnHXEaAi3pwQ6sVXOdO/6U4sqvRABwXOUcGI/YAEyX0pC
aSH3U9Gqyg+73Dh3229kYm9S60N5lSSeF6GR+DMSXVKdjNBA7gGd4EfvXWh7Kh4hdWZtpBTyBtuy
qHBcF8k9PbZE5K33oMrRLbTZTI0HBIdUjTOE1sbmyI+V9vsZsfdKwsGcQq+q8GB9GYt25GS++xZT
Mt/tRBxuT87xLggp4/93lzCIKisuBnoy/zDS5eouchb4NeP9jDgqQg/ghwF8KsYQjByCrlvCfKOv
DERSZyUFmQEbn1/v4WEJDadqfTbkvFSOUZ0Pyp5YBiEegyIf7MmF3luQu+9CGDE/Jbode3lTEL0X
c5L7uZwD6uCKvXcaiVfLZSoMJW1WpaEmp4VE4LNq076eP8pEnuLq9jI25y4m8OTczRZ48N67i+U8
+Pq78ef/+mviOcFEpR9DO1mm+23X5mQz3buySg9qvxYxw4l9IKZLKPOKP7WG3JfU1WLGcrFQebtE
5v0ZukwxGJ8fJ/ZiGD4tp0GJH2zi35hC4sVzjcReu/ecQFNCL5J4kHamitDTXxq5r3MCZom5sRit
jKmK/FxIwuob7MgR/SGZ586biX2OnJtt0kg8MfVcIPNuK5N5v0E2GV4yLWEg994+Cl5xCuCh9kC3
XN1FDbcH1oz3s+JoCL1z7ocAfDmA/9R7T700jwB4mnPuFqbSPwOjCv8IgOezIp/Z/58r9xHuuusu
3HrrrQCA/+ftv9UV/LGfiWd87GfGB5bWoW9R9NgpmiqfhKoLy59JsBCa0trgVnVerJ+RFnGgkQu9
b0U4lz4TC7lvrHMSkZ6T3NeSnMopBpaBZ3IPhbWC0ZOcHMEH5IGPBNNUFgm5728hWPNkPPieMcz+
0es3w12XB6pSWTk8eG9H6oc6iUoPQMx0381/lwnL5twDu57o9w4/VaWnkTHDhcfXz4l9DZEPzsbx
2MLNoM4FPVgqeTb09zYTEmyCgcjzOod10c+AEHa7OxvP4cQ+O78+lIPxnCqwb1Qi8SUVsUaxVx0r
MJB46SQDouevkPvuQHuZEQEvtXPJhablReQ+PM+Z2rOadnGXCaufq46h/PD/GmLf8vz5+yuMzbyL
2xuJzCd5MjA+61wyvKiufkzm+x2hTf2j//gQ/uh3fg3bKw5f8Vu/CAB47LHH6i92xSEhhZv/6gUO
t18z3s+MoyD0PZn/zwF8off+d9nud6BbIf1LAPxkf/zzAHwCuiXuAOBtAL7FOffRZB79lwF4DMC7
kMFrX/ta3H777QCAL/zy7xl3MCWpibAbYSLyZFuUhM5C4iLWqtedJBWrUOeH7Ni0XGa325HBv6DS
c0yJLOhOIn8byH3zM25R4aRTFBuz5UTb6+yvnU8qDfYTssMIvKioKySfE/ykLiPBV1Ei8DPmUODQ
vmVL3Q8/ehvOrvaDVj4f06jOU4Tj/8JXft/Yfgxk0I/fsifJnEime3jEme6BVKXfAFylxxl5hpuo
Wp3YE9SQeVrWeL68PzqMkXu+P3lnlpgyo70jIKp8X7fbInpOErHf9Bs0Yr8V1pLPgdpkIfCJI4j3
rUO5dmLP7eD1JhE/3NnR+Nx0cg9zskCO6twepXfQ2IyZyW1Fs1iaxjJHHWoRFmLfVHDapw1OtULx
PAleVGwmqoIKKbTvGJasI9ice9z27M/Cbc/+LDzxrA1++rV3AQAeeugh3HHHHXkDVxwTePj5RU6G
t2IBHJzQO+d+GMCdAL4CwBP9PBIAeMx7/5T3/nHn3I8BeI1z7k/QrTH/AwB+2Xv/K/2xb0VH3O91
zn0zgI8B8J0Afsh7Xwj8I7ZI5EM80FpiZX0hAZUw+KcGDXzeqNbnVA1kOuBWdV4l88A4wA+dYuAO
Qug9DZ2viiyQYCD3c4ITW8D+HOQD8mV35dddS+08cal88ToL5ZpJvlQMfV8rla0SgY/Km/sbl0ia
tlxlj63kIHMdEXOBxE0YCIcyN0GN36X2dHk0ZCdNyHQPj0SlBzqnHVXpAYfNdY/dZWcn9gh1yQ9k
aFfI/cleb3RubENkB7Elp95PJiLU3sz7ylV5gBHn0FQ2EPuiicyuS08Wjjd+//yZj/sZsY88Knq5
4rQddg/GY2tYqnCsE5bZnJipnaq1FDzsXFOfrRjzCFiZf0XZrbbMiCyxn8Fpa+1mR8eZH8j4aOP4
9ya0oWTcRJPh0XM4yXc74G336clQV5wUkvDzi6zOc6xL2E3HwQk9gK9D14z9n2z7ywC8vv/7LnT+
8DcAuALgzQBeHg703u+ccy9Gl9X+AQBPAHgdgG+datysJF8jjxYiT+yhAkeyZJw08KUZqpmaLA5g
w88qdV4nL8kAb+fje0EG4hHRp8cHhM5aGOzSOqXtUX1S2Y19vUiwRWcMeQ7IPAetzFzZue0aKkP0
czZpS9hZFZvSvOWhPPHkYEPh/ByBZ7/5POOSXep+L23z6nESPvhUJ6GG73ETVg7w9epvjAhTAAAg
AElEQVR8hP7bdeduTJQnZLoPUDPdh21Epe82AJFK38/ND7AQe7Mq3/DtimSigtzPigKZT1T5Qjk1
xJ6jROCnJIwEiP1hjnW0k9rBiD0/OPcsFCIfIK++UQGJGE5MYmJ2sLD7b1GhpXdmH1OMNAxjCzI9
RDxugpNEIva1Tm/A9lg1Z6GaBA9hP8T+gCbDi8rdxkvWVTmmVhw11vDzdQm7qTg4ofe+HDjrvf8Q
gFf0/7Rjfg/Ai2c0TYVINKRGn3e0SuihdVDqtj5Sq6kgZQ7D5yHRZFCXkMoKdd75dIwjkflBvYMh
9J56LgIK5J7XPxRlIfetaCmDk8sCwbegWh1RBubWMEWNxGuh5LzcIjGW7BNsU4upIPDSb7XY0u1h
Di1ui+j4ytR97fysGwAP352fTuYB3P8T34TPf8n3xQNDlul+DMEP24RM9+GUgkoPdPcuyZ2QI/YK
BjLP7rVGDsUywsEaQS6R+zmQIfIAIfOSKm8od5hfT7ZxYs9JTpHAW0m1guRdp8nThDaf5lFQyT37
rX5PBaI/BTXtr2n5OWu9nOCTPj1H5KeR5fpzqC2cyGtL7GpLqNZglrB7htx0DwDAJZ3Mh2umZD5u
x7w4/WyYg99PC5oaobVixZFhXcJuIqoIvXPuUwC8FJ3X5E8DuBnAHwL4NXTrvr+xJ983HqROg5NZ
TqZDQiULkZe2+3T8I4XhZ8PVM+p9ONeizksDU4mQhbVjLaH34jy4ArnXMHk+vrHsZiTekKWkQFqn
vFlV4JhJKomnp9DtBWUpVdSNtinPvpbARw4KnoSOoLT+tVWNtw6orz15GTeFjN7cCTgV/TfszlkY
9pDp3nWhxIVM9wEllR4Yvz+u1kf1q4oduZ/8XlAPJ5C+P8ojzc2LHqCR+wJKeSFoIshciL1K5P14
nrakY6TW99uA8V6bFPip75vk5LIQe8BG7lEm8fT8qoz0xr6jLnJBcEwakn6a1GLlXZGcixZIddY4
A7gaD1Q4BGbuB5tC/IXcCLycZJpEBZl3nrVnYZyl3PfdJWCz85GTd8VpYg0zj7AuYTcRJkLvnLsd
wPegC4f4ZQAPoktQdxXARwH4swC+G8APOue+B8D3nySxV+bKmSCR11I4lKFfjcqQzGN8WgzDV86J
THGQM5LTwVBQ5+ncepAOif4WyXxXhtt0pD4JvVdspGWZyH0B6gCptXM01m2O7FCOLZY/V+JGpRg+
l7M6IoAf3xA6aib5qCPwNdgUsnLsLguOjuh9ttf1yPZW+KuX4GidMw7i7v+Jb8Jf+Mrvi7+3ykz3
8B4urM6eUel3l91A9GmkT5bYE0gh9lKyKRWW9raS3JcIuxVNIfa+j6wIxABjFAQ/DsgRe+Ua5iTx
5HleerL7cX5zfyNzxJ6dGycSY+2Hot5Hx4VrblwzfjbHsNA3ayr11Po5kZ/cbhvrNhH5Ce8YXZpx
VnDfuvD9FR0SFWSev6tauH2YYz+U69rf4xVHgzXMfMS6hN1EWBX6NwL4XgBf6b1/VDvIOfe5AL4B
wDcC+IfTzTsCGEl+kbxXQiLySUgpPZ4Qex6Gb5lzp81hltT5sARLpM5v/RBxoJF5DOuwju6HnEov
XqeB3Ndk2l1SuU8g3WP10Pr3yTIgjOqoDJuce85lieCfXctXuH1aPsLFTOCFzWeZ9Y9L8JdSYpEj
8bl973jiOeNx4XNzbj7nDUK74nvS7kiYvUsGm5588yHTfVKeotJvrvvhm+XEXgrDj8uMQ+y1fB2Y
8Xu2kPvWzOZSHcXEd8OB3f82W9YhtBJ74ZhmVDiurMSewkoOJRLfbY//bwI9Nte31JQpRlPU3fxS
36US+XAPjGMW8RvX2lwhrB7Qn1Ux300GG2NOiCI4gWe/LWvXl971iMzTenzcbg3J8PooSF4HX7Ju
avuz4uBYw8x7rDkEpsNK6J9nyRbvvX8bgLc55wpBqSeOJZe3Uoi8+Xwyxhu6N95hCWRAVEfpQIiq
82yANM7Jkw0dyDwpa6izV+lFUq9co3Qd1P7cQGdfiYDM9WgDo4Y6fS3JWyKsnw9CCFHzhTW7awfI
JcIvQjhFSiw0ad6lgdREES7Kc9uGUSAL/3dknvscuP+NnUoP75NM950jjxB5YByEAlE0UEmlj/JE
hPPDILx/BpzYuyujLZoqT4lKdL8ZiUgJbiiwThml5N41zCWOTAj/r1Xl6cnJMXXEfjvlGpK+JT2E
f9fhGYfpBkViL1V7JtS1id9Rqf6WkGsnF5s00lV9S03IOu3nLIkMC0S+FjVkO7vyiPJMmjFMn+gM
NBF7od603+EbZKdlttzcPfM+IvPUYTCu9MMKDJ8865fWDPcnjzXMXMA6FaENJkJfs/Rby/EXHdbO
y7GBmtjR7GCeLz4Qex6aKHZQqZGaOh9sGQbW2eznwn6PIfQefYI86ZpqvPeR/U4m97ysucj9vpwE
RRzCjuLcf0IQGVmrJvhqyXZkswJHRLz9ZmqREhYSz3F1exmbc1cXWt4APsVFiwaSMt1L8z0lld5D
CJMtEPthnj5xIuRUeW5DBL7e99BOym2HBpE8NWKyKo9MO2kk9rUoXb/VMWcl9mcfSgs4vyl9Trm2
nd/T1u9bJfew9VNSOVXHku9PI/clIt8UpVAJicjn6tP2ZXMWsO/XQuzFegpRmKJTtuLe8VB7TubH
5+FJu8fM2Y3v8HD+AokdV+wda5i5jHUqQgOastw7554P4IsAPANsmOS9n5Z6+UQxpXM0Eflw7I4M
QCsSwcXb0x0iyQ/2cHWeht3mOmmuzLMl4kqh98P9oHZaBk0Z5W2vIfZJ3cJ930fyOw21VVunn1hz
CpQIfmmgVaimaEdu91zOngYS/+B7xjD7R6/fDMcT9C0wEL//jd+EL3vBd/Tl+7QtKmS6D8dkVfrN
2NaFTqNE7C2q/LBt5+XIimHqEdsutS+V5F5c+7wC5xNV+ShyIRxeS+wLyJJjMXQ8fz5HlEPhkkuI
PXXChffl0tW00OsfJrRF7J5eekq+j1kY16GvU7KnOxQ0cl8i8sP1mG2wX1gtkS/WbDi3itjnoivJ
9+7jRqHavsFRJ5B577rpAgOZH8ZZsnN0qCskxLvetTvrknWnjzXMXMU6FaEB1YTeOfctAL4LwG8A
eB/i5m5tYXJIBkYFIg+wgTT67O/hNF8k9QDS8D4hxFIim52NSNV5aut29CjTEiQyPzgDQvb9Uuh9
AB1j082lS6fXVCD3vJ59QbvvByX6Ggprgk8u3pJgbEriygoTJw1CG0i8hocfvW0sayJ5tIJnuu/+
hp7p3iNOxknKoSp9Nwe032kk9hZVPtzjzbm8AoH2BKQ1sGvJ/WSVLKfKd0ZkVfku2qn/8ywqspnY
F9Vt6hQ1nM8PzEVeUdU+EHta3oYUlAutpjYPJB7AbLluprRDM0El99rqIQtOFRwrF+rNHj+fTSVi
DwjvnEbihXJrISXBC+O3hMwXkuG5Xd/WOIzOv3W0veLiYp2K0IAWhf4bAHyt9/51M9ty8aAR+IBL
bNAanSuQZwarWp+E3EshwRLJ36JJnVfJPJgH2iMOvU9CYkMnSD3sZD+9ponk/iKgduAxdTxgWd5q
SlRCMeR+eI80mcRUzexoIfGcGG7J23121Q1RK5vzac6GLHYY2oEo033vhNMy3Tvvx2eaUem7RE/9
OcOF5ol9TpWnRB5An0wzJTbq/RrKIZsmkPsW5Ig8kCHzxPbRSdKfWknsSwReOqa0vauHHCdN6TIQ
e4A9E9J3bdiNcdtxn0biWxww0jVOje7KJrFrgORkKRF5NWTfYNdk8suj6PZA7AEkUWCWkP4stHvb
tz3ZJHjDg+q+uc15186KZfZRU5267wdiv+L0sM4PN2GditCAFkK/Q7d03YXDpI5bctjzded5wrAC
kU+OCYRamN9lDcOX7ABkkh+y2VvUeWqLRuYDgfcOpCMfB3u+zw8wkP6+PEeSHVnIfXRdYkjozIPz
JQjWHsiopohrc9vnWp/aTPILZc89nom+tUnKunGgbCAXH3zqCkIeCwDYXJtglsUmMdO9j/ezTPfo
laOSSs+n3AAGYj/UC3CCSIk80N0jfxYfQ6Epl/Soucl92bkjv/dSiH3UjuaSz9US+63yvWsk3ppp
niVYHBC2M4eLhSTSe8+nfFhIfKkuK0pRCMOmTPh0cqy2Dj0/tmB7icgP+yfcAzXUPHeS1O8Gp31L
EtLBKSDvlnJklJzJ8Xir3Afk2vCBzEvz5snSc0G176IcWZ39tx4y3Id33jusS9adLtb54QWEqQjE
+fE259zq/CighdC/FsDLAfzdmW05PCRSLh0medeFUL6EuBTn9CoDoNBv+Y7pDo2+RuwbOkeJZI3h
7wZ13vtkaaeEzHvfl9VfAwu9d7sxQZ7zo03U2WAh9+PBqZ1qpmtgHnJ/AaCGvi/oZNCmHmQx0Z4l
k0Jl603mcuuGXDs/68h8CM3e+WWdPUKm++7/eqb7YX9BpaffI43SAXRiT5PiASNJjFT5gso92Kit
wpH6GfrjyXaF3M8axhx4rkGVp9uHqUt8moKV2PN6pO0accnc4+L3VUnseZmc/GhLpk39zqMVVCod
fTV1qytipByPHcB+K325XsDCkHLaVKwwUS6///+EqAGJyG+23d85Z4NWdpbME3uDuLE592K4fSDz
m20c4QRgXbLudLHOD7djdX5UoIXQfx+An3HO/TaAdwGIMtp77//KHIYdDYwkXyQk1s7fQuQBYIdu
mBzWciakeDhfCsPPmZHpT0Nnw9V555Go80mxEpkPf289cObKoff9/aNLQ5nI/bAxvc5oQGgYCM2J
mvmbpQzwNwLMIfczlRcXXld2se4Mic/Zde3Jy7jCbHIeePDeZXOP1mS6d95332VJpQ+5MgC40IYV
iH2OyHe2jPVsth47oUEbBr6SE3IwMg0/t5D7osppfY98nSpPo6KGlQPCoZXEvpbER+2YsnJAhAWI
vVRXDYlvnxetk3vzcrCo6wui+3cmJONLCid/HhORzyj0riFcQHMOWosSRQmMRH44Jnff1DGDG8i8
mASPkPlB5BDWns/VsS5Zd7JY54fbsTo/KtBC6H8AXYb7XwTwR9h/N3F4SFfcEqKrhdcLRD46jgyA
imo95IHGYEJOzXdjp5Rdps7Hg3+NzEcK3pZ0aOE4Fno/gKo+C5H7vUBkR3u24QJhtueXmd87FREp
MpL4gEe2t8JfvRTNw1zyfXnr2/9Bmuk+CpOn9qeZ7ksqPWh0gZHYcyIPpKo8JcJS5meJ5Hdl0zJn
IPctCE25UZVPpjeRFQmaiT2tUwrl51PH6DFiCJuwzYJGYj+YslR7riTAS6ZwKEu/TkX0HtJnwZ/v
cFDhdy1qotgypF1U6GdADbHPTXNMiDzbXwVC5qUkeJTMD795Pb0T7+x6943vLoUEe7qzbcVJYJ0f
bsfq/KhAC6H/G+jmMfzM3MYcGpLX3Jo4xlR+KeSTE3kgIvODEhbUqT4zfHeqTuxbbOoK9YyQxB1d
qpCwzpCRearSA4bQewkGco9SKJpA7qfAHC4uHjfP9AgAkJYezNtTXfXpQxp0z7z8TxOJF/a944nn
dOddp4NWNwtJKEG8BiXTPTzgUFbpIzXXSOw5kQdGMs8V7SSiwLHj+DWSwbVK7gHwJcrmvPu1qnxE
5DkUYg90/ZhK7LnDOEfgwe9VaoaEaFpEIMLae8yI/ayYK9Q7c/9t57dVWyL3ljom5RHg183vYy6s
PjMVQO3vK1Ct2JeI/ERb6Nx4EDIPRuY3W2CrtetDnotRUFmXrDttrEvVVWF1flSghdD/MYDfntuQ
Y4BIlIT5mebssIWBdw2RjxD20azOmTB869qrya6hMyKKu6LOd3bS7QqZp/sLoffFgYdG7muI7aH7
RTVxUZsqUIOWMMdimQtPW6jCHsi7iAkkPmAbmBJbg94RRXYRCJnuASCX6d7tyNJ0GZVeJGglYh8O
k1R51u64kP15OIeTDV53XyfZwcl9Vw5l2zOTe86PyMBfVOVzpIg7QwKxP3PRPHtO7LkROQJfAzGK
gmJfxJ5+j8O1THtykiND+tanJt/rConfP6m+iNxrSyxy++a2DWDO8jKRp8nh5oJK7JktcxB57bwd
IfNSErzEmaCo89033/3zrrul65J1K24UUOdHnyDvjc65dXUABS2E/tsAfLtz7mXe+ydntuewEBpJ
J22UlvVpCrnvz1WIfLLP90pY6Cjp4LsQhq/boNudEnS2j9lGYzglMp+QqULovWSHidzTNXkrIiyW
JqOLDfhaoV1vhXqlOpuGOujB5mJnwRTyPiWksTg4NJp1dXsZm/Pxu9gX0kz3YE6KNNO93zibSq9B
I/YZVT4i8tJ1FLLcjzvItTFyz+3m6j3PtN4KSZUXw+sllKIcSsR+JgIPGEi8BGV61SSIJH4+WMuc
RfHVQtU1cm+0YcpUBvXcTFh9NH4I73iIylugb5AEgU14PyuIfMszjMh8OD+Q+V6dzzkUaDI82g5w
p+WK48e6TN1sWBPkFdBC6L8ewCcCeJ9z7neQJsW7fQa7jgdWkt+AHBniRH48xg9qtHcuUusByGH4
UyCo81LYfSAB1PaEzHNzeidAFyrrwUPvpfBIE7kn4KRuzikU1ZCUCqV3PqgDviZqIOeASsrlBRbs
2ENouYbNeTsL2PJMYUD2gdJ3+sH3PGf4+9HrN8NdZ0mw9nFLvB9Jokf3jM8Yke9toaTapNIX6+7/
H+rPhNfHUQL9H9KtH4i5pX47ue8PmoQkSouQ+SKRxzinFkATse9razU/WWFAIvFJxEO4l9JllVT7
HGpIfM0lH7QxhkriS3PbZ8spILwqatkWNR6EyMef9SKIxgwtinwLoadkvh/naGRer7ffvyP5i9Ar
9drKHiuOESsRnQdrgrwCWgj9T81uxbHAGpo+d+9jIfLoB80OSfZ3Tux5GH6zQkFVuISMs4H91kdz
1zUyHw32+3LcFmLovVjXBHI/2BLO4eS+9T5J4ZeGgS2QKn6TbFl6GRtx7WvjoARIB521BP8Uodye
6HtSyMvDj942Hn+AAVyOAEfv965z5RVV+oZBtETkqW3DN5UJz7YM3sX2o0DuTSh+H3EbXQqvD9hI
a1ArxH6oRiD2TaBhy1KbUJiTH9mEArGvwMEThUnfywx5AGL1vUzuneUF3Xk4oyyenWKSKYITeb46
hlaHGaG8qii8yjoax3q5JHhFMt8780IyvO5ffPy6ZN1JYSWi82BNkFdANaH33n/7EoYcA3KN/Sxq
N4embgpEnv7tmYqREHsehj8hlNyqzlObVTIvJn3p/iOF3oswkHt6jJjxPpwz03xqczmC7fraw/W2
mQZyE2CyKXdIKaHSoZUwiim2tJB48nNLRslnV92g8GzOZ1TdFEiZ7mn0jZTpfvhdUulbXk8rkWfH
R7BE8pScgxq5L3wTm4Ijpja8nhN5t/Xj6iB8mdIoxwhR7Rmx35YIUW5aliErfrKNRRDQ+73o+93o
XJpkUykRbiVM5N5oT0v/55iDIvHJRu1BXI94zRPyJAzqfik5YCsmPHdK5mkSPInM5zLvu62co2hd
su6ksBLRebAmyCugRaEvwjnnfGmkc2JYcqChEXmAkeDQyPMBHCf2YV7nxCWVQqdkUue9h+8NUMl8
n/ylU+5YxIMQel9EIeSQ2jKcssRkvRaUyO1ebDhwGcdwD7D0t01+ZMinZMMHn7oSbd9cm9W0ItwO
6fQXIdN9t/oGiip9yPxcZYOVyOdgcLDQJGf8WSQ2R4m12skwrcscXh/OIyRpnAtPHJkbF1+3FI5v
cZpmbKZ2iMcVCKM0xzmr2tfCoB6bijm3HewvkQrnnC5EiSGBSu6lC4/GEfphRfD8NsnSfeRvjcjP
PJUqeo8aVHsRU00k9zhakk4g89H75ZEkwxumPfpx/4qTwkpEZ8CaIK8ME6F3zr0LwHcA+AnvvTqs
dM49F8DdAN4D4NWzWHiBUUvkpWO8kFAoIfatCNUa1XlKnkUyD8TzcUuh91W2FgY0g91k0MsI5OIZ
2qXih2kGM9S9tAvNUH4uTLv4XIVMv3NjaYU7qUONSinbcu38DG6LIdze0XXclwRfYi7A90Tes0z3
fnTC5VR6miHePEVmCpEvgaqVVHVn155V7yvIr7h/ApHvzsP4PhEbnUbuhXD82ms4u0bbUHasROJz
hG7jxPvb1Acc2lebU4oTZ579PR5y0gwT2KVjSNF85/CdxsfOEtDFCT6JSLFMK5KiSaygCR5pfRGx
B+rIPTOjyyHS0OZcciOZp+o8K9udx226lAxvsMutS9adItZl6hbBmpdAgFWhfwWAfwzgh51zbwXw
qwDeC+ApAB8J4FPR3dxPA/BDAH5kflMvIEpEHigPtrhaDyTEfgokdT7KyNyr8470SyqZB8YwXSnc
2nXnhk4wDHatax1HZQ0V6odJS+k1wUpOhGepLrHXYMtcDomaKIaa1R2Kymfh+FpMOn/KvcyQeCC1
S7uH1568jCtAL3OP5z54793tthkRZboPSL5j9tug0vOl4orvwBJEXoJC7oG8el9CUU2bQuTBHb/9
tvDTxcdIxD4+A3EdPSiB57AQ+OS7p9Uq5P6gq3/MAX5fOKlsCXfnxB7I9z0KkR/VYttNltonLbms
icQTmyYhvN8sL07yDgnTPSw28W+t1rbcvHlK5tUIyD4ZHo1SWpesO36sWe33gjUvgQAToffe/wKA
z3bOfT6ArwLwXwH40wBuAvB+AL8G4PUA/pn3/k8WsvXCw6bKywQ3R+yb7fGI1fkkXIx5l3kCK0bm
R7sAbKTQez+oeB7juXRpqEnkHtifiiPdeikD8PKWqNhck9nG7mnyTa7NEp4WkCcOternZOxjYMQH
0z0kIkbxyPZW+KuX4jnT+3xZvO8GlOEb3jk10334nk0qPS0fBmJvbcNanqXGabnqmFPvS4S9UU0L
z91E5DkYsQcU1b5HDYEHULwms9Nj8AD3/xfm/NPDomNbYCWbjZCuWySVteAr2dAoM0m1LxH5GaAm
OsydNPct5+8PYtW+yjlUcEJVm5Yh84BC5j3kZHi6723FcWJVj5fHmpdAQNUceu/9/QDuX8iWC49k
+Z6zsL1M5IfzvR8Irkbs+XrL2TDonMdaSm5EBpZUne86HjJI1Mg8gprn0qzsvvtPCL0ft5OB3hRy
P9QRCuP7WnvxCb1sZjpFNWbKhK4RfRE196wwZ372cPhCeXMlRczVGyeJ0kk8XybvHU88pzuHLArq
nWt7LyqhJcFLjmP3z6LSJ4PsSsU+gWEgrpWpEdzkc86o90VyW/m8aoh8Wf0ndoRNTLUH6gn8ot+p
RO7pZ3Mp85xy5YK9r0t8RqLDdkYGJizNKKr2MxP5ltVkIuR8vDWkmyG0mbtL9Ib0/1NC7hNTBNt4
lEHT+54j8/38eGsyvM6m/jk7rEvWHT9W9Xh5rHkJBCySFO9kUaEuWlAacFnD65N5lky5TlWjvvE3
ZI61DDgTdd6n6nxSZnFeZhd6n6r0QAi9F2Eh94ZkeUNd3O4GiOGH0uAut2Se4fwiapPlHDpv5cJJ
8ayEXZqqMIW0NJF4ctw2vMzX47mhuW9uTrzlV751zHSPsW4t0/3goCuo9FIOjmZiX3EftGephQcn
7WZOvZ/JwTIrkZcQygk/c/e3ROCt7WsLQtGaE4YkEIvIvXQs//4X/nZE4qs6tSdURJ+/oNoPq51M
zO9gOSf7Hlm+O1944Dn0p9J2dCD3gmqf7DCQ+WZkyDwn68EWMRneMOWIHLouWXfsWNXjhbHmJZCx
EnorinPZlyk7ux6xkdjPBUosEnWe1pe7VySBVin0vpiNt5bcA/MPQKU6aHXi4CX1shcJvgEbYzbm
VuQiImref7WcPTgYxDwGwjvhfPsHXQqnHwagme/z6vYyNueHj6+MnuswtxNRpvsoKV5Gpae5AAYl
jZOgErEvkATP5mSbrzMzkM+q9xOxOJHnYMQeQLzsF5An8EB1G5okzss5T4Yd5XKT7POGd4WvmmBB
i3pcmy8kX1bvoJfuu6Dam9rRmfrB5DqrSDzdNsEI0qbkVPtSfVIb0BrFpS1PR/MLDdvDty8lw8Po
NA3nrUvWHT1W9XjFQbASeoLcgKkpvLuEEpHnx/h+wMwHQQViPyXyQFTnhU6pWA/S+1sKvY9/Iz+Y
lhQKSIRAV5daB8y5rNj6Sayz5xlw97E0TSWBdpZVEwxlSuWI00cm8nsreZ8dORIPmFTeR6/fDHe9
V7UZydsLajLdW1V6SR2jXDmQckLshwMsRCG5BrbTSPBzy2xZop6yZfH9+yLyHLQZXJjAc9DnbCH3
5pDvlnfEgDmmGEiKbH0ZBmIPpP1pVIg8lS0LeljFq1Ak8Ujf+yqEMh1rI5yi2mfqyZH5pucvkXlh
3jwl81oyPPSq/Wa7p3HBiklY1eP9Yk1COGIl9EYso8ALyqBC5GlZY/Z3I7HP2b7JOBWEQd/gKU7U
eYOCu/Nw5zv4S5tY0fOQQ+85GgYWWaVkJkU42+GbB01GqSNryAEU3dK8+O348g3vqxTiPnWJRck0
4/1Ycl66psZHhI2oQNv+xX740dvI/sXMyyKo7dj5cR4qC7cfEuQZVHq38ekzEb7piMAZSZppmlHj
c86p97ORxQYiH7XJmOCsshJ4yUFW6eCIqqEOHGTIPX3naILCTNXqc1kwCojaM3ueAYLc0qv6Sel9
ayOqvFxuGz22QOLptpZl68L7E3mn0sg+Su63gpeySObncOhkyDwANRneoM6j6wfWJetWrEiwJiHs
sYTuXAXn3Nc5537dOfdY/+8B59xfJPuvOOfucc693zn3AefcG5xzz2BlPNs59zPOuSecc484577H
uYZF27xf6B/kf/xe7IgN0I8Lx7iQhVrd35UZOgXpH3bQ/4WyhnKhh6BlOuRA5rvjgjeCHJ+bVqAh
19HSe8ht8eO/xUCe/fAMdvH2chkN/yoR3h/+z3Jdwz/n4n+Z7yBbT813M+F+iDa2ytwAACAASURB
VNcr2gr1Xwmb893wDzs/DhJpvdvxn4azq+PgdLNd+J2VQMl6aCsQnHrp8xvaE0rAknWqM9dseI/5
tzs+Q1S9B1F9Fd8PbT9skTi2f/SbKL1nQ939uzW8S7RdrwH/hgH5e5wL7J5L9zP+RuU2lD8L9bks
cQ0FeBf/S+Aq/mVget5unL4SElPO1pYk77J8r8VnI7z/Nf+k98A0fmJ2DRDIvNvpY6fsuIrWISTB
i8g8sTmZXz/MqfeIpiytOBo4557pnPs3zrnf7v//jPJZK2bEmoSwR5NC35PlTwLwDDCngPf+/6os
7vcAfDOA3+p/fw2Af+Gc+0zv/bsBfD+AF6Gbh/I4gHtAPDC9LT+LLtTiBege5r0ArgF4ZZUlB2os
06R3ynHCIJqeryn22bpzx/ixYxuO7TsXrs6r5VMyH7YFlZ7Y4HcOzgkKXgmhaHGeHO1BpXnSmeNb
bAjlaveDznfkdSXh//UhIX4zT7YcjdT7M8FHVnPPpIEO3c1XZ5g4AFedXRNRIvV0HremxJfwwaeu
jO+nAzbXaiycjqpM9yE8Pzh4wvnSXHp6HqAklJTsYb/50lxhEC6UZ77r/EBLU1QkDPkDWhR5EOJP
bYgzexMTS23qHoluWjf5243XqK9MQE7l0WWFcPLWbO1W8isdN+SnyynZMyB+z4IEPxJ5qU77daUH
Wvrp0jSDqNyZIiCH3BuGcD45UmDcFxyvU/LbqEnwBhu6HVydd7t4v/N17+yKvWJViA+LNQlhj2pC
75x7AYB/jm4deikNTRWr8N7/DNv0Sufc3wHwAufc7wP4WgAv9d7/Ul//ywC82zn3Od77twN4IYBP
BvBF3vv3A3inc+5VAF7tnPs27/05jJhKIKqxYwNmK5HnqCX2LaGZHvqcMoUAcjLvdrte6dvBnaMY
el8VSlp6dAVyPwXV0zGELMVzDKpr8zxUh3JnnT+VZQnlJfdx6nzBEpnK7Z7wPOh1tCZVunZ+Brcd
n9GgQu8JNZnuB/j8XHopVDhL7FFB5Gk0wXBwaEfkayyGLhsIfrFtLq50kt/f1RHKKjhQFQUw9Mhz
5Y+IiHHl8UBsW/Tcqf2WchOHIGtP9u2nkAjink3oKs0T+Vmq0N57zjrpM515fDWOd2h9/TdCp2YI
D0YawwzLAvfEunNWju1Yi33ZJHjhfhAnQmxjIPmeKPhNpqxYFqtCfFisSQh7tCj0PwrgVwH8JQDv
xYzDzF5t/6sAbgbwNgB39Db+QjjGe/8bzrnfBfC5AN6OTpV/Z0/mA94C4EcAfBqAXzcbMKXDaSEf
YQCSqTbpBKWstgElYs+OiyvKD6ijzkWoy+128MSXI5H5Qc07Qx96v4lUvZD1Pl2fvILYQ+noKTRy
3/r4p4zaJHJ/rDCN3qeUzwfk05qWvQ/oQ70zzHO89uRlXAFAwyydBx689+7JZVeDPgc6CA2Z7nc7
eGyKKn2cAC/+povEvkjkQ7nE1GQkzfM7pKSX2pRAeqwLJqkqEvmME0Mk9zWqvWRH+E0duMLzqvnu
LOTeXFZNvTXlG48VHUStTnqpfuMjMxH5mvqld1ztW+OfVofXlLY+jtoIf5CxB/h4gtvAviPf29Mn
p2uxjJJ5MQkeaUeHsVVIhseWqovatHXJumPEqhAfEGsSwhEthP65AL7Se/9bxSONcM79WXQE/ukA
PgDgL3vv/71z7rMAXPPeP85OeR+AZ/V/P6v/zfeHfXZCv+8MojVEHhgVMOeqiL3NlowX2yNW56NQ
OeE8lcyT+fPOlUPvQz1MsQcKA9Lh+omXft9LpYn3s2D/TvnbiOpIAe3aaxQJKUbn1HDIkGMguWeP
bG+Fv3ppWNIMwGGkvh3gxDji/vv1gvaVU+mF5d/4N8GJvZXIS0gSYmYO1sh9sGufsBL50dnab3ZA
jtyrDowC4cnl1ZjTaaaR+6YkcBqog3pu0MgcFi1gtjvXhhuJ/dyRR+L9V+zk1y1COrflcbDxAZBX
7QGdzEfz5r2PwuRb3pUcmR/rHsseku8Nv8NSdd2/kENiXbLuKLEqxEeEGznrfQuhfxDd/PnZCD2A
fw/gzwH4CHQfw+udc1+QOd6aHuQo6IXWIUhbNSJP/w7dl5XYT0WizoftRJ2nv1Uyv0M82C+E3ifX
4dJBXj5UltpRJvetg7zcadpgmNc1C2motF9TklUVQIr4qFVkDg3J0VI7wJyD3yn3bes3eMcTz+m2
XSeHO5dPWLgUeqUql+m+G3CWVXqARA0J5IR+0wmxLxH5wYmX3qNQp3nOMP8uGpXtrk77M8sS+Rwp
EbaL5F4hF47nrpjpPZtSTvQM6JLiLeSeXncrOeNFipEJxDa2akdCdCfNGQ+F1pwjjSsq6wPg2Emz
OCqAaWMVaZqNpNonNglkvp83HwkVTc4GncyH5KFxDgE5GR7dtma4Pw4ohHFViI8HN2xOAxOhd859
Bvn5gwD+e+fcswC8E8B1eqz3/t/VGtHPc/8P/c+HnHOfA+AbAPwfAJ7mnLuFqfTPwKjCPwLg+azI
Z/b/58p9grvuugu33norAODXfvNhAMCzPurT8TF/6tOrrmEOIqMlvRvAM0iHw0rEfgp8hTqfLM2V
kvnxWMRKmBR6n9gidNwmYh//MCn3vKiaENVoMGB7L4rP3oC5unuV6ItrxS84yJhatJG8i9dlUco0
aK9KzvnBbb0ek9DBqXYoBLVo54Cz4HhTDFJUemD8NhJiD4iqPbZsm0Lkc1CJpVC3eH5GvZ91Wkgl
kXfnMSEBiAOkhtxPId7ZVTHK55iWvVOmJWWdoiUSPzEarzStJrFtgWU5zcSeO+KnvLL8HeQEnxpT
TPJYdlJVI6PaR2DfGp83z1dcEKvKtRmq001xrPRjK7oqUbdvTJCXa//vu+8+3HfffdG2xx57TD9h
xRTcsITxRHDD5jSwKvT/Fl1zQpuw/4X8HfZ5VCbFU7ABcAXAOwCcA/gSAD8JAM655wH4BAAP9Me+
DcC3OOc+msyj/zIAjwF4V6mi1772tbj99tsBAC98/rePO/aoNJqIfGZ/kdjz8iqUDas6P2S6xrhd
I/PDtAHvY5We7qcDoFz0gTUc30Lum5PfTCfjCVrKqDyF3nMKdy6PxLLkRRvgTnEwTSVLE8h77lqr
3hOhGDGBG8HV7WVszg+STiuCNdN92G+aS89C6eP1xkPFsQ3RvpItVfN9FUZvJfcWlA6XstwXCNhA
5CWHVYbcDwQkOa/yXaP32HBq7luai9x3BdjqtOzXys0Xaq+vykG87S5aXGUE0Im99h7NObThZeVW
vMnd8yn9nXQrBef/AErmPUmC51Myn7ONz83XoM6bJ9cgqvMVwsCdd96JO++8M9r20EMP4Y477jDZ
uKIKNyxhPBHcsDkNrIT+OUsZ4Jz7bgBvQrd83YcD+GsAvhDAl3nvH3fO/RiA1zjn/gTd/PofAPDL
3vtf6Yt4Kzrifq9z7psBfAyA7wTwQ97766jBFALRQAZNqiydl1c4XiT2EqR6xGXdvKjOS0vRRefl
lHlSdjH0ns/XMxB7EzRy3/r8jQSDZ++fG9XlKo4ejeirpJ0WwZbb85t2Rj9VtalW3o0o3Wc+Jzk5
p3D+o9dvhrvuxnmVwLyDcCOGTPd+9NaJJJ9G1wRIKv3Ox98wm0YgqvasTA0DaZV2avN96SPJyfW5
pmXic9HUeCC2L5D4bnvvDCm0eZzcty7bxu9f9MwKoec10Mi9Svq5XTXztxeZviLcVOU+19yjzbUu
TGX3tO53kdgbifw+pkftZQpW5tPt9pMDGJkP5/N58+K5uXIVWMh8KIsvVRep88Ca4f54cMMSxhPB
DZvTwETovffvCX/3c9sf4MvBOecuAfg8AO9BHZ4J4PXoiPhjAP4dOjL/r/v9d6ELvHwDOtX+zQBe
TmzbOedejC6r/QMAngDwOgDfWmnHXlX5Yr0akWdKuHR+ROwn2DAsuZJEDIROkajz5LdK5kPYrmOD
MiX0Phm4ZtUZMuANm2qXu5vr+asEIh2IqwOelvlylyuP53bm1nVWwAn8wRPMERTJe+tgrSpDOCG9
GpidDz9623j+oQdxkmpMBqegK1eUVHrn4qRVTEWTVfv8+zTF6ROH0pNInYSl29T7JojOTvK3oMa3
RBJlyT0/OEPgk3JroyWAvHoq1Unb9mhudP387aY59EbyL0/fmuGF6e3nxB7IkHvMp8jn8lKkdbZV
Zn6WtC7pwvywUz+PkPlhShOfNz8Dcknwsqsf8DFXGDesGe6PBTcsYTwF3MhZ71uS4v0iOvLNswbe
2u+rXYf+bxb2fwjAK/p/2jG/B+DFNfWKmKLetTS2DYp8FI5uJPbNCJ0d6WByJGnowDQyD0TJtUqh
90Od4Sd3BACyah+84hOSWVWh8r3RBuf7TihXnOfJ13Xm5B04KgKfoBgmvky50V3Uji28M2dXx3nO
m+30aIVJEL7dQNYjh0VBpXfoxX5gCFcVl5mKolpkk9S1zVt6NEAl90BBvZ8RA7loUOOr6hGIRrRf
S+QpEbuGueFqZITJASu/J4Cu3h80UadWd8vzTIj9mRiObyHyITFbK9R55cqyk+mB7XUPNuScXMon
m5B5H+6Frs63RHVJSfCyxw+RI+EcHr2zZrg/FtzIhHHFcaNl+BPmynP8KXTq+I2JOQYNPORKKXsY
5BmJ/SRY1fmelIdt2RBjSaXvyeOwlF2U4GgasQdmIPczhGpLmJPcq2cYyyoS/Jb3KnffCtH4k9dz
P9Q4voHE03v/waeujINLB2yuzWdaE7a77vvkkTT0/waVPvo7tCNktC2R+2TteI3ID4N0gXSq68rz
wlxcZkBOvZ9ItGdR4w2Kd9YGcenRBT+eKJpKubdRW0hOpZeX9E2G9vOI/Y9mZIg9AC1UAEB8X8x9
jHTcxDwisyTqE6LygAK5F8h8dah9BcT7HQQS2i72+8OqIUn7umLFimrcaEvYmQm9c+4n+j89gNc5
5z5Edp8B+AyMiepW1MBI5DnMxD4JrTba5UPCGIOnmnihVTJPlb4NBoVvmPPvdkCYcy1du4XYa7CQ
+4VI+1hZkOAM4YotHfnMnf/iytbS99uIRa+zIUnUtfMzuC2GcHu38wcjIo4PLr2PM92DfMNFlV5Q
nGlSOK7aR3aQv7fCd5JRJcXQXAkWgs/V+xKvKbXhE9R4OWN2PbmvJu+V34s1GZpK7qOyyKk81UEF
id+cV1yD8XpbwsYnQSD2GpJpe4C5TZGTi2r3WrgHlvvX0AZHS1tayT1SMr9MPgVyz6VQe0bmw3Fh
qTquzv/cA69cxMYVNtxoxPACYfKKBM4559UlfY4LNQp9WAPDoUtOd5Xsuwbg/wbwT2ay6+TQoiaG
OVFmIq9sLxJ7DmMm6MRrnVPnvc9fh6T0l0LvNQKcI/YwDIwVcr8EiiR9yakAJSgEZhL4uzXl/i7Q
hprJe022dMu5FTZce/IyrgBRLJTzwIP33m00YD685Ve+FS/6+K/vfpTunUWl7yF+o4JqH/3J1Pju
b8NFtL5GwvfB2/kSoS8qj7VqPKT+gu6Uy570bc9J4I31WDKI83vr1R9CHTU2Womv5F/RLmPOto0R
++3T42EdJZZzQXPuq0Sfnz/j5ScrOyjkfqjXk/MKWe1bk6gWk+Dxeny8VB3fv+LgWJeqO000rUjg
nPtw3ILvwtPwFfh4XHa3ueu4hp/G43il9/4DC9g5C8yE3nv/MgBwzv0OgO/z3t+44fUzoZrIh/lf
BYXXTOwpCgmMiup8blvuujKh98NhgRQ2Entp3wA+160BuXpN92E4cWL4bq1TKQktbrBnAnldNK9B
oe7YkPQ4MXRbmi9s/MasKuUj21vhr17ChqYcPaDPB0CvzO8GdpJkuvf9/HiTSh+TTPEbjVR7ZgeQ
JVj7ChUf6qtKFy8UOUWNFwukJ9HtFeS+oh3Mv9fKOZbo8BZnRInELw3p/djnx8tzISxA5FsgkffE
MdawGIq0akyk2gPpMxkcpAqZTyrRx1tFMJU9KZcf67v2a1XnjxLrUnWnieoVCZxzH45b8QBejE/F
J2EzCCu/hZfjX+GLnXOfd6ykvnoOvff+2wHAOfcMAP8Jukt9+CKEnxwseY6FyEvbrcR+DrsK6nzc
eWU8zLSDLIXeh6z5PWuyEntpMDg1S3QOtVMllgpBVCEls5NQIvgtA8NMVEJ5+bcFMOG+mkl+4RzR
jv73O554TnceWXDTs+XdDgLtHd8hTpaXUekd2P1SyH30fVar8fr7lmyuXQN9IkSHW6EHtvRJSe6L
IZyXHkQPkElOsZ7anJjaTmGaBdBG6mrrXjK5ZFVo+oIwE/lFIqCEbZblTifkS5FyzySqPa3LK/Pm
gVSd92hzqGrTD9UpAGzMtKrzx4Z1qbrTBF2R4P0ALjvnfhvd8/s6AD8KPo3iFnwXXoxPxXOJXOMA
PBdn+Ev4FPwrfCeAv7vn6zChmtA75z4cwA8DeCnGIf/WOfe/A3i59/4x9eRjx5RGtIUk1hJ57bjS
nGy+NvhZXRy0WZ2vUaNz8D62uZLYZ7dpqqDFpiVgJXwWKMRdGywVr31qUjoNVtI1tfrSPZxBuapy
AuYcP9SW68HhQeo4hvHdzouZ7sPfJZUeYLxSIffRMZWj6VZSNTe5F78547sy1bEsEnyN3GtlSN+G
pX2txZzk3kLi90yUWkO2J8EYNWV2blQ4E7Ov1h7ufU61D9vVefNC++y8bza7lASvO6azz2/cqs4f
N9al6k4QdEUC59y/QTxt4gEAt5Dfv+mcez8+Ch+HT1JiL5+LMzwNX4GLQugB/M8APgvdMnFvQ9dc
fR6A/wHA/4iO6J8m9k3otfprBwEFYs+vy23Txa2zJL9GnRfqE+0pqfQbgRRYiX2p7gBG7hcfcOQy
8s9Q/+Ss8Esgd02aEht2TyYLtsOKycWSExrfN75NsO/q9jI254eOsWfY7fpM92eJ0y5qA0oqfX9O
gIXcV7er0nukRVEIzr7ueEaIMwRf/eYa3t3q9z3YKUQk5crlThIzeR8LIOdOv85cgkQATetvayS+
aQky43MRD5thXDDX0oXqXO7SedYILwDeGT0xC3dV2oox6rx5hkGd920SvSUJXiDzo9GrOn9MWBPh
XTjwaRI3s9+3ALgFV6B/8g7A0/G0Y02U10LoXwzghd77+8m2tzjn/haAN89j1gliqiIN2DzrOfJe
Ivb8OFq2QPIBuzof2WaBldQDyxB7Zv+i4PWUiH0rKgZeAOA2sQFzTEUoLn1HcchwVIra96DlvdHU
eIZHr98Md91FyZuOQp2ng03vh0z3/JiiSq84cXRyP6PttccJBD8h9zOr6UWUCHSB3Et1Fk3gt4VG
OGXIjpU8iwSfXqf0Dmihy0MhzImdOJ1ttlWBljkkdlEqallesJHYJ+Ry38RRqGaf0xt54s1SqH34
PdlGfnqGzK/q/FFiTYR3scCnTTyJUaEf8RSgTrXxAJ7C9WMk80Abof8jjBnvKR4D8CfTzLnBUEPk
A7a+U8LmIPbSObTeMD8WFep8ZR1V5y1F7JeAhcDNTewnElN+1ywDx0mDnoXneM5C3qeiMurm4Udv
G/52so9t73jTe+/pMt37HcSXlV6jRaWXoJH7gm2q2msBfSaSbQLBr1HvuwPs5qhQSPxgi7YsqYHc
i8gQ+NSGXDmG6IWGiIJs2bysOULerW1G5Kzi+ybWLyjOpXddI/LDVB7rvalZ8WOP5L1l6pyFzEfq
vNLkFUHvuXPpO6T9Pk6ecKNiTYR3scCnTfxtdFHlHwvgoxHI/ZMAHkaXIY7jN7HFNfyLfRjbghZC
/10AXuOc+2rv/XsBwDn3LADfC+A75zRu33DX20fQ/nJDXGBrhvDg2bUS+1ZoA0mmxhfVeTrgp4MT
i0oPpANUK7GXjs2h9X5JqmPNQFIi9i0DUW3goZUVLe6dDjqax6AKOUsOW9rxcqjBkVGNp6Df+tnV
cc7zZqsMQvcNNuBMMt1b59LX1CWg7KSx1xcRAktUgFh3RX1iBI3Sb5RIPDKETCL4ORJtJfCVDjI1
HD9ypBgiCixZ8TPniPtr2gZrW5x1WHED7NV356d9n0bsS0Q+Kq8Vyj1xB2qs5kp66/oxShSmXxn5
1uFMbf/SdeZ9PC1oVeePBWsivAsEOp+eIMyvfwY6sv8CfAiX8C/7vc8Dhiz3D8PjZ/FuPI5X7cvm
WrQQ+r8D4JMAvMc597v9tk8A8CEAtznn/nY40Ht/+3QTTwN0mTUrtHnruUFr1Ilbib2m4BjB1fkx
7Da1T7SbZ3zVSD35PZB6Wq+R2IPeVyWMdlZkCFw0MCgpK1PVpCnnSwP+mnvXMEAsZrk/5HJLOdtK
79CEufsffOrKOPB2wOaaray9QFKVuLJUUukr/Z4mcllS2i1J4DRyr5VZKq+FBDSSeF6XZ1Np0vbf
4LgzK+AzEkON3EfkSk+cZ1KcF2inBgjPPHkWE+yIzssReyuRP1S00sKoIvdim4Z0fFMLPsYZbEM6
b96jJw3H4LldQbAmwrtBEMj+kDjvgwB+At0s+6ejC8O/isfxFI52yTqgjdD/1OxWXATM0BgX1SeN
HBaIvWlec+N62oM6LxFzQF6+RcPOAzSENdi0Y/8vEfscCuGeum32Q0vXagmZbAlTrFYmNMJuvUcL
D0AWn2eZUxczCQZbknRFZWfCL6+dX4LbYgi3dzufqnuHwnbXkS6e6Z5E14RtkkrvdruY4Ex5X6um
+tRVk20vExJZYYh06BQlXsFUgq+iIsCgq1i5Nn5/ueM2salM7rPYF1GSnC3Ki6oSffFgoZ8T1N/I
YQ6dyA9tm/G21LTD2UMz73FTW2/op2rKDaH2yTktxL6CzMfRTqs6fyxQFN0VFxsvAfAeAE/Hh9DJ
1CPeecxkHpiwDv2K+VBD5HnnXiT2/HzIioyWvKxJnWf71esrhN4PqCX2JcVOs7vRI18cNHAVU3p2
xwCJ4NcOtGqOb0jeuJgtlnPDcyusJiA9zxyJjwj9k5dxBRjDvNANBB+8926D0Qtit9MjN3wvxRdU
+uwyk5YVOiTClLuvM6FJvW/Jpl5D4nNKKyd6JYIvwWr/DERMTopHNmhRF7nrViM7Cvvnxhyh6UN/
x/o5xaGdFB0IPiPyZmdUzb3KOFm0xLvdzhmn5ljL0tqOqeq8UH601vywP3XCrDgs1sz2Nza893/g
nPtVjMkQAx7HCURotCj0cM59BICvBPCJAL7Xe//HzrnbAbzPe//7cxp4kVFF5LXjSsQ+dy61RbKP
dfhZdZ6Wa1XlNVIffofBQRjQ8TnmGrEHO46iMQFdtYLQ+uws5yuonpZZIicFoiAeYzlHO/aYYLXN
mpiKl6mU769ewuacFmAzY68YnHpuJOtGlT5bZkCUlTo+rEjgM46YOVC1ioOlvGYSbzwOKBP82Zfb
6KFdT4GIpUnxpDLMG6Nd+8yu3oScfZXEXiXyrJ+eE1nSnlXoG6YsRk7/SnLPxzKaOo8Zogf633ze
fHrOqs4fAdbM9iteAuCnAfy5/vevA/iKU3DsVBN659xnAPh5dFnt/wyAfwLgjwH8FXRz6b96Rvv2
i+2EHu5s4qCoNNgvrXecI/a19XNY1XmLKm9BH3o/XAsn9vRvgdhnM19nlJ9ZBns1ZSjP7mAoOT+s
z5SWk1OijiUywQrj84muqtZBB8BdJ5uFzOqHwJveew9e9HGvgJjpfuf7EbFBpbc42CJlq5LAixm5
D3//NFRliac+D0LKecSI547cGidbBpKiO9kZYFBZab1VoeqATuQPmZ8D0K/bYleJ2JeI/JJRV3OH
1bfU2/K9K+p89XK8AqQkeKk6f2J94cXFmtn+BkdP3F9waDta0KLQvwbA67z3f885R+cT/CyAfz6P
WTcILOpS3ykXw+MWJodFdT5H5jmp0eYC0r/JPN16Ys9sFa4nCeucAiNRG+bLamtZs/Vyq6FFZLSi
Zq6t8vwnvYYLOzhKofMmWKJQavZdD+88+v/7vBq7b5Bv3nk/XoJRpS8q3CUnUss7cUDyVkz8mGzw
6k5riHQ1wZfKkOqqvPfqtWtOHbX8BnKvEfkWzBStE2GOd1Ih9rMReV6PAQd3SgNlcl+hzk9Fbt58
+O22Hn6zqvNHgjWz/YqTRQuhfz669fs4fh/As6aZc2AcqjOaQuS1sujggkce1EQTVMydN6vyVlLP
YCL2EqQwuszhk1FSZUvEXjtvZkxawxsoR2SUUDhnFsJ9CNSSfHIfN+cnotQkifFgV+kJ+NWmBH8m
ErkQquqztI/kZ67tT0hCcoCTj0VK8KeQ9+z1myOy2G++zJ5WdyY3QGl1hKMgnkaYV0cJ30qJyNck
qq04LmtjqY6GVSFoTeaVY0Qy70eBYmqfRsv2aX2Rk64n80c99ezGw5rZfsXJooXQfwjALcL25wH4
w2nmHBhTGtaWcP05iXyubEtZEslXOv6cOi/WPzOyxB4A+BKCl4Rrm9O+koqo3X6N2B8IRYJfms8M
CNc+4domZpMvopaMTJkikHknovDp664fCIbz2qucHbvdmOmePpvghDOq9BFKBF9zvWnfVE2ysSWn
fJidm/HPJhJfqSCnBL9wXq77yN1vxa7ismJLJCfdh4O0Ytm6qv49OCHCzwJ5FYn8FKJaY6uyFG8R
Lc9HWcmnyqkQIqByDqAG2wKZl+bNJ3lVvF/V+SPBmtl+xSmjhdD/NIB/4Jz7q/1v75z7BAD/GJ1n
63QxZa5qzUBSwlxEXkJyXQ2qf0GdT44v2dGo0lMUiX0AJ/iATPJbIarw45/RAEscwM5E7GsHHjUJ
3ax1snetKptzUvTCg/Da8ltIvjaopt+50O64TG6pg4KTBRrBY1LpC0mqjA4h7d0UNxsTY82aabvQ
JgxVWgl8TV21q1NMcbpNVjENRKyU8T4HzZE2NbJCgrS6hfZ8J+S3KSrTQpsjfi8LZLk/hsgHK7lP
Qu1nVOdFMu/jsUBQ5082Eu2CYM1sv+KioIXQfyO6l/8PANwE4JfQhdq//V9tmQAAIABJREFUDcB/
N59pJ4YWhZ405IsQeQ2i40Kuv0qdtxB5ihKpD+UZiWdVxmmJ5M8xB70Uzpi7pqnJz2rP1wbHVgIk
1VmKVrDUMxf2Mbgs1VFB4gHg7KobVNvNdrqfcE686b334EUf//Xdj/B9UvJIVfqz+LuOVHqK8A60
riNtcQBYM2C3EHy1zSufKppUCqW3QLuOGcn3UFVu8pI1y72Q92UyubdGSc2JGaJ6qqIMSE4KLfHr
pOS0LTgCQk9hup8zq/MAbGR+t4Pberz5335HUx0rZsOa2X7FhUDLOvSPAfhS59znA/gMAB8G4CHv
/c/PbdxFRxWJP+8lu0sLxSFLKiEl7wXiridAmtDBD8p1qKRNUV58nfcWz37JWTExBNEE7dmoa2s3
DA5zz3+h1bIOjkoSTzEQeAdsrs1r1izwHvA7wAvJN6lK7xnBODOSypqIByu0ZlYi+rmlI2cMDx+K
tKrwuWPEzP6Z4yuin4pouX7DOxCRe2vd0vO0TBOaCxUrLEyyI0Psxf18e60iXPOMW9v0PTsCFlXn
OWhxzImwkvmjwJrZfsWFQNM69ADgvb8fwP0z2nJ4NKyFOsAtxE7OZ4i95ddltbVGnT/fxvPwreQv
p9JvXFdOJbHnEJWlBdemjisSBloSaTimJdxyERUW0ONy11UiuEsP8jLl58Igk6zh/NxKEh/XO4bb
u53PsJkDIiGUOyDMFS6p9FzRnfkZVyXwktRl7fzaMHajPaZpJZIDtUTKciRfatcr6x9PzoU0K/0p
bx8K9Te133MS+SlOG2uUQg5aH2F11ChEfpGowNb73GIKd5bUzp2n/5f2S+OcWvRl0PdvDbU/OqyZ
7VdcCFQReufcBsDXoFtz/s+gG27+R3QhK/f6xSe93kDgRH7oXAK5nTLvUZrMqZB8qzpPt2th2N4D
5+fApUvp/hypD2Vaib1hoCMOEud6exPCo9yPHLE/tU/pGO0tRg+0FVsmUxPvRcd8uz898OC9d08r
bwmE7/GMEbKSSp8jzOF8aTvHDO+blKdAddYo9S2i+Gp10feuhYzVRCoYMWvyVu0dkPocqShhuoW6
hOqh26ua+pXl6YplaUT+WO7B3DC2HVXq/FQy77GG2h8/1sz2Ky4EzITeOefQJcT7cgC/DuCd6PrV
TwHwOnQk/7+Y38QTwRR1n4ISeanDHQZQhICXyD0n65Kt4jbiVS6p80CeyAdcfQq46ekpqZfqps6L
FmIvXMeAORVx8TkJ2zTSngvznWpHDlPvwQEVdBMKn2Q1GdlDFMXmnNa3eHX1GDLdn6XbLSq9EnBU
zLSenFDxblQ8N81ZU4rKUDFB/U5s2WeeFYpGhV49T1qmjh8/sU1XifyW9R0GWJ021dnVK+sfk78q
94n3tzRHz77I+zE5CSRH0A7yuISfN/U6ejJPfwcyD6yh9seENbP9iouCGoX+awB8AYAv8d7/It3h
nPtiAD/lnPtq7/3rZ7TvdDC1AygReb4vDO42m3rVXlLjNYdErTpPj9Ou4+pTwId/WHpOskwac15M
JfaSjRHmSIpneA8sxL4FSxP6lvDgAw7wZg8pXdoxBMBdJ9U514XdHxvo9619/5pKr8whcArDUon+
HGrzDES/KXTaWo8lZB0zTCnapxMRqb2Jsm5ch75Ubld4TG5Dm+Cl923irLmlSbOY/FVS7UtE/phI
977BVfOCOu+8b5ueFx4LbR/7/6+h9ofHmtl+xUVEDaG/E8A/5GQeALz3/9o592oAfw3AjUnoW2Al
8UBKFMPgQyL2QH1IvkTyacb2kjovdFwJ6EoAltD7wQ6F2Ifr5cS+Fa38L0e6LM4ZKdS0hcjNkaW/
FqXpBbk07ZXzYI8ScwyOE4IW/s9UniPAkOne7wDPPjiLSq9BWyXEyYxec9aI636r7ZEQLtC6jra1
TgaTCt8QBjxr3pBWhd6IIsFvcaQpRD57Ldp9tToGlTXnq+qSwPqJKGGgoNqfIolvcYYkMzxNq1IM
J2uFmhxmFmjz5ld1/iiwZrZfceFQQ+g/A8Dfy+x/E4Cvn2bOgbHvTs9Sn0bsOImlxD53XitK6vzO
6+QtDNZpFIAl9J4TfInYh7rp/zmmru9eC3qvJJLO7ZlLoa+F9v5NWbZujvqPAdns/DM8J+0dOQVI
yrxVpa9NaLlkmLlYtDInQCP6k6eFtBH4YlZzY94Q7fRDIrFTUDSLUzRqiHwJVkV16VU7hH5CVO3P
hHNy5Z0qSvaXlsI0qPORk7IC67z5o8ea2X7FhUMNof8oAO/L7H8fgI+cZs6KAVaiVCL2ALBjg1Q+
/zUDuzq/QzKikYj8zo+EyBJ6L5HdErHnkLYLpKw5ZNIaZh7uw47ZLZ3XlJNhphGlZv+ew78Xn/dZ
G1nhCu9ZCQYS7zyIitRWzV7B78UUlf4YoH52CtHf4zJ7Vd/DnNNDck1RritZ8PuVwpYjv1GByO8t
mWGJVFrAHcH0XEm1t5Z9ChFQNbBMaZGmCNHz53wvhjHTOm/+CLFmtl9x4VDDAM4AnGf2bzFhGbwA
59zfd87tnHOvIduuOOfucc693zn3AefcG5xzz2DnPds59zPOuSecc484576nz8p/WqBzwyX4nUz2
wnmhU9rtxn/Jsdv0n4BcNtxEnafHbPvEWTlbA86FV2oIl9yOYbFSZztcox8dBVb1NJxD/7WCOj0s
A4JwX6bWOxXc7jkGNH7H/il17OOf9IynPO+We9Vwj7XEcUeF7S6NjJFU+523vQct9Qv/3Nbb//n0
n4qd8q/4DmbO1f71UG3j1219t1ttzKHlmc7Z3hC43W74p5bdYK/0npjfnTnbWOnZznwPJ6PlfV/K
sZC791Z1fkr/g9HxtM6bPzq8BN2y2/+h//+a2X7FyaOGgDsAr3POfUjZf2WqMc655wP4W+iy6FN8
P4AXofvoHgdwD8icl564/yw6L9sL0IXP3AvgGoBXmg2YQq6mhuEW1+TmikOY4M7nsfb76WZtfiow
rh2vkHqtE0zVeciKfFLcDpGfRQu9p/UGUn92Nm7LKfatWHpgxG0vKfat5Vsxx5Jg/FnPeQ+nOjum
2JI7t2ZVglobwue7zacfOCh2OznvA90fvlWu0tfej9pkdNL2ivdc+wLV+ecFMlJch56VKx6fa781
GKOSLMhdg58jkSgv37osW6kcaV+4lTUf1xL9Qk2Zuel0kmpvrHtq9v5Z8zQ0YO8rUlRiDbU/LqyJ
8FZcdNQQ+n9qOKY5IZ5z7sMA/G8A/iaAV5HttwD4WgAv9d7/Ur/tZQDe7Zz7HO/92wG8EMAnA/gi
7/37AbzTOfcqAK92zn2b9z4XWTAPWsiHKSO6MpizEvsScoNFxfsvqvM7382XNdTnzwC32+RD70PZ
G0ICOLHXsuI3zHlbDJpKFOBcSuz3BW0AoxIgw3zfOXGoZbpKmPuaE0IDbK7NW8VioO0A/VYjMo9x
Lv1c68o3EJgpxy5FU2Yj8BYofYIrkdvc/RPWfp8MXl+Nk0Yry3JLteu09qVLO4TF6XTBA1gx9aPS
zr0teVcL5TvRVswYIKnz5BojdX6HJs/qukTdUWJNhLfiQsNM6L33L1vSEHSq+7/sM+a/imz/bHR2
/gKx5Tecc78L4HMBvB2dKv/OnswHvAXAjwD4NKSKv4wpBGJuz7SmfKrz6RRiP9kOozpvRSDllzYj
CQDkrPdAfL0SsQf2n0yuhNLAkA6+omN3yvY9o0b95EjmVGfOqRmE7htLv1MacexfbbfzONY59G96
7z140ce9ovv2vXKfNJX+iHxtRaKmfqd7tGEfKF1b9hue1xQRkhJbSgNTIvL7Ut2XaEfoOKWUBFcg
rMm+EiryRBxFT5xziJWGK3P50vpntIbaHxXWRHgrLjQmz3mfA865lwL4THTkneOZAK557x9n298H
4Fn9389CmrDvfWSfjdAfA7xC6nJEiZJdidiXQhpVWyrU+ZrQ4+0W/mxTDr2nIer7IPZzJDurqSeX
FO8YBvoWWKeKiOceE7tjkEyb+n5ZFEAHjGsYAw/ee/e0OpdEfz3DNBqLSl+bH6B2Gbkadbu4bOKR
foOW9qbmXT3W68xBIkr0m1Veg0GR3VeeiqmqPy/DEpWWU+PnIKyWqQ0ngEidF6cOYdzfUv4aan9s
WBPhrbjQODihd859PLo58l/qvb9ecypsGtZ+epja8OXkfIHIV2e6F4i9WJdRSahR50uqbj/Q9tst
3NlZXeh9LbE/JuSeoTVcco66jrH8UnTHseW0bFHgxIEiH2QL7OIopC4DQqIu6dOTVHoNtc4SdYnK
ijL2pahOhSUE3YJjuballhfNNCdJaLVkw9nE9qaGLFfNoZf6dyh5ZISyp5L4mgiOfS8ROyd4MrwJ
WMn8cYDNm/9DAA8CuA39HPoDmrZixew4OKEHcAe6D+wdzg091BmAL3DO/TcA/iKAK865W5hK/wyM
KvwjAJ7Pyn1m///cUnu46667cOuttwIA3vH+dwIAPuam5+Fjb35e3VW0zA21qvFSOVJ9UceqVyvb
Uu7AqtR5RuTpeR7bka/kQu8pkee/6Tmc2Ofm0OcGtXOpC7XEtxiKf0KoIR9FMjxxJDrFIbDNSHjU
cdQ0DaEsD3rnurD7Y8Wuz7C+6Qh7otIDo/JFVXptwF+rZKq5RSqeec6JaTnWgtL7YbFhLkJfY4Ox
vlnnV1fM8bcmX8ut1JJgYu6CycsKlmBYvq47Ll/M5Gd2LO3ShG+1qM6T41oSP+6bzN9333247777
om2PPfbYXm04UvB58/d77z/xgPasWLEYjoHQ/zyAT2fbXgfg3QBeDeD3AVwH8CUAfhIAnHPPA/AJ
AB7oj38bgG9xzn00mUf/ZQAeA/CuXOWvfe1rcfvttwNANy+0FS2dZA2J18gt3UYxV6dbo85HYfU8
Kz+z7Qxtoff095DcviLT7xJEuTZpkpoxe88DJY0U1RDhKfdzcafFhAF6zrYc2QeEBJX2+N7AXTrn
mfm0w4C3X2dsn3OpSl+rzk516uRQo9Av9a5a5jZP7VsOhVLUVkCDs6SUYb2KyJdgza1To05XkX+a
xW3TlOXeSWOJJfqb1pwpc0ZtNDlLhGR4Pd78zu+uL2/PuPPOO3HnnXdG2x566CHccccdB7LoaLDO
m19xw+DghN57/wQY6XbOPQHgj7z37+5//xiA1zjn/gTABwD8AIBf9t7/Sn/KW/sy7nXOfTOAjwHw
nQB+qCqMf99ZtWsS2eTIYIkozoCiOr/dxvUXOtVi6P1NTxdOMhL7Y4fleTUN4md6f4srK1RgSlK8
qQPO1uWLSijdB28k8MeidDXgTe+9By/6+K/v3pXdDthsbCp9rTNr37MualW/JYhzbZnWJermjj7I
lTmljEZ7RNJqrXOuY5eGRu6VtjTr2LBel+V5WPq0pVYCqoRVnV9xIbDOm19xw+DghF4Bb2XvQpfG
5g3o1rt/M4CXDwd7v3POvRhdVvsHADyBTuX/1n0YOwukwUhtZ1NS7WtRO3c+O72A7SuF3l99alTp
tXWJNWLfikUG54ZcBjWhv9m6ZiACU1EzOC8m1Jto35Tzpzgiasuj8Oz/xww+ILaq9DWofUdrnnmN
LftU6HOozatCsYTDcx9tiDUU/lQcusA8SfFo30ISjFY5Nqag1kmwj+luLaDqfP8bYCLGilPFS9At
TzesPX9Yc1asWA5HSei991/Mfn8IwCv6f9o5vwfgxQubNi+0jjchv8IyNTVlT4RJnS/ZEa7B78aQ
5FzoPT0nXLOV2B96UFFSuEvXcUq4EQc8sycHHMtz+8q+PSfCYJiq9Hw/Vekl1L77SyjDHDXK5NzI
vWOWSBxp2swSURBzOB732e4tGRWzVNlabhyN3OeeyVQbj2mKVUN5RXWebXvTu/9Ri2UrjgTe+z/A
utb8ihsER0nobxhYSbx0zlzqkqUcizofVDi+j11HF2qPYcCZDb2nZQP1xL4WzcvWFQbYOUdDxTSF
2TElAmRFOwr3cbMd59IfPTT1lC5hR1V6DXO9W3OSqn2E+1vsbZlKI51zbKtGBJQIfqndLGEn9LPH
MFWiBRZynzuHYk57LWUdqcNayscwa7LHFXsDy2r//wF4SU/qV6y48FgJPcWURryls7KS+NK5xfWU
WbnamrUCqtR5TvCVOvx2C3dpVOnV0HtO1K3EPofstIAZcyhYph9wxXLqgGfJ8ORDYKp9U8jdVCeY
taykbGBzra74g2G3E6a9ePhNQaU/FpS+992BCHBtO2SNNFDLLSz3Oee3YIFFwRdtopneZww7t7Yj
VUsmztDXa+R+6alMUjkWoWHf71HODj5NT0iGd/T94woJPKv9G7Eq9CtuEKyEnmLfhN5K4Et2lTpT
i+NAC+W3qvO83NI8vlLo/c6nAyQrsedTAPa5Pn3NOzRXZEFL3SvyyH2b1mkvJSjPy+08TmIOPUcf
dj8QK0mlr20nq50nczrl9pwk1QLpnSnNl7f2H/tCbci91b4SiafbJi5RdzBYyH3pvFzbVvMsLsoc
egVruP1JYs1qv+KGxUro54LWSbYO/qfOTaztPAX7a9R5v93CGcLH/c7DbVw59N4JSj2/thKxD5Dm
+M9J8qfOWTzlOfQc+w7jXPJ8DSVHnPTNW20Jr4IHHrz37jq79ow3vfeeeKlP2lZIn5c2bxU4zBz6
Y4Z2faXlTYfjhHOXSBrX8hz4NWiO29o6+HGzKdFWx/tCER1av27p76Wpb9mEnxW21GJuhb51jETU
eTUZ3opTxZrVfsUNi5XQL40apU/rSKaS+9bjq9T5zGCdV0VJvRJ6H5F6oI7YWwYOEslvVW2SOf+K
isHt5DjFgcQp2mzBFIW+lYAEnJhfx+92cOGeaEvYlcLtD/EeWUPVp5TRiiWSl7WK0kuHSpcIvvVe
1CSe2FVkn9zHtIKWYzVyr/XPFojRHg3nSMg5lFocfXPmy1jKGbRi31iz2q+4YbESeoopHUSL+mGZ
Z31IVKjzKoqhgHLoPYBuPj0vx0LspygQLeD1lRwzJWLfijmcOHPjGN7juVFS6KXoj4r74J3rwu5P
GZpKPxfmeK9K99jyee77OS2hkJbKzF3j2YkshTelDus1HrI9jcLvjWq8eUoJ+5DFKJEgb2c+mpal
QLNjpJZkkYo63yOo82u4/WlizWq/4kbGSugJ/IQ5k26O0atlQGANq7eWa5kzZ1Hn6f+TItL7Wg69
7+bTdyZWEvscsgOcPQ3Olyb2VtQkYQRsdvIyc8+kNpljLQ41hWHie9RNdZnJlqWx23XTZIBOpQ+D
ZE2lzzn/JOwz/wXHIZ0qU6O1WrPA12IvJNbaDhTaJxqBVdO2nB14dQBt2dqaaRkUteMcNUv+ieYh
4PC+PXplxUGxZrVfsWLESugPiZZkP9r5reSlFCXQqM5HJF4j+rnQezKQD2UlxD549U9FBdaS4NUS
+7nzNUwBt2XO0MVDPtd91H1+vnwdh4Sk0tfe11qnzhyhzMeAJfJHHEuODk4ES8vptd4LPoWqJsy+
BTWOkVYnSi25H/aT88J9kVTxooN/Iea7T+fZDvl58sfcLqzgWLPar1jRYyX0x4opS5DNOXCzqvNQ
SLx5PnEcet/Np78cH8KJPVXraV1Lz/msgWQLf1ZS4qLaMlswVQEHTpewl+qf+x2SyLvqqKsv/lB4
03vvwQtv+uupSg8A3qcq/VyoUSdbwnkvIlqvN0fi5ljar5bgaygR+FbSaI6KWKBMfqwWWq/mZemP
2Ur9uHU5wEI/EZ2Ty6B/4ClmHEoyvDXc/qSwZrVfsaLHSugppniJaxLyBOSSqWnIrSkfMBvhq1Tn
paRyBdKohd7n7kWR2GcrXMAzX8rKTHMRAHVLSklQyczCyv2pEaDi3OBGh0apXCN5nzLF5+gxLGE3
EfuYK3/smCN5X7b8I3sPuT3m5zcTgedorT+HZueKgdxLavycbfcSjqEWst/yfLk6vybDO3WsWe1X
rOixEvqZ0DI4dwZ1g5ebzNW3EPyWZDRAlTov2tKf9+bH/9fh55du/suk8xZD72n5io0qsW/FvgY9
1M6p5N6COZT4EmqdDFPKtGIf1y3W207eN9s23+BBEaJrtoTEEyeW3yyk0kvY9zPfNwGQru+QeQZa
UJv7xDw1rbRf6c9OFRq5F9V4Aebs9I021aClWZjyDJVkeCtODmtW+xUreqyE/sggDvy1ZdCkgZAw
4MuRiaxToXbuPLGNkniKn9v9uEjqxzJ26b5aYn/IkHted4lgaOR+jrpLqHWA7HuK/lQHzRRmPKHu
InnPLI+4udZc7cHwlqf+WRd2D4hL2O11Hb6ab2CWJdcmOhC0KIZDOaMkZO/pAtN25mqjj+keToHl
/uRUZ+242e6z1TGQW4KuIb9BAwlP1Hn2iqzh9qeFNav9ihUjVkJPMcnj21DdJjc30WDLDJ1znuwL
c9IzS8NpJJ7j53Y/ji89+6rYDi30nsJK7HOV555xbQbuAK6QtSpBUwnsEktI3aCYFA6fIewluJ0/
qTn0AyRHnKTSLw2JxC2ZLPIYokimhuQXp6bkIrzyp5rK4+156z0t3UupH7PAnKzUXuSsuUdqn28u
10zLSjFzRFAsHblDoT37VaU/GazZ7VesSLES+mNCLYkH0ozpdNscsKrztR2yQAASUq+hNGe+VaGf
ayB5Iw4MtHf3mOPHj/Q5OQ88eO/dhzajCoNKT5PjAYdR6TmOaVWIJVAip6XrPHSOgaUI/lz5UsxJ
8Y5kmcxSItaasoCUiC/xPFrah8nOtL7mI+0HVmSxZrdfsYJhJfSHRAuBL+2nWdNr6tHKtqrz7/+f
qoouhd777XbsaLWBUk0yvKHgBTrvKQOoA6JWhbbkfBAqqT9nOHWaeun8CUYsnKDJAySVPiCo9LXv
UO0rdyxLs82JqQp8ifDvew5+SWFuVYA1TI2CsEZuSc/h0O/jHP1Q7RikBU0J7hqeazhHsHkNtz8p
rNntV6xgWAn9XGjp1Epz4fZlR6msudX5HhKpDyq9aIeV2LcO4FodHznSoSzrt5fw4xJqr3ffubdO
dA59uWz9/fTOjVNdTgyiSh+F97p6J1LtS1c1xeUI5lifgOMvi5Z3dams/cfwPDnmSBZaimCw1snr
bXVAtGaGXyI6biJc1D6tOCGs2e1XrGBYCT3FvgcEDSGfIqEG4HIqyxxJ4mZU50sQST1gJ/aHTIpH
7cggWb1gDoKvXfchVaIDktNJCv+MGZRr4PyJzqHPIYTdt2S5r3Y6XUCF/lShfUPHQJ72nTxxSv0c
tQR/7jnjS5zXco+n2LEmwzt1rNntV6xgWAn9sYN3xrzj6ztzjegXUeoUF1LnA7TQe58byJeIfQ5L
DCZnWGN8jjXJVTVzDwNrzf6jiERYsT+QJew0lf5osI9w5BJaFNEl6mjFPu7hEo72GkfRPpsw4V4U
nb+W5WMn1L9ixbFhzW6/YkWKldBTTOnM9rXUTjI/nu2XCHauzFyUQCDwC6vzxaXsNEwh9jPCTMbn
XKLuAJjD6VBZYX7/1OeeG9QfKnz3xMfTdAm7CENyvIVxaoRkH/YuOQd6HzlJpoZ07+udqMkaP2Nu
EZPTdMl7MEdb2dKWt6wmQtT5Ndx+xYoVFwkroZ8LLZ3CEksW1RZptGEJdT6thCXVspIATuwbO+i9
EtbasMmWMovHV17vWcMUkSXvaVHZW67qSbjoA0ii0g/fL1nCrqqo3NKeAqqS7p2gU+0gmPt95e1O
qY1fIK9MTbtkzuOw1Hdd6CsSgu8z97O0dJ+lvz2SXAVz9S1ruP1pYF2q7v9v786jJKnqtI9/n2qg
G2g2YaCRHRFBUZRNQFCUAWQRl/HoOPqqg6OyOCLOHFEHRWTmHcUDAoM7IojSviAjywg0i56RtRto
NqFBkGanm6WbbmgaaLp+7x83sioqKjMrMysjl6rnc06dyoy4cePGjcjM+MWNuNesPgf0bRIt/MjV
fe59KOMmTxLa3eN6h56dH2qlz4sYXn8zgX2rz213szOyNqy7VvDT1dve+7SDt+510tSV1bZVvpU+
BgerDGHXjB4Punvh+B6rSsc7Dn3Zmg3waylrO3rtDoZ2dpI3Ku9u3ZnUoWPQrfP9zEPVmdXhgD6n
laC8/YXovR+YjrTOU+PW+6EgvfHAvuO3hpehpeOg+j7pSH2UEdiM9/M4jlu8x/NdoIkyvvl4VBvC
Ln+BrlFN3orbTIt+0z3od8lYn98x70oY63Bstf8VaO0zNlbnncVjpMHvlgnxvd+Kdg/z16gePFex
Cc1D1ZnV4YC+m3r9B7GDPduPkAUDI1r3ItIJXzMt9k2vt8X90e2xhjul062R3fx8dGvdPf6V0Kiq
Q9h1QjPHaA/cDtETF9vGNbxji88xd1Irz1rnTYSzpGrHQA8c/+PSht8j327fVzxUnVkdE+Gnqn16
oVO8TK0WQtVaT50T5nqtjWOdaFdrnb9i0c/qLjMeQ6302YlupewjesuuBPbtDhJabZUtI1hppSzN
PuPezhPrWq18jTxWMsnU/TwGzD73yx0sTYmqtdI3nUefBx1jacejNoyjhR1QC31jlKLGvm74okcz
gXsz36+NHoNNPJPe1Kg0zQ5H2ugx1UwnfmUsn9fKXSItfjf4dvu+5aHqzOpwQN8uLfw4tPUW/xbz
qluGlSu78oxotefpm2qtb7XM/f4DX2u72/V4RAdPutqiS4/Q9MSjOz2gait9s8dDs3U5GR93KHs0
iPGsu5qyvqea+X7q1F1V7fguqNXHQCvnHLmLI1Uf1ZiEHx/rXe4Iz6xxDujzxhO8ltVjcqND+JRx
glLtVvsSW+fzrhq8gAOmf2rEtLqt9TA5T+bzah0b3YwvHdw2Ra6u8WnieJs0z1yXOGxdUy3NY6jZ
F0Cjz9A3se97tp+Lats6aiSb5joRbOo4r7b+Zs5tevyxNeXK59vt+4I7wjNrkAP6XtPqD2K/ty5X
MeuFc0YF9VCjtR6GT3RavM271ZbVhkYraFY792etW1mb3N5StrPMhVENAAAgAElEQVSO8bZ09+xJ
ez0T7WOcG8KuI8/SN9Wjdxsqe7wXctvxOZ8gF81qBp7d/m1r9Dhp5tAez7E3RoDf9gtVzZS11c9D
p4f97fYxZY1yR3hmDep6QC/peOD4wuR7I+KN2fypwCnAR4GpwCzgyPxtN5I2A34M7AM8D/wS+Gr0
ehNMvR+VGj+i0UoTXpt+vDrVOp8364VzABpvrYfOXxTp1xPqDlw06FvjCvYmWF2MQ34IOyj/IlJT
rbRN5VySdnx3dPWxlhbW3WzQ12gd9dN3UKe/e6vtp17q3qSV46if9re1yh3hmTWo6wF95s/Avgyf
Y72am3cqcCCpA4ylwA/I3XajNMj2ZaQP+u6kK3jnAq8Ax3Wg7K1r54/YBO9pvV5gr+IzhZ0OsHul
M8Va292uVtE+O4Hys+w9ItdK39WOG8vQC+PQ96pmHwEq67G1Kpp5VKCUUpX53dTNY7KD627l+72y
L327fd9wR3hmDeqVgP7ViHi6OFHS2sBhwN9HxP9m0/4RmCdpt4iYAxwAbAe8OyKeAe6S9A3gO5K+
FRGvFvOtaTwN+q08SjjeHqDzSj7x7UbrfDXVAvsRrfX9pk23w0Od1swuBrbjCqrHe0xP8Itc/aLY
Sl+qZo6ZdnxnToRjrJ23f5dRjl6/qNNrGqyvMm5gVDvPacri46lvZHfi+pl5swb0SkD/ekmPAy8B
NwJfi4hHgZ1JZbymkjAi7pP0CLAHMIfUKn9XFsxXzAJ+BLwJuKMTGxAtnBRpoJUf1Bbuk5uArUjV
nq8fCuxbfda7xXpq6TGIXtBkh1YxEYKXRo3nZLcfTmo7Ld9K34wyL9K140LXePuV6IHgouefTOvy
nTaN1o9Kuoe95hC24/1sNNw3QOPf+60eS31xIcA6wj3bm7WmFwL6m4BPA/cBGwPfAv4kaQdgBvBK
RCwtLLMwm0f2f2GV+ZV5HQnoJ7JeaZ0vqnUbfi+cJFdVdrn6Pf+i8V6I8jliz2i1lb7UYK5Xvyea
NO7OI5u9yDIRlHGRu8PHU81Av92dl3agU7yOXVSK8O32vc8925u1oOsBfUTMyr39s6Q5wMPAR0gt
9tWIxnqeauoXtpVW9vFoqVW/hd/qVocW6pf22JqBfaf0SGDQbEtOs8dfK8ee2ZCWTtqbvCrTzDHd
juO5Fz77vVCGZpRV3gl4J9qk4U7xbJh7tjdrQdcD+qKIWCLpL8A2wNXAapLWLrTSb8hwK/wCYNdC
Nhtl/4st96Mcc8wxrLPOOgDc9upcAGYMbM7GA1s2WfAOXWHu8ElLr7bOV1MJ7HvVAWt+stwVtPF5
/HaZteyXLS97wLSPj2/ltca1Lts4vgtmn/vlNhakt8x66delr6Njz+q3yRVLf9HtIvDedQ5rfeFO
9FtSQuA2a/m5bc/zwBlHNp640309dCzfDg5b14KJ2jo/c+ZMZs6cOWLakiVLulSacXPP9mYtUPTY
VU5J00kt9N8k9Vb/NKlTvN9l87cF7gXeHhE3S3ovcCmwceU5ekmfA74LbBgRK2qsZyfg1ltvvZWd
dtqp7M0y65paFxJiReP9RQJc+cp57SiOmZmZlWju3LnsvPPOADtHxNxul6dRkjak0LO9n6E3G1vX
W+glfY8UkD8MbAKcQBq27jcRsVTSz4FTJC0mjTF/OnB9RNycZXElcA9wrqRjSc/hnwicUSuYN5tM
xtNKbmZmZtYJ7tnerDVdD+iBTYHzgPVJrfHXAbtHxLPZ/GNIg8L9FpgKXAEcVVk4IgYlHULq1f4G
YBlwNnB8h8pvZmZmZmZm1nFdD+gj4mNjzH8Z+Ofsr1aaR4FD2lw0MzMzMzMriYeqMxs/D+xkZmZm
ZmbdUBmqbuvs/4XdLY5Z/3FAb2ZmZmZm3eCh6szGyQG9mZmZmZl1Q3FoOg9VZ9akrj9Db2ZmZmZm
k9LfURiqrrvFMes/DujNzMzMzKzjPFSd2fj5lnszMzMzMzOzPuSA3szMzMzMzKwPOaA3MzMzM7PS
SdpI0rWS/pr937DbZTLrdw7ozczMzMysEzzuvFmbOaA3MzMzM7NO8LjzZm3mgN7MzMzMzDrB486b
tZmHrTMzMzMzs07wuPNmbeaA3szMzMzMSudx583az7fcm5mZmZmZmfUhB/RmZmZmZlYKD1VnVi4H
9GZmZmZmVhYPVWdWIgf0ZmZmZmZWFg9VZ1YiB/RmZmZmZlYWD1VnViL3cm9mZmZmZm0lSREReKg6
s1I5oDczMzMzs3GTtNam8O/T4dA9YNXtpRWbwiWPwUER8Xy3y2c2ETmgNzMzMzOzcZG01hZww4/g
je+FAQEBXAFHHQHvkbSng3qz9vMz9GZmZmZmNi6bwr//CN54YBbMAwg4EKb8ELbfDE7sZvnMJioH
9GZmZmZmNi7T4dD31ogtDoQp0+HQTpfJbDJwQG9mZmZmZi2TpPVgVdWaD6wLq0mqlcTMWuSA3szM
zMzMWhYRsRhWRK35wGJYkfV6b2Zt5IDezMzMzMzG5QW45ApYWW3e5bByGVzc6TKZTQYO6M3MzMzM
bFweg+OOgHmXwcpKM3wAl8HKI2Deo/CNbpbPbKJyQG+lmDlzZreLMGG5bsvjui2X67c8rtvyuG7L
47qdWCLi+Ydhz8PhjDfB/D3h8TfB/MPhjEfAQ9aZlaQnAnpJr5V0rqRnJL0o6Q5JOxXSfFvSE9n8
qyRtU5i/nqRfS1oiabGkMyWt2dktsQr/SJfHdVse1225XL/lcd2Wx3VbHtftxBMRzz8S8aV7Ira+
ETa7J2LrRyK+5GDerDxdD+glrQtcD7wMHABsD/wLsDiX5ljgC8Dngd2AZcAsSavlsjovW3Zf4GDg
ncBPOrAJZmZmZmaW4w7wzDpjlW4XAPgq8EhE/FNu2sOFNEcDJ0bEpQCSPgksBD4AnC9pe9LFgJ0j
4rYszT8Dv5f0rxGxoOyNMDMzMzMzM+ukrrfQA+8DbpF0vqSFkuZKGgruJW0FzACuqUyLiKXAbGCP
bNLuwOJKMJ+5mtQXx9vL3gAzMzMzMzOzTuuFFvqtgSOAk4H/IAXgp0t6KSJ+RQrmg9Qin7cwm0f2
/6n8zIhYKWlRLk3RNIB58+a1YxusYMmSJcydO7fbxZiQXLflcd2Wy/VbHtdteVy35XHdliN3bjut
m+Uws85Qtx9vkfQyMCci9s5NOw3YJSLeIWkP4DrgtRGxMJfmfODViPgHSV8DPhkR2xfyfgo4LiJ+
WmW9/wD8upytMjMzMzPrqo9HxHndLoSZlasXWuifBIrN5POAD2WvFwACNmJkK/2GwG25NBvmM5A0
BViP0S37FbOAjwMPAS+1VnQzMzMzs54yDdiSdK5rZhNcLwT01wNvKEx7A1nHeBExX9ICUu/1dwJI
Wpt0a/4PsvQ3AutKelvuOfp9SRcCZldbaUQ8S+oZ38zMzMxsIrmh2wUws87ohVvudyEF9d8CzicF
6j8BPhsRv8nSfAU4Fvg0qUX9ROBNwJsi4pUszWWkVvojgNWAs0i38v+fzm2NmZmZmZmZWWd0PaAH
kHQQ8B1gG2A+cHJEnFVI8y3gc8C6wLXAURHxQG7+usAZpF7zB4HfAkdHxIud2AYzMzMzMzOzTuqJ
gN7MzMzMzMzMmtML49CbmZmZmZmZWZMmVEAvaW9Jl0h6XNKgpEOrpNle0sWSnpP0gqTZkjbNzZ8q
6QeSnpH0vKTfStqwmM9k00jd5tL+JEvzxcL09ST9WtISSYslnSlpzfJL3/vq1a+kVSR9V9Kd2TH7
uKRzJG1cyMP1W0WD3wvflvSEpBclXSVpm8J8120DJA1IOlHSg1ldPiDpuCrp6ta3VSfptZLOzX6f
XpR0h6SdCmlct+Mk6WvZd8UpuWk+N2hBVpdzJC2VtFDS7yRtW0jjum0zSUdJmi9puaSbJO3a7TKZ
WXkmVEAPrAncDhwFjHqWQNLrSM/f3wO8E3gzqYO9/LB1pwIHA3+XpXktcGGppe4Pdeu2QtIHgN2A
x6vMPg/YnjQCwcGk+v1J20van+rV7xrAW4ETgLcBHySNBHFxIZ3rt7qxvheOBb4AfJ507C4DZkla
LZfMdduYr5Lq8UhgO+ArwFckfaGSoMH6toKsn5jrgZeBA0jH478Ai3NpXLfjlAU+nwXuKMzyuUFr
9gb+i9Th8d8CqwJXSlo9l8Z120aSPgqcDBxPOme4g/Q9sEFXC2Zm5YmICflH6hjv0MK0mcA5dZZZ
m3Sy9MHctDdkee3W7W3qlb9qdZtN3wR4hHSiOR/4Ym7edtlyb8tNOwB4FZjR7W3qpb9a9VtIswuw
Etg0e7+967e1ugWeAI7JvV8bWA58xHXbdP1eCvysMO23wC8brW//1azb7wD/O0Ya1+346ng6cB/w
HuCPwCm5evS5QXvqeIOs3vZy3ZZWxzcBp+XeC3gM+Eq3y+Y///mvnL+J1kJfkySRrgDfL+mK7Nav
myS9P5dsZ2AV4JrKhIi4jxSk7tHRAveZrH5/CZwUEfOqJNkDWBwRt+WmXU1qMX17B4o40axLqrvn
sve74/ptmqStgBmM/MwvBWYz/Jl33TbuBmBfSa8HkLQj8A7gsux9I/Vt1b0PuEXS+dnv11xJ/1SZ
6bptix8Al0bEHwrTd8HnBu1S+e1alL33eVcbSVqVVKf5+gzSb5br02yCmjQBPWmM+umk8ewvA/YD
fgf8t6S9szQzgFeyk6C8hdk8q+2rpLo7o8b8GcBT+QkRsZL0o+66bYKkqaTWuvMi4oVssuu3NTNI
J5cLC9Pzn3nXbeO+A/w/4F5JrwC3AqdGxG+y+Y3Ut1W3NXAEqQV5f+DHwOmSPpHNd92Og6S/Jz3a
9LUqszfC5wbjll34PxW4LiLuySb7vKu9NgCm4O8Bs0lllW4XoIMqFy8uiojTs9d3StoTOJz0bH0t
os5z45OdpJ2BL5Ke1Wp6cVy3DZO0CnABqc6ObGQRXL+taKTeXLejfRT4B+DvSX2VvBU4TdITEXFu
neVcl2MbAOZExDey93dIehMpyP9VneVct2NQ6hj3VGC/iFjRzKK4bpvxQ+CNwF4NpHXdtpfr02wC
m0wt9M+Qnnkt3g4+D9g8e70AWE3S2oU0GzL6aqcN2wv4G+BRSSskrQC2AE6R9GCWZgGpHodImgKs
h+u2IblgfjNg/1zrPLh+W7WAdKKzUWF6/jPvum3cScB/RsQFEXF3RPwa+D7DrZ6N1LdV9yRj/365
bluzM+k37Nbcb9i7gKOzO00WAlN9btA6SWcABwH7RMQTuVk+72qvZ0j96/h7wGwSmTQBfXbV/WZS
Zyt52wIPZ69vJQX9+1ZmZsOrbA7c2IFi9qtfAm8Bdsz9PUE6uT8gS3MjsK6kfCv+vqQT0NmdK2p/
ygXzWwP7RsTiQhLXbwsiYj7phDL/mV+b9Gz8Ddkk123j1mB0K9Ag2W9Ng/Vt1V3P6N+vN5D9frlu
x+Vq0qg3b2X4N+wW0p0Pldcr8LlBS7Jg/v3AuyPikcJsn3e1UXaueysj61PZe38PmE1QE+qW+2xc
6G1IJ9oAW2edMi2KiEeB7wG/kXQtqQfbA4FDSFfiiYilkn5OalleDDwPnA5cHxFzOrs1vaWBul1c
SL8CWBAR9wNExL2SZgE/k3QEsBppKJuZEbGgU9vRq+rVL+niyIWkk81DgFUlVa6+L4qIFa7f2ho4
dk8FjpP0APAQaSjLx8iGBXTdNuVS4N8kPQrcDewEHAOcmUtTt76tpu8D10v6GnA+KVD/J9IQaxWu
2xZExDLSIyJDJC0Dnq108upzg9ZI+iHwMeBQYFnut2tJRLzk865SnAKcI+lWYA7pO3gN4OxuFsrM
StTtbvbb+UcKzAdJtxvl/87Kpfk08BfS+LxzgUMKeUwlnaw/Q/phuQDYsNvb1u2/Ruq2kP5BcsPW
ZdPWJbV4LCFdAPgZsEa3t60X/urVL+nxheK8yvt3un5br9tcmm+RLpy8CMwCtink4bptrK7XJJ1M
zs++Y+8HTgBWKaSrW9/+q1m/BwF3ZvV2N3BYlTSu2/bU9R/Ihq3L3vvcoLV6rPbduxL4pOu21Ho/
knRRbznpToddul0m//nPf+X9KcJ9ZJiZmZmZmZn1m0nzDL2ZmZmZmZnZROKA3szMzMzMzKwPOaA3
MzMzMzMz60MO6M3MzMzMzMz6kAN6MzMzMzMzsz7kgN7MzMzMzMysDzmgNzMzMzMzM+tDDujNzMzM
zMzM+pADejMzMzMzM7M+5IDezMzMzMzMrA85oDczm4AkDUo6tNvlKIOkVSXdL2n37P0W2fa+pc3r
OVzSxe3M08zMzKydHNCbmfUJSb/IAteVkl6RtEDSlZL+UZIKyWcAlzeYb78F/0cAD0bETblpUcJ6
fg7sLOkdJeRtZmZmNm4O6M3M+svlpGB9C+C9wB+A04BLJQ19p0fEUxGxojtFLN1RwJmFacULGuOW
1d95wNHtztvMzMysHRzQm5n1l5cj4umIeDIibo+I7wDvBw4CPl1JlG91z25RP0PSE5KWS3pQ0rHZ
vPmk1u2LsmUezKZvLemi7C6A5yXNkbRvviCS5kv6mqSfS1oq6WFJny2k2UTSTEnPSnohy2fX3Pz3
S7o1K9cDkr6ZvzBRJGkXYGvgsjppBiSdJekeSZvk6uNzki6VtCybt7uk10n6Y1a26yVtVcjuUuB9
kqbWWp+ZmZlZtzigNzPrcxHxR+AO4EM1khwNHAJ8GNgW+ATwUDZvV1Lr9qdILf+VYHs68HvgPcBb
SXcGXCJp00LeXwZuztL8EPiRpG0BJK0J/AnYOFv/W4CTyH57JO0FnAN8H9gO+HxWjn+rs7l7AfdF
xLJqMyWtBvw2W9deEfF4bvZxwNnAjsA8Uuv7j4H/AHbO6uGMQpa3AKsCb69TJjMzM7OuWKXbBTAz
s7a4F3hzjXmbAfdHxA3Z+0crMyLimezx+yUR8VRu+p3Anbk8jpf0IeBQUuBe8fuI+HH2+ruSjgH2
Af4CfBxYH9gpIpZkaR7M5wn8Z0T8Knv/sKRvkoL+E2tsyxbAk1WmB7AW6SLEqsC7I+L5QpqzIuJC
AEknATcCJ0TE1dm004CzRmQasVzSkmy9ZmZmZj3FAb2Z2cQgancMdzZwlaT7gCuA/4mIq+pmllrX
TyDdyr8x6fdiGrB5IeldhfcLgA2z1zsCt+WC+aIdgT0lHZebNgVYTdK0iHipyjKrA9WmC5hJuljx
noh4uUqafFkXZv//XJg2TdL0iHghN305sEaNbTAzMzPrGt9yb2Y2MWwPzK82IyJuA7Yk3XI+DThf
0gVj5Hcy6dn8r5Juc9+RFPyuVkhX7HgvGP5tWT7GOqaTWul3zP3tAGxbI5gHeAZYr8a835Nutd+z
xvx8WaPOtOJv42uAp2vkaWZmZtY1bqE3M+tzkt5Dut3+5FppshbnC4ALJF0IXCFp3Yh4jhTUTiks
sidwdkRckq1jOumiQDPuBD6TW0/RXOANEfFglXm13AYcXmV6AD8C7iY9639wRPxpjLzGHOpO0tbA
1Gy9ZmZmZj3FAb2ZWX+ZKmkjUgC+EXAgqRX9EuDcagtI+hLpufPbSUHsR4Anc0H2Q8C+km4g9aL/
HHA/8CFJ/5Ol+TbNDw03E/g6qQf9r2dleBvweETMzvK8VNKjpI7sBsla6SPiGzXy/COwpqQ3RsQ9
+c0EiIgzJE3J8j0oIq6vU75q21OctjdpzPuqdz+YmZmZdZNvuTcz6y/vBZ4g3V5/OfAu4AsR8YGI
yLc451+/ABxL6o1+Nuk5+INy8/8F2A94hNRqDqn3+sXA9cDFpGfv5zJStRbuoWnZOO77AU+Rboe/
MyvHymz+laTe7/cD5pA6qfsSwz3wj848YhFwEamn/lrrPQ34FvB7Sbs3UtY60z4G/LRWeczMzMy6
SSPP/8zMzHqbpDcDVwLb1Bq+rk3r2R74A+mZ/mKP+WZmZmZd5xZ6MzPrKxFxF6mlf8uSV/Va4JMO
5s3MzKxXuYXezMzMzMzMrA+5hd7MzMzMzMysDzmgNzMzMzMzM+tDDujNzMzMzMzM+pADejMzMzMz
M7M+5IDezMzMzMzMrA85oDczMzMzMzPrQw7ozczMzMzMzPrQKt0ugFm/kDQN+AxwOLB6l4tjZmYT
XwAXAd+LiKe6XRgzM+s9iohul8Gsp1UC+amsfsbLLGcjNmUaawJCGkpE7s3wPw3lMTxv6DXDr5Vb
ZsTyubRQfdlq+dScX8i/OA2IwmakMld5nf2PEfPz+SqbX2VebpmotmyNPKNQHZX1RCHdyPLl3ufm
j8orP6+YV/Z+5HpG1mOtZWqur0Z5R5V5rLxqvW5gnY2UqX5ZYuw8WlhntfWMynNU2qg+vdp8Ch8t
ZesoTB86slRImuU38nAdvfzwITI8L3e05j6W+WVjaNHi+kWMWGbU8jXyHDk/Ri1Dblrltr2Glx8x
Pxgo1IMYriepWrrc/Fw+lbQDhenDeWbzVX3+gHLbV1xnLv/a80eWaWQ5GEpbyWBgRF7k6rGy7crN
Vy7d8Ouh6dJQuqXPD/LrC5/n1VeDI/9xHU7+0XMbObA3M7M8B/RmNRQD+RlszlZsz5paK0swgAZU
9TWQnWlmJ2YDA5VMYcTrygnc6GVGpa38r7wekbbG8oX1IBHKnYEW8xxjfkgjp1em5YL4GDU/XxZG
5ZNfPvL5jDg7VmF+nbSFdY5YZqD68sPLUP31qGUYrqcqy9TMp0qeVeczxvwxlm9pGarnU71uK9Ni
1Pxqy6R1xshyjMozFyCNWj5qpyXbhdWWzwVdGrVMVF1+KNArLDMqAFSMSDswavnIfVSHp+UD3uIy
A7k8BzQy6B2aVu011ecX80nzB0fOJ7/MIFNG5Tk4NH8Ko5efomCA4WlTKM4fHMprytCyg0zJ5Z9/
ndaTX+fg8HLk8xwcWv/wOgdzy1S2Y3BomaF0xFCeQ+vOrWdKrk4qy4ycFrkyMzyf7LVgSnakDE/T
UPA+PE9MUeX1wPD87Pt8IFt60eKVnPrT5zj9zOcc2JuZ2Sh+ht6sQNI0SUdNZfXlwBnr8TfswQHs
oN2Gg3kzM7MOeM16U/j2sevz4Jwt+fLh6/HTc5ey+jQt/Ncj1wtJG3a7fGZm1l0O6M0yDuTNzKxX
ObA3M7NqHNDbpOdA3szM+oUDezMzy3NAb5OWA3kzM+tXDuzNzAwc0Nsk5EDezMwmCgf2ZmaTmwN6
mzQcyJuZ2UTlwN7MbHJyQG8TngN5MzObLBzYm5lNLg7obUKTtLsDeTMzm2xqBfaSPtftspmZWfs4
oLeJbpOXWc5UVmd9ZrAG07tdHjMzs455zXpT2H+fNdhhu9VY/lIA7NDtMpmZWfs4oLcJLSIuBHZd
i3W5mzncyCyejEeIiG4XzczMrFTXzV7O/h99jHd94DFeWDYI8GHgS10ulpmZtZEDepvwIuKWp+MJ
AbuuwVoO7M3MbELLB/JPPb0S4MN3zXtlSkRcGBGD3S6fmZm1jwN6mzQc2JuZ2UTmQN7MbPJxQG+T
jgP72p5c/OduF6F0zzx8W7eLULold8/tdhE6Ytns27tdhNI99Yd53S5C6eb+/sluF6EjLrzoxdLy
diBvZjZ5OaC3ScuB/WhPPjfxA/pnJ0FAv/Tuib+NAC9OhoD+j/d2uwilu+2yBd0uQkf890XL256n
A3kzM3NAb5OeA3szM+snDuTNzKzCAb1ZxoG9mZn1MgfyZmZW5IDerMCBvZmZ9RIH8mZmVssq3S6A
Wa+KiFsASdplDda6+W7mMJ972DK2YxprAANoUCmxhMheV/6HQOl1Ph0aGE42UFlmYPifcvkM5Jar
TKu8HsitSwNVpuXXSe51ZTqj5q94dTmLnn9oRJGG1w0h5S4DanhaftM1crlQvkyMWiZGLDOcd4zI
M59Xmha5tNWqfrjMI5dZ8fIylix8YHQ5i8tXXuc3N5/XqG3Kl7/etlHYttx66q2z2vYV88zer3xx
GcsefmBEnjXLUa3MufVVW9dwXjEyfd1lomr+o/McXSYUhW1Pea18YRkv3fvXtAuVu9hWpUwj5gPK
5alcniM/ajEyfT5rxYiP0tD83P/CIZZNG17PQGVPVz7Sitx6ggFgxZKXWHz7o6PmD+TWNcDwOoe+
MrJpA7l1DgikwZHLZGnS8oNMKS7PYO4rJxjIlq9Mm0IgBofyH6AyPy0/Jb/+oXIM5l4Hyxav4IE5
i4aXYXDEMlMq64Sqy1fWOZAr88BQmQerphso1N0AxfmF5UdMi9z689Mqr2EgK+3QNIlFiwa57oaX
mUKljsWAKumGf0OmZN/nQix9fpAzznqOa/60nDdvvxqkQP53DuLNzKxCbnE0a4ykXYDjgUO6XRYz
M5s07gJOABzIm5nZKA7ozZokaVNgWrfLYWZmE14A8x3Im5lZLQ7ozczMzMzMzPqQO8UzMzMzMzMz
60MO6M3MzMzMzMz6kAN6MzMzMzMzsz7kgN7MzMzMzMysDzmgN5sEJM2XNFjl779qpP9UNn9lLu2L
nS53syRNl3SqpIckvSjpumy4wXrL7CPpVkkvSfqLpE91qryNkLS3pEskPZ7th0OrpPm2pCeybb5K
0jZj5Hl8lWPhnvK2or562yhpFUnflXSnpBeyNOdI2riBfI/Kjv3lkm6StGu5W1K3LHX3Y7ZP5mXb
uCjbj7uNkWdP7cesTGNt5y+qlPmyBvLtm32Zpdle0sWSnsv26exshJRaefbld66ZmXWfA3qzyWEX
YEbubz/ScEjn11lmSWGZLUouYzv8HNgX+DiwA3AVcHWt4E/SlsD/ANcAOwKnAWdK2q8ThW3QmsDt
wFGkfTaCpGOBLwCfB3YDlgGzJK02Rr5/BjZieP/u1cYyN6veNq4BvJU0DvfbgA8CbwAurpehpI8C
JwPHZ8vdQaqXDdpa8sbV3Y/Afdm8HYB3AA8BV0paf4x8e2k/wtjbCXA5I8v8sXoZ9tu+lPQ64Frg
HuCdwJuBE4GXxsi3H79zzcysyzxsndkkJOlU4KCI2LbG/FyGyhcAAAY4SURBVE8B34+I13S2ZK2T
NA14HnhfRFyRm34LcFlEfLPKMt8FDoyIt+SmzQTWiYiDOlDspkgaBD4QEZfkpj0BfC8ivp+9XxtY
CHwqIqpesJF0PPD+iNipA8VuSrVtrJJmF2A2sEVEPFYjzU3A7Ig4Onsv4FHg9Ig4qf0lb1yD27gW
KcDbNyL+WCNNz+5HqHm8/oL0+fpQE/n01b7MvkNeiYiG7/bpx+9cMzPrDW6hN5tkJK1KasH++RhJ
pyvduv6IpIskvbEDxRuPVYApwMuF6cup3Wq5O3B1YdosYI/2Fq0ckrYiteRdU5kWEUtJwe5Y2/D6
7Jbhv0r6laTNSixqu61Lahl9rtrM7BjfmZH1EqR93fP7Niv/50nbd8cYyftxP+4jaaGkeyX9UFLN
ILbf9mV2seFg4H5JV2TbeZOk9zeweL9955qZWQ9wQG82+XwQWAc4p06a+4DDgENJwf8AcIOkTcov
Xmsi4gXgRuAbkjaWNCDpE6ST/lrPW88gtWbnLQTWljS1vNK2zQxSYFttG2bUWe4m4NPAAcDhwFbA
nyStWUIZ2yrbL98Bzsv2eTUbkC7uNFsvXSXpYEnPk27NPhrYLyIW1VmkH/fj5cAngfcAXwHeBVyW
BcLV9Nu+3BCYDhwLXEZ6vOl3wH9L2rvOcn33nWtmZr1hlW4XwMw67jDg8ohYUCtBRNxEChYAkHQj
MA/4HOk51l71CeAs4HHgVWAucB7QzC3JlcCin59HEnXKHxGzcm//LGkO8DDwEeAXJZetZZJWAS4g
bduRrWRBb+/XP5D6ctgA+CxwgaTdIuKZaon7cT8WHgO5W9JdwF+BfYCqjxbU0Kv7stJQclFEnJ69
vlPSnqSLLtdWW6iPv3PNzKzL3EJvNolI2hz4W+BnzSwXEa8CtwF1e0/vtoiYHxHvJnVatVlE7A6s
BsyvscgCUudceRsCSyPilfJK2jYLSIFNtW0otmjWFBFLgL/Qw/s3F8xvBuxfp3Ue4BlgJeOsl06L
iOUR8WBEzImIz5IuSn2mieV7fj8WRcR80v6qVeZ+25fPkPbbvML0ecDmjWbSL9+5ZmbWfQ7ozSaX
w0gnwWMOE5UnaYDU+/aTZRSq3bLAaKGk9Ui3I19UI+mNpF7x8/bPpve8LBhaQG4bsk7x3g7c0Gg+
kqYDr6NH928umN+a1Enc4nrpI2IFcCsj60XZ+4brpQcMAA0/+tHr+7GabCi39alR5n7bl1l5byaN
xJC3LenuiYb023eumZl1j2+5N5skspPgTwNnR8RgYd45wOMR8fXs/TdIt38+QOqA7CukIZTO7GSZ
myVpf1KL9X3A64GTSC1jZ2fz/y+wSa736R8DX8h6uz+LFCR8GOiZHu6z56G3YfhRgK0l7QgsiohH
gVOB4yQ9QBrq7ETgMXLDukm6BrgwIn6Yvf8ecCkpwNiENCTcq8DMTmxTUb1tBJ4ALiQNXXcIsKqk
SmvtoiyAGrWNwCnAOZJuBeYAx5CGwDu7/C0abYxtfBb4N+ASUgC3AWkowteSLmRU8ujp/ZiVqd52
LiLdPn4h6ULUNsB3SXcVzMrl0bf7MvtMfg/4jaRrSY8RHEg6dt+Vy2NCfOeamVn3OaA3mzz+lnS7
crVnazcj3dZasR7wU1KnU4tJLWR7RMS9ZRdynNYB/pMU3CwCfgscFxGVbduYtK0ARMRDkg4mBQxf
JAXCn4mIYs/33bQLKSiI7O/kbPo5wGERcZKkNYCfkAKBa0lD8eUfGdiKFCRWbErqW2B94GngOmD3
iHi2zA2po942ngC8L5t+eza98vz0u4E/ZdNGbGNEnK80Tvm3Sbdr3w4cEBFPl7oltdXbxiOA7Uid
xW1ACvBvBvaKiPyt272+H6H+dh4JvIW0neuSLtbMAr5ZuTCT6ed9eVhEXCTpcODrwGmkC4wfioj8
nT8T5TvXzMy6zOPQm5mZmZmZmfUhP0NvZmZmZmZm1occ0JuZmZmZmZn1IQf0ZmZmZmZmZn3IAb2Z
mZmZmZlZH3JAb2ZmZmZmZtaHHNCbmZmZmZmZ9SEH9GZmZmZmZmZ9yAG9mZmZmZmZWR9yQG9mZmZm
ZmbWhxzQm5mZmZmZmfUhB/RmZmZmZmZmfcgBvZmZmZmZmVkf+v/NAK0XxJij0AAAAABJRU5ErkJg
gg==
)


There are many things the user can do with the API.
Here is another example that finds all glider deployments within a boundary box.

<div class="prompt input_prompt">
In&nbsp;[6]:
</div>

```python
bbox = [
    [-125.72, 32.60],
    [-117.57, 36.93]
]
```

The cell below defines two helper functions to parse the geometry from the JSON and convert the trajectory to a shapely `LineString` to prepare the data for GIS operations later.

<div class="prompt input_prompt">
In&nbsp;[7]:
</div>

```python
from shapely.geometry import LineString


def parse_geometry(geometry):
    """
    Filters out potentially bad coordinate pairs as returned from
    GliderDAC. Returns a safe geometry object.

    :param dict geometry: A GeoJSON Geometry object

    """
    coords = []
    for lon, lat in geometry['coordinates']:
        if lon is None or lat is None:
            continue
        coords.append([lon, lat])
    return {'coordinates': coords}


def fetch_trajectory(deployment):
    """
    Downloads the track as GeoJSON from GliderDAC

    :param dict deployment: The deployment object as returned from GliderDAC

    """
    track_url = 'http://data.ioos.us/gliders/status/api/track/{}'.format
    response = requests.get(track_url(deployment['deployment_dir']))
    if response.status_code != 200:
        raise IOError("Failed to get Glider Track for %s" % deployment['deployment_dir'])
    geometry = parse_geometry(response.json())
    coords = LineString(geometry['coordinates'])
    return coords
```

Now it is easy to check which tracks lie inside the box.

<div class="prompt input_prompt">
In&nbsp;[8]:
</div>

```python
from shapely.geometry import box

search_box = box(bbox[0][0], bbox[0][1], bbox[1][0], bbox[1][1])

inside = dict()
for deployment in response.json()['results']:
    try:
        coords = fetch_trajectory(deployment)
    except IOError:
        continue
    if search_box.intersects(coords):
        inside.update({deployment['name']: coords})
```

Finally, we can create an interactive map displaying the tracks found in the bounding box.

<div class="prompt input_prompt">
In&nbsp;[9]:
</div>

```python
def plot_track(coords, name, color='orange'):
    x, y = coords.xy
    locations = list(zip(y.tolist(), x.tolist()))

    folium.CircleMarker(locations[0], fill_color='green', radius=10).add_to(m)
    folium.CircleMarker(locations[-1], fill_color='red', radius=10).add_to(m)

    folium.PolyLine(
        locations=locations,
        color=color,
        weight=8,
        opacity=0.2,
        popup=name
    ).add_to(m)
```

<div class="prompt input_prompt">
In&nbsp;[10]:
</div>

```python
import folium


tiles = ('http://services.arcgisonline.com/arcgis/rest/services/'
         'World_Topo_Map/MapServer/MapServer/tile/{z}/{y}/{x}')

location = [search_box.centroid.y, search_box.centroid.x]

m = folium.Map(
    location=location,
    zoom_start=5,
    tiles=tiles,
    attr='ESRI'
)


for name, coords in inside.items():
    plot_track(coords, name, color='orange')

m
```






<br>
Right click and choose Save link as... to
[download](https://raw.githubusercontent.com/ioos/notebooks_demos/master/notebooks/2016-12-20-searching_glider_deployments.ipynb)
this notebook, or click [here](https://beta.mybinder.org/v2/gh/ioos/notebooks_demos/master?filepath=notebooks/2016-12-20-searching_glider_deployments.ipynb) to run a live instance of this notebook.