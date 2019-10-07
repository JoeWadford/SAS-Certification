/* PRACTICE QUIZZES */

/* Programming Question 1.01 */

data CanadaSales;
	set sashelp.prdsale;
	Diff=Actual-Predict;
	where Country='CANADA' and Quarter=1;
run;

/* Programming Question 1.02 */ 

/*This program analyzes blood pressure
in the SASHELP.HEART table.*/

proc freq data=sashelp.heart;
    table BP_Status;
run;

proc means data=sashelp.heart min max mean maxdec=0;
    var systolic diastolic;
    class BP_Status;
run;

/* Programming Question 1.03 */

%let path=~/ECRB94/data;
%let outpath=~/ECRB94/output;
libname cr "&path";

proc contents data=cr._all_ nods;
run;

/* Programming Question 1.04 */

proc import datafile='/home/u1421477/ECRB94/data/payroll.csv' dbms=csv out=work.payroll replace;
	guessingrows=max;
run;

proc contents data=work.payroll;
run;

/* Programming Question 1.05 */
options validvarname=v7;
libname xl xlsx "&path/employee.xlsx";
proc contents data=xl._all_;
run;

/* Programming Question 1.06 */

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

/* Programming Question 1.07 */

proc freq data=cr.employee_raw order=freq;
	table EmpID Country Department;
run;

proc print data=cr.employee_raw;
	where TermDate<HireDate;
	format HireDate TermDate date10.;
run;

/* Programming Question 1.08 */

proc sort data=cr.employee_raw out=emp_sort nodupkey dupout=emp_dup;
	By _all_;
run;

data emp_logistics;
	set emp_sort;
	where JobTitle contains 'Logistics';
	format HireDate TermDate year.;
run;

proc means data=emp_sort mean;
	where HireDate > "01Jan2010"d and TermDate =.;
	var Salary;
run;

proc sort data=emp_sort out=bossbitch;
	by descending Salary;
run;

proc univariate data=emp_sort;
	var Salary;
run;

/* Programming Question 2.01 */

data holiday2019;
	set sashelp.holiday;
	where end is missing and rule=0;
	CountryCode=substr(Category,length(category)-1,2);
	Date=mdy(Month, day, 2019);
	keep Desc CountryCode Date;
run;

proc freq data=holiday2019 order=freq;
	table CountryCode;
run;

/* Programming Question 2.02 */


data sales;
	set cr.employee;
	where Department='Sales' AND TermDate=.;
	length SalesLevel $ 6;
	if Jobtitle='Sales Rep. I' then SalesLevel='Entry';
	else if Jobtitle='Sales Rep. II' or Jobtitle='Sales Rep. III'
	then SalesLevel='Middle';
	else if Jobtitle='Sales Rep. IV' then SalesLevel='Senior';
run;

proc freq data=sales order=freq nlevels;
	table SalesLevel;
run;

/* Programming Question 2.03 */

data bonus;
	set cr.employee;
	where termdate is missing;
	YearsEmp=yrdif(HireDate,"01Jan2019"d,"age");
	if YearsEmp>=10 then do;
		Bonus=Salary*.03;
		Vacation=20;
	end;
	else do;
		Bonus=Salary*.02;
		Vacation=15;
	end;
run;

proc freq data=bonus order=freq;
	table Vacation;
run;

proc sort data=bonus out=bonus_sort;
	by descending YearsEmp;
run;

/* Programming Question 2.04 */

proc sort data=sashelp.baseball out=baseball_sort;
	by Team Name;
run;

title "Baseball Team Statistics";
proc print data=baseball_sort noobs label;
	var Name Position YrMajor nAtBat nHits nHome;
	sum nAtBat nHits nHome;
	by Team;
	ID Name;
	label 	Name="Player's Name"
			Position='Position(s) in 1986'
			YrMajor='Years in the Major Leagues'
			nAtBat='Times at Bat in 1986'
			nHits='Hits in 1986'
			nHome='Home Runs in 1986';
run;

title;

/* Programming Question 2.05 */

proc freq data=cr.employee order=freq nlevels;
	table City Department JobTitle;
run;

/* Programming Question 2.06 */

