# tsb

Name  Madhu Prasath Srinivasan

The following queries have been written in postgresql. I used db fiddle to create table, insert given data and to execute the SQL .
1. Find the ID of any Conditions which grant a right to a Tracker. (For clarity, this includes Initial Discount Tracker conditions.)
The table “Contractual_Condition” has the condition id and the details pertaining to a tracker.
 So, “like” operator can be used to find all the conditions which have the word tracker in it.

 

The ‘like’ operator is case-sensitive so as of now we are not aware whether the word ‘Tracker’ would always have uppercase ‘T’ or something else so the column attribute is converted to lowercase which enables the query to give intended results no matter how tracker is stored in the database.

2. Find any account(s) with any Condition which grants a right to a Tracker. (This includes Initial Discount Tracker conditions.)
The conditions and the account number are in different tables so join both of the tables using a common joining attribute which in this case is condition id.
I have used inner join because we just need the accounts with tracker condition so if there are accounts in the account condition relation table which doesn’t have a condition id in the contractual condition table it will be excluded implicitly.
 

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

3. How many accounts ever switched to a Tracker rate type (whether or not they had any right to a Tracker)?
From looking at the data in the product switch table we can infer that if one account switches to a tracker more than once the same account will be recorded more than once.
The following SQL gives all the accounts which have ever switched to a tracker type,
 

 
As you can see Account number 789 is recorded twice as the account had switched twice. So in order to exclude these repeated accounts we will be using distinct to find the unique number of accounts which have ever switched to a tracker.

SELECT *
FROM
TSB.PRODUCT_SWITCH
where "Post_switch_prod_id" in 
(SELECT "Prod_id" FROM tsb.Product where Product."Rate_type" = 'Tracker');

 
 
SELECT count(distinct PRODUCT_SWITCH."Acc_no")
FROM
TSB.PRODUCT_SWITCH
where "Post_switch_prod_id" in 
(SELECT "Prod_id" FROM tsb.Product where Product."Rate_type" = 'Tracker');

4. Which accounts had a right to a Tracker (including Initial Discount Tracker conditions), but never switched to a Tracker?
This can be found out either through sub-queries or joins.
Below describes solution using sub-queries.
 


 








