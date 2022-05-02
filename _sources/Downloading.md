# Downloading process
===============================================================================================================================

Jordan Graesser's `eosvault` script drives the download process, [see here](https://github.com/jgrss/eosvault#readme) for more information on the script process.

To run the script:
(GetCells)=
## 1. Get gridcells to download
Find a block of cells in need of processing in the `cell_blocks.txt` file in `raid-cel/sandbox/sandbox-cel/paraguay_lc` (anything without an X next to it is free)
(if working ion another country, replace 'paraguay' with your country name) 
```
cd ~/../../
cd raid-cel/sandbox/sandbox-cel/paraguay_lc/
vim cell_blocks.txt
#"check out" a block of cells by putting an X and your initials next to it. Note your range(s) somewhere.
#save the file:
:wq
[Enter]
```
(Downloading)=
## 2. Edit the download_eri.sh document for targeted grid cells and sensor product
```
cd ~/code/bash/
vim download_eri_pry.sh
```
To edit a line, type `i`, edit as desired, then hit `Esc` key and type `:wq`. Hit `Enter` key
Go here for [more info on editing in vim](vimCommands)

This is the default script (with line numbering added for reference below):
```
1-   #!/bin/bash -l
2-   
3-   #SBATCH -N 1 # number of nodes
4-   #SBATCH -n 4 # number of cores
5-   #SBATCH -t 0-04:00 # time (D-HH:MM)
6-   #SBATCH -p basic
7-   #SBATCH -o sldown.%N.%a.%j.out # STDOUT
8-   #SBATCH -e sldown.%N.%a.%j.err # STDERR
9-   #SBATCH --job-name="dwn-pry"
10-  #SBATCH --array=1
11-
12-  umask 002
13
14-  ##############################################
15-  START_DATE="2010-03-01"
16-  END_DATE="2020-11-01"
17- 
18-  # yes | no
19-  IGNORE_INCOMPLETE="no"
20-  
21-  # yes | no
22-  RESET_ASSET="no"
23- 
24-  ##############################################
25-  
26-  BATCH_SIZE=100000
27-  
28-  # As an array job
29-  GRID_ID=$SLURM_ARRAY_TASK_ID
30-  
31-  PROJECT_HOME="/raid-cel/sandbox/sandbox/sandbox-cel/paraguay_lc"
32-  OUT_DIR="${PROJECT_HOME}/raster/grids"
33-  
34-  MY_USERNAME=""
35-  
36-  GRID_FILE="${PROJECT_HOME}/vector/pry_grids.gpkg"
37-  CONFIG_FILE="/home/${MY_USERNAME}/project/config/config_eri_pry.yaml"
38-  CRS_PROJ4="+proj=aea +lat_1=-5 +lat_2=-42 +lat_0=-32 +lon_0=-60 +x_0=0 +y_0=0 +ellps=aust_SA +units=m +no_defs "
39-  RES=10.0
40-  #############################################
41-  # Turn off NumPy parallelism and rely on dask
42-  #############################################
43-  export CLOUDSDK_PYTHON=python3
44-  export OPENBLAS_NUM_THREADS=1
45-  export MKL_NUM_THREADS=1
46-  # This should be sufficient for OpenBlas and MKL
47-  export OMP_NUM_THREADS=1
48-  ################################################
49-  
50-  # activate the virtual environment
51-  source ~/.nasaenv/bin/activate
52- 
53-  SAT_SENSORS=("S2" "S2cp" "LT05" "LE07" "LC08")
54- 
55-  # Iterate over each sensor
56-  for SAT_SENSOR in "${SAT_SENSORS[@]}"
57-  do
58- 
59-  if [ "$SAT_SENSOR" == "S2" ]; then
60-    SATELLITE="sentinel-2"
61-    ASSET_ID="COPERNICUS/S2"
62-  elif [ "$SAT_SENSOR" == "S2cp" ]; then
63-    SATELLITE="sentinel-2"
64-    ASSET_ID="COPERNICUS/S2_CLOUD_PROBABILITY" 
65-  else
66-    SATELLITE="landsat" 
67-    ASSET_ID="LANDSAT/${SAT_SENSOR}/C01/T1_SR"
68-  fi
69-  
70-  if [ "$IGNORE_INCOMPLETE" == "yes" ]; then
71- 
72-    eosvault download --out-dir $OUT_DIR --grid-file $GRID_FILE --grid $GRID_ID --start-date $START_DATE --end-date
73-  $END_DATE --config-file $CONFIG_FILE --batch-size $BATCH_SIZE --max-workers $SLURM_CPUS_ON_NODE --satellite $SATELLITE
74-  --asset-id $ASSET_ID --grid-crs "${CRS_PROJ4}" --out-res $RES --ignore-incomplete #--reset-asset #--clear-all #--reset-db
75-  
76-  else
77- 
78-   if [ "$RESET_ASSET" == "yes" ]; then
79- 
80-    eosvault download --out-dir $OUT_DIR --grid-file $GRID_FILE --grid $GRID_ID --start-date $START_DATE --end-date 
81- $END_DATE --config-file $CONFIG_FILE --batch-size $BATCH_SIZE --max-workers $SLURM_CPUS_ON_NODE --satellite $SATELLITE 
82-  --asset-id $ASSET_ID --grid-crs "${CRS_PROJ4}" --out-res $RES --reset-asset #--clear-all #--reset-db
83- 
84-    else
85- 
86-     eosvault download --out-dir $OUT_DIR --grid-file $GRID_FILE --grid $GRID_ID --start-date $START_DATE --end-date 
87-  $END_DATE --config-file $CONFIG_FILE --batch-size $BATCH_SIZE --max-workers $SLURM_CPUS_ON_NODE --satellite $SATELLITE 
88-  --asset-id $ASSET_ID --grid-crs "${CRS_PROJ4}" --out-res $RES #--clear-all #--reset-db
89-
90-    fi
91-
92-  fi
93-
94-  done
95-
96-  deactivate
```
Most lines should stay as they are.  
**the lines that you need to change are:** 
* **MyUsername** (line 34 here): Insert your username between the quotes if it isn't already present.
* **Grid info:** `#SBATCH --array= ` (line 10 here). This is where you enter the gridcells you are processing.  
     You can enter a range (e.g. 898-908), But be mindful that you are not hogging all the computer bandwidth.
:::{admonition}You can limit the number of cells that are processed at one time by adding %n
For example, 898-908%4 would process 4 cells at a time. When the first 4 finish, the next will start.
::: {warning} Google Earth Engine caps downloading loads, so downloads will fail if too many are processed at once.
With Sentinel (which are heavy files), %2 is safest. With Landsat %4 is fine.
:::
* **Incomplete tag**: Check that `IGNORE_INCOMPLETE=`(line 19 here) is set to "No" for the first run. (You will change this in   
   step 5 if all images don't download properly the first time.)

After making these edits, save your script and exit:
``` 
#(If in insert mode), use [Esc] to exit insert mode 
:wq
```

For further information on the rest of the script (parts you will NOT likely edit):
>* Line 1
>    * This is the "shebang" line, used in all Unix scripts, that tells the interpreter to execute the script.
>* Lines 3-10
>    * These are all configuration flags for SLURM.
>    * #SBATCH tells SLURM these are not user comments
>    * (line 3): We aren't doing any distributed computing across nodes, so we will always use -N 1.
>    * (line 4): -n 8 are the number of CPUs for concurrent or parallel processing (controlled in the Python scripts).
>    * (line 5): -t is the estimated max runtime
>    * (line 6): -p tells SLURM this job should be sent to the standard partition (there are also 'short', 'gpu', and 'largemem' >available).
>    * (line 7&8): -o and -e are output log files
>    * (line 9): -job-name is the name of the batch job
>    * (line 10): array is the list of gridcells for processing (1,2,3,etc or 1-3)
>* Line 12
>    * umask sets the permission for files that are creates. left digit is for user, middle digit is for group and right digit is for others. 0= no permissions denied (read/write/execute allowed), 2= write permission denied, 7= all permissions denied  
>* Lines 15-16
>    * User variables for start and end dates to stream. (For Paraguay, the dates are from 1-Mar-2010 to 1-Nov-2020. For Chile, 
    the dates are from 1-Jan-1985 to 31-Dec-2020)
>* Lines 18-20
>    * if yes, flag as incomplete if all images did not download correctly. If no, consider complete even if some images are >missing
>* Lines 21-22
>    * Flag to reset databased to allow redownload if previously downloaded fully but want to redo
>* Line 26
>    * Do not modify
>* Lines 28-29
>    * Names process/files based on grid cell number
>* Lines 31-32
>    * File paths (do not modify for this project) for:
>    * (lines 36) output files (leave as is)
>    * (line 37) input configuration file. Make sure to fill in [username] in line 34.
>* Line 38
>    * Geographic projection (do not modify for this project)
>* Line 39
>    * Pixel rosolution (do not modify for this project. 10.0 is the default. using 30.0 for Chile)
>* Lines 40-48
>    * Do not modify.
>* Lines 50-96
>    * These are the actual command lines to run the process based on the settings above.
 
:::{note}
**For GRID IDs >1000**: SLURM processors usually do not allow array numbers >1000. To get around this, modify the GRID_ID on line 29 to: `GRID_ID=$(($SLURM_ARRAY_TASK_ID + 1000))` Then subtract 1000 from the array number on line 10.
:::


## 3. Run the process
```
#if not already in bash directory, navigate there:
cd ~/code/bash/
#submit the command:
sbatch download_eri.sh
``` 
:::{note}
Current run-time estimates for single grid cells:
* Sentinel 2: 40min - 1hr
* Sentinel 2 cloudmasks: 20min
* Landsat 8: 30-40min
* Landsat 7: 20-30min
* Landsat 5: 5-10min
:::
To check on job:
```
cat sldown.bellows.909.78.err (with 909 = grid running and 78 = job number)
```
To check your position in line (note: this only shows people in our own group, so not a great estimate):
```
squeue -u [username]
```
to see if there are a lot of jobs before you (but again, only shows people in our own group, not full queue):
```
squeue
```
see [other slurm commands here](SlurmCommands)

(checkDownload)=
## 4. Check downloading status:
Often some images do not download properly the first time. All images for a grid cell need to be downloaded (or exempted) for the remaining processing steps to work.

To check whether all images processed for a specific gridcell and task:
```
cat sldown.bellows.909.78.err (with 909 = grid running and 78 = job numb
# or, if multiple gridcells run:
ls ~/code/bash/
# then for each .err file found:
cat [filename]
```
If all images downloaded, you should see something like this (precentage bars to 100% for downloading and no errors):
![](/Images/cat_download_multi.png)

You can also generate a figure to see which cells downloaded fully: 
```
#Activate virtual environment:
source .nasaenv/bin/activate
#Run status command (choose a cell number to zoom around for zoom-grid, usually in the middle of your line):
#For Paraguay:
eosvault status --config-file ~/project/config/config_eri_pry.yaml --out-dir ~/code/bash --grid-file /raid-cel/sandbox/sandbox-cel/paraguay_lc/vector/pry_grids.gpkg --zoom-grid <id # of cell to zoom to> --zoom-offset 200000
#For Chile:
eosvault status --config-file ~/project/config/config_eri_chile.yaml --out-dir ~/code/bash --grid-file /raid-cel/sandbox/sandbox-cel/chile_lc/vector/chl_grids.gpkg --zoom-grid <id # of cell to zoom to> --zoom-offset 200000

#Deactivate virtual environment:
deactivate
#Note you can zoom to the full figure (without cell numbers) by dropping the last two cell flags.
```
To view the Download Progress figure:
Download file to view on local desktop by opening a separate terminal and typing:
```
rsync -raz --progress <username>@ssh.eri.ucsb.edu:~/code/bash/grid_status_eosvault.png Desktop/gridstat.png
```
Alternatively, you can view the file though an FPT such as WinSCP.

The figure will look something like this:
![](/Images/grid_status.png)
The key indicates which cells loaded completely for each product. Those not listed need to be redownloaded. (Turquoise (y) means all are good)
(incomplete)=
## 5. Rerun cells that did not load completely
For cells that did not load completely, check the .err and .out files:
```
#i.e for grid cell 909, if 78 is job number for an incomplete process:
cat sldown.bellows.909.78.err
cat sldown.bellows.909.78.out 
```
If the .out file does not indicate any big problems and the slider at the bottom of the .err file is near 100% with only a few files missing in the fraction next to it (e.g. 124/127), you probably do not need to worry about larger issues. These cells do need to be rerun, however, to register correctly in the database for further processing.
**Repeat steps 2 and 3 above**. In adition to editing the grid cells to correspond to those that you are rerunning, in the ```download_eri.sh``` file, you will also change the following line:
```
yes | no
IGNORE_INCOMPLETE="no"

#to 
yes | no
IGNORE_INCOMPLETE="yes"
```
By setting "Ignore Incomplete" to "yes", the database will populate correctly even if all of the images do not download. If they do not load the second time, they probably aren't going to, so you can move on as long as only a few images are missing. Check the .err and .out files to make sure there are no major errors, though.
```{attention} Remember to reset "IGNORE_INCOMPLETE="no"  before beginning new downloads
```
### To rerun only a specific sensor
Use the script: ```download_eri_single.sh```
The script is copied below with line numbers added for reference. 
* Edit the array in line 10 to contain the grid cells you want to download.
* Change IGNORE_INCOMPLETE in line 31to "yes"
* Enter your username in line 39f it isn't already present.
* Select the sensor product by uncommenting the corresponding lines (remove the # at the beginning of the line) and commenting those that don't correspond (replacing the # at the beginning of the line). (Only one product can be run at a time).   
    * **<span style='color:purple'> For Sentinel: </span>** uncomment `SATELLITE="sentinel-2` (line 21here) and   
    `ASSET_ID="COPERNICUS/S2"` (line 22here) for images or `ASSET_ID="COPERNICUS/S2_CLOUD_PROBABILITY"` (line 23here) for 
    cloud masks
    * **<span style='color:purple'> For Landsat: </span>** uncomment `LANDSAT_SENSOR="LC08"` 
    `ASSET_ID="LANDSAT/${LANDSAT_SENSOR}/C01/T1_SR"`and `SATELLITE="landsat"` (lines 25-28here). Also change the Landsat 
    sensor (line 26 ere) to correspond to the sensor you want to process: "LT05" for Landsat 5, "LE07" for Landsat 7, or 
    "LC08" for Landsat 8 (as noted in line 25.
```
#Download_eri_single.sh script, copied from jad-cel/sandbox-cel/paraguay_lc/templates/eosvault_download_eri_pry_single.sh
1-  #!/bin/bash -l
2-  
3-  #SBATCH -N 1 # number of nodes
4-  #SBATCH -n 2 # number of cores
5-  #SBATCH -t 0-04:00 # time (D-HH:MM)
6-  #SBATCH -p basic
7-  #SBATCH -o sldown.%N.%a.%j.out # STDOUT
8-  #SBATCH -e sldown.%N.%a.%j.err # STDERR
9-  #SBATCH --job-name="dwn-pry"
10- #SBATCH --array=894
11-
12- sleep $((RANDOM%30+1))
13- 
14- umask 002
15- 
16- #####################################################
17- START_DATE="2010-03-01"
18- END_DATE="2020-11-01"
19- 
20- # landsat | sentinel-2
21- #SATELLITE="sentinel-2"
22- #ASSET_ID="COPERNICUS/S2"
23- #ASSET_ID="COPERNICUS/S2_CLOUD_PROBABILITY"
24- 
25- # LT05 | LE07 | LC08
26- LANDSAT_SENSOR="LC08"
27- ASSET_ID="LANDSAT/${LANDSAT_SENSOR}/C01/T1_SR"
28- SATELLITE="landsat"
29- 
30- # yes | no
31- IGNORE_INCOMPLETE="no"
32- 
33- BATCH_SIZE=100000
34- #####################################################
35- 
36- # As an array job
37- GRID_ID=$SLURM_ARRAY_TASK_ID
38- 
39- MY_USERNAME=""
40- OUT_DIR="/raid-cel/sandbox/sandbox-cel/paraguay_lc/raster/grids"
41- GRID_FILE="/raid-cel/sandbox/sandbox-cel/paraguay_lc/vector/pry_grids.gpkg"
42- CONFIG_FILE="/home/${MY_USERNAME}/project/config/config_eri.yaml"
43- CRS_PROJ4="+proj=aea +lat_1=-5 +lat_2=-42 +lat_0=-32 +lon_0=-60 +x_0=0 +y_0=0 +ellps=aust_SA +units=m +no_defs "
44- RES=10.0
44- #############################################
46- # Turn off NumPy parallelism and rely on dask
47- #############################################
48- export CLOUDSDK_PYTHON=python3
49- export OPENBLAS_NUM_THREADS=1
50- export MKL_NUM_THREADS=1
51- # This should be sufficient for OpenBlas and MKL
52- export OMP_NUM_THREADS=1
53- ################################################
54- 
55- # activate the virtual environment
56- source ~/.nasaenv/bin/activate
57-
58- if [ "$IGNORE_INCOMPLETE" == "yes" ]; then
59-
60-   eosvault download --out-dir $OUT_DIR --grid-file $GRID_FILE --grid $GRID_ID --start-date $START_DATE --end-date $END_DATE 61- --config-file $CONFIG_FILE --batch-size $BATCH_SIZE --max-workers $SLURM_CPUS_ON_NODE --satellite $SATELLITE --asset-id 
62- $ASSET_ID --grid-crs "${CRS_PROJ4}" --out-res $RES --ignore-incomplete #--reset-asset #--clear-all #--reset-db
63- 
64- else
65-  
66 eosvault download --out-dir $OUT_DIR --grid-file $GRID_FILE --grid $GRID_ID --start-date $START_DATE --end-date $END_DATE 67--config-file $CONFIG_FILE --batch-size $BATCH_SIZE --max-workers $SLURM_CPUS_ON_NODE --satellite $SATELLITE --asset-id 
68$ASSET_ID --grid-crs "${CRS_PROJ4}" --out-res $RES #--reset-asset #--clear-all #--reset-db
69
70 fi
71
72 deactivate
```

To run the process:
```
#if not already in bash directory, navigate there:
cd ~/code/bash/
#submit the command:
sbatch download_eri_single.sh
``` 
Check .err and .out files as before.

## 6. Move .err and .out files to new directory to facilitate tracking of new downloads
All these files will soon make it difficult to find the new ones to monitor. Move them to an archive directory after checking to clean up the clutter.
```
#Make the directory (first time only)
mkdir /home/<username>/code/bash/archive/
#Move .err and .out files from /bash to /archive
cd ~/code/bash/
mv /home/<username>/code/bash/*.{err,out} /home/<username>/code/bash/archive/
```