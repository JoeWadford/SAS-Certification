/* PROCESSING REPETITIVE CODE */

/* USING ITERATIVE AND CONDITIONAL DO LOOPS */

%let path=~/ECRB94/data;
%let outpath=~/ECRB94/output;
libname cr "&path";

/* example of repetitive code */

data profit_forecast;
	set cr.profit_summary;
	Year=1;
	TotalProfit=TotalProfit*1.05;
	output;
	Year=2;
	TotalProfit=TotalProfit*1.05;
	output;
	Year=3;
	TotalProfit=TotalProfit*1.05;
	output;
run;

/* iterative do loop */

data profit_forecast;
	set cr.profit_summary;
	do year=1 to 3;
		TotalProfit=TotalProfit*1.05;
		output;
	end;
run;

/* rewarding best customers based on investments with conditional do loops */

data rewards;
	Reserve=1000000;
	Rewards=-100000;
	do while(Reserve >=0);
		Year+1;
		Reserve+25000;
		Reserve+(Reserve*.03);
		Reserve+Rewards;
		output;
	end;
	format Reserve Dollar12.;
run;

/* NESTED DO LOOPS */

data rates;
	input rate;
	datalines;
.02
.03
.04
.05
;
run;

data rewards;
	set rates;
	Reserve=1000000;
	Rewards=-100000;
	do Year=1 to 50 until(Reserve<=0);
		do Month=1 to 12;
			Reserve+5000;
			Reserve+(Reserve*rate/12);
		end;
		Reserve+Rewards;
	end;
	format Reserve Dollar12.;
	drop Rewards Month;
run;

/* quiz */

data work.invest1;
    capital=100000;
    do while(Capital le 500000);
        Year+1;
        capital+(capital*.10);
    end;
run;

data work.invest;
    capital=100000;
    do until(Capital gt 500000);
        Year+1;
        capital+(capital*.10);
    end;
run;

/* Shoe Profit */

data shoes_future;
	set cr.shoes_summary;
	do Year=1 to 5;
		ProfitPerStore=ProfitPerStore*1.03;
		output;
	end;
	drop TotalStores TotalProfit;
run;

/* wages vs retirement and medical */

data future_expenses;
   	Wages=12874000;
   	Retire=1765000;
   	Medical=649000;
   	
   	do Year=1 to 10;
  		Wages=Wages*1.06;
  		Retire=Retire*1.014;
  		Medical=Medical*1.095;
  		TotalCost=Medical+Wages+Retire;
  		output;
  	end;
  	
  	format Wages Retire Medical TotalCost COMMA10.;
run;

/* Income and TotalCost analysis */

data income_expenses;
	Wages=12874000;
	Retire=1765000;
	Medical=649000;
	Income=50000000;

	do while(TotalCost<Income);
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
