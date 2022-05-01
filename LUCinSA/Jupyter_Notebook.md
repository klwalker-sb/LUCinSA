# Using Jupyter Notebook to interact with data on cluster
================================================================================================================================

a bit about Jupyter Notebook...

## Install Jupyter notebooks on cluster

Create a folder in your home directory to house your Jupyter notebooks (e.g. `mkdir ~/Jupyter`)
If using the LUCinSA notebooks, clone repo from github into this directory
(....)

Install Jupyter Notebook into your Venv:


## Connect to cluster via ssh with tunnel to port for Jupyter

If using PuTTY:
Step1: on first PuTTY screen, enter SSH address to bula as normal
Step2: click `SSH`, then `Tunnels` on menu on left. In "Source Port", enter <port> (where <port> is 8889 here. Maybe pick a different number to avoid possible conflicts). In "Destination", enter `localhost:<port>`. Click "Add" to add the connection to the window above. 
Step3: add another Tunnel to bellows in the same way, with something like 222 in the "Source Port" and <user>@bellows.eri.ucsb.edu:22 in the destination port.
(the last number should match your original local port from the first PuTTY screen). Click "Add".
Go back to session and save session with name (e.g. `JupyterOnERI` here) for future use

Click "Open" to open the connection and enter password as normal.
Activate your virtual environment that has Jupyter installed on it (i.e: `source .nasaenv/bin/activate`)
At next prompt, enter: `jupyter notebook --no-browser --port=<port>` (using the port number from your SSH connection)

Leave this window running and open a browser from your computer and enter into the address bar: `http://localhost:<port>/`
You should now see a Jupter tree of your home directory on the cluster. Navigate to the notebook you want to use. 

(you can create a permanant password by entering `jupyter notebook password` at the prompt after activating a virtual environment with Jupyter installed, before entering the )