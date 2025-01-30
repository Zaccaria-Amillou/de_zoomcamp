# Module 1 Homework: Docker & SQL

In this homework, we'll prepare the environment and practice Docker and SQL.

When submitting your homework, you will also need to include a link to your GitHub repository or other public code-hosting site. This repository should contain the code for solving the homework. When your solution has SQL or shell commands and not code (e.g., Python files), include them directly in the README file of your repository.

---

## Question 1: Understanding Docker First Run

Run Docker with the `python:3.12.8` image in an interactive mode, using the entrypoint `bash`.

**What's the version of pip in the image?**

- [X] 24.3.1
- [ ] 24.2.1
- [ ] 23.3.1
- [ ] 23.2.1

```bash
$ winpty docker run -it python:3.12.8 bash
Unable to find image 'python:3.12.8' locally
3.12.8: Pulling from library/python
e00350058e07: Download complete
5f16749b32ba: Download complete
eb52a57aa542: Download complete
Digest: sha256:9cdef3d6a7d669fd9349598c2fc29f5d92da64ee76723c55184ed0c8605782cc
Status: Downloaded newer image for python:3.12.8
root@067488efc63d:/# pip --version
pip 24.3.1 from /usr/local/lib/python3.12/site-packages/pip (python 3.12)
```
## Question 2: Understanding Docker Networking and Docker-Compose

Given the following docker-compose.yaml, what is the hostname and port that pgAdmin should use to connect to the PostgreSQL database?

```docker
services:
  db:
    container_name: postgres
    image: postgres:17-alpine
    environment:
      POSTGRES_USER: 'postgres'
      POSTGRES_PASSWORD: 'postgres'
      POSTGRES_DB: 'ny_taxi'
    ports:
      - '5433:5432'
    volumes:
      - vol-pgdata:/var/lib/postgresql/data

  pgadmin:
    container_name: pgadmin
    image: dpage/pgadmin4:latest
    environment:
      PGADMIN_DEFAULT_EMAIL: "pgadmin@pgadmin.com"
      PGADMIN_DEFAULT_PASSWORD: "pgadmin"
    ports:
      - "8080:80"
    volumes:
      - vol-pgadmin_data:/var/lib/pgadmin  

volumes:
  vol-pgdata:
    name: vol-pgdata
  vol-pgadmin_data:
    name: vol-pgadmin_data
  ```

  **Options:**
- [ ] postgres:5433
- [ ] localhost:5432
- [ ] db:5433
- [ ] postgres:5432
- [X] db:5432

---

## Question 3: Trip Segmentation Count

During the period of October 1st, 2019 (inclusive) and November 1st, 2019 (exclusive), how many trips, respectively, happened:
- Up to 1 mile
- In between 1 (exclusive) and 3 miles (inclusive)
- In between 3 (exclusive) and 7 miles (inclusive)
- In between 7 (exclusive) and 10 miles (inclusive)
- Over 10 miles

```sql
-- Up to 1 mile
SELECT COUNT(1)
FROM green_tripdata t
WHERE t."trip_distance" <= 1;

-- In between 1 (exclusive) and 3 miles (inclusive)
SELECT COUNT(1)
FROM green_tripdata t
WHERE t."trip_distance" <= 3
  AND t."trip_distance" > 1;

-- In between 3 (exclusive) and 7 miles (inclusive)
SELECT COUNT(1)
FROM green_tripdata t
WHERE t."trip_distance" <= 7
  AND t."trip_distance" > 3;

-- In between 7 (exclusive) and 10 miles (inclusive)
SELECT COUNT(1)
FROM green_tripdata t
WHERE t."trip_distance" <= 10
  AND t."trip_distance" > 7;

-- Over 10 miles
SELECT COUNT(1)
FROM green_tripdata t
WHERE t."trip_distance" > 10;
```

**Options:**

- [ ] 104,802; 197,670; 110,612; 27,831; 35,281
- [ ] 104,802; 198,924; 109,603; 27,678; 35,189
- [ ] 104,793; 201,407; 110,612; 27,831; 35,281
- [ ] 104,793; 202,661; 109,603; 27,678; 35,189
- [X] 104,838; 199,013; 109,645; 27,688; 35,202

---

## Question 4: Longest Trip for Each Day

Which was the pickup day with the longest trip distance? Use the pickup time for your calculations.

SQL Query:
```sql
SELECT
    CAST(lpep_pickup_datetime AS DATE) AS "day",
    MAX(trip_distance) AS "max_dist"
FROM green_tripdata t
GROUP BY CAST(lpep_pickup_datetime AS DATE)
ORDER BY "max_dist" DESC
LIMIT 100;
```

Options:
- [ ] 2019-10-11
- [ ] 2019-10-24
- [ ] 2019-10-26
- [X] 2019-10-31

---

## Question 5: Three Biggest Pickup Zones

Which were the top pickup locations with over 13,000 in total_amount (across all trips) for 2019-10-18?

SQL Query:
```sql
SELECT
    CAST(lpep_pickup_datetime AS DATE) AS "day",
    SUM(total_amount) AS "sum",
    zpu."Zone" AS "PickUP_zone"
FROM green_tripdata t
LEFT JOIN zones zpu
    ON t."PULocationID" = zpu."LocationID"
WHERE DATE(lpep_pickup_datetime) = '2019-10-18'
GROUP BY 1, 3
HAVING SUM(total_amount) > 13000
ORDER BY "sum" DESC;
```

**Options:**
- [X] East Harlem North, East Harlem South, Morningside Heights
- [ ] East Harlem North, Morningside Heights
- [ ] Morningside Heights, Astoria Park, East Harlem South
- [ ] Bedford, East Harlem North, Astoria Park

---

## Question 6: Largest Tip

For the passengers picked up in October 2019 in the zone named "East Harlem North," which was the drop-off zone that had the largest tip?

SQL Query:
```sql
SELECT
    CAST(lpep_pickup_datetime AS DATE) AS "day",
    tip_amount,
    zpu."Zone" AS "PickUP_zone",
    zdo."Zone" AS "DropOFF_zone"
FROM green_tripdata t
LEFT JOIN zones zpu
    ON t."PULocationID" = zpu."LocationID"
LEFT JOIN zones zdo
    ON t."DOLocationID" = zdo."LocationID"
WHERE DATE(lpep_pickup_datetime) >= '2019-10-01'
  AND DATE(lpep_pickup_datetime) <= '2019-10-31'
  AND zpu."Zone" = 'East
  ```

**Options:**
- [ ] Yorkville West
- [X] JFK Airport
- [ ] East Harlem North
- [ ] East Harlem South

---

## Question 7. Terraform Workflow

Which of the following sequences, respectively, describes the workflow for:

Downloading the provider plugins and setting up backend,
Generating proposed changes and auto-executing the plan
Remove all resources managed by terraform`

**Options:**
- [ ] terraform import, terraform apply -y, terraform destroy
- [ ] teraform init, terraform plan -auto-apply, terraform rm
- [ ] terraform init, terraform run -auto-approve, terraform destroy
- [X] terraform init, terraform apply -auto-approve, terraform destroy
- [ ] terraform import, terraform apply -y, terraform rm