proc freq data=cr.profit;
	table Order_Date * Order_Source / nocum norow nocol;
	format Order_Date monname.;
run;

/* Programming Question 2.07 */

proc means data=cr.employee mean min max sum maxdec=0;
	where department = 'Sales';
	var Salary;
	class jobtitle department;
	ways 0 1;
run;

/* Programming Question 2.08 */

proc means data=cr.employee noprint;
	var Salary;
	class Department City;
	output out= work.salary_summary mean=AvgSalary sum=TotalSalary;
	ways 2;
run;

/* Programming Question 2.09 */

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

/* Programming Question 2.10 */

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

/* Programming Question 2.11 */

proc export data=cr.employee_current outfile="&outpath/employee_current.csv" dbms=csv;
run;

/* Programming Question 4.01 */

%let path=~/ECRB94/data;
%let outpath=~/ECRB94/output;
libname cr "&path";

data emp_US emp_AU;
	set cr.employee(keep=EmpID Name JobTitle Salary Department Country TermDate);
	where TermDate=.;
	Country=upcase(country);
	if Country="US" and TermDate=. then output emp_US;
	else if Country="AU" and TermDate=. then output emp_AU;
run;

/* Programming Question 4.02 */

data dead alive;
	set sashelp.heart;
	if status='Dead' then output dead;
	else if status='Alive' then output alive;
	drop Status DeathCause AgeAtDeath;
run;

/* Programming Question 4.03 */

proc means data=cr.employee_current noprint;
	var Salary;
	Class Department;
	output out=work.salary sum=TotalSalary;
	ways 1;
run;

data work.salaryforecast;
	set work.salary;
	Year=1;
	do Year=1 to 3;
	TotalSalary=TotalSalary*1.03;
	output;
	end;
	drop _Type_ _Freq_;
	format TotalSalary dollar12.;
run;
	
/* Programming Question 4.04 */

proc sort data=sashelp.stocks out=stocks_sorted;
	by Stock Date;
	where year(Date)=2005;
run;

data stocks_total;
	set stocks_sorted;
	if first.stock then YTDVolume=0;
	YTDVolume+Volume;
	by Stock Date;
run;

/* Programming Question 4.05 */

proc sort data=sashelp.shoes out=shoes_sort;
	by Product Sales;
run;
	

data highlow;
	Length HighLow $4;
	set shoes_sort;
	if first.Product=1 then HighLow='Low';
	retain HighLow;
	if last.Product=1 then HighLow='High';
	retain HighLow;
	By Product;
	if first.product or last.product;
run;

/* Programming Question 4.06 */

%let path=~/ECRB94/data;
%let outpath=~/ECRB94/output;
libname cr "&path";

proc sort data=cr.employee_current out=emp_sort;
	by Department Salary JobTitle;
run;

data dept_salary;
	set emp_sort;
	length LowSalaryJob HighSalaryJob $ 20;
	by Department;
	retain LowSalaryJob;
	
	if first.Department then do;
		TotalDeptSalary=0;
		LowSalaryJob=JobTitle;
	end;
	
	TotalDeptSalary+Salary;
	
	if last.Department then HighSalaryJob=JobTitle;
	
	if last.department; 
	keep Department TotalDeptSalary HighSalaryJob LowSalaryJob;
	format TotalDeptSalary dollar12.;
run;

/* Programming Question 4.07 */

data fish;
	set sashelp.fish;
	Length=round(mean(of Length:),.01);
	drop Length1 Length2 Length3;
run;

proc means data=fish mean max min maxdec=2;
	var Length;
	class Species;
run;

/* Programming Question 4.08 */

data outfield;
	set sashelp.baseball;
	where substr(Position,2,1)="F";
	Player=catx(" ",scan(Name,2,','),scan(Name,1));
	BatAvg=round(nHits/nAtBat,.001);
	keep Player BatAvg Position;
run;

proc sort data=outfield;
	by descending BatAvg;
run;

data outfield;
    set sashelp.baseball;
    where substr(Position, 2, 1)="F";
    Player=catx(" ", scan(Name, 2), scan(Name, 1));
    BatAvg=round(nHits/nAtBat, .001);
    keep Player BatAvg Position;
run;

