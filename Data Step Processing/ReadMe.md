/* DATA Step Processing */
/* PDV compilation = where the PDV is compiled and created

Compile-time functions/actions set where length variable_creation format keep drop

NOTE: where statement flags identified variables for the execution phase */
/* PDV execution = where observations are populated and processed and rows are written to the output table

each col is assigned a missing value to begin with the exception of _ERROR_ and _N_
_N_ = is a counter indicates the interation number of the data step
_ERROR_ if a data error is encountered for a row = 1, 0 otherwise

before the run statement, there is an implicit output where a row is written to the PDV output statement
then SAS loops back to the top of the data step

*/

%let path=~/ECRB94/data;
%let outpath=~/ECRB94/output;
libname cr "&path";

data dead(drop=Status) alive(drop=Status DeathCause AgeAtDeath);
	set sashelp.heart;
	Status=propcase(Status);

	if Status='Alive' then
		output alive;
	else if Status='Dead' then
		output dead;
run;

proc contents data=cr.employee_current;
run;

proc sort data=cr.employee_current;
	by Department;
run;

data work.salary;
	set cr.employee_current;
	by Department;
	length TotalSalary 8;
	retain TotalSalary;

	if first.Department then
		TotalSalary=0;

	if Department='Sales Management' then
		TotalSalary=sum(TotalSalary, Salary);

	if Department='Administration' then
		TotalSalary=sum(TotalSalary, Salary);

	if Department='Engineering' then
		TotalSalary=sum(TotalSalary, Salary);

	if Department='Sales' then
		TotalSalary=sum(TotalSalary, Salary);

	if Department='Executives' then
		TotalSalary=sum(TotalSalary, Salary);

	if Department='Group Financials' then
		TotalSalary=sum(TotalSalary, Salary);

	if Department='Secretary of the Board' then
		TotalSalary=sum(TotalSalary, Salary);

	if Department='Strategy' then
		TotalSalary=sum(TotalSalary, Salary);

	if Department='Concession Management' then
		TotalSalary=sum(TotalSalary, Salary);

	if Department='Logistics Management' then
		TotalSalary=sum(TotalSalary, Salary);

	if Department='Stock & Shipping' then
		TotalSalary=sum(TotalSalary, Salary);

	if Department='Accounts' then
		TotalSalary=sum(TotalSalary, Salary);

	if Department='Accounts Management' then
		TotalSalary=sum(TotalSalary, Salary);

	if Department='Group HR Management' then
		TotalSalary=sum(TotalSalary, Salary);

	if Department='IS' then
		TotalSalary=sum(TotalSalary, Salary);

	if Department='Marketing' then
		TotalSalary=sum(TotalSalary, Salary);

	if Department='Purchasing' then
		TotalSalary=sum(TotalSalary, Salary);
	keep Department TotalSalary;
	format TotalSalary DOLLAR15.;

	if last.Department=1;
run;

data salaryforecast;
	set work.salary;
	length Year 8;
	Year=1;

	do Year=1 to 3;
		TotalSalary=TotalSalary*1.03;
		output;
	end;
run;	
	