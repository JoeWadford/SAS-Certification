/* REVIEW OF BASIC SYNTAX and INTRO TO SAS */

/* p101_q1 */

/* ACCESSING DATA */

*libnames can be 8 characters or less
*when reading macros, you must use double quotation marks ""
*to enforce standard SAS naming recommendations use validvarname=v7

*suppress descriptor portions of proc contents = nods;

%let path=~/ECRB94/data;
%let outpath=~/ECRB94/output;
libname cr "&path";
libname ctryxl xlsx "&path/country_lookup.xlsx";

proc contents data=cr._all_ nods;
run;

proc import dbms=csv datafile="&path/payroll.csv" out=payroll replace;
guessingrows=max;
run;

proc contents data=payroll;
run;

options validvarname=v7;
libname xl xlsx "&path/employee.xlsx";
proc contents data=xl._all_;
run;

/* EXPLORING DATA */

/* Validate Country Lookup Excel Table */

proc print data=ctryxl.countries(obs=30);
run;

proc freq data=ctryxl.countries order=freq;
	tables Country_Key Country_Name;
run;

proc print data=ctryxl.countries;
where Country_Key in ('AG','CF','GB','US');
run;

proc sort data=ctryxl.countries out=country_clean nodupkey dupout=dups;
	by Country_Key;
run;

/* Validate Imported Orders Table */

proc print data=cr.orders;
	where Order_date>Delivery_date;
	var Order_ID Order_Date Delivery_Date;
run;

proc freq data=cr.orders;
	table Order_Type Customer_Country Customer_Continent;
run;

proc univariate data=cr.orders;
	var quantity retail_Price Cost_Price;
run;

/* Employee Raw Data Analysis Practice */

%let path=~/ECRB94/data;
%let outpath=~/ECRB94/output;
libname cr "&path";

proc freq data=cr.employee_raw order=freq;
	tables EmpID Country Department;
run;

proc print data=cr.employee_raw;
	where TermDate ne . and TermDate<HireDate;
run;

proc sort data=cr.employee_raw out=emp_sort nodupkey;
	by _all_;
	format TermDate HireDate ddmmyy10.;
run;

proc print data=emp_sort;
	where jobtitle like '%Logistics%';
run;

proc means data=emp_sort; 
	WHERE TermDate = . and HireDate>="01Jan2010"d;
	VAR Salary HireDate;
run;

proc univariate data=emp_sort;
	var salary HireDate;
run;