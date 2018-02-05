# councilmatic-docker
Docker instance for councilmatic-scraper.  

## Quickstart

###  Make sure you have Docker installed for your OS
1. https://www.docker.com/

### Edit volumes docker-compose.yml for your local environment

1. postgres
  * Create empty directory.
  * Replace "\<\<local_db_data_dir\>\>" with the full path to that directory.
2. solr
  * Create empty directory.
  * Replace "\<\<local_solr_data_dir\>\>" with the full path to that directory.
3. councilmatic-scraper
  * If you don't have the latest copy of councilmatic-scraper, you can clone it from Github:
  
```
   git clone https://github.com/openoakland/councilmatic-scraper

```
  * Replace "\<\<local_councilmatic-scraper_git_repo_dir\>\>" with the full path to councilmatic-scraper directory.
   
   
   
*Currently (12/13/17), we are working on the "events" branch. Switch use the "events" branch until further notice.*

```	
cd councilmatic-scraper
git checkout events
```

   
The local db data directory and the councilmatic scraper git repo directory cannot be subdirectories of each other. If you're still not sure what to set, you can take a look at docker_compose_files/docker-compose.yml.sample. This is how I have things set up for my Mac OS X:
```
version: "3"

services:
  postgres:
    image: ekkus93/councilmatic-docker:latest
    container_name: councilmatic_postgres
    environment:
      - POSTGRES_PASSWORD=str0ng*p4ssw0rd
      - PGDATA=/var/lib/postgresql/data
    volumes:
      ### Sample
      - <<local_db_data_dir>>:/var/lib/postgresql/data
      - <<local_councilmatic-scraper_git_repo_dir>>:/home/postgres/work            
      ### Phil
      #- /Users/phillipcchin/work/councilmatic/councilmatic-scraper-data:/var/lib/postgresql/data
      #- /Users/phillipcchin/work/councilmatic/councilmatic-scraper:/home/postgres/work
      ### Howard
      #- /Users/matis/Dropbox/OpenOakland/councilmatic-scraper-data:/var/lib/postgresql/data
      #- /Users/matis/Dropbox/OpenOakland/councilmatic-scraper:/home/postgres/work      
    ports:
      - 5432:5432
      - 8888:8888
  solr:
    image: solr
    container_name: councilmatic_solr
    ports:
     - "8983:8983"
    volumes:
      ### Sample
      - <<local_solr_data_dir>>:/opt/solr/server/solr/mycores
      ### Phil
      # - /Users/phillipcchin/work/councilmatic/councilmatic-solr:/opt/solr/server/solr/mycores
      ### Howard
      # - ???:/opt/solr/server/solr/mycores
    entrypoint:
      - docker-entrypoint.sh
      - solr-precreate
      - mycore
```

If you want, you can also change the POSTGRES_PASSWORD.  You will need this if you want to connect remotely to the database as "postgres".  The default postgres port, 5432, has been exposed.  You should be able to connect to the database on that port on 127.0.0.1 or the ip of your Docker host instance.

### Start docker instance with docker-compose
In directory with docker-compose.yml, run:
```
docker-compose up -d
```
**_To force downloading the latest version of the docker container:_**
```
docker-compose pull && docker-compose up -d
```

### Connect to your docker instance

1. Get the container id for your instance and then log in as postgres
```
docker ps
docker exec -it -u postgres <<container_id>> bash
```
or 
```
container_id=$(docker ps > d.tmp;grep -m 1 "councilmatic-docker" d.tmp | awk '{print $1}'); docker exec -it -u postgres $container_id bash
```
2. Activate councilmatic-scraper virtualenv
```
source /home/postgres/councilmatic/bin/activate
```
or
```
source_councilmatic
```
3. cd into councilmatic-scraper repo directory
```
cd /home/postgres/work
```

### Initialize database (**Only run once**)
```
cd /home/postgres/scripts
sh setup_db.sh
```
This will create the opencivicdata database for you and init it for US.  You do not have to run "createdb opencivicdata" or "pupa dbinit us".  It probably won't do any harm but it will give you errors.

After you have initialized your database, files should appear in your local db data directory. 

### Run pupa update (for Oakland)
```
cd /home/postgres/work
pupa update oakland
```

The work directory should have the directory "oakland".  These Python files inside the directory have already been generated by "pupa init" and have been updated.

### Testing Solr instance

Going to this url with a browser outside the container should bring up the Solr admin website:
```
http://localhost:8983/solr/
```

### Running Jupyter Notebook

1. Inside the container, start councilmatic-scraper virtualenv:
```
source_councilmatic
```
2. cd to your working directory (i.e. /home/postgres/work)
3. Run the following command:
```
jupyter notebook --no-browser --ip=0.0.0.0
```
4. You should see something like:
```
(councilmatic) postgres@dc9cd4f71cee:~/work$ jupyter notebook --no-browser --ip=0.0.0.0
[I 08:55:05.065 NotebookApp] Serving notebooks from local directory: /home/postgres/work
[I 08:55:05.066 NotebookApp] 0 active kernels
[I 08:55:05.066 NotebookApp] The Jupyter Notebook is running at:
[I 08:55:05.067 NotebookApp] http://0.0.0.0:8888/?token=2842b15e3c8460139a9d1d3ad2ce04cb53953e673da79fb2
[I 08:55:05.068 NotebookApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
[C 08:55:05.070 NotebookApp] 
    
    Copy/paste this URL into your browser when you connect for the first time,
    to login with a token:
        http://0.0.0.0:8888/?token=2842b15e3c8460139a9d1d3ad2ce04cb53953e673da79fb2
```
5. Copy url with token and replace "0.0.0.0" with "127.0.0.1".  Open the url with a web browser from your host.

### Shut down your docker instance
```
docker-compose down
```

You should do this to shut down the docker instance and release the volume mounts cleanly.  Don't just do "docker stop..."
