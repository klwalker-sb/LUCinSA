# Configuring eosvault
===============================================================================================================================

eosvault is used to download Earth observation satellite imagery from Google Earth Engine and and post-process this imagery externally. You can learn [more about eosvault here](https://github.com/jgrss/eosvault)

## 1. configure your Google SDK
To be able to ingest imagery from Google Earth Engine, you first need to set up your Google SDK and Google Earth Engine.
Follow the five steps (and links) of [this Google Earth Engine Quickstart](https://developers.google.com/earth-engine/reference/Quickstart). Make sure you use your UCSB email for your Google Earth Engine account.

**Get a private key file** for the service account you created in step 5 of the Google Earth Engine Quickstart. 

| 1. Click new service account in Google Cloud Platform       | 2.Click on 3 vertical dots, then "Manage Keys" | 3. Click "Add Key", then "Create new Key"                     |
| :---------------------------------: | :----------------------------------: | :-------------------------------------: |
| ![](/Images/GEE_ServiceAcct0.png)   | ![](/Images/GEE_ServiceAcct1.png)    | ![](/Images/GEE_ServiceAcct2.png)       |
**Then download the .json key file immeadiately**
:::{admonition}You will only have the option to download the .json key file at the moment you create the key
You need this file to proceed. (If you miss the opportunity, you can create a new key)
:::

## 2. upload your Google SDK key file to your cluster space
Transfer the .json key file into a chosen directory on your cluster space (i.e. ~/.eosvault), via one of these [file transfer options](Transfering).

### To check that your GEE configuration is working:
This can be done in iPython (which you already installed), which is a Python interpreter that allows small Python codes to be run without having to go through SLURM (the cluster's computing manager)
```
#Activate virtual environment first, then ipython:
source .nasaenv/bin/activate
ipython
In [1]: from google.auth.transport.requests import AuthorizedSession
In [2]: from google.oauth2 import service_account
# make sure to **edit your key path** in the command below:
In [3]: credentials = service_account.Credentials.from_service_account_file('/home/<username>/.eosvault/<yourkey>.json')
In [4]: scoped_credentials = credentials.with_scopes(['https://www.googleapis.com/auth/cloud-platform'])
In [5]: session = AuthorizedSession(scoped_credentials)
In [6]: url = 'https://earthengine.googleapis.com/v1alpha/projects/earthengine-public/assets/LANDSAT'
In [7]: response = session.get(url)
In [8]: from pprint import pprint
In [9]: import json
In [10]: pprint(json.loads(response.content))
# to Exit ipython:
In [x]: exit
```
If everything is configured correctly, the response before exiting should look something like this:
```
{'id': 'LANDSAT',
 'name': 'projects/earthengine-public/assets/LANDSAT',
 'type': 'FOLDER'}
```
## 3. Create a NASA Earthdata key
A NASA Earthdata secret key/code file pair is needed to download data from the Nasa Earthdata repository. These can be generated  through Geowombat (which you already installed in your Python environment) through IPython (the Python interpreter that you also already installed in your Python environment):
```
# From inside the directory where you want your keys (i.e. cd ~/.eosvault)
# Activate your venv
source .nasaenv/bin/activate
# open an IPython session
(.nasaenv) ipython
# the terminal window should then look something like
In [1]: 
# run the passkey code example
In [1]: from geowombat.data import PassKey
In [2]: pk = PassKey()
In [3]: pk.create_key('nasaearth.key')
In [4]: pk.create_passcode('nasaearth.key', 'nasaearth.code')
#This will prompt a password entry
In [5] exit
# Deactivate your venv
(.nasaenv) deactivate
```

## 4. Copy and edit the config.yaml file
**Copy the config.yaml template to the project/config directory on your space**
```
#Create project/config directory if it doesn't already exist
mkdir -p ~/project/config
cp /jad-cel/sandbox-cel/paraguay_lc/templates/eosvault_config_eri_pry.yaml ~/project/config/config_eri.yaml
```
**Open the template in the vim editor** 
```
vim ~/project/config/config_eri.yaml
```
To edit a line, type `i`, edit as desired, then hit `Esc` key and type `:wq`. Hit `Enter` key. For help editing in Vim, [see Vim Commands](VimCommands).
**Edit the following lines to match your paths:** 
* `Google secret key` (line 13 below): Edit path to match that of the secret key you uploaded in step #2 above
* `angles` (lines 17-18 of the template below): Edit path to match your own. First you need to **extract the angle files** into the corresponding directory: 
  ```
    #make the directory if it doesn't alreay exist:
    mkdir -p ~/code/bin/
    #move the ESPA.tar.gz file into the directory prior to extracting:
    mv ~/tmp/eosvault/files/ESPA.tar.gz ~/code/bin/ESPA.tar.gz
    #Navigate to the new directory 
    cd ~/code/bin/
    extract the angle files:
    tar -xzvf ESPA.tar.gz
    ```
* `Nasa Earth data`: In line 67, modify your username. In lines 68 and 69, modify the path to where you put your Nasa Earth key and code in step 3.

config_eri.yaml template (with added line #s for reference):
```
1-  project_path: '/jad-cel/sandbox-cel/paraguay_lc/raster/grids'
2-    # Root directory for the GCP index files
3-  index_dir: '/jad-cel/sandbox-cel/paraguay_lc/raster'
4-  
5-  # Directory to save text files of incomplete batch jobs
6-  log_dir: '/home/<username>/code/bash'
7-  
8-  # Length (in meters) on each side of grid
9-  grid_size: 20000
10-  
11- google:
12- 
13-  secret_key: '/home/<username>/.eosvault/<key>.json'
14- 
15- angles:
16- 
17-   l57_angles_path: '/home/<username>/code/bin/ESPA/landsat_angles'
18-   l8_angles_path: '/home/<username>/code/bin/ESPA/l8_angles'
19-   subsample: 10
20-   resampling: 'bilinear'
21- 
22- sixs:
23- 
24-   coarse_res: 100.0
25- 
26- io:
27- 
28-   # netcdf | geotiff
29-   file_format: 'netcdf'
30-   chunks: 512
31-   resampling: 'cubic'
32-   num_threads: 2
33-   nodataval: 65535
34- 
35-   to_netcdf_kwargs:
36-     zlib: True
37-     complevel: 5
38- 
39-   to_raster_kwargs:
40-     overwrite: True
41-     compress: 'lzw'
42-     tiled: True
43-     n_workers: 1
44-     n_threads: 2
45- 
46-   post:
47- 
48-     # Chunk read size
49-     chunks: 1024
50- 
51- storage:
52- 
53-   crs: 'utm'
54-   res: 10.0
55-   bands:
56-     - 'blue'
57-     - 'green'
58-     - 'red'
59-     - 'nir'
60-     - 'swir1'
61-     - 'swir2'
62- 
63- nasaearth:
64- 
65-   # The NASA Earthdata username to download HGT SRTM files
66-   # https://urs.earthdata.nasa.gov/
67-   username: '<username>'
68- 
69-   key_file: '/home/<username>/.eosvault/nasaearth.key'
70-   code_file: '/home/<username>/.eosvault/nasaearth.code'
71- 
72- srtm:
73- 
74-   srtm_path: '/jad-cel/sandbox-cel/paraguay_lc/raster/srtm'
75- 
76- masks:
77- 
78-   # 0-100 range
79-   s2cloudless_thresh: 60
``` 

## 5. Copy processing scripts
Copy the three primary processing scripts to the folder from which you will be submitting commands(home/<username>/code/bash).
```
#Make code/bash directory:
mkdir ~/code/bash
#Copy scripts over:
cp /jad-cel/sandbox-cel/paraguay_lc/templates/eosvault_download_eri_pry.sh ~/code/bash/download_eri.sh
cp /jad-cel/sandbox-cel/paraguay_lc/templates/eosvault_download_eri_pry_single.sh ~/code/bash/download_eri_single.sh
cp /jad-cel/sandbox-cel/paraguay_lc/templates/eosvault_post_eri_pry.sh ~/code/bash/post_eri.sh
cp /jad-cel/sandbox-cel/paraguay_lc/templates/pytuyau_pipeline_eri.sh ~/code/bash/pipeline_eri.sh
```
Edit the "download_eri.sh" "download_eri_single.sh" and "post_eri.sh" scripts to point to enter your username between the quotes in the line: MY_USERNAME="" 
(For help editing in Vim, [see Vim Commands](VimCommands).)
```
vim ~/code/bash/download_eri.sh
# Find line: MY_USERNAME="" and enter your username between the quotes
[Esc] #if in edit mode
:wq
vim ~/code/bash/download_eri_single.sh
# Find line: MY_USERNAME="" and enter your username between the quotes
[Esc] #if in edit mode
:wq
vim ~/code/bash/post_eri.sh
# Find line: MY_USERNAME="" and enter your username between the quotes
[Esc] #if in edit mode
:wq
```

