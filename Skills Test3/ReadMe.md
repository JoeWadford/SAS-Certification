/* skills test 3 */

%let path=~/ECRB94/data;
%let outpath=~/ECRB94/output;
libname cr "&path";

/* p104 */

proc import datafile="&path/payroll.csv" dbms=csv out=work.payroll;
	guessingrows=max;
run;

proc contents data=work.payroll nodetails;
run;

/* p105 */


options validvarname=V7;
libname xl xlsx "&path/employee.xlsx";
proc contents data=xl._all_;
run;

/* p106 */

proc sort data=sashelp.baseball out=baseball_Sort;
	by Team Name;
run;

proc print data=baseball_Sort;
	where Team in ("San Francisco", "Los Angeles", "Oakland");
	var Name Team Salary Cr:;
run;

proc freq data=baseball_Sort order=freq;
	tables Position;
run;

proc means data=baseball_Sort sum mean maxdec=0;
	var Salary;
run;

/* p107 */

proc freq data=cr.employee_raw order=freq;
	table EmpID;
run;


proc freq data=cr.employee_raw order=freq;
	table Country;
run;

proc print data=cr.employee_raw;
	where TermDate is not missing and TermDate<HireDate;
run;

/* p108 */

proc sort data=cr.employee_raw out=emp_sort nodupkey;
	by _all_;
run;


proc print data=emp_sort;
	where JobTitle contains 'Logistics';
	format HireDate Year.;
run;

proc means data=emp_sort mean;
	where HireDate >= "01JAN2010"d and TermDate is missing;
	var Salary;
run;

proc univariate data=emp_sort;
	var Salary;
run;

/* p109 */

data holiday2019;
	set sashelp.holiday;
	where end is missing and rule=0;
	CountryCode=substr(Category,4,2);
	Date=mdy(month, day, 2019);
	keep Desc CountryCode Date;
run;

/* 2.02 */

data sales;
	set cr.employee;
	length SalesLevel $ 6;
	where Department='Sales' and TermDate is missing;
	if JobTitle='Sales Rep. I' then SalesLevel='Entry';
	if JobTitle='Sales Rep. II' or JobTitle='Sales Rep. III' then SalesLevel='Middle';
	if JobTitle='Sales Rep. IV' then SalesLevel='Senior';
run;

proc freq data=sales order=freq;
	table SalesLevel;
run;

/* 203 */

data bonus;
	set cr.employee;
	where TermDate is missing;
	YearsEmp=YRDIF(HireDate,"01JAN2019"d, 'age');
	if YearsEmp>=10 then do;
		Bonus=.03*Salary;
		Vacation=20;
	end;
	
	else do;
		Bonus=.02*Salary;
		Vacation=15;
	end;
run;

proc sort data=bonus;
	by descending YearsEmp;
run;

proc freq data=bonus;
	table Vacation;
run;

/* p204 */

proc sort data=sashelp.baseball out=base_sort;
	by Team Name;
run;

title "Baseball Team Statistics";
proc print data=base_sort label noobs;
	by Team;
	var Position YrMajor nAtBat nHits nHome;
	sum nAtBat nHits nHome;
	ID Name;
	label Name="Player's Name"
			Position='Position(s) in 1986'
			YrMajor="Years in the Majors in 1986"
			nAtBat="Times at Bat in 1986"
			nHits='Hits in 1986'
			nHome'Home Runs in 1986';
run;

/* 205 */

proc freq data=cr.employee order=freq nlevels;
	table City Department JobTitle;
run;

/* 207 */

proc means data=cr.employee sum mean max min maxdec=0;
	where Department='Sales';
	var Salary;
	class JobTitle;
	ways 0 1;
run;