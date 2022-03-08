Data extraction - NASA climate plaintext data
A set of examples on how to extract machine-readable data from the raw, official sources. No pandas needed, just requests and regex and xlrd (for Excel spreadsheets)

(in progress)

File system setup
from os import makedirs
from os.path import dirname, join
DATA_DIR = join('data', 'climate')
import csv
import re
import requests
from xlrd import open_workbook
Global Surface Air Temperature Anomaly, via NASA
NASA GISS

Source: NASA Goddard Institute for Space Studies (GISS) Surface Temperature Analysis.

Our traditional analysis using only meteorological station data is a line plot of global annual-mean surface air temperature change, with the base period 1951-1980, derived from the meteorological station network [This is an update of Plate 6(b) in Hansen et al. (2001).] Uncertainty bars (95% confidence limits) are shown for both the annual and five-year means, account only for incomplete spatial sampling of data.

The data file
Direct link to the source data file: http://data.giss.nasa.gov/gistemp/graphs_v3/Fig.A.txt
Mirror: data/climate/raw/nasa-gistemp-figA.txt
The contents: From 1880 to 2015, the change in global average surface air temperature change, compared to the average global temperature measured in the period 1951 to 1980.

An excerpt of the file:

Global Surface Air Temperature Anomaly (C) (Base: 1951-1980)
------------------------------------------------------------
 Year  Annual_Mean 5-year_Mean
--------------------------------
 1880     -0.49         *
 1881     -0.47         *
 1882     -0.38     -0.48
 1883     -0.39     -0.48
 1884     -0.67     -0.52
 1885     -0.51     -0.58
...
 2012      0.80      0.86
 2013      0.84      0.87
 2014      0.90         *
 2015      1.01         *
The years 1885 and 2015 are said to have a global average temperature of -0.51 and +1.01 degrees Celsius, respectively, from the average temperature as measured in the period of 1951-1980.

The 5-year mean of 1882 -- -0.48 -- is the rolling average of the annual means for 1880 through 1884.

Parsing and wrangling the temperature text file
This can be done with using regular expressions and re.findall(). In the snippet below, I write two files, since there aren't 5-year mean values for every annual mean value:

data/climate/cleaned/nasa-gistemp-annual-mean.csv
data/climate/cleaned/nasa-gistemp-5year-mean.csv
# writing the annual means to file
source_data_url = 'http://data.giss.nasa.gov/gistemp/graphs_v3/Fig.A.txt'
resp = requests.get(source_data_url)
pattern = re.compile(r'^ +(\d{4}) +(-?\d+\.\d+)', re.MULTILINE)
data = re.findall(pattern, resp.text)

# write to file
destfilename = join(DATA_DIR, 'extracted', 'nasa-gistemp-annual-mean.csv')
makedirs(dirname(destfilename), exist_ok=True)
with open(destfilename, 'w') as f:
    c = csv.writer(f)
    c.writerow(['year', 'annual_mean'])
    c.writerows(data)
# writing the 5-year means to file
# assume that `resp` still contains the download
pattern = re.compile(r'^ +(\d{4}) .+?(-?\d+\.\d+) *$', re.MULTILINE)
data = re.findall(pattern, resp.text)

# write to file
destfilename = join(DATA_DIR,  'extracted', 'nasa-gistemp-5year-mean.csv')
makedirs(dirname(destfilename), exist_ok=True)
with open(destfilename, 'w') as f:
    c = csv.writer(f)
    c.writerow(['year', 'fiveyear_mean'])
    c.writerows(data)
NASA CO2 gases
Source...?: NASA GISS: Forcings in GISS Climate Model.

I'm not really sure what the landing page for the following data set comes from. It seems to consist of data from:

