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
If working on Paraguay, <cntry> = pry. if working on Chile, <cntry> = chile.
```
#Create project/config directory if it doesn't already exist
mkdir -p ~/project/config
cp /raid-cel/sandbox/sandbox-cel/templates/eosvault_config_eri.yaml ~/project/config/config_eri_<cntry>.yaml
```

**Open the template in the vim editor** 
```
vim ~/project/config/config_eri_<cntry>.yaml
```
To edit a line, type `i`, edit as desired, then hit `Esc` key and type `:wq`. Hit `Enter` key. For help editing in Vim, [see Vim Commands](VimCommands).
**Edit the following lines to match your paths:** 
* `<<MY_USERNAME>>` Change every instance of <<MY_USERNAME>> to your username (delete << brackets)
*`<<MY_COUNTRY>>` Change every instance of <<MY_COUNTRY>> to your country (paraguay/uruguay/chile). (delete << brackets)
* `Google secret key` (line 14 below): Edit path to match that of the secret key you uploaded in step #2 above
* `angles` (lines 18-19 of the template below): Make sure that the path matches yours. (You need to **extract the angle files** into the corresponding directory): 
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
* `Nasa Earthdata`: In line 68, modify your username to match the username you used for your NASA Earthdata account. Check that the paths in lines 70 and 71 match where you put your Nasa Earth key and code in step 3.

config_eri.yaml template (with added line #s for reference):
```
1- project_path: '/raid-cel/sandbox/sandbox-cel/<<MY_COUNTRY>>_lc/raster/grids'
2- 
3- # Root directory for the GCP index files
4- index_dir: '/raid-cel/sandbox/sandbox-cel/<<MY_COUNTRY>>_lc/raster'
5-
6- # Directory to save text files of incomplete batch jobs
7- log_dir: '/home/<<MY_USERNAME>>/code/bash'
8- 
9- # Length (in meters) on each side of grid
10- grid_size: 20000
11- 
12- google:
13- 
14- secret_key: '<key>.json'
15- 
16- angles:
17- 
18- l57_angles_path: '/home/$<<MY_USERNAME>>/code/bin/ESPA/landsat_angles'
19- l8_angles_path: '/home/$<<MY_USERNAME>>/code/bin/ESPA/l8_angles'
20- subsample: 10
21- resampling: 'bilinear'
22-
23- sixs:
24-
25-  coarse_res: 100.0
26-
27- io:
28-
29-  # netcdf | geotiff
30-  file_format: 'netcdf'
31-  chunks: 512
32-  resampling: 'cubic'
33-  num_threads: 2
34-  nodataval: 65535
35-
36-  to_netcdf_kwargs:
37-    zlib: True
38-    complevel: 5
39-
40-  to_raster_kwargs:
41-    overwrite: True
42-    compress: 'lzw'
43-    tiled: True
44-    n_workers: 1
45-    n_threads: 2
46-
47-  post:
48-
49-    # Chunk read size
50-    chunks: 1024
51-
52- storage:
53-
54-  crs: 'utm'
55-  res: 10.0
56-  bands:
57-    - 'blue'
58-    - 'green'
59-    - 'red'
60-    - 'nir'
61-    - 'swir1'
62-    - 'swir2'
63-
64- nasaearth:
65-
66-  # The NASA Earthdata username your username for the following account:
67-  # https://urs.earthdata.nasa.gov/
68-  username: '<username>'
69-
70-  key_file: '/home/<<MY_USERNAME>>/.eosvault/nasaearth.key'
71-  code_file: '/home/<<MY_USERNAME>>/.eosvault/nasaearth.code'
72-
73- srtm:
74-
75-  srtm_path: '/raid-cel/sandbox/sandbox-cel/<<MY_COUNTRY>>_lc/raster/srtm'
76-
77- masks:
78-
79-  # 0-100 range
80-  s2cloudless_thresh: 60
``` 

## 5. Copy processing scripts
Copy the three primary processing scripts to the folder from which you will be submitting commands(home/<username>/code/bash).
    Replace <country> and <cntry> with text that corresponds to the country you are working in:
    For Paraguay: <country> = paraguay and <cntry> = pry
    For Uruguay: <country> = uruguay and <cntry> = ury
    For Chile: <country> = chile and <cntry> = chile
```
#if it doesn't already exist, Make code/bash directory:
mkdir ~/code/bash
#Copy scripts over:
cp /raid-cel/sandbox/sandbox-cel/<country>_lc/templates/eosvault_download_eri_<cntry>.sh ~/code/bash/download_eri_<cntry>.sh
cp /raid-cel/sandbox/sandbox-cel/<country>_lc/templates/eosvault_download_eri_single_<cntry>.sh ~/code/bash/download_eri_single_<cntry>.sh
cp /raid-cel/sandbox/sandbox-cel/<country>_lc/templates/eosvault_post_eri_<cntry>.sh ~/code/bash/post_eri_<cntry>.sh
cp /raid-cel/sandbox/sandbox-cel/<country>_lc/templates/pipeline_eri_<cntry>.sh ~/code/bash/pipeline_<cntry>.sh
```   
    
Edit the "download_eri.sh" "download_eri_single.sh" and "post_eri.sh" scripts. enter your username between the quotes in the line: MY_USERNAME=""
(For help editing in Vim, [see Vim Commands](VimCommands).)
(if working on Uruguay/Chile, replace pry with ury/chile below:)
```
vim ~/code/bash/download_eri_pry.sh
# Find line: MY_USERNAME="" and enter your username between the quotes
[Esc] #if in edit mode
:wq
vim ~/code/bash/download_eri_single_pry.sh
# Find line: MY_USERNAME="" and enter your username between the quotes
[Esc] #if in edit mode
:wq
vim ~/code/bash/post_eri_pry.sh
# Find line: MY_USERNAME="" and enter your username between the quotes
[Esc] #if in edit mode
:wq
```

