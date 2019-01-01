# utl-adding-a-logical-datatype-to-SAS-is-not-a-good-idea
Plethora of SAS procedures to create percentages
    Plethora of SAS procedures to create percentages (8 base procs)

       Six Base Procedures (all produce an output dataset)

           1. Proc Report        (numeric 0/1 variables)
           2. Proc Freq          (should work with numeric or character variables)
           3. Proc Means/Summary (numeric 0/1 variables)
           4. Proc Tabulate      (should work with numeric or character variables)

           5. Proc SQL PG Stats profile (should work with numeric or character variables)
              https://communities.sas.com/t5/user/viewprofilepage/user-id/462

           6. Proc Corr          (numeric 0/1 variables)
           7. Proc Chart vbar    (numeric or charater)
           8. Proc Univariate    (numeric 0/1 variables)


    Adding a logical datatype to SAS is not a good idea

    SAS does not have logicals or factor datatypes, Thank God;
    Less is more and the added value of these additional datatypes is not worth it,
    especially when interoperability with other languages is needed;

    see github
    https://tinyurl.com/ydekwkbj
    https://github.com/rogerjdeangelis/utl-adding-a-logical-datatype-to-SAS-is-not-a-good-idea

    SAS Forum (related to)
    https://communities.sas.com/t5/SAS-Programming/percentage-Caluculation/m-p/523710



    INPUT
    =====

    * this transforms to 0/1 but not needed for many of the procedures)
    proc format;
     invalue yn2num
      'Y'=1
      'N'=0
    ;run;quit;

    data have;
     retain
     informat var1-var3 yn2num.;
     input accno Var1$ var2$ var3$;

    cards4;
    111 Y Y N
    112 Y Y Y
    113 N Y Y
    114 Y N N
    115 Y Y Y
    116 Y Y Y
    117 Y Y N
    118 Y Y Y
    119 Y Y Y
    120 N Y Y
    ;;;;
    run;quit;

    Up to 40 obs WORK.HAVE total obs=10    | RULES

                                           | Percent
                                           | -------
    Obs    ACCNO    VAR1    VAR2    VAR3   | Var3
                                           |
      1     111      1       1       0     |
      2     112      1       1       1     |
      3     113      0       1       1     |
      4     114      1       0       0     |
      5     115      1       1       1     |
      6     116      1       1       1     |
      7     117      1       1       0     |
      8     118      1       1       1     |
      9     119      1       1       1     |
     10     120      0       1       1     | 70%   sum(Var3)/count(var3)


    EXAMPLE OUTPUT
    --------------

    ===============
    1. Proc Report
    ===============

    proc report data=have nowd missing out=wantRpt(keep=pct1 pct2 pct3);

    cols accno var1 var2 var3 pct1 pct2 pct3;
    define accno /  n 'N';
    define var1  / analysis sum;
    define var2  / analysis sum;
    define var3  / analysis sum;
    define pct1  / computed format=percent.;
    define pct2  / computed format=percent.;
    define pct3  / computed format=percent.;
    compute pct3;
      pct1=var1.sum/accno.n;
      pct2=var2.sum/accno.n;
      pct3=var3.sum/accno.n;
    endcomp;
    run;quit;

    Up to 40 obs from WANTRPT total obs=1

    Obs    PCT1    PCT2    PCT3

     1      0.8     0.9     0.7


    ============
    2. Proc freq
    ============

    ods output onewayfreqs=wantFrq
       (keep=table percent cumpercent where=(cumpercent=100));
    proc freq data=have;
    tables var1 var2 var3;
    run;quit;

    Up to 40 obs from WANTFRQ total obs=3

        TABLE       PERCENT    CUMPERCENT

      Table VAR1       80          100
      Table VAR2       90          100
      Table VAR3       70          100



    =====================
    3  Proc Means/Summary
    =====================

    proc means data=have ;
    var var1 var2 var3;
    output out=wantAve(drop=_type_)  mean=;
    run;quit;

    Up to 40 obs from WANTAVE total obs=1

      _FREQ_    VAR1    VAR2    VAR3

        10       0.8     0.9     0.7


    ================
    4. Proc Tabulate
    =================

    * I am not a fan of tabulate becuse the output dataset is difficult to use;
    proc tabulate data=have out=wantTab(drop=_: where=(coalesce(var1,var2,var3)));
    class var1 var2 var3;
    table var1 var2 var3,(n pctn);
    run;quit;

    Up to 40 obs from WANTTAB total obs=3

    Obs    VAR1    VAR2    VAR3    N    PCTN_000

     1       1       .       .     8       80
     2       .       1       .     9       90
     3       .       .       1     7       70


    ==================================================================
    5. Proc SQL PG Stats profile
       https://communities.sas.com/t5/user/viewprofilepage/user-id/462
    ==================================================================

    %array(pcts,values=1-3);

    proc sql;
      create
         table wantSql as
      select
         count(*) as tot
        ,%do_over(pcts,phrase=%str(
            sum(var?)/count(var?) as pctVar?),between=comma
         )
      from
         have
    ;quit;

    Up to 40 obs from WANTSQL total obs=1

    Obs    TOT    PCTVAR1    PCTVAR2    PCTVAR3

     1      10      0.8        0.9        0.7


    ============
    6. Proc Corr
    ============

    ods trace on;
    ods output simplestats=wantCorr;
    proc corr data=have (keep=var1-var3);
    var var1 var2 var3;
    run;quit;
    ods trace off;

    Up to 40 obs from WANTCORR total obs=3

    Obs    VARIABLE    NOBS    MEAN     STDDEV    SUM    MIN    MAX

     1       VAR1       10      0.8    0.42164     8      0      1
     2       VAR2       10      0.9    0.31623     9      0      1
     3       VAR3       10      0.7    0.48305     7      0      1


    ============
    7. Proc Chart
    ============

    ods trace on;
    options ls=64 ;
    ods output hbar=wantHbr (where=(index(batch,'100.00')>0));
    proc chart data=have;
     hbar var1 var2 var2 var3;
    run;quit;
    ods trace off;


    ==================
    8. Proc Univariate
    ==================
                              ++
    * Slightly edited

    Up to 40 obs from WANTHBR total obs=4

    Obs TYPE                          BATCH

     1   D   0.9   |****************       8    10    80.00   100.00
     2   D   0.9   |******************     9    10    90.00   100.00
     3   D   0.9   |******************     9    10    90.00   100.00
     4   D   0.9   |**************         7    10    70.00   100.00


    ods trace on;
    ods output moments=wantUnv(keep=varname label1 nvalue1 where=(label1 in ('N','Mean'))) ;
    proc univariate data=have;
    var var1-var3;
    run;quit;
    ods trace off;

    Up to 40 obs from WANTUNV total obs=6

    Obs    VARNAME    LABEL1    NVALUE1

     1      VAR1       N          10.0
     2      VAR1       Mean        0.8
     3      VAR2       N          10.0
     4      VAR2       Mean        0.9
     5      VAR3       N          10.0
     6      VAR3       Mean        0.7

