# Downloading process
===============================================================================================================================

Jordan Graesser's `eosvault` script drives the download process, [see here](https://github.com/jgrss/eosvault#readme) for more information on the script process.

To run the script:
(GetCells)=
## 1. Get gridcells to download
Find a block of cells in need of processing in the `cell_blocks.txt` file in `cel/paraguay_lc` (anything without an X next to it is free)
```
cd ~/../
cd cel/paragual_lc
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
vim download_eri.sh
```
To edit a line, type `i`, edit as desired, then hit `Esc` key and type `:wq`. Hit `Enter` key
Go here for [more info on editing in vim](vimCommands)

This is the default script (with line numbering added for reference below):
```
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
14- # Change permissions on output files
15- # user = 7 = rwx
16- # group = 7 = rwx
17- # others = 4 = r 
18- #umask 774
19- 
20- #####################################################
21- START_DATE="2010-03-01"
22- END_DATE="2020-11-01"
23- 
24- # landsat | sentinel-2
25- #SATELLITE="sentinel-2"
26- #ASSET_ID="COPERNICUS/S2"
27- #ASSET_ID="COPERNICUS/S2_CLOUD_PROBABILITY"
28- 
29- # LT05 | LE07 | LC08
30- LANDSAT_SENSOR="LC08"
31- ASSET_ID="LANDSAT/${LANDSAT_SENSOR}/C01/T1_SR"
32- SATELLITE="landsat"
33- 
34- # yes | no
35- IGNORE_INCOMPLETE="no"
36- 
37- BATCH_SIZE=100000
38- #####################################################
39- 
40- # As an array job
41- GRID_ID=$SLURM_ARRAY_TASK_ID
42- 
43- OUT_DIR="/jad-cel/paraguay_lc/raster/grids"
44- GRID_FILE="/jad-cel/paraguay_lc/vector/pry_grids.gpkg"
45- CONFIG_FILE="/home/<username>/project/config/config_eri.yaml"
46- CRS_PROJ4="+proj=aea +lat_1=-5 +lat_2=-42 +lat_0=-32 +lon_0=-60 +x_0=0 +y_0=0 +ellps=aust_SA +units=m +no_defs "
47- 
48- #############################################
49- # Turn off NumPy parallelism and rely on dask
50- #############################################
51- export CLOUDSDK_PYTHON=python3
52- export OPENBLAS_NUM_THREADS=1
53- export MKL_NUM_THREADS=1
54- # This should be sufficient for OpenBlas and MKL
55- export OMP_NUM_THREADS=1
56- ################################################
57- 
58- # activate the virtual environment
59- source ~/.nasaenv/bin/activate
60- 
61- if [ "$IGNORE_INCOMPLETE" == "yes" ]; then
62-
63-   eosvault download --out-dir $OUT_DIR --grid-file $GRID_FILE --grid $GRID_ID --start-date $START_DATE --end-date $END_DATE 64-  --config-file $CONFIG_FILE --batch-size $BATCH_SIZE --max-workers $SLURM_CPUS_ON_NODE --satellite $SATELLITE --asset-id 65-  65-  $ASSET_ID --grid-crs "${CRS_PROJ4}" --ignore-incomplete #--reset-asset #--clear-all #--reset-db
66- 
67-  else
68- 
69-   eosvault download --out-dir $OUT_DIR --grid-file $GRID_FILE --grid $GRID_ID --start-date $START_DATE --end-date $END_DATE 70-  --config-file $CONFIG_FILE --batch-size $BATCH_SIZE --max-workers $SLURM_CPUS_ON_NODE --satellite $SATELLITE --asset-id 71-  $ASSET_ID --grid-crs "${CRS_PROJ4}" #--reset-asset #--clear-all #--reset-db
72-
73-  fi
74-
75-  deactivate
```
Most lines should stay as they are.  
**the lines that you need to change are:**  
**Grid info:** `#SBATCH --array= ` (line 10 here). This is where you enter the gridcells you are processing.  
     You can enter a range (e.g. 898-908), But be mindful that you are not hogging all the computer bandwidth.
