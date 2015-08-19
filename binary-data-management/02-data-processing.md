---
layout: page
title: Data Management in the Ocean, Weather, and Climate Sciences
subtitle: Data Processing
minutes: 30
---
> ## Learning Objectives {.objectives}
>
> * Access the metadata within netCDF files
> * Process and plot the contents of a netCDF file
> * Use vectorisation instead of looping to speed up data processing
> * Awareness of the python libraries used to make plotting and metadata handling easier

## Binary file formats

We can guess from the `.nc` extension that we are dealing with a Network Common Data Form (netCDF) file. 
It's also compressed, hence the `.gz`. 
Had we actually downloaded the file to our computer and uncompressed it, 
our initial impulse might have been to type, 

~~~
!cat IMOS_ACORN_V_20121001T000000Z_TURQ_FV00_monthly-1-hour-avg_END-20121029T180000Z_C-20121030T160000Z.nc
~~~

but such a command would produce an incomprehensible mix symbols and letters. 
The reason is that up until now, 
we have been dealing with [text files](http://en.wikipedia.org/wiki/Text_file). 
These consist of a simple sequence of character data 
(represented using ASCII, Unicode, or some other standard) 
separated into lines, 
meaning that text files are human-readable when opened with a text editor or displayed using `cat`. 

All other file types (including netCDF) are known collectively as [binary files](http://en.wikipedia.org/wiki/Binary_file). 
They tend to be smaller and faster for the computer to interpret than text files, 
but the payoff is that they aren't human-readable unless you have the right intpreter 
(e.g. `.doc` files aren't readable with your text editor and must instead be opened with Microsoft Word). 
To view the contents of a netCDF file we'd need to use a special command line utility called 
[`ncdump`](http://www.unidata.ucar.edu/software/netcdf/docs/netcdf/ncdump.html), 
which is included with the netCDF C software 
(i.e. if you're using any software that knows how to deal with netCDF - 
such as the Anaconda scientific Python distribution used in Software Carpentry workshops -
then you probably have `ncdump` installed without even knowing it).   

For this example, however, we haven't actually downloaded the netCDF file to our machine. 
Instead, IMOS has made the data available via a THREDDS server, 
which means we can just pass a URL to the `netCDF4.Dataset` function in order to obtain the data.

~~~ {.python}
print acorn_DATA
~~~

~~~ {.output}
<type 'netCDF4.Dataset'>
root group (NETCDF3_CLASSIC data model, file format UNDEFINED):
    project: Integrated Marine Observing System (IMOS)
    Conventions: IMOS version 1.3
    institution: Australian Coastal Ocean Radar Network (ACORN)
    title: IMOS ACORN Turquoise Coast site (WA) (TURQ), monthly aggregation of one hour averaged current data
    instrument: CODAR Ocean Sensors/SeaSonde
    site_code: TURQ, Turqoise Coast
    ssr_Stations: SeaBird (SBRD), Cervantes (CRVT)
    id: IMOS/ACORN/au/TURQ/2012-10-08T15:00:00Z.sea_state
    date_created: 2012-10-30T16:31:50Z
    abstract: These data have not been quality controlled. The ACORN facility is producing NetCDF files with radials data for each station every ten minutes.  Radials represent the surface sea water state component  along the radial direction from the receiver antenna  and are calculated from the shift of an area under  the bragg peaks in a Beam Power Spectrum.  The radial values have been calculated using software provided  by the manufacturer of the instrument. eMII is using a Matlab program to read all the netcdf files with radial data for two different stations  and produce a one hour average product with U and V components of the current. The final product is produced on a regular geographic (latitude longitude) grid More information on the data processing is available through the IMOS MEST  http://imosmest.aodn.org.au/geonetwork/srv/en/main.home. This file is an aggregation over a month of distinct hourly averaged files.
    history: 2012-10-09T03:31:35 Convert totl_TURQ_2012_10_08_1500.tuv to netcdf format using CODAR_Convert_File.
2012-10-09T03:31:36 Write CODAR file totl_TURQ_2012_10_08_1500.tuv. Modification of the NetCDF format by eMII to visualise the data using ncWMS �
	0x0006 %
    source: Terrestrial HF radar
    keywords: Oceans
    netcdf_version: 3.6
    naming_authority: Integrated Marine Observing System (IMOS)
    quality_control_set: 1
    file_version: Level 0 - Raw data
    file_version_quality_control: Data in this file has not been quality controlled
    geospatial_lat_min: [-29.5386582 -29.5927878 -29.6469169 -29.7010456 -29.7551738 -29.8093016
 -29.863429  -29.9175559 -29.9716823 -30.0258084 -30.0799339 -30.1340591
 -30.1881837 -30.242308  -30.2964318 -30.3505551 -30.404678  -30.4588004
 -30.5129224 -30.5670439 -30.621165  -30.6752857 -30.7294059 -30.7835256
 -30.8376449 -30.8917637 -30.9458821 -31.        -31.0541175 -31.1082345
 -31.162351  -31.2164671 -31.2705828 -31.324698  -31.3788127 -31.432927
 -31.4870408 -31.5411542 -31.5952671 -31.6493795 -31.7034915 -31.7576031
 -31.8117141 -31.8658247 -31.9199349 -31.9740446 -32.0281538 -32.0822625
 -32.1363708 -32.1904787 -32.244586  -32.2986929 -32.3527994 -32.4069054
 -32.4610109]
    geospatial_lat_max: [-29.5252994 -29.5794147 -29.6335296 -29.687644  -29.741758  -29.7958715
 -29.8499845 -29.904097  -29.9582091 -30.0123207 -30.0664318 -30.1205425
 -30.1746527 -30.2287624 -30.2828717 -30.3369805 -30.3910888 -30.4451966
 -30.499304  -30.5534109 -30.6075173 -30.6616232 -30.7157287 -30.7698337
 -30.8239382 -30.8780423 -30.9321458 -30.9862489 -31.0403515 -31.0944536
 -31.1485553 -31.2026564 -31.2567571 -31.3108573 -31.364957  -31.4190563
 -31.473155  -31.5272533 -31.5813511 -31.6354484 -31.6895452 -31.7436416
 -31.7977374 -31.8518328 -31.9059277 -31.960022  -32.0141159 -32.0682094
 -32.1223023 -32.1763947 -32.2304867 -32.2845781 -32.3386691 -32.3927596
 -32.4468495]
    geospatial_lat_units: degrees_north
    geospatial_lon_min: [ 112.3100111  112.3090067  112.3080002  112.3069914  112.3059805
  112.3049674  112.3039521  112.3029346  112.3019149  112.3008929
  112.2998688  112.2988424  112.2978139  112.2967831  112.29575
  112.2947147  112.2936772  112.2926374  112.2915953  112.290551
  112.2895044  112.2884556  112.2874044  112.286351   112.2852953
  112.2842373  112.283177   112.2821143  112.2810494  112.2799821
  112.2789125  112.2778406  112.2767663  112.2756897  112.2746108
  112.2735294  112.2724458  112.2713597  112.2702713  112.2691805
  112.2680873  112.2669917  112.2658937  112.2647933  112.2636905
  112.2625853  112.2614777  112.2603676  112.2592551  112.2581402
  112.2570228  112.2559029  112.2547806  112.2536558  112.2525286]
    geospatial_lon_max: [ 115.7758038  115.7766744  115.7775469  115.7784212  115.7792975
  115.7801756  115.7810557  115.7819376  115.7828215  115.7837073
  115.784595   115.7854846  115.7863762  115.7872697  115.7881651
  115.7890625  115.7899618  115.7908631  115.7917663  115.7926715
  115.7935787  115.7944878  115.7953989  115.796312   115.7972271
  115.7981441  115.7990632  115.7999843  115.8009074  115.8018325
  115.8027596  115.8036887  115.8046199  115.8055531  115.8064883
  115.8074256  115.8083649  115.8093063  115.8102497  115.8111952
  115.8121427  115.8130924  115.8140441  115.8149979  115.8159538
  115.8169118  115.8178719  115.8188341  115.8197984  115.8207648
  115.8217334  115.822704   115.8236768  115.8246518  115.8256289]
    geospatial_lon_units: degrees_east
    geospatial_vertical_min: 0.0
    geospatial_vertical_max: 0.0
    geospatial_vertical_units: m
    time_coverage_start: 2012-10-01T00:00:00Z
    time_coverage_duration: PT1H19M
    local_time_zone: 8.0
    data_centre_email: info@emii.org.au
    data_centre: eMarine Information Infrastructure (eMII)
    author: Besnard, Laurent
    author_email: laurent.besnard@utas.edu.au
    institution_references: http://www.imos.org.au/acorn.html
    principal_investigator: Wyatt, Lucy
    citation: Citation to be used in publications should follow the format:"IMOS.[year-of-data-download],[Title],[Data access URL],accessed [date-of-access]"
    acknowledgment: Data was sourced from the Integrated Marine Observing System (IMOS) - IMOS is supported by the Australian Government through the National Collaborative Research Infrastructure Strategy (NCRIS) and the Super Science Initiative (SSI).
    distribution_statement: Data may be re-used, provided that related metadata explaining the data has been reviewed by the user, and the data is appropriately acknowledged. Data, products and services from IMOS are provided "as is" without any warranty as to fitness for a particular purpose.
    comment: These data have not been quality controlled. They represent values calculated using software provided by CODAR Ocean Sensors. The file has been modified by eMII in order to visualise the data using the ncWMS software.This NetCDF file has been created using the IMOS NetCDF User Manual v1.2. A copy of the document is available at http://imos.org.au/facility_manuals.html
    data_center: eMarine Information Infrastructure (eMII)
    date_modified: 2012-10-30T16:31:50Z
    time_coverage_end: 2012-10-29T18:00:00Z
    featureType: grid
    data_center_email: info@emii.org.au
    acknowledgement: Data was sourced from Integrated Marine Observing System (IMOS) - an initiative of the Australian Government being conducted as part of the National Calloborative Research Infrastructure Strategy.
    dimensions(sizes): I(55), J(57), TIME(493)
    variables(dimensions): float64 LATITUDE(I,J), float64 LONGITUDE(I,J), int8 LATITUDE_quality_control(I,J), int8 LONGITUDE_quality_control(I,J), float64 TIME(TIME), float32 SPEED(TIME,I,J), float32 UCUR(TIME,I,J), float32 VCUR(TIME,I,J), int8 TIME_quality_control(TIME), int8 SPEED_quality_control(TIME,I,J), int8 UCUR_quality_control(TIME,I,J), int8 VCUR_quality_control(TIME,I,J)
    groups:
~~~


The great thing about netCDF files is that they contain metadata - 
that is, data about the data. 
There are global attributes that give information about the file as a whole 
(shown above - we will come back to these later), 
while each variable also has its own attributes.

~~~ {.python}
print 'The file contains the following variables:'
print acorn_DATA.variables.keys()
~~~

~~~ {.output}
The file contains the following variables:
[u'LATITUDE', u'LONGITUDE', u'LATITUDE_quality_control', u'LONGITUDE_quality_control', u'TIME', u'SPEED', u'UCUR', u'VCUR', u'TIME_quality_control', u'SPEED_quality_control', u'UCUR_quality_control', u'VCUR_quality_control']
~~~

(The 'u' means each variable name is represented by a Unicode string.)

~~~ {.python}
print 'These are the attributes of the time axis:'
print acorn_DATA.variables['TIME']
print 'These are some of the time values:'
print acorn_DATA.variables['TIME'][0:10]
~~~

~~~ {.output}
These are the attributes of the time axis:
<type 'netCDF4.Variable'>
float64 TIME(TIME)
    standard_name: time
    long_name: time
    units: days since 1950-01-01 00:00:00
    axis: T
    valid_min: 0.0
    valid_max: 999999.0
    _FillValue: -9999.0
    calendar: gregorian
    comment: Given time lays at the middle of the averaging time period.
    local_time_zone: 8.0
unlimited dimensions: 
current shape = (493,)
filling off

These are some of the time values:
[ 22919.          22919.04166667  22919.08333333  22919.125       22919.16666667
  22919.20833333  22919.25        22919.29166667  22919.33333333  22919.375     ]
~~~ 

The raw time values are fairly meaningless, 
but we can use the time attributes to convert them to a more meaningful format...

~~~ {.python}
from netCDF4 import num2date
units = acorn_DATA.variables['TIME'].units
calendar = acorn_DATA.variables['TIME'].calendar

times = num2date(acorn_DATA.variables['TIME'][:], units=units, calendar=calendar)
print times[0:10]
~~~

~~~ {.output}
[datetime.datetime(2012, 10, 1, 0, 0) datetime.datetime(2012, 10, 1, 1, 0)
 datetime.datetime(2012, 10, 1, 2, 0) datetime.datetime(2012, 10, 1, 3, 0)
 datetime.datetime(2012, 10, 1, 4, 0) datetime.datetime(2012, 10, 1, 5, 0)
 datetime.datetime(2012, 10, 1, 6, 0) datetime.datetime(2012, 10, 1, 7, 0)
 datetime.datetime(2012, 10, 1, 8, 0) datetime.datetime(2012, 10, 1, 9, 0)]
~~~

> ## Climate and Forecast (CF) metadata convention {.callout}
>
> When performing simple data analysis tasks on netCDF files, 
> command line tools like the Climate Data Operators 
> ([CDO](https://code.zmaw.de/projects/cdo))   
> are often a better alternative to writing your own functions in Python. 
> However, let's put ourselves in the shoes of the developers of CDO for a minute. 
> In order to calculate the time mean of a dataset for a given start and end date (for example), 
> CDO must first identify the units of the time axis. 
> This isn't as easy as you'd think, 
> since the creator of the netCDF file could easily have called the `units` attribute `measure`, 
> or `scale`, or something else completely unpredictable. 
> They could also have defined the units as `weeks since 1-01-01 00:00:00` or `milliseconds after 1979-12-31`. 
> Obviously what is needed is a standard method for defining netCDF attributes, 
> and that’s where the [Climate and Forecast (CF) metadata convention](http://cf-pcmdi.llnl.gov/) comes in.
>
> The CF metadata standard was first defined back in the early 2000s and has now been adopted by all the major institutions and projects in the weather/climate sciences. 
> There's a nice [blog post](http://drclimate.wordpress.com/2014/06/09/are-you-cf-compliant/) on the topic if you'd like more information, 
> but for the most part you just need to be aware that if a tool like CDO isn't working, 
> it might be because your netCDF file isn't CF compliant.

## Calculating the current speed

For the sake of example, 
let's say that our data file contained the zonal (east/west; 'UCUR') and meridional (north/south; 'VCUR') surface current components, 
but not the total current speed. 
To calculate it, 
we first need to assign a variable to the zonal and meridional current data.

~~~ {.python}
uData = acorn_DATA.variables['UCUR'][:,:,:]
vData = acorn_DATA.variables['VCUR'][:,:,:]
~~~

Both `uData` and `vData` are a special type of numpy array 
(which we have met previously) known as a masked array, 
whereby some of the points in the time/latitude/longitude grid have missing (or masked) values. 
Just as with a normal numpy array, 
we can check the shape of our data 
(in fact, masked arrays can do everything normal numpy arrays can do and more).

~~~ {.python}
print type(uData)
print uData.shape
~~~

~~~ {.output}
<class 'numpy.ma.core.MaskedArray'>
(493, 55, 57)
~~~

In other words, 493 time steps, 55 latitudes and 57 longitudes. 
We can now go ahead and calculate the current speed.

~~~ {.python}
spData = (uData**2 + vData**2)**0.5
~~~

### Viewing the result

It's a good idea to regularly view your data throughout the code development process, 
just to ensure nothing that crazy has happened along the way. 
Below is a code except from 
[this example](https://github.com/aodn/imos-user-code-library/blob/master/Python/demos/acorn.py) 
in the AODN user code library, 
which simply plots one of the 493 timesteps.

~~~ {.python}
%matplotlib inline
from matplotlib.pyplot import figure, pcolor, colorbar, xlabel, ylabel, title, draw, quiver, show, savefig

LAT = acorn_DATA.variables['LATITUDE']
LON = acorn_DATA.variables['LONGITUDE']
TIME = acorn_DATA.variables['TIME']

# Only one time value is being plotted. modify timeIndex if desired (value between 0 and length(timeData)-1 )
timeIndex = 4
speedData = spData[timeIndex,:,:]
latData = LAT[:]
lonData = LON[:]

# sea water U and V components
uData = acorn_DATA.variables['UCUR'][timeIndex,:,:]
vData = acorn_DATA.variables['VCUR'][timeIndex,:,:]
units = acorn_DATA.variables['UCUR'].units

figure1 = figure(figsize=(13, 10), dpi=80, facecolor='w', edgecolor='k')
pcolor(lonData , latData, speedData)
cbar = colorbar()
cbar.ax.set_ylabel('Current speed in ' + units)

title(acorn_DATA.title + '\n' + num2date(TIME[timeIndex], TIME.units, TIME.calendar).strftime('%d/%m/%Y'))
xlabel(LON.long_name + ' in ' + LON.units)
ylabel(LAT.long_name + ' in ' + LAT.units)

# plot velocity field
Q = quiver(lonData[:], latData[:], uData, vData, units='width')
show()
~~~

![Figure: surface current](fig/surface_current.svg)

> ## Plotting options {.callout}
>
> Quite a few lines of code were required to create our publication quality figure using [matplotlib](http://matplotlib.org/), 
> and there would have been even more had we wanted to use the [basemap](http://matplotlib.org/basemap/) library to plot coastlines or change the map projection. 
> Recognising this burden, 
> the team at the UK Met Office have developed [Iris](http://scitools.org.uk/iris/) and [Cartopy](http://scitools.org.uk/cartopy/), 
> which build on matplotlib and basemap to provide a more convenient interface 
> (read: shorter, less complex code) 
> for plotting in the weather, climate and ocean sciences.
>
> The easiest way to install Iris and Cartopy is to use the
> [conda](http://www.continuum.io/blog/conda) package installer that comes with Anaconda. 
> Simply enter the following at the command line:  
>  
> ~~~ 
> conda install -c scitools iris
> ~~~
>
> What if you want to view the contents of a netCDF file quickly, 
> rather than go to the effort of producing something that is publication quality?
> There are numerous tools out there for doing this, 
> including [Panoply](http://www.giss.nasa.gov/tools/panoply/) 
> and [UV-CDAT](http://uvcdat.llnl.gov/).

> ## Vectorisation {.challenge}
>
> Let's say (hypothetically) that the TURQ site radar has been found to be unreliable for surface current speeds greater than 0.9 m/s.
> To correct for this problem, 
> the IMOS documentation suggests setting all values greater than 0.9 m/s, back to 0.9 m/s. 
> The most obvious solution to this problem would be to loop through every element in `spData` and check its value:
>
> ~~~ {.python}
> for t in range(0, len(TIME[:])):
>     for y in range(0, len(LAT[:])):
>         for x in range(0, len(LON[:])):
>             if spData[t, y, x] > 0.9:
>                 spData[t, y, x] = 0.9
> ~~~
>
> The problem is that not only is this nested loop kind of ugly, it's also pretty slow. 
> If our data array was even larger 
> (e.g. like the huge data arrays that high resolution global climate models produce), 
> then it would probably be prohibitively slow.
> The reason is that *high level* languages like Python and Matlab are built for usability 
> (i.e. they make it easy to write concise, readable code), not speed. 
> To get around this problem, 
> people have written Python libraries (like `numpy`) in *low level* languages like C and Fortran, 
> which are built for speed (but definitely not usability). 
> Fortunately we don't ever need to see the C code under the hood of the `numpy` library, 
> but we should use it to [vectorise](http://en.wikipedia.org/wiki/Array_programming) our array operations whenever possible. 
> 
> With this in mind:
>
> * Use the `numpy.ma.where()` function to correct all values in `spData` greater than 0.9 (hint: you'll need to `import numpy.ma`)
> * Use the `%timeit` cell magic to compare the speed of your answer to the nested loop described above 