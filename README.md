# China-Mobile-Users

**introduction** 

This project is performing an ad-hoc analysis of Mobile usage situation in China. The statistical program language utilized in this project is SQL (Platform: PostgreSQL).

**Data Source**

The datasets were downloaded from [Kaggle-China Mobile User](https://www.kaggle.com/chinapage/china-mobile-user-gemographics) and the original datasets are provided by Talkingdata ([About Talkingdata](http://www.talkingdata.com/))

The datasets contain 8 .csv files and the total data size is 1.18GB. They are:
* app_events.csv (989.22MB)
* app_labels.csv (10.67MB)
* events.csv (186.38MB)
* gender_age_test.csv (2.18MB)
* gender_age_train.csv (2.26MB)
* label_categories.csv (16.06MB)
* phone_brand_device_model.csv (6.4MB)
* sample_submission.csv (11.16MB)

More details about columns in each spreadsheets could be checked in [Kaggle-China Mobile User](https://www.kaggle.com/chinapage/china-mobile-user-gemographics)

**Abstract**
This project is going to:
* Review the data category in the datasets, find out the relationship between each spreadsheet.
* Create tables in the database, prepare to import raw data into the database.
* Review raw data in each table. Perform data cleaning and set constraints for each table.
* Utilize the polished data to do a ad-hoc analysis.
* Join the table and prepare for the further study. 

**Main**
Create tables in database for the importing purpose

```SQL
--- Create tables in database for the importing purpose ---
Create Table labelcategories(label_id numeric, category varchar);
Create table applabels (app_id numeric, label_id numeric);
Create table appevents (event_id numeric, app_id numeric , is_installed varchar, is_active varchar);
Create table phonebrand (device_id numeric, phone_brand varchar, device_model varchar);
Create table trainingset (device_id numeric, gender char, age integer, age_group varchar);
Create table events (event_id numeric, device_id numeric, timestamps varchar, longitude numeric, latitude numeric); 

```
Here is an execution successful screenshot:

![Image of result](https://github.com/Dan-95/China-Mobile-Users/blob/master/results/Create%20Table%20Example%20Result.png)

Then we import the data into the empty table:

```SQL
-------------Import data in the Database -----------------
copy labelcategories from 'D:/CSV/label_categories.csv' 
	WITH (FORMAT CSV, ENCODING 'UTF8', DELIMITER ',', QUOTE '"', NULL '', HEADER True);
copy applabels from 'D:/CSV/app_labels.csv'
	WITH (FORMAT CSV, ENCODING 'UTF8', DELIMITER ',', QUOTE '"', NULL '', HEADER True);
copy appevents from 'D:/CSV/app_events.csv'
	WITH (FORMAT CSV, ENCODING 'UTF8', DELIMITER ',', QUOTE '"', NULL '', HEADER True);
copy phonebrand from 'D:/CSV/phone_brand_device_model.csv'
	WITH (FORMAT CSV, ENCODING 'UTF8', DELIMITER ',', QUOTE '"', NULL '', HEADER True);
copy trainingset from 'D:/CSV/gender_age_train.csv'
	WITH (FORMAT CSV, ENCODING 'UTF8', DELIMITER ',', QUOTE '"', NULL '', HEADER True);
copy events from 'D:/CSV/events.csv'
	WITH (FORMAT CSV, ENCODING 'UTF8', DELIMITER ',', QUOTE '"', NULL '', HEADER True);

```
Here is an execution successful screenshot:

![Image of result](https://github.com/Dan-95/China-Mobile-Users/blob/master/results/Import%20Table%20Result.png)

After importing all raw data into the database, we'd like to build a relationship diagram between each tables:

![Image of ERD](https://github.com/Dan-95/China-Mobile-Users/blob/master/china_mobile_users%20EDR.png)

Based on this graphic, we need to ensure the unique value for the primary Key. Also, it's necessary to check are there any missing/abnormal/null value.

Also, considering that there are two many to many relationship among there enities, we must add a transitive table to divide them into one to many relationship

```SQL
Select label_id, count(*) from labelcategories group by label_id having count(*) > 1;
Select device_id, count(*) from phonebrand group by label_id having count(*) > 1;
Select even_id, count(*) from events group by event_id having count(*) > 1; 

```
According to the result, there are 529 duplicated device_id in Table phonbrand 

![Image of duplicated id](https://github.com/Dan-95/China-Mobile-Users/blob/master/results/Duplicated%20records%201.png)

We need to check duplicated records and decide whether we need to modify them or just delete them. By doing some inquiries, we found two types of duplicates:
* Values are same in all columns (523 records)
```SQL
Select device_id, phone_brand, device_model, count(*) from phonebrand
	Group by (device_id, phone_brand, device_model)
	having count(*) > 1
	order by device_id;
```
![Image_of_dulicated_id](https://github.com/Dan-95/China-Mobile-Users/blob/master/results/type%201%20duplicated.png)

* Values are same in device_id and phone_brand (6 records)
```SQL
Select * from 
(select dense_rank() over ( PARTITION by device_id order by(phone_brand,device_model) desc),* from phonebrand) as t1
where dense_rank > 1;
```
![Image_of_dulicated_id](https://github.com/Dan-95/China-Mobile-Users/blob/master/results/type%202%20duplicated.png)

Considering that in the real life, the device_id of the phone can only map one type of brand with the certain model. Therefore, we will remove all these duplicated records.
```SQL

Select * into storage1 from (Select * from 
(select dense_rank() over ( PARTITION by device_id order by(phone_brand,device_model) desc),* from phonebrand) as t1
where dense_rank = 1) as t2;

select t1.device_id, t1.phone_brand,t1.device_model into storage2 
	from (Select device_id, phone_brand, device_model, count(*) 
		  from storage1
		  Group by (device_id, phone_brand, device_model)
		  having count(*) = 1
		  order by device_id) as t1;

select * into newphonebrand 
	from (select * from storage2) as t1;
```
Then we got a clean phonebrand table:
![Image_of_table](https://github.com/Dan-95/China-Mobile-Users/blob/master/results/newphonebrand.png)

Then could set primary key for these three tables
```SQL

ALTER TABLE newphonebrand
  ADD CONSTRAINT device_id_pk 
    PRIMARY KEY (device_id);
	
ALTER TABLE events
  ADD CONSTRAINT event_id_pk 
    PRIMARY KEY (event_id);	
	
ALTER TABLE labelcategories
  ADD CONSTRAINT label_id_pk 
    PRIMARY KEY (label_id);

```
We Successfully set up Primary Keys:

![image_of_pk](https://github.com/Dan-95/China-Mobile-Users/blob/master/results/PK.png)

After doing these steps, Let's take a glance on some tables(Which will be used for joining):

* events
Check how many events happened in the same location, or at the same time.
```SQL
select longitude,latitude, count(*) from events
	group by (longitude,latitude)
	order by count(*) desc;
	
select timestamps, count(*) from events
	group by (timestamps)
	order by count(*) desc;
```
* trainingsets:
Check the demographic in the sample of trainingsets (Age, gender, etc.) 

```SQL
select gender,count(*) from trainingset
	group by gender;
	
select gender, round(avg(age),2) from trainingset
	group by gender;
	
select gender, age_group, count(*) from trainingset
	group by (gender, age_group)
	order by count(*) desc;
```

* newphonebrand:
Check the most popular brands, device_model

```SQL
select count (distinct(phone_brand)) from newphonebrand;

Select phone_brand, count(*) from newphonebrand
	group by phone_brand
	order by count(*) desc
	limit 10;

Select phone_brand, device_model, count(*) from newphonebrand
	group by phone_brand, device_model
	order by count(*) desc
	limit 10;
```
Start to prepare a new data spreadsheet (Join table)

``` SQL 
select device_id, gender, age, age_group, phone_brand, device_model into step1 from
(select t1.device_id, gender, age, age_group, phone_brand, device_model
	from trainingset t1
	left join newphonebrand t2
	on t1.device_id = t2.device_id
	where t1.device_id is not NULL) t3;

select device_id, gender, age, age_group, phone_brand, device_model,timestamps, longitude, latitude into finalchart
	from (select t1.device_id, gender, age, age_group, phone_brand, device_model,timestamps, longitude, latitude 
		from step1 t1
		left join events t2
		on t1.device_id = t2.device_id
		where t1.device_id is not NULL) t3;

```
Joined Success:

![image_of_chart](https://github.com/Dan-95/China-Mobile-Users/blob/master/results/final_chart.png)

After producing this new tables, we need to do some examinations and polish.

```SQL

select * from finalchart where timestamps is NULL;

Delete from finalchart where timestamps is null;

Select device_id, timestamps, gender,count(*) from finalchart
	group by(device_id, timestamps, gender)
	having (count(*) >= 2)
	order by (count(*))
	desc;

Select * into ChineseMobileUsers
	from (Select distinct * from finalchart) t1;
```
Duplicated and Null results:

![image of final dupliacted](https://github.com/Dan-95/China-Mobile-Users/blob/master/results/duplicated%20records%20final.png)


Now, we have already produced a polished data spreadsheet for further analysis.
