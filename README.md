# sigplot-data-service

This is a work in progress of a Sigplot data service (sds) that will provide some serverside data thining for sigplot applications. 

#### Suppoert files in repo
* makedata.py - a utility used to make 2D data files for the purpose of testing
* sds.ipynb - a Juypter notebook for interacting and tests the SDS that can request data and plot it.


## URL

The URL for this service is <host:port>/sds/<LocationName>/path/to/filename?mode=<mode>
  * LocationName needs to match one of the LocationDetails structs in the config file
  * Modes are currently "hdr" which returns a JSON of MIDAS header information and "rds" the Raster Data Service whose additional arguments are described below.
  * Examples: 'http://111.222.333.444:5555/sds/ServiceDirData/mydata_SL_500_1000.tmp?mode=hdr' - Header Mode
              'http://111.222.333.444:5555/sds/ServiceDir/mydata_SL_500_1000.tmp?mode=rds&x1=0&y1=0&x2=100&y2=100&outxsize=50&outysize=50&transform=mean



## RDS 

The SDS is intended to take 2D files and providing two methods to reduce the data sze of the files. First a sub section of the file can be specified so that only data from that subset can be returned instead of the enire file. Second, the selection can thinned or downsample to create a output that represents the same data but is smaller.  

First slide_data_from_file is called and returns only the subset of the file that was requested. It is assumed that the data and the selction are 2D. The sub selection is specified by two points (x1,y1) and (x2,y2) that represent the oposite points of a rectangle of the data selction.  

Second the data slice is passed into down_sample_data where the data is downsized to be of size (outxsize by outysize). This method supports several different transform types, mean, max, min, first, and absolute max. 

Currently the web service has one end point /sds that takes the following parameters:
  * x1 - x point for the first point of the selection rectangle. 
  * y1 - y point for the first point of the selection rectangle. 
  * x2 - x point for the second point of the selection rectangle. 
  * y2 - y point for the second point of the selection rectangle. 
  * outxsize - x size of the data output 
  * outysize - y size of the data output 
  * transform - transform to use to down sample data. Possible options 'max', 'min', 'mean', 'first', 'absmax'. 'mean2' is a different implmentation of mean that is usually faster. 
  * cxmode - Optional Parameter. Used if the inputfile is complex. Options are 'mag','phase','real','imag','10log','20log'. Default is 'mag'
  * outfmt - Optional Parameter. Used to change the output format from what the input file was. Default is to return the same type as input file. Options are "B", "I", "L", "F", "D", "RGBA". Type conversion support is limited, does not scale data, trucates decimal. In the case of "RGBA" the value is converted an RGB value using the colormap.
  * colormap - Optional Parameter. Color map names. Currently support "Greyscale", "RampColormap","ColorWheel","Spectrum"
  * zmin - Optional Parameter. Integer value used for RGB mode and sets the minimum value for the color map. Defaults to minimum value in selection.
  * zmax - Optional Parameter. Integer value used for RGB mode and sets the maximum value for the color map. Defaults to minimum value in selection.
  

## Juypter testing

The sds.ipynb jupyter notebook can be used to interact with the SDS and plot the 2D files and the sub selections that come back to SDS. It is assumed that Juypter can find the same data files in the same relative path as the server side application. This is only needed to test and compare the files before and after the sub selections. 

## UI Development Mode

```
cd ui
nvm use # assumes you have run nvm install at least once
SDS_URL="http://localhost:5055/sds" ROOT_URL="/ui/" ember serve
```

Now you can visit http://localhost:4200/ui/demo.

## Docker

The Docker version currently *MUST* be run behind an NGINX proxy rooted at /sigplot/

```
make docker

docker run -it --rm -p 5055:5055 sds:0.1
```
