/* PREPARING DATA */

%let path=~/ECRB94/data;
%let outpath=~/ECRB94/output;
libname cr "&path";

data profit;
	set cr.orders;
	length Order_Source $ 8;
	where Delivery_Date>=Order_Date;
	Customer_Country=upcase(Customer_Country);
	if Quantity < 0 then Quantity=.;
	Profit=(Retail_Price-Cost_Price)*Quantity;
	format Profit dollar12.2;
	ShipDays=Delivery_Date-Order_Date;
	Age_Range=substr(Customer_Age_Group,1,5);
	If Order_Type=1 then Order_Source='Retail';
	else if Order_Type=2 then Order_Source='Phone';
	else if Order_Type=3 then Order_Source='Internet';
	else Order_Source='Unknown';
	drop Retail_Price Cost_Price Customer_Age_Group Order_Type;
run;

/* Review Other Preparation Functions 
Manipulating Data with Functions from Doing More with SAS 

Also review data step processing (compilation and execution)
*/

proc sql;
	create table profit_country as
		select profit.*, Country_Name
			from profit inner join country_clean
			on profit.Customer_Country=country_clean.Country_Key
/* country_clean was created in week 1 */
			order by Order_Date desc;
quit;

/* p104q1 */

data sales;
	set cr.employee;
	where Department ='Sales' and TermDate=.;
	Length SalesLevel $ 6;
	if JobTitle='Sales Rep. I' then SalesLevel='Entry';
	else if JobTitle in('Sales Rep. II', 'Sales Rep. III') then SalesLevel='Middle';
	else if JobTitle='Sales Rep. IV' then SalesLevel='Senior';
run;

proc freq data=sales;
	tables SalesLevel;
quit;

data bonus;
	set cr.employee;
	where TermDate is missing; 
	YearsEmp=yrdif(HireDate,"01Jan2019"d,'age');
	if YearsEmp>=10 then 
		do;
			Bonus=Salary*.03;
			Vacation=20;
		end;
	else do; 
			Bonus=Salary*.02;
			Vacation=15;
		end;
run;

proc freq data=bonus;
	tables Vacation;
quit;


proc sort data=bonus;
	by descending YearsEmp;
quit;
	