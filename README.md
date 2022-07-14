# CIC_JDL_HMMA_APP

## General Info

The code for this project has been deployed in two branches, corresponding to the programming language they are written in: Python and R. This documentation contains information mostly related to the Python app, as this was directly developed during this project timespan. Both the apps are deployed in Docker, on a Nimbus instance.
The apps should be available at the following IPs:  
  
**PythonApp- Main interface**:    [ http://146.118.69.181:8501/](http://146.118.69.181:8501/)    
           **Neo4j DB backhand**: [http://146.118.69.181:7474/](http://146.118.69.181:7474/browser/)   
  
**RApp-      Main interface**:    [http://146.118.69.181:3434/HMMA_new/](http://146.118.69.181:3838/HMMA_new/)  

# Python App  
This app was developed by Carlo Martinotti ([CIC](https://computation.curtin.edu.au/)) in conjunction with Alex Walker and Brent McInnes ([JdL](https://jdlc.curtin.edu.au/)). For any feature request, bug report, questions use the dedicated Github page.  

## Operate the app
As previously mentioned the app is deployed in docker with docker-compose. These are the necessary commands for a non expert user:  

### If the you want to reboot the app follow these step:
1- ssh into your instance (nimbus)  
2- navigate to /home/ubuntu/CIC_JDL_MNA/Python_code/docker_compose  
3- type: docker compose down    
4- type: docker compose up -d  
You're done  

### Installing the app on a new server:  

1- install docker on the server   
2- clone the github repo wherever you want to install (install_dir)  
3- navigate to install_dir/CIC_JDL_MNA/Python_code/docker_jdl-environment/  
4- type: docker build -t strl_app .  
5- navigate to install_dir/CIC_JDL_MNA/Python_code/docker_neo4j-disentangled/  
6- type: docker build -t neo4j:ssh  
7- type: mkdir ~/HMMA_ARCHIVE &&  mkdir ~/HMMA_ARCHIVE/DB_BACKUPS &&  mkdir ~/HMMA_ARCHIVE/DB_RESTORE  
8- navigate to: install_dir/CIC_JDL_MNA/Python_code/docker_compose/  
9- type: docker compose up -d   
The app should be running at http://localhost:8501/(http://localhost:8501/)

## How the app works
This app is built using [streamlit](https://streamlit.io/), a python framework for building web apps. The app interaces with a [neo4j](https://neo4j.com/) database, a graph knowledge based database. To check how is youre database looking you can use the neo4j browser backhand at the [relevant address](http://146.118.69.181:7474/browser/).  
This is useful to visualize data and keep new commits to the database under control.  
The app will at the beginning ask you to select one of the quantities of interest stored in the database relationships (eg volume fraction, mass fraction, etc)  and if you want to display polygon on the map, all the necessary shape files. Then the current interface is diveded into 5 sections accessible by the menu bar:

### 1) Map and Data
This section is the default page when you connect to the app should almost always be run first as it is tightly related to section 2. The app initially query all the database points, store them into a [pandas dataframe](https://pandas.pydata.org/) and plots them onto a map using [geopandas](https://geopandas.org/en/stable/) and [plotly](https://plotly.com/) libraries. Then it presents a series of filters with which you can trim down the displayed points. It is important to realize that any correlation calculated in section 2 will be calculated using THIS subset of points. This means that selecting nothing on the widgets will run the calculations on the whole database, whereas filtering down the dataframe will reduce in a subsample that will yield different correlations relative to the subsample.  
The app works pretty much intuitively, in the sidebar there is a series of filters that can be used from top to bottom, to filter down the dataframe.  
The dataframe obtained is displayed under the map, and there is the possibility to save it as a csv with a button click.  

### 2) Correlation Network
The dataframe from section 1 will be carried over to this page. In here the [Pearson correlations](https://en.wikipedia.org/wiki/Pearson_correlation_coefficient) between the mineral phases in the dataframe are calculated and plotted in a network graph using [pyvis](https://pyvis.readthedocs.io/en/latest/documentation.html). Again in the sidebar a series of widgets can be used to make a subelection of the graph to display.  Each node is a mineral with some hover_over information, and each arrow is the mineral_i mineral_j correlation value (displayed in a hover_over window). The thickness of the arrows is proportional to the magnitude of the Pearson correlation.  
The app works pretty much intuitively, in the sidebar there is a series of filters that can be used from top to bottom, to filter down the dataframe. 
The correlation matrix  obtained is displayed under the network graph, and there is the possibility to save it as a csv with a button click.

### 3) Similarity Query
This section enables the user to perform a [Jaccard Similarity](https://neo4j.com/docs/graph-data-science/current/algorithms/node-similarity/) directly on the whole database. The calculation is done through neo4j [Graph Data Science library (gds)](https://neo4j.com/docs/graph-data-science/current/algorithms/), so it follows the regular gds workflow. To calculate quantities with gds we need to create a temporary projection of the graph in the database (for more info see the relevant section in gds documentation). To do that do the following:  
1-Type a name in the first widget  
2-Select the Sample property of which you want to find similar sample of, e.g.: You're looking for samples with similar longitude/latitude/depth/etc..  
3-Select the Relationship property that you want to use as a weight for the similarity calculation, e.g: You're looking for samples with similar longitude WEIGHTED ON their mass_percentage composition of the minerals they share  
4-Hit create SUBGRAPH  

If you didn't try to overwrite a previous subgraph the similarity values should now show up. Otherwise it means that you forgot to clear your old subgraph, so hit DELETE SUBGRAPH and try again.


### 4) Database Management
This section is designated to the update, upkeep and backup of the database and should be used with care. It is loosely password protected with a hashed password that can be found in the CIC_JDL_MNA/Python_code/docker_jdl-environment/ folder. The multiple menu is by now divided in the following.  

#### BACKUP AND RESTORE  
This section is designated for the backup and restore of the neo4j database. Hitting the BACKUP_DATABASE button triggers a shutdown, backup and restart of the neo4j database. This is due to how the free version of the neo4j software worksm which requires the neo4j process to be shut down before being able to dump or overwrite the current database. This is not  optimal as it increases the time required from seconds to about 2 minutes. Neo4j enterprise version doesn't have this problem, so if in the future there is a commercialization avenue for the app this might be a necessary improvement.  
**To backup the database in the current state**: This option should be taken often enough not to risk major data losses, and especially before any change to the database using the CSV INGESTER. 
To do so simply hit the BACKUP button and wait for the success message before issuing any other command.  
The backup files are automatically named backup_db_DATE-TIME.dump and stored on the server in the folder:  
/home/ubuntu/HMMA_ARCHIVE/DB_BACKUPS   
Once saved the users can retrieve them from the folder and name them as they are pleased  

**To restore the database to a previous state**: If the database has been muddled with in a way that is difficult to correct you may want to restore it to a previous point in time (obtained previously through the backup utility).  
To do so copy the .dump file that contains the state of the database that you want to restore in the folder:  
/home/ubuntu/HMMA_ARCHIVE/DB_RESTORE  
Once there the file will appear in the select menu in the app, select the file that you want to restore and then hit the RESTORE button and wait until the success message before issuing any other command.  Your database should now have been restored to the state you selected.  

#### CSV INGESTER
Allows a user to add data to the database through a CSV file. This can be either a samples csv or a mineral properties csv. The CSV must respect some format:  
**For samples.csv:**  
1-The first column must be the sample id  
2-Column names and values MUST avoid special characters such as ' " ` and blank spaces      
3-Column names and values SHOULD avoid other special characters as much as possible. Stick to underscores or dashes if possible, the program is equipped to deal with most of them, but this has not been extensively tested  
4-Mineral phases columns should come after the sample id column and before the meta properties  
5-Meta properties columns (longitude,latitude...etc) should come AFTER the Mineral phases  
6-If a column contains multiple values that must be separated, write them in the format of value1;value2;value3 . AVOID blank spaces  

**For mineral_properties.csv**:  
1- The column containing the names of the minerals must be named Mineral  
2- The same indication for columns names and values are valid here.   
3- In general, respect the same format of the csvs in CIC_JDL_MNA/Python_code/docker_jdl-environment/data/  

Now for the actual instructions to use the page:  

1-Once you have the CSVs to upload hit the uploaders widgets and press UPLOAD.  
2- Select the column that contains the ID (should be the first, but just in case)  
3- Select the column that contains the last mineral phase. Columns from here on will be treated as meta properties.  
4- IMPORTANT. Give a name to the numerical property of the samples.csv that you are uploading. e.g: grain_counts, mass_percent ...etc  
5- IMPORTANT. If you're uploading some non numerical metaproperties e.g: ClientName, Color..etc , be sure to tick the relevant box widget  

Now a Typo check will be displayed with relevant explanation. When you are sure that the data you are uploading is clean and nicely formatted you can hit the relevant upload buttons.   
The Normalize data checkbox normalizes the uploaded sample quantities (e.g. mass_percent,grain_counts etc) to 1 sample by sample.    

### 5) About
Display some links to Geoscience Australia and John de Laeter center.


# R App
This part is just a docker redeployment of the HMMA app provided by Alex Walker. 

## To install the app
1- install docker on the server   
2- clone the github repo wherever you want to install (install_dir)     
3- navigate to install_dir/CIC_JDL_MNA/R_code  
4- type:  docker pull rocker/shiny  
5- type:  docker build -t r-hmma .  
6- type:  docker run -d -p 3838:3838 --name r-hmma -v ~/CIC_JDL_MNA/R_code:/srv/shiny-server/ -v ~/CIC_JDL_MNA/R_code/logs/:/var/log/shiny-server/ r-hmma  
The app should be running at  [http://localhost:3434/](http://localhost:3434/)