:::{admonition}You can limit the number of cells that are processed at one time by adding %n
For example, 898-908%4 would process 4 cells at a time. When the first 4 finish, the next will start.
::: {warning} Google Earth Engine caps downloading loads, so downloads will fail if too many are processed at once.
With Sentinel (which are heavy files), %2 is safest. With Landsat %4 is fine.
:::
**Product info:** Lines following `# landsat | sentinel-2` (Lines 25-32 here). This is where you select the satellite product, by uncommenting the corresponding lines (remove the # at the beginning of the line) and commenting those that don't correspond (replacing the # at the beginning of the line). (Only one product can be run at a time).   

**<span style='color:purple'> For Sentinel: </span>** uncomment `SATELLITE="sentinel-2` (line 25 here) and `ASSET_ID="COPERNICUS/S2` (line 26 here) **or** `ASSET_ID=COPERNICUS/S2_CLOUD_PROBABILITY` (line 27 here). (You will do both, but they are sepparate processes).  

**<span style='color:purple'> For Landsat: </span>** uncomment `LANDSAT_SENSOR="LC08"` `ASSET_ID="LANDSAT/${LANDSAT_SENSOR}/C01/T1_SR"`and `SATELLITE="landsat"` (lines 30-32 here). Also change the Landsat sensor (line 30 here) to correspond to the sensor you want to download: "LT05" for Landsat 5, "LE07" for Landsat 7, or "LC08" for Landsat 8 (as noted in line 29). You will do all three, separately.   

**Incomplete tag**: Check that `IGNORE_INCOMPLETE=`(line 35 here) is set to "No" for the first run. (You will change this in step 5 if all images don't download properly the first time.)

After making these edits, save your script and exit:
``` 
#(If in insert mode), use [Esc] to exit insert mode 
:wq
```

For further information on the rest of the script (parts you will NOT likely edit):
>* Lines 1-10
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
>    * (line 12): Staggars initiation of downloading to avoid bottleneck that often occurs at very beginning of process
>* Lines 14-18
>    * Set file permissions so that everyone in group can manipulate files
>*Lines 21-22
>    * User variables for start and end dates to stream. (For this project, the dates are from 1-Mar-2010 to 1-Nov-2020)
>* Lines 25-27
>    * For downloading Sentinel-2. Uncomment Line 25 and 26 or 27. (and make sure 29-32 are commented)
>* Lines 29-32
>    * For downloading Landsat. (uncomment lines 30-32 and comment lines 25-27)
>    * (line 30): Change LANDSAT_SENSOR="LT05" to download Landsat 5, e.g. (as per options on line 29)
>* Lines 34-35
>    * if yes, flag as incomplete if all images did not download correctly. If no, consider complete even if some images are >missing
>* Line 37
>    * Do not modify
>* Lines 40-41
>    * Names process/files based on grid cell number
>* Lines 43-47
>    * File paths for:
>    * (lines 43-45) output files (leave as is)
>    * (line 46) input configuration file. Make sure [username] is replaced with your own username.
>* Lines 48-56
>    * Do not modify.
>* Lines 43-47
>    * These are the actual command lines to run the process based on the settings above.
 

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
To check your position in line:
```
squeue -u [username]
```
to see if there are a lot of jobs before you:
```
top
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
If all images downloaded, you should see:
![](/Images/cat_download.png)

If you run many cells in the same job, this will get tedious. Instead, you can generate a figure to see which cells downloaded fully.
To generate the Download Progress figure: 
```
#Activate virtual environment:
source .nasaenv/bin/activate
#Run status command:
eosvault status --config-file ~/project/config/config_eri.yaml --out-dir ~/code/bash --grid-file /home/cel/paraguay_lc/vector/pry_grids.gpkg --zoom
#Deactivate virtual environment:
deactivate
```
To view the Download Progress figure:
Download file to view on local computer:
```
rsync -raz --progress <username>@ssh.eri.ucsb.edu:<ERI path> <local path>
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
**Repeat steps 2 and 3 above**. In adition to editing the grid cells and sensor to correspond to those that you are rerunning, in the ```download_eri.sh``` file, you will also change the following line:
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

## 6. Move .err and .out files to new directory to facilitate tracking of new downloads
All these files will soon make it difficult to find the new ones to monitor. Move them to an archive directory after checking to clean up the clutter.
```
#Make the directory (first ime only)
mkdir home/<username>/code/bash/archive/
#Move .err and .out files from /bash to /archive
cd ~/code/bash/
mv /home/<username>/code/bash/*.{err,out} /home/<username>/code/bash/archive/
```