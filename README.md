## McDonalds Menu Analysis

## TABLE OD CONTENT
- [PROJECT OVERVIEW](project-overview)
- [DATA SOURCE](data-source)
- [TOOLS](tools)
- [DATA CLEANING](data-cleaning)
- [EXPLORATORY DATA ANALYSIS](exploratory-data-analysis)
- [ANALYSIS](analysis)
- [INSIGHTS](insights)
- [RECOMMENDATION](recommendation)


## PROJECT OVERVIEW
This dataset provides a nutrition analysis of every menu item on the US McDonald’s menu. The goal is to perform an exploratory data analysis (EDA) to get insights that can help McDonald make informed decisions on their menu item. 

## DATA SOURCE
The dataset used in this project was provided by the company Oasis Infobyte.

## TOOLS
1.	EXCEL: The dataset was gotten as a CSV, so I used Excel to open it. Go through the dataset to know the information it contains as well as cleaning.
2.	SQL: The dataset was loaded to SQL for cleaning and querying.
3.	Power BI: This tool was used for visualization.

## DATA CLEANING
1.	Confirming the total normal of dataset and columns.
2.	Making sure they all represent their various data type.
3.	Checking for duplicates and null values.

## EXPLORATORY DATA ANALYSIS
1.	Total calories of all the item and their respective categories.
2.	Total number of both the categories in the menu as well as the items within each respective category.
3.	How does the nutritional value affect each item?
4.	How does the nutritional density of each item affect the menu?

## ANALYSIS
```SQL
SELECT * FROM menu.menuu;


-- data cleaning
-- total item
select
count(*) 
from menuu;

-- total colum
select count(column_name)
from information_schema.columns
where table_name = 'menuu';

-- data type
select column_name, data_type
from information_schema.columns
where table_name = 'menuu';

-- duplicates and null values
with duplicate_cte as
(
select 
row_number() over 
(partition by category, item, serving_size, calories, calories_from_fat, 
Total_Fat, Total_Fat_Daily_Value_pct,Saturated_Fat,Saturated_Fat_Daily_Value_pct,
Trans_Fat,Cholesterol,Cholesterol_Daily_Value_pct,Sodium,Sodium_Daily_Value_pct,
Carbohydrates,Carbohydrates_Daily_Value_pct,Dietary_Fiber,Dietary_Fiber_Daily_Value_pct,
Sugars,Protein, VitaminA_Daily_Value_pct,VitaminC_Daily_Value_pct,Calcium_Daily_Value_pct,Iron_Daily_Value_pct) as row_num
from menuu
)

select *
from duplicate_cte
where row_num > 1
and row_num is null;

-- EDA
select *
from menuu;


-- average calories
select round(avg(calories), 0) as avg_calories
from menuu;

-- total item via category
select category, count(*) as total_item
from menuu
group by 1
order by 2 desc;

-- max calories in each category
select category, max(calories) as max_calories
from menuu
group by 1
order by 2 desc;


-- create a nutritional score for each item with the 2000 colorie rule
select round((total_fat_daily_value_pct) + (saturated_fat_daily_value_pct) + (cholesterol_daily_value_pct) + 
(sodium_daily_value_pct) + (Carbohydrates_Daily_Value_pct) + (dietary_fiber_daily_value_pct) + 
(VitaminA_Daily_Value_pct) + (VitaminC_Daily_Value_pct) + (Calcium_Daily_Value_pct) + (Iron_Daily_Value_pct) + 
((calories / 2000)* 100 + (sugars / 2000)* 100 + (Protein / 2000)* 100), 1) as Nutritional_score
from menuu;

alter table menuu
add column Nutritional_score int;

update menuu
set nutritional_score =  round((total_fat_daily_value_pct) + (saturated_fat_daily_value_pct) + (cholesterol_daily_value_pct) + 
(sodium_daily_value_pct) + (Carbohydrates_Daily_Value_pct) + (dietary_fiber_daily_value_pct) + 
(VitaminA_Daily_Value_pct) + (VitaminC_Daily_Value_pct) + (Calcium_Daily_Value_pct) + (Iron_Daily_Value_pct) + 
((calories / 2000)* 100 + (sugars / 2000)* 100 + (Protein / 2000)* 100), 1);


-- create a nutritional density (allows you know amount of nutrient per item serving size)
select round((nutritional_score / Serving_Size ), 1) as Nutritional_density
from menuu;

alter table menuu
add column Nutritional_density int;

update menuu
set  Nutritional_density = round((nutritional_score / Serving_Size ), 1) ;

select category,  max(Nutritional_density), min(Nutritional_density) 
from menuu
group by 1;

select 
case 
	when nutritional_density > 71 then 'Extremely Nutrient Dense'
	when nutritional_density between 51 and 70 then 'High Nutrient Dense'
	when nutritional_density between 31 and 50 then 'Moderately Nutrient Dense'
	when nutritional_density <=30 then 'Low Nutrient Dense'
end as Nutrient_Density_Grade
from menuu;

alter table menuu
add column Nutrient_Density_Grade varchar(100);

update menuu
set Nutrient_Density_Grade = 
case 
	when nutritional_density > 71 then 'Extremely Nutrient Dense'
	when nutritional_density between 51 and 70 then 'High Nutrient Dense'
	when nutritional_density between 31 and 50 then 'Moderately Nutrient Dense'
	when nutritional_density <=30 then 'Low Nutrient Dense'
end;

-- total count of nutrient density grade on category
select Nutrient_density_grade, count(*) as total_count
from menuu
group by 1;
```

## INSIGHTS
1.	The category with the highest calories is Coffee & Tea with 27k calories while the category with the least calories is Salads as well as Dessert.
2.	The serving size of the items plays a crucial role in determining both it’s nutritional score and nutritional density. 
3.	The nutritional density compares different meals and their nutritional value per serving size.
4.	The ‘Low Nutrient Dense’ grade has the highest frequency, predominantly found in the Coffee & Tea category, as well as the Smoothies & Shakes category.

## RECOMMENDATION
A higher nutritional score is generally desirable, indicating a meal provides sufficient essential nutrients. However, for sustainable generally consumption, I recommend aiming for a score close to the recommended daily value of 2000 calorie, considering factors like age, sex, eating habit, occupation. 

