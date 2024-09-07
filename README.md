# Movie_Data_Analysis

Process and Ingest only the quality movie data in Redshift Data Warehouse

# Tech Stack
Extract:

## S3
Glue Crawler
Glue Catalog

## Transform:
Glue Catalog Table Data Quality
Glue Low Code ETL

## Load:
Redshift

## Alerting:
Event Bridge
SNS

# S3

Directory’s structure:
![image](https://github.com/user-attachments/assets/ce4da950-5d64-4baa-b7d3-cdf829d736c6)

Create a crawler for the S3 input file:
![image](https://github.com/user-attachments/assets/40d571bb-14fb-41ce-afe3-4b9daa9181d7)

Run the above crawler and check the catalog table created for the S3 input file - imdb_movies_rating_csv:
![image](https://github.com/user-attachments/assets/86bc75ca-c3f3-471a-8237-e2f4798b5038)

![image](https://github.com/user-attachments/assets/c81a3335-6fbc-4a4a-80ad-f66754bf3202)

# Glue Catalog Table Data Quality

Create a ruleset for Historical Data Quality checks on files present in the S3 input path:

![image](https://github.com/user-attachments/assets/5b4659ad-fe17-4a18-973b-da36ec480a4b)

Run the DQ check and save the output of the data quality results to historical_data_rule_outcome directory:
![image](https://github.com/user-attachments/assets/635efc5b-7707-42d0-bc35-6296090909bf)

DQ results will be either Passed or Fail:
![image](https://github.com/user-attachments/assets/3db9d8dc-6ca7-4354-b966-bac3f11b6258)

# Redshift

Now let’s create the destination Table in Redshift:

CREATE SCHEMA movies;

CREATE TABLE movies.imdb_movies_rating ( Poster_Link VARCHAR(MAX), Series_Title VARCHAR(MAX), Released_Year VARCHAR(10), Certificate VARCHAR(50), Runtime VARCHAR(50), Genre VARCHAR(200), IMDB_Rating DECIMAL(10,2), Overview VARCHAR(MAX), Meta_score INT, Director VARCHAR(200), Star1 VARCHAR(200), Star2 VARCHAR(200), Star3 VARCHAR(200), Star4 VARCHAR(200), No_of_Votes INT, Gross VARCHAR(20) );

Create JDBC connection for redshift cluster:
![image](https://github.com/user-attachments/assets/b6f29802-c91f-49f6-9ac8-2dcedcb4c316)

Now create a crawler using this JDBC connection to create a Catalog table inside our database

![image](https://github.com/user-attachments/assets/5fec305b-bffd-4610-82ea-e33909034883)

Run the above crawler and check the catalog table created : dev_movies_imdb_movies_rating

![image](https://github.com/user-attachments/assets/a29994da-a728-4701-b8e1-e384c5095b0a)
![image](https://github.com/user-attachments/assets/beb07f32-38b6-4721-a747-1bc058af8b0c)

# Glue Low Code ETL

![image](https://github.com/user-attachments/assets/928a9a58-1add-4967-af54-f61990048ddb)

S3 file as source:

![image](https://github.com/user-attachments/assets/0846d2d2-2e70-42aa-a757-b30713be66a5)

Transform - Evaluate Data Quality:

![image](https://github.com/user-attachments/assets/2434658b-b7b0-4ddb-87ee-3c186cba924d)

![image](https://github.com/user-attachments/assets/f9fff4ba-25d3-4883-82dc-503e4bab7472)

We’ll continue the job even if rules are failing for any record and publish the results in CloudWatch to let EventBridge match the failure pattern and later send SNS notification accordingly:

![image](https://github.com/user-attachments/assets/3b7e3b1d-9266-44c3-9c8f-535b86af9127)

We need to create VPC endpoints for Glue and Cloudwatch (to keep cloudwatch and Glue both in the same VPC so that the data transfer will happen)

Data quality outputs:

![image](https://github.com/user-attachments/assets/11a7d06d-1ecf-4490-bc9a-a892a76c53e4)

Data quality results will go in ruleOutcomes node:
![image](https://github.com/user-attachments/assets/47b75b5b-4165-4390-819f-28742d04eaea)

This is how the visual ETL would look:

![image](https://github.com/user-attachments/assets/b22df66b-eff5-4ad8-a3ca-b16b257792bf)

Store this ruleOutcomes result in rule_outcome directory in S3:

![image](https://github.com/user-attachments/assets/9a7fcd3e-46f3-4c8c-945f-117e48568300)

Conditional Router: Now the row level outcomes can be further divided as Failed_Records and Passed_Records(i.e., default_group)

![image](https://github.com/user-attachments/assets/d30ff90c-feb8-4b3a-a8b3-65bf18d97c12)

Conditions for Failed_Records in the conditional router:

![image](https://github.com/user-attachments/assets/9ee90768-192b-49a0-a114-b48a56314a79)

Keep these Failed_Records in bad_records directory of S3:

![image](https://github.com/user-attachments/assets/47af5112-7b27-47a3-a78a-84ad5ff686ae)

Now from the Passed_Records (i.e., default_group), drop the Drop the DataQuality columns. For other columns keep the Data type as expected in the destination redshift table(imdb_movies_rating).

![image](https://github.com/user-attachments/assets/801fc02e-80c7-4375-ae8f-b6bd9bbee54b)

Redshift table load :

![image](https://github.com/user-attachments/assets/5297f36b-5d84-4de9-af45-6b4a2b7f89fe)

![image](https://github.com/user-attachments/assets/3756bc68-540c-4c68-9c59-2075b814c30c)


Note - S3 VPC endpoint is needed to be created to use this S3 directory for Glue

Now run the Glue job and check the destination table contents in redshift:


# Eventbridge

Event Pattern:
![image](https://github.com/user-attachments/assets/cc06ddc9-9f9d-4f69-9c5c-2cf3cf1bd538)

Eventbridge rule will look like this:

![image](https://github.com/user-attachments/assets/23ecb529-7bdd-43ef-9fe8-6a6e3b52e9c8)
![image](https://github.com/user-attachments/assets/f3da7783-541e-46d1-8137-3ada6cf89047)


