# snowflakes
snowflakes

create database if not exists mydb;
create schema if not exists enternal_stages;

CREATE STAGE IF NOT EXISTS ext_stage
URL = 's3://test0459/PUBLIC.LOAN_PAYMENT.xlsx'  -- or whatever external location you're using
CREDENTIALS = (AWS_KEY_ID = 'AKIAZI2LEBUAPU6WTVXC' AWS_SECRET_KEY = 'ZuPvCDXHAvN0YrjC7vketZ868b7wqInyyKHhEqPD');

desc stage ext_stage;

// Create storage integration object
create or replace storage integration s3_int
  TYPE = EXTERNAL_STAGE
  STORAGE_PROVIDER = S3
  ENABLED = TRUE 
  STORAGE_AWS_ROLE_ARN = 'arn:aws:iam::637423324416:role/s3-to-snowflake'
  STORAGE_ALLOWED_LOCATIONS = ('s3://test0459/PUBLIC.LOAN_PAYMENT.xlsx')
  COMMENT = 'Integration with aws s3 buckets' ;
//To get the snowflake arn deatils while intereating with aws s3.
desc integration s3_int;

// Create file format object
CREATE or replace file format  enternal_stages.csv_fileformat
    type = csv
    field_delimiter = '|'
    skip_header = 1
    empty_field_as_null = TRUE;   

    // Create stage object with integration object & file format object
CREATE OR REPLACE STAGE enternal_stages.csv_fileformat
    URL = 's3://test0459/PUBLIC.LOAN_PAYMENT.xlsx'
    STORAGE_INTEGRATION = s3_int
    FILE_FORMAT = enternal_stages.csv_fileformat

    list @enternal_stages.csv_fileformat;

    
//Creating the table
CREATE OR REPLACE TABLE Mydb.PUBLIC.LOAN_PAYMENT (
  "Loan_ID" STRING,
  "loan_status" STRING,
  "Principal" STRING,
  "terms" STRING,
  "effective_date" STRING,
  "due_date" STRING,
  "paid_off_time" STRING,
  "past_due_days" STRING,
  "age" STRING,
  "education" STRING,
  "Gender" STRING
 );

 
// Use Copy command to load the files
COPY INTO  Mydb.PUBLIC.LOAN_PAYMENT
    FROM @enternal_stages.csv_fileformat
    PATTERN = '.*LOAN.*';   
    continue;

    //Validate the data
SELECT * FROM Mydb.PUBLIC.LOAN_PAYMENT
