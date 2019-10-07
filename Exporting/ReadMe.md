/* EXPORTING DATA */
/* uses code and tables from week 1, Basic Review, Preparing Data, and Analyzing Data*/ 

%let path=~/ECRB94/data;
%let outpath=~/ECRB94/output;
libname cr "&path";

proc print data=profit_summary label noobs;
	var Age_Range TotalProfit MedProfit;
	label Age_Range="Age Range"
			TotalProfit="Total Profit"
			MedProfit="Median Profit per Order";
	format TotalProfit MedProfit dollar10.;
run;

/* Using Proc Export to export excel files */

proc export data=profit_Country outfile="&outpath/orders_update.csv" dbms=csv replace;
run;

/* using the library to export excel files */

libname outxl xlsx "&outpath/orders_update.xlsx";

data outxl.Orders_Update;
	set profit_country;
run;

data outxl.Country_Lookup;
	set country_clean;
run;

proc means data=profit noprint;
	var profit;
	class Age_Range;
	ways 1;
	output out=outxl.profit_summary;
run;

libname outxl clear;

/* Output Delivery System */
/* exporting PDF and PDF options */

ods pdf /* can specify html, rtf (word), excel, etc */ file="&outpath/orders_validation.pdf" pdftoc=1;
/* pdftoc=1 displays only the first level of bookmarks by default*/

ods proclabel "Orders with Order Date After Delivery Date";
title "Orders with Order Date After Delivery Date";
proc print data=cr.orders;
	where Order_Date>Delivery_Date;
	var Order_ID Order_Date Delivery_Date;
run;

ods proclabel "Examine Values of Numeric Columns in Orders";
title "Examine Values of Numeric Columns in Orders";
proc freq data=cr.orders;
	tables Order_Type Customer_Country Customer_Continent;
run;

ods proclabel "Examine Values of Categorical Columns in Orders";
title "Examine Values of Categorical Columns in Orders";
proc means data=cr.orders maxdec=0;
	var Quantity Retail_Price Cost_Price;
run;

ods pdf close;

/* Excel Options */

ods excel file="&outpath/Orders_Analysis.xlsx" 
	options(embedded_titles="yes") style=analysis;

ods excel options(sheet_name="Orders by Source, Continent");
title "Orders by Month";
proc freq data=profit_country order=freq;
	tables Order_Date / nocum;
	format Order_Date monname.;
run;

ods excel options(sheet_name="Orders by Month");
title "Orders by Customer Continent/Order Source";
proc freq data=profit_country order=freq;
	tables Customer_Continent*Order_Source / norow nocol;
run;

proc means data=profit noprint;
	var profit;
	class Age_Range;
	output out=profit_summary;
run;

proc means data=profit median sum noprint;
	var profit;
	class Age_Range;
	output out=profit_summary median=MedProfit sum=TotalProfit;
	ways 1;
run;

ods excel options(sheet_name="Profit by Age");
title "Profit by Customer Age Range";
proc print data=profit_summary noobs label;
	var Age_Range TotalProfit MedProfit;
	label 	Age_Range="Age Range"
			TotalProfit="Total Profit"
			MedProfit="Median Profit per Order";
	format TotalProfit MedProfit dollar10.;
run;

ods excel close;

proc export data=cr.employee_current outfile="&outpath/employee_current.csv" dbms=csv;
run;

/* Review ODS */ 