# utl-within-a-datastep-call-r-and-create-a-dataset-and-modify-that-dataset-with-sas-defer
Within a datastep call r and create a dataset and modify that dataset with sas defer
    %let pgm=utl-within-a-datastep-call-r-and-create-a-dataset-and-modify-that-dataset-with-sas-defer;

    Within a datastep call r and create a dataset and modify that dataset with sas defer
    github                                                                                                                        
    https://tinyurl.com/22y8rhw4                                                                                                  
    https://github.com/rogerjdeangelis/utl-within-a-datastep-call-r-and-create-a-dataset-and-modify-that-dataset-with-sas-defer   

    Within a single datastep call R and create a regression sas dataset
    then read the r created dataset, record by record, and use sas
    code to add a residual column.

    The key to this solution is deferring the opening a second sas dataset until
    R has created the second dataset. All in one datastep.

                                                Defered dataset
                                             ========================
       set sd1.class(obs=1 keep=name in=opn) tmp.want(in=r) open=defer

       Like a sas hash you need to define all the variables before execution.
       Note I added the variable predict to the first sas dataset because has
       to build the full PDV before execution.
       Before execution SAS has to know about all the variables needed
       in the final output dataset.

       However if the deferred file is a v5 transport you do not
       need to have all variables in the first dataset?
       You can dynamically add variables when using a xport file.
       Meeds more testing?


       FOUR SOLUTIONS

            1. r deferred v5 xpt ile
            2. r deferred sas dataset (uses stattransfer)
            3. python deferred v5 xpt
            4. python deferred sas dataset (uses stattransfer)

    macros
    https://tinyurl.com/y9nfugth
    https://github.com/rogerjdeangelis/utl-macros-used-in-many-of-rogerjdeangelis-repositories

    /*                   _
    (_)_ __  _ __  _   _| |_
    | | `_ \| `_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    */

    options validvarname=upcase;
    libname sd1 "d:/sd1";
    data sd1.class;
     set sashelp.class(keep=height weight);
     length predict 8;
    run;quit;

    /*              _       __                   _         ____               _
    / |  _ __    __| | ___ / _| ___ _ __ ___  __| | __   _| ___|  __  ___ __ | |_
    | | | `__|  / _` |/ _ \ |_ / _ \ `__/ _ \/ _` | \ \ / /___ \  \ \/ / `_ \| __|
    | | | |    | (_| |  __/  _|  __/ | |  __/ (_| |  \ V / ___) |  >  <| |_) | |_
    |_| |_|     \__,_|\___|_|  \___|_|  \___|\__,_|   \_/ |____/  /_/\_\ .__/ \__|
                                                                       |_|
    */

    /**************************************************************************************************************************/
    /*                            |                                                   |                                       */
    /*       INPUT                |            PROCESS                                |            OUTPUT                     */
    /*                            |                                                   |                                       */
    /* SD1.CLASS total obs=19     |  proc datasets lib=work mt=data                   |                FROM R     FROM SAS    */
    /*                            |    mt=view nolist nodetails;                      | HEIGHT WEIGHT  PREDICT    RESIDUAL    */
    /* Obs HEIGHT WEIGHT  PREDICT |    delete classfit;                               |                                       */
    /*                            |  run;quit;                                        |  69.0   112.5  126.006    -13.5062    */
    /*   1  51.3    50.5     .    |                                                   |  56.5    84.0   77.268      6.7317    */
    /*   2  56.3    77.0     .    |  %utlfkil("d:/xpt/want.xpt");                     |  65.3    98.0  111.580    -13.5798    */
    /*   3  56.5    84.0     .    |                                                   |  62.8   102.5  101.832      0.6678    */
    /*   4  57.3    83.0     .    |  data classfit;                                   |  63.5   102.5  104.562     -2.0615    */
    /*   5  57.5    85.0     .    |                                                   |  57.3    83.0   80.388      2.6125    */
    /*   6  59.0    99.5     .    |   set  sd1.class(obs=1 in=opn)                    |  59.8    84.5   90.135     -5.6351    */
    /*   7  59.8    84.5     .    |        xpt.want (in=r) open=defer;                |  62.5   112.5  100.662     11.8375    */
    /*   8  62.5   112.5     .    |                                                   |  62.5    84.0  100.662    -16.6625    */
    /*   9  62.5    84.0     .    |   * execute r ehile reading sd1.class;            |  59.0    99.5   87.016     12.4841    */
    /*  10  62.8   102.5     .    |   if _n_=1 then do;                               |  51.3    50.5   56.993     -6.4933    */
    /*  11  63.5   102.5     .    |     rc=dosubl('                                   |  64.3    90.0  107.681    -17.6807    */
    /*  12  64.3    90.0     .    |      %utl_submit_r64x("                           |  56.3    77.0   76.488      0.5115    */
    /*  13  64.8   128.0     .    |        library(haven);                            |  66.5   112.0  116.259     -4.2586    */
    /*  14  65.3    98.0     .    |        library(SASxport);                         |  72.0   150.0  137.703     12.2967    */
    /*  15  66.5   112.0     .    |        source(`c:/temp/fn_tosas9.R`);             |  64.8   128.0  109.630     18.3698    */
    /*  16  66.5   112.0     .    |        want <- read_sas(`d:/sd1/clas.sas7bdat`);  |  67.0   133.0  118.208     14.7919    */
    /*  17  67.0   133.0     .    |        modl = lm(WEIGHT ~ HEIGHT, data=want);     |  57.5    85.0   81.167      3.8327    */
    /*  18  69.0   112.5     .    |        str(modl);                                 |  66.5   112.0  116.259     -4.2586    */
    /*  19  72.0   150.0     .    |        want$PREDICT<-modl$fitted.values;          |                                       */
    /*                            |        want;                                      |                                       */
    /*                            |        write.xport(want,file=`d:/xpt/want.xpt`);  |                                       */
    /*                            |      ");                                          |                                       */
    /*                            |      ');                                          |                                       */
    /*                            |                                                   |                                       */
    /*                            |   end;                                            |                                       */
    /*                            |                                                   |                                       */
    /*                            |   * add residual column using new tmp.want;       |                                       */
    /*                            |   * tmp.want opened and available;                |                                       */
    /*                            |   if r then do;                                   |                                       */
    /*                            |       residual=weight-predict;                    |                                       */
    /*                            |       output;                                     |                                       */
    /*                            |   end;                                            |                                       */
    /*                            |                                                   |                                       */
    /*                            |   drop rc;                                        |                                       */
    /*                            |                                                   |                                       */
    /*                            |  run;quit;                                        |                                       */
    /*                            |                                                   |                                       */
    /*                            |  proc print data=classfit;                        |                                       */
    /*                            |  run;quit;                                        |                                       */
    /*                            |                                                   |                                       */
    /**************************************************************************************************************************/

    /*___               _       __                             _        _     _
    |___ \   _ __    __| | ___ / _| ___ _ __   ___  __ _ ___  | |_ __ _| |__ | | ___
      __) | | `__|  / _` |/ _ \ |_ / _ \ `__| / __|/ _` / __| | __/ _` | `_ \| |/ _ \
     / __/  | |    | (_| |  __/  _|  __/ |    \__ \ (_| \__ \ | || (_| | |_) | |  __/
    |_____| |_|     \__,_|\___|_|  \___|_|    |___/\__,_|___/  \__\__,_|_.__/|_|\___|
    */

    /**************************************************************************************************************************/
    /*                            |                                                |                                          */
    /*       INPUT                |              PROCESS                           |                OUTP                      */
    /*                            |                                                |                                          */
    /* SD1.CLASS total obs=19     | %utlfkil(c:/temp/want.sas7bdat);               |                FROM R     FROM SAS       */
    /*                            |                                                | HEIGHT WEIGHT  PREDICT    RESIDUAL       */
    /* Obs HEIGHT WEIGHT  PREDICT | proc datasets lib=work mt=data                 |                                          */
    /*                            |   mt=view nolist nodetails;                    |  69.0   112.5  126.006    -13.5062       */
    /*   1  51.3    50.5     .    |   delete classfit;                             |  56.5    84.0   77.268      6.7317       */
    /*   2  56.3    77.0     .    | run;quit;                                      |  65.3    98.0  111.580    -13.5798       */
    /*   3  56.5    84.0     .    |                                                |  62.8   102.5  101.832      0.6678       */
    /*   4  57.3    83.0     .    | data classfit;                                 |  63.5   102.5  104.562     -2.0615       */
    /*   5  57.5    85.0     .    |  set                                           |  57.3    83.0   80.388      2.6125       */
    /*   6  59.0    99.5     .    |    sd1.class(obs=1 in=opn)                     |  59.8    84.5   90.135     -5.6351       */
    /*   7  59.8    84.5     .    |    tmp.want (in=r) open=defer;                 |  62.5   112.5  100.662     11.8375       */
    /*   8  62.5   112.5     .    |                                                |  62.5    84.0  100.662    -16.6625       */
    /*   9  62.5    84.0     .    |  * execute r while reading sd1.class;          |  59.0    99.5   87.016     12.4841       */
    /*  10  62.8   102.5     .    |  if _n_=1 then do;                             |  51.3    50.5   56.993     -6.4933       */
    /*  11  63.5   102.5     .    |   rc=dosubl('                                  |  64.3    90.0  107.681    -17.6807       */
    /*  12  64.3    90.0     .    |    %utl_submit_r64x("                          |  56.3    77.0   76.488      0.5115       */
    /*  13  64.8   128.0     .    |      library(haven);                           |  66.5   112.0  116.259     -4.2586       */
    /*  14  65.3    98.0     .    |      source(`c:/temp/fn_tosas9.R`);            |  72.0   150.0  137.703     12.2967       */
    /*  15  66.5   112.0     .    |      want <-read_sas(`d:/sd1/clas.sas7bdat`);  |  64.8   128.0  109.630     18.3698       */
    /*  16  66.5   112.0     .    |      modl = lm(WEIGHT ~ HEIGHT, data=want);    |  67.0   133.0  118.208     14.7919       */
    /*  17  67.0   133.0     .    |      str(modl);                                |  57.5    85.0   81.167      3.8327       */
    /*  18  69.0   112.5     .    |      want$PREDICT<-modl$fitted.values;         |  66.5   112.0  116.259     -4.2586       */
    /*  19  72.0   150.0     .    |      want<-data.frame(want);                   |                                          */
    /*                            |      fn_tosas9(dataf=want);                    |                                          */
    /*                            |    ");                                         |                                          */
    /*                            |   ');                                          |                                          */
    /*                            |                                                |                                          */
    /*                            |  end;                                          |                                          */
    /*                            |                                                |                                          */
    /*                            |  * add residual column using new tmp.want;     |                                          */
    /*                            |  * tmp.want opened and available;              |                                          */
    /*                            |  if r then do;                                 |                                          */
    /*                            |      residual=weight-predict;                  |                                          */
    /*                            |      output;                                   |                                          */
    /*                            |  end;                                          |                                          */
    /*                            |                                                |                                          */
    /*                            |  drop rc;                                      |                                          */
    /*                            | run;quit;                                      |                                          */
    /*                            |                                                |                                          */
    /*                            | proc print data=classfit;                      |                                          */
    /*                            | run;quit;                                      |                                          */
    /*                            |                                                |                                          */
    /**************************************************************************************************************************/

    /*____                     _       __                   _         ____              _
    |___ /   _ __  _   _    __| | ___ / _| ___ _ __ ___  __| | __   _| ___| __  ___ __ | |_
      |_ \  | `_ \| | | |  / _` |/ _ \ |_ / _ \ `__/ _ \/ _` | \ \ / /___ \ \ \/ / `_ \| __|
     ___) | | |_) | |_| | | (_| |  __/  _|  __/ | |  __/ (_| |  \ V / ___) | >  <| |_) | |_
    |____/  | .__/ \__, |  \__,_|\___|_|  \___|_|  \___|\__,_|   \_/ |____/ /_/\_\ .__/ \__|
            |_|    |___/                                                         |_|
    */

    /**************************************************************************************************************************/
    /*                            |                                                    |                                      */
    /*       INPUT                |                 PROCESS                            |                OUTPUT                */
    /*                            |                                                    |                                      */
    /* SD1.CLASS total obs=19     | options validvarname=upcase;                       |                FROM R     FROM SAS   */
    /*                            |                                                    | HEIGHT WEIGHT  PREDICT    RESIDUAL   */
    /* Obs HEIGHT WEIGHT  PREDICT | libname sd1 "d:/sd1";                              |                                      */
    /*                            |                                                    |  69.0   112.5  126.006    -13.5062   */
    /*   1  51.3    50.5     .    | %utlfkil(d:/xpt/want.xpt);                         |  56.5    84.0   77.268      6.7317   */
    /*   2  56.3    77.0     .    |                                                    |  65.3    98.0  111.580    -13.5798   */
    /*   3  56.5    84.0     .    | libname xpy xport "d:/xpt/want.xpt";               |  62.8   102.5  101.832      0.6678   */
    /*   4  57.3    83.0     .    |                                                    |  63.5   102.5  104.562     -2.0615   */
    /*   5  57.5    85.0     .    | proc datasets lib=work mt=data                     |  57.3    83.0   80.388      2.6125   */
    /*   6  59.0    99.5     .    |   mt=view nolist nodetails;                        |  59.8    84.5   90.135     -5.6351   */
    /*   7  59.8    84.5     .    |   delete classfit;                                 |  62.5   112.5  100.662     11.8375   */
    /*   8  62.5   112.5     .    | run;quit;                                          |  62.5    84.0  100.662    -16.6625   */
    /*   9  62.5    84.0     .    |                                                    |  59.0    99.5   87.016     12.4841   */
    /*  10  62.8   102.5     .    | data classfit;                                     |  51.3    50.5   56.993     -6.4933   */
    /*  11  63.5   102.5     .    |                                                    |  64.3    90.0  107.681    -17.6807   */
    /*  12  64.3    90.0     .    |  set                                               |  56.3    77.0   76.488      0.5115   */
    /*  13  64.8   128.0     .    |       sd1.class(obs=1 in=opn)                      |  66.5   112.0  116.259     -4.2586   */
    /*  14  65.3    98.0     .    |       xpt.want (in=r) open=defer;                  |  72.0   150.0  137.703     12.2967   */
    /*  15  66.5   112.0     .    |                                                    |  64.8   128.0  109.630     18.3698   */
    /*  16  66.5   112.0     .    |  if _n_=1 then do;                                 |  67.0   133.0  118.208     14.7919   */
    /*  17  67.0   133.0     .    |     rc=dosubl('                                    |  57.5    85.0   81.167      3.8327   */
    /*  18  69.0   112.5     .    |                                                    |  66.5   112.0  116.259     -4.2586   */
    /*  19  72.0   150.0     .    | %utl_submit_py64_310x("                            |                                      */
    /*                            | import pandas as pd;                               |                                      */
    /*                            | import numpy as np;                                |                                      */
    /*                            | import statsmodels.api as sm;                      |                                      */
    /*                            | import xport;                                      |                                      */
    /*                            | import xport.v56;                                  |                                      */
    /*                            | import pyreadstat as ps;                           |                                      */
    /*                            | clas                                               |                                      */
    /*                            | ,meta =ps.read_sas7bdat(`d:/sd1/class.sas7bdat`);  |                                      */
    /*                            | print(clas);                                       |                                      */
    /*                            | clas[`Y`]=clas[`WEIGHT`];                          |                                      */
    /*                            | clas[`X`]=clas[`HEIGHT`];                          |                                      */
    /*                            | lm = sm.OLS.from_formula(`Y ~ X`, clas);           |                                      */
    /*                            | result = lm.fit();                                 |                                      */
    /*                            | predictions = result.predict(clas[`X`]);           |                                      */
    /*                            | print(predictions);                                |                                      */
    /*                            | clas[`PREDICT`]=predictions;                       |                                      */
    /*                            | clas[`WEIGHT`]=clas[`Y`];                          |                                      */
    /*                            | clas[`HEIGHT`]=clas[`X`];                          |                                      */
    /*                            | print(clas);                                       |                                      */
    /*                            | want = xport.Dataset(clas, name=`WANT`             |                                      */
    /*                            |  ,label=`Linear Regression`);                      |                                      */
    /*                            | with open(`d:/xpt/want.xpt`, `wb`) as f:;          |                                      */
    /*                            | .   xport.v56.dump(want, f);                       |                                      */
    /*                            | ");                                                |                                      */
    /*                            | ');                                                |                                      */
    /*                            |  end;                                              |                                      */
    /*                            |                                                    |                                      */
    /*                            |  * add residual column using new tmp.want;         |                                      */
    /*                            |  * tmp.want opened and available;                  |                                      */
    /*                            |  if r then do;                                     |                                      */
    /*                            |      residual=weight-predict;                      |                                      */
    /*                            |      output;                                       |                                      */
    /*                            |  end;                                              |                                      */
    /*                            |                                                    |                                      */
    /*                            |  drop rc;                                          |                                      */
    /*                            |                                                    |                                      */
    /*                            | run;quit;                                          |                                      */
    /*                            |                                                    |                                      */
    /*                            | proc print data=classfit;                          |                                      */
    /*                            | run;quit;                                          |                                      */
    /*                            |                                                    |                                      */
    /**************************************************************************************************************************/

    /*  _                         _       __             _        _     _
    | || |    ___  __ _ ___    __| | ___ / _| ___ _ __  | |_ __ _| |__ | | ___
    | || |_  / __|/ _` / __|  / _` |/ _ \ |_ / _ \ `__| | __/ _` | `_ \| |/ _ \
    |__   _| \__ \ (_| \__ \ | (_| |  __/  _|  __/ |    | || (_| | |_) | |  __/
       |_|   |___/\__,_|___/  \__,_|\___|_|  \___|_|     \__\__,_|_.__/|_|\___|

    */


    /**************************************************************************************************************************/
    /*                            | options validvarname=upcase;                     |                                        */
    /*       INPUT                |                                                  |                OUTPUT                  */
    /*                            | libname sd1 "d:/sd1";                            |                                        */
    /* SD1.CLASS total obs=19     | libname tmp "c:/temp";                           |                FROM R     FROM SAS     */
    /*                            |                                                  | HEIGHT WEIGHT  PREDICT    RESIDUAL     */
    /* Obs HEIGHT WEIGHT  PREDICT | %utlfkil(d:/xpt/want.xpt);                       |                                        */
    /*                            |                                                  |  69.0   112.5  126.006    -13.5062     */
    /*   1  51.3    50.5     .    | libname xpy xport "d:/xpt/want.xpt";             |  56.5    84.0   77.268      6.7317     */
    /*   2  56.3    77.0     .    |                                                  |  65.3    98.0  111.580    -13.5798     */
    /*   3  56.5    84.0     .    | proc datasets lib=work mt=data                   |  62.8   102.5  101.832      0.6678     */
    /*   4  57.3    83.0     .    |   mt=view nolist nodetails;                      |  63.5   102.5  104.562     -2.0615     */
    /*   5  57.5    85.0     .    |   delete classfit;                               |  57.3    83.0   80.388      2.6125     */
    /*   6  59.0    99.5     .    | run;quit;                                        |  59.8    84.5   90.135     -5.6351     */
    /*   7  59.8    84.5     .    |                                                  |  62.5   112.5  100.662     11.8375     */
    /*   8  62.5   112.5     .    | data classfit;                                   |  62.5    84.0  100.662    -16.6625     */
    /*   9  62.5    84.0     .    |                                                  |  59.0    99.5   87.016     12.4841     */
    /*  10  62.8   102.5     .    |  set                                             |  51.3    50.5   56.993     -6.4933     */
    /*  11  63.5   102.5     .    |       sd1.class(obs=1 in=opn)                    |  64.3    90.0  107.681    -17.6807     */
    /*  12  64.3    90.0     .    |       tmp.want (in=r) open=defer;                |  56.3    77.0   76.488      0.5115     */
    /*  13  64.8   128.0     .    |                                                  |  66.5   112.0  116.259     -4.2586     */
    /*  14  65.3    98.0     .    |  if _n_=1 then do;                               |  72.0   150.0  137.703     12.2967     */
    /*  15  66.5   112.0     .    |     rc=dosubl('                                  |  64.8   128.0  109.630     18.3698     */
    /*  16  66.5   112.0     .    |                                                  |  67.0   133.0  118.208     14.7919     */
    /*  17  67.0   133.0     .    | %utl_submit_py64_310x("                          |  57.5    85.0   81.167      3.8327     */
    /*  18  69.0   112.5     .    | import os;                                       |  66.5   112.0  116.259     -4.2586     */
    /*  19  72.0   150.0     .    | import sys;                                      |                                        */
    /*                            | import subprocess;                               |                                        */
    /*                            | import time;                                     |                                        */
    /*                            | import pandas as pd;                             |                                        */
    /*                            | import numpy as np;                              |                                        */
    /*                            | import statsmodels.api as sm;                    |                                        */
    /*                            | import pyreadstat as ps;                         |                                        */
    /*                            | exec(open(`c:/temp/fn_tosas9.py`).read());       |                                        */
    /*                            | clas                                             |                                        */
    /*                            | ,meta =ps.read_sas7bdat(`d:/sd1/class.sas7bdat`);|                                        */
    /*                            | print(clas);                                     |                                        */
    /*                            | clas[`Y`]=clas[`WEIGHT`];                        |                                        */
    /*                            | clas[`X`]=clas[`HEIGHT`];                        |                                        */
    /*                            | lm = sm.OLS.from_formula(`Y ~ X`, clas);         |                                        */
    /*                            | result = lm.fit();                               |                                        */
    /*                            | predictions = result.predict(clas[`X`]);         |                                        */
    /*                            | print(predictions);                              |                                        */
    /*                            | clas[`PREDICT`]=predictions;                     |                                        */
    /*                            | clas[`WEIGHT`]=clas[`Y`];                        |                                        */
    /*                            | clas[`HEIGHT`]=clas[`X`];                        |                                        */
    /*                            | want=clas;                                       |                                        */
    /*                            | fn_tosas9(                                       |                                        */
    /*                            |    want                                          |                                        */
    /*                            |    ,dfstr=`want`                                 |                                        */
    /*                            |    ,timeest=3                                    |                                        */
    /*                            |    );                                            |                                        */
    /*                            | ");                                              |                                        */
    /*                            | ');                                              |                                        */
    /*                            |  end;                                            |                                        */
    /*                            |                                                  |                                        */
    /*                            |  * add residual column using new tmp.want;       |                                        */
    /*                            |  * tmp.want opened and available;                |                                        */
    /*                            |  if r then do;                                   |                                        */
    /*                            |      residual=weight-predict;                    |                                        */
    /*                            |      output;                                     |                                        */
    /*                            |  end;                                            |                                        */
    /*                            |                                                  |                                        */
    /*                            |  drop rc;                                        |                                        */
    /*                            |                                                  |                                        */
    /*                            | run;quit;                                        |                                        */
    /*                            |                                                  |                                        */
    /*                            | proc print data=classfit;                        |                                        */
    /*                            | run;quit;                                        |                                        */
    /*                            |                                                  |                                        */
    /**************************************************************************************************************************/

    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */
