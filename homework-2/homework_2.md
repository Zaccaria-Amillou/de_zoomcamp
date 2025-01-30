# Module 2 Homework

## Instructions
- If you don’t find an exact match for an option, select the closest one.
- For this homework, you’ll work with the **Green Taxi dataset**, which can be accessed [here](https://github.com/DataTalksClub/nyc-tlc-data/releases/tag/green/download).


---

## Assignment
So far in the course, we’ve processed data for the years **2019** and **2020**. Your task is to extend the existing workflows to include data for the year **2021**.

---

## Quiz Questions
Test your understanding of workflow orchestration, Kestra, and ETL pipelines for data lakes and warehouses by answering the following multiple-choice questions.

### Question 1 
Within the execution for Yellow Taxi data for the year **2020** and month **12**, what is the uncompressed file size (i.e., the output file `yellow_tripdata_2020-12.csv` of the extract task)?  
- [X] 128.3 MB  
- [ ] 134.5 MB  
- [ ] 364.7 MB  
- [ ] 692.6 MB  

---

### Question 2  
What is the value of the variable `file` when the inputs are:  
- `taxi` = `green`  
- `year` = `2020`  
- `month` = `04`  

Options:  
- [ ] `{{inputs.taxi}}_tripdata_{{inputs.year}}-{{inputs.month}}.csv`  
- [X] `green_tripdata_2020-04.csv`  
- [ ] `green_tripdata_04_2020.csv`  
- [ ] `green_tripdata_2020.csv`  

---

### Question 3  
How many rows are there for the Yellow Taxi data for the year **2020**?  
- [ ] 13,537,299  
- [X] 24,648,499  
- [ ] 18,324,219  
- [ ] 29,430,127  

Query:  
```sql
SELECT COUNT(*) 
FROM yellow_tripdata
WHERE CONTAINS_SUBSTR(filename, '2020')
LIMIT 1000;
```

### Question 4
How many rows are there for the Green Taxi data for the year 2020?
- [ ] 5,327,301
- [ ] 936,199  
- [X] 1,734,051 
- [ ] 1,342,034

Query:
  ```sql
SELECT COUNT(*)
FROM green_tripdata
WHERE CONTAINS_SUBSTR(filename, '2020')
LIMIT 1000
```

### Question 5
How many rows are there for the Yellow Taxi data for March 2021?
- [ ] 1,428,092
- [ ] 706,911  
- [X] 1,925,152
- [ ] 2,561,031


Query:
  ```sql
SELECT COUNT(*)
FROM yellow_tripdata_2021_03
```

### Question 6
How would you configure the timezone to New York in a Schedule trigger?
  
- [ ] Add a timezone property set to EST in the Schedule trigger configuration
- [X] Add a timezone property set to America/New_York in the Schedule trigger configuration
- [ ] Add a timezone property set to UTC-5 in the Schedule trigger configuration
- [ ] Add a location property set to New_York in the Schedule trigger configuration