proc sort data=outfield;
    by descending BatAvg;
run;

/* Programming Question 4.09 */

data emp_new;
	set cr.employee_new;
	EmpID=substr(EmpID, 4);
	HireDate=input(HireDate, date9.);
	Salary=input(AnnualSalary, 12.);
run;	
	
/* Programming Question 4.10 */

proc format;
	value BMIRANGE  low-<18.5='Underweight'
					18.5-24.9='Normal'
					25-29.9='Overweight'
					30-high='Obese';
run;

proc freq data=sashelp.bmimen;
	table BMI;
	format BMI BMIRANGE.;
	where Age>=21;
run;
			
/* Programming Question 4.11 */

data CONTFMT;
	set cr.continent_codes;
	FmtName="CONTFMT";
	Start=Code;
	Label=Continent;
run;

proc format cntlin=contfmt;
run;

data continent_population;
	set cr.demographics;
	if first.cont then TotalPopulation=0;
	TotalPopulation+Pop;
	by Cont;
	format cont contfmt.;
	if last.cont;
	drop pop;
run;

proc freq data=cr.demographics;
	table Cont;
	format Cont contfmt.;
run;

proc means data=cr.demographics sum maxdec=0;
	var pop;
	class cont;
	format cont contfmt.;
run;

/* Programming Question 4.12 */

proc format;
	value $statfmt S="Single"
	              M="Married"
	              O="Other";
	              
	value salrange low-<50000="Under $50K"
	               50000-100000="50K-100K"
	               100000-high="Over 100K";
run;

proc freq data=cr.employee;
	tables Status;
	tables City*Salary / nopercent nocol;
	format status $statfmt. Salary salrange.;
run;

/* Programming Question 5.01 */

data q3_sales;
	set cr.m7_sales cr.m8_sales(rename=(Employee_ID=EmpID)) cr.m9_sales;
run;

proc freq data=q3_sales order=freq;
	table Order_Type;
run;

/* Programming Question 5.02 */

proc sort data=cr.employee_addresses out=address_sort;
	by Employee_ID;
run;

data emp_full;
	merge cr.employee(in=inemp) address_sort(rename=(Employee_ID=EmpID) in=inadd);
	by EmpID;
	if inemp=1;
run;

/* Programming Question 5.03 */

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

/* Programming Question 5.04 */

data shoes_future;
	set cr.shoes_summary;
	Year=1;
	do Year=1 to 5;
	ProfitPerStore=ProfitPerStore*1.03;
	output;
	end;
	drop TotalStores TotalProfit;
run;

/* Programming Question 5.05 */

data future_expenses;
   Wages=12874000;
   Retire=1765000;
   Medical=649000;
  do Year=1 to 10;
  Wages=Wages*1.06;
  Retire=Retire*1.014;
  Medical=Medical*1.095;
  TotalCost=sum(Wages,Retire,Medical);
  output;
  end;
  format Wages Retire Medical TotalCost COMMA15.;
run;

/* Programming Question 5.06 */

data income_expenses;
	Wages=12874000;
	Retire=1765000;
	Medical=649000;
	Income=50000000;

	do until(TotalCost>Income);
		Year+1;
		Wages=Wages*1.06;
		Retire=Retire*1.014;
		Medical=Medical *1.095;
		TotalCost=sum(Wages, Retire, Medical);
		Income=Income*1.01;
		output;
	end;
	keep Year TotalCost;
	format TotalCost comma12.;
run;

/* Programming Question 5.07 */

proc sort data=sashelp.shoes out=shoes_sort nodupkey;
	by Region Subsidiary Product;
run;

proc transpose data=shoes_sort out=shoes_sales(drop=_Name_ _Label_);
	var Sales;
	By Region Subsidiary;
	ID Product;
run;

/* Programming Question 5.08 */


proc sort data=cr.employee_training out=train_sort;
	by name;
run;

proc transpose data=train_sort out=training_narrow(rename=(_Name_=Training COL1=Date_Completed));
	var Compliance_Training Corporate_Security On_the_Job_Safety;
	by Name;
run;

proc freq data=training_narrow order=freq;
	table Date_completed;
	format Date_Completed monname.;
run;