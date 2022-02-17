# CIE DATA ANALYST ASSESSMENT 

## Name:  Madhu Prasath Srinivasan

The following queries have been written in postgresql (version 13). I used db [fiddle](https://www.db-fiddle.com/) to create table, insert given data and to execute the SQL . 

Please find the schema below.
```
CREATE SCHEMA tsb;
SET search_path = tsb;

CREATE TABLE Contractual_Condition (
  "Cond_ID" int,
  "Attribute" varchar(70)
  
);

INSERT INTO Contractual_Condition
  ("Cond_ID", "Attribute")
VALUES
  (1, 'Right to a Tracker'),
  (2, 'Right to a Variable'),
  (3, 'Initial Discount Tracker rate: ECB+ 1% for first year'),
  (4, 'Initial Discount Tracker rate: ECB+ 1% for first 2 years'),
  (5, 'Initial Fixed Interest Rate of 3% for 2 years');
  
  
  CREATE TABLE Acc_cond_rel
  ("Acc_no" INT ,
   "Cond_id" INT);
   
   INSERT INTO Acc_cond_rel
   ("Acc_no","Cond_id")
   values
    (123,1),
	(123,5),
	(456,1),
	(456,3),
	(789,2),
	(789,4),
	(321,5);
    
   CREATE TABLE Product_Switch
 ("Acc_no" SMALLINT,
  "Acc_open_date"  DATE,
  "Switch_date" DATE,
  "Pre_switch_prod_id" SMALLINT,
  "Post_switch_prod_id" SMALLINT);
  
  INSERT INTO Product_Switch 
  ("Acc_no", "Acc_open_date", "Switch_date", "Pre_switch_prod_id", "Post_switch_prod_id")
VALUES
(456,'01/05/2006','01/05/2007',1,4),
(789,'01/06/2006','01/06/2008',2,3),
(321,'01/07/2006','01/07/2008',4,6),
(123,'01/08/2006','01/08/2008',4,3),
(789,'01/06/2006','01/06/2009',3,1);


CREATE TABLE Product (
  "Prod_id" SMALLINT,
  "Rate_type" VARCHAR(20),
  "Fixed_Rate" VARCHAR(5),
  "Tracker_Margin" VARCHAR(5),
  "Term_years" SMALLINT
);

INSERT INTO Product
  ("Prod_id", "Rate_type", "Fixed_Rate", "Tracker_Margin", "Term_years")
VALUES
(1,'Tracker',NULL,'1%',	1),

(2,'Tracker',NULL,'1%',2),

(3,'Tracker',NULL,'2%',NULL),

(4,'Fixed','3%',NULL,2),

(5,	'Fixed','4%',NULL,3),

(6,	'Variable',NULL,NULL,NULL);


```


___1. Find the ID of any Conditions which grant a right to a Tracker. (For clarity, this includes Initial Discount Tracker conditions.)
The table “Contractual_Condition” has the condition id and the details pertaining to a tracker.
 So, “like” operator can be used to find all the conditions which have the word tracker in it.___

```
SELECT * FROM tsb.Contractual_Condition where lower("Attribute") like '%track%'
```


![image](https://user-images.githubusercontent.com/78327987/154588262-a636a88a-f55d-434a-8153-722038e2f36c.png)

 ![image](https://user-images.githubusercontent.com/78327987/154590105-3f1d872a-437e-4d4a-aa63-ebd5a7f306a3.png)


The ‘like’ operator is case-sensitive so as of now we are not aware whether the word ‘Tracker’ would always have uppercase ‘T’ or something else so the column attribute is converted to lowercase which enables the query to give intended results no matter how tracker is stored in the database.

___2. Find any account(s) with any Condition which grants a right to a Tracker. (This includes Initial Discount Tracker conditions.)___  

The conditions and the account number are in different tables so join both of the tables using a common joining attribute which in this case is condition id.  
I have used inner join because we just need the accounts with tracker condition so if there are accounts in the account condition relation table which doesn’t have a condition id in the contractual condition table it will be excluded implicitly
 
```
/*Joining Contractual condition table and Account Relation table to get the accounts with a tracker*/
SELECT	
Acnt_Cond_Realtion."Cond_id",
Acnt_Cond_Realtion."Acc_no"
FROM 
tsb.Contractual_Condition 
inner join
tsb.Acc_cond_rel as Acnt_Cond_Realtion
on Contractual_Condition."Cond_ID" = Acnt_Cond_Realtion."Cond_id"
where lower(Contractual_Condition."Attribute") like '%tracker%';
```
![image](https://user-images.githubusercontent.com/78327987/154589017-a752c077-5c8b-4fec-95f8-8e418dc2f7eb.png)

As we can see there are duplicate accounts in order to get only the unique accounts we can use distinct but the above results helps us to know the account number 456 had different trackers in a given point of time.  

__3. How many accounts ever switched to a Tracker rate type (whether or not they had any right to a Tracker)?__  

From looking at the data in the product switch table we can infer that if one account switches to a tracker more than once the same account will be recorded more than once. Example Account number 789.

The following SQL gives all the accounts which have ever switched to a tracker type,  

```
SELECT *
FROM
TSB.PRODUCT_SWITCH
where "Post_switch_prod_id" in 
(SELECT "Prod_id" FROM tsb.Product where Product."Rate_type" = 'Tracker');
```
![image](https://user-images.githubusercontent.com/78327987/154589488-d7540cad-8d6f-4555-ab72-a8d8e4002e0f.png)

As you can see Account number 789 is recorded twice as the account had switched twice. So in order to exclude these repeated accounts we will be using distinct to find the unique number of accounts which have ever switched to a tracker.
```
SELECT count(distinct PRODUCT_SWITCH."Acc_no")
FROM
TSB.PRODUCT_SWITCH
where "Post_switch_prod_id" in 
(SELECT "Prod_id" FROM tsb.Product where Product."Rate_type" = 'Tracker');
```
![image](https://user-images.githubusercontent.com/78327987/154589470-209da11c-c0c1-4c00-a9b2-75eefa0eb931.png)

___4. Which accounts had a right to a Tracker (including Initial Discount Tracker conditions), but never switched to a Tracker?__  

This can be found out either through sub-queries or joins.  
Below describes solution using sub-queries.
```
SELECT *
FROM
TSB.PRODUCT_SWITCH
where "Post_switch_prod_id" not in 
(select "Prod_id" from tsb.Product where Product."Rate_type"='Tracker')
and "Pre_switch_prod_id" in 
( Select "Prod_id" from tsb.Product Where  Product."Rate_type"='Tracker')
```
![image](https://user-images.githubusercontent.com/78327987/154589967-6fb4fccf-ee19-4df5-be0d-943676a56075.png)





