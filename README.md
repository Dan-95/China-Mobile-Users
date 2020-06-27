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
Create Table labelcategories(label_id varchar, category varchar);
create table applabels (app_id varchar, label_id varchar);
create table appevents (event_id varchar, app_id varchar , is_installed varchar, is_active varchar);
create table phonebrand (device_id varchar, phone_brand varchar, device_model varchar);
create table trainingset (device_id varchar, gender char, age integer, age_group varchar);
create table events (event_id varchar, device_id varchar, timestamps varchar, longitude numeric, laititude numeric); 

```

