/* SKILLS TEST REV2 */

/* 1.01 */

data CanadaSales;
	set sashelp.prdsale;
	diff=Actual-Predict;
	where Country='CANADA' and Quarter=1;
run;

/* 1.02 */

/*This program analyzes blood pressure
in the SASHELP.HEART table.*/

proc freq data=sashelp.heart;
    table BP_Status;
run;

proc means data=sashelp.heart min max mean maxdec=0;
    var systolic diastolic;
    class BP_Status;
run;

/* 1.03 */

%let path=~/ECRB94/data;
%let outpath=~/ECRB94/output;
libname cr "&path";

proc contents data=cr._all_ nods;
run;

/* 1.04 */

proc import datafile="&path/payroll.csv" dbms=csv out=work.payroll;
	guessingrows=max;
run;

proc contents data=work.payroll;
run;

/* 1.05 */

option validvarname=V7;
libname xl xlsx "&path/employee.xlsx";
proc contents data=xl._all_;
run;

/* 1.06 */

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

proc freq data=baseball_Sort;
	where Team in ("San Francisco", "Los Angeles", "Oakland");
	table name;
run;

/* 1.07 */

proc freq data=cr.employee_raw order=freq;
	table EmpID;
run;

proc freq data=cr.employee_raw;
	table Country;
run;

proc freq data=cr.employee_raw order=freq;
	table Department;
run;

proc print data=cr.employee_raw;
	where TermDate is not missing and TermDate<HireDate;
run;

/* 1.08 */

proc sort data=cr.employee_raw out=emp_sort nodupkey;
	by _all_;
run;

proc print data=emp_sort;
	where JobTitle contains 'Logistics';
	format HireDate year.;
run;

proc means data=emp_sort mean;
	where HireDate >= "01JAN2010"d and TermDate is missing;
	var Salary;
run;

proc univariate data=emp_sort;
	var Salary;
run;

/* 2.01 */

data holiday2019;
	set sashelp.holiday;
	where end is missing and rule=0;
	CountryCode=substr(Category,lengthn(Category)-1,2);
	Date=mdy(month, day, 2019);
	keep Desc CountryCode Date;
run;

/* 2.02 */

data sales;
	Length SalesLevel $ 6;
	set cr.employee;
	where Department='Sales' and TermDate is missing;
	if JobTitle='Sales Rep. I' then SalesLevel='Entry';
	if JobTitle='Sales Rep. II' or JobTitle='Sales Rep. III' then SalesLevel='Middle';
	if JobTitle='Sales Rep. IV' then SalesLevel='Senior';
run;

proc freq data=sales order=freq;
	Table SalesLevel;
run;

/* 2.03 */

data bonus;
	set cr.employee;
	where TermDate is missing;
	YearsEmp=yrdif(HireDate, "01JAN2019"d, 'age');
	
	if YearsEmp>=10 then do;
		Bonus=.03*Salary;
		Vacation=20;
	end;
	
	else do;
		Bonus=.02*Salary;
		Vacation=15;
	end;
	
run;

proc freq data=bonus order=freq;
	table Vacation;
run;

proc sort data=bonus;
	by Descending YearsEmp;
run;

/* 2.04 */

proc sort data=sashelp.baseball out=baseball_sort;
	by Team Name;
run;

title "Baseball Team Statistics";
proc print data=baseball_sort noobs label;
	by Team;
	var Name Position YrMajor nAtBat nHits nHome;
	sum nAtBat nHits nHome;
	ID Name;
	Label 	Name="Player's Name"
			Position='Position(s) in 1986'
			YrMajor='Years in the Major Leagues'
			nAtBat='Times at Bat in 1986'
			nHits='Hits in 1986'
			nHome='Home Runs in 1986';
run;

/* 2.05 */

proc freq data=cr.employee order=freq nlevels;
	table City Department JobTitle;
run;

/* 2.06 */

proc freq data=cr.profit;
	table Order_Date*Order_Source / nocum norow nocol;
	format Order_Date monname.;
run;

/* 2.07 */

proc means data=cr.employee sum mean min max maxdec=0;
	Var Salary;
	Class JobTitle;
	where Department='Sales';
	ways 0 1;
run;

/* 2.08 */

