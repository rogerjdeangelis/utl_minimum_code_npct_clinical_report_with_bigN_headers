# utl_minimum_code_npct_clinical_report_with_bigN_headers
Minimum code npct clinical report with bigN headers.  Keywords: sas sql join merge big data analytics macros oracle teradata mysql sas communities stackoverflow statistics artificial inteligence AI Python R Java Javascript WPS Matlab SPSS Scala Perl C C# Excel MS Access JSON graphics maps NLP natural language processing machine learning igraph DOSUBL DOW loop stackoverflow SAS community.
    Minimum code for Npct clinical report with bigN headers

    github
    https://tinyurl.com/y9jcwqoh
    https://github.com/rogerjdeangelis/utl_minimum_code_npct_clinical_report_with_bigN_headers

    https://tinyurl.com/ybf6c8pu
    https://communities.sas.com/t5/ODS-and-Base-Reporting/Proc-Report-Column-Header-using-variable/m-p/487910

    for more flexible solutuons see (or search on 'report' in github)
    https://github.com/rogerjdeangelis/utl_flexible_proc_report
    https://github.com/rogerjdeangelis/utl_clinical_report
    https://github.com/rogerjdeangelis/utl_crosstab_dataset_3_lines_of_code


    OUTPUT
    ====
                        MINIMAL CODE CLINICAL REPORT

                                    Aspirin       Placebo
      MAJOR        MINOR            (N= 17 )      (N= 15 )

      AGEGROUP     18-39            9(64.3%)      5(35.7%)
                   40-64            1(33.3%)      2(66.7%)
                   40-65            2(66.7%)      1(33.3%)
                   40-66            1(33.3%)      2(66.7%)
                   65+              4(44.4%)      5(55.6%)

      ETHNICITY    Hispanic         5(29.4%)      12(70.6%)
                   Non-Hispanic     12(80.0%)      3(20.0%)

      GENDER       F                13(61.9%)     8(38.1%)
                   M                4(36.4%)      7(63.6%)

      RACE         Black            5(55.6%)      4(44.4%)
                   Multi-Race       3(100.0%)     0(0.0%)
                   White            9(45.0%)      11(55.0%)
     INPUT
    ==========

    data have;
       retain trt;
       informat Race Gender Ethnicity AgeGroup $24.;
       input patient_ID Race Gender Ethnicity AgeGroup Followup_Flag Vaccine_Flag;
       if Vaccine_Flag=0 then trt='Aspirin';else trt='Placebo';
       drop vaccine_flag;
    cards4;
    1 White F Hispanic 18-39 1 1
    2 White M Non-Hispanic 18-39 0 0
    3 Black F Hispanic 40-64 0 0
    4 White M Hispanic 65+ 1 1
    5 Multi-Race F Non-Hispanic 18-39 0 0
    6 White M Hispanic 18-39 1 1
    7 White F Non-Hispanic 40-65 0 1
    8 White F Hispanic 65+ 1 1
    9 Black M Non-Hispanic 18-39 0 1
    10 White F Hispanic 18-39 1 0
    11 White M Hispanic 40-66 1 0
    12 Black F Non-Hispanic 65+ 0 1
    13 Black F Hispanic 40-64 0 1
    14 White F Hispanic 65+ 1 1
    15 Multi-Race F Non-Hispanic 18-39 0 0
    16 White M Hispanic 18-39 1 1
    17 White F Non-Hispanic 40-65 0 0
    18 White F Non-Hispanic 65+ 1 0
    19 Black M Non-Hispanic 18-39 0 0
    20 White F Hispanic 18-39 0 0
    21 White M Hispanic 40-66 1 1
    22 Black F Non-Hispanic 65+ 0 0
    23 Black F Hispanic 40-64 0 1
    24 White F Hispanic 65+ 1 1
    25 Multi-Race F Non-Hispanic 18-39 0 0
    26 White M Hispanic 18-39 1 1
    27 White F Non-Hispanic 40-65 0 0
    28 White F Non-Hispanic 65+ 1 0
    29 Black M Non-Hispanic 18-39 0 0
    30 White F Hispanic 18-39 0 0
    31 White M Hispanic 40-66 1 1
    32 Black F Non-Hispanic 65+ 0 0
    ;;;;
    run;quit;


    ALL THE CODE
    ============

      * Normalize ie make long and skinny;
      
      proc transpose data=have out=havxpo;
      by patient_id trt ;
      var Race Gender Ethnicity AgeGroup;
      run;quit;

      * create big N header macro variables witht he same names as proc report treatement names;
      proc sql;
        select resolve(catx(" ",'%Let',trt,'=',trt,'#(N=',Put(Count(patient_id),4.),');'))
             from have  Group by trt
      ;quit;

      * cretae major and minor categories for report;
      data havadd;
       set havxpo;
       mjrMnr=catx('@',_name_,col1);
      run;quit;

      * transpose so treament columns are across;
      ods exclude all;
      ods output observed      =xpocnt(drop=sum);
      ods output rowprofilespct=xporowpct;
      proc corresp data=havadd all dimens=1 print=both;
       tables   mjrMnr, trt;
      run;quit;
      ods select all;

      * merge percent and counts so that we have N(PCT) for each treament;
      data havNpc(where=(label ne 'Sum'));
        length Major Minor label  Aspirin Placebo $32 ;
        merge xpocnt   (rename=(Placebo=Placebocnt Aspirin=Aspirincnt))
              xpoRowPct(rename=(Placebo=Placebopct Aspirin=Aspirinpct));
        Aspirin     = cats(put(Aspirincnt,4.),'(',put(Aspirinpct,5.1),'%)');
        Placebo     = cats(put(Placebocnt,4.),'(',put(Placebopct,5.1),'%)');
        Major=scan(label,1,'@');
        Minor=scan(label,2,'@');
        keep Major Minor label Aspirin Placebo;
      run;quit;

      *just a dumb proc report - note we have  table configured like the report - tabulate doe not provide a report;
      ( anso notice the macro vatiables with bigNs;
      proc report data=havNpc nowd split='#';
        title " MINIMALL CODE CLINICAL REPORT";
        cols Major Minor Aspirin Placebo;
        define major / order;
        define minor / order;
        define aspirin / "&aspirin";
        define placebo /"&placebo";
      run;quit;

    
