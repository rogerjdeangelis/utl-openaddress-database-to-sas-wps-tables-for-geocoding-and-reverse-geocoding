# utl-openaddress-database-to-sas-wps-tables-for-geocoding-and-reverse-geocoding
Openaddress database to tiny (&lt;1gb) sas/wps table for geocoding and reverse geocoding
    %let pgm=utl-openaddress-database-to-sas-wps-tables-for-geocoding-and-reverse-geocoding;

    %stop_submission;  * incase you accidentally submit the whole thing at once *;

    Openaddress database to tiny (<1gb) sas/wps table for geocoding and reverse geocoding

    gethub
    https://tinyurl.com/4a94kyh3
    https://github.com/rogerjdeangelis/utl-openaddress-database-to-sas-wps-tables-for-geocoding-and-reverse-geocoding

    For the augmented larger address table see
    github
    https://tinyurl.com/yck8yu68
    https://github.com/rogerjdeangelis/utl-given-a-list-of-messy-addresses-geocode-and-reverse-geocode-using-us-address-database

    Below is sas/wps code to convert the 7000+ US json files to one small(900mb) wps/sas table
    with over 120+ million zipcoded addresses.

    SOURCES
    =======

    https://batch.openaddresses.io/data#map=0.45/ 23.2/46.3
    https://bcdcspatial.blogspot.com/2012/09/normalize-to-usps-street-abbreviations.html
    sas/wps sashelp.zipcode


    GET FINAL OPENADDRESS TABLE
    ===========================

    Download, unzip and run the code below to create the 120+ million address table.
    https://1drv.ms/u/s!AovFHZtMPA-7gh8dZTtlhqnaYc4r?e=tmqBLu    954mb win10 pro 999,706,624 bytes

    data adr_010opnDtaFin(compress=char);

    infile "m:/adr/csv/adr_010opnstafin.csv" firstobs=2 delimiter=',' DSD lrecl=32767;

    informat
        LON         11.6
        LAT         11.6
        ADR         $96.
        ZIPCODE     $5.
        ZIPSTATE    $5.
        ;
    input
        LON
        LAT
        ADR
        ZIPCODE
        ZIPSTATE
    ;
    run;quit;

    USAGE
    =====

    Ones you compile all the macros all you have to run is this driver

    %opnfix(AK);
    %opnfix(AL);
    %opnfix(AR);
    ...
    %opnfix(WV);
    %opnfix(WY);
    %opnfix(TX);

    AND APPEND THE STATES
    =====================

    data opn.adr_010opnSta(compress=char);

    length
      FRO         $2.
      STREET      $96.
      ADR         $96.
      ZIPCODE     $5.
      ZIPSTATE    $5.
      FLG         $2.
    ;
    set
    OPN.ADR_010OPNAKFIX (in= AK1 )
    OPN.ADR_010OPNALFIX (in= AL1 )
    OPN.ADR_010OPNARFIX (in= AR1 )
    .....
    OPN.ADR_010OPNWIFIX (in= WI1 )
    OPN.ADR_010OPNWVFIX (in= WV1 )
    OPN.ADR_010OPNWYFIX (in= WY1 )
    ;

    adr=tranwrd(adr,' DRIVE ',' DR ');
    if length(strip(zipcode))=5;

    ;select;
    when ( AK1 ) fro= "AK" ;
    when ( AL1 ) fro= "AL" ;
    when ( AR1 ) fro= "AR" ;
    ....
    when ( WI1 ) fro= "WI" ;
    when ( WV1 ) fro= "WV" ;
    when ( WY1 ) fro= "WY" ;
    otherwise fro="ER";
    end;

    zipstate=coalescec(fro,zipstate);

    run;quit


    PROCESS
    =======

          You will need (change drive as needed)

              M:/adr
              M:/adr/zip  for the download
              M:/adr/csv  for the unzipped download


          1. download unzip
                https://batch.openaddresses.io/data#map=0.45/ 23.2/46.3
                download US Midwest
                         US West
                         US South
                         US Northeast
                create folder
                         M:/adr/json
                copy all the json files
                  you should have

                         M:/adr/json/ak  * you do not have to create the AK folder just copy from download *
                                     anchorage-addresses-county.jason
                                     haines-addresses-county.json
                                    ...
                                    /wy
                                     albany-addresses-county.json
                Note the json files have lattitude, longitude and address, however
                many states only have zipcode populated half the time

          2. Use wps/sas sashelp.zipcode  keep only zipcode, x, and y.

          3.
              Run code that create a long filename for Gorgia

               filename ga (
               "d:\adr\json\ga\appling-addresses-county.geojson"
               ,"d:\adr\json\ga\atkinson-addresses-county.geojson"
               ,"d:\adr\json\ga\bacon-addresses-county.geojson"
               ,"d:\adr\json\ga\bartow-addresses-county.geojson"
               ,"d:\adr\json\ga\ben_hill-addresses-county.geojson"
               ,"d:\adr\json\ga\berrien-addresses-county.geojson"
               ,"d:\adr\json\ga\bibb-addresses-county.geojson"
               ,"d:\adr\json\ga\bleckley-addresses-county.geojson"
               ,"d:\adr\json\ga\brantley-addresses-county.geojson"
               ,"d:\adr\json\ga\brooks-addresses-county.geojson"
               ,"d:\adr\json\ga\bryan-addresses-county.geojson"
               ,"d:\adr\json\ga\burke-addresses-county.geojson"
               ,"d:\adr\json\ga\calhoun-addresses-county.geojson"
               ,"d:\adr\json\ga\camden-addresses-county.geojson"
               ,"d:\adr\json\ga\candler-addresses-county.geojson"
               ,"d:\adr\json\ga\carroll-addresses-county.geojson"
                ...

          4. Concatenate all the Gorgia json files  into one Georgia file "%sysfunc(pathname(work))/ga,json"

          5. Convert the nested json to a simple linear jason that wps/sas can easily read.

          6. Convert the simple linear jason into a sas/wps dataset
             All of Georgia in now in one table

          7. Split Gorgia into addresses with and withou zipcode

          8. Assign zipcode to json addresses that are missing zipcode, by
             selecting the json zipcode closest sashelp zipcode based on latatudes and longitudes
             in both sashelp.zipcode and json.

          9. Clean and standardize

    */

    /*                   _
    (_)_ __  _ __  _   _| |_ ___
    | | `_ \| `_ \| | | | __/ __|
    | | | | | |_) | |_| | |_\__ \
    |_|_| |_| .__/ \__,_|\__|___/
            |_|

    https://batch.openaddresses.io/data#map=0.45/ 23.2/46.3
    https://bcdcspatial.blogspot.com/2012/09/normalize-to-usps-street-abbreviations.html

         _                     _        _       _     _
     ___(_)_ __   ___ ___   __| | ___  | | __ _| |_  | | ___  _ __
    |_  / | `_ \ / __/ _ \ / _` |/ _ \ | |/ _` | __| | |/ _ \| `_ \
     / /| | |_) | (_| (_) | (_| |  __/ | | (_| | |_  | | (_) | | | |
    /___|_| .__/ \___\___/ \__,_|\___| |_|\__,_|\__| |_|\___/|_| |_|
          |_|
    */

    /*----                                                                   ----*/
    /*---- https://1drv.ms/u/s!AovFHZtMPA-7gh6PgVepoPUV2_dI?e=shjVT9         ----*/
    /*---- wps/sas zipcode dataset                                           ----*/
    /*----                                                                   ----*/

    data adr_010UsPlaceZip;
       informat
           LONGITUDE 11.6
           LATITUDE 11.6
           POSTALCODE $5.
           ADMIN_CODE1 $2.
       ;
       infile 'm:/adr/csv/adr_010UsPlaceZip.csv' firstobs=2 delimiter=',' DSD lrecl=32767;

       input
          LONGITUDE
          LATITUDE
          POSTALCODE
          ADMIN_CODE1
       ;
    run;quit;

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /*  last 41 obs from ADR_010USPLACEZIP total obs=41,140                                                                   */
    /*                                                  ADMIN_                                                                */
    /*    Obs    LONGITUDE    LATITUDE    POSTALCODE    CODE1                                                                 */
    /*                                                                                                                        */
    /*  41100     -167.436     65.6638      99783         AK                                                                  */
    /*  41101     -163.261     64.8683      99784         AK                                                                  */
    /*  41102     -166.825     65.4594      99785         AK                                                                  */
    /*  41103     -157.868     67.0846      99786         AK                                                                  */
    /*  41104     -145.274     66.5647      99788         AK                                                                  */
    /*  41105     -151.126     70.2358      99789         AK                                                                  */
    /*  41106     -147.724     64.8430      99790         AK                                                                  */
    /*  41107     -157.428     70.4722      99791         AK                                                                  */
    /*  41108     -134.527     58.3565      99801         AK                                                                  */
    /*  41109     -134.427     58.3042      99802         AK                                                                  */
    /*  41110     -134.593     58.3740      99803         AK                                                                  */
    /*  41111     -134.158     58.4750      99811         AK                                                                  */
    /*                                                                                                                        */
    /**************************************************************************************************************************/
    /*                                                    _   _  ___
     _ __ ___   __ _  ___ _ __ ___  ___    __ _ _ __   __| | (_)/ (_)_ __   ___
    | `_ ` _ \ / _` |/ __| `__/ _ \/ __|  / _` | `_ \ / _` |   / /| | `_ \ / __|
    | | | | | | (_| | (__| | | (_) \__ \ | (_| | | | | (_| |  / /_| | | | | (__
    |_| |_| |_|\__,_|\___|_|  \___/|___/  \__,_|_| |_|\__,_| /_/(_)_|_| |_|\___|
                                                     _ _
     _ __ ___ _ __ ___   _____   _____   _   _ _ __ (_) |_ ___
    | `__/ _ \ `_ ` _ \ / _ \ \ / / _ \ | | | | `_ \| | __/ __|
    | | |  __/ | | | | | (_) \ V /  __/ | |_| | | | | | |_\__ \
    |_|  \___|_| |_| |_|\___/ \_/ \___|  \__,_|_| |_|_|\__|___/

    */

    %macro unit(unit);

      if index(adr," &unit ") then do;
         *put adr=;
         idx=index(adr," &unit " );
         nxt=scan(substr(adr,idx),2);
         *put nxt=;
         wrdlen=length(catx(" ","&unit" ,strip(nxt)));
        *put wrdlen=;
         substr(adr,idx+1,wrdlen)="";
         *put adr=;
         adr=compbl(adr);
         *put adr= // ;
         flg="UN";
      end;

    %mend unit;


    /*   _                  _               _              __  __ _
     ___| |_ __ _ _ __   __| | __ _ _ __ __| |  ___ _   _ / _|/ _(_)_  __
    / __| __/ _` | `_ \ / _` |/ _` | `__/ _` | / __| | | | |_| |_| \ \/ /
    \__ \ || (_| | | | | (_| | (_| | | | (_| | \__ \ |_| |  _|  _| |>  <
    |___/\__\__,_|_| |_|\__,_|\__,_|_|  \__,_| |___/\__,_|_| |_| |_/_/\_\

    */

    filename ft15f001 "%sysfunc(pathname(work))/adr_010sfx.sas";
    parmcards4;
    when (strip(sfx)='BOULEVARD') substr(adr,length(adr)-len) =' BLVD';
    when (strip(sfx)='HARBOR') substr(adr,length(adr)-len) =' HBR';
    when (strip(sfx)='TRPK') substr(adr,length(adr)-len) =' TPKE';
    when (strip(sfx)='FORGES') substr(adr,length(adr)-len) =' FRGS';
    when (strip(sfx)='BYPAS') substr(adr,length(adr)-len) =' BYP';
    when (strip(sfx)='VIADUCT') substr(adr,length(adr)-len) =' VIA';
    when (strip(sfx)='MNT') substr(adr,length(adr)-len) =' MT';
    when (strip(sfx)='LNDNG') substr(adr,length(adr)-len) =' LNDG';
    when (strip(sfx)='VILL') substr(adr,length(adr)-len) =' VLG';
    when (strip(sfx)='MILL') substr(adr,length(adr)-len) =' ML';
    when (strip(sfx)='CENTERS') substr(adr,length(adr)-len) =' CTRS';
    when (strip(sfx)='CNTER') substr(adr,length(adr)-len) =' CTR';
    when (strip(sfx)='HRBOR') substr(adr,length(adr)-len) =' HBR';
    when (strip(sfx)='TR') substr(adr,length(adr)-len) =' TRL';
    when (strip(sfx)='PASSAGE') substr(adr,length(adr)-len) =' PSGE';
    when (strip(sfx)='WALKS') substr(adr,length(adr)-len) =' WALK';
    when (strip(sfx)='CREST') substr(adr,length(adr)-len) =' CRST';
    when (strip(sfx)='MEADOWS') substr(adr,length(adr)-len) =' MDWS';
    when (strip(sfx)='FREEWY') substr(adr,length(adr)-len) =' FWY';
    when (strip(sfx)='GARDEN') substr(adr,length(adr)-len) =' GDN';
    when (strip(sfx)='BLUFFS') substr(adr,length(adr)-len) =' BLFS';
    when (strip(sfx)='TRK') substr(adr,length(adr)-len) =' TRAK';
    when (strip(sfx)='SQUARES') substr(adr,length(adr)-len) =' SQS';
    when (strip(sfx)='FRRY') substr(adr,length(adr)-len) =' FRY';
    when (strip(sfx)='DIV') substr(adr,length(adr)-len) =' DV';
    when (strip(sfx)='STRAVEN') substr(adr,length(adr)-len) =' STRA';
    when (strip(sfx)='CMP') substr(adr,length(adr)-len) =' CP';
    when (strip(sfx)='GRDNS') substr(adr,length(adr)-len) =' GDNS';
    when (strip(sfx)='VILLG') substr(adr,length(adr)-len) =' VLG';
    when (strip(sfx)='MEADOW') substr(adr,length(adr)-len) =' MDW';
    when (strip(sfx)='TRAILS') substr(adr,length(adr)-len) =' TRL';
    when (strip(sfx)='STREETS') substr(adr,length(adr)-len) =' STS';
    when (strip(sfx)='PRAIRIE') substr(adr,length(adr)-len) =' PR';
    when (strip(sfx)='CRESCENT') substr(adr,length(adr)-len) =' CRES';
    when (strip(sfx)='PORT') substr(adr,length(adr)-len) =' PRT';
    when (strip(sfx)='BLUF') substr(adr,length(adr)-len) =' BLF';
    when (strip(sfx)='AVNUE') substr(adr,length(adr)-len) =' AVE';
    when (strip(sfx)='LIGHTS') substr(adr,length(adr)-len) =' LGTS';
    when (strip(sfx)='HARBORS') substr(adr,length(adr)-len) =' HBRS';
    when (strip(sfx)='LODG') substr(adr,length(adr)-len) =' LDG';
    when (strip(sfx)='TRACKS') substr(adr,length(adr)-len) =' TRAK';
    when (strip(sfx)='PKWAY') substr(adr,length(adr)-len) =' PKWY';
    when (strip(sfx)='BOT') substr(adr,length(adr)-len) =' BTM';
    when (strip(sfx)='DRV') substr(adr,length(adr)-len) =' DR';
    when (strip(sfx)='DIVIDE') substr(adr,length(adr)-len) =' DV';
    when (strip(sfx)='FORDS') substr(adr,length(adr)-len) =' FRDS';
    when (strip(sfx)='AVENU') substr(adr,length(adr)-len) =' AVE';
    when (strip(sfx)='RIVR') substr(adr,length(adr)-len) =' RIV';
    when (strip(sfx)='GATEWAY') substr(adr,length(adr)-len) =' GTWY';
    when (strip(sfx)='STREAM') substr(adr,length(adr)-len) =' STRM';
    when (strip(sfx)='BAYOO') substr(adr,length(adr)-len) =' BYU';
    when (strip(sfx)='KNOLL') substr(adr,length(adr)-len) =' KNL';
    when (strip(sfx)='EXPRESSWAY') substr(adr,length(adr)-len) =' EXPY';
    when (strip(sfx)='SPRNG') substr(adr,length(adr)-len) =' SPG';
    when (strip(sfx)='FLAT') substr(adr,length(adr)-len) =' FLT';
    when (strip(sfx)='GRDEN') substr(adr,length(adr)-len) =' GDN';
    when (strip(sfx)='TRAIL') substr(adr,length(adr)-len) =' TRL';
    when (strip(sfx)='JCTNS') substr(adr,length(adr)-len) =' JCTS';
    when (strip(sfx)='TUNNEL') substr(adr,length(adr)-len) =' TUNL';
    when (strip(sfx)='GROVES') substr(adr,length(adr)-len) =' GRVS';
    when (strip(sfx)='VALLY') substr(adr,length(adr)-len) =' VLY';
    when (strip(sfx)='FERRY') substr(adr,length(adr)-len) =' FRY';
    when (strip(sfx)='PARKWAY') substr(adr,length(adr)-len) =' PKWY';
    when (strip(sfx)='RADIEL') substr(adr,length(adr)-len) =' RADL';
    when (strip(sfx)='STRVNUE') substr(adr,length(adr)-len) =' STRA';
    when (strip(sfx)='OVERPASS') substr(adr,length(adr)-len) =' OPAS';
    when (strip(sfx)='PLAZA') substr(adr,length(adr)-len) =' PLZ';
    when (strip(sfx)='ESTATE') substr(adr,length(adr)-len) =' EST';
    when (strip(sfx)='MNTN') substr(adr,length(adr)-len) =' MTN';
    when (strip(sfx)='LOCK') substr(adr,length(adr)-len) =' LCK';
    when (strip(sfx)='ORCHRD') substr(adr,length(adr)-len) =' ORCH';
    when (strip(sfx)='STRVN') substr(adr,length(adr)-len) =' STRA';
    when (strip(sfx)='LOCKS') substr(adr,length(adr)-len) =' LCKS';
    when (strip(sfx)='BEND') substr(adr,length(adr)-len) =' BND';
    when (strip(sfx)='JUNCTIONS') substr(adr,length(adr)-len) =' JCTS';
    when (strip(sfx)='MOUNTIN') substr(adr,length(adr)-len) =' MTN';
    when (strip(sfx)='BURGS') substr(adr,length(adr)-len) =' BGS';
    when (strip(sfx)='PINE') substr(adr,length(adr)-len) =' PNE';
    when (strip(sfx)='LDGE') substr(adr,length(adr)-len) =' LDG';
    when (strip(sfx)='CAUSWAY') substr(adr,length(adr)-len) =' CSWY';
    when (strip(sfx)='BEACH') substr(adr,length(adr)-len) =' BCH';
    when (strip(sfx)='MOTORWAY') substr(adr,length(adr)-len) =' MTWY';
    when (strip(sfx)='BLUFF') substr(adr,length(adr)-len) =' BLF';
    when (strip(sfx)='COURT') substr(adr,length(adr)-len) =' CT';
    when (strip(sfx)='GROV') substr(adr,length(adr)-len) =' GRV';
    when (strip(sfx)='SPRNGS') substr(adr,length(adr)-len) =' SPGS';
    when (strip(sfx)='OVL') substr(adr,length(adr)-len) =' OVAL';
    when (strip(sfx)='VILLAG') substr(adr,length(adr)-len) =' VLG';
    when (strip(sfx)='VDCT') substr(adr,length(adr)-len) =' VIA';
    when (strip(sfx)='NECK') substr(adr,length(adr)-len) =' NCK';
    when (strip(sfx)='ORCHARD') substr(adr,length(adr)-len) =' ORCH';
    when (strip(sfx)='LIGHT') substr(adr,length(adr)-len) =' LGT';
    when (strip(sfx)='SHORE') substr(adr,length(adr)-len) =' SHR';
    when (strip(sfx)='GREEN') substr(adr,length(adr)-len) =' GRN';
    when (strip(sfx)='ISLND') substr(adr,length(adr)-len) =' IS';
    when (strip(sfx)='TURNPIKE') substr(adr,length(adr)-len) =' TPKE';
    when (strip(sfx)='MISSION') substr(adr,length(adr)-len) =' MSN';
    when (strip(sfx)='SPNGS') substr(adr,length(adr)-len) =' SPGS';
    when (strip(sfx)='COURSE') substr(adr,length(adr)-len) =' CRSE';
    when (strip(sfx)='TRAFFICWAY') substr(adr,length(adr)-len) =' TRFY';
    when (strip(sfx)='TERRACE') substr(adr,length(adr)-len) =' TER';
    when (strip(sfx)='HWAY') substr(adr,length(adr)-len) =' HWY';
    when (strip(sfx)='AVENUE') substr(adr,length(adr)-len) =' AVE';
    when (strip(sfx)='GLEN') substr(adr,length(adr)-len) =' GLN';
    when (strip(sfx)='BOUL') substr(adr,length(adr)-len) =' BLVD';
    when (strip(sfx)='INLET') substr(adr,length(adr)-len) =' INLT';
    when (strip(sfx)='LA') substr(adr,length(adr)-len) =' LN';
    when (strip(sfx)='BROOK') substr(adr,length(adr)-len) =' BRK';
    when (strip(sfx)='SHOAR') substr(adr,length(adr)-len) =' SHR';
    when (strip(sfx)='BYPASS') substr(adr,length(adr)-len) =' BYP';
    when (strip(sfx)='MTIN') substr(adr,length(adr)-len) =' MTN';
    when (strip(sfx)='ALLY') substr(adr,length(adr)-len) =' ALY';
    when (strip(sfx)='FOREST') substr(adr,length(adr)-len) =' FRST';
    when (strip(sfx)='JUNCTION') substr(adr,length(adr)-len) =' JCT';
    when (strip(sfx)='VIEWS') substr(adr,length(adr)-len) =' VWS';
    when (strip(sfx)='WELLS') substr(adr,length(adr)-len) =' WLS';
    when (strip(sfx)='CEN') substr(adr,length(adr)-len) =' CTR';
    when (strip(sfx)='CRT') substr(adr,length(adr)-len) =' CT';
    when (strip(sfx)='CORNERS') substr(adr,length(adr)-len) =' CORS';
    when (strip(sfx)='FRWAY') substr(adr,length(adr)-len) =' FWY';
    when (strip(sfx)='PRARIE') substr(adr,length(adr)-len) =' PR';
    when (strip(sfx)='CROSSING') substr(adr,length(adr)-len) =' XING';
    when (strip(sfx)='EXTN') substr(adr,length(adr)-len) =' EXT';
    when (strip(sfx)='CLIFFS') substr(adr,length(adr)-len) =' CLFS';
    when (strip(sfx)='MANORS') substr(adr,length(adr)-len) =' MNRS';
    when (strip(sfx)='PORTS') substr(adr,length(adr)-len) =' PRTS';
    when (strip(sfx)='GATEWY') substr(adr,length(adr)-len) =' GTWY';
    when (strip(sfx)='SQUARE') substr(adr,length(adr)-len) =' SQ';
    when (strip(sfx)='HARB') substr(adr,length(adr)-len) =' HBR';
    when (strip(sfx)='LOOPS') substr(adr,length(adr)-len) =' LOOP';
    when (strip(sfx)='HILL') substr(adr,length(adr)-len) =' HL';
    when (strip(sfx)='HIGHWAY') substr(adr,length(adr)-len) =' HWY';
    when (strip(sfx)='BROOKS') substr(adr,length(adr)-len) =' BRKS';
    when (strip(sfx)='BRNCH') substr(adr,length(adr)-len) =' BR';
    when (strip(sfx)='AVEN') substr(adr,length(adr)-len) =' AVE';
    when (strip(sfx)='SHORES') substr(adr,length(adr)-len) =' SHRS';
    when (strip(sfx)='ROUTE') substr(adr,length(adr)-len) =' RTE';
    when (strip(sfx)='PLACE') substr(adr,length(adr)-len) =' PL';
    when (strip(sfx)='SUMIT') substr(adr,length(adr)-len) =' SMT';
    when (strip(sfx)='PINES') substr(adr,length(adr)-len) =' PNES';
    when (strip(sfx)='TRKS') substr(adr,length(adr)-len) =' TRAK';
    when (strip(sfx)='SHOAL') substr(adr,length(adr)-len) =' SHL';
    when (strip(sfx)='STRT') substr(adr,length(adr)-len) =' ST';
    when (strip(sfx)='FRWY') substr(adr,length(adr)-len) =' FWY';
    when (strip(sfx)='HEIGHTS') substr(adr,length(adr)-len) =' HTS';
    when (strip(sfx)='RANCHES') substr(adr,length(adr)-len) =' RNCH';
    when (strip(sfx)='BOULEVARD') substr(adr,length(adr)-len) =' BLVD';
    when (strip(sfx)='EXTNSN') substr(adr,length(adr)-len) =' EXT';
    when (strip(sfx)='HOLLOWS') substr(adr,length(adr)-len) =' HOLW';
    when (strip(sfx)='VSTA') substr(adr,length(adr)-len) =' VIS';
    when (strip(sfx)='PLAINS') substr(adr,length(adr)-len) =' PLNS';
    when (strip(sfx)='STATION') substr(adr,length(adr)-len) =' STA';
    when (strip(sfx)='CIRCL') substr(adr,length(adr)-len) =' CIR';
    when (strip(sfx)='MNTNS') substr(adr,length(adr)-len) =' MTNS';
    when (strip(sfx)='VILLAGES') substr(adr,length(adr)-len) =' VLGS';
    when (strip(sfx)='HAVEN') substr(adr,length(adr)-len) =' HVN';
    when (strip(sfx)='TURNPK') substr(adr,length(adr)-len) =' TPKE';
    when (strip(sfx)='EXPR') substr(adr,length(adr)-len) =' EXPY';
    when (strip(sfx)='STN') substr(adr,length(adr)-len) =' STA';
    when (strip(sfx)='EXPW') substr(adr,length(adr)-len) =' EXPY';
    when (strip(sfx)='STREET') substr(adr,length(adr)-len) =' ST';
    when (strip(sfx)='STR') substr(adr,length(adr)-len) =' ST';
    when (strip(sfx)='SPURS') substr(adr,length(adr)-len) =' SPUR';
    when (strip(sfx)='CRECENT') substr(adr,length(adr)-len) =' CRES';
    when (strip(sfx)='RAD') substr(adr,length(adr)-len) =' RADL';
    when (strip(sfx)='RANCH') substr(adr,length(adr)-len) =' RNCH';
    when (strip(sfx)='WELL') substr(adr,length(adr)-len) =' WL';
    when (strip(sfx)='SHOALS') substr(adr,length(adr)-len) =' SHLS';
    when (strip(sfx)='ALLEY') substr(adr,length(adr)-len) =' ALY';
    when (strip(sfx)='PLZA') substr(adr,length(adr)-len) =' PLZ';
    when (strip(sfx)='MEDOWS') substr(adr,length(adr)-len) =' MDWS';
    when (strip(sfx)='ALLEE') substr(adr,length(adr)-len) =' ALY';
    when (strip(sfx)='HAVN') substr(adr,length(adr)-len) =' HVN';
    when (strip(sfx)='PATHS') substr(adr,length(adr)-len) =' PATH';
    when (strip(sfx)='BYPA') substr(adr,length(adr)-len) =' BYP';
    when (strip(sfx)='MILLS') substr(adr,length(adr)-len) =' MLS';
    when (strip(sfx)='PARKS') substr(adr,length(adr)-len) =' PARK';
    when (strip(sfx)='BYPS') substr(adr,length(adr)-len) =' BYP';
    when (strip(sfx)='TUNNELS') substr(adr,length(adr)-len) =' TUNL';
    when (strip(sfx)='CLUB') substr(adr,length(adr)-len) =' CLB';
    when (strip(sfx)='SQRS') substr(adr,length(adr)-len) =' SQS';
    when (strip(sfx)='HLLW') substr(adr,length(adr)-len) =' HOLW';
    when (strip(sfx)='MANOR') substr(adr,length(adr)-len) =' MNR';
    when (strip(sfx)='CENTRE') substr(adr,length(adr)-len) =' CTR';
    when (strip(sfx)='TRACK') substr(adr,length(adr)-len) =' TRAK';
    when (strip(sfx)='HGTS') substr(adr,length(adr)-len) =' HTS';
    when (strip(sfx)='CRCLE') substr(adr,length(adr)-len) =' CIR';
    when (strip(sfx)='FALLS') substr(adr,length(adr)-len) =' FLS';
    when (strip(sfx)='LANDING') substr(adr,length(adr)-len) =' LNDG';
    when (strip(sfx)='PLAINES') substr(adr,length(adr)-len) =' PLNS';
    when (strip(sfx)='VIADCT') substr(adr,length(adr)-len) =' VIA';
    when (strip(sfx)='GROVE') substr(adr,length(adr)-len) =' GRV';
    when (strip(sfx)='CAMP') substr(adr,length(adr)-len) =' CP';
    when (strip(sfx)='TPK') substr(adr,length(adr)-len) =' TPKE';
    when (strip(sfx)='DRIVE') substr(adr,length(adr)-len) =' DR';
    when (strip(sfx)='FREEWAY') substr(adr,length(adr)-len) =' FWY';
    when (strip(sfx)='POINTS') substr(adr,length(adr)-len) =' PTS';
    when (strip(sfx)='EXP') substr(adr,length(adr)-len) =' EXPY';
    when (strip(sfx)='COURTS') substr(adr,length(adr)-len) =' CTS';
    when (strip(sfx)='PKY') substr(adr,length(adr)-len) =' PKWY';
    when (strip(sfx)='CORNER') substr(adr,length(adr)-len) =' COR';
    when (strip(sfx)='CRSSING') substr(adr,length(adr)-len) =' XING';
    when (strip(sfx)='UNIONS') substr(adr,length(adr)-len) =' UNS';
    when (strip(sfx)='LODGE') substr(adr,length(adr)-len) =' LDG';
    when (strip(sfx)='CIRCLE') substr(adr,length(adr)-len) =' CIR';
    when (strip(sfx)='BRIDGE') substr(adr,length(adr)-len) =' BRG';
    when (strip(sfx)='EXPRESS') substr(adr,length(adr)-len) =' EXPY';
    when (strip(sfx)='TUNLS') substr(adr,length(adr)-len) =' TUNL';
    when (strip(sfx)='KNOLLS') substr(adr,length(adr)-len) =' KNLS';
    when (strip(sfx)='GREENS') substr(adr,length(adr)-len) =' GRNS';
    when (strip(sfx)='TUNEL') substr(adr,length(adr)-len) =' TUNL';
    when (strip(sfx)='FIELDS') substr(adr,length(adr)-len) =' FLDS';
    when (strip(sfx)='COMMON') substr(adr,length(adr)-len) =' CMN';
    when (strip(sfx)='RIVER') substr(adr,length(adr)-len) =' RIV';
    when (strip(sfx)='VIEW') substr(adr,length(adr)-len) =' VW';
    when (strip(sfx)='CRSENT') substr(adr,length(adr)-len) =' CRES';
    when (strip(sfx)='RNCHS') substr(adr,length(adr)-len) =' RNCH';
    when (strip(sfx)='CRSCNT') substr(adr,length(adr)-len) =' CRES';
    when (strip(sfx)='RDGE') substr(adr,length(adr)-len) =' RDG';
    when (strip(sfx)='CAUSEWAY') substr(adr,length(adr)-len) =' CSWY';
    when (strip(sfx)='PARKWY') substr(adr,length(adr)-len) =' PKWY';
    when (strip(sfx)='JUNCTON') substr(adr,length(adr)-len) =' JCT';
    when (strip(sfx)='STATN') substr(adr,length(adr)-len) =' STA';
    when (strip(sfx)='GARDN') substr(adr,length(adr)-len) =' GDN';
    when (strip(sfx)='MNTAIN') substr(adr,length(adr)-len) =' MTN';
    when (strip(sfx)='CRSSNG') substr(adr,length(adr)-len) =' XING';
    when (strip(sfx)='RAPID') substr(adr,length(adr)-len) =' RPD';
    when (strip(sfx)='KEY') substr(adr,length(adr)-len) =' KY';
    when (strip(sfx)='WY') substr(adr,length(adr)-len) =' WAY';
    when (strip(sfx)='THROUGHWAY') substr(adr,length(adr)-len) =' TRWY';
    when (strip(sfx)='ESTATES') substr(adr,length(adr)-len) =' ESTS';
    when (strip(sfx)='CK') substr(adr,length(adr)-len) =' CRK';
    when (strip(sfx)='LOAF') substr(adr,length(adr)-len) =' LF';
    when (strip(sfx)='HOLLOW') substr(adr,length(adr)-len) =' HOLW';
    when (strip(sfx)='CANYON') substr(adr,length(adr)-len) =' CYN';
    when (strip(sfx)='VILLAGE') substr(adr,length(adr)-len) =' VLG';
    when (strip(sfx)='CR') substr(adr,length(adr)-len) =' CRK';
    when (strip(sfx)='CT') substr(adr,length(adr)-len) =' CTS';
    when (strip(sfx)='JCTION') substr(adr,length(adr)-len) =' JCT';
    when (strip(sfx)='MSSN') substr(adr,length(adr)-len) =' MSN';
    when (strip(sfx)='BRDGE') substr(adr,length(adr)-len) =' BRG';
    when (strip(sfx)='CENT') substr(adr,length(adr)-len) =' CTR';
    when (strip(sfx)='FRT') substr(adr,length(adr)-len) =' FT';
    when (strip(sfx)='PK') substr(adr,length(adr)-len) =' PARK';
    when (strip(sfx)='LANES') substr(adr,length(adr)-len) =' LN';
    when (strip(sfx)='GTWAY') substr(adr,length(adr)-len) =' GTWY';
    when (strip(sfx)='PRK') substr(adr,length(adr)-len) =' PARK';
    when (strip(sfx)='STRAVENUE') substr(adr,length(adr)-len) =' STRA';
    when (strip(sfx)='HIWAY') substr(adr,length(adr)-len) =' HWY';
    when (strip(sfx)='VILLE') substr(adr,length(adr)-len) =' VL';
    when (strip(sfx)='PLAIN') substr(adr,length(adr)-len) =' PLN';
    when (strip(sfx)='MOUNT') substr(adr,length(adr)-len) =' MT';
    when (strip(sfx)='CENTR') substr(adr,length(adr)-len) =' CTR';
    when (strip(sfx)='PRR') substr(adr,length(adr)-len) =' PR';
    when (strip(sfx)='AVN') substr(adr,length(adr)-len) =' AVE';
    when (strip(sfx)='SPNG') substr(adr,length(adr)-len) =' SPG';
    when (strip(sfx)='HIWY') substr(adr,length(adr)-len) =' HWY';
    when (strip(sfx)='DAM') substr(adr,length(adr)-len) =' DM';
    when (strip(sfx)='CRCL') substr(adr,length(adr)-len) =' CIR';
    when (strip(sfx)='SQRE') substr(adr,length(adr)-len) =' SQ';
    when (strip(sfx)='JCTN') substr(adr,length(adr)-len) =' JCT';
    when (strip(sfx)='MOUNTAIN') substr(adr,length(adr)-len) =' MTN';
    when (strip(sfx)='KEYS') substr(adr,length(adr)-len) =' KYS';
    when (strip(sfx)='PARKWAYS') substr(adr,length(adr)-len) =' PKWY';
    when (strip(sfx)='DRIVES') substr(adr,length(adr)-len) =' DRS';
    when (strip(sfx)='CENTER') substr(adr,length(adr)-len) =' CTR';
    when (strip(sfx)='DRIV') substr(adr,length(adr)-len) =' DR';
    when (strip(sfx)='SUMITT') substr(adr,length(adr)-len) =' SMT';
    when (strip(sfx)='CANYN') substr(adr,length(adr)-len) =' CYN';
    when (strip(sfx)='HARBR') substr(adr,length(adr)-len) =' HBR';
    when (strip(sfx)='REST') substr(adr,length(adr)-len) =' RST';
    when (strip(sfx)='SHOARS') substr(adr,length(adr)-len) =' SHRS';
    when (strip(sfx)='VIST') substr(adr,length(adr)-len) =' VIS';
    when (strip(sfx)='ISLNDS') substr(adr,length(adr)-len) =' ISS';
    when (strip(sfx)='HILLS') substr(adr,length(adr)-len) =' HLS';
    when (strip(sfx)='CRESENT') substr(adr,length(adr)-len) =' CRES';
    when (strip(sfx)='POINT') substr(adr,length(adr)-len) =' PT';
    when (strip(sfx)='LAKE') substr(adr,length(adr)-len) =' LK';
    when (strip(sfx)='VLLY') substr(adr,length(adr)-len) =' VLY';
    when (strip(sfx)='STRAV') substr(adr,length(adr)-len) =' STRA';
    when (strip(sfx)='CROSSROAD') substr(adr,length(adr)-len) =' XRD';
    when (strip(sfx)='STRAVE') substr(adr,length(adr)-len) =' STRA';
    when (strip(sfx)='STRAVN') substr(adr,length(adr)-len) =' STRA';
    when (strip(sfx)='KNOL') substr(adr,length(adr)-len) =' KNL';
    when (strip(sfx)='FORGE') substr(adr,length(adr)-len) =' FRG';
    when (strip(sfx)='CNTR') substr(adr,length(adr)-len) =' CTR';
    when (strip(sfx)='CAPE') substr(adr,length(adr)-len) =' CPE';
    when (strip(sfx)='HEIGHT') substr(adr,length(adr)-len) =' HTS';
    when (strip(sfx)='HIGHWY') substr(adr,length(adr)-len) =' HWY';
    when (strip(sfx)='TRNPK') substr(adr,length(adr)-len) =' TPKE';
    when (strip(sfx)='BOULV') substr(adr,length(adr)-len) =' BLVD';
    when (strip(sfx)='CIRCLES') substr(adr,length(adr)-len) =' CIRS';
    when (strip(sfx)='VALLEYS') substr(adr,length(adr)-len) =' VLYS';
    when (strip(sfx)='VST') substr(adr,length(adr)-len) =' VIS';
    when (strip(sfx)='CREEK') substr(adr,length(adr)-len) =' CRK';
    when (strip(sfx)='SPRING') substr(adr,length(adr)-len) =' SPG';
    when (strip(sfx)='HOLWS') substr(adr,length(adr)-len) =' HOLW';
    when (strip(sfx)='TRACE') substr(adr,length(adr)-len) =' TRCE';
    when (strip(sfx)='BOTTOM') substr(adr,length(adr)-len) =' BTM';
    when (strip(sfx)='STREME') substr(adr,length(adr)-len) =' STRM';
    when (strip(sfx)='ISLES') substr(adr,length(adr)-len) =' ISLE';
    when (strip(sfx)='CIRC') substr(adr,length(adr)-len) =' CIR';
    when (strip(sfx)='FORKS') substr(adr,length(adr)-len) =' FRKS';
    when (strip(sfx)='BURG') substr(adr,length(adr)-len) =' BG';
    when (strip(sfx)='TRLS') substr(adr,length(adr)-len) =' TRL';
    when (strip(sfx)='RADIAL') substr(adr,length(adr)-len) =' RADL';
    when (strip(sfx)='LAKES') substr(adr,length(adr)-len) =' LKS';
    when (strip(sfx)='EXTENSION') substr(adr,length(adr)-len) =' EXT';
    when (strip(sfx)='ISLAND') substr(adr,length(adr)-len) =' IS';
    when (strip(sfx)='TERR') substr(adr,length(adr)-len) =' TER';
    when (strip(sfx)='UNION') substr(adr,length(adr)-len) =' UN';
    when (strip(sfx)='EXTENSIONS') substr(adr,length(adr)-len) =' EXTS';
    when (strip(sfx)='PKWYS') substr(adr,length(adr)-len) =' PKWY';
    when (strip(sfx)='ISLANDS') substr(adr,length(adr)-len) =' ISS';
    when (strip(sfx)='ROAD') substr(adr,length(adr)-len) =' RD';
    when (strip(sfx)='ROADS') substr(adr,length(adr)-len) =' RDS';
    when (strip(sfx)='GLENS') substr(adr,length(adr)-len) =' GLNS';
    when (strip(sfx)='SPRINGS') substr(adr,length(adr)-len) =' SPGS';
    when (strip(sfx)='MISSN') substr(adr,length(adr)-len) =' MSN';
    when (strip(sfx)='RIDGE') substr(adr,length(adr)-len) =' RDG';
    when (strip(sfx)='ARCADE') substr(adr,length(adr)-len) =' ARC';
    when (strip(sfx)='BAYOU') substr(adr,length(adr)-len) =' BYU';
    when (strip(sfx)='CRSNT') substr(adr,length(adr)-len) =' CRES';
    when (strip(sfx)='JUNCTN') substr(adr,length(adr)-len) =' JCT';
    when (strip(sfx)='VALLEY') substr(adr,length(adr)-len) =' VLY';
    when (strip(sfx)='FORK') substr(adr,length(adr)-len) =' FRK';
    when (strip(sfx)='MOUNTAINS') substr(adr,length(adr)-len) =' MTNS';
    when (strip(sfx)='BOTTM') substr(adr,length(adr)-len) =' BTM';
    when (strip(sfx)='FORG') substr(adr,length(adr)-len) =' FRG';
    when (strip(sfx)='HT') substr(adr,length(adr)-len) =' HTS';
    when (strip(sfx)='FORD') substr(adr,length(adr)-len) =' FRD';
    when (strip(sfx)='GRDN') substr(adr,length(adr)-len) =' GDN';
    when (strip(sfx)='FORT') substr(adr,length(adr)-len) =' FT';
    when (strip(sfx)='TRACES') substr(adr,length(adr)-len) =' TRCE';
    when (strip(sfx)='CNYN') substr(adr,length(adr)-len) =' CYN';
    when (strip(sfx)='FLATS') substr(adr,length(adr)-len) =' FLTS';
    when (strip(sfx)='ANEX') substr(adr,length(adr)-len) =' ANX';
    when (strip(sfx)='GATWAY') substr(adr,length(adr)-len) =' GTWY';
    when (strip(sfx)='RAPIDS') substr(adr,length(adr)-len) =' RPDS';
    when (strip(sfx)='VILLIAGE') substr(adr,length(adr)-len) =' VLG';
    when (strip(sfx)='COVES') substr(adr,length(adr)-len) =' CVS';
    when (strip(sfx)='RVR') substr(adr,length(adr)-len) =' RIV';
    when (strip(sfx)='AV') substr(adr,length(adr)-len) =' AVE';
    when (strip(sfx)='PIKES') substr(adr,length(adr)-len) =' PIKE';
    when (strip(sfx)='VISTA') substr(adr,length(adr)-len) =' VIS';
    when (strip(sfx)='FORESTS') substr(adr,length(adr)-len) =' FRST';
    when (strip(sfx)='FIELD') substr(adr,length(adr)-len) =' FLD';
    when (strip(sfx)='BRANCH') substr(adr,length(adr)-len) =' BR';
    when (strip(sfx)='DALE') substr(adr,length(adr)-len) =' DL';
    when (strip(sfx)='ANNEX') substr(adr,length(adr)-len) =' ANX';
    when (strip(sfx)='SQR') substr(adr,length(adr)-len) =' SQ';
    when (strip(sfx)='COVE') substr(adr,length(adr)-len) =' CV';
    when (strip(sfx)='SQU') substr(adr,length(adr)-len) =' SQ';
    when (strip(sfx)='SKYWAY') substr(adr,length(adr)-len) =' SKWY';
    when (strip(sfx)='RIDGES') substr(adr,length(adr)-len) =' RDGS';
    when (strip(sfx)='TUNNL') substr(adr,length(adr)-len) =' TUNL';
    when (strip(sfx)='UNDERPASS') substr(adr,length(adr)-len) =' UPAS';
    when (strip(sfx)='CLIFF') substr(adr,length(adr)-len) =' CLF';
    when (strip(sfx)='LANE') substr(adr,length(adr)-len) =' LN';
    when (strip(sfx)='DVD') substr(adr,length(adr)-len) =' DV';
    when (strip(sfx)='CURVE') substr(adr,length(adr)-len) =' CURV';
    when (strip(sfx)='SUMMIT') substr(adr,length(adr)-len) =' SMT';
    when (strip(sfx)='GARDENS') substr(adr,length(adr)-len) =' GDNS';
    ;;;;
    run;quit;

    /*       _                        _                         _       _                  _               _
    (_)_ __ | |_ ___ _ __ _ __   __ _| | __      _____  _ __ __| |  ___| |_ __ _ _ __   __| | __ _ _ __ __| |
    | | `_ \| __/ _ \ `__| `_ \ / _` | | \ \ /\ / / _ \| `__/ _` | / __| __/ _` | `_ \ / _` |/ _` | `__/ _` |
    | | | | | ||  __/ |  | | | | (_| | |  \ V  V / (_) | | | (_| | \__ \ || (_| | | | | (_| | (_| | | | (_| |
    |_|_| |_|\__\___|_|  |_| |_|\__,_|_|   \_/\_/ \___/|_|  \__,_| |___/\__\__,_|_| |_|\__,_|\__,_|_|  \__,_|

    */

    filename ft15f001 clear;
    filename ft15f001 "%sysfunc(pathname(work))/adr_010sx1.sas";
    parmcards4;
         select;
            when (index(adr,' STREET '))      adr=tranwrd(adr,' STREET '     , ' ST '       );
            when (index(adr,' AVENUE '))      adr=tranwrd(adr,' AVENUE '     , ' AVE '      );
            when (index(adr,' ROAD '))        adr=tranwrd(adr,' ROAD '       , ' RD '       );
            when (index(adr,' BOULEVARD '))   adr=tranwrd(adr,' BOULEVARD '  , ' BLVD '     );
            when (index(adr,' VALLEY '))      adr=tranwrd(adr,' VALLEY '     , ' VLY '      );
            when (index(adr,' CNTER '))       adr=tranwrd(adr,' CNTER '      , ' CTR '      );
            when (index(adr,' ROUTE '))       adr=tranwrd(adr,' ROUTE '      , ' RTE '      );
            when (index(adr,' HEIGHTS '))     adr=tranwrd(adr,' HEIGHTS '    , ' HTS '      );
            when (index(adr,' EXPRESSWAY '))  adr=tranwrd(adr,' EXPRESSWAY ' , ' EXPY '     );
            when (index(adr,' CREEK '))       adr=tranwrd(adr,' CREEK '      , ' CRK '      );
            when (index(adr,' CIRCL '))       adr=tranwrd(adr,' CIRCL '      , ' CIR '      );
            when (index(adr,' TRPK '))        adr=tranwrd(adr,' TRPK '       , ' TPKE '     );
            when (index(adr,' PKY '))         adr=tranwrd(adr,' PKY '        , ' PKWY '     );
            when (index(adr,' FORGES '))      adr=tranwrd(adr,' FORGES '     , ' FRGS '     );
            when (index(adr,' BYPAS '))       adr=tranwrd(adr,' BYPAS '      , ' BYP '      );
            when (index(adr,' VIADUCT '))     adr=tranwrd(adr,' VIADUCT '    , ' VIA '      );
            when (index(adr,' MNT '))         adr=tranwrd(adr,' MNT '        , ' MT '       );
            when (index(adr,' LNDNG '))       adr=tranwrd(adr,' LNDNG '      , ' LNDG '     );
            when (index(adr,' VILL '))        adr=tranwrd(adr,' VILL '       , ' VLG '      );
            when (index(adr,' MILL '))        adr=tranwrd(adr,' MILL '       , ' ML '       );
            when (index(adr,' CENTERS '))     adr=tranwrd(adr,' CENTERS '    , ' CTRS '     );
            when (index(adr,' HRBOR '))       adr=tranwrd(adr,' HRBOR '      , ' HBR '      );
            when (index(adr,' TR '))          adr=tranwrd(adr,' TR '         , ' TRL '      );
            when (index(adr,' PASSAGE '))     adr=tranwrd(adr,' PASSAGE '    , ' PSGE '     );
            when (index(adr,' WALKS '))       adr=tranwrd(adr,' WALKS '      , ' WALK '     );
            when (index(adr,' CREST '))       adr=tranwrd(adr,' CREST '      , ' CRST '     );
            when (index(adr,' MEADOWS '))     adr=tranwrd(adr,' MEADOWS '    , ' MDWS '     );
            when (index(adr,' FREEWY '))      adr=tranwrd(adr,' FREEWY '     , ' FWY '      );
            when (index(adr,' GARDEN '))      adr=tranwrd(adr,' GARDEN '     , ' GDN '      );
            when (index(adr,' BLUFFS '))      adr=tranwrd(adr,' BLUFFS '     , ' BLFS '     );
            when (index(adr,' TRK '))         adr=tranwrd(adr,' TRK '        , ' TRAK '     );
            when (index(adr,' SQUARES '))     adr=tranwrd(adr,' SQUARES '    , ' SQS '      );
            when (index(adr,' HARBOR '))      adr=tranwrd(adr,' HARBOR '     , ' HBR '      );
            when (index(adr,' FRRY '))        adr=tranwrd(adr,' FRRY '       , ' FRY '      );
            when (index(adr,' DIV '))         adr=tranwrd(adr,' DIV '        , ' DV '       );
            when (index(adr,' STRAVEN '))     adr=tranwrd(adr,' STRAVEN '    , ' STRA '     );
            when (index(adr,' CMP '))         adr=tranwrd(adr,' CMP '        , ' CP '       );
            when (index(adr,' GRDNS '))       adr=tranwrd(adr,' GRDNS '      , ' GDNS '     );
            when (index(adr,' VILLG '))       adr=tranwrd(adr,' VILLG '      , ' VLG '      );
            when (index(adr,' MEADOW '))      adr=tranwrd(adr,' MEADOW '     , ' MDW '      );
            when (index(adr,' TRAILS '))      adr=tranwrd(adr,' TRAILS '     , ' TRL '      );
            when (index(adr,' STREETS '))     adr=tranwrd(adr,' STREETS '    , ' STS '      );
            when (index(adr,' PRAIRIE '))     adr=tranwrd(adr,' PRAIRIE '    , ' PR '       );
            when (index(adr,' CRESCENT '))    adr=tranwrd(adr,' CRESCENT '   , ' CRES '     );
            when (index(adr,' PORT '))        adr=tranwrd(adr,' PORT '       , ' PRT '      );
            when (index(adr,' TRAFFICWAY '))  adr=tranwrd(adr,' TRAFFICWAY ' , ' TRFY '     );
            when (index(adr,' BLUF '))        adr=tranwrd(adr,' BLUF '       , ' BLF '      );
            when (index(adr,' AVNUE '))       adr=tranwrd(adr,' AVNUE '      , ' AVE '      );
            when (index(adr,' LIGHTS '))      adr=tranwrd(adr,' LIGHTS '     , ' LGTS '     );
            when (index(adr,' HARBORS '))     adr=tranwrd(adr,' HARBORS '    , ' HBRS '     );
            when (index(adr,' LODG '))        adr=tranwrd(adr,' LODG '       , ' LDG '      );
            when (index(adr,' TRACKS '))      adr=tranwrd(adr,' TRACKS '     , ' TRAK '     );
            when (index(adr,' PKWAY '))       adr=tranwrd(adr,' PKWAY '      , ' PKWY '     );
            when (index(adr,' BOT '))         adr=tranwrd(adr,' BOT '        , ' BTM '      );
            when (index(adr,' DRV '))         adr=tranwrd(adr,' DRV '        , ' DR '       );
            when (index(adr,' DIVIDE '))      adr=tranwrd(adr,' DIVIDE '     , ' DV '       );
            when (index(adr,' FORDS '))       adr=tranwrd(adr,' FORDS '      , ' FRDS '     );
            when (index(adr,' AVENU '))       adr=tranwrd(adr,' AVENU '      , ' AVE '      );
            when (index(adr,' RIVR '))        adr=tranwrd(adr,' RIVR '       , ' RIV '      );
            when (index(adr,' GATEWAY '))     adr=tranwrd(adr,' GATEWAY '    , ' GTWY '     );
            when (index(adr,' STREAM '))      adr=tranwrd(adr,' STREAM '     , ' STRM '     );
            when (index(adr,' BAYOO '))       adr=tranwrd(adr,' BAYOO '      , ' BYU '      );
            when (index(adr,' KNOLL '))       adr=tranwrd(adr,' KNOLL '      , ' KNL '      );
            when (index(adr,' SPRNG '))       adr=tranwrd(adr,' SPRNG '      , ' SPG '      );
            when (index(adr,' FLAT '))        adr=tranwrd(adr,' FLAT '       , ' FLT '      );
            when (index(adr,' GRDEN '))       adr=tranwrd(adr,' GRDEN '      , ' GDN '      );
            when (index(adr,' TRAIL '))       adr=tranwrd(adr,' TRAIL '      , ' TRL '      );
            when (index(adr,' JCTNS '))       adr=tranwrd(adr,' JCTNS '      , ' JCTS '     );
            when (index(adr,' TUNNEL '))      adr=tranwrd(adr,' TUNNEL '     , ' TUNL '     );
            when (index(adr,' GROVES '))      adr=tranwrd(adr,' GROVES '     , ' GRVS '     );
            when (index(adr,' VALLY '))       adr=tranwrd(adr,' VALLY '      , ' VLY '      );
            when (index(adr,' FERRY '))       adr=tranwrd(adr,' FERRY '      , ' FRY '      );
            when (index(adr,' PARKWAY '))     adr=tranwrd(adr,' PARKWAY '    , ' PKWY '     );
            when (index(adr,' RADIEL '))      adr=tranwrd(adr,' RADIEL '     , ' RADL '     );
            when (index(adr,' STRVNUE '))     adr=tranwrd(adr,' STRVNUE '    , ' STRA '     );
            when (index(adr,' OVERPASS '))    adr=tranwrd(adr,' OVERPASS '   , ' OPAS '     );
            when (index(adr,' PLAZA '))       adr=tranwrd(adr,' PLAZA '      , ' PLZ '      );
            when (index(adr,' ESTATE '))      adr=tranwrd(adr,' ESTATE '     , ' EST '      );
            when (index(adr,' MNTN '))        adr=tranwrd(adr,' MNTN '       , ' MTN '      );
            when (index(adr,' LOCK '))        adr=tranwrd(adr,' LOCK '       , ' LCK '      );
            when (index(adr,' ORCHRD '))      adr=tranwrd(adr,' ORCHRD '     , ' ORCH '     );
            when (index(adr,' STRVN '))       adr=tranwrd(adr,' STRVN '      , ' STRA '     );
            when (index(adr,' LOCKS '))       adr=tranwrd(adr,' LOCKS '      , ' LCKS '     );
            when (index(adr,' BEND '))        adr=tranwrd(adr,' BEND '       , ' BND '      );
            when (index(adr,' JUNCTIONS '))   adr=tranwrd(adr,' JUNCTIONS '  , ' JCTS '     );
            when (index(adr,' MOUNTIN '))     adr=tranwrd(adr,' MOUNTIN '    , ' MTN '      );
            when (index(adr,' BURGS '))       adr=tranwrd(adr,' BURGS '      , ' BGS '      );
            when (index(adr,' PINE '))        adr=tranwrd(adr,' PINE '       , ' PNE '      );
            when (index(adr,' LDGE '))        adr=tranwrd(adr,' LDGE '       , ' LDG '      );
            when (index(adr,' CAUSWAY '))     adr=tranwrd(adr,' CAUSWAY '    , ' CSWY '     );
            when (index(adr,' BEACH '))       adr=tranwrd(adr,' BEACH '      , ' BCH '      );
            when (index(adr,' MOTORWAY '))    adr=tranwrd(adr,' MOTORWAY '   , ' MTWY '     );
            when (index(adr,' BLUFF '))       adr=tranwrd(adr,' BLUFF '      , ' BLF '      );
            when (index(adr,' COURT '))       adr=tranwrd(adr,' COURT '      , ' CT '       );
            when (index(adr,' GROV '))        adr=tranwrd(adr,' GROV '       , ' GRV '      );
            when (index(adr,' SPRNGS '))      adr=tranwrd(adr,' SPRNGS '     , ' SPGS '     );
            when (index(adr,' OVL '))         adr=tranwrd(adr,' OVL '        , ' OVAL '     );
            when (index(adr,' VILLAG '))      adr=tranwrd(adr,' VILLAG '     , ' VLG '      );
            when (index(adr,' VDCT '))        adr=tranwrd(adr,' VDCT '       , ' VIA '      );
            when (index(adr,' NECK '))        adr=tranwrd(adr,' NECK '       , ' NCK '      );
            when (index(adr,' ORCHARD '))     adr=tranwrd(adr,' ORCHARD '    , ' ORCH '     );
            when (index(adr,' LIGHT '))       adr=tranwrd(adr,' LIGHT '      , ' LGT '      );
            when (index(adr,' SHORE '))       adr=tranwrd(adr,' SHORE '      , ' SHR '      );
            when (index(adr,' GREEN '))       adr=tranwrd(adr,' GREEN '      , ' GRN '      );
            when (index(adr,' ISLND '))       adr=tranwrd(adr,' ISLND '      , ' IS '       );
            when (index(adr,' TURNPIKE '))    adr=tranwrd(adr,' TURNPIKE '   , ' TPKE '     );
            when (index(adr,' MISSION '))     adr=tranwrd(adr,' MISSION '    , ' MSN '      );
            when (index(adr,' SPNGS '))       adr=tranwrd(adr,' SPNGS '      , ' SPGS '     );
            when (index(adr,' COURSE '))      adr=tranwrd(adr,' COURSE '     , ' CRSE '     );
            when (index(adr,' TERRACE '))     adr=tranwrd(adr,' TERRACE '    , ' TER '      );
            when (index(adr,' HWAY '))        adr=tranwrd(adr,' HWAY '       , ' HWY '      );
            when (index(adr,' GLEN '))        adr=tranwrd(adr,' GLEN '       , ' GLN '      );
            when (index(adr,' BOUL '))        adr=tranwrd(adr,' BOUL '       , ' BLVD '     );
            when (index(adr,' INLET '))       adr=tranwrd(adr,' INLET '      , ' INLT '     );
            when (index(adr,' LA '))          adr=tranwrd(adr,' LA '         , ' LN '       );
            when (index(adr,' BROOK '))       adr=tranwrd(adr,' BROOK '      , ' BRK '      );
            when (index(adr,' SHOAR '))       adr=tranwrd(adr,' SHOAR '      , ' SHR '      );
            when (index(adr,' BYPASS '))      adr=tranwrd(adr,' BYPASS '     , ' BYP '      );
            when (index(adr,' MTIN '))        adr=tranwrd(adr,' MTIN '       , ' MTN '      );
            when (index(adr,' ALLY '))        adr=tranwrd(adr,' ALLY '       , ' ALY '      );
            when (index(adr,' FOREST '))      adr=tranwrd(adr,' FOREST '     , ' FRST '     );
            when (index(adr,' JUNCTION '))    adr=tranwrd(adr,' JUNCTION '   , ' JCT '      );
            when (index(adr,' VIEWS '))       adr=tranwrd(adr,' VIEWS '      , ' VWS '      );
            when (index(adr,' WELLS '))       adr=tranwrd(adr,' WELLS '      , ' WLS '      );
            when (index(adr,' CEN '))         adr=tranwrd(adr,' CEN '        , ' CTR '      );
            when (index(adr,' CRT '))         adr=tranwrd(adr,' CRT '        , ' CT '       );
            when (index(adr,' CORNERS '))     adr=tranwrd(adr,' CORNERS '    , ' CORS '     );
            when (index(adr,' FRWAY '))       adr=tranwrd(adr,' FRWAY '      , ' FWY '      );
            when (index(adr,' PRARIE '))      adr=tranwrd(adr,' PRARIE '     , ' PR '       );
            when (index(adr,' CROSSING '))    adr=tranwrd(adr,' CROSSING '   , ' XING '     );
            when (index(adr,' EXTN '))        adr=tranwrd(adr,' EXTN '       , ' EXT '      );
            when (index(adr,' CLIFFS '))      adr=tranwrd(adr,' CLIFFS '     , ' CLFS '     );
            when (index(adr,' MANORS '))      adr=tranwrd(adr,' MANORS '     , ' MNRS '     );
            when (index(adr,' PORTS '))       adr=tranwrd(adr,' PORTS '      , ' PRTS '     );
            when (index(adr,' GATEWY '))      adr=tranwrd(adr,' GATEWY '     , ' GTWY '     );
            when (index(adr,' SQUARE '))      adr=tranwrd(adr,' SQUARE '     , ' SQ '       );
            when (index(adr,' HARB '))        adr=tranwrd(adr,' HARB '       , ' HBR '      );
            when (index(adr,' LOOPS '))       adr=tranwrd(adr,' LOOPS '      , ' LOOP '     );
            when (index(adr,' HILL '))        adr=tranwrd(adr,' HILL '       , ' HL '       );
            when (index(adr,' HIGHWAY '))     adr=tranwrd(adr,' HIGHWAY '    , ' HWY '      );
            when (index(adr,' BROOKS '))      adr=tranwrd(adr,' BROOKS '     , ' BRKS '     );
            when (index(adr,' BRNCH '))       adr=tranwrd(adr,' BRNCH '      , ' BR '       );
            when (index(adr,' AVEN '))        adr=tranwrd(adr,' AVEN '       , ' AVE '      );
            when (index(adr,' SHORES '))      adr=tranwrd(adr,' SHORES '     , ' SHRS '     );
            when (index(adr,' PLACE '))       adr=tranwrd(adr,' PLACE '      , ' PL '       );
            when (index(adr,' SUMIT '))       adr=tranwrd(adr,' SUMIT '      , ' SMT '      );
            when (index(adr,' PINES '))       adr=tranwrd(adr,' PINES '      , ' PNES '     );
            when (index(adr,' TRKS '))        adr=tranwrd(adr,' TRKS '       , ' TRAK '     );
            when (index(adr,' SHOAL '))       adr=tranwrd(adr,' SHOAL '      , ' SHL '      );
            when (index(adr,' STRT '))        adr=tranwrd(adr,' STRT '       , ' ST '       );
            when (index(adr,' FRWY '))        adr=tranwrd(adr,' FRWY '       , ' FWY '      );
            when (index(adr,' RANCHES '))     adr=tranwrd(adr,' RANCHES '    , ' RNCH '     );
            when (index(adr,' BOULEVARD '))   adr=tranwrd(adr,' BOULEVARD '  , ' BLVD '     );
            when (index(adr,' EXTNSN '))      adr=tranwrd(adr,' EXTNSN '     , ' EXT '      );
            when (index(adr,' HOLLOWS '))     adr=tranwrd(adr,' HOLLOWS '    , ' HOLW '     );
            when (index(adr,' VSTA '))        adr=tranwrd(adr,' VSTA '       , ' VIS '      );
            when (index(adr,' PLAINS '))      adr=tranwrd(adr,' PLAINS '     , ' PLNS '     );
            when (index(adr,' STATION '))     adr=tranwrd(adr,' STATION '    , ' STA '      );
            when (index(adr,' MNTNS '))       adr=tranwrd(adr,' MNTNS '      , ' MTNS '     );
            when (index(adr,' VILLAGES '))    adr=tranwrd(adr,' VILLAGES '   , ' VLGS '     );
            when (index(adr,' HAVEN '))       adr=tranwrd(adr,' HAVEN '      , ' HVN '      );
            when (index(adr,' TURNPK '))      adr=tranwrd(adr,' TURNPK '     , ' TPKE '     );
            when (index(adr,' EXPR '))        adr=tranwrd(adr,' EXPR '       , ' EXPY '     );
            when (index(adr,' STN '))         adr=tranwrd(adr,' STN '        , ' STA '      );
            when (index(adr,' EXPW '))        adr=tranwrd(adr,' EXPW '       , ' EXPY '     );
            when (index(adr,' STR '))         adr=tranwrd(adr,' STR '        , ' ST '       );
            when (index(adr,' SPURS '))       adr=tranwrd(adr,' SPURS '      , ' SPUR '     );
            when (index(adr,' CRECENT '))     adr=tranwrd(adr,' CRECENT '    , ' CRES '     );
            when (index(adr,' RAD '))         adr=tranwrd(adr,' RAD '        , ' RADL '     );
            when (index(adr,' RANCH '))       adr=tranwrd(adr,' RANCH '      , ' RNCH '     );
            when (index(adr,' WELL '))        adr=tranwrd(adr,' WELL '       , ' WL '       );
            when (index(adr,' SHOALS '))      adr=tranwrd(adr,' SHOALS '     , ' SHLS '     );
            when (index(adr,' ALLEY '))       adr=tranwrd(adr,' ALLEY '      , ' ALY '      );
            when (index(adr,' PLZA '))        adr=tranwrd(adr,' PLZA '       , ' PLZ '      );
            when (index(adr,' MEDOWS '))      adr=tranwrd(adr,' MEDOWS '     , ' MDWS '     );
            when (index(adr,' ALLEE '))       adr=tranwrd(adr,' ALLEE '      , ' ALY '      );
            when (index(adr,' HAVN '))        adr=tranwrd(adr,' HAVN '       , ' HVN '      );
            when (index(adr,' PATHS '))       adr=tranwrd(adr,' PATHS '      , ' PATH '     );
            when (index(adr,' BYPA '))        adr=tranwrd(adr,' BYPA '       , ' BYP '      );
            when (index(adr,' MILLS '))       adr=tranwrd(adr,' MILLS '      , ' MLS '      );
            when (index(adr,' PARKS '))       adr=tranwrd(adr,' PARKS '      , ' PARK '     );
            when (index(adr,' BYPS '))        adr=tranwrd(adr,' BYPS '       , ' BYP '      );
            when (index(adr,' TUNNELS '))     adr=tranwrd(adr,' TUNNELS '    , ' TUNL '     );
            when (index(adr,' CLUB '))        adr=tranwrd(adr,' CLUB '       , ' CLB '      );
            when (index(adr,' SQRS '))        adr=tranwrd(adr,' SQRS '       , ' SQS '      );
            when (index(adr,' HLLW '))        adr=tranwrd(adr,' HLLW '       , ' HOLW '     );
            when (index(adr,' MANOR '))       adr=tranwrd(adr,' MANOR '      , ' MNR '      );
            when (index(adr,' CENTRE '))      adr=tranwrd(adr,' CENTRE '     , ' CTR '      );
            when (index(adr,' TRACK '))       adr=tranwrd(adr,' TRACK '      , ' TRAK '     );
            when (index(adr,' HGTS '))        adr=tranwrd(adr,' HGTS '       , ' HTS '      );
            when (index(adr,' CRCLE '))       adr=tranwrd(adr,' CRCLE '      , ' CIR '      );
            when (index(adr,' FALLS '))       adr=tranwrd(adr,' FALLS '      , ' FLS '      );
            when (index(adr,' LANDING '))     adr=tranwrd(adr,' LANDING '    , ' LNDG '     );
            when (index(adr,' PLAINES '))     adr=tranwrd(adr,' PLAINES '    , ' PLNS '     );
            when (index(adr,' VIADCT '))      adr=tranwrd(adr,' VIADCT '     , ' VIA '      );
            when (index(adr,' GROVE '))       adr=tranwrd(adr,' GROVE '      , ' GRV '      );
            when (index(adr,' CAMP '))        adr=tranwrd(adr,' CAMP '       , ' CP '       );
            when (index(adr,' TPK '))         adr=tranwrd(adr,' TPK '        , ' TPKE '     );
            when (index(adr,' DRIVE '))       adr=tranwrd(adr,' DRIVE '      , ' DR '       );
            when (index(adr,' FREEWAY '))     adr=tranwrd(adr,' FREEWAY '    , ' FWY '      );
            when (index(adr,' POINTS '))      adr=tranwrd(adr,' POINTS '     , ' PTS '      );
            when (index(adr,' EXP '))         adr=tranwrd(adr,' EXP '        , ' EXPY '     );
            when (index(adr,' COURTS '))      adr=tranwrd(adr,' COURTS '     , ' CTS '      );
            when (index(adr,' CORNER '))      adr=tranwrd(adr,' CORNER '     , ' COR '      );
            when (index(adr,' CRSSING '))     adr=tranwrd(adr,' CRSSING '    , ' XING '     );
            when (index(adr,' UNIONS '))      adr=tranwrd(adr,' UNIONS '     , ' UNS '      );
            when (index(adr,' LODGE '))       adr=tranwrd(adr,' LODGE '      , ' LDG '      );
            when (index(adr,' CIRCLE '))      adr=tranwrd(adr,' CIRCLE '     , ' CIR '      );
            when (index(adr,' BRIDGE '))      adr=tranwrd(adr,' BRIDGE '     , ' BRG '      );
            when (index(adr,' EXPRESS '))     adr=tranwrd(adr,' EXPRESS '    , ' EXPY '     );
            when (index(adr,' TUNLS '))       adr=tranwrd(adr,' TUNLS '      , ' TUNL '     );
            when (index(adr,' KNOLLS '))      adr=tranwrd(adr,' KNOLLS '     , ' KNLS '     );
            when (index(adr,' GREENS '))      adr=tranwrd(adr,' GREENS '     , ' GRNS '     );
            when (index(adr,' TUNEL '))       adr=tranwrd(adr,' TUNEL '      , ' TUNL '     );
            when (index(adr,' FIELDS '))      adr=tranwrd(adr,' FIELDS '     , ' FLDS '     );
            when (index(adr,' COMMON '))      adr=tranwrd(adr,' COMMON '     , ' CMN '      );
            when (index(adr,' RIVER '))       adr=tranwrd(adr,' RIVER '      , ' RIV '      );
            when (index(adr,' VIEW '))        adr=tranwrd(adr,' VIEW '       , ' VW '       );
            when (index(adr,' CRSENT '))      adr=tranwrd(adr,' CRSENT '     , ' CRES '     );
            when (index(adr,' RNCHS '))       adr=tranwrd(adr,' RNCHS '      , ' RNCH '     );
            when (index(adr,' CRSCNT '))      adr=tranwrd(adr,' CRSCNT '     , ' CRES '     );
            when (index(adr,' RDGE '))        adr=tranwrd(adr,' RDGE '       , ' RDG '      );
            when (index(adr,' CAUSEWAY '))    adr=tranwrd(adr,' CAUSEWAY '   , ' CSWY '     );
            when (index(adr,' PARKWY '))      adr=tranwrd(adr,' PARKWY '     , ' PKWY '     );
            when (index(adr,' JUNCTON '))     adr=tranwrd(adr,' JUNCTON '    , ' JCT '      );
            when (index(adr,' STATN '))       adr=tranwrd(adr,' STATN '      , ' STA '      );
            when (index(adr,' GARDN '))       adr=tranwrd(adr,' GARDN '      , ' GDN '      );
            when (index(adr,' MNTAIN '))      adr=tranwrd(adr,' MNTAIN '     , ' MTN '      );
            when (index(adr,' CRSSNG '))      adr=tranwrd(adr,' CRSSNG '     , ' XING '     );
            when (index(adr,' RAPID '))       adr=tranwrd(adr,' RAPID '      , ' RPD '      );
            when (index(adr,' KEY '))         adr=tranwrd(adr,' KEY '        , ' KY '       );
            when (index(adr,' WY '))          adr=tranwrd(adr,' WY '         , ' WAY '      );
            when (index(adr,' THROUGHWAY '))  adr=tranwrd(adr,' THROUGHWAY ' , ' TRWY '     );
            when (index(adr,' ESTATES '))     adr=tranwrd(adr,' ESTATES '    , ' ESTS '     );
            when (index(adr,' CK '))          adr=tranwrd(adr,' CK '         , ' CRK '      );
            when (index(adr,' LOAF '))        adr=tranwrd(adr,' LOAF '       , ' LF '       );
            when (index(adr,' HOLLOW '))      adr=tranwrd(adr,' HOLLOW '     , ' HOLW '     );
            when (index(adr,' CANYON '))      adr=tranwrd(adr,' CANYON '     , ' CYN '      );
            when (index(adr,' VILLAGE '))     adr=tranwrd(adr,' VILLAGE '    , ' VLG '      );
            when (index(adr,' CR '))          adr=tranwrd(adr,' CR '         , ' CRK '      );
            when (index(adr,' CT '))          adr=tranwrd(adr,' CT '         , ' CTS '      );
            when (index(adr,' JCTION '))      adr=tranwrd(adr,' JCTION '     , ' JCT '      );
            when (index(adr,' MSSN '))        adr=tranwrd(adr,' MSSN '       , ' MSN '      );
            when (index(adr,' BRDGE '))       adr=tranwrd(adr,' BRDGE '      , ' BRG '      );
            when (index(adr,' CENT '))        adr=tranwrd(adr,' CENT '       , ' CTR '      );
            when (index(adr,' FRT '))         adr=tranwrd(adr,' FRT '        , ' FT '       );
            when (index(adr,' PK '))          adr=tranwrd(adr,' PK '         , ' PARK '     );
            when (index(adr,' LANES '))       adr=tranwrd(adr,' LANES '      , ' LN '       );
            when (index(adr,' GTWAY '))       adr=tranwrd(adr,' GTWAY '      , ' GTWY '     );
            when (index(adr,' PRK '))         adr=tranwrd(adr,' PRK '        , ' PARK '     );
            when (index(adr,' STRAVENUE '))   adr=tranwrd(adr,' STRAVENUE '  , ' STRA '     );
            when (index(adr,' HIWAY '))       adr=tranwrd(adr,' HIWAY '      , ' HWY '      );
            when (index(adr,' VILLE '))       adr=tranwrd(adr,' VILLE '      , ' VL '       );
            when (index(adr,' PLAIN '))       adr=tranwrd(adr,' PLAIN '      , ' PLN '      );
            when (index(adr,' MOUNT '))       adr=tranwrd(adr,' MOUNT '      , ' MT '       );
            when (index(adr,' CENTR '))       adr=tranwrd(adr,' CENTR '      , ' CTR '      );
            when (index(adr,' PRR '))         adr=tranwrd(adr,' PRR '        , ' PR '       );
            when (index(adr,' AVN '))         adr=tranwrd(adr,' AVN '        , ' AVE '      );
            when (index(adr,' SPNG '))        adr=tranwrd(adr,' SPNG '       , ' SPG '      );
            when (index(adr,' HIWY '))        adr=tranwrd(adr,' HIWY '       , ' HWY '      );
            when (index(adr,' DAM '))         adr=tranwrd(adr,' DAM '        , ' DM '       );
            when (index(adr,' CRCL '))        adr=tranwrd(adr,' CRCL '       , ' CIR '      );
            when (index(adr,' SQRE '))        adr=tranwrd(adr,' SQRE '       , ' SQ '       );
            when (index(adr,' JCTN '))        adr=tranwrd(adr,' JCTN '       , ' JCT '      );
            when (index(adr,' MOUNTAIN '))    adr=tranwrd(adr,' MOUNTAIN '   , ' MTN '      );
            when (index(adr,' KEYS '))        adr=tranwrd(adr,' KEYS '       , ' KYS '      );
            when (index(adr,' PARKWAYS '))    adr=tranwrd(adr,' PARKWAYS '   , ' PKWY '     );
            when (index(adr,' DRIVES '))      adr=tranwrd(adr,' DRIVES '     , ' DRS '      );
            when (index(adr,' CENTER '))      adr=tranwrd(adr,' CENTER '     , ' CTR '      );
            when (index(adr,' DRIV '))        adr=tranwrd(adr,' DRIV '       , ' DR '       );
            when (index(adr,' SUMITT '))      adr=tranwrd(adr,' SUMITT '     , ' SMT '      );
            when (index(adr,' CANYN '))       adr=tranwrd(adr,' CANYN '      , ' CYN '      );
            when (index(adr,' HARBR '))       adr=tranwrd(adr,' HARBR '      , ' HBR '      );
            when (index(adr,' REST '))        adr=tranwrd(adr,' REST '       , ' RST '      );
            when (index(adr,' SHOARS '))      adr=tranwrd(adr,' SHOARS '     , ' SHRS '     );
            when (index(adr,' VIST '))        adr=tranwrd(adr,' VIST '       , ' VIS '      );
            when (index(adr,' ISLNDS '))      adr=tranwrd(adr,' ISLNDS '     , ' ISS '      );
            when (index(adr,' HILLS '))       adr=tranwrd(adr,' HILLS '      , ' HLS '      );
            when (index(adr,' CRESENT '))     adr=tranwrd(adr,' CRESENT '    , ' CRES '     );
            when (index(adr,' POINT '))       adr=tranwrd(adr,' POINT '      , ' PT '       );
            when (index(adr,' LAKE '))        adr=tranwrd(adr,' LAKE '       , ' LK '       );
            when (index(adr,' VLLY '))        adr=tranwrd(adr,' VLLY '       , ' VLY '      );
            when (index(adr,' STRAV '))       adr=tranwrd(adr,' STRAV '      , ' STRA '     );
            when (index(adr,' CROSSROAD '))   adr=tranwrd(adr,' CROSSROAD '  , ' XRD '      );
            when (index(adr,' STRAVE '))      adr=tranwrd(adr,' STRAVE '     , ' STRA '     );
            when (index(adr,' STRAVN '))      adr=tranwrd(adr,' STRAVN '     , ' STRA '     );
            when (index(adr,' KNOL '))        adr=tranwrd(adr,' KNOL '       , ' KNL '      );
            when (index(adr,' FORGE '))       adr=tranwrd(adr,' FORGE '      , ' FRG '      );
            when (index(adr,' CNTR '))        adr=tranwrd(adr,' CNTR '       , ' CTR '      );
            when (index(adr,' CAPE '))        adr=tranwrd(adr,' CAPE '       , ' CPE '      );
            when (index(adr,' HEIGHT '))      adr=tranwrd(adr,' HEIGHT '     , ' HTS '      );
            when (index(adr,' HIGHWY '))      adr=tranwrd(adr,' HIGHWY '     , ' HWY '      );
            when (index(adr,' TRNPK '))       adr=tranwrd(adr,' TRNPK '      , ' TPKE '     );
            when (index(adr,' BOULV '))       adr=tranwrd(adr,' BOULV '      , ' BLVD '     );
            when (index(adr,' CIRCLES '))     adr=tranwrd(adr,' CIRCLES '    , ' CIRS '     );
            when (index(adr,' VALLEYS '))     adr=tranwrd(adr,' VALLEYS '    , ' VLYS '     );
            when (index(adr,' VST '))         adr=tranwrd(adr,' VST '        , ' VIS '      );
            when (index(adr,' SPRING '))      adr=tranwrd(adr,' SPRING '     , ' SPG '      );
            when (index(adr,' HOLWS '))       adr=tranwrd(adr,' HOLWS '      , ' HOLW '     );
            when (index(adr,' TRACE '))       adr=tranwrd(adr,' TRACE '      , ' TRCE '     );
            when (index(adr,' BOTTOM '))      adr=tranwrd(adr,' BOTTOM '     , ' BTM '      );
            when (index(adr,' STREME '))      adr=tranwrd(adr,' STREME '     , ' STRM '     );
            when (index(adr,' ISLES '))       adr=tranwrd(adr,' ISLES '      , ' ISLE '     );
            when (index(adr,' CIRC '))        adr=tranwrd(adr,' CIRC '       , ' CIR '      );
            when (index(adr,' FORKS '))       adr=tranwrd(adr,' FORKS '      , ' FRKS '     );
            when (index(adr,' BURG '))        adr=tranwrd(adr,' BURG '       , ' BG '       );
            when (index(adr,' TRLS '))        adr=tranwrd(adr,' TRLS '       , ' TRL '      );
            when (index(adr,' RADIAL '))      adr=tranwrd(adr,' RADIAL '     , ' RADL '     );
            when (index(adr,' LAKES '))       adr=tranwrd(adr,' LAKES '      , ' LKS '      );
            when (index(adr,' EXTENSION '))   adr=tranwrd(adr,' EXTENSION '  , ' EXT '      );
            when (index(adr,' ISLAND '))      adr=tranwrd(adr,' ISLAND '     , ' IS '       );
            when (index(adr,' TERR '))        adr=tranwrd(adr,' TERR '       , ' TER '      );
            when (index(adr,' UNION '))       adr=tranwrd(adr,' UNION '      , ' UN '       );
            when (index(adr,' EXTENSIONS '))  adr=tranwrd(adr,' EXTENSIONS ' , ' EXTS '     );
            when (index(adr,' PKWYS '))       adr=tranwrd(adr,' PKWYS '      , ' PKWY '     );
            when (index(adr,' ISLANDS '))     adr=tranwrd(adr,' ISLANDS '    , ' ISS '      );
            when (index(adr,' ROADS '))       adr=tranwrd(adr,' ROADS '      , ' RDS '      );
            when (index(adr,' GLENS '))       adr=tranwrd(adr,' GLENS '      , ' GLNS '     );
            when (index(adr,' SPRINGS '))     adr=tranwrd(adr,' SPRINGS '    , ' SPGS '     );
            when (index(adr,' MISSN '))       adr=tranwrd(adr,' MISSN '      , ' MSN '      );
            when (index(adr,' RIDGE '))       adr=tranwrd(adr,' RIDGE '      , ' RDG '      );
            when (index(adr,' ARCADE '))      adr=tranwrd(adr,' ARCADE '     , ' ARC '      );
            when (index(adr,' BAYOU '))       adr=tranwrd(adr,' BAYOU '      , ' BYU '      );
            when (index(adr,' CRSNT '))       adr=tranwrd(adr,' CRSNT '      , ' CRES '     );
            when (index(adr,' JUNCTN '))      adr=tranwrd(adr,' JUNCTN '     , ' JCT '      );
            when (index(adr,' FORK '))        adr=tranwrd(adr,' FORK '       , ' FRK '      );
            when (index(adr,' MOUNTAINS '))   adr=tranwrd(adr,' MOUNTAINS '  , ' MTNS '     );
            when (index(adr,' BOTTM '))       adr=tranwrd(adr,' BOTTM '      , ' BTM '      );
            when (index(adr,' FORG '))        adr=tranwrd(adr,' FORG '       , ' FRG '      );
            when (index(adr,' HT '))          adr=tranwrd(adr,' HT '         , ' HTS '      );
            when (index(adr,' FORD '))        adr=tranwrd(adr,' FORD '       , ' FRD '      );
            when (index(adr,' GRDN '))        adr=tranwrd(adr,' GRDN '       , ' GDN '      );
            when (index(adr,' FORT '))        adr=tranwrd(adr,' FORT '       , ' FT '       );
            when (index(adr,' TRACES '))      adr=tranwrd(adr,' TRACES '     , ' TRCE '     );
            when (index(adr,' CNYN '))        adr=tranwrd(adr,' CNYN '       , ' CYN '      );
            when (index(adr,' FLATS '))       adr=tranwrd(adr,' FLATS '      , ' FLTS '     );
            when (index(adr,' ANEX '))        adr=tranwrd(adr,' ANEX '       , ' ANX '      );
            when (index(adr,' GATWAY '))      adr=tranwrd(adr,' GATWAY '     , ' GTWY '     );
            when (index(adr,' RAPIDS '))      adr=tranwrd(adr,' RAPIDS '     , ' RPDS '     );
            when (index(adr,' VILLIAGE '))    adr=tranwrd(adr,' VILLIAGE '   , ' VLG '      );
            when (index(adr,' COVES '))       adr=tranwrd(adr,' COVES '      , ' CVS '      );
            when (index(adr,' RVR '))         adr=tranwrd(adr,' RVR '        , ' RIV '      );
            when (index(adr,' AV '))          adr=tranwrd(adr,' AV '         , ' AVE '      );
            when (index(adr,' PIKES '))       adr=tranwrd(adr,' PIKES '      , ' PIKE '     );
            when (index(adr,' VISTA '))       adr=tranwrd(adr,' VISTA '      , ' VIS '      );
            when (index(adr,' FORESTS '))     adr=tranwrd(adr,' FORESTS '    , ' FRST '     );
            when (index(adr,' FIELD '))       adr=tranwrd(adr,' FIELD '      , ' FLD '      );
            when (index(adr,' BRANCH '))      adr=tranwrd(adr,' BRANCH '     , ' BR '       );
            when (index(adr,' DALE '))        adr=tranwrd(adr,' DALE '       , ' DL '       );
            when (index(adr,' ANNEX '))       adr=tranwrd(adr,' ANNEX '      , ' ANX '      );
            when (index(adr,' SQR '))         adr=tranwrd(adr,' SQR '        , ' SQ '       );
            when (index(adr,' COVE '))        adr=tranwrd(adr,' COVE '       , ' CV '       );
            when (index(adr,' SQU '))         adr=tranwrd(adr,' SQU '        , ' SQ '       );
            when (index(adr,' SKYWAY '))      adr=tranwrd(adr,' SKYWAY '     , ' SKWY '     );
            when (index(adr,' RIDGES '))      adr=tranwrd(adr,' RIDGES '     , ' RDGS '     );
            when (index(adr,' TUNNL '))       adr=tranwrd(adr,' TUNNL '      , ' TUNL '     );
            when (index(adr,' UNDERPASS '))   adr=tranwrd(adr,' UNDERPASS '  , ' UPAS '     );
            when (index(adr,' CLIFF '))       adr=tranwrd(adr,' CLIFF '      , ' CLF '      );
            when (index(adr,' LANE '))        adr=tranwrd(adr,' LANE '       , ' LN '       );
            when (index(adr,' DVD '))         adr=tranwrd(adr,' DVD '        , ' DV '       );
            when (index(adr,' CURVE '))       adr=tranwrd(adr,' CURVE '      , ' CURV '     );
            when (index(adr,' SUMMIT '))      adr=tranwrd(adr,' SUMMIT '     , ' SMT '      );
            when (index(adr,' GARDENS '))     adr=tranwrd(adr,' GARDENS '    , ' GDNS '     );
            otherwise;
         end;
    ;;;;
    run;quit;

    /*---- from my autoexec                                                  ----*/

    %let states50q="AL","AK","AZ","AR","CA","CO","CT","DE","FL","GA","HI","ID","IL","IN","IA","KS","KY","LA","ME","MD","MA","MI","MN","MS","MO","MT",
    "NE","NV","NH","NJ","NM","NY","NC","ND","OH","OK","OR","PA","RI","SC","SD","TN","TX","UT","VT","VA","WA","WV","WI","WY";


    %stop_submission;


    /*               _          _                      __ _ _
     _ __ ___   __ _| | _____  | | ___  _ __   __ _   / _(_) | ___ _ __   __ _ _ __ ___   ___
    | `_ ` _ \ / _` | |/ / _ \ | |/ _ \| `_ \ / _` | | |_| | |/ _ \ `_ \ / _` | `_ ` _ \ / _ \
    | | | | | | (_| |   <  __/ | | (_) | | | | (_| | |  _| | |  __/ | | | (_| | | | | | |  __/
    |_| |_| |_|\__,_|_|\_\___| |_|\___/|_| |_|\__, | |_| |_|_|\___|_| |_|\__,_|_| |_| |_|\___|
                                              |___/
    */

    %macro mkefyl(_st,_fyl="%sysfunc(pathname(work)/fyl&_st..sas");

    /*----                                                                   ----*/
    /*---- %let _st = AK; %let _fyl="%sysfunc(pathname(work))/fyl&_st..sas"; ----*/
    /*----                                                                   ----*/

        proc datasets lib=work nolist nodetails;
          delete dir dirfix dirfin;
        run;quit;

        data dir;
           length root path $200 dir 8;
           call missing(path,dir);
           root=cats('m:/adr/json/',"&_st");
        run;quit;

        data dir;
          modify dir;
          rc=filename('tmp',catx('/',root,path));
          dir=dopen('tmp');
          replace;
          if dir;
          path0=path;
          do _N_=1 to dnum(dir);
            path=catx('/',path0,dread(dir,_N_));
            output;
            end;
          rc=dclose(dir);
        run;quit;

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /*                                                                                                                        */
    /* WORK.DIR total obs=49                                                                                                  */
    /*                                                                                                                        */
    /*  Obs         ROOT         PATH                                                          DIR                            */
    /*                                                                                                                        */
    /*    1    m:/adr/json/AK                                                                   1                             */
    /*    2    m:/adr/json/AK    anchorage-addresses-county.geojson                             0                             */
    /*    3    m:/adr/json/AK    anchorage-addresses-county.geojson.meta                        0                             */
    /*    4    m:/adr/json/AK    anchorage-buildings-county.geojson                             0                             */
    /*    5    m:/adr/json/AK    anchorage-buildings-county.geojson.meta                        0                             */
    /*    6    m:/adr/json/AK    anchorage-parcels-county.geojson                               0                             */
    /*    7    m:/adr/json/AK    anchorage-parcels-county.geojson.meta                          0                             */
    /*    8    m:/adr/json/AK    city_of_bethel-buildings-city.geojson                          0                             */
    /*    9    m:/adr/json/AK    city_of_bethel-buildings-city.geojson.meta                     0                             */
    /*   10    m:/adr/json/AK    city_of_juneau-addresses-city.geojson                          0                             */
    /*   11    m:/adr/json/AK    city_of_juneau-addresses-city.geojson.meta                     0                             */
    /*   ....                                                                                                                 */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

        options ps=300;
         data dirfix;
           length fld $255;
           set dir(where=(dir=0));
           fld=catx('/',root,path);
           if index(fld,'addresses')>0;
           if index(fld,'.meta')=0;
           fld = cats(',',quote(strip(fld)));
           keep fld;
           *utlog fld;
         run;quit;

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /*                                                                                                                        */
    /* WORK.DIRFIX total obs=10 (only 10 have addresses)                                                                      */
    /*                                                                                                                        */
    /* Obs                                      FLD                                                                           */
    /*                                                                                                                        */
    /*   1    ,"m:/adr/json/AK/anchorage-addresses-county.geojson"                                                            */
    /*   2    ,"m:/adr/json/AK/city_of_juneau-addresses-city.geojson"                                                         */
    /*   3    ,"m:/adr/json/AK/fairbanks_north_star_borough-addresses-county.geojson"                                         */
    /*   4    ,"m:/adr/json/AK/haines-addresses-county.geojson"                                                               */
    /*   5    ,"m:/adr/json/AK/kenai_peninsula_borough-addresses-county.geojson"                                              */
    /*   6    ,"m:/adr/json/AK/ketchikan-addresses-county.geojson"                                                            */
    /*   7    ,"m:/adr/json/AK/kodiak_island_borough-addresses-county.geojson"                                                */
    /*   8    ,"m:/adr/json/AK/matanuska_susitna_borough-addresses-county.geojson"                                            */
    /*   9    ,"m:/adr/json/AK/wrangell-addresses-county.geojson"                                                             */
    /*  10    ,"m:/adr/json/AK/yakutat_borough-addresses-county.geojson"                                                      */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

        data dirfin;
          set dirfix end=dne;
          file &_fyl;
          if _n_=1 then do;
              lyn = "filename &_st (";
              *putlog lyn;
              call execute(lyn);
              fld=compress(fld,',');
              *putlog fld;
              put fld;
              call execute(fld);
          end;

          else do;
              put fld;
              *putlog fld;
              call execute(fld);
          end;
          if dne then do;
                fld = ') lrecl=1024 recfm=v;';
                put  fld;
                *putlog fld;
                call execute(fld);
          end;
        run;quit;

    /**************************************************************************************************************************/
    /*  EXECUTE THIS FILENAME                                                                                                 */
    /*                                                                                                                        */
    /* + filename AK (                                                                                                        */
    /* + "m:/adr/json/AK/anchorage-addresses-county.geojson"                                                                  */
    /* + ,"m:/adr/json/AK/city_of_juneau-addresses-city.geojson"                                                              */
    /* + ,"m:/adr/json/AK/fairbanks_north_star_borough-addresses-county.geojson"                                              */
    /* + ,"m:/adr/json/AK/haines-addresses-county.geojson"                                                                    */
    /* + ,"m:/adr/json/AK/kenai_peninsula_borough-addresses-county.geojson"                                                   */
    /* + ,"m:/adr/json/AK/ketchikan-addresses-county.geojson"                                                                 */
    /* + ,"m:/adr/json/AK/kodiak_island_borough-addresses-county.geojson"                                                     */
    /* + ,"m:/adr/json/AK/matanuska_susitna_borough-addresses-county.geojson"                                                 */
    /* + ,"m:/adr/json/AK/wrangell-addresses-county.geojson"                                                                  */
    /* + ,"m:/adr/json/AK/yakutat_borough-addresses-county.geojson"                                                           */
    /* + ) lrecl=1024 recfm=v;                                                                                                */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    %mend mkefyl;

    /*                                     _                      _
     _ __ ___   __ _  ___ _ __ ___     ___| | ___  ___  ___   ___(_)_ __
    | `_ ` _ \ / _` |/ __| `__/ _ \   / __| |/ _ \/ __|/ _ \ |_  / | `_ \
    | | | | | | (_| | (__| | | (_) | | (__| | (_) \__ \  __/  / /| | |_) |
    |_| |_| |_|\__,_|\___|_|  \___/   \___|_|\___/|___/\___| /___|_| .__/
                                                                   |_|
    */


    /*---- assign closest zipcode                                             ----*/

    %macro slice(first=1)/des="add zipcodes 10k records atr a time";

        %let obs = %eval(&first + 9999);

        proc sql;
          create
             table adr_010Opn&st.&first (compress=binary) as
          select
            distinct
             l.*
            ,r.longitude
            ,r.latitude
            ,r.postalcode as zipcode
            ,geodist(l.lat,l.lon,r.latitude,r.longitude,'M') as geodist
          from
            adr_010&st.notzipunq(firstobs=&first obs=&obs drop=postcode) as l left join
              adr_010UsPlaceZipSrt(keep=latitude longitude postalcode admin_code1) as r
          on
            1=1
          group
              by l.lon, l.lat
          having
              geodist(l.lat,l.lon,r.latitude,r.longitude,'M')
                = min(geodist(l.lat,l.lon,r.latitude,r.longitude,'M'));
        ;quit;

        proc append data=adr_010Opn&st.&first base=adr_010Opn&st.mat;
        run;quit;

    %mend slice;

    /*            ____ _____  _    ____ _____
    __/\____/\__ / ___|_   _|/ \  |  _ \_   _| __/\____/\__
    \    /\    / \___ \ | | / _ \ | |_) || |   \    /\    /
    /_  _\/_  _\  ___) || |/ ___ \|  _ < | |   /_  _\/_  _\
      \/    \/   |____/ |_/_/   \_\_| \_\|_|     \/    \/
               _     _             _         _                   _                     _
      __ _  __| | __| |  _ __ ___ (_)___ ___(_)_ __   __ _   ___(_)_ __   ___ ___   __| | ___  ___
     / _` |/ _` |/ _` | | `_ ` _ \| / __/ __| | `_ \ / _` | |_  / | `_ \ / __/ _ \ / _` |/ _ \/ __|
    | (_| | (_| | (_| | | | | | | | \__ \__ \ | | | | (_| |  / /| | |_) | (_| (_) | (_| |  __/\__ \
     \__,_|\__,_|\__,_| |_| |_| |_|_|___/___/_|_| |_|\__, | /___|_| .__/ \___\___/ \__,_|\___||___/
                                                     |___/        |_|

    */

    libname opn    "m:/adr/opn";
    libname opnfin "m:/adr/opn";
    libname opnfix "m:/adr/opn";


    %macro sts(st);

       /*----  %let st=AK;                                                      ----*/

       /*               _           __ _ _        _ _     _
        _ __ ___   __ _| | _____   / _(_) | ___  | (_)___| |_
       | `_ ` _ \ / _` | |/ / _ \ | |_| | |/ _ \ | | / __| __|
       | | | | | | (_| |   <  __/ |  _| | |  __/ | | \__ \ |_
       |_| |_| |_|\__,_|_|\_\___| |_| |_|_|\___| |_|_|___/\__|

       */

       %mkefyl(&st);

       /*----                                                                   ----*/
       /*---- concatenate all the json files into one AK file                   ----*/
       /*----                                                                   ----*/

       data _null_;
         infile &st;
         input;
         file "%sysfunc(pathname(work))/&st.apn.json";
         put _infile_;
       run;quit;

       /**************************************************************************************************************************/
       /*                                                                                                                        */
       /* Two example output records NESTED JSON                                                                                 */
       /*                                                                                                                        */
       /* 1. {"type":"Feature","properties":{"hash":"ecd2ce1b580dba7e","number":"26960","street":"EKLUTNA VILLAGE RD","unit":"", */
       /*       "city":"Chugiak","district":"","region":"AK","postcode":"99567","id":""},"geometry":{"type":"Point",             */
       /*       "coordinates":[-149.3603915,61.4636244]}}                                                                        */
       /*                                                                                                                        */
       /* 2. {"type":"Feature","properties":{"hash":"ecd2ce1b580dba7e","number":"26636","street":"EKLUTNA VILLAGE RD","unit":"", */
       /*       "city":"Chugiak","district":"","region":"AK","postcode":"99567","id":""},"geometry":{"type":"Point",             */
       /*       "coordinates":[-149.3603915,61.4636244]}}                                                                        */
       /*                                                                                                                        */
       /**************************************************************************************************************************/

       data _null_;

       infile "%sysfunc(pathname(work))/&st.apn.json";

       length str nopt nofeat nobra $500;
       input;

       _infile_ = tranwrd(_infile_,'": ','":');
       _infile_ = tranwrd(_infile_,'", ','",');
       _infile_ = tranwrd(_infile_,'"}, "','"},"');

       str=compbl(_infile_);

       nofeat=cats('{',substr(str,index(str,'"number":')));

       nopt=prxchange('s/"geometry":\{"type":"Point",//',-1,nofeat);

       nobra=prxchange('s/\[|\]/"/',-1,nopt);
       nobra=prxchange('s/\{|\}//',-1,nobra);
       nobra=prxchange('s/\[\]//',-1,nobra);
       nobra=cats('{',nobra,'}');

       file "%sysfunc(pathname(work))/&st..json";

       if mod(_n_,500000)=0 then putlog _n_;
       put nobra;

       run;quit;

       /**************************************************************************************************************************/
       /*                                                                                                                        */
       /* Two example output records SIMPLE LINEAR JSON                                                                          */
       /*                                                                                                                        */
       /* 1. {"number":"26960","street":"EKLUTNA VILLAGE RD","unit":"","city":"Chugiak","district":"",                           */
       /*      "region":"AK","postcode":"99567","id":"","coordinates":"-149.3603915,61.4636244"}                                 */
       /*                                                                                                                        */
       /* 2. {"number":"26636","street":"EKLUTNA VILLAGE RD","unit":"","city":"Chugiak","district":"","                          */
       /*      region":"AK","postcode":"99567","id":"","coordinates":"-149.3614712,61.4607111"}                                  */
       /*                                                                                                                        */
       /**************************************************************************************************************************/

       data _null_;
         infile "%sysfunc(pathname(work))/&st..json";
         input;
         if mod(_n_,1000000)=0 then put _n_;
         if countc(_infile_,':') in (0,10,2) then return;
         _infile_= compress(_infile_,'"}{');
         _infile_=TRANSLATE(_infile_, ' ', ',');
         _infile_=compbl(TRANSLATE(_infile_, '=', ':'));
         equals=countc(_infile_,'=');
         file "%sysfunc(pathname(work))/&st..Prep.txt";
         put _infile_;
         if mod(_n_,1000000)=0 then put _n_;
         keep equals;
       run;quit;

       /**************************************************************************************************************************/
       /*                                                                                                                        */
       /* FINAL file for wps/sas to convert to a table                                                                           */
       /*                                                                                                                        */
       /* number=26960 street=EKLUTNA RD unit=city=Chugiak district=region=AK postcode=99567 id=coordinates=-149.36039 61.46362  */
       /* number=26636 street=EKLUTNA RD unit=city=Chugiak district=region=AK postcode=99567 id=coordinates=-149.36147 61.46071  */
       /* number=26585 street=EKLUTNA RD unit=city=Chugiak district=region=AK postcode=99567 id=coordinates=-149.36080 61.46061  */
       /* number=26612 street=EKLUTNA RD unit=city=Chugiak district=region=AK postcode=99567 id=coordinates=-149.36163 61.46038  */
       /* number=26527 street=EKLUTNA RD unit=city=Chugiak district=region=AK postcode=99567 id=coordinates=-149.36111 61.46016  */
       /* number=26527 street=EKLUTNA RD unit=city=Chugiak district=region=AK postcode=99567 id=coordinates=-149.36111 61.46016  */
       /* number=26527 street=EKLUTNA RD unit=city=Chugiak district=region=AK postcode=99567 id=coordinates=-149.36111 61.46016  */
       /* number=26527 street=EKLUTNA RD unit=city=Chugiak district=region=AK postcode=99567 id=coordinates=-149.36111 61.46016  */
       /* number=26450 street=EKLUTNA RD unit=city=Chugiak district=region=AK postcode=99567 id=coordinates=-149.36176 61.46000  */
       /* number=26428 street=EKLUTNA RD unit=city=Chugiak district=region=AK postcode=99567 id=coordinates=-149.36276 61.45966  */
       /* number=26439 street=EKLUTNA RD unit=city=Chugiak district=region=AK postcode=99567 id=coordinates=-149.36122 61.45965  */
       /*                                                                                                                        */
       /**************************************************************************************************************************/

       /*----                                                                   ----*/
       /*----  slit into records that have zipcode and records that do not      ----*/
       /*----                                                                   ----*/

       data adr_010&st.NotZip (compress=char)  adr_010&st.HasZip(compress=char) ;
          infile "%sysfunc(pathname(work))/&st..Prep.txt";
          format lat lon 23.16;
          input @;
          if mod(_n_,1000000)=0 then put _n_;
          countequals = countc(_infile_,'=') ;
          if countequals in (0,10,2) then delete;

          if countequals = 9 then               input number=$20. street=$96. unit=$64. city=$64. district=$64. region=$64. postcode=$20. id=$20.  coordinates=$44.;
          else if countc(_infile_,'=') = 8 then input number=$20. street=$96. city=$64. district=$64. region=$64. postcode=$20. hash=$44.          coordinates=$44.;

          if not missing(street);
          lon=input(scan(coordinates,1," "),?? best18.);
          lat=input(scan(coordinates,2," "),?? best18.);
          if length(strip(compress(postcode)))<5 or missing(input(substr(left(postcode),1,5),?? 5.)) then output adr_010&st.NotZip;
          else output adr_010&st.hasZip;
          drop id hash countequals city district unit coordinates;
       run;quit;

       /**************************************************************************************************************************/
       /*                                                                                                                        */
       /*    179,195 RECORDS HAVE ZIPCODE                                                                                        */
       /*                                                                                                                        */
       /*    WORK.ADR_010AKHASZIP total obs=179,195                                                                              */
       /*                                                                                                                        */
       /*     Obs      LAT         LON      NUMBER    STREET                REGION    POSTCODE                                   */
       /*                                                                                                                        */
       /*       1    61.4655    -149.376    26800     EKLUTNA VILLAGE RD      AK       99567                                     */
       /*       2    61.4636    -149.360    26960     EKLUTNA VILLAGE RD      AK       99567                                     */
       /*       3    61.4607    -149.361    26636     EKLUTNA VILLAGE RD      AK       99567                                     */
       /*       4    61.4606    -149.361    26585     EKLUTNA VILLAGE RD      AK       99567                                     */
       /*       5    61.4604    -149.362    26612     EKLUTNA VILLAGE RD      AK       99567                                     */
       /*       6    61.4602    -149.361    26527     EKLUTNA VILLAGE RD      AK       99567                                     */
       /*       7    61.4602    -149.361    26527     EKLUTNA VILLAGE RD      AK       99567                                     */
       /*       8    61.4602    -149.361    26527     EKLUTNA VILLAGE RD      AK       99567                                     */
       /*       9    61.4602    -149.361    26527     EKLUTNA VILLAGE RD      AK       99567                                     */
       /*      10    61.4600    -149.362    26450     EKLUTNA VILLAGE RD      AK       99567                                     */
       /*      11    61.4597    -149.363    26428     EKLUTNA VILLAGE RD      AK       99567                                     */
       /*     ....                                                                                                               */
       /*                                                                                                                        */
       /*                                                                                                                        */
       /*    WE NEED TO ADD ZIP TO THESE 56,119 THAT DO NOT HAVE POSTCODE                                                        */
       /*                                                                                                                        */
       /*                                                                                                                        */
       /*    ADR_010AKNOTZIP total obs=56,119                                                                                    */
       /*                                                                                                                        */
       /*    Obs      LAT         LON      NUMBER    STREET                  REGION    POSTCODE                                  */
       /*                                                                                                                        */
       /*      1    58.4009    -134.587    9582      Whitewater Ct                                                               */
       /*      2    58.4008    -134.586    9578      Whitewater Ct                                                               */
       /*      3    58.5069    -134.771    25150     Glacier Hwy                                                                 */
       /*      4    58.5069    -134.773    25160     Glacier Hwy                                                                 */
       /*      5    58.3836    -134.571    8525      Jennifer Dr                                                                 */
       /*      6    58.3599    -134.550    7630      Glacier Hwy                                                                 */
       /*      7    58.3839    -134.572    3472      Tongass Blvd                                                                */
       /*      8    58.3838    -134.572    3470      Tongass Blvd                                                                */
       /*      9    58.3550    -134.498    5594      Tonsgard Ct                                                                 */
       /*     10    58.3555    -134.499    5711      Concrete Way                                                                */
       /*     11    58.3915    -134.620    11265     Goat Hill Rd                                                                */
       /*     12    58.2557    -134.316    4970      Thane Rd                                                                    */
       /*     13    58.3919    -134.619    11255     Goat Hill Rd                                                                */
       /*     14    58.2930    -134.396    880       S Franklin St                                                               */
       /*     15    58.3920    -134.619    11275     Goat Hill Rd                                                                */
       /*                                                                                                                        */
       /**************************************************************************************************************************/

       /*   _          _               _           _   _
         __| | ___  __| |_   _ _ __   | |__   ___ | |_| |__
        / _` |/ _ \/ _` | | | | `_ \  | `_ \ / _ \| __| `_ \
       | (_| |  __/ (_| | |_| | |_) | | |_) | (_) | |_| | | |
        \__,_|\___|\__,_|\__,_| .__/  |_.__/ \___/ \__|_| |_|
                              |_|
       */

       proc sort data=adr_010&st.haszip out=adr_010&st.haszipunq noequals nodupkey;
         by lon lat number street postcode;
       run;quit;

       proc sort data=adr_010&st.notzip out=adr_010&st.notzipunq noequals nodupkey;
         by lon lat number street postcode;
       run;quit;

       /*   _         _               _       _
        ___(_)_ __   | | ___  _ __   | | __ _| |_
       |_  / | `_ \  | |/ _ \| `_ \  | |/ _` | __|
        / /| | |_) | | | (_) | | | | | | (_| | |_
       /___|_| .__/  |_|\___/|_| |_| |_|\__,_|\__|
             |_|
       */

       proc sort data=adr_010UsPlaceZip (where=(upcase(ADMIN_CODE1) = "&st")) out=adr_010UsPlaceZipSrt force noequals;
          by longitude latitude;
       run;quit;

       /*         _     _       _
         __ _  __| | __| |  ___(_)_ __
        / _` |/ _` |/ _` | |_  / | `_ \
       | (_| | (_| | (_| |  / /| | |_) |
        \__,_|\__,_|\__,_| /___|_| .__/
                                 |_|
        _   _                                     _
       | |_| |__   ___    _____  _____  ___ _   _| |_ ___  _ __
       | __| `_ \ / _ \  / _ \ \/ / _ \/ __| | | | __/ _ \| `__|
       | |_| | | |  __/ |  __/>  <  __/ (__| |_| | || (_) | |
        \__|_| |_|\___|  \___/_/\_\___|\___|\__,_|\__\___/|_|

       */

       /*----                                                                ----*/
       /*---- adds zips to the records without zip 10k records at a time     ----*/
       /*---- if 10k records without zip slice macro is called 10 times      ----*/
       /*----                                                                ----*/


       proc datasets lib=work nolist nodetails;delete adr_010Opn&st.mat; run;quit;

       data _null_;
         retain obs 9999 cnt10k 0 cnt 0;
         set adr_010&st.notzipunq;
         n=_n_-1;
         if mod(n,10000)=0 then do;
            cnt=cnt+1;
            cnt10k =cnt*10000 - 9999;;
            putlog n cnt cnt10k;
            output;
            call execute(cats('%slice(first=',cnt10k,');'));
            *if cnt=3 then stop;
         end;
       run;quit;

       /**************************************************************************************************************************/
       /*                                                                                                                        */
       /*                                                                                                                        */
       /*  Fake Sample WORK.ADR_010OPNAKMAT total obs=56,116                                              MIN DISTANCE MILES     */
       /*                                                                                      BEST       TO NEAREST ZIPCODE     */
       /*   Obs      LAT         LON      NUMBER    STREET            LONGITUDE    LATITUDE    ZIPCODE    GEODIST                */
       /*                                                                                                                        */
       /*  6076    58.4008    -134.559     8311     Gladstone St       -134.593     58.3740     99803      0.2290                */
       /*  6077    58.4008    -134.556     4532     Glacier Spur Rd    -134.593     58.3740     99803      0.3068                */
       /*  6078    58.4008    -134.553     8110     Circle Dr          -134.593     58.3740     99803      0.3756                */
       /*  6079    58.4009    -134.548     4532     Trafalgar Ave      -134.593     58.3740     99803      0.4817                */
       /*  6080    58.4009    -134.549     7946     Gladstone St       -134.593     58.3740     99803      0.4549                */
       /*  6081    58.4009    -134.557     8117     Gladstone St       -134.593     58.3740     99803      0.2848                */
       /*  6082    58.4009    -134.546     4552     Kanat'A St         -134.593     58.3740     99803      0.5306                */
       /*  6083    58.4009    -134.551     8265     Garnet St          -134.593     58.3740     99803      0.4093                */
       /*                                                                                                                        */
       /*  Macro called six times. First call result below                                                                       */
       /*                                                                                                                        */
       /*  NOTE: There were 10000 observations read from the data set WORK.ADR_010OPNAK10001.                                    */
       /*  NOTE: 10000 observations added.                                                                                       */
       /*  NOTE: The data set WORK.ADR_010OPNAKMAT has 20000 observations and 9 variables.                                       */
       /*                                                                                                                        */
       /*  %slice(first=1)                                                                                                       */
       /*  %slice(first=10001)                                                                                                   */
       /*  ...                                                                                                                   */
       /*  %slice(first=50001)                                                                                                   */
       /*                                                                                                                        */
       /*                                                                                                                        */
       /**************************************************************************************************************************/

       /*   _          _                   _                  _     _          _
         __| | ___  __| |_   _ _ __    ___(_)_ __    __ _  __| | __| | ___  __| |
        / _` |/ _ \/ _` | | | | `_ \  |_  / | `_ \  / _` |/ _` |/ _` |/ _ \/ _` |
       | (_| |  __/ (_| | |_| | |_) |  / /| | |_) || (_| | (_| | (_| |  __/ (_| |
        \__,_|\___|\__,_|\__,_| .__/  /___|_| .__/  \__,_|\__,_|\__,_|\___|\__,_|
                              |_|           |_|
       */

       proc sort data=adr_010Opn&st.mat out=adr_010Opn&st.matunq noequals nodupkey;
       by lat lon number street region longitude latitude zipcode geodist;
       run;quit;

       /*                                _            _ _   _        __                   _                     _
         __ _ _ __  _ __   ___ _ __   __| | __      _(_) |_| |__    / /_      _____   ___(_)_ __   ___ ___   __| | ___
        / _` | `_ \| `_ \ / _ \ `_ \ / _` | \ \ /\ / / | __| `_ \  / /\ \ /\ / / _ \ |_  / | `_ \ / __/ _ \ / _` |/ _ \
       | (_| | |_) | |_) |  __/ | | | (_| |  \ V  V /| | |_| | | |/ /  \ V  V / (_) | / /| | |_) | (_| (_) | (_| |  __/
        \__,_| .__/| .__/ \___|_| |_|\__,_|   \_/\_/ |_|\__|_| |_/_/    \_/\_/ \___/ /___|_| .__/ \___\___/ \__,_|\___|
             |_|   |_|                                                                     |_|

       */

       data adr_010opn&st.pre(compress=binary drop=longitude latitude);
         length zipcode $20;
         set
          adr_010opn&st.mat
          adr_010&st.haszip(rename=postcode=zipcode);
       run;quit;

       /*    _
         ___| | ___  __ _ _ __  _   _ _ __
        / __| |/ _ \/ _` | `_ \| | | | `_ \
       | (__| |  __/ (_| | | | | |_| | |_) |
        \___|_|\___|\__,_|_| |_|\__,_| .__/
                                     |_|
       */

       proc sort data=adr_010Opn&st.Pre(where=(not missing(lat))) out=adr_010Opn&st (compress=binary) nodupkey force;
         by lon lat number street region zipcode geodist;
       run;quit;

       proc sql;
         create
            table adr_010Opn&st.fin (compress=binary) as
         select
            number
           ,street
           ,coalesce(upcase(region),"&st")       as region
           ,zipstate(substr(left(zipcode),1,5))  as state
           ,lon
           ,lat
           ,substr(left(zipcode),1,5) as zipcode
           ,geodist
         from
           adr_010Opn&st
         where
             /* Street has only suffix                                       ----*/
             strip(street) not in ("RD","DR","TRL","ST","AVE");
       ;quit;


       /**************************************************************************************************************************/
       /*                                                                                                                        */
       /*   OPN.ADR_010OPNAKFIN total obs=199,485                                                                                */
       /*                                                                                                                        */
       /*    NUMBER  STREET              REGION    STATE        LON          LAT         ZIPCODE    GEODIST                      */
       /*                                                                                                                        */
       /*    11560   SALMON CREEK RD       AK       AK     -149.4109651     60.1456335    99664       .     ==> had zip          */
       /*    928     TIMBERLINE DR         AK       AK     -149.1286352     60.9514255    99587       .                          */
       /*    429     TIMBERLINE DR         AK       AK     -149.1242314     60.9568488    99587       .                          */
       /*    2695    WAUGSTROE DR          AK       AK     -147.9152493     64.8256954    99706      1.1616 ==> computed zip     */
       /*    753     HAMAN ST              AK       AK     -147.9126111     64.8288836    99706      1.0057                      */
       /*    2395    OLD CHENA RIDGE RD    AK       AK     -147.8849203     64.8340975    99706      0.1540                      */
       /*    1110    MILLER HILL RD EXT    AK       AK     -147.8793055     64.9089600    99708      2.3318                      */
       /*    600     AUDREY DR             AK       AK     -147.8664694     64.8409647    99706      0.6688                      */
       /*    300     WOODRIDGE ST          AK       AK     -147.8614301     64.8456866    99706      1.0225                      */
       /*    4004    FAHRENKAMP AVE        AK       AK     -147.8230478     64.8459548    99706      1.8987                      */
       /*    3005    MACK BLVD             AK       AK     -147.7893995     64.8592060    99790      2.2273                      */
       /*    2750    TALKEETNA AVE         AK       AK     -147.7848119     64.8399917    99790      1.7998                      */
       /*    114     CHIEF EVAN DR         AK       AK     -147.7843368     64.8442733    99790      1.7760                      */
       /*    1740    BRIDGEWATER DR        AK       AK     -147.7604428     64.8603017    99790      1.6076                      */
       /*    1648    BRIDGEWATER DR        AK       AK     -147.7558671     64.8592432    99790      1.4640                      */
       /*                                                                                                                        */
       /**************************************************************************************************************************/

    %mend sts;

    %sts(AK);
    %sts(AL);
    %sts(AR);
    %sts(AZ);
    %sts(CA);
    %sts(CO);
    %sts(CT);
    %sts(DC);
    %sts(DE);
    %sts(FL);
    %sts(GA);
    %sts(HI);
    %sts(IA);
    %sts(ID);
    %sts(IL);
    %sts(IN);
    %sts(KS);
    %sts(KY);
    %sts(LA);
    %sts(MA);
    %sts(MD);
    %sts(ME);
    %sts(MI);
    %sts(MN);
    %sts(MO);
    %sts(MS);
    %sts(MT);
    %sts(NC);
    %sts(ND);
    %sts(NE);
    %sts(NH);
    %sts(NJ);
    %sts(NM);
    %sts(NV);
    %sts(NY);
    %sts(OH);
    %sts(OK);
    %sts(OR);
    %sts(PA);
    %sts(RI);
    %sts(SC);
    %sts(SD);
    %sts(TN);
    %sts(UT);
    %sts(VA);
    %sts(VT);
    %sts(WA);
    %sts(WI);
    %sts(WV);
    %sts(WY);
    %sts(TX);

    /*                                                        __ _
     _ __ ___   __ _  ___ _ __ ___     ___  _ __   ___ _ __  / _(_)_  __
    | `_ ` _ \ / _` |/ __| `__/ _ \   / _ \| `_ \ / _ \ `_ \| |_| \ \/ /
    | | | | | | (_| | (__| | | (_) | | (_) | |_) |  __/ | | |  _| |>  <
    |_| |_| |_|\__,_|\___|_|  \___/   \___/| .__/ \___|_| |_|_| |_/_/\_\
                                           |_|
          _                              _
      ___| | ___  __ _ _ __     __ _  __| |_ __
     / __| |/ _ \/ _` | `_ \   / _` |/ _` | `__|
    | (__| |  __/ (_| | | | | | (_| | (_| | |
     \___|_|\___|\__,_|_| |_|  \__,_|\__,_|_|

    */

    %macro opnfix(st);

        /* %let st=AK;  */

        options obs=max compress=char;

        /*    _                        _       _
          ___| | ___  __ _ _ __    ___| | __ _| |_ ___
         / __| |/ _ \/ _` | `_ \  / __| |/ _` | __/ _ \
        | (__| |  __/ (_| | | | | \__ \ | (_| | ||  __/
         \___|_|\___|\__,_|_| |_| |___/_|\__,_|\__\___|

        */

        proc datasets lib=work nolist nodetails;
           delete adr_010opn&st.fix
                  adr_010opn&st.err;
        run;quit;

        proc datasets lib=work nolist nodetails;
           delete adr_010opnfix
                  adr_010opngeo
                  adr_010opnkx
                  &st.fix
                  &st.err
                  ;
        run;quit;

        /*   _                  _               _ _
         ___| |_ __ _ _ __   __| | __ _ _ __ __| (_)_______
        / __| __/ _` | `_ \ / _` |/ _` | `__/ _` | |_  / _ \
        \__ \ || (_| | | | | (_| | (_| | | | (_| | |/ /  __/
        |___/\__\__,_|_| |_|\__,_|\__,_|_|  \__,_|_/___\___|

        */

        data
           adr_010opnfix (compress=binary keep=flg adr street zipcode zipstate geodist lon lat)
           opnfix.adr_010opn&st.err (compress=binary keep=flg adr street zipcode zipstate geodist lon lat)
          ;

        retain zro 0 number num street adr zipcode zipstate lon lat geodist;

        set
         adr_010opn&st.fin (where=(length(strip(street))>1));
        ;

        flg="K1";

        if anyalpha(street)=0 then flg="D1";

        if scan(strip(street),-1,' ') in ('NW','NE','SE','SW') then do;
           enx=scan(strip(street),-1,' ');
           adr=trim(substr(street,1,length(street)-3));
           len=length(strip(scan(adr,-1,' ')));
           sfx=strip(scan(adr,-1,' '));
           select;
              %include "%sysfunc(pathname(work))/adr_010sfx.sas";
              otherwise;
           end;
           street=catx(' ',adr,enx);
        end;

        if missing(geodist) then geodist=99999;

        if not (length(compress(zipcode))<5 or missing(input(substr(left(zipcode),1,5),?? 5.))) then zipcode=substr(left(zipcode),1,5);

        if input(zipcode,5.)=0 then flg="D2";;

        zipstate = zipstate(zipcode);

        adr=left(compbl(compress(upcase(street),'','kads')));

        if missing(adr) then flg="D3";
        num=compress(upcase(number),'','kad');

        if missing(lat) or missing(lon) or lat=0 or lon=0 then flg="D4";

        if num=""  or input(num,?? best.) = 0  then num='2';

        possiblenum=strip(scan(adr,1,' '));
        if possiblenum=num then adr=substr(adr,length(possiblenum)+1);

        adr=tranwrd(adr,'NORTH','N');
        adr=tranwrd(adr,'SOUTH','S');
        adr=tranwrd(adr,'EAST','E');
        adr=tranwrd(adr,'WEST','W');
        adr=tranwrd(adr,'NORTHEAST','NE');
        adr=tranwrd(adr,'SOUTHEAST','SE');
        adr=tranwrd(adr,'NORTHWEST','NW');
        adr=tranwrd(adr,'SOUTHWEST','SW');

        zipstate = zipstate(zipcode);

        adr=catx(' ',num,adr);

        if anydigit(scan(adr,1,' '))=0 then adr=catx(' ','2',adr);

        adr=left(catx(' ',adr,zipcode));

        adr = substr(adr,verify(adr,'0'));

        /*                                               _ _
         _ __ ___ _ __ ___   _____   _____   _   _ _ __ (_) |_ ___
        | `__/ _ \ `_ ` _ \ / _ \ \ / / _ \ | | | | `_ \| | __/ __|
        | | |  __/ | | | | | (_) \ V /  __/ | |_| | | | | | |_\__ \
        |_|  \___|_| |_| |_|\___/ \_/ \___|  \__,_|_| |_|_|\__|___/

        */
        %unit(APT);
        %unit(UNIT);
        %unit(STE);
        %unit(BLDG);

        /*--- stadardize internal words using appreviatons                  ----*/

        %inc "%sysfunc(pathname(work))/adr_010sx1.sas"; /*----internal word ----*/

        /*---- nebraska street variable already has city state and zipcode  ----*/
        if "&st" = "NE" then do;
          select;
            when (index(adr,'NE 68') or index(adr,'NE 69')) do;
                if input(scan(adr,-1,' '),?? best.)>67999 and input(scan(adr,-2,' '),?? best.)>67999 then do;
                  *put 'BEF ' adr=;
                  call scan(adr,-4,pos,len);
                  adr = catx(' ',substr(adr,1,pos-1),zipcode);
                  *put pos= len=;
                  *put 'AFT ' adr= //;
                  flg="Z1";
                end;
            end;
            otherwise;
            end;
        end;

        if flg =:"D" then output opnfix.adr_010opn&st.err;
        else output adr_010opnfix;

        keep flg zro number num street adr zipcode  zipstate lon lat geodist;

        run;quit;


        /**************************************************************************************************************************/
        /*                        _                                                                                               */
        /*   __ _  ___   ___   __| |                                                                                              */
        /*  / _` |/ _ \ / _ \ / _` |                                                                                              */
        /* | (_| | (_) | (_) | (_| |                                                                                              */
        /*  \__, |\___/ \___/ \__,_|                                                                                              */
        /*  |___/                                                                                                                 */
        /*                                                                                                                        */
        /*  obs from ADR_010OPNFIX total obs=199,484 22NOV2023:10:54:30                                                           */
        /*    STREET              ADR             ZIPCODE    ZIPSTATE       LON        LAT      GEODIST    FLG                    */
        /*                                                                                                                        */
        /*   Carmel St    128 CARMEL ST 99608      99608        AK       -154.434    57.5631     1.6163    K1                     */
        /*   First St     1308 FIRST ST 99608      99608        AK       -154.002    57.5308    17.7520    K1                     */
        /*   First St     1304 FIRST ST 99608      99608        AK       -154.001    57.5309    17.7889    K1                     */
        /*   First St     1300 FIRST ST 99608      99608        AK       -154.000    57.5309    17.8408    K1                     */
        /*   First St     1210 FIRST ST 99608      99608        AK       -153.998    57.5311    17.8920    K1                     */
        /*   First St     1209 FIRST ST 99608      99608        AK       -153.997    57.5306    17.9341    K1                     */
        /*   First St     1207 FIRST ST 99608      99608        AK       -153.997    57.5309    17.9461    K1                     */
        /*   First St     1205 FIRST ST 99608      99608        AK       -153.997    57.5311    17.9604    K1                     */
        /*   Second St    1208 SECOND ST 99608     99608        AK       -153.996    57.5300    17.9820    K1                     */
        /*   First St     1203 FIRST ST 99608      99608        AK       -153.996    57.5314    17.9741    K1                     */
        /*   Second St    1206 SECOND ST 99608     99608        AK       -153.996    57.5302    17.9971    K1                     */
        /*   First St     1201 FIRST ST 99608      99608        AK       -153.996    57.5317    17.9912    K1                     */
        /*   Second St    1204 SECOND ST 99608     99608        AK       -153.995    57.5305    18.0128    K1                     */
        /*                                                                                                                        */
        /*  _               _                                                                                                     */
        /* | |__   __ _  __| |                                                                                                    */
        /* | `_ \ / _` |/ _` |                                                                                                    */
        /* | |_) | (_| | (_| |                                                                                                    */
        /* |_.__/ \__,_|\__,_|                                                                                                    */
        /*                                                                                                                        */
        /*  D1 street only has numbers                                                                                            */
        /*  D2 00000 zipcode                                                                                                      */
        /*  GEODIS=99999 means that original zipcode present. No need to compute zipcode.                                         */
        /*                                                                                                                        */
        /* ADR_010OPNTXERR total obs=2,444,738                                                                                    */
        /*                                                                                                                        */
        /*   STREET            ADR                       ZIPCODE    ZIPSTATE       LON        LAT       GEODIST    FLG            */
        /*                                                                                                                        */
        /*   401               401 78834                  78834        TX       -99.8519    28.5216    99999.00    D1             */
        /*   469               2 469 78017                78017        TX       -99.0639    28.5397        3.07    D1             */
        /*   469               2 469 78014                78014        TX       -99.0608    28.5294        2.94    D1             */
        /*   469               2 469 78017                78017        TX       -99.0548    28.5423        3.28    D1             */
        /*   624               2 624 78021                78021        TX       -99.0248    28.4584        2.72    D1             */
        /*   649               4 649 78361                78361        TX       -98.8856    27.0769    99999.00    D1             */
        /*   16539             16539 78253                78253        TX       -98.8255    29.4338        5.05    D1             */
        /*   97                2 97 78021                 78021        TX       -98.8103    28.4658        4.67    D1             */
        /*   107               219 107 78253              78253        TX       -98.8076    29.4306        4.15    D1             */
        /*   117               405 117 78643              78643        TX       -98.7196    30.7154    99999.00    D1             */
        /*   INDIANA STREET    811 INDIANA ST 00000       00000                 -97.6762    27.7889    99999.00    D2             */
        /*   CR 28             2474 CRK 28 00000          00000                 -97.5783    27.7180    99999.00    D2             */
        /*                                                                                                                        */
        /**************************************************************************************************************************/

        /*   _          _
          __| | ___  __| |_   _ _ __
         / _` |/ _ \/ _` | | | | `_ \
        | (_| |  __/ (_| | |_| | |_) |
         \__,_|\___|\__,_|\__,_| .__/
                               |_|
        */
        proc sort data=adr_010opnfix out=adr_010opngeo nodupkey;
        by adr geodist ;
        run;quit;

        /*                                    _                   _          _                     _
         ___  __ _ _ __ ___   ___    __ _  __| |_ __    __ _  ___| |_    ___| | ___  ___  ___  ___| |_
        / __|/ _` | `_ ` _ \ / _ \  / _` |/ _` | `__|  / _` |/ _ \ __|  / __| |/ _ \/ __|/ _ \/ __| __|
        \__ \ (_| | | | | | |  __/ | (_| | (_| | |    | (_| |  __/ |_  | (__| | (_) \__ \  __/\__ \ |_
        |___/\__,_|_| |_| |_|\___|  \__,_|\__,_|_|     \__, |\___|\__|  \___|_|\___/|___/\___||___/\__|
                                                       |___/
        */

        data  opnfix.adr_010opn&st.fix adr_010opnkx;
          set adr_010opngeo;
          by adr;
          if first.adr then do; flg="KP"; output opnfix.adr_010opn&st.fix;end;
          else do;  flg="KX"; output adr_010opnkx; end;
        run;quit;

        /*                    __ _             _       _        _                  _
          __ _ _ __  _ __    / _(_)_ __   __ _| |  ___| |_ __ _| |_ ___   __ _  __| |_ __ ___
         / _` | `_ \| `_ \  | |_| | `_ \ / _` | | / __| __/ _` | __/ _ \ / _` |/ _` | `__/ __|
        | (_| | |_) | | | | |  _| | | | | (_| | | \__ \ || (_| | ||  __/  (_| | (_| | |  \__ \
         \__,_| .__/|_| |_| |_| |_|_| |_|\__,_|_| |___/\__\__,_|\__\___| \__,_|\__,_|_|  |___/
              |_|
        */

        proc append base=opnfix.adr_010opn&st.err data=adr_010opnkx;
        run;quit;

    %mend opnfix;

    %symdel st /nowarn;


    %opnfix(AK);
    %opnfix(AL);
    %opnfix(AR);
    %opnfix(AZ);
    %opnfix(CA);
    %opnfix(CO);
    %opnfix(CT);
    %opnfix(DC);
    %opnfix(DE);
    %opnfix(FL);
    %opnfix(GA);
    %opnfix(HI);
    %opnfix(IA);
    %opnfix(ID);
    %opnfix(IL);
    %opnfix(IN);
    %opnfix(KS);
    %opnfix(KY);
    %opnfix(LA);
    %opnfix(MA);
    %opnfix(MD);
    %opnfix(ME);
    %opnfix(MI);
    %opnfix(MN);
    %opnfix(MO);
    %opnfix(MS);
    %opnfix(MT);
    %opnfix(NC);
    %opnfix(ND);
    %opnfix(NE);
    %opnfix(NH);
    %opnfix(NJ);
    %opnfix(NM);
    %opnfix(NV);
    %opnfix(NY);
    %opnfix(OH);
    %opnfix(OK);
    %opnfix(OR);
    %opnfix(PA);
    %opnfix(RI);
    %opnfix(SC);
    %opnfix(SD);
    %opnfix(TN);
    %opnfix(UT);
    %opnfix(VA);
    %opnfix(VT);
    %opnfix(WA);
    %opnfix(WI);
    %opnfix(WV);
    %opnfix(WY);
    %opnfix(TX);


    /*                         _
      ___ ___  _ __   ___ __ _| |_    ___  _ __  _ __
     / __/ _ \| `_ \ / __/ _` | __|  / _ \| `_ \| `_ \
    | (_| (_) | | | | (_| (_| | |_  | (_) | |_) | | | |
     \___\___/|_| |_|\___\__,_|\__|  \___/| .__/|_| |_|
                                          |_|
    */


    options obs=max;

    data opn.adr_010opnSta(compress=char);

    length
      FRO         $2.
      STREET      $96.
      ADR         $96.
      ZIPCODE     $5.
      ZIPSTATE    $5.
      FLG         $2.
    ;
    set
    OPN.ADR_010OPNAKFIX (in= AK1 )
    OPN.ADR_010OPNALFIX (in= AL1 )
    OPN.ADR_010OPNARFIX (in= AR1 )
    OPN.ADR_010OPNAZFIX (in= AZ1 )
    OPN.ADR_010OPNCAFIX (in= CA1 )
    OPN.ADR_010OPNCOFIX (in= CO1 )
    OPN.ADR_010OPNCTFIX (in= CT1 )
    OPN.ADR_010OPNDCFIX (in= DC1 )
    OPN.ADR_010OPNDEFIX (in= DE1 )
    OPN.ADR_010OPNFLFIX (in= FL1 )
    OPN.ADR_010OPNGAFIX (in= GA1 )
    OPN.ADR_010OPNHIFIX (in= HI1 )
    OPN.ADR_010OPNIAFIX (in= IA1 )
    OPN.ADR_010OPNIDFIX (in= ID1 )
    OPN.ADR_010OPNILFIX (in= IL1 )
    OPN.ADR_010OPNINFIX (in= IN1 )
    OPN.ADR_010OPNKSFIX (in= KS1 )
    OPN.ADR_010OPNKYFIX (in= KY1 )
    OPN.ADR_010OPNLAFIX (in= LA1 )
    OPN.ADR_010OPNMAFIX (in= MA1 )
    OPN.ADR_010OPNMDFIX (in= MD1 )
    OPN.ADR_010OPNMEFIX (in= ME1 )
    OPN.ADR_010OPNMIFIX (in= MI1 )
    OPN.ADR_010OPNMNFIX (in= MN1 )
    OPN.ADR_010OPNMOFIX (in= MO1 )
    OPN.ADR_010OPNMSFIX (in= MS1 )
    OPN.ADR_010OPNMTFIX (in= MT1 )
    OPN.ADR_010OPNNCFIX (in= NC1 )
    OPN.ADR_010OPNNDFIX (in= ND1 )
    OPN.ADR_010OPNNEFIX (in= NE1 )
    OPN.ADR_010OPNNHFIX (in= NH1 )
    OPN.ADR_010OPNNJFIX (in= NJ1 )
    OPN.ADR_010OPNNMFIX (in= NM1 )
    OPN.ADR_010OPNNVFIX (in= NV1 )
    OPN.ADR_010OPNNYFIX (in= NY1 )
    OPN.ADR_010OPNOHFIX (in= OH1 )
    OPN.ADR_010OPNOKFIX (in= OK1 )
    OPN.ADR_010OPNORFIX (in= OR1 )
    OPN.ADR_010OPNPAFIX (in= PA1 )
    OPN.ADR_010OPNRIFIX (in= RI1 )
    OPN.ADR_010OPNSDFIX (in= SD1 )
    OPN.ADR_010OPNSCFIX (in= SC1 )
    OPN.ADR_010OPNTNFIX (in= TN1 )
    OPN.ADR_010OPNTXFIX (in= TX1 )
    OPN.ADR_010OPNUTFIX (in= UT1 )
    OPN.ADR_010OPNVAFIX (in= VA1 )
    OPN.ADR_010OPNVTFIX (in= VT1 )
    OPN.ADR_010OPNWAFIX (in= WA1 )
    OPN.ADR_010OPNWIFIX (in= WI1 )
    OPN.ADR_010OPNWVFIX (in= WV1 )
    OPN.ADR_010OPNWYFIX (in= WY1 )
    ;

    adr=tranwrd(adr,' DRIVE ',' DR ');
    if length(strip(zipcode))=5;

    ;select;
    when ( AK1 ) fro= "AK" ;
    when ( AL1 ) fro= "AL" ;
    when ( AR1 ) fro= "AR" ;
    when ( AZ1 ) fro= "AZ" ;
    when ( CA1 ) fro= "CA" ;
    when ( CO1 ) fro= "CO" ;
    when ( CT1 ) fro= "CT" ;
    when ( DC1 ) fro= "DC" ;
    when ( DE1 ) fro= "DE" ;
    when ( FL1 ) fro= "FL" ;
    when ( GA1 ) fro= "GA" ;
    when ( HI1 ) fro= "HI" ;
    when ( IA1 ) fro= "IA" ;
    when ( ID1 ) fro= "ID" ;
    when ( IL1 ) fro= "IL" ;
    when ( IN1 ) fro= "IN" ;
    when ( KS1 ) fro= "KS" ;
    when ( KY1 ) fro= "KY" ;
    when ( LA1 ) fro= "LA" ;
    when ( MA1 ) fro= "MA" ;
    when ( MD1 ) fro= "MD" ;
    when ( ME1 ) fro= "ME" ;
    when ( MI1 ) fro= "MI" ;
    when ( MN1 ) fro= "MN" ;
    when ( MO1 ) fro= "MO" ;
    when ( MS1 ) fro= "MS" ;
    when ( MT1 ) fro= "MT" ;
    when ( NC1 ) fro= "NC" ;
    when ( ND1 ) fro= "ND" ;
    when ( NE1 ) fro= "NE" ;
    when ( NH1 ) fro= "NH" ;
    when ( NJ1 ) fro= "NJ" ;
    when ( NM1 ) fro= "NM" ;
    when ( NV1 ) fro= "NV" ;
    when ( NY1 ) fro= "NY" ;
    when ( OH1 ) fro= "OH" ;
    when ( OK1 ) fro= "OK" ;
    when ( OR1 ) fro= "OR" ;
    when ( PA1 ) fro= "PA" ;
    when ( RI1 ) fro= "RI" ;
    when ( SD1 ) fro= "SD" ;
    when ( SC1 ) fro= "SC" ;
    when ( TN1 ) fro= "TN" ;
    when ( TX1 ) fro= "TX" ;
    when ( UT1 ) fro= "UT" ;
    when ( VA1 ) fro= "VA" ;
    when ( VT1 ) fro= "VT" ;
    when ( WA1 ) fro= "WA" ;
    when ( WI1 ) fro= "WI" ;
    when ( WV1 ) fro= "WV" ;
    when ( WY1 ) fro= "WY" ;
    otherwise fro="ER";
    end;

    zipstate=coalescec(fro,zipstate);

    run;quit

    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */
