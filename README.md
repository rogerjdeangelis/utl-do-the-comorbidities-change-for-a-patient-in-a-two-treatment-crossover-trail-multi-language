# utl-do-the-comorbidities-change-for-a-patient-in-a-two-treatment-crossover-trail-multi-language
Do the comorbidities change for a patient in a two treatment crossover clinical trail multi language
    %let pgm=utl-do-the-comorbidities-change-for-a-patient-in-a-two-treatment-crossover-trail-multi-language;

    Do the comorbidities change for a patient in a two treatment crossover clinical trail multi language

         SOLUTIONS

           1 sas datastep
           2 sas w/wo sql arrays
           3 r sql arrays
           4 python sql arrays
           5 r no sql
           6 sas transpose
           7 sas pure sql xpos (you can also run this in r and python - with slight syntax changes)
           8 r loop array
           9 sas loop array

    github
    https://tinyurl.com/39aebf4s
    https://github.com/rogerjdeangelis/utl-do-the-comorbidities-change-for-a-patient-in-a-two-treatment-crossover-trail-multi-language


    SOAPBOX ON
    Looping or sql are considered non-pythonic or
    non r codicum. However sql and loops can be cleaner
    more flexible, maintainable and use simpler sytactically

    Compilers can easily parallize repeatative code.

    Performance is over rated
    SOAPBOX OFF

    stackoverflow R
    https://tinyurl.com/4mpfdsn2
    https://stackoverflow.com/questions/78933594/how-to-count-mismatches-between-two-rows-column-by-column-r


    /**************************************************************************************************************************/
    /*                                  |                                                          |                          */
    /*                                  |                                                          |                          */
    /*         INPUT                    |       PROCESS                                            | OUTPUT                   */
    /*                                  |                                                          |                          */
    /*                                  |                                                          |                          */
    /*  SD1.HAVE total obs=4            | Columns with non matching comobidities                   |                          */
    /*                                  |                                                          |                          */
    /*           comorbidities          |                                                          | PT   PCTDIF              */
    /*        -----------------------   |                                                          |                          */
    /* PT TRT V1 V2 V3 V4 V5 V6 V7 V8   | PT TRT  NE           NE                                  | 1    28.5714 (2/7)       */
    /*                                  |         --           --                                  | 2    14.2857 (1/7)       */
    /*  1  a   0  1  1  2  .  2  1  1   |  1   a   0  1 1 2 NA  2  1  1  2/7 change comorbidites   |                          */
    /*  1  b   1  1  1  2  1  1  1  1   |  1   b   1  1 1 2 1   1  1  1  NA column removed         |                          */
    /*  2  a   2  2  1  0  0  .  0  0   |                                                          |                          */
    /*  2  b   2  2  1  1  0  .  0  0   |         NE                                               |                          */
    /*                                  |         --                                               |                          */
    /*                                  |  2   a   2  2 1 0   0 NA  0  0  1/7 non matches          |                          */
    /*                                  |  2   b   2  2 1 1   0 NA  0  0  NA column removed        |                          */
    /*                                  |                                                          |                          */
    /*                                  |                                                          |                          */
    /*                                  | SAS looping is almost as fast as assembler coding        |                          */
    /*                                  | =================================================        |                          */
    /*                                  |                                                          |                          */
    /*                                  | data want;                                               |                          */
    /*                                  |                                                          |                          */
    /*                                  |  merge sd1.have                                          |                          */
    /*                                  |        sd1.have(firstobs=2 rename=(v1-v8=r1-r8));        |                          */
    /*                                  |   if mod(_n_,2) ne 0; * remove odd rows;                 |                          */
    /*                                  |                                                          |                          */
    /*                                  |   array vs v1-v8;                                        |                          */
    /*                                  |   array rs r1-r8;                                        |                          */
    /*                                  |                                                          |                          */
    /*                                  |   numer=0;                                               |                          */
    /*                                  |   denom=0;                                               |                          */
    /*                                  |                                                          |                          */
    /*                                  |   do idx=1 to 8;                                         |                          */
    /*                                  |     if (not missing(vs[idx]*rs[idx])) then do;           |                          */
    /*                                  |       denom=denom+1;                                     |                          */
    /*                                  |       if vs[idx] ne rs[idx] then numer=numer+1;          |                          */
    /*                                  |     end;                                                 |                          */
    /*                                  |   end;                                                   |                          */
    /*                                  |                                                          |                          */
    /*                                  |   pctDif=100*numer/denom;                                |                          */
    /*                                  |   keep pt pctdif;                                        |                          */
    /*                                  |                                                          |                          */
    /*                                  | run;quit;                                                |                          */
    /*                                  |                                                          |                          */
    /*                                  | proc print data=want;                                    |                          */
    /*                                  | run;quit;                                                |                          */
    /*                                  |                                                          |                          */
    /*                                  |                                                          |                          */
    /*------------------------------------------------------------------------------------------------------------------------*/
    /*                                  |                                                          |                          */
    /*                                  | R                                                        |                          */
    /*                                  | =                                                        |                          */
    /*                                  |                                                          |                          */
    /*                                  | df %>%                                                   | # A tibble: 2 Ã— 2       */
    /*                                  |   pivot_longer(-V1) %>%                                  |   grp     mismatch       */
    /*                                  |   group_by(grp = sub("(.*\\d+).*"                        |   <chr>   <chr>          */
    /*                                  |    ,"\\1", V1)                                           | 1 Sample1 2/7            */
    /*                                  |    ,name) %>%                                            | 2 Sample2 1/7            */
    /*                                  |   filter(!any(is.na(value))) %>%                         | >                        */
    /*                                  |   summarize(mismatch = value[1] != value[2]) %>%         |                          */
    /*                                  |   ungroup() %>%                                          |                          */
    /*                                  |   summarize(mismatch = paste0(sum(mismatch)              |                          */
    /*                                  |    , "/", n()), .by = grp)                               |                          */
    /*                                  |                                                          |                          */
    /*                                  |                                                          |                          */
    /*                                  | SOAPBOX ON                                               |                          */
    /*                                  | Looping or sql are considered non-pythonic or            |                          */
    /*                                  | non r codicum. However sql and loops can be cleaner      |                          */
    /*                                  | more flexible, maintainable and use simpler sytactically |                          */
    /*                                  |                                                          |                          */
    /*                                  | Compilers can easily parallize repeatative code.         |                          */
    /*                                  |                                                          |                          */
    /*                                  | Performance is over rated                                |                          */
    /*                                  | SOAPBOX OFF                                              |                          */
    /*                                  |                                                          |                          */
    /*                                  |                                                          |                          */
    /**************************************************************************************************************************/

    /*                   _
    (_)_ __  _ __  _   _| |_
    | | `_ \| `_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    */

    options validvarname=upcase;
    libname sd1 "d:/sd1";
    data sd1.have;
     input pt $ trt $ v1-v8;
    cards4;
    1 a 0 1 1 2 .  2 1 1
    1 b 1 1 1 2 1  1 1 1
    2 a 2 2 1 0 0  . 0 0
    2 b 2 2 1 1 0  . 0 0
    ;;;;
    run;quit;

    /*                       _       _            _
    / |  ___  __ _ ___    __| | __ _| |_ __ _ ___| |_ ___ _ __
    | | / __|/ _` / __|  / _` |/ _` | __/ _` / __| __/ _ \ `_ \
    | | \__ \ (_| \__ \ | (_| | (_| | || (_| \__ \ ||  __/ |_) |
    |_| |___/\__,_|___/  \__,_|\__,_|\__\__,_|___/\__\___| .__/
                                                         |_|
    */

    data want;

     merge sd1.have
           sd1.have(firstobs=2 rename=(v1-v8=r1-r8));
      if mod(_n_,2) ne 0; * remove odd rows;

      array vs v1-v8;
      array rs r1-r8;

      numer=0;
      denom=0;

      do idx=1 to 8;
        if (not missing(vs[idx]*rs[idx])) then do;
          denom=denom+1;
          if vs[idx] ne rs[idx] then numer=numer+1;
        end;
      end;

      pctDif=100*numer/denom;
      keep pt trt pctdif;

    run;quit;

    proc print data=want;
    run;quit;

    /*___                              __                          _
    |___ \   ___  __ _ ___  __      __/ /_      _____    ___  __ _| |   __ _ _ __ _ __ __ _ _   _ ___
      __) | / __|/ _` / __| \ \ /\ / / /\ \ /\ / / _ \  / __|/ _` | |  / _` | `__| `__/ _` | | | / __|
     / __/  \__ \ (_| \__ \  \ V  V / /  \ V  V / (_) | \__ \ (_| | | | (_| | |  | | | (_| | |_| \__ \
    |_____| |___/\__,_|___/   \_/\_/_/    \_/\_/ \___/  |___/\__, |_|  \__,_|_|  |_|  \__,_|\__, |___/
                                                                |_|                         |___/
    */


    proc datasets lib=work nodetails nolist;
     delete sqlwant;
    run;quit;

    proc sql;
      create
         table sqlwant as
      select
         l.pt
        ,sum(
         (l.v1 ne r.v1) and not missing(l.v1 * r.v1)
        ,(l.v2 ne r.v2) and not missing(l.v2 * r.v2)
        ,(l.v3 ne r.v3) and not missing(l.v3 * r.v3)
        ,(l.v4 ne r.v4) and not missing(l.v4 * r.v4)
        ,(l.v5 ne r.v5) and not missing(l.v5 * r.v5)
        ,(l.v6 ne r.v6) and not missing(l.v6 * r.v6)
        ,(l.v7 ne r.v7) and not missing(l.v7 * r.v7)
        ,(l.v8 ne r.v8) and not missing(l.v8 * r.v8)
          ) /
        sum(
         (not missing(l.v1 * r.v1))
        ,(not missing(l.v2 * r.v2))
        ,(not missing(l.v3 * r.v3))
        ,(not missing(l.v4 * r.v4))
        ,(not missing(l.v5 * r.v5))
        ,(not missing(l.v6 * r.v6))
        ,(not missing(l.v7 * r.v7))
        ,(not missing(l.v8 * r.v8))
          ) as denom

      from
          sd1.have as l, sd1.have as r
      where
              l.trt ="a"
          and r.trt ="b"
          and l.pt = r.pt
    ;quit;


    proc sql;
      create
         table sqlwant as
      select
         l.pt
        ,((l.v1 ne r.v1) and not missing(l.v1 * r.v1)
        +(l.v2 ne r.v2) and not missing(l.v2 * r.v2)
        +(l.v3 ne r.v3) and not missing(l.v3 * r.v3)
        +(l.v4 ne r.v4) and not missing(l.v4 * r.v4)
        +(l.v5 ne r.v5) and not missing(l.v5 * r.v5)
        +(l.v6 ne r.v6) and not missing(l.v6 * r.v6)
        +(l.v7 ne r.v7) and not missing(l.v7 * r.v7)
        +(l.v8 ne r.v8) and not missing(l.v8 * r.v8))
          / (
         (not missing(l.v1 * r.v1))
        +(not missing(l.v2 * r.v2))
        +(not missing(l.v3 * r.v3))
        +(not missing(l.v4 * r.v4))
        +(not missing(l.v5 * r.v5))
        +(not missing(l.v6 * r.v6))
        +(not missing(l.v7 * r.v7))
        +(not missing(l.v8 * r.v8)))
           as denom
      from
          sd1.have as l, sd1.have as r
      where
              l.trt ="a"
          and r.trt ="b"
          and l.pt = r.pt
    ;quit;

    /*        _ _   _                 _
    __      _(_) |_| |__    ___  __ _| |  __ _ _ __ _ __ __ _ _   _ ___
    \ \ /\ / / | __| `_ \  / __|/ _` | | / _` | `__| `__/ _` | | | / __|
     \ V  V /| | |_| | | | \__ \ (_| | || (_| | |  | | | (_| | |_| \__ \
      \_/\_/ |_|\__|_| |_| |___/\__, |_| \__,_|_|  |_|  \__,_|\__, |___/
                                   |_|                        |___/
    */

    %array(_ix,values=1-8);

    proc sql;
      select
         l.pt
        ,sum(
           %do_over(_ix,phrase=(l.v? ne r.v?) and not missing(l.v? * r.v?)
               ,between=comma)
           ) /
         sum(
           %do_over(_ix,phrase=not missing(l.v? * r.v?)
               ,between=comma)
          ) as frac
      from
          sd1.have as l, sd1.have as r
      where
              l.trt ="a"
          and r.trt ="b"
          and l.pt = r.pt
    ;quit;

    %arraydelete(_ix);

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /* WORK.SQLWANT total obs=2                                                                                               */
    /*                                                                                                                        */
    /*   PT     DENOM                                                                                                         */
    /*                                                                                                                        */
    /*   1     0.28571                                                                                                        */
    /*   2     0.14286                                                                                                        */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*____                    _
    |___ /   _ __   ___  __ _| |   __ _ _ __ _ __ __ _ _   _ ___
      |_ \  | `__| / __|/ _` | |  / _` | `__| `__/ _` | | | / __|
     ___) | | |    \__ \ (_| | | | (_| | |  | | | (_| | |_| \__ \
    |____/  |_|    |___/\__, |_|  \__,_|_|  |_|  \__,_|\__, |___/
                           |_|                         |___/
    */


    %array(_ix,values=1-8);

    * IF YOU WANT TO GENERATE THE NEEDED CODE TO COPY INTO R;

    %utlnopts;
    data _null_;
    %do_over(_ix,phrase=%str(put
       "( (l.v? != r.v?) and ((l.v? * r.v?) is not null) ) +"; ));
    run;quit;
    %utlopts;

    ( (l.v1 != r.v1) and ((l.v1 * r.v1) is not null) ) +
    ( (l.v2 != r.v2) and ((l.v2 * r.v2) is not null) ) +
    ( (l.v3 != r.v3) and ((l.v3 * r.v3) is not null) ) +
    ( (l.v4 != r.v4) and ((l.v4 * r.v4) is not null) ) +
    ( (l.v5 != r.v5) and ((l.v5 * r.v5) is not null) ) +
    ( (l.v6 != r.v6) and ((l.v6 * r.v6) is not null) ) +
    ( (l.v7 != r.v7) and ((l.v7 * r.v7) is not null) ) +
    ( (l.v8 != r.v8) and ((l.v8 * r.v8) is not null) ) +

    %utlnopts;
    data _null_;
    %do_over(_ix,phrase=%str(put
       "((l.v? * r.v?) is not null) +"; ));
    run;quit;
    %utlopts;

    ((l.v1 * r.v1) is not null) +
    ((l.v2 * r.v2) is not null) +
    ((l.v3 * r.v3) is not null) +
    ((l.v4 * r.v4) is not null) +
    ((l.v5 * r.v5) is not null) +
    ((l.v6 * r.v6) is not null) +
    ((l.v7 * r.v7) is not null) +
    ((l.v8 * r.v8) is not null) +

    * NOTE WE NEED TO MUTIPLE THE BOLEANS BY 1.0 to CONVERT TO FLOATS ;


    %utl_submit_r64x(resolve('
     library(sqldf);
     library(haven);
     source("c:/oto/fn_tosas9x.R");
     have<-read_sas("d:/sd1/have.sas7bdat");
     have;
     want<-sqldf("
      select
         l.pt
        ,1.0 * (%do_over(_ix,phrase=
           ( (l.v? != r.v?) and ((l.v? * r.v?) is not null))
           ,between=+))
          /
          (1.0 * (%do_over(_ix,phrase=
               ((l.v? * r.v?) is not null)
               ,between=+))) as denom
      from
          have as l inner join have as r
      on
              l.trt =`a`
          and r.trt =`b`
          and l.pt = r.pt
      ");
     want;
     fn_tosas9x(
           inp    = want
          ,outlib ="d:/sd1/"
          ,outdsn ="rwant"
          );
    '));

    proc print data=sd1.rwant;
    run;quit;

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /* STATTRANSFER adds ROWNAMES                                                                                             */
    /*                                                                                                                        */
    /*   ROWNAMES    PT     DENOM                                                                                             */
    /*                                                                                                                        */
    /*       1       1     0.28571                                                                                            */
    /*       2       2     0.14286                                                                                            */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*  _                 _   _                             _
    | || |    _ __  _   _| |_| |__   ___  _ __    ___  __ _| |   __ _ _ __ _ __ __ _ _   _ ___
    | || |_  | `_ \| | | | __| `_ \ / _ \| `_ \  / __|/ _` | |  / _` | `__| `__/ _` | | | / __|
    |__   _| | |_) | |_| | |_| | | | (_) | | | | \__ \ (_| | | | (_| | |  | | | (_| | |_| \__ \
       |_|   | .__/ \__, |\__|_| |_|\___/|_| |_| |___/\__, |_|  \__,_|_|  |_|  \__,_|\__, |___/
             |_|    |___/                                |_|                         |___/
    */

    libname tmp "c:/temp";
    proc datasets lib=tmp;
      delete pwant;
    run;quit;

    %array(_ix,values=1-8);

    %utl_submit_py64_310x(resolve('
     import pyperclip;
     import os;
     from os import path;
     import sys;
     import subprocess;
     import time;
     import pandas as pd;
     import pyreadstat as ps;
     import numpy as np;
     import pandas as pd;
     from pandasql import sqldf;
     mysql = lambda q: sqldf(q, globals());
     from pandasql import PandaSQL;
     pdsql = PandaSQL(persist=True);
     sqlite3conn = next(pdsql.conn.gen).connection.connection;
     sqlite3conn.enable_load_extension(True);
     sqlite3conn.load_extension("c:/temp/libsqlitefunctions.dll");
     mysql = lambda q: sqldf(q, globals());
     have, meta = ps.read_sas7bdat("d:/sd1/have.sas7bdat");
     exec(open("c:/temp/fn_tosas9.py").read());
     print(have);
     want = pdsql("""
      select
         l.pt
        ,1.0 * (%do_over(_ix,phrase=
           ( (l.v? != r.v?) and ((l.v? * r.v?) is not null))
           ,between=+))
          /
          (1.0 * (%do_over(_ix,phrase=
               ((l.v? * r.v?) is not null)
               ,between=+))) as denom
      from
          have as l inner join have as r
      on
              l.trt =`a`
          and r.trt =`b`
          and l.pt = r.pt
     """);
     print(want);
     fn_tosas9(
        want
        ,dfstr="pwant"
        ,timeest=3
        );
    '));

    libname tmp "c:/temp";
    proc print data=tmp.pwant;
    run;quit;

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /*  Obs    PT     DENOM                                                                                                   */
    /*                                                                                                                        */
    /*   1     1     0.28571                                                                                                  */
    /*   2     2     0.14286                                                                                                  */
    /*                                                                                                                        */
    /**************************************************************************************************************************/


    /*___                                 _
    | ___|   _ __   _ __   ___  ___  __ _| |
    |___ \  | `__| | `_ \ / _ \/ __|/ _` | |
     ___) | | |    | | | | (_) \__ \ (_| | |
    |____/  |_|    |_| |_|\___/|___/\__, |_|
                                       |_|
    */

    %utl_rbeginx;
    parmcards4;
    library(dplyr)
    library(tidyr)
    df <- structure(list(V1 = c("Sample1", "Sample1b", "Sample2", "Sample2b"
    ), V2 = c(0L, 1L, 2L, 2L), V3 = c(1L, 1L, 2L, 2L), V4 = c(1L,
    1L, 1L, 1L), V5 = c(2L, 2L, 0L, 1L), V6 = c(NA, 1L, 0L, 0L),
        V7 = c(2L, 1L, NA, NA), V8 = c(1L, 1L, 0L, 0L), V9 = c(1L,
        1L, 0L, 0L)), class = "data.frame", row.names = c(NA, -4L
    ))
    want<-df %>%
      pivot_longer(-V1) %>%
      group_by(grp = sub("(.*\\d+).*"
       ,"\\1", V1)
       ,name) %>%
      filter(!any(is.na(value))) %>%
      summarize(mismatch = value[1] != value[2]) %>%
      ungroup() %>%
      summarize(mismatch = paste0(sum(mismatch)
       , "/", n()), .by = grp)
    str(want)
    clp<-paste(want$mismatch,collapse='@')
    clp
    writeClipboard(clp)
    ;;;;
    %utl_rendx(return=clp);

    %put &=clp;
    %let clpfix=%left(%sca
    %put &=clpfix;

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /*  R                                                                                                                     */
    /*  =                                                                                                                     */
    /*  > clp                                                                                                                 */
    /*  [1] "2/7@1/7"                                                                                                         */
    /*                                                                                                                        */
    /*                                                                                                                        */
    /* SAS                                                                                                                    */
    /* ===                                                                                                                    */
    /* %put &=clp;                                                                                                            */
    /*                                                                                                                        */
    /* CLP=2/7@1/7                                                                                                            */
    /*                                                                                                                        */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*__                     _
     / /_    ___  __ _ ___  | |_ _ __ __ _ _ __  ___ _ __   ___  ___  ___
    | `_ \  / __|/ _` / __| | __| `__/ _` | `_ \/ __| `_ \ / _ \/ __|/ _ \
    | (_) | \__ \ (_| \__ \ | |_| | | (_| | | | \__ \ |_) | (_) \__ \  __/
     \___/  |___/\__,_|___/  \__|_|  \__,_|_| |_|___/ .__/ \___/|___/\___|
                                                    |_|
    */

    proc transpose data=sd1.have out=havxpo;
    by pt;
    id trt;
    run;quit;

    proc sql;
      create
         table swant as
      select
         pt
        ,sum(a ne b and not missing (a*b)) / sum(not missing (a*b))
      from
         havxpo
      group
         by pt
    ;quit;


    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /* WORK.SWANT total obs=2                                                                                                 */
    /*                                                                                                                        */
    /*  Obs    PT    _TEMA005                                                                                                 */
    /*                                                                                                                        */
    /*   1     1      0.28571                                                                                                 */
    /*   2     2      0.14286                                                                                                 */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*____                                                    _
    |___  |  ___  __ _ ___   _ __  _   _ _ __ ___   ___  __ _| | __  ___ __   ___  ___
       / /  / __|/ _` / __| | `_ \| | | | `__/ _ \ / __|/ _` | | \ \/ / `_ \ / _ \/ __|
      / /   \__ \ (_| \__ \ | |_) | |_| | | |  __/ \__ \ (_| | |  >  <| |_) | (_) \__ \
     /_/    |___/\__,_|___/ | .__/ \__,_|_|  \___| |___/\__, |_| /_/\_\ .__/ \___/|___/
                            |_|                            |_|        |_|
    */

    %array(_ix,values=1-8);

    proc sql;
      create
         table sqlxpo as
      select
         pt
        ,sum(numer)/sum(denom) as frac
      from (
         %do_over(_ix,phrase=%str(
            select
               l.pt
              ,sum(not missing (l.v?*r.v?))                    as denom
              ,sum(l.v? ne r.v? and not missing (l.v? * r.v?)) as numer
           from
               sd1.have as l, sd1.have as r
           where
                   l.trt ='a' and r.trt='b'
               and l.pt  = r.pt
           group
               by l.pt)
          ,between=union all))
      group by pt

    ;quit;

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /* WORK.SQLXPO total obs=2                                                                                                */
    /*                                                                                                                        */
    /* bs    PT      FRAC                                                                                                     */
    /*                                                                                                                        */
    /* 1     1     0.28571                                                                                                    */
    /* 2     2     0.14286                                                                                                    */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*___
     ( _ )   ___  __ _ ___    __ _ _ __ _ __ __ _ _   _
     / _ \  / __|/ _` / __|  / _` | `__| `__/ _` | | | |
    | (_) | \__ \ (_| \__ \ | (_| | |  | | | (_| | |_| |
     \___/  |___/\__,_|___/  \__,_|_|  |_|  \__,_|\__, |
                                                  |___/
    */

    data numary;
      array xy %utl_numary(sd1.have,drop=pt trt);
    run;quit;


    /*___           _
     ( _ )   _ __  | | ___   ___  _ __     __ _ _ __ _ __ __ _ _   _
     / _ \  | `__| | |/ _ \ / _ \| `_ \   / _` | `__| `__/ _` | | | |
    | (_) | | |    | | (_) | (_) | |_) | | (_| | |  | | | (_| | |_| |
     \___/  |_|    |_|\___/ \___/| .__/   \__,_|_|  |_|  \__,_|\__, |
              _ _   _            |_|  _     _                  |___/
    __      _(_) |_| |__   ___  _   _| |_  | | ___   ___  _ __
    \ \ /\ / / | __| `_ \ / _ \| | | | __| | |/ _ \ / _ \| `_ \
     \ V  V /| | |_| | | | (_) | |_| | |_  | | (_) | (_) | |_) |
      \_/\_/ |_|\__|_| |_|\___/ \__,_|\__| |_|\___/ \___/| .__/
                                                         |_|
    */

    * this heps you figure out the looping;

    %utl_rbeginx;
    parmcards4;
     library(sqldf)
     library(haven)
     source("c:/oto/fn_tosas9x.R")
     have<-read_sas("d:/sd1/have.sas7bdat")[,3:10]
     have
     mismat<-data.frame()
       mismat[1,1] = 1.0*(!is.na(have[1,1] * have[2,1]))
       mismat[2,1] = 1.0*(!is.na(have[1,2] * have[2,2]))
       mismat[3,1] = 1.0*(!is.na(have[1,3] * have[2,3]))
       mismat[4,1] = 1.0*(!is.na(have[1,4] * have[2,4]))
       mismat[5,1] = 1.0*(!is.na(have[1,5] * have[2,5]))
       mismat[6,1] = 1.0*(!is.na(have[1,6] * have[2,6]))
       mismat[7,1] = 1.0*(!is.na(have[1,7] * have[2,7]))
       mismat[8,1] = 1.0*(!is.na(have[1,8] * have[2,8]))

       mismat[1,2] = 1.0*(!is.na(have[3,1] * have[4,1]))
       mismat[2,2] = 1.0*(!is.na(have[3,2] * have[4,2]))
       mismat[3,2] = 1.0*(!is.na(have[3,3] * have[4,3]))
       mismat[4,2] = 1.0*(!is.na(have[3,4] * have[4,4]))
       mismat[5,2] = 1.0*(!is.na(have[3,5] * have[4,5]))
       mismat[6,2] = 1.0*(!is.na(have[3,6] * have[4,6]))
       mismat[7,2] = 1.0*(!is.na(have[3,7] * have[4,7]))
       mismat[8,2] = 1.0*(!is.na(have[3,8] * have[4,8]))

       mismat[1,3] = mismat[1,1] * 1.0*(have[1,1] != have[2,1])
       mismat[2,3] = mismat[2,1] * 1.0*(have[1,2] != have[2,2])
       mismat[3,3] = mismat[3,1] * 1.0*(have[1,3] != have[2,3])
       mismat[4,3] = mismat[4,1] * 1.0*(have[1,4] != have[2,4])
       mismat[5,3] = mismat[5,1] * 1.0*(have[1,5] != have[2,5])
       mismat[6,3] = mismat[6,1] * 1.0*(have[1,6] != have[2,6])
       mismat[7,3] = mismat[7,1] * 1.0*(have[1,7] != have[2,7])
       mismat[8,3] = mismat[8,1] * 1.0*(have[1,8] != have[2,8])

       mismat[1,4] = mismat[1,2] * 1.0*(have[3,1] != have[4,1])
       mismat[2,4] = mismat[2,2] * 1.0*(have[3,2] != have[4,2])
       mismat[3,4] = mismat[3,2] * 1.0*(have[3,3] != have[4,3])
       mismat[4,4] = mismat[4,2] * 1.0*(have[3,4] != have[4,4])
       mismat[5,4] = mismat[5,2] * 1.0*(have[3,5] != have[4,5])
       mismat[6,4] = mismat[6,2] * 1.0*(have[3,6] != have[4,6])
       mismat[7,4] = mismat[7,2] * 1.0*(have[3,7] != have[4,7])
       mismat[8,4] = mismat[8,2] * 1.0*(have[3,8] != have[4,8])

       sum(mismat[,1])
       sum(mismat[,2])
       sum(mismat[,3],na.rm = TRUE)
       sum(mismat[,4],na.rm = TRUE)

       pat1<-sum(mismat[,3],na.rm = TRUE)/sum(mismat[,1])
       pat2<-sum(mismat[,4],na.rm = TRUE)/sum(mismat[,2])

       pat1
       pat2

       clp<-paste(pat1,pat2,collapse='@')
       clp
       writeClipboard(clp)
    ;;;;
    %utl_rendx;

    /*        _ _   _       _
    __      _(_) |_| |__   | | ___   ___  _ __
    \ \ /\ / / | __| `_ \  | |/ _ \ / _ \| `_ \
     \ V  V /| | |_| | | | | | (_) | (_) | |_) |
      \_/\_/ |_|\__|_| |_| |_|\___/ \___/| .__/
                                         |_|
    */

    * this could be simplified using the sas technique in 9;

    %utl_rbeginx;
    parmcards4;
     library(haven)
     source("c:/oto/fn_tosas9x.R")
     have<-read_sas("d:/sd1/have.sas7bdat")[,3:10]
     have
     mismat<-data.frame(top1=double(),bot1=double(),top2=double(),bot2=double())
     for ( b in seq(1,8,1) ) {
       mismat[b,1] = 1.0*(!is.na(have[1,b] * have[2L,b]))
       mismat[b,2] = 1.0*(!is.na(have[3,b] * have[3L,b]))
       mismat[b,3] = mismat[b,1] * 1.0*(have[1,b] != have[2,b])
       mismat[b,4] = mismat[b,3] * 1.0*(have[3,b] != have[4,b])
       }
     pat1<-sum(mismat[,3],na.rm = TRUE)/sum(mismat[,1])
     pat2<-sum(mismat[,4],na.rm = TRUE)/sum(mismat[,2])

     clp<-paste(pat1,pat2,collapse='@')
     clp
       writeClipboard(clp)
    ;;;;
    %utl_rendx;

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /*  R                                                                                                                     */
    /* > clp
    /* [1] "0.285714285714286 0.142857142857143"
    /*
    /*                                                                                                                        */
    /*                                                                                                                        */
    /* SAS                                                                                                                    */
    /* ===                                                                                                                    */
    /* %put &=clp;                                                                                                            */
    /*                                                                                                                        */
    /* CLP=2/7@1/7                                                                                                            */
    /*                                                                                                                        */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*___                    _
     / _ \   ___  __ _ ___  | | ___   ___  _ __     __ _ _ __ _ __ __ _ _   _
    | (_) | / __|/ _` / __| | |/ _ \ / _ \| `_ \   / _` | `__| `__/ _` | | | |
     \__, | \__ \ (_| \__ \ | | (_) | (_) | |_) | | (_| | |  | | | (_| | |_| |
       /_/  |___/\__,_|___/ |_|\___/ \___/| .__/   \__,_|_|  |_|  \__,_|\__, |
                                          |_|                           |___/
    */

    data sasloop;

     array mismat %utl_numary(sd1.have,drop=pt trt);

     botPt1=0;
     botPt1=0;
     topPt1=0;
     topPt2=0;

     do b = 1 to 8;
       topPt1 = sum(topPt1 , (not missing(mismat[1,b])*(not missing(mismat[2,b]))) * (mismat[1,b] ne mismat[2,b]));
       botPt1 = sum(botPt1 , (not missing(mismat[1,b] * mismat[2,b])));
       topPt2 = sum(topPt2 , (not missing(mismat[1,b])*(not missing(mismat[3,b]))) * (mismat[3,b] ne mismat[4,b]));
       botPt2 = sum(botPt2 , (not missing(mismat[3,b] * mismat[4,b])));
     end;

     rat1 = topPt1/botPt1;
     rat2 = topPt2/botPt2;

     put rat1= / rat2=;

    run;quit;


    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /* RAT1=0.2857142857                                                                                                      */
    /* RAT2=0.1428571429                                                                                                      */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */
