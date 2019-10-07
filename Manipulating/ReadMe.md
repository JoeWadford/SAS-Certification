/* MANIPULATING DATA */

%let path=~/ECRB94/data;
%let outpath=~/ECRB94/output;
libname cr "&path";

/* review the functions listed in the certification exam overview */

/* creating first name columns and customer age and promo date */
data qtr_detail;
	set cr.qtr_sales;
	TotalPurchase=sum(of qtr:);
	AvgPurchase=round(mean(of qtr:),.01);
	CustomerAge=int(yrdif(BirthDate,Today(),"age"));
	PromoDate=mdy(month(BirthDate),1,year(today()));
	FirstName=scan(Name,1, " ");
	ID=put(Customer_ID, z5.);
	format TotalPurchase AvgPurchase dollar12.2 PromoDate mmddyy10.;
	drop Customer_ID qtr:;
run;

data qtr_detail;
	retain ID Name FirstName BirthDate Customer_Age Promo_Date TotalPurchase AvgPurchase;
	set qtr_detail;
run;

/* INTNX and INTCK */

data _6months;
	set cr.profit;
	where Order_Date>=intnx('month', "09NOV2018"d, -6, "same"); /* uses the same date six months ago instead
	of the 1st of the month six months ago*/
	BusDays=intck("weekday",Order_Date,Delivery_Date);
	keep Order_ID Order_Date Delivery_Date BusDays;
run;

/* analysis of Fish Length by Species */

data fish; 
	set sashelp.fish;
	Length = round(mean(of Length:),.01);
	drop Length1 Length2 Length3;
run;

proc sort data=fish;
	by Species;
run;

proc means data=fish mean max min maxdec=2;
	by Species;
run;

proc freq data=fish;
	table Species;
run;

/* analyzing outfielders in baseball */

proc sort data=sashelp.baseball out=work.outfieldSort;
	by Position Name;
	where substr(Position,2,1)="F";
run;

data outfield;
	set work.outfieldSort;
	Player=catx(" ",scan(Name,2),scan(Name,1));
	BatAvg=round(nHits/nAtBat,.001);
	Keep Player BatAvg Position;
run;

proc sort data=outfield;
	by descending BatAvg Player;
run;

/* p203q3 */
/* analyzing hire dates and salaries */

%let path=~/ECRB94/data;
%let outpath=~/ECRB94/output;
libname cr "&path";

data emp_new;
	set cr.employee_new;
	EmpID=substr(EmpID, 4);
	HireDate=input(HireDate, date9.);
	Salary=input(AnnualSalary, dollar12.);
run;
