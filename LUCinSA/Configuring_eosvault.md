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
A NASA Earthdata secret key/code file pair is needed to download data from the Nasa Earthdata repository. 

You first need to create a free NASA Earthdata account at https://urs.earthdata.nasa.gov/home. Remember your username and your password for the steps below.

Now generate your key/code files through Geowombat (which you already installed in your Python environment) using IPython (the Python interpreter that you also already installed in your Python environment):
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
#This will prompt a password entry, where you will enter the password you used to set up your NASA Earthdata account
In [5] exit
# Deactivate your venv
(.nasaenv) deactivate
```

## 4. Copy and edit the config.yaml file
**Copy the config.yaml template to the project/config directory on your space**
```
#Create project/config directory if it doesn't already exist
mkdir -p ~/project/config
cp /jad-cel/sandbox-cel/templates/eosvault_config_eri_pry.yaml ~/project/config/config_eri.yaml
```
:::{note}if you are working with more than one country, you will want to give your copy of the config file a more descriptive name (i.e. config_eri_pry.yaml or config_eri_chl.yaml):::

**Open the template in the vim editor** 
```
vim ~/project/config/config_eri.yaml
```
To edit a line, type `i`, edit as desired, then hit `Esc` key and type `:wq`. Hit `Enter` key. For help editing in Vim, [see Vim Commands](VimCommands).
**Edit the following lines to match your paths:** 
* `MY_USERNAME` (line 1 of the template below): enter your ERI username between the quotes. This is a variable and should autofill everywhere else it says ${MY_USERNAME}
* `MY_COUNTRY` (line 3 of the template below): enter the name of the country you are working on between the quotes (without caps). This is a variable and should autofill everywhere else it says ${MY_CONUTRY}
* `Google secret key` (line 18 below): Edit path to match that of the secret key you uploaded in step #2 above
* `angles` (lines 22-23 of the template below): Make sure that the path matches yours. (You need to **extract the angle files** into the corresponding directory): 
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
* `Nasa Earthdata`: In line 72, modify your username to match the username you used for your NASA Earthdata account. Check that the paths in lines 74 and 75 match where you put your Nasa Earth key and code in step 3.

config_eri.yaml template (with added line #s for reference):
```
1- MY_USERNAME=""
2- # Current country options(note no caps): paraguay, chile
3- MY_COUNTRY=""
4- 
5- project_path: '/jad-cel/sandbox-cel/${MY_COUNTRY}_lc/raster/grids'
6- 
7- # Root directory for the GCP index files
8- index_dir: '/jad-cel/sandbox-cel/{MY_COUNTRY}_lc/raster'
9-
10- # Directory to save text files of incomplete batch jobs
11- log_dir: '/home/${MY_USERNAME}/code/bash'
12- 
13- # Length (in meters) on each side of grid
14- grid_size: 20000
15- 
16- google:
17- 
18- secret_key: '<key>.json'
19- 
20- angles:
21- 
22- l57_angles_path: '/home/${MY_USERNAME}/code/bin/ESPA/landsat_angles'
23- l8_angles_path: '/home/${MY_USERNAME}/code/bin/ESPA/l8_angles'
24- subsample: 10
25- resampling: 'bilinear'
26-
27- sixs:
28-
29-  coarse_res: 100.0
30-
31- io:
32-
33-  # netcdf | geotiff
34-  file_format: 'netcdf'
35-  chunks: 512
36-  resampling: 'cubic'
37-  num_threads: 2
38-  nodataval: 65535
39-
40-  to_netcdf_kwargs:
41-    zlib: True
42-    complevel: 5
43-
44-  to_raster_kwargs:
45-    overwrite: True
46-    compress: 'lzw'
47-    tiled: True
48-    n_workers: 1
49-    n_threads: 2
50-
51-  post:
52-
53-    # Chunk read size
54-    chunks: 1024
55-
56- storage:
57-
58-  crs: 'utm'
59-  res: 10.0
60-  bands:
61-    - 'blue'
62-    - 'green'
63-    - 'red'
64-    - 'nir'
65-    - 'swir1'
66-    - 'swir2'
67-
68- nasaearth:
69-
70-  # The NASA Earthdata username your username for the following account:
71-  # https://urs.earthdata.nasa.gov/
72-  username: '<username>'
73-
74-  key_file: '/home/${MY_USERNAME}/.eosvault/nasaearth.key'
75-  code_file: '/home/${MY_USERNAME}/.eosvault/nasaearth.code'
76-
77- srtm:
78-
79-  srtm_path: '/jad-cel/cel-sandbox/${MY_COUNTRY}_lc/raster/srtm'
80-
81- masks:
82-
83-  # 0-100 range
84-  s2cloudless_thresh: 60
``` 

## 5. Copy processing scripts
Copy the three primary processing scripts to the folder from which you will be submitting commands(home/<username>/code/bash).
```
#Make code/bash directory:
mkdir ~/code/bash
#Copy scripts over:
cp /jad-cel/sandbox-cel/templates/eosvault_download_eri.sh ~/code/bash/download_eri.sh
cp /jad-cel/sandbox-cel/templates/eosvault_download_eri_single.sh ~/code/bash/download_eri_single.sh
cp /jad-cel/sandbox-cel/templates/eosvault_post_eri.sh ~/code/bash/post_eri.sh
#If working on Paraguay:
cp /jad-cel/sandbox-cel/templates/paraguay_lc/pytuyau_pipeline_eri.sh ~/code/bash/pipeline_pty.sh
```
:::{note}if you are working with more than one country, you will want to give your copies a more descriptive name (i.e. download_eri_pry.sh or download_eri_chl.sh, etc.  and change the generic names throughout this guide to your own names):::    
    
Edit the "download_eri.sh" "download_eri_single.sh" and "post_eri.sh" scripts to point to enter your username between the quotes in the line: MY_USERNAME="" and country (no caps) between the quotes in the line: MY_COUNTRY""
(For help editing in Vim, [see Vim Commands](VimCommands).)
```
vim ~/code/bash/download_eri.sh
# Find line: MY_USERNAME="" and enter your username between the quotes
# Find line: MY_COUNTRY="" and enter your country between the quotes (no caps)  
[Esc] #if in edit mode
:wq
vim ~/code/bash/download_eri_single.sh
# Find line: MY_USERNAME="" and enter your username between the quotes
# Find line: MY_COUNTRY="" and enter your country between the quotes (no caps)  
[Esc] #if in edit mode
:wq
vim ~/code/bash/post_eri.sh
# Find line: MY_USERNAME="" and enter your username between the quotes
# Find line: MY_COUNTRY="" and enter your country between the quotes (no caps)  
[Esc] #if in edit mode
:wq
```

