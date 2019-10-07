/* ANALYZING DATA */

%let path=~/ECRB94/data;
%let outpath=~/ECRB94/output;
libname cr "&path";

proc sql;
	create table profit_country as
		select profit.*, Country_Name
		from profit inner join country_clean
		on profit.Customer_Country=country_clean.Country_Key
		order by Order_Date desc;
quit;
/* this pulls tables from week 1 and week 2
Basic Review and Preparing Data */

/* Analysis by Order Source */
ods noproctitle;
title "Number of Orders by Month";
title2 "and Customer Continent/Order Source";
proc freq data=profit_country order=freq;
	tables Order_Date / nocum;
	format Order_Date monname.;
	tables Customer_Continent*Order_Source / norow nocol;
run;

/* Ship Days Summary */

proc sort data=profit_Country out=profit_sort;
	by Order_Source;
run;

%let os=Retail;
title 'Days to Ship by Country';
proc means data=profit_sort min max mean;
	var ShipDays;
	class Country_Name;
	where ShipDays>0;
	by Order_Source;
run;
/* This would be a good report to export to excel for each Order Source */

/* Profit by Customer Age */

proc means data=profit_country noprint;
	var profit;
	class Age_Range;
	output out=profit_summary median=MedProfit sum=TotalProfit;
	ways 1;
/* ways specifies that you only want statistics for each unique value of the class column */
run;

proc print data=profit_summary noobs label;
	var Age_Range TotalProfit MedProfit;
	label 	Age_Range="Age Range"
			TotalProfit="Total Profit"
			MedProfit="Median Profit per Order";
	format TotalProfit MedProfit dollar10.;
run;
/* review details about freq, means, print */

/* Analysis Employment by City Dep and Jobtitle */
proc sort data=cr.employee out=employee_sort;
	by City Department JobTitle;
run;

proc freq data=employee_sort order=freq nlevels;
	tables City Department JobTitle;
run;

/* Analyze Order Date by Order Source */

proc freq data=cr.profit;
	tables Order_Date*Order_Source / norow nocol;
	format Order_Date monname.;
run;

proc sort data=cr.employee out=employee_sort;
	by JobTitle;
run;

proc means data=employee_sort sum mean min max maxdec=0;
	where Department="Sales";
	var Salary;
	Class JobTitle;
	ways 0 1;
run;

proc means data=cr.employee;
	Var Salary;
	Class Department City;
	ways 2;
	output out=salary_summary Sum=TotalSalary Mean=AvgSalary;
run;
	