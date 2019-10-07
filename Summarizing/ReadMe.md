/* SUMMARIZING DATA */

%let path=~/ECRB94/data;
%let outpath=~/ECRB94/output;
libname cr "&path";

data new;
	putlog "NOTE: Value of HeightCM at the top of the DATA step";
	putlog HeightCM=;
	retain HeightCM 0;
	set sashelp.class(obs=3);
	HeightCM=Height*2.54;
	putlog "NOTE: Value of HeightCM at the bottom of the DATA step";
	putlog HeightCM=;
run;

/* accumulating sales for Month and Day */
proc sort data=cr.profit out=decDaily;
	where month(Order_Date)=12;
	by Order_Date;
run;

data DecSales;
	set decDaily;
	by Order_Date;
	MTDSales + Profit;
	/* the sum statement retains MTDSales, initializes the value to 0, adds the value of Profit,
	and ignores missing values */
	if first.Order_Date=1 then DailySales=0;
	DailySales+Profit;
	if last.Order_Date=1;
	Format MTDSales DailySales dollar12.;
	keep Order_ID Order_Date MTDSales DailySales;
run;

/* sorting and accumulating by Stock and Date for 2005 */
proc sort data=sashelp.stocks out=work.StockSort;
	by Stock Date;
	where year(Date)=2005;
run;

data stocks_total;
	set work.StockSort;
	by Stock;
	YTDVolume+Volume;
	keep Stock Date YTDVolume Volume;
run;
	
/* finding the highest and lowest sales for each product by region and subsidiary */	
proc sort data=sashelp.shoes out=work.ShoeSort;
	by Product Sales;
run;

data highlow;
	set work.ShoeSort;
	length HighLow $ 8;
	by Product;
	if first.Product then HighLow='Low';
	if last.Product then HighLow='High';
	if first.Product or last.Product;
run;

/* p202q3 high and low salary job and total salaries by department */

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
	
