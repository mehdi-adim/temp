# Ambient Wanderer back-end's installation.
    
   The installation and usage guide contains for moment three part:
  
   [Install and use Ambient Wanderer back-end using Docker](#1-install-and-use-ambient-wanderer-back-end-using-docker)
   
   [PACE scenario](#2-pace-scenario)
   
   [GeoMF scenario](#3-geomf-scenario)
           

### 1. Install and use Ambient Wanderer back-end using Docker

- Requirements : 
        - **Docker** version 18.09 (or newer).
        - **Docker-compose** version 1.17 (or newer).
        - Get **Gowalla** Dataset:
            - NLE VPN connection is required.
            - Use any SFTP client to access:  **wood.int.europe.naverlabs.com**.
            - Data is located on **/gfs/project/likemind/gwalla**. 
            - Load data to your local (download folder path) using the SFTP client. 
            - Or (you can use secure copy over ssh connection on wood server to clone data).
            
    - Cloning project:
        - VPN connection & https://oss.navercorp.com/situational-recommender/ authentication are required:
        - Cloning the project: 
            ```
            git clone https://oss.navercorp.com/situational-recommender/LikeMind.git
            cd LikeMind
            ```
     
    - Change environment path DATA_PATH in .env file:
        - .env file is located in LikeMind root path:
        - Edit file and Change DATA_PATH to your local data path (download folder path).
        - To edit .env file on Unix based OS (Linux, Mac OS) use:
            ```
            nano .env
            ```
        - On windows: 
            - show hidden files and edit .env file on any text editor.
            - Change DATA_PATH to your local data path (download folder path).
            - In a CMD windows: **SET COMPOSE_CONVERT_WINDOWS_PATHS=1**
            - Restart docker-engine.
    
    - Build services & install dependencies:
        ```
        docker-compose build && docker-compose up -d
        ```
    - Wait until data populating is done. and server is running on localhost.
        -  Maybe you might see a warning on the hint on “max_wal_size”, because the operation is exceptionally huge (uploading 1.5 GB of check-ins to postgres container). 
        -  To avoid this warning and make a Database Populating more efficient. We can use some techniques there are suggestions on postgres documentation:
        https://www.postgresql.org/docs/current/populate.html
    
        - you can see postgres service logs by running :
    
            ```
            docker-compose logs -f postgresdb
            ```
        - After Data populating process is ended, the database is ready to use: 
        
          ![ERM](docs/pgsql_logs.png)
        
        - you can see api service logs by running :
            ```
            docker-compose logs -f api
            ```
        - After Data populating process is ended, the flask server is successfully running: 
        
          ![ERM](docs/api_logs.png)
---

### 2. PACE scenario
    
   - When services are up, you can run PACE predictions by:
    
        ```
        docker exec -it <CONTAINER_ID> python pace/core/make_predictions.py <ID_USER> <K>
        ```
        - CONTAINER_ID is the id of API container.
    
        - K : is the number of Top-k pois 
    
    
   - Example :
        ```
        docker container ls
        ```
        - List containers :
        
          ![ERM](docs/container_id.png)
        
        - Run:
            - The user 12062 checked-in locations related to foody person 
            
            - we make a prediction using PACE : with user 12062 and 7 as top-7 pois to recommend.
            
            ```
            docker exec -it 5f11c4d6e111 python pace/core/make_predictions.py 12062 7
            ```
       - You get:
       
          The probability score is the relevance probability between user and poi.
          
            ![ERM](docs/pace_output_cli.png)
            
   - Changing pace config and re-train the model:
        - Edit pace/core/pace_config.py
        - Rebuild api service:
            ```
            docker-compose up --build --force-recreate --no-deps -d api
            ```
        - Overwrite and Train again:
            ```
            docker exec -it 5f11c4d6e111 python pace/core/model.py
            ```
   - Get prediction throw API:
        - The user 12062 checked-in locations related to foody person
        
        - Request to /recommend_pace:
        
            ![ERM](docs/pace_input_api.png)
            
        - Response:
            
            Top-5 POI recommended by PACE:
            
            ![ERM](docs/pace_output_api.png)

---
### 3. GeoMF scenario

    - When services are up, you can run GeoMF predictions by:
    
        ```
        docker exec -it <CONTAINER_ID> python geoMF/core/make_predictions.py <ID_USER> <K>
        ```
        - CONTAINER_ID is the id of API container.
    
        - K : is the number of Top-k pois 
    
    
   - Example :
        ```
        docker container ls
        ```
        - List containers :
        
          ![ERM](docs/container_id.png)
        
        - Run:
        
            The user 28 checked-in locations related to these categories : 
            
            'Dive Bar', 'Fountain', 'Historical Landmark'. related to entertainment and discovering
            
            - we make a prediction using GeoMF : with user 28 and 5 as top-5 pois to propose.
            
            ```
            docker exec -it 5f11c4d6e111 python geoMF/core/make_predictions.py 28 5
            ```
       - You get:
            
            ![ERM](docs/geomf_output_cli.png)
            

   - Get prediction throw API:
   
          The user 28 checked-in locations related to entertainment and discovering: 
          'Dive Bar', 'Fountain', 'Historical Landmark'.  
        - Request to /recommend_geoMF:
        
            ![ERM](docs/geomf_input.png)
            
        - Response:
            
            Top-5 POI recommended by GeoMF.
            
            ![ERM](docs/geoMf_output.png)

