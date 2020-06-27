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
Create table events (event_id numeric, device_id numeric, timestamps varchar, longitude numeric, laititude numeric); 

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





