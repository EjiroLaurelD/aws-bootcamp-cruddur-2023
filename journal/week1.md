# Week 1 — App Containerization
- I started the week project by creating my docker files in the front and back ends
-  I cd'd into the backend directory and ran the command `pip3 Install -r requirements.txt` to set up my environment
- I added the environment variables and ran the python command to start flask
```
export FRONTEND_URL="*"
export BACKEND_URL="*"
python3 -m flask run --host=0.0.0.0 --port=4567
```

I confirmed my api end point was working with the url https://4567-ejirolaurel-awsbootcamp-59goxtf5kfq.ws-eu87.gitpod.io/api/activities/home
![Port300](assets/api-endpoint.png)  


I unset my environment variables to be sure it doesnt conflict with docker using the commands
```
unset BACKEND_URL
unset FRONTEND_URL
```

- I cd'd to the work directory /workspace/aws-bootcamp-cruddur-2023 and built the docker image for the backend flask with the command
`docker build -t  backend-flask ./backend-flask` 

- I ran the docker image with the command below with environment variables attached
 `docker run --rm -p 4567:4567 -it -e FRONTEND_URL='*' -e BACKEND_URL='*' backend-flask`

- I confirmed my endpoint was working fine 
https://4567-ejirolaurel-awsbootcamp-59goxtf5kfq.ws-eu87.gitpod.io/api/activities/home
![Backend-endpoint](assets/backend-api.png)  

After confirming my docker image for the backened i cd'd into the frontend directory and ran npm install
```
cd frontend-react-js
npm i
```
Then i cd'd back to the work directory and created the docker-compose.yml file with the code 

```
version: "3.8"
services:
  backend-flask:
    environment:
      FRONTEND_URL: "https://3000-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
      BACKEND_URL: "https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
    build: ./backend-flask
    ports:
      - "4567:4567"
    volumes:
      - ./backend-flask:/backend-flask
  frontend-react-js:
    environment:
      REACT_APP_BACKEND_URL: "https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
    build: ./frontend-react-js
    ports:
      - "3000:3000"
    volumes:
      - ./frontend-react-js:/frontend-react-js

# the name flag is a hack to change the default prepend folder
# name when outputting the image names
networks: 
  internal-network:
    driver: bridge
    name: cruddur
```

- I ran `docker ccompose up` to start docker compose. I unlocked my ports and previewed the app on port 3000  https://3000-ejirolaurel-awsbootcamp-59goxtf5kfq.ws-eu87.gitpod.io/
## Cruddr App Home page
![cruddrapp](assets/cruddrapp.png) 

## Sign in page
![signin](assets/signin.png) 

## Sign up page
![signup](assets/signup.png)  

## Edited backend homepage
- Here i edited the backend services file, home-activities.py to edit Handler "Andrew Brown" to my name and "Cloud is fun" to "Cloud is very fun"
![homepage](assets/editedbackend.png)  

## Created new files for Notification on the back end and front end 
- Here i edited the files following the video tutorial
![notifications](assets/notifications.png)  

I edited my docker compose file and included dynamodb and postgres
I ran `docker compose up` then went on to create a the table for psql on the terminal using aws cli with the follwoing code
## create table
```
aws dynamodb create-table \
    --endpoint-url http://localhost:8000 \
    --table-name Music \
    --attribute-definitions \
        AttributeName=Artist,AttributeType=S \
        AttributeName=SongTitle,AttributeType=S \
    --key-schema AttributeName=Artist,KeyType=HASH AttributeName=SongTitle,KeyType=RANGE \
    --provisioned-throughput ReadCapacityUnits=1,WriteCapacityUnits=1 \
    --table-class STANDARD
```
## create item 
```
aws dynamodb put-item \
    --endpoint-url http://localhost:8000 \
    --table-name Music \
    --item \
        '{"Artist": {"S": "No One You Know"}, "SongTitle": {"S": "Call Me Today"}, "AlbumTitle": {"S": "Somewhat Famous"}}' \
    --return-consumed-capacity TOTAL  
```
## List tables 
`aws dynamodb list-tables --endpoint-url http://localhost:8000`

## Get records
`aws dynamodb scan --table-name cruddur_cruds --query "Items" --endpoint-url http://localhost:8000`

I made sure the postgress client will be always installed by adding the code below to my gitpod.yml file
```
  - name: postgres
    init: |
      curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc|sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/postgresql.gpg
      echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" |sudo tee  /etc/apt/sources.list.d/pgdg.list
      sudo apt update
      sudo apt install -y postgresql-client-13 libpq-dev

```
I checked psql was working well with the command 
`psql -Upostgres --host localhost`
I inputed my password and used the postgress command `/l` to confirm the list of tables 
![postgres](assets/postgres.png)  