proc means data=cr.employee mean min max sum noprint;
	Var Salary;
	Class City Department;
	ways 2;
	output out=salary_summary mean=AvgSalary sum=TotalSalary;
run;

/* 2.09 */

ods graphics on;
ods noproctitle;

ods excel file="&outpath/heart.xlsx";
title "Distribution of Patient Status";
proc freq data=sashelp.heart order=freq;
	tables DeathCause Chol_Status BP_Status / nocum plots=freqplot;
run;

title "Summary of Measures for Patients";
proc means data=sashelp.heart mean;
	var AgeAtDeath Cholesterol Weight Smoking;
	class Sex;
run;
ods excel close;	

/* 2.10 */

ods noproctitle;
ods pdf file="&outpath/truck.pdf" startpage=never style=journal;
title "Truck Summary";
title2 "SASHELP.CARS Table";

proc freq data=sashelp.cars;
	where Type="Truck";
	tables Make / nocum;
run;

proc print data=sashelp.cars;
	where Type="Truck";
	id Make;
	var Model MSRP MPG_City MPG_Highway;
run;

ods pdf close;

/* 2.11 */

proc export data=cr.employee_current outfile="&outpath/employee_current.csv" dbms=csv replace;
run;

/* 4.01 */

data emp_US emp_AU;
	set cr.employee(keep=EmpID Name JobTitle Salary Country Department TermDate);
	Country=upcase(Country);
	if Country="US" then output emp_US;
	if Country="AU" then output emp_AU;
	where TermDate is missing;
run;

/* 4.02 */

data Dead Alive;
	set sashelp.heart;
	if status='Dead' then output Dead;
	if status='Alive' then output Alive;
	drop Status DeathCause AgeAtDeath;
run;

/* 4.03 */

proc sort data=cr.employee_current out=emp_sort;
	by Department;
run;

proc means data=emp_sort min max sum mean;
	var Salary;
	class Department;
	output out=salary sum=TotalSalary;
	ways 1;
run;

data salaryforecast;
	set salary;
	
	do Year=1 to 3;
		TotalSalary=TotalSalary*1.03;
		output;
	end;

	format TotalSalary 12.;
run;

/* 4.04 */

proc sort data=sashelp.stocks out=stocks_sort;
	By Stock Date;
run;

data stocks_total;
	set stocks_sort;
	where year(Date)=2005;
	
	if first.Stock=1 then YTDVolume=0;
	YTDVolume+Volume;
	keep Stock Date Volume YTDVolume;
	by Stock;
run;

/* 4.05 */

proc sort data=sashelp.shoes out=shoes_sort;
	by Product Sales;
run;

data highlow;
	length HighLow $ 4;
	set shoes_sort;
	if first.Product then HighLow='Low';
	if last.Product then HighLow='High';
	By Product;
	if first.Product or last.Product;
run;

/* 4.06 */

proc sort data=cr.employee_current out=emp_sort;
	by Department Salary;
run;

data dept_salary;
	set emp_sort;
	
	if first.Department then do;
		TotalDeptSalary=0;
		LowSalaryJob=JobTitle;
	end;
	
	retain LowSalaryJob;
	
	TotalDeptSalary+Salary;
	
	if last.department then do;
		HighSalaryJob=JobTitle;
	end;
	
	by Department;
	keep Department TotalDeptSalary HighSalaryJob LowSalaryJob;
	format TotalDeptSalary dollar12.;
	if last.Department;
run;

/* 4.07 */

proc sort data=sashelp.fish out=fish_sort;
	by Species;
run;

data fish;
	set fish_sort;
	Length=round(mean(of Length:),.01);
	drop Length1 Length2 Length3;
run;

proc means data=fish mean maxdec=2;
	var Length;
	class Species;
run;

proc freq data=fish order=freq;
	table Species;
run;

/* 4.08 */

data outfield;
	set sashelp.baseball;
	where substr(Position,2,1)='F';
	Player= catx(" ", scan(Name,2,','),scan(Name,1));
	BatAvg=round(nHits/nAtBat,.001);
	keep Player BatAvg;
run;

proc sort data=outfield;
	by descending BatAvg;
run;

/* 4.09 */

data emp_new;
	set cr.employee_new(rename=(HireDate=HireDateC));
	EmpID=substr(EmpID, 4);
	HireDate=input(HireDateC, anydtdte10.);
	Salary=input(AnnualSalary, dollar12.);
run;

/* 4.10 */ 

proc format;
	value BMIRANGE 	low-<18.5='Underweight'
					18.5-24.9='Normal'
					25-29.9='Overweight'
					30-high='Obese';
run;

proc freq data=sashelp.bmimen;
	where Age>=21;
	table BMI;
	format BMI BMIRANGE.;
run;

/* 4.11 */

data contfmt;
	set cr.continent_codes;
	FmtName="CONTFMT";
	Start=Code;
	Label=Continent;
	keep FmtName Start Label;
run;

proc format cntlin=contfmt;
run;

proc means data=cr.demographics sum;
	var Pop;
	Class Cont;
	format Cont contfmt.;
run;

/* 4.12 */

proc format;
	value $statfmt S="Single"
	               M="Married"
	               O="Other";
	value salrange low-<50000="Under $50K"
	               50000-100000="50K-100K"
	               100000<-high="Over 100K";
run;

proc freq data=cr.employee;
	tables Status;
	tables City*Salary / nopercent nocol;
	format Salary salrange.;
	format Status $statfmt.;
run;

/* 5.01 */

data q3_sales;
	set cr.m7_sales cr.m8_sales(rename=(Employee_ID=EmpID)) cr.m9_sales;
run;

proc freq data=q3_sales;
	table Order_type;
run;

/* 5.02 */

proc sort data=cr.employee;
	by EmpID;
run;

proc sort data=cr.employee_addresses;
	by Employee_ID;
run;

data emp_full;
	merge cr.employee(in=inemp) cr.employee_addresses(rename=(Employee_ID=EmpID) in=inadd);
	if inemp=1;
	by EmpID;
run;


/* 5.03 */

proc sort data=cr.employee(keep=EmpID Name Department) out=emp_sort;
	by EmpID;
run;

proc sort data=cr.employee_donations out=donate_sort;
	by EmpID;
run;

data donation nodonation;
	merge emp_sort(in=in_emp) donate_sort(in=in_don);
	by EmpID;
		TotalDonation=sum(of Qtr1-Qtr4);
	if in_don=1 and in_emp=1 then output donation;	
	if in_don=0 and in_emp=1 then output nodonation;
run;

/* 5.04 */

data shoes_future;
	set cr.shoes_summary;
	
	do year=1 to 5;
		ProfitPerStore=ProfitPerStore*1.03;
		output;
	end;
	
	Drop TotalStores TotalProfit;
run;

/* 5.05 */

data future_expenses;
   Wages=12874000;
   Retire=1765000;
   Medical=649000;
	
	do year=1 to 10;
		Wages=Wages*1.06;
		Retire=Retire*1.014;
		Medical=Medical*1.095;
		TotalCost=sum(Wages,Retire,Medical);
	output;
	end;
	
	format Wages Retire Medical TotalCost 12.;
run;

/* 5.06 */

data income_expenses;
	Wages=12874000;
	Retire=1765000;
	Medical=649000;
	Income=50000000;

	do until(TotalCost>Income);
		Year +1;
		Wages=Wages*1.06;
		Retire=Retire*1.014;
		Medical=Medical *1.095;
		Income=Income*1.01;
		TotalCost=sum(Wages, Retire, Medical);
		output;
	end;
	
	keep Year TotalCost;
	format TotalCost comma12.;
run;

/* 5.07 */

proc sort data=sashelp.shoes out=shoes_sort nodupkey;
	by Region Subsidiary Product;
run;

proc transpose data=shoes_sort out=shoes_sales(drop=_Name_ _Label_);
	var Sales;
	By Region Subsidiary;
	ID Product;
run;

/* 5.08 */

proc sort data=cr.employee_training out=train_sort;
	by Name;
run;

proc transpose data=train_sort out=training_narrow(rename=(_Name_=Training Col1=Date_Completed));
	var Compliance_Training Corporate_Security On_the_Job_Safety;
	by Name;
run;

proc freq data=training_narrow;
	table Date_Completed;
	format Date_Completed monname.;
run;