---
layout: page
title: Data Management in the Ocean, Weather, and Climate Sciences
subtitle: Data Provenance
minutes: 20
---
> ## Learning Objectives {.objectives}
> * Record data processing steps in the history attribute of netCDF files
> * Include this provenance tracking in a personal data management plan

Now that we've developed some code 
for reading in zonal and meridional surface current data and calculating the speed, 
the logical next step is to put that code in a script called `calc_current_speed.py`
so that we can repeat the process quickly and easily.

The output of that script will be a new netCDF file containing the current speed data, 
however there's one more thing to consider before we go ahead and start creating new files. 
Looking closely at the global attributes of 
`IMOS_ACORN_V_20121001T000000Z_TURQ_FV00_monthly-1-hour-avg_END-20121029T180000Z_C-20121030T160000Z.nc` 
you can see that the entire history of the file, 
all the way back to its initial download, 
has been recorded in the `history` attribute.

~~~ {.python}
print acorn_DATA.history
~~~

~~~ {.output}
2012-10-09T03:31:35 Convert totl_TURQ_2012_10_08_1500.tuv to netcdf format using CODAR_Convert_File.
2012-10-09T03:31:36 Write CODAR file totl_TURQ_2012_10_08_1500.tuv. Modification of the NetCDF format by eMII to visualise the data using ncWMS
~~~

