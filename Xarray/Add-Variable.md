One process that might be necessary would be to add a variable (DataSet) to an existing Xarray DataArray

Let's take a look into a GFS weather model .grb data file and append a new variable with data.

~~~Python
import xarray as xr
import pygrib as pg
import metpy

File = 'GFS_Global_0p25deg_20190910_0000.grib2.nc'

data = xr.open_dataset(File,engine="pynio")

data = data.metpy.parse_cf().squeeze()

#for var in data:
#    print(var)
#type(data)
data
~~~

~~~Python
>>>
<xarray.Dataset>
Dimensions:                                             (height_above_ground1: 2, height_above_ground3: 7, height_above_ground4: 3, isobaric: 31, isobaric1: 29, isobaric4: 34, isobaric5: 22, isobaric6: 21, lat: 161, lon: 321, potential_vorticity_surface: 2, time: 7)
Coordinates:
  * time                                                (time) datetime64[ns] 2019-09-10T03:00:00 ... 2019-09-10T21:00:00
  * lat                                                 (lat) float32 60.0 ... 20.0
  * lon                                                 (lon) float32 230.0 ... 310.0
    crs                                                 object Projection: latitude_longitude
    height_above_ground                                 float32 2.0
  * height_above_ground1                                (height_above_ground1) float32 2.0 80.0
    height_above_ground2                                float32 80.0
  * height_above_ground3                                (height_above_ground3) float32 10.0 ... 100.0
  * height_above_ground4                                (height_above_ground4) float32 2.0 ... 100.0
    height_above_ground_layer                           float32 1500.0
    height_above_ground_layer1                          float32 3000.0
  * isobaric                                            (isobaric) float32 100.0 ... 100000.0
  * isobaric1                                           (isobaric1) float32 40.0 ... 100000.0
  * isobaric4                                           (isobaric4) float32 40.0 ... 100000.0
  * isobaric5                                           (isobaric5) float32 5000.0 ... 100000.0
  * isobaric6                                           (isobaric6) float32 10000.0 ... 100000.0
  * potential_vorticity_surface                         (potential_vorticity_surface) float32 -2e-06 2e-06
Data variables:
    Composite_reflectivity_entire_atmosphere            (time, lat, lon) float32 ...
    LatLon_Projection                                   int32 ...
    Convective_available_potential_energy_surface       (time, lat, lon) float32 ...
    Convective_inhibition_surface                       (time, lat, lon) float32 ...
    MSLP_Eta_model_reduction_msl                        (time, lat, lon) float32 ...
    Planetary_Boundary_Layer_Height_surface             (time, lat, lon) float32 ...
    Precipitable_water_entire_atmosphere_single_layer   (time, lat, lon) float32 ...
    Precipitation_rate_surface                          (time, lat, lon) float32 ...
    Sunshine_Duration_surface                           (time, lat, lon) float32 ...
    Surface_Lifted_Index_surface                        (time, lat, lon) float32 ...
    Visibility_surface                                  (time, lat, lon) float32 ...
    Apparent_temperature_height_above_ground            (time, lat, lon) float32 ...
    Dewpoint_temperature_height_above_ground            (time, lat, lon) float32 ...
    Relative_humidity_height_above_ground               (time, lat, lon) float32 ...
    Specific_humidity_height_above_ground               (time, height_above_ground1, lat, lon) float32 ...
    Pressure_height_above_ground                        (time, lat, lon) float32 ...
    u-component_of_wind_height_above_ground             (time, height_above_ground3, lat, lon) float32 ...
    v-component_of_wind_height_above_ground             (time, height_above_ground3, lat, lon) float32 ...
    Temperature_height_above_ground                     (time, height_above_ground4, lat, lon) float32 ...
    Storm_relative_helicity_height_above_ground_layer   (time, lat, lon) float32 ...
    U-Component_Storm_Motion_height_above_ground_layer  (time, lat, lon) float32 ...
    V-Component_Storm_Motion_height_above_ground_layer  (time, lat, lon) float32 ...
    Relative_humidity_isobaric                          (time, isobaric, lat, lon) float32 ...
    u-component_of_wind_isobaric                        (time, isobaric, lat, lon) float32 ...
    v-component_of_wind_isobaric                        (time, isobaric, lat, lon) float32 ...
    Absolute_vorticity_isobaric                         (time, isobaric1, lat, lon) float32 ...
    Geopotential_height_isobaric                        (time, isobaric4, lat, lon) float32 ...
    Temperature_isobaric                                (time, isobaric4, lat, lon) float32 ...
    Graupel_snow_pellets_isobaric                       (time, isobaric5, lat, lon) float32 ...
    Ice_water_mixing_ratio_isobaric                     (time, isobaric5, lat, lon) float32 ...
    Rain_mixing_ratio_isobaric                          (time, isobaric5, lat, lon) float32 ...
    Total_cloud_cover_isobaric                          (time, isobaric5, lat, lon) float32 ...
    Vertical_velocity_pressure_isobaric                 (time, isobaric6, lat, lon) float32 ...
    Geopotential_height_potential_vorticity_surface     (time, potential_vorticity_surface, lat, lon) float32 ...
    u-component_of_wind_potential_vorticity_surface     (time, potential_vorticity_surface, lat, lon) float32 ...
    v-component_of_wind_potential_vorticity_surface     (time, potential_vorticity_surface, lat, lon) float32 ...

 ~~~

The variable we want to add would be a an array of PV values of 2e-06 and have the heights be our new vertical level data

Thus in similar fashion to the RH variable with <b><i>isobaric</i></b> being the vertical levels
~~~Python
Relative_humidity_isobaric                          (time, isobaric, lat, lon) float32 ...
~~~

The new variable should have a similar DataSet with Dimensions and <b><i>vert_levels</i></b> are the vertical levels

~~~Python
PV                          (time, vert_levels, lat, lon) float32 ...
~~~
# Make a new variable:

~~~Python
vert_levs = np.arange(19000,22000,1)
~~~

If you wanted to add a new dimension to all variables already in the data set. This isn't exactly what we want, so we'll ignore it...
~~~Python
data_newcoord = data.assign_coords(vert_levs=vert_levs)
data = data_newcoord.expand_dims('vert_levels')
print(data)
~~~
~~~Python
>>>

~~~

Now let's make a variable to add to the gfs_data array:
* To test, we'll grab the values of 2e-06 for one vertical level:
~~~Python
%%time

data2 = data["Geopotential_height_potential_vorticity_surface"][:,1,:,:].squeeze()
print(data2.shape)
data2 = np.zeros((len(time.values),len(vert_levs),len(data['lat']),len(data['lon'])))
data2.shape

for m in range(0,1):
    for l in range(lat_start_index[0][0],lat_end_index[0][0]-15+1):
        for k in range(lon_start_index[0][0],lon_end_index[0][0]-30+1):
            for j in range(0,1,1):
                if vert_levs[j] == data["Geopotential_height_potential_vorticity_surface"][m,1,l,k].values.astype(int):
                    print("ahh:",vert_levs[j],m,l,k)
                    data2[m,1,l,k] = 2e-06

~~~


~~~Python
AHHA = xr.DataArray(data2, coords=[time.values,vert_levs, data['lat'],data['lon']],
             dims=['time',"vert_levs",'lat','lon'])

~~~
