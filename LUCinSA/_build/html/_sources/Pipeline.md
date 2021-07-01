(pipeline)=
# Pipeline process
===============================================================================================================================

Jordan Graesser's `pytuyau` script drives the pipeline process, [see here](https://github.com/jgrss/pytuyau#readme) for more information on the script process.

Pipeline template (with line numbers added for reference below):
```
1-   #!/bin/bash -l
2-  
3-   #SBATCH -N 1 # number of nodes
4-   #SBATCH -n 8 # number of cores
5-   #SBATCH -t 0-08:00 # time (D-HH:MM)
6-   #SBATCH -p basic 
7-   #SBATCH -o slpipe.%N.%a.%j.out # STDOUT
8-   #SBATCH -e slpipe.%N.%a.%j.err # STDERR
9-   #SBATCH --job-name="slpipe-pre"
10-  #SBATCH --array=910
11- 
12-  #############################################
13-  # Turn off NumPy parallelism and rely on dask
14-  #############################################
15-  export OPENBLAS_NUM_THREADS=1
16-  export MKL_NUM_THREADS=1
17-  # This should be sufficient for OpenBlas and MKL
18-  export OMP_NUM_THREADS=1
19-  ################################################
20- 
21-  # Choices are {preprocess, fusion, topo, reconstruct, segment, classify}
22-  # 1. preprocess
23-  #    a. Checks 'no data' images
24-  #    b. Co-registers Sentinel
25-  # X2. mask
26-  #    Masks clouds and shadows
27-  # X3. fusion
28-  #    - Fusion of Landsat to Sentinel 10m using modified StarFM
29-  # 4. topo
30-  #    - not currently implemented
31-  # 5. reconstruct
32-  #    - Reconstructs weekly time series using dynamic temporal smoothing (DTS)
33-  # 6. reindex_vi
34-  #    - Reindexes weekly vegetation indices to bi-weekly
35-  # 7. segment
36-  #    - Segments vegetation objects using SACFEI
37-  # 8. classify
38-  #    - Classifies land cover
39-  # 9. assess
40-  #    - Assesses land cover predictions
41-  # 10. clean
42-  #    - Removes unwanted files
43-  STEP="preprocess"
44- 
45-  # Comma-separated list of grids to process
46-  #GRIDS="250"
47-  GRIDS="${SLURM_ARRAY_TASK_ID}"
48- 
49-  #########
50-  # MASKING
51-  #########
52- 
53-  MASK_ONLY="no"
54-  REF_RES=40.0
55-  RESET_CLOUD_DB="True"
56- 
57-  #######
58-  # CLEAN
59-  #######
60- 
61-  # masks, downloads, nodata, nocoreg, fusion, vis, vrts, old_seg
62-  #REMOVE_ITEMS="[downloads,masks,nodata,nocoreg,fusion,vis,vrts]"
63-  REMOVE_ITEMS="[]"
64- 
65-  ########
66-  # FUSION
67-  ########
68- 
69-  WINDOW_SIZE=9
70-  MAX_YEARS=2
71-  MAX_DAYS=15
72-  FOVERWRITE="True"
73-
74-  #############
75-  # RECONSTRUCT
76-  #############
77- 
78-  RINPUT="ms"
79-  START_PAD="2010-03-01"
80-  START="2010-07-01"
81-  END_PAD="2020-11-01"
82-  END="2020-07-01"
83-  SKIP_INTERVAL=7
84-  SKIP_YEARS=10
85-  REC_VI="evi2"
86-  ROVERWRITE="False"
87-  SM_CHUNKS=512
88-  PREFILL_GAPS="True"
89-  DTS_MAX_WIN=61
90-  DTS_MIN_WIN=15
91-  PREFILL_MIN_RSQ=0.2
92-  PREFILL_YEARS=2
93-  DTS_t=5
94- 
95-  ############
96-  # REINDEX VI
97-  ############
98-  REX_VI="evi2"
99-
100- ##########
101- # CLASSIFY
102- ##########
103-
104- # sample | fit | fit_predict | predict
105- CLSMETHOD="fit"
106- CINPUT="ms"
107- # temperate_gss_vi2; tropical_gss_vi2
108- CLS_REGION_NAME="tropical_gss_vi2"
109- CLS_MODEL_NAME="${CLS_REGION_NAME}_11_11"
110- LC_VECTOR=".kml"
111- KEEP_FEATURES="[1,1,1,1]"
112- # update/overwrite the sample stack
113- UPDATE_SAMPLES="True"
114- # overwrite each grid sample dataset
115- OVERWRITE_SAMPLES="False"
116- OVERWRITE_MODEL="True"
117- OVERWRITE_IMAGE="False"
118- CLS_BANDS="[evi2,wi]"
119- CLS_NITERS=2000
120-
121- # OUTPUT files
122- TRAIN_SAMPLES="/training/south_america_extract_${CLS_REGION_NAME}.gpkg"
123- MODEL_FILE="/training/south_america_${CLS_MODEL_NAME}.model"
124- 
125- #########
126- # SEGMENT
127- #########
128- 
129- # fit | fit_predict | predict
130- SEGMETHOD="predict"
131- OVERWRITE_EDGE_MODEL="False"
132- OVERWRITE_EDGE_IMAGE="True"
133- OVERWRITE_SEG_IMAGE="True"
134- SEG_REGION_NAME="temperate_gss_vi2"
135- SEG_BANDS="[evi2,wi]"
136- SEG_NITERS=2000
137- 
138- SEG_TRAIN_PATH="/training/edges"
139- SEG_MODEL_FILE="/training/south_america_extract_${SEG_REGION_NAME}.edges"
140- 
141- ###############################
142- # DO NOT MODIFY BELOW THIS LINE
143- ###############################
144- 
145- # activate the virtual environment
146- source ~/.nasaenv/bin/activate
147- 
148- CONFIG_UPDATES="grids:[${GRIDS}] main_path:/jad-cel/sandbox/cel/paraguay_lc/raster/grids num_workers:${SLURM_CPUS_ON_NODE} cloud_mask:reset_db:${RESET_CLOUD_DB} cloud_mask:ref_res:${REF_RES} fusion:window_size:${WINDOW_SIZE} fusion:fill_max_years:${MAX_YEARS} fusion:fill_max_days:${MAX_DAYS} fusion:overwrite:${FOVERWRITE} reconstruct:input:${RINPUT} reconstruct:start_pad:${START_PAD} reconstruct:end_pad:${END_PAD} reconstruct:start:${START} reconstruct:end:${END} reconstruct:skip_interval:${SKIP_INTERVAL} reconstruct:skip_years:${SKIP_YEARS} reconstruct:vi:${REC_VI} reconstruct:overwrite:${ROVERWRITE} reconstruct:chunks:${SM_CHUNKS} reconstruct:smooth_kwargs:max_window:${DTS_MAX_WIN} reconstruct:smooth_kwargs:min_window:${DTS_MIN_WIN} reconstruct:smooth_kwargs:min_prefill_rsquared:${PREFILL_MIN_RSQ} reconstruct:smooth_kwargs:max_prefill_years:${PREFILL_YEARS} reconstruct:smooth_kwargs:prefill_gaps:${PREFILL_GAPS} reconstruct:smooth_kwargs:t:${DTS_t} reindex_vi:vi:${REX_VI} classify:lc_vector:${LC_VECTOR} classify:train_samples:${TRAIN_SAMPLES} classify:model_file:${MODEL_FILE} classify:update_samples:${UPDATE_SAMPLES} classify:overwrite_samples:${OVERWRITE_SAMPLES} classify:overwrite_model:${OVERWRITE_MODEL} classify:overwrite_image:${OVERWRITE_IMAGE} classify:method:${CLSMETHOD} classify:input:${CINPUT} classify:image_bands:${CLS_BANDS} classify:image_bands_pred:${CLS_BANDS} classify:keep_features:${KEEP_FEATURES} classify:augment:n_iters:${CLS_NITERS} segment:method:${SEGMETHOD} segment:image_bands:${SEG_BANDS} segment:overwrite_model:${OVERWRITE_EDGE_MODEL} segment:overwrite_edges:${OVERWRITE_EDGE_IMAGE} segment:overwrite_seg:${OVERWRITE_SEG_IMAGE} segment:training_path:${SEG_TRAIN_PATH} segment:model_file:${SEG_MODEL_FILE} segment:augment:n_iters:${SEG_NITERS} clean:remove_items:${REMOVE_ITEMS}"

if [ "$MASK_ONLY" == "yes" ]; then
  if [ "$STEP" == "preprocess" ]; then
    tuyau $STEP --config-updates $CONFIG_UPDATES --mask-only
  else
    tuyau $STEP --config-updates $CONFIG_UPDATES
  fi
else
  tuyau $STEP --config-updates $CONFIG_UPDATES
fi

deactivate
```
Most lines should stay as they are.  
**the lines that you need to change are:**  
* **Grid info:** `#SBATCH --array= ` (line 10 here). This is where you enter the gridcells you are processing.  
     You can enter a range (e.g. 898-908), But be mindful that you are not hogging all the computer bandwidth.
:::{admonition}You can limit the number of cells that are processed at one time by adding %n
For example, 898-908%4 would process 4 cells at a time. When the first 4 finish, the next will start.

## Run the process
```
#if not already in bash directory, navigate there:
cd ~/code/bash/
#submit the command:
sbatch pipeline_eri.sh
``` 
:::{note}
Current run-time estimates for single grid cells:
* Preprocess
    * preprocess
    * Mask
    * Fusion
    * Topo
    * Reconstruct
    * Re-Index_vi
    * Segment
    * Classify
    * Assess
    * Clean
:::

## Get grid pipeline status

To generate the Pipeline Progress figure: 
```
#Activate virtual environment:
source .nasaenv/bin/activate
#Run status command:
tuyau status --config-updates status:project_path:/jad-cel/sandbox-cel/paraguay_lc/raster/grids status:out_path:<PNG location> status:grid_file:/jad-cel/sandbox-cel/paraguay_lc/vector/pry_grids.gpkg status:zoom:True
#Deactivate virtual environment:
deactivate
```
To view the Download Progress figure:
Download file to view on local computer:
```
rsync -raz --progress <username>@ssh.eri.ucsb.edu:<ERI path> <local path>
```
Alternatively, you can view the file though an FPT such as WinSCP.
