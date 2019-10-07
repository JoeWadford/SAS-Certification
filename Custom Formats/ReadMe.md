/* Custom Formats */

%let path=~/ECRB94/data;
%let outpath=~/ECRB94/output;
libname cr "&path";

/* adding a shiprange format for tables and analysis */
proc format;
	value shiprange 0="Same day"
					1-3="1-3 days"
					4-7="4-7 days"
					8-high="8+ days"
					.="Unknown";
run;

/* creating a custom format with a label for each country code */

data country;
	set cr.country_clean;
	FmtName="$ctryfmt";
	Start=Country_Key;
	Label=Country_Name;
	keep FmtName Start Label;
run;

proc format cntlin=country;
run;

data profit2;
	set cr.profit;
	ShipRange=put(shipdays,shiprange.);
	format Customer_Country ctryfmt.;
run;

proc freq data=cr.profit;
	table ShipDays;
	format ShipDays shiprange.;
run;


/* analyzing BMI for men */

proc format;
	value BMIRANGE 	low-<18.5="Underweight"
					18.5-24.9="Normal"
					25-29.9="Overweight"
					30-high="Obese";
run;

proc freq data=sashelp.bmimen;
	table BMI;
	where Age >=21;
	format BMI BMIRANGE.;
run;

/* analyzing population per continent */

data CONTFMT;
	set cr.continent_codes;
	FmtName="$CONTFMT";
	Start=Code;
	Label=Continent;
	keep FmtName Start Label;
run;

proc format cntlin=CONTFMT;
run;

proc freq data=cr.demographics nlevels;
	table Cont;
	format Cont CONTFMT.;
run;

proc means data=cr.demographics sum;
	by Cont;
	format Cont CONTFMT.;
run;

/* analyzing marital status and salary */

proc format;
	value $statfmt S="Single"
	               M="Married"
	               O="Other";
run;

proc format;	          
	value salrange low-<50000="Under $50K"
	               50000-<100000="50K-100K"
	               100000-high="Over 100K";
run;

proc freq data=cr.employee;
	tables Status;
	tables City*Salary / nopercent nocol;
	format Status $statfmt. Salary salrange.;
run;

	
	