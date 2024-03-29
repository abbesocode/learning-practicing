Kaggle Automobile Regression Practice
================
me
6/30/2022

############################################################ 

\##About Data Set (from Kaggle)
<https://www.kaggle.com/datasets/toramky/automobile-dataset?resource=download>

Context This dataset consist of data From 1985 Ward’s Automotive
Yearbook. Here are the sources

Sources:

1)  1985 Model Import Car and Truck Specifications, 1985 Ward’s
    Automotive Yearbook.
2)  Personal Auto Manuals, Insurance Services Office, 160 Water Street,
    New York, NY 10038
3)  Insurance Collision Report, Insurance Institute for Highway Safety,
    Watergate 600, Washington, DC 20037

Content This data set consists of three types of entities: (a) the
specification of an auto in terms of various characteristics, (b) its
assigned insurance risk rating, (c) its normalized losses in use as
compared to other cars. The second rating corresponds to the degree to
which the auto is more risky than its price indicates. Cars are
initially assigned a risk factor symbol associated with its price. Then,
if it is more risky (or less), this symbol is adjusted by moving it up
(or down) the scale. Actuarians call this process “symboling”. A value
of +3 indicates that the auto is risky, -3 that it is probably pretty
safe.

The third factor is the relative average loss payment per insured
vehicle year. This value is normalized for all autos within a particular
size classification (two-door small, station wagons, sports/speciality,
etc…), and represents the average loss per car per year.

Note: Several of the attributes in the database could be used as a
“class” attribute.

================================================================================
\#Objective/Problem Definition

The data contains mainly information on the car engineering, but what is
more interesting to me is:

2)  its assigned insurance risk rating (symboling)

# Using the engineering information my objective will be to explain as much of the variation in losses and risk rating as possible with a linear regresssion model.

===============================================================================
\# 1. Setup
===============================================================================

## 1.1) Download data

Source:
<https://www.kaggle.com/datasets/toramky/automobile-dataset?resource=download>

``` r
library(psych)
auto_data= read.csv("Automobile_data.csv")
```

## 1.2) Load and describe raw data

``` r
describe(auto_data)
```

    ##                    vars   n    mean     sd median trimmed    mad    min    max
    ## make*                 1 205   13.20   6.27   13.0   13.48   8.90    1.0   22.0
    ## symboling             2 205    0.83   1.25    1.0    0.81   1.48   -2.0    3.0
    ## normalized.losses*    3 205   21.45  16.99   20.0   20.52  23.72    1.0   52.0
    ## fuel.type*            4 205    1.90   0.30    2.0    2.00   0.00    1.0    2.0
    ## aspiration*           5 205    1.18   0.39    1.0    1.10   0.00    1.0    2.0
    ## num.of.doors*         6 205    2.42   0.51    2.0    2.42   0.00    1.0    3.0
    ## body.style*           7 205    3.61   0.86    4.0    3.64   1.48    1.0    5.0
    ## drive.wheels*         8 205    2.33   0.56    2.0    2.34   0.00    1.0    3.0
    ## engine.location*      9 205    1.01   0.12    1.0    1.00   0.00    1.0    2.0
    ## wheel.base           10 205   98.76   6.02   97.0   98.08   4.00   86.6  120.9
    ## length               11 205  174.05  12.34  173.2  173.79  10.23  141.1  208.1
    ## width                12 205   65.91   2.15   65.5   65.66   2.08   60.3   72.3
    ## height               13 205   53.72   2.44   54.1   53.70   2.37   47.8   59.8
    ## curb.weight          14 205 2555.57 520.68 2414.0 2513.05 572.28 1488.0 4066.0
    ## engine.type*         15 205    4.01   1.05    4.0    4.04   0.00    1.0    7.0
    ## num.of.cylinders*    16 205    3.12   0.80    3.0    3.06   0.00    1.0    7.0
    ## engine.size          17 205  126.91  41.64  120.0  120.58  34.10   61.0  326.0
    ## fuel.system*         18 205    4.25   2.01    6.0    4.32   1.48    1.0    8.0
    ## bore*                19 205   19.58  10.33   18.0   19.44  13.34    1.0   39.0
    ## stroke*              20 205   20.83   9.04   23.0   21.23   8.90    1.0   37.0
    ## compression.ratio    21 205   10.14   3.97    9.0    9.04   0.59    7.0   23.0
    ## horsepower*          22 205   33.14  18.65   40.0   33.65  22.24    1.0   60.0
    ## peak.rpm*            23 205   13.37   5.34   14.0   13.51   5.93    1.0   24.0
    ## city.mpg             24 205   25.22   6.54   24.0   24.76   7.41   13.0   49.0
    ## highway.mpg          25 205   30.75   6.89   30.0   30.40   7.41   16.0   54.0
    ## price*               26 205   95.00  54.83   97.0   95.53  71.16    1.0  187.0
    ##                     range  skew kurtosis    se
    ## make*                21.0 -0.24    -1.20  0.44
    ## symboling             5.0  0.21    -0.71  0.09
    ## normalized.losses*   51.0  0.28    -1.28  1.19
    ## fuel.type*            1.0 -2.69     5.28  0.02
    ## aspiration*           1.0  1.65     0.72  0.03
    ## num.of.doors*         2.0  0.09    -1.49  0.04
    ## body.style*           4.0 -0.66     0.93  0.06
    ## drive.wheels*         2.0 -0.06    -0.71  0.04
    ## engine.location*      1.0  8.02    62.70  0.01
    ## wheel.base           34.3  1.03     0.92  0.42
    ## length               67.0  0.15    -0.14  0.86
    ## width                12.0  0.89     0.62  0.15
    ## height               12.0  0.06    -0.49  0.17
    ## curb.weight        2578.0  0.67    -0.10 36.37
    ## engine.type*          6.0 -0.53     3.13  0.07
    ## num.of.cylinders*     6.0  2.11    10.66  0.06
    ## engine.size         265.0  1.92     5.07  2.91
    ## fuel.system*          7.0 -0.24    -1.66  0.14
    ## bore*                38.0  0.13    -1.15  0.72
    ## stroke*              36.0 -0.39    -0.72  0.63
    ## compression.ratio    16.0  2.57     5.00  0.28
    ## horsepower*          59.0 -0.28    -1.37  1.30
    ## peak.rpm*            23.0 -0.20    -0.46  0.37
    ## city.mpg             36.0  0.65     0.50  0.46
    ## highway.mpg          38.0  0.53     0.37  0.48
    ## price*              186.0 -0.07    -1.24  3.83

``` r
head(auto_data)
```

    ##          make symboling normalized.losses fuel.type aspiration num.of.doors
    ## 1 alfa-romero         3                 ?       gas        std          two
    ## 2 alfa-romero         3                 ?       gas        std          two
    ## 3 alfa-romero         1                 ?       gas        std          two
    ## 4        audi         2               164       gas        std         four
    ## 5        audi         2               164       gas        std         four
    ## 6        audi         2                 ?       gas        std          two
    ##    body.style drive.wheels engine.location wheel.base length width height
    ## 1 convertible          rwd           front       88.6  168.8  64.1   48.8
    ## 2 convertible          rwd           front       88.6  168.8  64.1   48.8
    ## 3   hatchback          rwd           front       94.5  171.2  65.5   52.4
    ## 4       sedan          fwd           front       99.8  176.6  66.2   54.3
    ## 5       sedan          4wd           front       99.4  176.6  66.4   54.3
    ## 6       sedan          fwd           front       99.8  177.3  66.3   53.1
    ##   curb.weight engine.type num.of.cylinders engine.size fuel.system bore stroke
    ## 1        2548        dohc             four         130        mpfi 3.47   2.68
    ## 2        2548        dohc             four         130        mpfi 3.47   2.68
    ## 3        2823        ohcv              six         152        mpfi 2.68   3.47
    ## 4        2337         ohc             four         109        mpfi 3.19    3.4
    ## 5        2824         ohc             five         136        mpfi 3.19    3.4
    ## 6        2507         ohc             five         136        mpfi 3.19    3.4
    ##   compression.ratio horsepower peak.rpm city.mpg highway.mpg price
    ## 1               9.0        111     5000       21          27 13495
    ## 2               9.0        111     5000       21          27 16500
    ## 3               9.0        154     5000       19          26 16500
    ## 4              10.0        102     5500       24          30 13950
    ## 5               8.0        115     5500       18          22 17450
    ## 6               8.5        110     5500       19          25 15250

``` r
unique(auto_data$make)
```

    ##  [1] "alfa-romero"   "audi"          "bmw"           "chevrolet"    
    ##  [5] "dodge"         "honda"         "isuzu"         "jaguar"       
    ##  [9] "mazda"         "mercedes-benz" "mercury"       "mitsubishi"   
    ## [13] "nissan"        "peugot"        "plymouth"      "porsche"      
    ## [17] "renault"       "saab"          "subaru"        "toyota"       
    ## [21] "volkswagen"    "volvo"

``` r
length(auto_data$make)
```

    ## [1] 205

``` r
str(auto_data)
```

    ## 'data.frame':    205 obs. of  26 variables:
    ##  $ make             : chr  "alfa-romero" "alfa-romero" "alfa-romero" "audi" ...
    ##  $ symboling        : int  3 3 1 2 2 2 1 1 1 0 ...
    ##  $ normalized.losses: chr  "?" "?" "?" "164" ...
    ##  $ fuel.type        : chr  "gas" "gas" "gas" "gas" ...
    ##  $ aspiration       : chr  "std" "std" "std" "std" ...
    ##  $ num.of.doors     : chr  "two" "two" "two" "four" ...
    ##  $ body.style       : chr  "convertible" "convertible" "hatchback" "sedan" ...
    ##  $ drive.wheels     : chr  "rwd" "rwd" "rwd" "fwd" ...
    ##  $ engine.location  : chr  "front" "front" "front" "front" ...
    ##  $ wheel.base       : num  88.6 88.6 94.5 99.8 99.4 ...
    ##  $ length           : num  169 169 171 177 177 ...
    ##  $ width            : num  64.1 64.1 65.5 66.2 66.4 66.3 71.4 71.4 71.4 67.9 ...
    ##  $ height           : num  48.8 48.8 52.4 54.3 54.3 53.1 55.7 55.7 55.9 52 ...
    ##  $ curb.weight      : int  2548 2548 2823 2337 2824 2507 2844 2954 3086 3053 ...
    ##  $ engine.type      : chr  "dohc" "dohc" "ohcv" "ohc" ...
    ##  $ num.of.cylinders : chr  "four" "four" "six" "four" ...
    ##  $ engine.size      : int  130 130 152 109 136 136 136 136 131 131 ...
    ##  $ fuel.system      : chr  "mpfi" "mpfi" "mpfi" "mpfi" ...
    ##  $ bore             : chr  "3.47" "3.47" "2.68" "3.19" ...
    ##  $ stroke           : chr  "2.68" "2.68" "3.47" "3.4" ...
    ##  $ compression.ratio: num  9 9 9 10 8 8.5 8.5 8.5 8.3 7 ...
    ##  $ horsepower       : chr  "111" "111" "154" "102" ...
    ##  $ peak.rpm         : chr  "5000" "5000" "5000" "5500" ...
    ##  $ city.mpg         : int  21 21 19 24 18 19 19 19 17 16 ...
    ##  $ highway.mpg      : int  27 27 26 30 22 25 25 25 20 22 ...
    ##  $ price            : chr  "13495" "16500" "16500" "13950" ...

car makes: “alfa-romero” “audi” “bmw” “chevrolet”  
“dodge” “honda” “isuzu” “jaguar”  
“mazda” “mercedes-benz” “mercury” “mitsubishi”  
“nissan” “peugot” “plymouth” “porsche”  
“renault” “saab” “subaru” “toyota”  
“volkswagen” “volvo”

===============================================================================
\#2. Transforming Data
===============================================================================

\##Transforming and recognizing missing valuesauto_data.c= auto_data

