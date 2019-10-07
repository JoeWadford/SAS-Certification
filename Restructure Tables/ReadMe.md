/* RESTRUCTURING TABLES */

%let path=~/ECRB94/data;
%let outpath=~/ECRB94/output;
libname cr "&path";

/* using the DATA step to restructure tables */

data sales_n;
	set cr.qtr_sales;
	Qtr="Qtr1";
	Sales=Qtr1;
	output;
	Qtr="Qtr2";
	Sales=Qtr2;
	output;
	Qtr="Qtr3";
	Sales=Qtr3;
	output;
	Qtr="Qtr4";
	Sales=Qtr4;
	output;
	keep Qtr Sales;
run;
	
proc means data=sales_n;
	var Sales;
	class Qtr;
	ways 0 1;
run;

data sales_w;
	set cr.sales;
	by Customer_ID;
	retain Qtr1 Qtr2 Qtr3 Qtr4;
	if Qtr="Qtr1" then Qtr1=Sales;
	else if Qtr="Qtr2" then Qtr2=Sales;
	else if Qtr="Qtr3" then Qtr3=Sales;
	else if Qtr="Qtr4" then Qtr4=Sales;
	if last.Customer_ID;
	drop Qtr Sales;
run;

/* using proc transpose to restructure */

/* wide to narrow */
proc transpose data=cr.qtr_sales out=sales_n(rename=(col1=Sales) drop=_label_) name=Qtr;
	var Qtr:;
	by Customer_ID Name;
run;

/* narrow to wide */

proc transpose data=cr.sales out=sale_w(drop=_name_);
	var Sales;
	by Customer_ID Name;
	id Qtr;
run;

/* Analyzing Product by Region and Subsidiary */

proc sort data=sashelp.shoes out=shoes_sort nodupkey;
	by Region Subsidiary Product;
run;

proc transpose data=shoes_sort out=shoes_sales(drop=_name_ _label_);
	var Sales;
	by Region Subsidiary;
	ID product;
run;

/* training per employee */

proc sort data=cr.employee_training out=work.empSort;
	by Name;
run;

proc transpose data=work.empSort out=training_narrow(rename=(col1=Date_Completed)) name=Course;
	format Compliance_Training Corporate_Security On_the_Job_Safety monname.;
	var Compliance_Training Corporate_Security On_the_Job_Safety;
	by Name;
run;

proc freq data=training_narrow order=freq;
	table Date_Completed Course;
run;