NOAA's World Data Center for Paleoclimatology's ice core research.
NASA measurements from Mauna Loa
NOAA/ESRL measurements
The data file
Direct link to the source file: http://data.giss.nasa.gov/modelforce/ghgases/Fig1A.ext.txt
Mirror: data/climate/raw/nasa-ghgases-fig1A.ext.txt
The observed global average of carbon dioxide gas in parts-per million.

An excerpt of the file:

               Global Mean CO2 Mixing Ratios (ppm): Observations
----------------------------------------------------------------------------------
Data                                     Data
Source  Year  MixR          Yar   MixR   Source Year  MixR          Year  MixR
----------------------------------------------------------------------------------
Ice-    1850  285.2         1900  295.7         1950  311.3         2000  369.64
Core    1851  285.1         1901  296.2         1951  311.8         2001  371.15
Data    1852  285.0         1902  296.6         1952  312.2         2002  373.15
Adjus-  1853  285.0         1903  297.0         1953  312.6         2003  375.64
ted     1854  284.9         1904  297.5         1954  313.2   NOAA/ 2004  377.44
for     1855  285.1         1905  298.0         1955  313.7   ESRL/ 2005  379.46
Global  1856  285.4         1906  298.4         1956  314.3  trends 2006  381.59
Mean    1857  285.6         1907  298.8         1957  314.8  change 2007  383.37
        1858  285.9         1908  299.3  SIO    1958  315.34  added 2008  385.46
        1859  286.1         1909  299.7  Mauna  1959  316.18     to 2009  386.95
        1860  286.4         1910  300.1  Loa    1960  317.07   2003 2010  389.21
        1861  286.6         1911  300.6    &    1961  317.73   data 2011  391.15
        1862  286.7         1912  301.0  South  1962  318.43
        1863  286.8         1913  301.3  Pole   1963  319.08
        1864  286.9         1914  301.4  Adjus- 1964  319.65
        1865  287.1         1915  301.6  ted    1965  320.23
        1866  287.2         1916  302.0  for    1966  321.59
        1867  287.3         1917  302.4  Global 1967  322.31
        1868  287.4         1918  302.8  Mean   1968  323.04
        1869  287.5         1919  303.0         1969  324.23
Parsing and wrangling the global gases file
As in the temperatures-file example, just adroit use of regular expressions. However, there's one wrinkle: the data file contains two sections; observations, as excerpted above, and "Future Scenarios":

             Global Mean CO2 Mixing Ratio (ppm): Future Scenarios
----------------------------------------------------------------------------------
           Alternative Scenario                 2 Degree C Scenario
        Year  MixR      Year  MixR          Year  MixR      Year  MixR
----------------------------------------------------------------------------------
        2000  370.0     2050  445.0         2000  370.0     2050  486.2
        2001  371.7     2051  446.2         2001  371.7     2051  489.2
        2002  373.4     2052  447.4         2002  373.4     2052  492.1
        2003  375.1     2053  448.5         2003  375.1     2053  494.9
We want to wrangle only the data before the future scenarios section:

# writing the gases data to file
source_data_url = 'http://data.giss.nasa.gov/modelforce/ghgases/Fig1A.ext.txt'
resp = requests.get(source_data_url)
txt = resp.text
stop_line_num = [x.strip() for x in txt.splitlines()].index('Global Mean CO2 Mixing Ratio (ppm): Future Scenarios')
# just get the lines before the unwanted Future Scenarios section
excerpt = "\n".join(txt.splitlines()[:stop_line_num])
pattern = re.compile(r'(\b\d{4})  (\d{3}\.\d{1,2}\b)')
data = re.findall(pattern, excerpt)

# I like non-mutating operations
sorteddata = sorted(data, key=lambda d: int(d[0]))
# write to file
destfilename = join(DATA_DIR, 'extracted', 'nasa-ghgases-co2-mean.csv')
makedirs(dirname(destfilename), exist_ok=True)
with open(destfilename, 'w') as f:
    c = csv.writer(f)
    c.writerow(['year', 'co2_ppm_mean'])
    c.writerows(sorteddata)