``` r
#Turning 'columns'?' to NA
auto_data[auto_data == '?'] <- NA
auto_data.c= auto_data

#Converting data to proper data types
type.convert(auto_data.c)
```

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ##              make symboling normalized.losses fuel.type aspiration num.of.doors
    ## 1     alfa-romero         3                NA       gas        std          two
    ## 2     alfa-romero         3                NA       gas        std          two
    ## 3     alfa-romero         1                NA       gas        std          two
    ## 4            audi         2               164       gas        std         four
    ## 5            audi         2               164       gas        std         four
    ## 6            audi         2                NA       gas        std          two
    ## 7            audi         1               158       gas        std         four
    ## 8            audi         1                NA       gas        std         four
    ## 9            audi         1               158       gas      turbo         four
    ## 10           audi         0                NA       gas      turbo          two
    ## 11            bmw         2               192       gas        std          two
    ## 12            bmw         0               192       gas        std         four
    ## 13            bmw         0               188       gas        std          two
    ## 14            bmw         0               188       gas        std         four
    ## 15            bmw         1                NA       gas        std         four
    ## 16            bmw         0                NA       gas        std         four
    ## 17            bmw         0                NA       gas        std          two
    ## 18            bmw         0                NA       gas        std         four
    ## 19      chevrolet         2               121       gas        std          two
    ## 20      chevrolet         1                98       gas        std          two
    ## 21      chevrolet         0                81       gas        std         four
    ## 22          dodge         1               118       gas        std          two
    ## 23          dodge         1               118       gas        std          two
    ## 24          dodge         1               118       gas      turbo          two
    ## 25          dodge         1               148       gas        std         four
    ## 26          dodge         1               148       gas        std         four
    ## 27          dodge         1               148       gas        std         four
    ## 28          dodge         1               148       gas      turbo         <NA>
    ## 29          dodge        -1               110       gas        std         four
    ## 30          dodge         3               145       gas      turbo          two
    ## 31          honda         2               137       gas        std          two
    ## 32          honda         2               137       gas        std          two
    ## 33          honda         1               101       gas        std          two
    ## 34          honda         1               101       gas        std          two
    ## 35          honda         1               101       gas        std          two
    ## 36          honda         0               110       gas        std         four
    ## 37          honda         0                78       gas        std         four
    ## 38          honda         0               106       gas        std          two
    ## 39          honda         0               106       gas        std          two
    ## 40          honda         0                85       gas        std         four
    ## 41          honda         0                85       gas        std         four
    ## 42          honda         0                85       gas        std         four
    ## 43          honda         1               107       gas        std          two
    ## 44          isuzu         0                NA       gas        std         four
    ## 45          isuzu         1                NA       gas        std          two
    ## 46          isuzu         0                NA       gas        std         four
    ## 47          isuzu         2                NA       gas        std          two
    ## 48         jaguar         0               145       gas        std         four
    ## 49         jaguar         0                NA       gas        std         four
    ## 50         jaguar         0                NA       gas        std          two
    ## 51          mazda         1               104       gas        std          two
    ## 52          mazda         1               104       gas        std          two
    ## 53          mazda         1               104       gas        std          two
    ## 54          mazda         1               113       gas        std         four
    ## 55          mazda         1               113       gas        std         four
    ## 56          mazda         3               150       gas        std          two
    ## 57          mazda         3               150       gas        std          two
    ## 58          mazda         3               150       gas        std          two
    ## 59          mazda         3               150       gas        std          two
    ## 60          mazda         1               129       gas        std          two
    ## 61          mazda         0               115       gas        std         four
    ## 62          mazda         1               129       gas        std          two
    ## 63          mazda         0               115       gas        std         four
    ## 64          mazda         0                NA    diesel        std         <NA>
    ## 65          mazda         0               115       gas        std         four
    ## 66          mazda         0               118       gas        std         four
    ## 67          mazda         0                NA    diesel        std         four
    ## 68  mercedes-benz        -1                93    diesel      turbo         four
    ## 69  mercedes-benz        -1                93    diesel      turbo         four
    ## 70  mercedes-benz         0                93    diesel      turbo          two
    ## 71  mercedes-benz        -1                93    diesel      turbo         four
    ## 72  mercedes-benz        -1                NA       gas        std         four
    ## 73  mercedes-benz         3               142       gas        std          two
    ## 74  mercedes-benz         0                NA       gas        std         four
    ## 75  mercedes-benz         1                NA       gas        std          two
    ## 76        mercury         1                NA       gas      turbo          two
    ## 77     mitsubishi         2               161       gas        std          two
    ## 78     mitsubishi         2               161       gas        std          two
    ## 79     mitsubishi         2               161       gas        std          two
    ## 80     mitsubishi         1               161       gas      turbo          two
    ## 81     mitsubishi         3               153       gas      turbo          two
    ## 82     mitsubishi         3               153       gas        std          two
    ## 83     mitsubishi         3                NA       gas      turbo          two
    ## 84     mitsubishi         3                NA       gas      turbo          two
    ## 85     mitsubishi         3                NA       gas      turbo          two
    ## 86     mitsubishi         1               125       gas        std         four
    ## 87     mitsubishi         1               125       gas        std         four
    ## 88     mitsubishi         1               125       gas      turbo         four
    ## 89     mitsubishi        -1               137       gas        std         four
    ## 90         nissan         1               128       gas        std          two
    ## 91         nissan         1               128    diesel        std          two
    ## 92         nissan         1               128       gas        std          two
    ## 93         nissan         1               122       gas        std         four
    ## 94         nissan         1               103       gas        std         four
    ## 95         nissan         1               128       gas        std          two
    ## 96         nissan         1               128       gas        std          two
    ## 97         nissan         1               122       gas        std         four
    ## 98         nissan         1               103       gas        std         four
    ## 99         nissan         2               168       gas        std          two
    ## 100        nissan         0               106       gas        std         four
    ## 101        nissan         0               106       gas        std         four
    ## 102        nissan         0               128       gas        std         four
    ## 103        nissan         0               108       gas        std         four
    ## 104        nissan         0               108       gas        std         four
    ## 105        nissan         3               194       gas        std          two
    ## 106        nissan         3               194       gas      turbo          two
    ## 107        nissan         1               231       gas        std          two
    ## 108        peugot         0               161       gas        std         four
    ## 109        peugot         0               161    diesel      turbo         four
    ## 110        peugot         0                NA       gas        std         four
    ## 111        peugot         0                NA    diesel      turbo         four
    ## 112        peugot         0               161       gas        std         four
    ## 113        peugot         0               161    diesel      turbo         four
    ## 114        peugot         0                NA       gas        std         four
    ## 115        peugot         0                NA    diesel      turbo         four
    ## 116        peugot         0               161       gas        std         four
    ## 117        peugot         0               161    diesel      turbo         four
    ## 118        peugot         0               161       gas      turbo         four
    ## 119      plymouth         1               119       gas        std          two
    ## 120      plymouth         1               119       gas      turbo          two
    ## 121      plymouth         1               154       gas        std         four
    ## 122      plymouth         1               154       gas        std         four
    ## 123      plymouth         1               154       gas        std         four
    ## 124      plymouth        -1                74       gas        std         four
    ## 125      plymouth         3                NA       gas      turbo          two
    ## 126       porsche         3               186       gas        std          two
    ## 127       porsche         3                NA       gas        std          two
    ## 128       porsche         3                NA       gas        std          two
    ## 129       porsche         3                NA       gas        std          two
    ## 130       porsche         1                NA       gas        std          two
    ## 131       renault         0                NA       gas        std         four
    ## 132       renault         2                NA       gas        std          two
    ## 133          saab         3               150       gas        std          two
    ## 134          saab         2               104       gas        std         four
    ## 135          saab         3               150       gas        std          two
    ## 136          saab         2               104       gas        std         four
    ## 137          saab         3               150       gas      turbo          two
    ## 138          saab         2               104       gas      turbo         four
    ## 139        subaru         2                83       gas        std          two
    ## 140        subaru         2                83       gas        std          two
    ## 141        subaru         2                83       gas        std          two
    ## 142        subaru         0               102       gas        std         four
    ## 143        subaru         0               102       gas        std         four
    ## 144        subaru         0               102       gas        std         four
    ## 145        subaru         0               102       gas        std         four
    ## 146        subaru         0               102       gas      turbo         four
    ## 147        subaru         0                89       gas        std         four
    ## 148        subaru         0                89       gas        std         four
    ## 149        subaru         0                85       gas        std         four
    ## 150        subaru         0                85       gas      turbo         four
    ## 151        toyota         1                87       gas        std          two
    ## 152        toyota         1                87       gas        std          two
    ## 153        toyota         1                74       gas        std         four
    ## 154        toyota         0                77       gas        std         four
    ## 155        toyota         0                81       gas        std         four
    ## 156        toyota         0                91       gas        std         four
    ## 157        toyota         0                91       gas        std         four
    ## 158        toyota         0                91       gas        std         four
    ## 159        toyota         0                91    diesel        std         four
    ## 160        toyota         0                91    diesel        std         four
    ## 161        toyota         0                91       gas        std         four
    ## 162        toyota         0                91       gas        std         four
    ## 163        toyota         0                91       gas        std         four
    ## 164        toyota         1               168       gas        std          two
    ## 165        toyota         1               168       gas        std          two
    ## 166        toyota         1               168       gas        std          two
    ## 167        toyota         1               168       gas        std          two
    ## 168        toyota         2               134       gas        std          two
    ## 169        toyota         2               134       gas        std          two
    ## 170        toyota         2               134       gas        std          two
    ## 171        toyota         2               134       gas        std          two
    ## 172        toyota         2               134       gas        std          two
    ## 173        toyota         2               134       gas        std          two
    ## 174        toyota        -1                65       gas        std         four
    ## 175        toyota        -1                65    diesel      turbo         four
    ## 176        toyota        -1                65       gas        std         four
    ## 177        toyota        -1                65       gas        std         four
    ## 178        toyota        -1                65       gas        std         four
    ## 179        toyota         3               197       gas        std          two
    ## 180        toyota         3               197       gas        std          two
    ## 181        toyota        -1                90       gas        std         four
    ## 182        toyota        -1                NA       gas        std         four
    ## 183    volkswagen         2               122    diesel        std          two
    ## 184    volkswagen         2               122       gas        std          two
    ## 185    volkswagen         2                94    diesel        std         four
    ## 186    volkswagen         2                94       gas        std         four
    ## 187    volkswagen         2                94       gas        std         four
    ## 188    volkswagen         2                94    diesel      turbo         four
    ## 189    volkswagen         2                94       gas        std         four
    ## 190    volkswagen         3                NA       gas        std          two
    ## 191    volkswagen         3               256       gas        std          two
    ## 192    volkswagen         0                NA       gas        std         four
    ## 193    volkswagen         0                NA    diesel      turbo         four
    ## 194    volkswagen         0                NA       gas        std         four
    ## 195         volvo        -2               103       gas        std         four
    ## 196         volvo        -1                74       gas        std         four
    ## 197         volvo        -2               103       gas        std         four
    ## 198         volvo        -1                74       gas        std         four
    ## 199         volvo        -2               103       gas      turbo         four
    ## 200         volvo        -1                74       gas      turbo         four
    ## 201         volvo        -1                95       gas        std         four
    ## 202         volvo        -1                95       gas      turbo         four
    ## 203         volvo        -1                95       gas        std         four
    ## 204         volvo        -1                95    diesel      turbo         four
    ## 205         volvo        -1                95       gas      turbo         four
    ##      body.style drive.wheels engine.location wheel.base length width height
    ## 1   convertible          rwd           front       88.6  168.8  64.1   48.8
    ## 2   convertible          rwd           front       88.6  168.8  64.1   48.8
    ## 3     hatchback          rwd           front       94.5  171.2  65.5   52.4
    ## 4         sedan          fwd           front       99.8  176.6  66.2   54.3
    ## 5         sedan          4wd           front       99.4  176.6  66.4   54.3
    ## 6         sedan          fwd           front       99.8  177.3  66.3   53.1
    ## 7         sedan          fwd           front      105.8  192.7  71.4   55.7
    ## 8         wagon          fwd           front      105.8  192.7  71.4   55.7
    ## 9         sedan          fwd           front      105.8  192.7  71.4   55.9
    ## 10    hatchback          4wd           front       99.5  178.2  67.9   52.0
    ## 11        sedan          rwd           front      101.2  176.8  64.8   54.3
    ## 12        sedan          rwd           front      101.2  176.8  64.8   54.3
    ## 13        sedan          rwd           front      101.2  176.8  64.8   54.3
    ## 14        sedan          rwd           front      101.2  176.8  64.8   54.3
    ## 15        sedan          rwd           front      103.5  189.0  66.9   55.7
    ## 16        sedan          rwd           front      103.5  189.0  66.9   55.7
    ## 17        sedan          rwd           front      103.5  193.8  67.9   53.7
    ## 18        sedan          rwd           front      110.0  197.0  70.9   56.3
    ## 19    hatchback          fwd           front       88.4  141.1  60.3   53.2
    ## 20    hatchback          fwd           front       94.5  155.9  63.6   52.0
    ## 21        sedan          fwd           front       94.5  158.8  63.6   52.0
    ## 22    hatchback          fwd           front       93.7  157.3  63.8   50.8
    ## 23    hatchback          fwd           front       93.7  157.3  63.8   50.8
    ## 24    hatchback          fwd           front       93.7  157.3  63.8   50.8
    ## 25    hatchback          fwd           front       93.7  157.3  63.8   50.6
    ## 26        sedan          fwd           front       93.7  157.3  63.8   50.6
    ## 27        sedan          fwd           front       93.7  157.3  63.8   50.6
    ## 28        sedan          fwd           front       93.7  157.3  63.8   50.6
    ## 29        wagon          fwd           front      103.3  174.6  64.6   59.8
    ## 30    hatchback          fwd           front       95.9  173.2  66.3   50.2
    ## 31    hatchback          fwd           front       86.6  144.6  63.9   50.8
    ## 32    hatchback          fwd           front       86.6  144.6  63.9   50.8
    ## 33    hatchback          fwd           front       93.7  150.0  64.0   52.6
    ## 34    hatchback          fwd           front       93.7  150.0  64.0   52.6
    ## 35    hatchback          fwd           front       93.7  150.0  64.0   52.6
    ## 36        sedan          fwd           front       96.5  163.4  64.0   54.5
    ## 37        wagon          fwd           front       96.5  157.1  63.9   58.3
    ## 38    hatchback          fwd           front       96.5  167.5  65.2   53.3
    ## 39    hatchback          fwd           front       96.5  167.5  65.2   53.3
    ## 40        sedan          fwd           front       96.5  175.4  65.2   54.1
    ## 41        sedan          fwd           front       96.5  175.4  62.5   54.1
    ## 42        sedan          fwd           front       96.5  175.4  65.2   54.1
    ## 43        sedan          fwd           front       96.5  169.1  66.0   51.0
    ## 44        sedan          rwd           front       94.3  170.7  61.8   53.5
    ## 45        sedan          fwd           front       94.5  155.9  63.6   52.0
    ## 46        sedan          fwd           front       94.5  155.9  63.6   52.0
    ## 47    hatchback          rwd           front       96.0  172.6  65.2   51.4
    ## 48        sedan          rwd           front      113.0  199.6  69.6   52.8
    ## 49        sedan          rwd           front      113.0  199.6  69.6   52.8
    ## 50        sedan          rwd           front      102.0  191.7  70.6   47.8
    ## 51    hatchback          fwd           front       93.1  159.1  64.2   54.1
    ## 52    hatchback          fwd           front       93.1  159.1  64.2   54.1
    ## 53    hatchback          fwd           front       93.1  159.1  64.2   54.1
    ## 54        sedan          fwd           front       93.1  166.8  64.2   54.1
    ## 55        sedan          fwd           front       93.1  166.8  64.2   54.1
    ## 56    hatchback          rwd           front       95.3  169.0  65.7   49.6
    ## 57    hatchback          rwd           front       95.3  169.0  65.7   49.6
    ## 58    hatchback          rwd           front       95.3  169.0  65.7   49.6
    ## 59    hatchback          rwd           front       95.3  169.0  65.7   49.6
    ## 60    hatchback          fwd           front       98.8  177.8  66.5   53.7
    ## 61        sedan          fwd           front       98.8  177.8  66.5   55.5
    ## 62    hatchback          fwd           front       98.8  177.8  66.5   53.7
    ## 63        sedan          fwd           front       98.8  177.8  66.5   55.5
    ## 64        sedan          fwd           front       98.8  177.8  66.5   55.5
    ## 65    hatchback          fwd           front       98.8  177.8  66.5   55.5
    ## 66        sedan          rwd           front      104.9  175.0  66.1   54.4
    ## 67        sedan          rwd           front      104.9  175.0  66.1   54.4
    ## 68        sedan          rwd           front      110.0  190.9  70.3   56.5
    ## 69        wagon          rwd           front      110.0  190.9  70.3   58.7
    ## 70      hardtop          rwd           front      106.7  187.5  70.3   54.9
    ## 71        sedan          rwd           front      115.6  202.6  71.7   56.3
    ## 72        sedan          rwd           front      115.6  202.6  71.7   56.5
    ## 73  convertible          rwd           front       96.6  180.3  70.5   50.8
    ## 74        sedan          rwd           front      120.9  208.1  71.7   56.7
    ## 75      hardtop          rwd           front      112.0  199.2  72.0   55.4
    ## 76    hatchback          rwd           front      102.7  178.4  68.0   54.8
    ## 77    hatchback          fwd           front       93.7  157.3  64.4   50.8
    ## 78    hatchback          fwd           front       93.7  157.3  64.4   50.8
    ## 79    hatchback          fwd           front       93.7  157.3  64.4   50.8
    ## 80    hatchback          fwd           front       93.0  157.3  63.8   50.8
    ## 81    hatchback          fwd           front       96.3  173.0  65.4   49.4
    ## 82    hatchback          fwd           front       96.3  173.0  65.4   49.4
    ## 83    hatchback          fwd           front       95.9  173.2  66.3   50.2
    ## 84    hatchback          fwd           front       95.9  173.2  66.3   50.2
    ## 85    hatchback          fwd           front       95.9  173.2  66.3   50.2
    ## 86        sedan          fwd           front       96.3  172.4  65.4   51.6
    ## 87        sedan          fwd           front       96.3  172.4  65.4   51.6
    ## 88        sedan          fwd           front       96.3  172.4  65.4   51.6
    ## 89        sedan          fwd           front       96.3  172.4  65.4   51.6
    ## 90        sedan          fwd           front       94.5  165.3  63.8   54.5
    ## 91        sedan          fwd           front       94.5  165.3  63.8   54.5
    ## 92        sedan          fwd           front       94.5  165.3  63.8   54.5
    ## 93        sedan          fwd           front       94.5  165.3  63.8   54.5
    ## 94        wagon          fwd           front       94.5  170.2  63.8   53.5
    ## 95        sedan          fwd           front       94.5  165.3  63.8   54.5
    ## 96    hatchback          fwd           front       94.5  165.6  63.8   53.3
    ## 97        sedan          fwd           front       94.5  165.3  63.8   54.5
    ## 98        wagon          fwd           front       94.5  170.2  63.8   53.5
    ## 99      hardtop          fwd           front       95.1  162.4  63.8   53.3
    ## 100   hatchback          fwd           front       97.2  173.4  65.2   54.7
    ## 101       sedan          fwd           front       97.2  173.4  65.2   54.7
    ## 102       sedan          fwd           front      100.4  181.7  66.5   55.1
    ## 103       wagon          fwd           front      100.4  184.6  66.5   56.1
    ## 104       sedan          fwd           front      100.4  184.6  66.5   55.1
    ## 105   hatchback          rwd           front       91.3  170.7  67.9   49.7
    ## 106   hatchback          rwd           front       91.3  170.7  67.9   49.7
    ## 107   hatchback          rwd           front       99.2  178.5  67.9   49.7
    ## 108       sedan          rwd           front      107.9  186.7  68.4   56.7
    ## 109       sedan          rwd           front      107.9  186.7  68.4   56.7
    ## 110       wagon          rwd           front      114.2  198.9  68.4   58.7
    ## 111       wagon          rwd           front      114.2  198.9  68.4   58.7
    ## 112       sedan          rwd           front      107.9  186.7  68.4   56.7
    ## 113       sedan          rwd           front      107.9  186.7  68.4   56.7
    ## 114       wagon          rwd           front      114.2  198.9  68.4   56.7
    ## 115       wagon          rwd           front      114.2  198.9  68.4   58.7
    ## 116       sedan          rwd           front      107.9  186.7  68.4   56.7
    ## 117       sedan          rwd           front      107.9  186.7  68.4   56.7
    ## 118       sedan          rwd           front      108.0  186.7  68.3   56.0
    ## 119   hatchback          fwd           front       93.7  157.3  63.8   50.8
    ## 120   hatchback          fwd           front       93.7  157.3  63.8   50.8
    ## 121   hatchback          fwd           front       93.7  157.3  63.8   50.6
    ## 122       sedan          fwd           front       93.7  167.3  63.8   50.8
    ## 123       sedan          fwd           front       93.7  167.3  63.8   50.8
    ## 124       wagon          fwd           front      103.3  174.6  64.6   59.8
    ## 125   hatchback          rwd           front       95.9  173.2  66.3   50.2
    ## 126   hatchback          rwd           front       94.5  168.9  68.3   50.2
    ## 127     hardtop          rwd            rear       89.5  168.9  65.0   51.6
    ## 128     hardtop          rwd            rear       89.5  168.9  65.0   51.6
    ## 129 convertible          rwd            rear       89.5  168.9  65.0   51.6
    ## 130   hatchback          rwd           front       98.4  175.7  72.3   50.5
    ## 131       wagon          fwd           front       96.1  181.5  66.5   55.2
    ## 132   hatchback          fwd           front       96.1  176.8  66.6   50.5
    ## 133   hatchback          fwd           front       99.1  186.6  66.5   56.1
    ## 134       sedan          fwd           front       99.1  186.6  66.5   56.1
    ## 135   hatchback          fwd           front       99.1  186.6  66.5   56.1
    ## 136       sedan          fwd           front       99.1  186.6  66.5   56.1
    ## 137   hatchback          fwd           front       99.1  186.6  66.5   56.1
    ## 138       sedan          fwd           front       99.1  186.6  66.5   56.1
    ## 139   hatchback          fwd           front       93.7  156.9  63.4   53.7
    ## 140   hatchback          fwd           front       93.7  157.9  63.6   53.7
    ## 141   hatchback          4wd           front       93.3  157.3  63.8   55.7
    ## 142       sedan          fwd           front       97.2  172.0  65.4   52.5
    ## 143       sedan          fwd           front       97.2  172.0  65.4   52.5
    ## 144       sedan          fwd           front       97.2  172.0  65.4   52.5
    ## 145       sedan          4wd           front       97.0  172.0  65.4   54.3
    ## 146       sedan          4wd           front       97.0  172.0  65.4   54.3
    ## 147       wagon          fwd           front       97.0  173.5  65.4   53.0
    ## 148       wagon          fwd           front       97.0  173.5  65.4   53.0
    ## 149       wagon          4wd           front       96.9  173.6  65.4   54.9
    ## 150       wagon          4wd           front       96.9  173.6  65.4   54.9
    ## 151   hatchback          fwd           front       95.7  158.7  63.6   54.5
    ## 152   hatchback          fwd           front       95.7  158.7  63.6   54.5
    ## 153   hatchback          fwd           front       95.7  158.7  63.6   54.5
    ## 154       wagon          fwd           front       95.7  169.7  63.6   59.1
    ## 155       wagon          4wd           front       95.7  169.7  63.6   59.1
    ## 156       wagon          4wd           front       95.7  169.7  63.6   59.1
    ## 157       sedan          fwd           front       95.7  166.3  64.4   53.0
    ## 158   hatchback          fwd           front       95.7  166.3  64.4   52.8
    ## 159       sedan          fwd           front       95.7  166.3  64.4   53.0
    ## 160   hatchback          fwd           front       95.7  166.3  64.4   52.8
    ## 161       sedan          fwd           front       95.7  166.3  64.4   53.0
    ## 162   hatchback          fwd           front       95.7  166.3  64.4   52.8
    ## 163       sedan          fwd           front       95.7  166.3  64.4   52.8
    ## 164       sedan          rwd           front       94.5  168.7  64.0   52.6
    ## 165   hatchback          rwd           front       94.5  168.7  64.0   52.6
    ## 166       sedan          rwd           front       94.5  168.7  64.0   52.6
    ## 167   hatchback          rwd           front       94.5  168.7  64.0   52.6
    ## 168     hardtop          rwd           front       98.4  176.2  65.6   52.0
    ## 169     hardtop          rwd           front       98.4  176.2  65.6   52.0
    ## 170   hatchback          rwd           front       98.4  176.2  65.6   52.0
    ## 171     hardtop          rwd           front       98.4  176.2  65.6   52.0
    ## 172   hatchback          rwd           front       98.4  176.2  65.6   52.0
    ## 173 convertible          rwd           front       98.4  176.2  65.6   53.0
    ## 174       sedan          fwd           front      102.4  175.6  66.5   54.9
    ## 175       sedan          fwd           front      102.4  175.6  66.5   54.9
    ## 176   hatchback          fwd           front      102.4  175.6  66.5   53.9
    ## 177       sedan          fwd           front      102.4  175.6  66.5   54.9
    ## 178   hatchback          fwd           front      102.4  175.6  66.5   53.9
    ## 179   hatchback          rwd           front      102.9  183.5  67.7   52.0
    ## 180   hatchback          rwd           front      102.9  183.5  67.7   52.0
    ## 181       sedan          rwd           front      104.5  187.8  66.5   54.1
    ## 182       wagon          rwd           front      104.5  187.8  66.5   54.1
    ## 183       sedan          fwd           front       97.3  171.7  65.5   55.7
    ## 184       sedan          fwd           front       97.3  171.7  65.5   55.7
    ## 185       sedan          fwd           front       97.3  171.7  65.5   55.7
    ## 186       sedan          fwd           front       97.3  171.7  65.5   55.7
    ## 187       sedan          fwd           front       97.3  171.7  65.5   55.7
    ## 188       sedan          fwd           front       97.3  171.7  65.5   55.7
    ## 189       sedan          fwd           front       97.3  171.7  65.5   55.7
    ## 190 convertible          fwd           front       94.5  159.3  64.2   55.6
    ## 191   hatchback          fwd           front       94.5  165.7  64.0   51.4
    ## 192       sedan          fwd           front      100.4  180.2  66.9   55.1
    ## 193       sedan          fwd           front      100.4  180.2  66.9   55.1
    ## 194       wagon          fwd           front      100.4  183.1  66.9   55.1
    ## 195       sedan          rwd           front      104.3  188.8  67.2   56.2
    ## 196       wagon          rwd           front      104.3  188.8  67.2   57.5
    ## 197       sedan          rwd           front      104.3  188.8  67.2   56.2
    ## 198       wagon          rwd           front      104.3  188.8  67.2   57.5
    ## 199       sedan          rwd           front      104.3  188.8  67.2   56.2
    ## 200       wagon          rwd           front      104.3  188.8  67.2   57.5
    ## 201       sedan          rwd           front      109.1  188.8  68.9   55.5
    ## 202       sedan          rwd           front      109.1  188.8  68.8   55.5
    ## 203       sedan          rwd           front      109.1  188.8  68.9   55.5
    ## 204       sedan          rwd           front      109.1  188.8  68.9   55.5
    ## 205       sedan          rwd           front      109.1  188.8  68.9   55.5
    ##     curb.weight engine.type num.of.cylinders engine.size fuel.system bore
    ## 1          2548        dohc             four         130        mpfi 3.47
    ## 2          2548        dohc             four         130        mpfi 3.47
    ## 3          2823        ohcv              six         152        mpfi 2.68
    ## 4          2337         ohc             four         109        mpfi 3.19
    ## 5          2824         ohc             five         136        mpfi 3.19
    ## 6          2507         ohc             five         136        mpfi 3.19
    ## 7          2844         ohc             five         136        mpfi 3.19
    ## 8          2954         ohc             five         136        mpfi 3.19
    ## 9          3086         ohc             five         131        mpfi 3.13
    ## 10         3053         ohc             five         131        mpfi 3.13
    ## 11         2395         ohc             four         108        mpfi 3.50
    ## 12         2395         ohc             four         108        mpfi 3.50
    ## 13         2710         ohc              six         164        mpfi 3.31
    ## 14         2765         ohc              six         164        mpfi 3.31
    ## 15         3055         ohc              six         164        mpfi 3.31
    ## 16         3230         ohc              six         209        mpfi 3.62
    ## 17         3380         ohc              six         209        mpfi 3.62
    ## 18         3505         ohc              six         209        mpfi 3.62
    ## 19         1488           l            three          61        2bbl 2.91
    ## 20         1874         ohc             four          90        2bbl 3.03
    ## 21         1909         ohc             four          90        2bbl 3.03
    ## 22         1876         ohc             four          90        2bbl 2.97
    ## 23         1876         ohc             four          90        2bbl 2.97
    ## 24         2128         ohc             four          98        mpfi 3.03
    ## 25         1967         ohc             four          90        2bbl 2.97
    ## 26         1989         ohc             four          90        2bbl 2.97
    ## 27         1989         ohc             four          90        2bbl 2.97
    ## 28         2191         ohc             four          98        mpfi 3.03
    ## 29         2535         ohc             four         122        2bbl 3.34
    ## 30         2811         ohc             four         156         mfi 3.60
    ## 31         1713         ohc             four          92        1bbl 2.91
    ## 32         1819         ohc             four          92        1bbl 2.91
    ## 33         1837         ohc             four          79        1bbl 2.91
    ## 34         1940         ohc             four          92        1bbl 2.91
    ## 35         1956         ohc             four          92        1bbl 2.91
    ## 36         2010         ohc             four          92        1bbl 2.91
    ## 37         2024         ohc             four          92        1bbl 2.92
    ## 38         2236         ohc             four         110        1bbl 3.15
    ## 39         2289         ohc             four         110        1bbl 3.15
    ## 40         2304         ohc             four         110        1bbl 3.15
    ## 41         2372         ohc             four         110        1bbl 3.15
    ## 42         2465         ohc             four         110        mpfi 3.15
    ## 43         2293         ohc             four         110        2bbl 3.15
    ## 44         2337         ohc             four         111        2bbl 3.31
    ## 45         1874         ohc             four          90        2bbl 3.03
    ## 46         1909         ohc             four          90        2bbl 3.03
    ## 47         2734         ohc             four         119        spfi 3.43
    ## 48         4066        dohc              six         258        mpfi 3.63
    ## 49         4066        dohc              six         258        mpfi 3.63
    ## 50         3950        ohcv           twelve         326        mpfi 3.54
    ## 51         1890         ohc             four          91        2bbl 3.03
    ## 52         1900         ohc             four          91        2bbl 3.03
    ## 53         1905         ohc             four          91        2bbl 3.03
    ## 54         1945         ohc             four          91        2bbl 3.03
    ## 55         1950         ohc             four          91        2bbl 3.08
    ## 56         2380       rotor              two          70        4bbl   NA
    ## 57         2380       rotor              two          70        4bbl   NA
    ## 58         2385       rotor              two          70        4bbl   NA
    ## 59         2500       rotor              two          80        mpfi   NA
    ## 60         2385         ohc             four         122        2bbl 3.39
    ## 61         2410         ohc             four         122        2bbl 3.39
    ## 62         2385         ohc             four         122        2bbl 3.39
    ## 63         2410         ohc             four         122        2bbl 3.39
    ## 64         2443         ohc             four         122         idi 3.39
    ## 65         2425         ohc             four         122        2bbl 3.39
    ## 66         2670         ohc             four         140        mpfi 3.76
    ## 67         2700         ohc             four         134         idi 3.43
    ## 68         3515         ohc             five         183         idi 3.58
    ## 69         3750         ohc             five         183         idi 3.58
    ## 70         3495         ohc             five         183         idi 3.58
    ## 71         3770         ohc             five         183         idi 3.58
    ## 72         3740        ohcv            eight         234        mpfi 3.46
    ## 73         3685        ohcv            eight         234        mpfi 3.46
    ## 74         3900        ohcv            eight         308        mpfi 3.80
    ## 75         3715        ohcv            eight         304        mpfi 3.80
    ## 76         2910         ohc             four         140        mpfi 3.78
    ## 77         1918         ohc             four          92        2bbl 2.97
    ## 78         1944         ohc             four          92        2bbl 2.97
    ## 79         2004         ohc             four          92        2bbl 2.97
    ## 80         2145         ohc             four          98        spdi 3.03
    ## 81         2370         ohc             four         110        spdi 3.17
    ## 82         2328         ohc             four         122        2bbl 3.35
    ## 83         2833         ohc             four         156        spdi 3.58
    ## 84         2921         ohc             four         156        spdi 3.59
    ## 85         2926         ohc             four         156        spdi 3.59
    ## 86         2365         ohc             four         122        2bbl 3.35
    ## 87         2405         ohc             four         122        2bbl 3.35
    ## 88         2403         ohc             four         110        spdi 3.17
    ## 89         2403         ohc             four         110        spdi 3.17
    ## 90         1889         ohc             four          97        2bbl 3.15
    ## 91         2017         ohc             four         103         idi 2.99
    ## 92         1918         ohc             four          97        2bbl 3.15
    ## 93         1938         ohc             four          97        2bbl 3.15
    ## 94         2024         ohc             four          97        2bbl 3.15
    ## 95         1951         ohc             four          97        2bbl 3.15
    ## 96         2028         ohc             four          97        2bbl 3.15
    ## 97         1971         ohc             four          97        2bbl 3.15
    ## 98         2037         ohc             four          97        2bbl 3.15
    ## 99         2008         ohc             four          97        2bbl 3.15
    ## 100        2324         ohc             four         120        2bbl 3.33
    ## 101        2302         ohc             four         120        2bbl 3.33
    ## 102        3095        ohcv              six         181        mpfi 3.43
    ## 103        3296        ohcv              six         181        mpfi 3.43
    ## 104        3060        ohcv              six         181        mpfi 3.43
    ## 105        3071        ohcv              six         181        mpfi 3.43
    ## 106        3139        ohcv              six         181        mpfi 3.43
    ## 107        3139        ohcv              six         181        mpfi 3.43
    ## 108        3020           l             four         120        mpfi 3.46
    ## 109        3197           l             four         152         idi 3.70
    ## 110        3230           l             four         120        mpfi 3.46
    ## 111        3430           l             four         152         idi 3.70
    ## 112        3075           l             four         120        mpfi 3.46
    ## 113        3252           l             four         152         idi 3.70
    ## 114        3285           l             four         120        mpfi 3.46
    ## 115        3485           l             four         152         idi 3.70
    ## 116        3075           l             four         120        mpfi 3.46
    ## 117        3252           l             four         152         idi 3.70
    ## 118        3130           l             four         134        mpfi 3.61
    ## 119        1918         ohc             four          90        2bbl 2.97
    ## 120        2128         ohc             four          98        spdi 3.03
    ## 121        1967         ohc             four          90        2bbl 2.97
    ## 122        1989         ohc             four          90        2bbl 2.97
    ## 123        2191         ohc             four          98        2bbl 2.97
    ## 124        2535         ohc             four         122        2bbl 3.35
    ## 125        2818         ohc             four         156        spdi 3.59
    ## 126        2778         ohc             four         151        mpfi 3.94
    ## 127        2756        ohcf              six         194        mpfi 3.74
    ## 128        2756        ohcf              six         194        mpfi 3.74
    ## 129        2800        ohcf              six         194        mpfi 3.74
    ## 130        3366       dohcv            eight         203        mpfi 3.94
    ## 131        2579         ohc             four         132        mpfi 3.46
    ## 132        2460         ohc             four         132        mpfi 3.46
    ## 133        2658         ohc             four         121        mpfi 3.54
    ## 134        2695         ohc             four         121        mpfi 3.54
    ## 135        2707         ohc             four         121        mpfi 2.54
    ## 136        2758         ohc             four         121        mpfi 3.54
    ## 137        2808        dohc             four         121        mpfi 3.54
    ## 138        2847        dohc             four         121        mpfi 3.54
    ## 139        2050        ohcf             four          97        2bbl 3.62
    ## 140        2120        ohcf             four         108        2bbl 3.62
    ## 141        2240        ohcf             four         108        2bbl 3.62
    ## 142        2145        ohcf             four         108        2bbl 3.62
    ## 143        2190        ohcf             four         108        2bbl 3.62
    ## 144        2340        ohcf             four         108        mpfi 3.62
    ## 145        2385        ohcf             four         108        2bbl 3.62
    ## 146        2510        ohcf             four         108        mpfi 3.62
    ## 147        2290        ohcf             four         108        2bbl 3.62
    ## 148        2455        ohcf             four         108        mpfi 3.62
    ## 149        2420        ohcf             four         108        2bbl 3.62
    ## 150        2650        ohcf             four         108        mpfi 3.62
    ## 151        1985         ohc             four          92        2bbl 3.05
    ## 152        2040         ohc             four          92        2bbl 3.05
    ## 153        2015         ohc             four          92        2bbl 3.05
    ## 154        2280         ohc             four          92        2bbl 3.05
    ## 155        2290         ohc             four          92        2bbl 3.05
    ## 156        3110         ohc             four          92        2bbl 3.05
    ## 157        2081         ohc             four          98        2bbl 3.19
    ## 158        2109         ohc             four          98        2bbl 3.19
    ## 159        2275         ohc             four         110         idi 3.27
    ## 160        2275         ohc             four         110         idi 3.27
    ## 161        2094         ohc             four          98        2bbl 3.19
    ## 162        2122         ohc             four          98        2bbl 3.19
    ## 163        2140         ohc             four          98        2bbl 3.19
    ## 164        2169         ohc             four          98        2bbl 3.19
    ## 165        2204         ohc             four          98        2bbl 3.19
    ## 166        2265        dohc             four          98        mpfi 3.24
    ## 167        2300        dohc             four          98        mpfi 3.24
    ## 168        2540         ohc             four         146        mpfi 3.62
    ## 169        2536         ohc             four         146        mpfi 3.62
    ## 170        2551         ohc             four         146        mpfi 3.62
    ## 171        2679         ohc             four         146        mpfi 3.62
    ## 172        2714         ohc             four         146        mpfi 3.62
    ## 173        2975         ohc             four         146        mpfi 3.62
    ## 174        2326         ohc             four         122        mpfi 3.31
    ## 175        2480         ohc             four         110         idi 3.27
    ## 176        2414         ohc             four         122        mpfi 3.31
    ## 177        2414         ohc             four         122        mpfi 3.31
    ## 178        2458         ohc             four         122        mpfi 3.31
    ## 179        2976        dohc              six         171        mpfi 3.27
    ## 180        3016        dohc              six         171        mpfi 3.27
    ## 181        3131        dohc              six         171        mpfi 3.27
    ## 182        3151        dohc              six         161        mpfi 3.27
    ## 183        2261         ohc             four          97         idi 3.01
    ## 184        2209         ohc             four         109        mpfi 3.19
    ## 185        2264         ohc             four          97         idi 3.01
    ## 186        2212         ohc             four         109        mpfi 3.19
    ## 187        2275         ohc             four         109        mpfi 3.19
    ## 188        2319         ohc             four          97         idi 3.01
    ## 189        2300         ohc             four         109        mpfi 3.19
    ## 190        2254         ohc             four         109        mpfi 3.19
    ## 191        2221         ohc             four         109        mpfi 3.19
    ## 192        2661         ohc             five         136        mpfi 3.19
    ## 193        2579         ohc             four          97         idi 3.01
    ## 194        2563         ohc             four         109        mpfi 3.19
    ## 195        2912         ohc             four         141        mpfi 3.78
    ## 196        3034         ohc             four         141        mpfi 3.78
    ## 197        2935         ohc             four         141        mpfi 3.78
    ## 198        3042         ohc             four         141        mpfi 3.78
    ## 199        3045         ohc             four         130        mpfi 3.62
    ## 200        3157         ohc             four         130        mpfi 3.62
    ## 201        2952         ohc             four         141        mpfi 3.78
    ## 202        3049         ohc             four         141        mpfi 3.78
    ## 203        3012        ohcv              six         173        mpfi 3.58
    ## 204        3217         ohc              six         145         idi 3.01
    ## 205        3062         ohc             four         141        mpfi 3.78
    ##     stroke compression.ratio horsepower peak.rpm city.mpg highway.mpg price
    ## 1     2.68              9.00        111     5000       21          27 13495
    ## 2     2.68              9.00        111     5000       21          27 16500
    ## 3     3.47              9.00        154     5000       19          26 16500
    ## 4     3.40             10.00        102     5500       24          30 13950
    ## 5     3.40              8.00        115     5500       18          22 17450
    ## 6     3.40              8.50        110     5500       19          25 15250
    ## 7     3.40              8.50        110     5500       19          25 17710
    ## 8     3.40              8.50        110     5500       19          25 18920
    ## 9     3.40              8.30        140     5500       17          20 23875
    ## 10    3.40              7.00        160     5500       16          22    NA
    ## 11    2.80              8.80        101     5800       23          29 16430
    ## 12    2.80              8.80        101     5800       23          29 16925
    ## 13    3.19              9.00        121     4250       21          28 20970
    ## 14    3.19              9.00        121     4250       21          28 21105
    ## 15    3.19              9.00        121     4250       20          25 24565
    ## 16    3.39              8.00        182     5400       16          22 30760
    ## 17    3.39              8.00        182     5400       16          22 41315
    ## 18    3.39              8.00        182     5400       15          20 36880
    ## 19    3.03              9.50         48     5100       47          53  5151
    ## 20    3.11              9.60         70     5400       38          43  6295
    ## 21    3.11              9.60         70     5400       38          43  6575
    ## 22    3.23              9.41         68     5500       37          41  5572
    ## 23    3.23              9.40         68     5500       31          38  6377
    ## 24    3.39              7.60        102     5500       24          30  7957
    ## 25    3.23              9.40         68     5500       31          38  6229
    ## 26    3.23              9.40         68     5500       31          38  6692
    ## 27    3.23              9.40         68     5500       31          38  7609
    ## 28    3.39              7.60        102     5500       24          30  8558
    ## 29    3.46              8.50         88     5000       24          30  8921
    ## 30    3.90              7.00        145     5000       19          24 12964
    ## 31    3.41              9.60         58     4800       49          54  6479
    ## 32    3.41              9.20         76     6000       31          38  6855
    ## 33    3.07             10.10         60     5500       38          42  5399
    ## 34    3.41              9.20         76     6000       30          34  6529
    ## 35    3.41              9.20         76     6000       30          34  7129
    ## 36    3.41              9.20         76     6000       30          34  7295
    ## 37    3.41              9.20         76     6000       30          34  7295
    ## 38    3.58              9.00         86     5800       27          33  7895
    ## 39    3.58              9.00         86     5800       27          33  9095
    ## 40    3.58              9.00         86     5800       27          33  8845
    ## 41    3.58              9.00         86     5800       27          33 10295
    ## 42    3.58              9.00        101     5800       24          28 12945
    ## 43    3.58              9.10        100     5500       25          31 10345
    ## 44    3.23              8.50         78     4800       24          29  6785
    ## 45    3.11              9.60         70     5400       38          43    NA
    ## 46    3.11              9.60         70     5400       38          43    NA
    ## 47    3.23              9.20         90     5000       24          29 11048
    ## 48    4.17              8.10        176     4750       15          19 32250
    ## 49    4.17              8.10        176     4750       15          19 35550
    ## 50    2.76             11.50        262     5000       13          17 36000
    ## 51    3.15              9.00         68     5000       30          31  5195
    ## 52    3.15              9.00         68     5000       31          38  6095
    ## 53    3.15              9.00         68     5000       31          38  6795
    ## 54    3.15              9.00         68     5000       31          38  6695
    ## 55    3.15              9.00         68     5000       31          38  7395
    ## 56      NA              9.40        101     6000       17          23 10945
    ## 57      NA              9.40        101     6000       17          23 11845
    ## 58      NA              9.40        101     6000       17          23 13645
    ## 59      NA              9.40        135     6000       16          23 15645
    ## 60    3.39              8.60         84     4800       26          32  8845
    ## 61    3.39              8.60         84     4800       26          32  8495
    ## 62    3.39              8.60         84     4800       26          32 10595
    ## 63    3.39              8.60         84     4800       26          32 10245
    ## 64    3.39             22.70         64     4650       36          42 10795
    ## 65    3.39              8.60         84     4800       26          32 11245
    ## 66    3.16              8.00        120     5000       19          27 18280
    ## 67    3.64             22.00         72     4200       31          39 18344
    ## 68    3.64             21.50        123     4350       22          25 25552
    ## 69    3.64             21.50        123     4350       22          25 28248
    ## 70    3.64             21.50        123     4350       22          25 28176
    ## 71    3.64             21.50        123     4350       22          25 31600
    ## 72    3.10              8.30        155     4750       16          18 34184
    ## 73    3.10              8.30        155     4750       16          18 35056
    ## 74    3.35              8.00        184     4500       14          16 40960
    ## 75    3.35              8.00        184     4500       14          16 45400
    ## 76    3.12              8.00        175     5000       19          24 16503
    ## 77    3.23              9.40         68     5500       37          41  5389
    ## 78    3.23              9.40         68     5500       31          38  6189
    ## 79    3.23              9.40         68     5500       31          38  6669
    ## 80    3.39              7.60        102     5500       24          30  7689
    ## 81    3.46              7.50        116     5500       23          30  9959
    ## 82    3.46              8.50         88     5000       25          32  8499
    ## 83    3.86              7.00        145     5000       19          24 12629
    ## 84    3.86              7.00        145     5000       19          24 14869
    ## 85    3.86              7.00        145     5000       19          24 14489
    ## 86    3.46              8.50         88     5000       25          32  6989
    ## 87    3.46              8.50         88     5000       25          32  8189
    ## 88    3.46              7.50        116     5500       23          30  9279
    ## 89    3.46              7.50        116     5500       23          30  9279
    ## 90    3.29              9.40         69     5200       31          37  5499
    ## 91    3.47             21.90         55     4800       45          50  7099
    ## 92    3.29              9.40         69     5200       31          37  6649
    ## 93    3.29              9.40         69     5200       31          37  6849
    ## 94    3.29              9.40         69     5200       31          37  7349
    ## 95    3.29              9.40         69     5200       31          37  7299
    ## 96    3.29              9.40         69     5200       31          37  7799
    ## 97    3.29              9.40         69     5200       31          37  7499
    ## 98    3.29              9.40         69     5200       31          37  7999
    ## 99    3.29              9.40         69     5200       31          37  8249
    ## 100   3.47              8.50         97     5200       27          34  8949
    ## 101   3.47              8.50         97     5200       27          34  9549
    ## 102   3.27              9.00        152     5200       17          22 13499
    ## 103   3.27              9.00        152     5200       17          22 14399
    ## 104   3.27              9.00        152     5200       19          25 13499
    ## 105   3.27              9.00        160     5200       19          25 17199
    ## 106   3.27              7.80        200     5200       17          23 19699
    ## 107   3.27              9.00        160     5200       19          25 18399
    ## 108   3.19              8.40         97     5000       19          24 11900
    ## 109   3.52             21.00         95     4150       28          33 13200
    ## 110   3.19              8.40         97     5000       19          24 12440
    ## 111   3.52             21.00         95     4150       25          25 13860
    ## 112   2.19              8.40         95     5000       19          24 15580
    ## 113   3.52             21.00         95     4150       28          33 16900
    ## 114   2.19              8.40         95     5000       19          24 16695
    ## 115   3.52             21.00         95     4150       25          25 17075
    ## 116   3.19              8.40         97     5000       19          24 16630
    ## 117   3.52             21.00         95     4150       28          33 17950
    ## 118   3.21              7.00        142     5600       18          24 18150
    ## 119   3.23              9.40         68     5500       37          41  5572
    ## 120   3.39              7.60        102     5500       24          30  7957
    ## 121   3.23              9.40         68     5500       31          38  6229
    ## 122   3.23              9.40         68     5500       31          38  6692
    ## 123   3.23              9.40         68     5500       31          38  7609
    ## 124   3.46              8.50         88     5000       24          30  8921
    ## 125   3.86              7.00        145     5000       19          24 12764
    ## 126   3.11              9.50        143     5500       19          27 22018
    ## 127   2.90              9.50        207     5900       17          25 32528
    ## 128   2.90              9.50        207     5900       17          25 34028
    ## 129   2.90              9.50        207     5900       17          25 37028
    ## 130   3.11             10.00        288     5750       17          28    NA
    ## 131   3.90              8.70         NA       NA       23          31  9295
    ## 132   3.90              8.70         NA       NA       23          31  9895
    ## 133   3.07              9.31        110     5250       21          28 11850
    ## 134   3.07              9.30        110     5250       21          28 12170
    ## 135   2.07              9.30        110     5250       21          28 15040
    ## 136   3.07              9.30        110     5250       21          28 15510
    ## 137   3.07              9.00        160     5500       19          26 18150
    ## 138   3.07              9.00        160     5500       19          26 18620
    ## 139   2.36              9.00         69     4900       31          36  5118
    ## 140   2.64              8.70         73     4400       26          31  7053
    ## 141   2.64              8.70         73     4400       26          31  7603
    ## 142   2.64              9.50         82     4800       32          37  7126
    ## 143   2.64              9.50         82     4400       28          33  7775
    ## 144   2.64              9.00         94     5200       26          32  9960
    ## 145   2.64              9.00         82     4800       24          25  9233
    ## 146   2.64              7.70        111     4800       24          29 11259
    ## 147   2.64              9.00         82     4800       28          32  7463
    ## 148   2.64              9.00         94     5200       25          31 10198
    ## 149   2.64              9.00         82     4800       23          29  8013
    ## 150   2.64              7.70        111     4800       23          23 11694
    ## 151   3.03              9.00         62     4800       35          39  5348
    ## 152   3.03              9.00         62     4800       31          38  6338
    ## 153   3.03              9.00         62     4800       31          38  6488
    ## 154   3.03              9.00         62     4800       31          37  6918
    ## 155   3.03              9.00         62     4800       27          32  7898
    ## 156   3.03              9.00         62     4800       27          32  8778
    ## 157   3.03              9.00         70     4800       30          37  6938
    ## 158   3.03              9.00         70     4800       30          37  7198
    ## 159   3.35             22.50         56     4500       34          36  7898
    ## 160   3.35             22.50         56     4500       38          47  7788
    ## 161   3.03              9.00         70     4800       38          47  7738
    ## 162   3.03              9.00         70     4800       28          34  8358
    ## 163   3.03              9.00         70     4800       28          34  9258
    ## 164   3.03              9.00         70     4800       29          34  8058
    ## 165   3.03              9.00         70     4800       29          34  8238
    ## 166   3.08              9.40        112     6600       26          29  9298
    ## 167   3.08              9.40        112     6600       26          29  9538
    ## 168   3.50              9.30        116     4800       24          30  8449
    ## 169   3.50              9.30        116     4800       24          30  9639
    ## 170   3.50              9.30        116     4800       24          30  9989
    ## 171   3.50              9.30        116     4800       24          30 11199
    ## 172   3.50              9.30        116     4800       24          30 11549
    ## 173   3.50              9.30        116     4800       24          30 17669
    ## 174   3.54              8.70         92     4200       29          34  8948
    ## 175   3.35             22.50         73     4500       30          33 10698
    ## 176   3.54              8.70         92     4200       27          32  9988
    ## 177   3.54              8.70         92     4200       27          32 10898
    ## 178   3.54              8.70         92     4200       27          32 11248
    ## 179   3.35              9.30        161     5200       20          24 16558
    ## 180   3.35              9.30        161     5200       19          24 15998
    ## 181   3.35              9.20        156     5200       20          24 15690
    ## 182   3.35              9.20        156     5200       19          24 15750
    ## 183   3.40             23.00         52     4800       37          46  7775
    ## 184   3.40              9.00         85     5250       27          34  7975
    ## 185   3.40             23.00         52     4800       37          46  7995
    ## 186   3.40              9.00         85     5250       27          34  8195
    ## 187   3.40              9.00         85     5250       27          34  8495
    ## 188   3.40             23.00         68     4500       37          42  9495
    ## 189   3.40             10.00        100     5500       26          32  9995
    ## 190   3.40              8.50         90     5500       24          29 11595
    ## 191   3.40              8.50         90     5500       24          29  9980
    ## 192   3.40              8.50        110     5500       19          24 13295
    ## 193   3.40             23.00         68     4500       33          38 13845
    ## 194   3.40              9.00         88     5500       25          31 12290
    ## 195   3.15              9.50        114     5400       23          28 12940
    ## 196   3.15              9.50        114     5400       23          28 13415
    ## 197   3.15              9.50        114     5400       24          28 15985
    ## 198   3.15              9.50        114     5400       24          28 16515
    ## 199   3.15              7.50        162     5100       17          22 18420
    ## 200   3.15              7.50        162     5100       17          22 18950
    ## 201   3.15              9.50        114     5400       23          28 16845
    ## 202   3.15              8.70        160     5300       19          25 19045
    ## 203   2.87              8.80        134     5500       18          23 21485
    ## 204   3.40             23.00        106     4800       26          27 22470
    ## 205   3.15              9.50        114     5400       19          25 22625

