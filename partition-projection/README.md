# Testing Athena Partition projection

1. Create a S3 bucket

2. Copy sample.csv files to the bucket. Leave ./csvTest/2022/07/03/sample.csv which will be added later 

```
aws s3 cp ./csvTest/2022/07/01/sample.csv s3://athena-workshop-394254462122/csvTest/2022/07/01/sample.csv
aws s3 cp ./csvTest/2022/07/02/sample.csv s3://athena-workshop-394254462122/csvTest/2022/07/02/sample.csv
```

3. On AWS Console, go to Athena query editor and run below query which creats a virtual table with partition projection enabled.

```
CREATE EXTERNAL TABLE IF NOT EXISTS default.mycsv_partition (
  `time` STRING,
  `user_id` STRING,
  `board_name` STRING,
  `action` STRING
) 
PARTITIONED BY (year int, month int, day int)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
ESCAPED BY '\\'
LINES TERMINATED BY '\n'
LOCATION 's3://athena-workshop-394254462122/csvTest/'
TBLPROPERTIES (
'skip.header.line.count'='1',
'projection.enabled'='true',
'projection.day.digits'='2',
'projection.day.range'='01,31',
'projection.day.type'='integer',
'projection.month.digits'='2',
'projection.month.range'='01,12',
'projection.month.type'='integer',
'projection.year.digits'='4',
'projection.year.range'='2020,2022', 
'projection.year.type'='integer', 
"storage.location.template" = "s3://athena-workshop-394254462122/csvTest/${year}/${month}/${day}"
);

SELECT * FROM mycsv_partition
where user_id='bob';
```

4. Query on the new table.

```
SELECT * FROM mycsv_partition
where user_id='bob';

#	time	user_id	board_name	action	year	month	day
3	2022-07-02 10:06	bob	game	insert	2022	7	2
4	2022-07-02 14:06	bob	game	insert	2022	7	2
5	2022-07-01 10:06	bob	game	insert	2022	7	1
6	2022-07-01 14:06	bob	game	insert	2022	7	1


```

5. Upload the other sample.csv file to create new partition on S3

```
aws s3 cp ./csvTest/2022/07/03/sample.csv s3://athena-workshop-394254462122/csvTest/2022/07/03/sample.csv
```

6. Query on the table again and check if data from new partition is added

```
SELECT * FROM mycsv_partition
where user_id='bob';

#	time	user_id	board_name	action	year	month	day
1	2022-07-03 22:06	bob	game	insert	2022	7	3
2	2022-03-05 22:06	bob	game	insert	2022	7	3
3	2022-07-02 10:06	bob	game	insert	2022	7	2
4	2022-07-02 14:06	bob	game	insert	2022	7	2
5	2022-07-01 10:06	bob	game	insert	2022	7	1
6	2022-07-01 14:06	bob	game	insert	2022	7	1
```







