/* CONCATENATING TABLES */

%let path=~/ECRB94/data;
%let outpath=~/ECRB94/output;
libname cr "&path";

/* analying product lines and category across two concatenated tables */

data profit;
	length Customer_Continent $ 20;
	set cr.orders cr.orders2017(rename=(Line=Product_Line Category=Product_Category));
	length Order_Source $ 8;
	where Delivery_Date>=Order_Date;
	Customer_Country=upcase(Customer_Country);
	if Quantity < 0 then Quantity=.;
	Profit=(Retail_Price-Cost_Price)*Quantity;
	format Profit dollar12.2;
	ShipDays=Delivery_Date-Order_Date;
	Age_Range=substr(Customer_Age_Group,1,5);
	If Order_Type=1 then Order_Source='Retail';
	else if Order_Type=2 then Order_Source='Phone';
	else if Order_Type=3 then Order_Source='Internet';
	else Order_Source='Unknown';
	drop Retail_Price Cost_Price Customer_Age_Group Order_Type;
run;

/* analyzing product lines and category with merged tables */

proc sort data=cr.profit out=profit_sort;
	by Product_ID;
run;

proc sort data=cr.products out=products_sort;
	by Product_ID;
run;

/* using in statement to assign values to temporary variables inprof and inprod and the BY statement
if Product_ID is in the merge, then inprof = 1 if not inprof=0.  inprod=1 because it contains all Product_IDs
so we can use logic to see which Product_IDs have been sold and which have not */
 
data profit_detail product_nosales(keep=Product_ID Product_Name);
	merge profit_sort(in=inprof) products_sort(in=inprod);
	by Product_ID;
	if inprof=1 AND inprod=1 then output profit_detail;
	if inprof=0 and inprod=1 then output product_nosales;
run;

/* concatenate my7-my9 sales files */

%let path=~/ECRB94/data/;
%let outpath=~/ECRB94/output/;
libname cr "&path";

data q3_sales;
	set cr.m7_sales cr.m8_sales cr.m9_sales(rename=(EmpID=Employee_ID));
run;

proc sort data=q3_sales;
	by Order_Type;
run;

proc freq data=q3_sales;
	table Order_Type;
run;

/* merge employee and employee_addresses datasets by Employee_ID */

proc sort data=cr.employee_addresses(rename=(Employee_ID=EmpID)) out=employeeSort;
	by EmpID;
run;

data emp_full;
	merge cr.employee(in=inemp) employeeSort;
	by EmpID;
	if inemp;
run;

proc freq data=cr.employee_addresses order=freq;
	table Employee_ID;
run;

/* analyze employee and their donations */

proc sort data=cr.employee(keep=EmpID Name Department) out=emp_sort;
	by EmpID;
run;

proc sort data=cr.employee_donations out=donate_sort;
	by EmpID;
run;

data donation nodonation;
	merge emp_sort(in=in_emp) donate_sort(in=in_don);
	by EmpID;
	if in_don=1 and in_emp=1 then TotalDonation=sum(of Qtr1-Qtr4);
	if in_don=1 and in_emp=1 then output donation;
	if in_don=0 and in_emp=1 then output nodonation;
run;

	