This practice of recording the history of the file ensures the 
[provenance](http://en.wikipedia.org/wiki/Provenance) of the data.
In other words, a complete record of everything that has been done to the data is stored with the data, 
which avoids any confusion in the event that the data is ever moved, 
passed around to different users,
or viewed by its creator many months later.  

If we want to create our own entry for the history attribute, 
we'll need to be able to create a: 

* Time stamp
* Record of what was entered at the command line in order to execute `calc_current_speed.py`
* Method of indicating which verion of the script was run (i.e. because the script is in our git repository)

### Time stamp

A library called `datetime` can be used to find out the time and date right now:

~~~ {.python}
import datetime
 
time_stamp = datetime.datetime.now().strftime("%Y-%m-%dT%H:%M:%S")
print time_stamp
~~~

~~~ {.output}
2015-02-17T20:26:25
~~~

The `strftime` function can be used to customise the appearance of a datetime object;
in this case we've made it look just like the other time stamps in our data file.

### Command line record

In the [lesson on command line programs](http://www.software-carpentry.org/v5/novice/python/06-cmdline.html) 
we met `sys.argv`, 
which contains all the arguments entered by the user at the command line:

~~~ {.python}
import sys
print sys.argv
~~~

~~~ {.output}
['-c', '-f', '/opt/ipython/profile_default/security/kernel-349ad03b-2b0e-46ef-98b0-39af6d92db4e.json', "--IPKernelApp.parent_appname='ipython-notebook'", '--profile-dir', '/opt/ipython/profile_default', '--parent=1']
~~~

In launching this IPython notebook,
you can see that a number of command line arguments were used. 
To join all these list elements up, 
we can use the `join` function that belongs to Python strings:

~~~ {.python}
args = " ".join(sys.argv)
print args
~~~

~~~ {.output}
-c -f /opt/ipython/profile_default/security/kernel-349ad03b-2b0e-46ef-98b0-39af6d92db4e.json --IPKernelApp.parent_appname='ipython-notebook' --profile-dir /opt/ipython/profile_default --parent=1
~~~

While this list of arguments is very useful, 
it doesn't tell us which Python installation was used to execute those arguments. 
The `sys` library can help us out here too:

~~~ {.python}
exe = sys.executable
print exe
~~~

~~~ {.output}
/opt/python/bin/python
~~~ 

### Git hash

In the Software Carpentry lessons on git we learned that each commit is associated with 
a unique 40-character identifier known as a hash. 
We can use the git Python library to get the hash associated with the script:

~~~ {.python}
from git import Repo
import os

git_hash = Repo(os.getcwd()).heads[0].commit
print git_hash
~~~

~~~ {.output}
765320b051cc5f9947f899637a2f38a8de3bcc3c
~~~

We can now put all this information together for our history entry: 

~~~ {.python}
entry = """%s: %s %s (Git hash: %s)""" %(time_stamp, exe, args, str(git_hash)[0:7])
print entry
~~~

~~~ {.output}
2015-02-17T20:26:25: /opt/python/bin/python -c -f /opt/ipython/profile_default/security/kernel-349ad03b-2b0e-46ef-98b0-39af6d92db4e.json --IPKernelApp.parent_appname='ipython-notebook' --profile-dir /opt/ipython/profile_default --parent=1 (Git hash: 765320b)
~~~

## Putting it all together

So far we've been experimenting in the IPython notebook to familiarise ourselves with 
`netCDF4` and the other Python libraries that might be useful for calculating the surface current speed.
We should now go ahead and write a script, 
so we can repeat the process with a single entry at the command line: 

~~~ {.input}
$ cat code/calc_current_speed.py
~~~

~~~ {.output}
import os, sys, argparse
import datetime
from git import Repo
from netCDF4 import Dataset


def calc_speed(u, v):
    """Calculate the speed"""

    speed = (u**2 + v**2)**0.5

    return speed


def copy_dimensions(infile, outfile):
    """Copy the dimensions of the infile to the outfile"""
        
    for dimName, dimData in infile.dimensions.iteritems():
        outfile.createDimension(dimName, len(dimData))


def copy_variables(infile, outfile):
    """Create variables corresponding to the file dimensions 
    by copying from infile"""
            
    for var_name in ['TIME', 'LATITUDE', 'LONGITUDE']:
        varin = infile.variables[var_name]
        outVar = outfile.createVariable(var_name, varin.datatype, 
                                        varin.dimensions, 
                                        fill_value=varin._FillValue)
        outVar[:] = varin[:]
            
        var_atts = {}
        for att in varin.ncattrs():
            if not att == '_FillValue':
                var_atts[att] = eval('varin.'+att) 
        outVar.setncatts(var_atts)


def create_history():
    """Create the new entry for the global history file attribute"""

    time_stamp = datetime.datetime.now().strftime("%Y-%m-%dT%H:%M:%S")
    exe = sys.executable
    args = " ".join(sys.argv)
    git_hash = Repo(os.getcwd()).heads[0].commit

    return """%s: %s %s (Git hash: %s)""" %(time_stamp, exe, args, str(git_hash)[0:7])


def read_data(ifile, uVar, vVar):
    """Read data from ifile corresponding to the U and V variable"""

    input_DATA = Dataset(ifile)
    uData = input_DATA.variables[uVar][:]
    vData = input_DATA.variables[vVar][:]

    return uData, vData, input_DATA


def set_global_atts(infile, outfile):
    """Set the global attributes for outfile.
        
    Note that the global attributes are simply copied from
    infile and the history attribute updated accordingly.
        
    """
        
    global_atts = {}
    for att in infile.ncattrs():
        global_atts[att] = eval('infile.'+att)  
        
    new_history = create_history()
    global_atts['history'] = """%s\n%s""" %(new_history,  global_atts['history'])
    outfile.setncatts(global_atts)


def write_speed(infile, outfile, spData):
    """Write the current speed data to outfile"""
        
    u = infile.variables['UCUR']   
    spcur = outfile.createVariable('SPCUR', u.datatype, u.dimensions, fill_value=u._FillValue)
    spcur[:,:,:] = spData    
    
    spcur.standard_name = 'sea_water_speed'
    spcur.long_name = 'sea water speed'
    spcur.units = u.units
    spcur.coordinates = u.coordinates
    

def main(inargs):
    """Run the program"""
    
    inFile = inargs.infile
    uVar = inargs.uvar
    vVar = inargs.vvar
    outfile_name = inargs.outfile

    # Read input data 
    uData, vData, input_DATA = read_data(inFile, uVar, vVar)
    
    # Calculate the current speed
    spData = calc_speed(uData, vData)
    
    # Write the output file
    outfile = Dataset(outfile_name, 'w', format='NETCDF4')
    set_global_atts(input_DATA, outfile)
    copy_dimensions(input_DATA, outfile)
    copy_variables(input_DATA, outfile)
    write_speed(input_DATA, outfile, spData) 
    
    outfile.close()


if __name__ == '__main__':

    extra_info ="""example:
  python calc_current_speed.py http://thredds.aodn.org.au/thredds/dodsC/IMOS/eMII/demos/ACORN/monthly_gridded_1h-avg-current-map_non-QC/TURQ/2012/IMOS_ACORN_V_20121001T000000Z_TURQ_FV00_monthly-1-hour-avg_END-20121029T180000Z_C-20121030T160000Z.nc.gz UCUR VCUR IMOS_ACORN_SPCUR_20121001T000000Z_TURQ_monthly-1-hour-avg_END-20121029T180000Z.nc 

author:
  Damien Irving, irving.damien@gmail.com

"""

    description='Calculate the current speed'
    parser = argparse.ArgumentParser(description=description,
                                     epilog=extra_info,
                                     formatter_class=argparse.RawDescriptionHelpFormatter)

    parser.add_argument("infile", type=str, help="Input file name")
    parser.add_argument("uvar", type=str, help="Name of the zonal flow variable")
    parser.add_argument("vvar", type=str, help="Name of the meridional flow variable")
    parser.add_argument("outfile", type=str, help="Output file name")

    args = parser.parse_args()            

    print 'Input file: ', args.infile
    print 'Output file: ', args.outfile  

    main(args)
~~~

> ## Introducing xray {.callout}
>
> It took four separate functions in `calc_current_speed.py` to create the output file,
> because we had to copy the dimensions 
> and most of the global and variable attributes 
> from the original file to the new file. 
> This is such a common problem that a Python library called 
> [xray](http://xray.readthedocs.org/en/stable/) has been developed, 
> which conserves metadata whenever possible. 
> When xray is used to read a netCDF file, 
> the data are stored as an `xray.DataArray` (as opposed to a `numpy.ndarray`).
> These special data arrays carry their dimension information and variable atributes with them, 
> which means you don't have to retrieve them manually. 
> xray also comes with a bunch of convenience functions 
> for doing typical weather/climate/ocean tasks (calculating climatologies, anomalies, etc),
> which can be a pain using numpy. 
>
> Similar to Iris and Cartopy, 
> the easiest way to install xray is with the conda package installer. 
> Simply type the following at the command line:  
>
> ~~~
> conda install xray dask netCDF4 bottleneck
> ~~~
>

Using the help information that the `argparse` library provides, 
we can now go ahead and run this script:

~~~ {.input}
$ python calc_current_speed.py -h
~~~

~~~ {.output}
usage: calc_current_speed.py [-h] infile uvar vvar outfile

Calculate the current speed

positional arguments:
  infile      Input file name
  uvar        Name of the zonal flow variable
  vvar        Name of the meridional flow variable
  outfile     Output file name

optional arguments:
  -h, --help  show this help message and exit

example:
  python calc_current_speed.py http://thredds.aodn.org.au/thredds/dodsC/IMOS/eMII/demos/ACORN/monthly_gridded_1h-avg-current-map_non-QC/TURQ/2012/IMOS_ACORN_V_20121001T000000Z_TURQ_FV00_monthly-1-hour-avg_END-20121029T180000Z_C-20121030T160000Z.nc.gz UCUR VCUR IMOS_ACORN_SPCUR_20121001T000000Z_TURQ_monthly-1-hour-avg_END-20121029T180000Z.nc 

author:
  Damien Irving, irving.damien@gmail.com
~~~

~~~ {.input}
$ python calc_current_speed.py http://thredds.aodn.org.au/thredds/dodsC/IMOS/eMII/demos/ACORN/monthly_gridded_1h-avg-current-map_non-QC/TURQ/2012/IMOS_ACORN_V_20121001T000000Z_TURQ_FV00_monthly-1-hour-avg_END-20121029T180000Z_C-20121030T160000Z.nc.gz UCUR VCUR IMOS_ACORN_SPCUR_20121001T000000Z_TURQ_monthly-1-hour-avg_END-20121029T180000Z.nc 
~~~

~~~ {.output}
Input file:  http://thredds.aodn.org.au/thredds/dodsC/IMOS/eMII/demos/ACORN/monthly_gridded_1h-avg-current-map_non-QC/TURQ/2012/IMOS_ACORN_V_20121001T000000Z_TURQ_FV00_monthly-1-hour-avg_END-20121029T180000Z_C-20121030T160000Z.nc.gz
Output file:  IMOS_ACORN_SPCUR_20121001T000000Z_TURQ_monthly-1-hour-avg_END-20121029T180000Z.nc
~~~

## The finished product

We can now inspect the attributes in our new file:

~~~ {.input}
$ ncdump -h IMOS_ACORN_SPCUR_20121001T000000Z_TURQ_monthly-1-hour-avg_END-20121029T180000Z.nc
~~~

~~~ {.output}
netcdf IMOS_ACORN_SPCUR_20121001T000000Z_TURQ_monthly-1-hour-avg_END-20121029T180000Z {
dimensions:
	I = 55 ;
	J = 57 ;
	TIME = 493 ;
variables:
	double TIME(TIME) ;
		TIME:_FillValue = -9999. ;
		TIME:comment = "Given time lays at the middle of the averaging time period." ;
		TIME:valid_min = 0. ;
		TIME:long_name = "time" ;
		TIME:standard_name = "time" ;
		TIME:local_time_zone = 8. ;
		TIME:units = "days since 1950-01-01 00:00:00" ;
		TIME:calendar = "gregorian" ;
		TIME:valid_max = 999999. ;
		TIME:axis = "T" ;
	double LATITUDE(I, J) ;
		LATITUDE:_FillValue = 9999. ;
		LATITUDE:valid_min = -90. ;
		LATITUDE:reference_datum = "geographical coordinates, WGS84 projection" ;
		LATITUDE:long_name = "latitude" ;
		LATITUDE:standard_name = "latitude" ;
		LATITUDE:units = "degrees_north" ;
		LATITUDE:valid_max = 90. ;
		LATITUDE:axis = "Y" ;
	double LONGITUDE(I, J) ;
		LONGITUDE:_FillValue = 9999. ;
		LONGITUDE:valid_min = -180. ;
		LONGITUDE:reference_datum = "geographical coordinates, WGS84 projection" ;
		LONGITUDE:long_name = "longitude" ;
		LONGITUDE:standard_name = "longitude" ;
		LONGITUDE:units = "degrees_east" ;
		LONGITUDE:valid_max = 180. ;
		LONGITUDE:axis = "X" ;
	float SPCUR(TIME, I, J) ;
		SPCUR:_FillValue = 9999.f ;
		SPCUR:standard_name = "sea_water_speed" ;
		SPCUR:long_name = "sea water speed" ;
		SPCUR:units = "m s-1" ;
		SPCUR:coordinates = "TIME LATITUDE LONGITUDE" ;

// global attributes:
		:comment = "These data have not been quality controlled. They represent values calculated using software provided by CODAR Ocean Sensors. The file has been modified by eMII in order to visualise the data using the ncWMS software.This NetCDF file has been created using the IMOS NetCDF User Manual v1.2. A copy of the document is available at http://imos.org.au/facility_manuals.html" ;
		:distribution_statement = "Data may be re-used, provided that related metadata explaining the data has been reviewed by the user, and the data is appropriately acknowledged. Data, products and services from IMOS are provided \"as is\" without any warranty as to fitness for a particular purpose." ;
		:geospatial_vertical_max = 0. ;
		:featureType = "grid" ;
		:abstract = "These data have not been quality controlled. The ACORN facility is producing NetCDF files with radials data for each station every ten minutes.  Radials represent the surface sea water state component  along the radial direction from the receiver antenna  and are calculated from the shift of an area under  the bragg peaks in a Beam Power Spectrum.  The radial values have been calculated using software provided  by the manufacturer of the instrument. eMII is using a Matlab program to read all the netcdf files with radial data for two different stations  and produce a one hour average product with U and V components of the current. The final product is produced on a regular geographic (latitude longitude) grid More information on the data processing is available through the IMOS MEST  http://imosmest.aodn.org.au/geonetwork/srv/en/main.home. This file is an aggregation over a month of distinct hourly averaged files." ;
		:citation = "Citation to be used in publications should follow the format:\"IMOS.[year-of-data-download],[Title],[Data access URL],accessed [date-of-access]\"" ;
		:geospatial_lat_units = "degrees_north" ;
		:geospatial_lon_units = "degrees_east" ;
		:data_center = "eMarine Information Infrastructure (eMII)" ;
		:keywords = "Oceans" ;
		:id = "IMOS/ACORN/au/TURQ/2012-10-08T15:00:00Z.sea_state" ;
		:naming_authority = "Integrated Marine Observing System (IMOS)" ;
		:institution_references = "http://www.imos.org.au/acorn.html" ;
		:geospatial_lat_max = -29.5252994, -29.5794147, -29.6335296, -29.687644, -29.741758, -29.7958715, -29.8499845, -29.904097, -29.9582091, -30.0123207, -30.0664318, -30.1205425, -30.1746527, -30.2287624, -30.2828717, -30.3369805, -30.3910888, -30.4451966, -30.499304, -30.5534109, -30.6075173, -30.6616232, -30.7157287, -30.7698337, -30.8239382, -30.8780423, -30.9321458, -30.9862489, -31.0403515, -31.0944536, -31.1485553, -31.2026564, -31.2567571, -31.3108573, -31.364957, -31.4190563, -31.473155, -31.5272533, -31.5813511, -31.6354484, -31.6895452, -31.7436416, -31.7977374, -31.8518328, -31.9059277, -31.960022, -32.0141159, -32.0682094, -32.1223023, -32.1763947, -32.2304867, -32.2845781, -32.3386691, -32.3927596, -32.4468495 ;
		:acknowledgment = "Data was sourced from the Integrated Marine Observing System (IMOS) - IMOS is supported by the Australian Government through the National Collaborative Research Infrastructure Strategy (NCRIS) and the Super Science Initiative (SSI)." ;
		:title = "IMOS ACORN Turquoise Coast site (WA) (TURQ), monthly aggregation of one hour averaged current data" ;
		:author_email = "laurent.besnard@utas.edu.au" ;
		:netcdf_version = "3.6" ;
		:site_code = "TURQ, Turqoise Coast" ;
		:file_version = "Level 0 - Raw data" ;
		:instrument = "CODAR Ocean Sensors/SeaSonde" ;
		:file_version_quality_control = "Data in this file has not been quality controlled" ;
		:acknowledgement = "Data was sourced from Integrated Marine Observing System (IMOS) - an initiative of the Australian Government being conducted as part of the National Calloborative Research Infrastructure Strategy." ;
		:time_coverage_duration = "PT1H19M" ;
		:local_time_zone = 8. ;
		:geospatial_lat_min = -29.5386582, -29.5927878, -29.6469169, -29.7010456, -29.7551738, -29.8093016, -29.863429, -29.9175559, -29.9716823, -30.0258084, -30.0799339, -30.1340591, -30.1881837, -30.242308, -30.2964318, -30.3505551, -30.404678, -30.4588004, -30.5129224, -30.5670439, -30.621165, -30.6752857, -30.7294059, -30.7835256, -30.8376449, -30.8917637, -30.9458821, -31., -31.0541175, -31.1082345, -31.162351, -31.2164671, -31.2705828, -31.324698, -31.3788127, -31.432927, -31.4870408, -31.5411542, -31.5952671, -31.6493795, -31.7034915, -31.7576031, -31.8117141, -31.8658247, -31.9199349, -31.9740446, -32.0281538, -32.0822625, -32.1363708, -32.1904787, -32.244586, -32.2986929, -32.3527994, -32.4069054, -32.4610109 ;
		:time_coverage_start = "2012-10-01T00:00:00Z" ;
		:geospatial_vertical_min = 0. ;
		:time_coverage_end = "2012-10-29T18:00:00Z" ;
		:data_center_email = "info@emii.org.au" ;
		:institution = "Australian Coastal Ocean Radar Network (ACORN)" ;
		:geospatial_lon_max = 115.7758038, 115.7766744, 115.7775469, 115.7784212, 115.7792975, 115.7801756, 115.7810557, 115.7819376, 115.7828215, 115.7837073, 115.784595, 115.7854846, 115.7863762, 115.7872697, 115.7881651, 115.7890625, 115.7899618, 115.7908631, 115.7917663, 115.7926715, 115.7935787, 115.7944878, 115.7953989, 115.796312, 115.7972271, 115.7981441, 115.7990632, 115.7999843, 115.8009074, 115.8018325, 115.8027596, 115.8036887, 115.8046199, 115.8055531, 115.8064883, 115.8074256, 115.8083649, 115.8093063, 115.8102497, 115.8111952, 115.8121427, 115.8130924, 115.8140441, 115.8149979, 115.8159538, 115.8169118, 115.8178719, 115.8188341, 115.8197984, 115.8207648, 115.8217334, 115.822704, 115.8236768, 115.8246518, 115.8256289 ;
		:geospatial_lon_min = 112.3100111, 112.3090067, 112.3080002, 112.3069914, 112.3059805, 112.3049674, 112.3039521, 112.3029346, 112.3019149, 112.3008929, 112.2998688, 112.2988424, 112.2978139, 112.2967831, 112.29575, 112.2947147, 112.2936772, 112.2926374, 112.2915953, 112.290551, 112.2895044, 112.2884556, 112.2874044, 112.286351, 112.2852953, 112.2842373, 112.283177, 112.2821143, 112.2810494, 112.2799821, 112.2789125, 112.2778406, 112.2767663, 112.2756897, 112.2746108, 112.2735294, 112.2724458, 112.2713597, 112.2702713, 112.2691805, 112.2680873, 112.2669917, 112.2658937, 112.2647933, 112.2636905, 112.2625853, 112.2614777, 112.2603676, 112.2592551, 112.2581402, 112.2570228, 112.2559029, 112.2547806, 112.2536558, 112.2525286 ;
		:data_centre_email = "info@emii.org.au" ;
		:date_modified = "2012-10-30T16:31:50Z" ;
		:principal_investigator = "Wyatt, Lucy" ;
		:data_centre = "eMarine Information Infrastructure (eMII)" ;
		:author = "Besnard, Laurent" ;
		:Conventions = "IMOS version 1.3" ;
		:project = "Integrated Marine Observing System (IMOS)" ;
		:quality_control_set = "1" ;
		:source = "Terrestrial HF radar" ;
		:ssr_Stations = "SeaBird (SBRD), Cervantes (CRVT)" ;
		:date_created = "2012-10-30T16:31:50Z" ;
		:geospatial_vertical_units = "m" ;
		:history = "2015-02-17T20:26:35: /opt/python/bin/python calc_current_speed.py http://thredds.aodn.org.au/thredds/dodsC/IMOS/eMII/demos/ACORN/monthly_gridded_1h-avg-current-map_non-QC/TURQ/2012/IMOS_ACORN_V_20121001T000000Z_TURQ_FV00_monthly-1-hour-avg_END-20121029T180000Z_C-20121030T160000Z.nc.gz UCUR VCUR IMOS_ACORN_SPCUR_20121001T000000Z_TURQ_monthly-1-hour-avg_END-20121029T180000Z.nc (Git hash: 765320b)\n2012-10-09T03:31:35 Convert totl_TURQ_2012_10_08_1500.tuv to netcdf format using CODAR_Convert_File.\n2012-10-09T03:31:36 Write CODAR file totl_TURQ_2012_10_08_1500.tuv. Modification of the NetCDF format by eMII to visualise the data using ncWMS" ;
}
~~~

There are a lot of global and variable attributes in our output file, 
because we copied everything from the input file. 
It might be tempting to edit `calc_current_speed.py` or use the `ncatted` command line utility 
(which, like `ncdump`, comes with the [NCO](http://nco.sourceforge.net/) install) 
to start cleaning up (i.e. delete) some of these attributes, 
but we should resist that urge. 
The `id` global attribute, for instance, makes little sense to anyone who doesn't work at IMOS. 
While this might seem like a reasonable argument for deleting that attribute, 
once an attribute is deleted it's gone forever. 
The `id` attribute is taking up a negligible amount of memory, 
so why not just leave it there just in case?
When in doubt, keep metadata.

> ## A provenance plan {.challenge}
>
> Does your data management plan from the first challenge adequately address this issue of data provenance? 
> If not, go ahead and add to your plan now. 
> Things to consider include:
>
> * How to capture and store metadata relating to output files that aren't self describing (i.e. unlike `.nc` files, formats like `.csv` or `.png` don't store things like global and variable attributes within them) 
>
> Discuss the additions you've made to your plan with your partner.