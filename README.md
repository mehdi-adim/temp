# Ambient Wanderer back-end's.
  
   1- [Install AW back-end using Docker](#1-install-and-use-ambient-wanderer-back-end-using-docker).
   
   2- [Running PACE scenario](#2-pace-scenario) : two ways to get recommendation (using CLI or API).
   
   - [Pace scenrio using Gowalla Dataset](#1-pace-scenrio-using-gowalla-dataset)
        
   - [Pace scenario using Foursquare Dataset](#2-pace-scenario-using-foursquare-dataset)
            
   3- [Running GeoMF scenario](#3-geomf-scenario) : two ways to get recommendation (using CLI or API).
      
   - [GeoMF scenario using Gowalla Dataset](#1-geomf-scenario-using-gowalla-dataset)
    
   - [GeoMF scenario using Foursquare Dataset](#2-geomf-scenario-using-foursquare-dataset)
     
   4- [AW API Documentation](#4-aw-api-documentation).
   
   - [Switch between Foursquare and Gowalla throw API](#switch-between-foursquare-and-gowalla-throw-api).
   
   5- [Design and Architecture](#5-design-and-architecture).
           
---
### 1. Install and use Ambient Wanderer back-end using Docker.

- Requirements : 
    - **Docker** version 18.09 (or newer).
    - **Docker-compose** version 1.17 (or newer).
    - NLE VPN connection is required.
    - Use any SFTP client to access:  **wood.int.europe.naverlabs.com**.
    - Get **Gowalla** Dataset:
        - Data is located on **/gfs/project/likemind/gwalla**. 
        - Load data to your local (download folder path) using the SFTP client. 
        - Or (you can use secure copy over ssh connection on wood server to clone data).
    - Get **Foursquare** Dataset:
        - Data is located on **/gfs/project/likemind/foursquare**. 
        - Load data to your local (download folder path) using the SFTP client. 

- Cloning project:
    - VPN connection & https://oss.navercorp.com/situational-recommender/ authentication are required:
    - Cloning the project: 
        ```
        git clone https://oss.navercorp.com/situational-recommender/LikeMind.git
        cd LikeMind
        ```

- Change environment path GOWALLA_PATH and FOURSQUARE_PATH in .env file:
    - .env file is located in LikeMind root path:
    - Edit file and Change GOWALLA_PATH and FOURSQUARE_PATH to your local data paths (for foursquare and gowalla).
    - To edit .env file on Unix based OS (Linux, Mac OS) use:
        ```
        nano .env
        ```
    - On windows: 
        - show hidden files and edit .env file on any text editor.
        - Change GOWALLA_PATH to your local data path (download folder path for gowalla).
        - Change FOURSQUARE_PATH to your local data path (download folder path for foursquare).
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
        docker-compose logs -f gowalla
        docker-compose logs -f foursquare
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
        docker exec -it <CONTAINER_ID> python pace/core/make_predictions.py <ID_USER> <K> <database>
        ```
        - CONTAINER_ID is the id of API container.
    
        - K : is the number of Top-k pois 
        
        - database : gowalla or foursquare
    
        #### 1. Pace scenrio using Gowalla Dataset
        ```
        docker container ls
        ```
        - List containers :
        
          ![ERM](docs/container_id.png)
        
        - Run:
            - The user 12062 checked-in locations related to foodie person 
            
            - we make a prediction using PACE : with user 12062 and 7 as top-7 pois to recommend.
            
            ```com
            docker exec -it 5f11c4d6e111 python pace/core/make_predictions.py 12062 7 gowalla
            ```
       - You get:
       
          The probability score is the relevance probability between user and poi.
          
            ![ERM](docs/pace_output_cli.png)
            
       #### 2. Pace scenario using Foursquare Dataset
        
        - Run:
            
            - we make a prediction using PACE : with user 22207 and 7 as top-7 pois to recommend.
            
            ```com
            docker exec -it 5f11c4d6e111 python pace/core/make_predictions.py 22207 7 foursquare
            ```
        - You get:
       
          The probability score is the relevance probability between user and poi.
          
            ![ERM](docs/cli_output_pace_foursquare.png).
            
   - Changing pace config and re-train the model:
        - Edit pace/core/pace_config.py
        - Rebuild api service:
            ```
            docker-compose up --build --force-recreate --no-deps -d api
            ```
        - Overwrite and Train again:
            ```
            docker exec -it 5f11c4d6e111 python pace/core/model.py <database>
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
        docker exec -it <CONTAINER_ID> python geoMF/core/make_predictions.py <ID_USER> <K> <database>
        ```
        - CONTAINER_ID is the id of API container.

        - K : is the number of Top-k pois 
        
        - database : foursquare or gowalla

    
   #### 1. GeoMF scenario using Gowalla Dataset

   ```
    docker container ls
   ```
    
   - List containers :
    
      ![ERM](docs/container_id.png)
    
   - Run:

   The user 28 checked-in locations related to these categories: 
    
   'Dive Bar', 'Fountain', 'Historical Landmark'. related to entertainment and discovering
    
   - we make a prediction using GeoMF : with user 28 and 5 as top-5 pois to propose.
    
   ```
    docker exec -it 5f11c4d6e111 python geoMF/core/make_predictions.py 28 5 gowalla
   ```

   - You get:

       ![ERM](docs/geomf_output_cli.png)
            
   #### 2. GeoMF scenario using Foursquare Dataset
        
   - Run:
    
        The user 28 checked-in locations related to these categories: 
        
        'Plaza', 'Soccer Field', 'cafe', 'Pizza Place' related to entertainment
        
        - we make a prediction using GeoMF : with user 28 and 5 as top-5 pois to propose.
        
        ```
        docker exec -it 5f11c4d6e111 python geoMF/core/make_predictions.py 28 5 foursquare
        ```
   
   - You get:
            
     ![ERM](docs/geo_output_cli_foursquare.png)
            

   - Get prediction throw API:
   
        The user 28 checked-in locations related to entertainment and discovering: 
        'Dive Bar', 'Fountain', 'Historical Landmark'.  
        
        - Request to /recommend_geoMF:
        
            ![ERM](docs/geomf_input.png)
            
        - Response:
            
            Top-5 POI recommended by GeoMF.
            
            ![ERM](docs/geoMf_output.png)

---

### 4. AW API Documentation
   -  API Documentation
    
      We used postman to publish documentation, is available at the [link](https://documenter.getpostman.com/view/10475669/SzKWuHcQ?version=latest).

   -  Using API      
        - Recommendation endpoint: Sending HTTP request through curl client (Ask for recommendation).
            ```
            curl --location --request GET 'http://0.0.0.0:5000/recommend' \
            --header 'Content-Type: application/json' \
            --header 'Content-Type: application/json' \
            --data-raw '{
                "radius": 50,
                "context_location": {
                    "lat": 48.860755,
                    "lng": 2.352266
                },
                "user_mindset": 5,
                "user_local_time": "2018-06-29 12:21:41"
            }'
            ```
        - Bookmarking endpoint: Sending HTTP request through curl client (Bookmarking POI).
            ```
            curl --location --request POST 'http://10.57.19.62:5010/bookmark' \
            --header 'Content-Type: application/json' \
            --header 'Content-Type: application/json' \
            --data-raw '{
            "user_id": 71878,
            "poi_id": 11550
            }'
            ```
        - Recommendation request through API client (Postman) 

            ![ERM](docs/postman_example.png)
            
   #### Switch between Foursquare and Gowalla throw API
   
   To switch to Foursquare Dataset:
   
   ```
   curl --request POST 0.0.0.0:5000/switch/foursquare
   ```
   To switch to Foursquare Dataset:
   
   ```
   curl --request POST 0.0.0.0:5000/switch/gowalla
   ```
   
 ---
 ### 5. Design and Architecture

   - Data Modeling: UML Entity relationship model. 

        ![ERM](docs/ERD.jpg)

   - Software design architecture
        
        - Zoom on LikeMind
        
            ![Zoom on LikeMind](docs/Architecture.png)
            
        - Zoom on PACE 
        
            ![Zoom on PACE](docs/GeoMF_Architecture.png)