``` r
#Counting all NA values
colSums(is.na(auto_data))
```

    ##              make         symboling normalized.losses         fuel.type 
    ##                 0                 0                41                 0 
    ##        aspiration      num.of.doors        body.style      drive.wheels 
    ##                 0                 2                 0                 0 
    ##   engine.location        wheel.base            length             width 
    ##                 0                 0                 0                 0 
    ##            height       curb.weight       engine.type  num.of.cylinders 
    ##                 0                 0                 0                 0 
    ##       engine.size       fuel.system              bore            stroke 
    ##                 0                 0                 4                 4 
    ## compression.ratio        horsepower          peak.rpm          city.mpg 
    ##                 0                 2                 2                 0 
    ##       highway.mpg             price 
    ##                 0                 4

\##Dropping “NA” values

``` r
auto_data.c= auto_data.c[complete.cases(auto_data.c),]
auto_data.c= type.convert(auto_data.c)
```

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(x[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

``` r
#dropping engine location bc it has only 1 factor in the cleaned data
auto_data.c= subset(auto_data.c, select= -c(engine.location) )
```

*Attempts were made to group missing values by make, car type, etc. and
average missing integer values or use retype(auto_data.c) mode values
for characters/strings but this likely would lead to false conclusions
since car make is a large actor in safety. \[see below for alternative
atttempts\]*

\##Turning Characters to Factors

``` r
auto_data.c <- as.data.frame(unclass(auto_data.c),stringsAsFactors=TRUE)
str(auto_data.c)
```

    ## 'data.frame':    159 obs. of  25 variables:
    ##  $ make             : Factor w/ 18 levels "audi","bmw","chevrolet",..: 1 1 1 1 2 2 2 2 3 3 ...
    ##  $ symboling        : int  2 2 1 1 2 0 0 0 2 1 ...
    ##  $ normalized.losses: int  164 164 158 158 192 192 188 188 121 98 ...
    ##  $ fuel.type        : Factor w/ 2 levels "diesel","gas": 2 2 2 2 2 2 2 2 2 2 ...
    ##  $ aspiration       : Factor w/ 2 levels "std","turbo": 1 1 1 2 1 1 1 1 1 1 ...
    ##  $ num.of.doors     : Factor w/ 2 levels "four","two": 1 1 1 1 2 1 2 1 2 2 ...
    ##  $ body.style       : Factor w/ 5 levels "convertible",..: 4 4 4 4 4 4 4 4 3 3 ...
    ##  $ drive.wheels     : Factor w/ 3 levels "4wd","fwd","rwd": 2 1 2 2 3 3 3 3 2 2 ...
    ##  $ wheel.base       : num  99.8 99.4 105.8 105.8 101.2 ...
    ##  $ length           : num  177 177 193 193 177 ...
    ##  $ width            : num  66.2 66.4 71.4 71.4 64.8 64.8 64.8 64.8 60.3 63.6 ...
    ##  $ height           : num  54.3 54.3 55.7 55.9 54.3 54.3 54.3 54.3 53.2 52 ...
    ##  $ curb.weight      : int  2337 2824 2844 3086 2395 2395 2710 2765 1488 1874 ...
    ##  $ engine.type      : Factor w/ 5 levels "dohc","l","ohc",..: 3 3 3 3 3 3 3 3 2 3 ...
    ##  $ num.of.cylinders : Factor w/ 5 levels "eight","five",..: 3 2 2 2 3 3 4 4 5 3 ...
    ##  $ engine.size      : int  109 136 136 131 108 108 164 164 61 90 ...
    ##  $ fuel.system      : Factor w/ 6 levels "1bbl","2bbl",..: 5 5 5 5 5 5 5 5 2 2 ...
    ##  $ bore             : num  3.19 3.19 3.19 3.13 3.5 3.5 3.31 3.31 2.91 3.03 ...
    ##  $ stroke           : num  3.4 3.4 3.4 3.4 2.8 2.8 3.19 3.19 3.03 3.11 ...
    ##  $ compression.ratio: num  10 8 8.5 8.3 8.8 8.8 9 9 9.5 9.6 ...
    ##  $ horsepower       : int  102 115 110 140 101 101 121 121 48 70 ...
    ##  $ peak.rpm         : int  5500 5500 5500 5500 5800 5800 4250 4250 5100 5400 ...
    ##  $ city.mpg         : int  24 18 19 17 23 23 21 21 47 38 ...
    ##  $ highway.mpg      : int  30 22 25 20 29 29 28 28 53 43 ...
    ##  $ price            : int  13950 17450 17710 23875 16430 16925 20970 21105 5151 6295 ...

===============================================================================
\#3 Visualizing
===============================================================================

\##Visualizing Distribution of Symboling (rating of how risky the car is
compared to what tis price indicates)

``` r
hist(auto_data.c$symboling)
```

![](Automobile_Dataset_RegressionTesting_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

``` r
hist(auto_data.c$normalized.losses)
```

![](Automobile_Dataset_RegressionTesting_files/figure-gfm/unnamed-chunk-6-2.png)<!-- -->

``` r
hist(auto_data.c$price)
```

![](Automobile_Dataset_RegressionTesting_files/figure-gfm/unnamed-chunk-6-3.png)<!-- -->

*It appears as though symboling ratings are approximately normal while
losses and price are heavily skewed* *This posits that the latter two
should be normalized in regression testing should a predictive model be
built*

\##Make vs Symboling

``` r
#Creating a Table containing only the make and mean symbol rating
SYMvMAKE = (aggregate(auto_data.c$symboling, by=list(auto_data.c$make) ,FUN =  mean )) 
SYMvMAKE$x = as.numeric(SYMvMAKE$x)
colnames(SYMvMAKE)= c("make","mean")

library(ggplot2)
```

    ## 
    ## Attaching package: 'ggplot2'

    ## The following objects are masked from 'package:psych':
    ## 
    ##     %+%, alpha

``` r
ggplot(SYMvMAKE, aes(x=factor(make),y=mean)) + geom_bar(stat = "identity") + theme(axis.text.x = element_text(angle = 45))
```

![](Automobile_Dataset_RegressionTesting_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

*From Documentation: A value of +3 indicates that the auto is risky, -3
that it is probably pretty safe.* *It appears that for this sample, on
average, Volvo is the safest car option while Porsche Saab, & VW are*
*particularly dangerous*

\##Transforming factor levels for character variables to numeric levels

``` r
auto_data.n = auto_data.c

for (i in 1:ncol(auto_data.n)){
  if (is.factor(auto_data.n[,i])){
   auto_data.n[,i] = as.numeric(auto_data.n[,i])
  }
}
```

*Alternative solution: data %\<\>% mutate_if(is.factor, as.numeric)*

\##Showing correlation of variables

``` r
library(psych)
library(corrplot)
```

    ## corrplot 0.92 loaded

``` r
corrplot(cor(auto_data.n), 
         tl.cex = 0.6, type = 'lower', diag = FALSE)
```

![](Automobile_Dataset_RegressionTesting_files/figure-gfm/unnamed-chunk-9-1.png)<!-- -->

*NOTE: a positive rating in symboling denotes a more risky vehicle*

*Symboling* *Positive correlation: losses, \# doors* *Negative
correlation: body style, wheel base, dimensions* *Losses* *Positive
correlation: \# doors,* *Negative correlation: body style, wheel base,
dimensions*

================================================================================
\#4 Regression testing with the base data.frame
================================================================================

\##Regression Test 1: Full Data, All Variables

``` r
auto.lm1= lm(symboling~. , data=auto_data.c)
summary(auto.lm1)
```

    ## 
    ## Call:
    ## lm(formula = symboling ~ ., data = auto_data.c)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -1.1884 -0.2067  0.0000  0.2510  1.4717 
    ## 
    ## Coefficients: (3 not defined because of singularities)
    ##                         Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)            7.278e+00  7.007e+00   1.039 0.301308    
    ## makebmw               -2.544e+00  7.620e-01  -3.338 0.001161 ** 
    ## makechevrolet         -2.417e+00  8.221e-01  -2.940 0.004028 ** 
    ## makedodge             -2.235e+00  7.381e-01  -3.028 0.003086 ** 
    ## makehonda             -1.722e+00  7.624e-01  -2.259 0.025941 *  
    ## makejaguar            -2.187e+00  1.291e+00  -1.694 0.093232 .  
    ## makemazda             -1.825e+00  7.006e-01  -2.605 0.010501 *  
    ## makemercedes-benz     -2.353e+00  6.907e-01  -3.407 0.000926 ***
    ## makemitsubishi        -1.386e+00  7.445e-01  -1.862 0.065306 .  
    ## makenissan            -1.818e+00  6.527e-01  -2.785 0.006333 ** 
    ## makepeugot            -1.446e+00  2.122e+00  -0.681 0.497106    
    ## makeplymouth          -2.068e+00  7.422e-01  -2.786 0.006315 ** 
    ## makeporsche           -1.834e+00  9.411e-01  -1.949 0.053944 .  
    ## makesaab               1.881e-01  6.936e-01   0.271 0.786819    
    ## makesubaru            -2.205e+00  1.055e+00  -2.091 0.038901 *  
    ## maketoyota            -2.029e+00  7.303e-01  -2.779 0.006442 ** 
    ## makevolkswagen        -3.344e-02  6.336e-01  -0.053 0.958011    
    ## makevolvo             -3.161e+00  7.724e-01  -4.092  8.3e-05 ***
    ## normalized.losses      3.145e-03  2.788e-03   1.128 0.261801    
    ## fuel.typegas           2.841e+00  2.326e+00   1.221 0.224669    
    ## aspirationturbo        5.876e-01  2.765e-01   2.125 0.035860 *  
    ## num.of.doorstwo        5.820e-01  1.641e-01   3.547 0.000580 ***
    ## body.stylehardtop     -3.743e-01  6.600e-01  -0.567 0.571855    
    ## body.stylehatchback   -7.120e-01  6.458e-01  -1.103 0.272706    
    ## body.stylesedan       -9.240e-01  6.656e-01  -1.388 0.167951    
    ## body.stylewagon       -5.901e-01  6.749e-01  -0.874 0.383845    
    ## drive.wheelsfwd       -3.459e-01  2.886e-01  -1.198 0.233389    
    ## drive.wheelsrwd        8.155e-02  4.354e-01   0.187 0.851762    
    ## wheel.base            -7.079e-02  3.901e-02  -1.815 0.072395 .  
    ## length                -1.253e-02  1.730e-02  -0.725 0.470273    
    ## width                  3.763e-02  9.206e-02   0.409 0.683551    
    ## height                -7.263e-02  5.496e-02  -1.322 0.189116    
    ## curb.weight           -4.071e-04  6.205e-04  -0.656 0.513208    
    ## engine.typel          -1.311e-02  1.652e+00  -0.008 0.993684    
    ## engine.typeohc         4.506e-02  4.527e-01   0.100 0.920888    
    ## engine.typeohcf               NA         NA      NA       NA    
    ## engine.typeohcv       -7.194e-01  5.174e-01  -1.390 0.167280    
    ## num.of.cylindersfive  -1.146e+00  1.152e+00  -0.995 0.322020    
    ## num.of.cylindersfour  -9.304e-01  1.460e+00  -0.637 0.525418    
    ## num.of.cylinderssix   -5.738e-01  1.299e+00  -0.442 0.659502    
    ## num.of.cylindersthree         NA         NA      NA       NA    
    ## engine.size            1.729e-02  1.041e-02   1.660 0.099744 .  
    ## fuel.system2bbl        5.908e-01  4.603e-01   1.284 0.202061    
    ## fuel.systemidi                NA         NA      NA       NA    
    ## fuel.systemmfi         2.326e+00  8.595e-01   2.707 0.007913 ** 
    ## fuel.systemmpfi        4.159e-01  4.944e-01   0.841 0.402117    
    ## fuel.systemspdi        8.171e-02  5.792e-01   0.141 0.888086    
    ## bore                   2.568e-01  6.385e-01   0.402 0.688391    
    ## stroke                -4.844e-01  4.238e-01  -1.143 0.255554    
    ## compression.ratio      2.393e-01  1.729e-01   1.384 0.169227    
    ## horsepower            -4.581e-03  8.573e-03  -0.534 0.594192    
    ## peak.rpm               2.744e-04  2.525e-04   1.086 0.279723    
    ## city.mpg              -6.220e-02  4.614e-02  -1.348 0.180468    
    ## highway.mpg            6.007e-02  3.979e-02   1.509 0.134135    
    ## price                 -1.012e-06  4.321e-05  -0.023 0.981351    
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.5206 on 107 degrees of freedom
    ## Multiple R-squared:  0.871,  Adjusted R-squared:  0.8096 
    ## F-statistic: 14.17 on 51 and 107 DF,  p-value: < 2.2e-16

*With an*
![R^2](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;R%5E2 "R^2")
*value of \~.81 the model seems to explain much of the variation in
symbol ratings even without log normalizing skewed variables*

\##Creating test and train DF *Although creating a test/train set is not
necessary/traditional fro simple regression it helped to plot and
visualize the strength of the model*

``` r
dt = sort(sample(nrow(auto_data.c), nrow(auto_data.c)*.5))
train<-auto_data.c[dt,]
test<-auto_data.c[-dt,]

#There are an odd number of rows n the cleaned data set so we have to drop 1 for an even split
test= test[-c(1),]
```

``` r
auto.lm_train= lm(symboling~. , data=train)

comparison <- data.frame(actual= test$symboling, predicted=predict(auto.lm_train))
comparison
```

    ##     actual     predicted
    ## 3        2  1.000000e+00
    ## 7        1  1.805621e-01
    ## 8        2 -1.805621e-01
    ## 9        0  2.000000e+00
    ## 10       0  1.000000e+00
    ## 14       1  1.000000e+00
    ## 19       1  3.000000e+00
    ## 20       1  1.883353e+00
    ## 22       1  1.201187e+00
    ## 26       1 -3.079605e-01
    ## 27      -1  4.318592e-01
    ## 29       2 -2.604484e-01
    ## 30       1  5.200952e-02
    ## 31       1  2.493838e-14
    ## 34       0  1.037263e+00
    ## 35       0  1.444439e+00
    ## 39       1  6.710070e-01
    ## 42       0 -1.527093e-01
    ## 47       1  2.162159e-14
    ## 50       1  2.243989e+00
    ## 55       1  2.048991e+00
    ## 56       0  1.244870e+00
    ## 57       1  1.264811e+00
    ## 58       0  6.154514e-01
    ## 59       0 -4.181124e-01
    ## 61      -1  7.981343e-01
    ## 62      -1  9.141085e-01
    ## 63      -1  5.771811e-01
    ## 64       3  1.038375e+00
    ## 65       2  9.286421e-01
    ## 66       2  1.414596e+00
    ## 69       1  2.141801e+00
    ## 71       3  1.484467e-01
    ## 72       1  3.871608e-02
    ## 82       1 -2.285786e-01
    ## 83       1  2.460758e-01
    ## 84       0 -1.749725e-02
    ## 85       0  1.329170e+00
    ## 86       0  8.026610e-01
    ## 88       3  5.499845e-01
    ## 89       3  1.038787e+00
    ## 90       1 -7.206015e-01
    ## 91       0  3.000000e+00
    ## 92       0  3.000000e+00
    ## 98       0  1.380413e+00
    ## 102      0 -3.006351e-04
    ## 103      1 -2.304434e-03
    ## 105      2  8.593218e-02
    ## 106      3  2.565527e-01
    ## 107      2  2.514419e-01
    ## 109      3  2.826516e-02
    ## 110      2  7.894064e-01
    ## 111      2  9.762084e-01
    ## 113      2 -1.187298e-01
    ## 115      0 -1.141973e-01
    ## 116      0  7.117575e-02
    ## 117      0  4.046382e-01
    ## 119      1 -5.918129e-02
    ## 122      0  4.988293e-02
    ## 123      0  1.141801e+00
    ## 126      0  1.000000e+00
    ## 127      0  1.898457e+00
    ## 130      1  1.959742e+00
    ## 132      1  2.000000e+00
    ## 134      2 -9.992033e-01
    ## 141      2  2.081038e+00
    ## 142      2  2.301172e+00
    ## 143     -1  1.868261e+00
    ## 145     -1  2.110833e+00
    ## 146     -1  2.064876e+00
    ## 147     -1  1.573821e+00
    ## 149      3 -1.528815e+00
    ## 151      3 -1.561121e+00
    ## 152     -1 -1.313145e+00
    ## 153      2 -2.159462e+00
    ## 155      3 -1.731642e+00
    ## 157     -1 -1.038716e+00
    ## 158     -1 -1.000000e+00
    ## 159     -1 -6.670997e-01

``` r
plot(x=comparison$predicted, y= comparison$actual,
     xlab='Predicted Values',
     ylab='Actual Values',
     main='Predicted vs. Actual Values')
abline(a=0, b=1)
```

![](Automobile_Dataset_RegressionTesting_files/figure-gfm/unnamed-chunk-13-1.png)<!-- -->

\##Regression Test 2: Full Data, No Highly Correlated Values

``` r
auto.lm2= lm(symboling~.-city.mpg -highway.mpg -length -width -fuel.type -price , data=train)
summary(auto.lm2)
```

    ## 
    ## Call:
    ## lm(formula = symboling ~ . - city.mpg - highway.mpg - length - 
    ##     width - fuel.type - price, data = train)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -0.6027 -0.1883  0.0000  0.1395  0.9693 
    ## 
    ## Coefficients: (3 not defined because of singularities)
    ##                         Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)           27.4992857  8.1769102   3.363 0.001878 ** 
    ## makebmw               -3.3955295  1.0451374  -3.249 0.002560 ** 
    ## makechevrolet         -2.4526168  1.3651021  -1.797 0.081026 .  
    ## makedodge             -2.8273446  1.3350696  -2.118 0.041373 *  
    ## makehonda             -1.8435018  1.3979474  -1.319 0.195827    
    ## makemazda             -2.1399341  1.1948792  -1.791 0.081958 .  
    ## makemercedes-benz     -4.7580831  1.1328664  -4.200 0.000174 ***
    ## makemitsubishi        -1.2230293  1.2316098  -0.993 0.327510    
    ## makenissan            -1.5837227  1.1652577  -1.359 0.182803    
    ## makepeugot            -4.3110767  3.0815491  -1.399 0.170614    
    ## makeplymouth          -2.2174998  1.2867008  -1.723 0.093641 .  
    ## makeporsche           -0.2266005  1.2225936  -0.185 0.854029    
    ## makesaab               1.1796540  1.0144800   1.163 0.252771    
    ## makesubaru            -2.9682086  1.5968017  -1.859 0.071474 .  
    ## maketoyota            -2.7792225  1.2170514  -2.284 0.028579 *  
    ## makevolkswagen         0.1113134  1.2186502   0.091 0.927742    
    ## makevolvo             -3.8233826  1.1910313  -3.210 0.002841 ** 
    ## normalized.losses     -0.0057151  0.0055947  -1.022 0.314016    
    ## aspirationturbo        0.9063443  0.2761938   3.282 0.002344 ** 
    ## num.of.doorstwo        0.3264888  0.2217708   1.472 0.149905    
    ## body.stylehardtop     -0.0642586  0.5419757  -0.119 0.906299    
    ## body.stylehatchback   -1.1265450  0.6278326  -1.794 0.081401 .  
    ## body.stylesedan       -1.4970524  0.5954674  -2.514 0.016685 *  
    ## body.stylewagon       -1.3784688  0.6251420  -2.205 0.034114 *  
    ## drive.wheelsfwd        0.3077233  0.5006423   0.615 0.542757    
    ## drive.wheelsrwd        1.4040369  0.7552220   1.859 0.071436 .  
    ## wheel.base            -0.0842118  0.0462588  -1.820 0.077250 .  
    ## height                -0.1155686  0.0629245  -1.837 0.074770 .  
    ## curb.weight            0.0004084  0.0006283   0.650 0.519946    
    ## engine.typel           1.4996269  2.0718178   0.724 0.473986    
    ## engine.typeohc        -0.6434259  0.8424678  -0.764 0.450141    
    ## engine.typeohcf               NA         NA      NA       NA    
    ## engine.typeohcv       -1.1479488  1.4549425  -0.789 0.435425    
    ## num.of.cylindersfour   0.8718126  1.5031247   0.580 0.565630    
    ## num.of.cylinderssix           NA         NA      NA       NA    
    ## num.of.cylindersthree         NA         NA      NA       NA    
    ## engine.size            0.0523719  0.0278639   1.880 0.068514 .  
    ## fuel.system2bbl        0.4639190  0.5435595   0.853 0.399195    
    ## fuel.systemidi         1.5300045  2.6552727   0.576 0.568158    
    ## fuel.systemmfi         3.3071860  0.8969179   3.687 0.000763 ***
    ## fuel.systemmpfi        0.6055190  0.4967910   1.219 0.231049    
    ## fuel.systemspdi       -0.1790121  0.6896304  -0.260 0.796711    
    ## bore                  -2.3496771  1.8714967  -1.256 0.217612    
    ## stroke                -1.6647221  1.4254434  -1.168 0.250756    
    ## compression.ratio     -0.1555369  0.2010693  -0.774 0.444391    
    ## horsepower            -0.0360400  0.0104662  -3.443 0.001507 ** 
    ## peak.rpm               0.0002065  0.0003302   0.625 0.535898    
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.4144 on 35 degrees of freedom
    ## Multiple R-squared:  0.9424, Adjusted R-squared:  0.8716 
    ## F-statistic: 13.32 on 43 and 35 DF,  p-value: 2.03e-12

*By removing variables that we saw were highly correlated in the
correlation plot and leaving only the percieved “root” feature we saw a
significant improvement in the R^2 value*

``` r
comparison2 <- data.frame(actual= test$symboling, predicted=predict(auto.lm2))
comparison2
```

    ##     actual     predicted
    ## 3        2  1.000000e+00
    ## 7        1  1.520129e-01
    ## 8        2 -1.520129e-01
    ## 9        0  2.000000e+00
    ## 10       0  1.000000e+00
    ## 14       1  1.000000e+00
    ## 19       1  3.000000e+00
    ## 20       1  1.856395e+00
    ## 22       1  1.186705e+00
    ## 26       1 -3.205104e-01
    ## 27      -1  5.109875e-01
    ## 29       2 -1.306747e-01
    ## 30       1 -1.029024e-01
    ## 31       1  1.909584e-14
    ## 34       0  1.228727e+00
    ## 35       0  1.232811e+00
    ## 39       1  6.766302e-01
    ## 42       0 -1.381681e-01
    ## 47       1  1.154632e-14
    ## 50       1  2.263352e+00
    ## 55       1  2.030712e+00
    ## 56       0  1.254599e+00
    ## 57       1  1.270935e+00
    ## 58       0  5.776633e-01
    ## 59       0 -3.972619e-01
    ## 61      -1  8.416380e-01
    ## 62      -1  8.668290e-01
    ## 63      -1  5.827989e-01
    ## 64       3  9.606614e-01
    ## 65       2  8.803067e-01
    ## 66       2  1.420945e+00
    ## 69       1  2.195933e+00
    ## 71       3  1.852447e-01
    ## 72       1  6.564418e-02
    ## 82       1 -1.570454e-01
    ## 83       1  2.455108e-01
    ## 84       0 -8.846544e-02
    ## 85       0  1.404171e+00
    ## 86       0  8.195986e-01
    ## 88       3  5.361446e-01
    ## 89       3  1.037620e+00
    ## 90       1 -7.975340e-01
    ## 91       0  3.000000e+00
    ## 92       0  3.000000e+00
    ## 98       0  1.411307e+00
    ## 102      0 -3.270949e-02
    ## 103      1 -1.938273e-02
    ## 105      2 -3.558101e-02
    ## 106      3  3.204259e-01
    ## 107      2  1.795228e-01
    ## 109      3  1.764172e-01
    ## 110      2  8.403071e-01
    ## 111      2  8.627700e-01
    ## 113      2 -9.208756e-02
    ## 115      0 -1.408362e-01
    ## 116      0  2.996849e-02
    ## 117      0  4.350253e-01
    ## 119      1 -1.804438e-01
    ## 122      0  7.717879e-02
    ## 123      0  1.195933e+00
    ## 126      0  1.000000e+00
    ## 127      0  1.873649e+00
    ## 130      1  1.930419e+00
    ## 132      1  2.000000e+00
    ## 134      2 -8.318821e-01
    ## 141      2  1.988478e+00
    ## 142      2  2.329385e+00
    ## 143     -1  1.823236e+00
    ## 145     -1  2.189873e+00
    ## 146     -1  2.113463e+00
    ## 147     -1  1.555565e+00
    ## 149      3 -1.473235e+00
    ## 151      3 -1.463841e+00
    ## 152     -1 -1.286059e+00
    ## 153      2 -2.193499e+00
    ## 155      3 -1.734496e+00
    ## 157     -1 -1.065644e+00
    ## 158     -1 -1.000000e+00
    ## 159     -1 -7.832260e-01

``` r
plot(x=comparison2$predicted, y= comparison2$actual,
     xlab='Predicted Values',
     ylab='Actual Values',
     main='Predicted vs. Actual Values')
abline(a=0, b=1)
```

![](Automobile_Dataset_RegressionTesting_files/figure-gfm/unnamed-chunk-16-1.png)<!-- -->

\##Creating a test/train set with log transformed variables

``` r
dt = sort(sample(nrow(auto_data.c), nrow(auto_data.c)*.5))
train2<-auto_data.c[dt,]
test2<-auto_data.c[-dt,]

#Log normalizing skewed variables of interest
train2$normalized.losses = log(train2$normalized.losses)
test2$normalized.losses = log(test2$normalized.losses)

train2$price = log(train2$price)
test2$price = log(test2$price)

#Adding 4 because log(0) is undefined
train2$symboling = log(4+train2$symboling)
test2$symboling = log(4+test2$symboling)

test2= test2[-c(1),]
```

``` r
auto.lm1_2= lm(symboling~. , data=train2)
summary(auto.lm2)
```

    ## 
    ## Call:
    ## lm(formula = symboling ~ . - city.mpg - highway.mpg - length - 
    ##     width - fuel.type - price, data = train)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -0.6027 -0.1883  0.0000  0.1395  0.9693 
    ## 
    ## Coefficients: (3 not defined because of singularities)
    ##                         Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)           27.4992857  8.1769102   3.363 0.001878 ** 
    ## makebmw               -3.3955295  1.0451374  -3.249 0.002560 ** 
    ## makechevrolet         -2.4526168  1.3651021  -1.797 0.081026 .  
    ## makedodge             -2.8273446  1.3350696  -2.118 0.041373 *  
    ## makehonda             -1.8435018  1.3979474  -1.319 0.195827    
    ## makemazda             -2.1399341  1.1948792  -1.791 0.081958 .  
    ## makemercedes-benz     -4.7580831  1.1328664  -4.200 0.000174 ***
    ## makemitsubishi        -1.2230293  1.2316098  -0.993 0.327510    
    ## makenissan            -1.5837227  1.1652577  -1.359 0.182803    
    ## makepeugot            -4.3110767  3.0815491  -1.399 0.170614    
    ## makeplymouth          -2.2174998  1.2867008  -1.723 0.093641 .  
    ## makeporsche           -0.2266005  1.2225936  -0.185 0.854029    
    ## makesaab               1.1796540  1.0144800   1.163 0.252771    
    ## makesubaru            -2.9682086  1.5968017  -1.859 0.071474 .  
    ## maketoyota            -2.7792225  1.2170514  -2.284 0.028579 *  
    ## makevolkswagen         0.1113134  1.2186502   0.091 0.927742    
    ## makevolvo             -3.8233826  1.1910313  -3.210 0.002841 ** 
    ## normalized.losses     -0.0057151  0.0055947  -1.022 0.314016    
    ## aspirationturbo        0.9063443  0.2761938   3.282 0.002344 ** 
    ## num.of.doorstwo        0.3264888  0.2217708   1.472 0.149905    
    ## body.stylehardtop     -0.0642586  0.5419757  -0.119 0.906299    
    ## body.stylehatchback   -1.1265450  0.6278326  -1.794 0.081401 .  
    ## body.stylesedan       -1.4970524  0.5954674  -2.514 0.016685 *  
    ## body.stylewagon       -1.3784688  0.6251420  -2.205 0.034114 *  
    ## drive.wheelsfwd        0.3077233  0.5006423   0.615 0.542757    
    ## drive.wheelsrwd        1.4040369  0.7552220   1.859 0.071436 .  
    ## wheel.base            -0.0842118  0.0462588  -1.820 0.077250 .  
    ## height                -0.1155686  0.0629245  -1.837 0.074770 .  
    ## curb.weight            0.0004084  0.0006283   0.650 0.519946    
    ## engine.typel           1.4996269  2.0718178   0.724 0.473986    
    ## engine.typeohc        -0.6434259  0.8424678  -0.764 0.450141    
    ## engine.typeohcf               NA         NA      NA       NA    
    ## engine.typeohcv       -1.1479488  1.4549425  -0.789 0.435425    
    ## num.of.cylindersfour   0.8718126  1.5031247   0.580 0.565630    
    ## num.of.cylinderssix           NA         NA      NA       NA    
    ## num.of.cylindersthree         NA         NA      NA       NA    
    ## engine.size            0.0523719  0.0278639   1.880 0.068514 .  
    ## fuel.system2bbl        0.4639190  0.5435595   0.853 0.399195    
    ## fuel.systemidi         1.5300045  2.6552727   0.576 0.568158    
    ## fuel.systemmfi         3.3071860  0.8969179   3.687 0.000763 ***
    ## fuel.systemmpfi        0.6055190  0.4967910   1.219 0.231049    
    ## fuel.systemspdi       -0.1790121  0.6896304  -0.260 0.796711    
    ## bore                  -2.3496771  1.8714967  -1.256 0.217612    
    ## stroke                -1.6647221  1.4254434  -1.168 0.250756    
    ## compression.ratio     -0.1555369  0.2010693  -0.774 0.444391    
    ## horsepower            -0.0360400  0.0104662  -3.443 0.001507 ** 
    ## peak.rpm               0.0002065  0.0003302   0.625 0.535898    
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.4144 on 35 degrees of freedom
    ## Multiple R-squared:  0.9424, Adjusted R-squared:  0.8716 
    ## F-statistic: 13.32 on 43 and 35 DF,  p-value: 2.03e-12

``` r
comparison1_2 <- data.frame(actual= test2$symboling, predicted=predict(auto.lm1_2))
comparison1_2
```

    ##       actual predicted
    ## 2   1.609438 1.7803011
    ## 4   1.791759 1.6208963
    ## 6   1.386294 1.3862944
    ## 9   1.386294 1.7917595
    ## 11  1.609438 1.3862944
    ## 12  1.609438 1.6297241
    ## 13  1.609438 1.6683042
    ## 17  1.609438 1.5302855
    ## 19  1.098612 1.9459101
    ## 20  1.609438 1.8594400
    ## 21  1.609438 1.8320174
    ## 24  1.386294 1.5529366
    ## 26  1.386294 1.3660601
    ## 28  1.386294 1.3550915
    ## 32  1.386294 1.6094379
    ## 34  1.386294 1.7378321
    ## 37  1.386294 1.5621585
    ## 38  1.609438 1.5811803
    ## 40  1.609438 1.3337366
    ## 41  1.609438 1.6091385
    ## 50  1.386294 1.7651408
    ## 51  1.386294 1.8018658
    ## 52  1.386294 1.8082719
    ## 53  1.098612 1.8497117
    ## 54  1.098612 1.7056364
    ## 61  1.386294 1.4983888
    ## 62  1.098612 1.6732873
    ## 63  1.945910 1.5227876
    ## 64  1.945910 1.5903199
    ## 66  1.609438 1.7394345
    ## 67  1.609438 1.5384896
    ## 68  1.609438 1.6085492
    ## 69  1.098612 1.8341510
    ## 71  1.609438 1.4387113
    ## 72  1.609438 1.3430264
    ## 73  1.386294 1.4244983
    ## 74  1.945910 1.2937057
    ## 76  1.386294 1.9816981
    ## 77  1.386294 1.6713027
    ## 78  1.386294 1.2693432
    ## 80  1.609438 1.4034748
    ## 81  1.609438 1.4310869
    ## 84  1.609438 1.4412725
    ## 85  1.945910 1.6036128
    ## 87  1.791759 1.5374661
    ## 90  1.791759 1.1764092
    ## 91  1.791759 1.9459101
    ## 94  1.386294 1.9579027
    ## 96  1.386294 2.0038796
    ## 97  1.386294 1.7217975
    ## 99  1.386294 1.6756251
    ## 100 1.609438 1.7159414
    ## 104 1.386294 1.3820426
    ## 106 1.386294 1.4770330
    ## 107 1.386294 1.4764305
    ## 108 1.386294 1.4990955
    ## 109 1.386294 1.2888226
    ## 110 1.386294 1.5730903
    ## 112 1.386294 1.4512020
    ## 114 1.609438 1.4624932
    ## 118 1.609438 1.3700877
    ## 119 1.609438 1.4997836
    ## 126 1.791759 1.6214304
    ## 129 1.791759 1.7565306
    ## 130 1.791759 1.7493679
    ## 131 1.098612 1.7597347
    ## 133 1.098612 1.1108486
    ## 135 1.791759 1.1681310
    ## 136 1.791759 1.1356121
    ## 138 1.791759 1.8414955
    ## 139 1.791759 1.8307922
    ## 140 1.791759 1.3181449
    ## 146 1.791759 1.7607335
    ## 148 1.098612 1.9769361
    ## 149 1.098612 0.7704126
    ## 151 1.098612 0.8120332
    ## 152 1.098612 0.9089877
    ## 153 1.098612 0.8451587
    ## 154 1.098612 0.9400740

``` r
plot(x=comparison1_2$predicted, y= comparison1_2$actual,
     xlab='Predicted Values',
     ylab='Actual Values',
     main='Predicted vs. Actual Values')
abline(a=0, b=1)
```

![](Automobile_Dataset_RegressionTesting_files/figure-gfm/unnamed-chunk-20-1.png)<!-- -->
*We see here that after transforming skewed parameters to their more
normalized forms our R^2 value decreases significantly although the line
fit appears much more reasonable*

Although it is evident a non-linear model would be more suitable these
linear-regression models demonstrate that the physical components of the
car do a great deal in explaining variance in the symbol safety ratings.

\#Unused Ideas Catalog \#SQL -\> dbplyr:
<https://dbplyr.tidyverse.org/articles/sql-translation.html>

``` r
#Transforming integer NA values into mean values
# for (i in 1:ncol(auto_data)) {
#   
#     auto_data[,i][is.na(auto_data[,i])] = mean(auto_data[,i], na.rm=TRUE)
#   
# }
```

``` r
#Transforming integer NA values into mode values
  # library(tidyverse)
  # install.packages("upstartr")
  # library(upstartr)

# for (i in 1:ncol(auto_data)) {
#   
#     auto_data[,i][is.na(auto_data[,i])] = calc_mode(auto_data[,i])
#   
# }

# library(dplyr)
# library(tidyr)
# auto_data$normalized.losses = replace_na(auto_data$normalized.losses,mean(auto_data$normalized.losses, na.rm = TRUE))
# auto_data$num.of.doors = replace_na(auto_data$num.of.doors,calc_mode(auto_data$num.of.doors, na.rm = TRUE))
```

\#Generating sample data for Stack Overflow w/ dput()

``` r
# dput(auto_data[(1:7),1:7])
# 
# make = c("alfa-romero", "alfa-romero", "alfa-romero", 
# "audi", "audi", "audi", "audi")
# symboling = c(3L, 3L, 1L, 2L, 2L, 2L, 1L)
# normalized.losses = c(NA, NA, NA, 164L, 164L, NA, 
# 158L)
# fuel.type = c("gas", "gas", "gas", "gas", "gas", "gas", "gas")
# aspiration = c("std", "std", "std", "std", "std", "std", "std")
# num.of.doors = c("two", "two", "two", "four", "four", "two", "four")
# body.style = c("convertible", "convertible","hatchback", "sedan", "sedan", "sedan", "sedan")
# price= c(13495, 18705, NA, 17217, 17293, NA, 18304)
# auto_data_sample= data.frame(make,symboling,fuel.type,aspiration, num.of.doors, body.style, price)
```
