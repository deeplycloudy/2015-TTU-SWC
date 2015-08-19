---
layout: page
title: Data Management in the Ocean, Weather, and Climate Sciences
subtitle: Data Management
minutes: 20
---
> ## Learning Objectives {.objectives}
>
> * Develop a personal data management plan 

In this lesson we are going to process some data collected by 
Australia's Integrated Marine Observing System ([IMOS](http://www.imos.org.au/)).

First off, let's load our data:

~~~ {.python}
from netCDF4 import Dataset
acorn_URL = 'http://thredds.aodn.org.au/thredds/dodsC/IMOS/eMII/demos/ACORN/monthly_gridded_1h-avg-current-map_non-QC/TURQ/2012/IMOS_ACORN_V_20121001T000000Z_TURQ_FV00_monthly-1-hour-avg_END-20121029T180000Z_C-20121030T160000Z.nc.gz'
acorn_DATA = Dataset(acorn_URL) 
~~~

The first thing to notice is the distinctive Data Reference Syntax (DRS) associated with the file. 
The staff at IMOS have archived the data according to the following directory structure:

~~~
http://thredds.aodn.org.au/thredds/dodsC/<project>/<organisation>/<collection>/<facility>/<data-type>/<site-code>/<year>/
~~~

From this we can deduce,
without even inspecting the contents of the file,
that we have data from the IMOS project that is run by the eMarine Information Infrastructure (eMII). 
It was collected in 2012 at the Turquoise Coast, 
Western Australia (TURQ) site of the Australian Coastal Ocean Radar Network (ACORN),
which is a network of high frequency radars that measure the ocean surface current
(see [this page](http://researchdata.ands.org.au/imos-acorn-turquoise-australia-australia/442676) 
on the Research Data Australia website for a nice overview of the dataset). 

The data type has a sub-DRS of its own, 
which tells us that the data represents the 1-hourly average surface current for a single month (October 2012), 
and that it is archived on a regularly spaced spatial grid and has not been quality controlled. 
The file is located in the "demos" directory, 
as it has been generated for the purpose of providing an example for users in the very helpful 
[Australian Ocean Data Network](http://portal.aodn.org.au/aodn/) (AODN) 
[user code library](https://github.com/aodn/imos-user-code-library).

Just in case the file gets separated from this informative directory stucture, 
much of the information is repeated in the file name itself, 
along with some more detailed information about the start and end time of the data, 
and the last time the file was modified.

~~~
<project>_<facility>_V_<time-start>_<site-code>_FV00_<data-type>_<time-end>_<modified>.nc.gz
~~~

In the first instance this level of detail seems like a bit of overkill, 
but consider the scope of the IMOS data archive. 
It is the final resting place for data collected by the entire national array of oceanographic observing equipment in Australia, 
which monitors the open oceans and coastal marine environment covering physical, chemical and biological variables. 
Since the data are so well labelled, 
locating all monthly timescale ACORN data from the Turquoise Coast and Rottnest Shelf sites 
(which represents hundreds of files) 
would be as simple as typing the following at the command line: 

~~~ {.input}
$ ls */ACORN/monthly_*/{TURQ,ROT}/*/*.nc
~~~

While it's unlikely that your research will ever involve cataloging data from such a large observational network, 
it's still a very good idea to develop your own personal DRS for the data you do have. 
This often involves investing some time at the beginning of a project to think carefully about the design of your directory and file name structures, 
as these can be very hard to change later on 
(a good example is [the DRS](http://cmip-pcmdi.llnl.gov/cmip5/docs/cmip5_data_reference_syntax.pdf) 
used by the Climate Model Intercomparison Project). 
The combination of bash shell wildcards and a well planned DRS is one of the easiest ways to make your research more efficient and reliable.

> ## Data management plan {.challenge}
>
> We haven't even looked inside our IMOS data file and 
> already we have the beginnings of a detailed data management plan. 
> The first step in any research project should be to develop such a plan, 
> so for this challenge we are going to turn back time. 
> If you could start your current research project all over again, 
> what would your data management plan look like? 
> Things to consider include:  
> 
> * Data Reference Syntax
> * How long it will take to obtain the data
> * Storage and backup (here's a [post](http://drclimate.wordpress.com/2013/04/16/backing-up-your-work/) with some backup ideas)
>
> Write down and discuss your plan with your partner.