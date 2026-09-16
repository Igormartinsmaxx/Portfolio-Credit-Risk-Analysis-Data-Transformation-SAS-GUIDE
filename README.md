# Portfolio-Credit-Risk-Analysis-Data-Transformation-SAS-GUIDE
Script Name: credit_risk_analysis.sas    Description: Portfolio Credit Risk Analysis &amp; Data Transformation    Author: Igor Martins    Tools: SAS Enterprise Guide / Base SAS

SAS
/* =====================================================================
   Script Name: credit_risk_analysis.sas
   Description: Portfolio Credit Risk Analysis & Data Transformation
   Author: Igor Martins
   Tools: SAS Enterprise Guide / Base SAS
   ===================================================================== */

/* 1. Create a simulated bank loan dataset */
data work.raw_loans;
    input CustomerID $ CreditScore Balance Limit TotalDebt;
    datalines;
CUST001 720 15000 20000 5000
CUST002 580 45000 50000 32000
CUST003 640 12000 15000 8000
CUST004 490 85000 90000 70000
CUST005 800 2000  10000 500
CUST006 610 30000 35000 20000
;
run;

/* 2. Feature Engineering & Risk Rules Application */
data work.credit_risk_prepared;
    set work.raw_loans;
    
    /* Calculate Credit Limit Utilization Rate */
    if Limit > 0 then CreditUtilization = Balance / Limit;
    else CreditUtilization = 0;
    
    /* Categorize Customers into Risk Bands based on Credit Score */
    if CreditScore >= 750 then RiskBand = '1 - Low Risk';
    else if CreditScore >= 650 then RiskBand = '2 - Medium-Low Risk';
    else if CreditScore >= 550 then RiskBand = '3 - Medium-High Risk';
    else RiskBand = '4 - High Risk (Watchlist)';

    /* Assign Estimated Probability of Default (PD) per Risk Band */
    select (RiskBand);
        when ('1 - Low Risk')              Estimated_PD = 0.01;
        when ('2 - Medium-Low Risk')       Estimated_PD = 0.03;
        when ('3 - Medium-High Risk')      Estimated_PD = 0.08;
        when ('4 - High Risk (Watchlist)') Estimated_PD = 0.20;
        otherwise Estimated_PD = .;
    end;

    /* Calculate Exposure at Default (EAD) */
    EAD = Balance + (TotalDebt * 0.50);

    format CreditUtilization percent8.2 
           Estimated_PD percent8.2 
           Balance Limit TotalDebt EAD dollar12.2;
run;

/* 3. Aggregation and Summarized Portfolio Risk Report */
proc means data=work.credit_risk_prepared n mean sum maxdec=2;
    class RiskBand;
    var Balance EAD Estimated_PD CreditUtilization;
    title "Summary of Credit Risk Portfolio by Risk Band";
run;

/* 4. Filter High-Risk Accounts for Compliance and Monitoring */
proc sql;
    create table work.high_risk_watchlist as
    select 
        CustomerID,
        CreditScore,
        Balance,
        EAD,
        Estimated_PD,
        CreditUtilization
    from work.credit_risk_prepared
    where RiskBand = '4 - High Risk (Watchlist)'
    order by Balance desc;
quit;

/* Clear report titles */
title;
