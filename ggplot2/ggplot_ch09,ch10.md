ggplot2 Elegant Graphics for Data Analysis
================
true
2018년 2월

<style>
mystyle{
    font-family :  Georgia;
    font-size : 26px;
    color : PaleVioletRed  ;
}
</style>

# Chapter 9

> <mystyle> Data Analysis </mystyle>

시작하기 전에, 8장을 복습하겠다.

``` r
dtemp <- data_frame(
  months = factor(rep(substr(month.name, 1, 3), 4), 
                  levels = substr(month.name, 1, 3)),
                    city = rep(c("Tokyo", "New York", "Berlin", "London"), each = 12),
                    temp = c(7.0, 6.9, 9.5, 14.5, 18.2, 21.5, 25.2, 26.5, 23.3, 18.3, 13.9, 9.6,
                             -0.2, 0.8, 5.7, 11.3, 17.0, 22.0, 24.8, 24.1, 20.1, 14.1, 8.6, 2.5,
                             -0.9, 0.6, 3.5, 8.4, 13.5, 17.0, 18.6, 17.9, 14.3, 9.0, 3.9, 1.0,
                             3.9, 4.2, 5.7, 8.5, 11.9, 15.2, 17.0, 16.6, 14.2, 10.3, 6.6, 4.8))
```

    ## Warning: `data_frame()` is deprecated, use `tibble()`.
    ## This warning is displayed once per session.

``` r
ggplot(dtemp, aes(x = months, y = temp, group = city, color = city)) +
    geom_line() +
    geom_point(size = 1.1) +
    ggtitle("Monthly Average Temperature") +
    theme_wsj() +
    theme(panel.background = element_rect(fill = "white")) +
    scale_color_wsj("colors6", "")
```

![](ggplot_ch09,-ch10_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->

`substr(month.name, 1, 3)` 과 `month.abb`는 동일하다.

Tidy Data란?

1.  Each variable must have its own column.
2.  Each observation must have its own row.
3.  Each value must have its own cell.

<!-- end list -->

``` r
ec2 <- read_csv(file.choose())
```

  - 월은 열에 저장한다.
  - 년은 열의 이름에 분산시킨다
  - rate은 각 셀의 값이다

두 가지 중요한 도구가 있다. - Spread & gather - Separate & unite

# Spread and Gather

![](figures/capture15.png)

첫번째 형태를 indexted data, 두번째 형태를 Cartesian data라고 부른다. 두가지 형태 모두 “A”, “B”,
“C”, “D”가 의미하는 바에 따라 tidy가 될 수 있다. 또한 결측값은 한가지 형태에는 명백하지만 다른 형태에서는
암시적이다. NA는 presence of absence이지만 때때로 absence of presence일 수
있다.

## Gather

  - data : 바꾸고자 하는 데이터 셋
  - key & value : key는 열의 이름으로부터 만들어지는 변수의 이름이고 value는 셀값으로부터 만들어지는 변수의
    이름이다.
  - … : A,B,C,D나 A:D로 명시할 수 있다. gather하지 않으려면 -E, -F와 같이 명시한다.

<!-- end list -->

``` r
gather(ec2, key = year, value = unemp, `2006`:`2015`)
```

`: backtick, advanced R 참고 여기서 열의 이름이 R에서의 표준변수이름이 아니므로`를
사용하였다.

``` r
gather(ec2, key = year, value = unemp, -month)
```

``` r
economics_2 <- gather(ec2, year, rate, `2006`:`2015`, convert = TRUE, na.rm = TRUE)
```

`convert = TRUE`를 사용하면 year를 문자열에서 숫자로 바꿔준다. `na.rm = TRUE`는 데이터가 없는 월을
제거해준다.

장기간이나 계절적 변동을 강조할 수 있다.

``` r
ggplot(economics_2, aes(year + (month - 1) / 12, rate)) +
  geom_line()

ggplot(economics_2, aes(month, rate, group = year)) +
  geom_line(aes(colour = year), size = 1)
```

## Spread

``` r
weather <- data_frame(
  day = rep(1:3, 2),
  obs = rep(c("temp", "rain"), each = 3),
  val = c(c(23, 22, 20), c(0, 0, 5))
)
weather
```

    ## # A tibble: 6 x 3
    ##     day obs     val
    ##   <int> <chr> <dbl>
    ## 1     1 temp     23
    ## 2     2 temp     22
    ## 3     3 temp     20
    ## 4     1 rain      0
    ## 5     2 rain      0
    ## 6     3 rain      5

Spread는 이런 지저분한 indexed form을 tidy Cartesian형태로 바꿔준다.

``` r
spread(weather, key = obs, value = val)
```

    ## # A tibble: 3 x 3
    ##     day  rain  temp
    ##   <int> <dbl> <dbl>
    ## 1     1     0    23
    ## 2     2     0    22
    ## 3     3     5    20

## Exercises

1.  How can you convert back and forth between the economics and
    economics long datasets built into ggplot2?

<!-- end list -->

``` r
gather(economics, key = variable, value = value, -date)
```

    ## # A tibble: 2,870 x 3
    ##    date       variable value
    ##    <date>     <chr>    <dbl>
    ##  1 1967-07-01 pce       507.
    ##  2 1967-08-01 pce       510.
    ##  3 1967-09-01 pce       516.
    ##  4 1967-10-01 pce       512.
    ##  5 1967-11-01 pce       517.
    ##  6 1967-12-01 pce       525.
    ##  7 1968-01-01 pce       531.
    ##  8 1968-02-01 pce       534.
    ##  9 1968-03-01 pce       544.
    ## 10 1968-04-01 pce       544 
    ## # ... with 2,860 more rows

``` r
economics_long
```

    ## # A tibble: 2,870 x 4
    ##    date       variable value  value01
    ##    <date>     <chr>    <dbl>    <dbl>
    ##  1 1967-07-01 pce       507. 0       
    ##  2 1967-08-01 pce       510. 0.000265
    ##  3 1967-09-01 pce       516. 0.000762
    ##  4 1967-10-01 pce       512. 0.000471
    ##  5 1967-11-01 pce       517. 0.000916
    ##  6 1967-12-01 pce       525. 0.00157 
    ##  7 1968-01-01 pce       531. 0.00207 
    ##  8 1968-02-01 pce       534. 0.00230 
    ##  9 1968-03-01 pce       544. 0.00322 
    ## 10 1968-04-01 pce       544  0.00319 
    ## # ... with 2,860 more rows

2.  Install the EDAWR package from <https://github.com/rstudio/EDAWR>.
    Tidy the storms, population and tb datasets.

<https://www.gis-blog.com/data-management-with-r-tidyr-part-1/> 를 참고하라

``` r
# devtools::install_github("rstudio/EDAWR")
library(EDAWR)
```

    ## 
    ## Attaching package: 'EDAWR'

    ## The following object is masked from 'package:dplyr':
    ## 
    ##     storms

    ## The following objects are masked from 'package:tidyr':
    ## 
    ##     population, who

``` r
storms
```

    ##     storm wind pressure       date
    ## 1 Alberto  110     1007 2000-08-03
    ## 2    Alex   45     1009 1998-07-27
    ## 3 Allison   65     1005 1995-06-03
    ## 4     Ana   40     1013 1997-06-30
    ## 5  Arlene   50     1010 1999-06-11
    ## 6  Arthur   45     1010 1996-06-17

``` r
cases # tidy data가 아니다 
```

    ##   country  2011  2012  2013
    ## 1      FR  7000  6900  7000
    ## 2      DE  5800  6000  6200
    ## 3      US 15000 14000 13000

``` r
gather(cases, key = year, value = value, -country)
```

    ##   country year value
    ## 1      FR 2011  7000
    ## 2      DE 2011  5800
    ## 3      US 2011 15000
    ## 4      FR 2012  6900
    ## 5      DE 2012  6000
    ## 6      US 2012 14000
    ## 7      FR 2013  7000
    ## 8      DE 2013  6200
    ## 9      US 2013 13000

``` r
pollution
```

    ##       city  size amount
    ## 1 New York large     23
    ## 2 New York small     14
    ## 3   London large     22
    ## 4   London small     16
    ## 5  Beijing large    121
    ## 6  Beijing small     56

![](figures/capture16.png)

# Advantages of tidy data

storm 이름과 압력을 추출하기 위해 간단히 `storms$storm`, `storms$pressure`를 입력하면 된다.

두번째 예시는 “untidy”인 pollution dataset이다. 도시의 이름을 추출하거나 큰 입자의 양을 추출하기 위해서는
`pollution$city[1,3,5]` `pollution$amount[2,4,6]`과 같이 지저분하고 복잡한 코드를
입력해야한다. 데이터가 커지면 더 복잡해질 것이다.

``` r
spread(pollution, key = size, value = amount)
```

    ##       city large small
    ## 1  Beijing   121    56
    ## 2   London    22    16
    ## 3 New York    23    14

![](figures/capture17.png)

![](figures/capture18.png)

## separate & unite

`separate()`는 하나의 열을 다수의 열로 나눌 때 이용 create new df with 3 new columns
“year”, “month”, “day” out of the initial "date
column

``` r
storms.sep <- separate(storms, date, c("year", "month", "day"), sep = "-")
```

`unite()`는 다수의 열을 하나의 열로 합칠 때 이용

``` r
unite(storms.sep, "date", 4:6, sep = "-")
```

    ## # A tibble: 6 x 4
    ##   storm    wind pressure date      
    ##   <chr>   <int>    <int> <chr>     
    ## 1 Alberto   110     1007 2000-08-03
    ## 2 Alex       45     1009 1998-07-27
    ## 3 Allison    65     1005 1995-06-03
    ## 4 Ana        40     1013 1997-06-30
    ## 5 Arlene     50     1010 1999-06-11
    ## 6 Arthur     45     1010 1996-06-17

# Separate and Unite

Spread와 gather이 변수가 잘못된 장소에 위치해 있을 때 도움을 준다면 Separate와 unite는 다수의 변수가
하나의 열로 쑤셔 넣거나 다수의 열로 분산시킬때 사용한다.

``` r
trt <- data_frame(
  var = paste0(rep(c("beg", "end"), each = 3), "_", rep(c("a", "b", "c"))),
  val = c(1, 4, 2, 10, 5, 11)
)
trt
```

    ## # A tibble: 6 x 2
    ##   var     val
    ##   <chr> <dbl>
    ## 1 beg_a     1
    ## 2 beg_b     4
    ## 3 beg_c     2
    ## 4 end_a    10
    ## 5 end_b     5
    ## 6 end_c    11

4개의 인자가 있다.

  - data : 데이터프레임
  - col : 나눌 변수의 이름
  - into : 새 변수의 이름(문자벡터)
  - sep : 변수를 어떻게 나눌 것인지에 대한 설명. 정규표현식도 가능하다.

<!-- end list -->

``` r
separate(trt, var, c("time", "treatment"), "_")
```

    ## # A tibble: 6 x 3
    ##   time  treatment   val
    ##   <chr> <chr>     <dbl>
    ## 1 beg   a             1
    ## 2 beg   b             4
    ## 3 beg   c             2
    ## 4 end   a            10
    ## 5 end   b             5
    ## 6 end   c            11

`extract(data, col, into, regex = "([[:alnum:]]+)", remove = TRUE,
convert = FALSE, ...)`

## Exercises

1.  Install the EDAWR package from <https://github.com/rstudio/EDAWR>.
    Tidy the who dataset.

<!-- end list -->

``` r
data(package = "EDAWR", who)
car::some(who) # WHO의 1995~2013 폐결핵(tuberculosis)데이터 
```

    ## # A tibble: 10 x 60
    ##    country iso2  iso3   year new_sp_m014 new_sp_m1524 new_sp_m2534
    ##    <chr>   <chr> <chr> <int>       <int>        <int>        <int>
    ##  1 Armenia AM    ARM    1986          NA           NA           NA
    ##  2 Belarus BY    BLR    1999          NA           NA           NA
    ##  3 Brunei~ BN    BRN    1994          NA           NA           NA
    ##  4 Gabon   GA    GAB    1982          NA           NA           NA
    ##  5 Kenya   KE    KEN    2013          NA           NA           NA
    ##  6 Malawi  MW    MWI    1985          NA           NA           NA
    ##  7 Marsha~ MH    MHL    1990          NA           NA           NA
    ##  8 Sloven~ SI    SVN    2008           0            3           12
    ##  9 Surina~ SR    SUR    1983          NA           NA           NA
    ## 10 Togo    TG    TGO    1997          NA           NA           NA
    ## # ... with 53 more variables: new_sp_m3544 <int>, new_sp_m4554 <int>,
    ## #   new_sp_m5564 <int>, new_sp_m65 <int>, new_sp_f014 <int>,
    ## #   new_sp_f1524 <int>, new_sp_f2534 <int>, new_sp_f3544 <int>,
    ## #   new_sp_f4554 <int>, new_sp_f5564 <int>, new_sp_f65 <int>,
    ## #   new_sn_m014 <int>, new_sn_m1524 <int>, new_sn_m2534 <int>,
    ## #   new_sn_m3544 <int>, new_sn_m4554 <int>, new_sn_m5564 <int>,
    ## #   new_sn_m65 <int>, new_sn_f014 <int>, new_sn_f1524 <int>,
    ## #   new_sn_f2534 <int>, new_sn_f3544 <int>, new_sn_f4554 <int>,
    ## #   new_sn_f5564 <int>, new_sn_f65 <int>, new_ep_m014 <int>,
    ## #   new_ep_m1524 <int>, new_ep_m2534 <int>, new_ep_m3544 <int>,
    ## #   new_ep_m4554 <int>, new_ep_m5564 <int>, new_ep_m65 <int>,
    ## #   new_ep_f014 <int>, new_ep_f1524 <int>, new_ep_f2534 <int>,
    ## #   new_ep_f3544 <int>, new_ep_f4554 <int>, new_ep_f5564 <int>,
    ## #   new_ep_f65 <int>, new_rel_m014 <int>, new_rel_m1524 <int>,
    ## #   new_rel_m2534 <int>, new_rel_m3544 <int>, new_rel_m4554 <int>,
    ## #   new_rel_m5564 <int>, new_rel_m65 <int>, new_rel_f014 <int>,
    ## #   new_rel_f1524 <int>, new_rel_f2534 <int>, new_rel_f3544 <int>,
    ## #   new_rel_f4554 <int>, new_rel_f5564 <int>, new_rel_f65 <int>

첫번째 3세글자는 TB의 new or old

두번째 두 글자는

  - rel stands for cases of relapse(재발)
  - ep stands for cases of extrapulmonary(폐의 바깥) TB
  - sn stands for cases of pulmonary(폐의) TB that could not be diagnosed
    by a pulmonary smear (smear negative)
  - sp stands for cases of pulmonary TB that could be diagnosed be a
    pulmonary smear (smear positive)

6번째 글자는 TB환자의 성별 f or m

나머지 숫자들은 TB환자의 연령대 014 : 0 ~14, 1424 : 15~24 etc.

``` r
who2 <- who
(who2 <- gather(who2, "code", "value", 5:60))
```

    ## # A tibble: 405,440 x 6
    ##    country     iso2  iso3   year code        value
    ##    <chr>       <chr> <chr> <int> <chr>       <int>
    ##  1 Afghanistan AF    AFG    1980 new_sp_m014    NA
    ##  2 Afghanistan AF    AFG    1981 new_sp_m014    NA
    ##  3 Afghanistan AF    AFG    1982 new_sp_m014    NA
    ##  4 Afghanistan AF    AFG    1983 new_sp_m014    NA
    ##  5 Afghanistan AF    AFG    1984 new_sp_m014    NA
    ##  6 Afghanistan AF    AFG    1985 new_sp_m014    NA
    ##  7 Afghanistan AF    AFG    1986 new_sp_m014    NA
    ##  8 Afghanistan AF    AFG    1987 new_sp_m014    NA
    ##  9 Afghanistan AF    AFG    1988 new_sp_m014    NA
    ## 10 Afghanistan AF    AFG    1989 new_sp_m014    NA
    ## # ... with 405,430 more rows

``` r
(who2 <- separate(who2, code, c("new", "var", "sexage")))
```

    ## # A tibble: 405,440 x 8
    ##    country     iso2  iso3   year new   var   sexage value
    ##    <chr>       <chr> <chr> <int> <chr> <chr> <chr>  <int>
    ##  1 Afghanistan AF    AFG    1980 new   sp    m014      NA
    ##  2 Afghanistan AF    AFG    1981 new   sp    m014      NA
    ##  3 Afghanistan AF    AFG    1982 new   sp    m014      NA
    ##  4 Afghanistan AF    AFG    1983 new   sp    m014      NA
    ##  5 Afghanistan AF    AFG    1984 new   sp    m014      NA
    ##  6 Afghanistan AF    AFG    1985 new   sp    m014      NA
    ##  7 Afghanistan AF    AFG    1986 new   sp    m014      NA
    ##  8 Afghanistan AF    AFG    1987 new   sp    m014      NA
    ##  9 Afghanistan AF    AFG    1988 new   sp    m014      NA
    ## 10 Afghanistan AF    AFG    1989 new   sp    m014      NA
    ## # ... with 405,430 more rows

``` r
(who2 <- separate(who2, sexage, c("sex", "age"), sep = 1)) # sep = 1 기억 
```

    ## # A tibble: 405,440 x 9
    ##    country     iso2  iso3   year new   var   sex   age   value
    ##    <chr>       <chr> <chr> <int> <chr> <chr> <chr> <chr> <int>
    ##  1 Afghanistan AF    AFG    1980 new   sp    m     014      NA
    ##  2 Afghanistan AF    AFG    1981 new   sp    m     014      NA
    ##  3 Afghanistan AF    AFG    1982 new   sp    m     014      NA
    ##  4 Afghanistan AF    AFG    1983 new   sp    m     014      NA
    ##  5 Afghanistan AF    AFG    1984 new   sp    m     014      NA
    ##  6 Afghanistan AF    AFG    1985 new   sp    m     014      NA
    ##  7 Afghanistan AF    AFG    1986 new   sp    m     014      NA
    ##  8 Afghanistan AF    AFG    1987 new   sp    m     014      NA
    ##  9 Afghanistan AF    AFG    1988 new   sp    m     014      NA
    ## 10 Afghanistan AF    AFG    1989 new   sp    m     014      NA
    ## # ... with 405,430 more rows

``` r
(who2 <- spread(who2, var, value))
```

    ## # A tibble: 101,360 x 11
    ##    country     iso2  iso3   year new   sex   age      ep   rel    sn    sp
    ##    <chr>       <chr> <chr> <int> <chr> <chr> <chr> <int> <int> <int> <int>
    ##  1 Afghanistan AF    AFG    1980 new   m     014      NA    NA    NA    NA
    ##  2 Afghanistan AF    AFG    1981 new   m     014      NA    NA    NA    NA
    ##  3 Afghanistan AF    AFG    1982 new   m     014      NA    NA    NA    NA
    ##  4 Afghanistan AF    AFG    1983 new   m     014      NA    NA    NA    NA
    ##  5 Afghanistan AF    AFG    1984 new   m     014      NA    NA    NA    NA
    ##  6 Afghanistan AF    AFG    1985 new   m     014      NA    NA    NA    NA
    ##  7 Afghanistan AF    AFG    1986 new   m     014      NA    NA    NA    NA
    ##  8 Afghanistan AF    AFG    1987 new   m     014      NA    NA    NA    NA
    ##  9 Afghanistan AF    AFG    1988 new   m     014      NA    NA    NA    NA
    ## 10 Afghanistan AF    AFG    1989 new   m     014      NA    NA    NA    NA
    ## # ... with 101,350 more rows

2.  Work through the demos included in the tidyr package (demo(package =
    “tidyr”))

<!-- end list -->

``` r
grades <- tbl_df(read.table(header = TRUE, text = "
   ID   Test Year   Fall Spring Winter
    1   1   2008    15      16      19
    1   1   2009    12      13      27
    1   2   2008    22      22      24
    1   2   2009    10      14      20
    2   1   2008    12      13      25
    2   1   2009    16      14      21
    2   2   2008    13      11      29
    2   2   2009    23      20      26
    3   1   2008    11      12      22
    3   1   2009    13      11      27
    3   2   2008    17      12      23
    3   2   2009    14      9       31
   "))

grades %>%
    gather(Semester, Score, Fall:Winter) %>%
    mutate(Test = paste0("Test", Test),
           Semester = factor(Semester, ordered = TRUE, levels = c("Spring", "Fall", "Winter"))) %>% #변수로 바꿔야 하기 때문에
    spread(Test, Score) %>%
    arrange(ID, Year, Semester)
```

    ## # A tibble: 18 x 5
    ##       ID  Year Semester Test1 Test2
    ##    <int> <int> <ord>    <int> <int>
    ##  1     1  2008 Spring      16    22
    ##  2     1  2008 Fall        15    22
    ##  3     1  2008 Winter      19    24
    ##  4     1  2009 Spring      13    14
    ##  5     1  2009 Fall        12    10
    ##  6     1  2009 Winter      27    20
    ##  7     2  2008 Spring      13    11
    ##  8     2  2008 Fall        12    13
    ##  9     2  2008 Winter      25    29
    ## 10     2  2009 Spring      14    20
    ## 11     2  2009 Fall        16    23
    ## 12     2  2009 Winter      21    26
    ## 13     3  2008 Spring      12    12
    ## 14     3  2008 Fall        11    17
    ## 15     3  2008 Winter      22    23
    ## 16     3  2009 Spring      11     9
    ## 17     3  2009 Fall        13    14
    ## 18     3  2009 Winter      27    31

``` r
set.seed(10)
activities <- data_frame(
  id = sprintf("x1.%02d", 1:10),
  trt = sample(c("cnt", "tr"), 10, replace = TRUE),
  work.T1 = runif(10),
  play.T1 = runif(10),
  talk.T1 = runif(10),
  work.T2 = runif(10),
  play.T2 = runif(10),
  talk.T2 = runif(10)
)
activities %>% 
  gather(key , value, -id, -trt) %>% 
  separate(key, into = c("location", "time")) %>% 
  arrange(id, trt, time) %>% 
  spread(location, value)
```

    ## # A tibble: 20 x 6
    ##    id    trt   time   play   talk   work
    ##    <chr> <chr> <chr> <dbl>  <dbl>  <dbl>
    ##  1 x1.01 tr    T1    0.865 0.536  0.652 
    ##  2 x1.01 tr    T2    0.354 0.0319 0.275 
    ##  3 x1.02 cnt   T1    0.615 0.0931 0.568 
    ##  4 x1.02 cnt   T2    0.936 0.114  0.229 
    ##  5 x1.03 cnt   T1    0.775 0.170  0.114 
    ##  6 x1.03 cnt   T2    0.246 0.469  0.0144
    ##  7 x1.04 tr    T1    0.356 0.900  0.596 
    ##  8 x1.04 tr    T2    0.473 0.397  0.729 
    ##  9 x1.05 cnt   T1    0.406 0.423  0.358 
    ## 10 x1.05 cnt   T2    0.192 0.834  0.250 
    ## 11 x1.06 cnt   T1    0.707 0.748  0.429 
    ## 12 x1.06 cnt   T2    0.583 0.761  0.161 
    ## 13 x1.07 cnt   T1    0.838 0.823  0.0519
    ## 14 x1.07 cnt   T2    0.459 0.573  0.0170
    ## 15 x1.08 cnt   T1    0.240 0.955  0.264 
    ## 16 x1.08 cnt   T2    0.467 0.448  0.486 
    ## 17 x1.09 tr    T1    0.771 0.685  0.399 
    ## 18 x1.09 tr    T2    0.400 0.0838 0.103 
    ## 19 x1.10 cnt   T1    0.356 0.501  0.836 
    ## 20 x1.10 cnt   T2    0.505 0.219  0.802

separate의 sep인자는 디폴트가 non-alphanumeric값의 나열이다.

# Case Studies

``` r
scores <- dplyr::data_frame(
  person = rep(c("Greg", "Sally", "Sue"), each = 2),
  time = rep(c("pre", "post"), 3),
  test1 = round(rnorm(6, mean = 80, sd = 4), 0),
  test2 = round(jitter(test1, 15), 0)
)
scores_1 <- gather(scores, key = test, value = score, 3:4)
scores_2 <- spread(scores_1, key = time, value = score)
scores_3 <- mutate(scores_2, diff = post - pre)

ggplot(scores_3, aes(person, diff, colour = test)) +
  geom_hline(size = 2, colour = "white", yintercept = 0) +
  geom_point() +
  geom_path(aes(group = person), colour = "grey50",
            arrow = arrow(length = unit(0.25, "cm")))
```

![](ggplot_ch09,-ch10_files/figure-gfm/unnamed-chunk-26-1.png)<!-- -->

# Summary

  - `convert = TRUE`와 `na.rm = TRUE`를 활용하자.
  - `sprintf("x1.%02d", 1:10)`를 활용
  - tidy data로 만든 후 분석하자.
  - 시간의 전후 데이터는 차이를 시각화 해보자.

# Chapter 10

> <mystyle> Data Transformation </mystyle>

# Filter Observations

좋은 데이터 분석 방법은 하나의 관측 단위에 대해 다른 단위에 대해 결론을 일반화하도록 시도하기 전에 어떻게 작동하는지를 이해하는
것이다. 소규모의 집합에 집중하고 마스터해서 더 큰 집합에 집중하고 전체 데이터셋에 결론을 적용시키는 것이다.

필터링은 또한 이상치를 추출하는데 유용하다. 일반적으로 단순히 이상치를 버리는것을 원하지 않는다. 넓은 트랜드를 요약한다. 어떻게
진행되고있는지를 이해할 수 있다면 이상치를 보기위해 조사하자.

``` r
ggplot(diamonds, aes(x, y)) +
  geom_bin2d()
```

![](ggplot_ch09,-ch10_files/figure-gfm/unnamed-chunk-27-1.png)<!-- -->

``` r
filter(diamonds, x == 0 | y == 0) # 이상치 확인
```

    ## # A tibble: 8 x 10
    ##   carat cut       color clarity depth table price     x     y     z
    ##   <dbl> <ord>     <ord> <ord>   <dbl> <dbl> <int> <dbl> <dbl> <dbl>
    ## 1  1.07 Ideal     F     SI2      61.6    56  4954     0  6.62     0
    ## 2  1    Very Good H     VS2      63.3    53  5139     0  0        0
    ## 3  1.14 Fair      G     VS1      57.5    67  6381     0  0        0
    ## 4  1.56 Ideal     G     VS2      62.2    54 12800     0  0        0
    ## 5  1.2  Premium   D     VVS1     62.1    59 15686     0  0        0
    ## 6  2.25 Premium   H     SI2      62.8    59 18034     0  0        0
    ## 7  0.71 Good      F     SI2      64.1    60  2130     0  0        0
    ## 8  0.71 Good      F     SI2      64.1    60  2130     0  0        0

실제 분석에서 데이터 품질 문제의 원인을 찾기위해 이상치의 세부사항을 들여다 본다. 이 경우에는 단순히 제거하고 나서 나머지것들을
보자.

``` r
diamonds_ok <- filter(diamonds, x > 0, y > 0, y < 20)
ggplot(diamonds_ok, aes(x, y)) +
  geom_bin2d() +
  geom_abline(slope = 1, colour = "white", size = 1, alpha = 0.5)
```

![](ggplot_ch09,-ch10_files/figure-gfm/unnamed-chunk-29-1.png)<!-- -->
x, y의 강한 관계를 볼 수 있다. 참조 선을 추가했다. 하지만 이 플랏에는 여전히 문제가 있다.

  - 이 플랏은 대부분 비어있다. 그 이유는 데이터의 대부분은 대각선을 따라 있다.
  - 몇가지 이변량 이상치가 있지만 단순히 `filter`로는 찾기 힘들다.

## Useful Tools

  - `x %in% c("a", "b", "c")`: x is one of the values in the right hand
    side

  - `x & y`: TRUE if both x and y are TRUE

  - `x | y`: TRUE if either x or y (or both) are TRUE

  - `xor(x, y)` : TRUE if either x or y are TRUE, but not both

  - Size is between 1 and 2 carats : `floor(carat) == 1`

  - An average dimension greater than 3 : (x + y + z) / 3 \> 3

좀 더 복잡해지면 새로운 변수를 만들자. 그러면 부분집합 하기 전에 계산을 제대로 했는지 확인 할 수 있다.

## Missing Values

결측치를 찾는 데에는 `is.na(x)`를 사용하라.

## Exercises

1.  Practice your filtering skills by: • A carat smaller than the median
    carat.

<!-- end list -->

``` r
diamonds %>% 
  filter(carat < median(carat))
```

• Cost more than $10,000 per carat

``` r
diamonds %>% 
  filter(price > 10000 & floor(carat))
```

    ## # A tibble: 5,222 x 10
    ##    carat cut       color clarity depth table price     x     y     z
    ##    <dbl> <ord>     <ord> <ord>   <dbl> <dbl> <int> <dbl> <dbl> <dbl>
    ##  1  1.7  Ideal     J     VS2      60.5    58 10002  7.73  7.74  4.68
    ##  2  1.03 Ideal     E     VVS2     60.6    59 10003  6.5   6.53  3.95
    ##  3  1.23 Very Good G     VVS2     60.6    55 10004  6.93  7.02  4.23
    ##  4  1.25 Ideal     F     VS2      61.6    55 10006  6.93  6.96  4.28
    ##  5  2.01 Very Good I     SI2      61.4    63 10009  8.19  7.96  4.96
    ##  6  1.21 Very Good F     VS1      62.3    58 10009  6.76  6.85  4.24
    ##  7  1.51 Premium   I     VS2      59.9    60 10010  7.42  7.36  4.43
    ##  8  1.01 Fair      D     SI2      64.6    58 10011  6.25  6.2   4.02
    ##  9  1.05 Ideal     F     VVS2     60.5    55 10011  6.67  6.58  4.01
    ## 10  1.6  Ideal     J     VS1      62      53 10011  7.57  7.56  4.69
    ## # ... with 5,212 more rows

2.  Repeat the analysis of outlying values with z. Compared to x and y,
    how would you characterise the relationship of x and z, or y and z?

<!-- end list -->

``` r
temp <- diamonds %>% 
  filter(x > 0 & z > 0 & z < 30)
ggplot(temp, aes(x, z)) +
  geom_bin2d()
```

![](ggplot_ch09,-ch10_files/figure-gfm/unnamed-chunk-32-1.png)<!-- -->

3.  Install the ggplot2movies package and look at the movies that have a
    missing budget. How are they different from the movies with a
    budget? (Hint: try a frequency polygon plus colour = is.na(budget).)

<!-- end list -->

``` r
library(ggplot2movies)
```

    ## Warning: package 'ggplot2movies' was built under R version 3.5.2

``` r
temp <- movies
temp <- mutate(temp, budget_na = as.factor(ifelse(is.na(budget), 1, 0)))

ggplot(temp, aes(length, colour = budget_na)) +
  geom_freqpoly(aes(y = ..density..), binwidth = 5) +
  xlim(0, 250)
```

    ## Warning: Removed 90 rows containing non-finite values (stat_bin).

    ## Warning: Removed 4 rows containing missing values (geom_path).

![](ggplot_ch09,-ch10_files/figure-gfm/unnamed-chunk-33-1.png)<!-- -->
이런식으로 다른 변수들도 확인

# Create New Variables

x와 y의 관계를 더 잘 탐구하기 위해 데이터를 대각이 아닌 flat하도록 회전시키는것은 유용하다.

``` r
diamonds_ok2 <- mutate(diamonds_ok,
  sym = x - y,
  size = sqrt(x ^ 2 + y ^ 2)
)
diamonds_ok2
```

    ## # A tibble: 53,930 x 12
    ##    carat cut   color clarity depth table price     x     y     z      sym
    ##    <dbl> <ord> <ord> <ord>   <dbl> <dbl> <int> <dbl> <dbl> <dbl>    <dbl>
    ##  1 0.23  Ideal E     SI2      61.5    55   326  3.95  3.98  2.43 -0.0300 
    ##  2 0.21  Prem~ E     SI1      59.8    61   326  3.89  3.84  2.31  0.05   
    ##  3 0.23  Good  E     VS1      56.9    65   327  4.05  4.07  2.31 -0.02   
    ##  4 0.290 Prem~ I     VS2      62.4    58   334  4.2   4.23  2.63 -0.03   
    ##  5 0.31  Good  J     SI2      63.3    58   335  4.34  4.35  2.75 -0.01000
    ##  6 0.24  Very~ J     VVS2     62.8    57   336  3.94  3.96  2.48 -0.02   
    ##  7 0.24  Very~ I     VVS1     62.3    57   336  3.95  3.98  2.47 -0.0300 
    ##  8 0.26  Very~ H     SI1      61.9    55   337  4.07  4.11  2.53 -0.04   
    ##  9 0.22  Fair  E     VS2      65.1    61   337  3.87  3.78  2.49  0.09   
    ## 10 0.23  Very~ H     VS1      59.4    61   338  4     4.05  2.39 -0.0500 
    ## # ... with 53,920 more rows, and 1 more variable: size <dbl>

``` r
ggplot(diamonds_ok2, aes(size, sym)) +
  stat_bin2d()
```

![](ggplot_ch09,-ch10_files/figure-gfm/unnamed-chunk-35-1.png)<!-- --> 이
플랏은 두가지 이점을 가지고 있다: 대부분의 다이아로부터 패턴을 쉽게 볼 수 있고, 이상치를 쉽게 선택할 수 있다.

``` r
ggplot(diamonds_ok2, aes(abs(sym))) +
  geom_histogram(binwidth = 0.10)
```

![](ggplot_ch09,-ch10_files/figure-gfm/unnamed-chunk-36-1.png)<!-- -->

``` r
diamonds_ok3 <- filter(diamonds_ok2, abs(sym) < 0.20)
ggplot(diamonds_ok3, aes(abs(sym))) +
  geom_histogram(binwidth = 0.01)
```

![](ggplot_ch09,-ch10_files/figure-gfm/unnamed-chunk-36-2.png)<!-- -->
대부분의 다이아가 대칭에 가깝지만 완벽한 대칭은 거의 없음을 알 수 있다.

## Useful Tools

일반적으로, 변환은 기본지식에 달려있다. 하지만 놀랍게도 넓은 범위의 상황에서 유용한 몇가지 변환이 있다.

  - Log-transformations. <span style="background : yellow"> 곱의 관계에 있는
    것들을 합의 관계로 바꾸어준다. 즉 선형관계로 바꾸어준다.</span> additive model : `Data
    = Seasonal effect + Trend + Cylical + Residual` Multiplicative model
    : `Data = Seasonal effect X Trend X Cylical X Residual -> log(Data)
    = log(Seasonal effect) + log(Trend) + log(Cyclical) + log(Residual)`

<http://stats.stackexchange.com/questions/27951> 를 참고하자

![](figures/capture19.png)

log를 취하기전에는 알 수 없었던 하락이 log를 취하면 알 수 있다. 세계2차대전때문에 감소한 것이다.

![](figures/capture20.png)

  - Relative differnce : <span style="background : yellow"> 두 변수간의 상대적
    차이에 관심이 있다면, `log(x / y)` 를 사용하라.</span> x / y 보다 좋은 이유는 대칭이기
    때문이다. 만약 x \< y이면, x / y는 \[0, 1), 하지만 만약 x \> y이면 x / y는 (1,
    Inf)이다.

  - 때때로 적분하거나 미분하는것이 좀 더 해석하기 좋다. 만약 거리와 시간을 가지고 있다면 속도나 가속도가 좀 더 유용하거나
    거꾸로 이거나. <span style="background : yellow"> 적분은 데이터를 좀 더 매끄럽게 하고
    미분은 덜 매끄럽게 한다.</span>

  - 수를 `abs(x) and sign(x)`로 규모와 방향으로 나누어라.

  - 이전에 봤던 것 처럼 전반적인 크기와 차이를 나누는 것은 유용하다

  - 강한 추세를 볼 수 있다면, 패턴과 잔차로 나누기 위해 모델링 하는것은 유용하다.

  - 때때로 position을 극좌표로 바꾸는것은 유용하다(거꾸로하거나). distance (sqrt(x^2 + y^2))
    and angle(atan2(y, x))

## Exercises

1.  Practice your variable creation skills by creating the following new
    variables: • The approximate volume of the diamond (using x, y, and
    z).

사각뿔 부피 + 사다리꼴(옆면) 부피하면 될 것 같다. 사각뿔은 depth 곱하기 z / 100이 높이고 가로 세로 x, y인
직사각형을 밑면으로 하는 사각뿔의 부피. 사다리꼴부피는 table width = table 곱하기 x / 100임을 이용해
구한다.

• The approximate density of the diamond.

1캐럿이 0.2g이므로 carat / volume

• The price per carat.

price / carat

• Log transformation of carat and price.

log(carat) log(price)

2.  How can you improve the data density of ggplot(diamonds, aes(x, z))
    + stat bin2d(). What transformation makes it easier to extract
    outliers?

우선 x = 0 , z = 0은 제거하자.

``` r
temp <- filter(diamonds, x > 0 & z > 0)
ggplot(temp, aes(x, z)) +
  stat_binhex()
```

![](ggplot_ch09,-ch10_files/figure-gfm/unnamed-chunk-37-1.png)<!-- -->

``` r
library(car)
```

    ## Warning: package 'car' was built under R version 3.5.3

    ## Loading required package: carData

    ## 
    ## Attaching package: 'car'

    ## The following object is masked from 'package:dplyr':
    ## 
    ##     recode

    ## The following object is masked from 'package:purrr':
    ## 
    ##     some

``` r
model <- lm(z ~ x, data = temp)
summary(model)
```

    ## 
    ## Call:
    ## lm(formula = z ~ x, data = temp)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -3.0963 -0.0398  0.0047  0.0447 28.6344 
    ## 
    ## Coefficients:
    ##              Estimate Std. Error  t value Pr(>|t|)    
    ## (Intercept) 0.0313385  0.0034770    9.013   <2e-16 ***
    ## x           0.6121661  0.0005954 1028.197   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.1548 on 53918 degrees of freedom
    ## Multiple R-squared:  0.9515, Adjusted R-squared:  0.9515 
    ## F-statistic: 1.057e+06 on 1 and 53918 DF,  p-value: < 2.2e-16

``` r
par(mfrow = c(2,2))
plot(model)
```

![](ggplot_ch09,-ch10_files/figure-gfm/unnamed-chunk-37-2.png)<!-- -->
log를 취하자

``` r
model <- update(model, log(z) ~ log(x))
summary(model)
```

    ## 
    ## Call:
    ## lm(formula = log(z) ~ log(x), data = temp)
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -1.33922 -0.01141  0.00194  0.01309  2.30762 
    ## 
    ## Coefficients:
    ##               Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept) -0.4691850  0.0010598  -442.7   <2e-16 ***
    ## log(x)       0.9925738  0.0006098  1627.8   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.02743 on 53918 degrees of freedom
    ## Multiple R-squared:  0.9801, Adjusted R-squared:  0.9801 
    ## F-statistic: 2.65e+06 on 1 and 53918 DF,  p-value: < 2.2e-16

``` r
plot(model)
```

![](ggplot_ch09,-ch10_files/figure-gfm/unnamed-chunk-38-1.png)<!-- -->![](ggplot_ch09,-ch10_files/figure-gfm/unnamed-chunk-38-2.png)<!-- -->![](ggplot_ch09,-ch10_files/figure-gfm/unnamed-chunk-38-3.png)<!-- -->![](ggplot_ch09,-ch10_files/figure-gfm/unnamed-chunk-38-4.png)<!-- -->

``` r
ggplot(temp, aes(log(x), log(z))) +
  stat_binhex()
```

![](ggplot_ch09,-ch10_files/figure-gfm/unnamed-chunk-38-5.png)<!-- -->

``` r
outlierTest(model)
```

    ##        rstudent unadjusted p-value Bonferroni p
    ## 14628 -49.94746         0.0000e+00   0.0000e+00
    ## 20686 -40.57509         0.0000e+00   0.0000e+00
    ## 21646 -43.08113         0.0000e+00   0.0000e+00
    ## 48394  90.27568         0.0000e+00   0.0000e+00
    ## 49887  17.95928         6.5839e-72   3.5501e-67
    ## 24059  17.58877         4.6864e-69   2.5269e-64
    ## 49173  17.38667         1.5922e-67   8.5854e-63
    ## 34266  16.99033         1.4269e-64   7.6940e-60
    ## 33086 -16.10315         3.3159e-58   1.7879e-53
    ## 47122 -13.77312         4.3766e-43   2.3599e-38

4.  Compare the distribution of symmetry for diamonds with x \> y vs. x
    \<
y.

<!-- end list -->

``` r
diamonds_ok2$haha <- as.factor(ifelse(diamonds_ok2$x > diamonds_ok2$y,1, 0))
ggplot(diamonds_ok2, aes(abs(sym), y = ..density.., colour = haha)) +
  geom_freqpoly(binwidth = 0.005) +
  xlim(NA, 0.2)
```

    ## Warning: Removed 96 rows containing non-finite values (stat_bin).

    ## Warning: Removed 2 rows containing missing values (geom_path).

![](ggplot_ch09,-ch10_files/figure-gfm/unnamed-chunk-39-1.png)<!-- -->
도수 다각형은 범주의 레벨간에 걸쳐 분포를 비교할 때 적합하다. geom\_density()의 사용은 3장을 참고하라.
geom\_freqpoly()의 사용은 5장을 참고하라.

# Group\_wise Summaries

``` r
sum_clarity <- diamonds %>% 
  group_by(clarity) %>% 
  summarise(price = mean(price))
ggplot(sum_clarity, aes(clarity, price)) +
  geom_line(aes(group = 1), colour = "grey80") +  # 주의하자
  geom_point(size = 2)
```

![](ggplot_ch09,-ch10_files/figure-gfm/unnamed-chunk-40-1.png)<!-- -->
품질이 좋은게 더 싼 것 같다. 이상하다.

``` r
cut_depth <- diamonds %>% 
  group_by(cut, depth) %>% 
  summarise(n = n()) %>% 
  filter(depth > 55, depth < 70)
cut_depth
```

    ## # A tibble: 455 x 3
    ## # Groups:   cut [5]
    ##    cut   depth     n
    ##    <ord> <dbl> <int>
    ##  1 Fair   55.1     3
    ##  2 Fair   55.2     6
    ##  3 Fair   55.3     5
    ##  4 Fair   55.4     2
    ##  5 Fair   55.5     3
    ##  6 Fair   55.6     4
    ##  7 Fair   55.8     7
    ##  8 Fair   55.9     9
    ##  9 Fair   56       5
    ## 10 Fair   56.1     5
    ## # ... with 445 more rows

``` r
ggplot(cut_depth, aes(depth, n, colour = cut)) +
  geom_line()
```

![](ggplot_ch09,-ch10_files/figure-gfm/unnamed-chunk-42-1.png)<!-- -->

``` r
cut_depth <- mutate(cut_depth, prop = n / sum(n))
ggplot(cut_depth, aes(depth, prop, colour = cut)) +
  geom_line()
```

![](ggplot_ch09,-ch10_files/figure-gfm/unnamed-chunk-43-1.png)<!-- -->

``` r
ggplot(diamonds, aes(depth, colour = cut)) +
  geom_freqpoly(aes(y = ..density..), binwidth = 0.1) +
  xlim(55, 70)
```

    ## Warning: Removed 45 rows containing non-finite values (stat_bin).

    ## Warning: Removed 10 rows containing missing values (geom_path).

![](ggplot_ch09,-ch10_files/figure-gfm/unnamed-chunk-43-2.png)<!-- -->
추가적으로 geom\_line이나 geom\_freqpoly나 여기서는 거의 비슷. geom\_freqpoly가
binwidth를 조절할 수 있으며 새로운 데이터를 만들지 않아도 되므로 geom\_freqpoly가 더 좋아 보인다.

## Useful Tools

  - Counts : n(), n\_distinct(x)
  - Middle : mean(x), median(x)
  - Spread : sd(x), mad(x), IQR(x) 여기서 mad는 median absolute deviation
  - Extremes : quantile(x), min(x), max(x)
  - Positions : first(x), last(x), nth(x, 2)

논리형 벡터와 sum(), mean()을 함께 사용해보자

## Statistical Considerations

평균이나 중앙값으로 요약할 때 갯수와 분산척도를 포함하는 것이 좋다. 이것을 포함하지 않는다면 잘못된 결론을 내릴 수 있다.

``` r
by_clarity <- diamonds %>% 
  group_by(clarity) %>% 
  summarise(
    n = n(),
    mean = mean(price),
    lq = quantile(price, 0.25),
    uq = quantile(price, 0.75)
  )
by_clarity
```

    ## # A tibble: 8 x 5
    ##   clarity     n  mean    lq    uq
    ##   <ord>   <int> <dbl> <dbl> <dbl>
    ## 1 I1        741 3924. 2080  5161 
    ## 2 SI2      9194 5063. 2264  5777.
    ## 3 SI1     13065 3996. 1089  5250 
    ## 4 VS2     12258 3925.  900  6024.
    ## 5 VS1      8171 3839.  876  6023 
    ## 6 VVS2     5066 3284.  794. 3638.
    ## 7 VVS1     3655 2523.  816  2379 
    ## 8 IF       1790 2865.  895  2388.

``` r
ggplot(by_clarity, aes(clarity, mean)) +
  geom_line(aes(group = 1), colour ="grey 50") +
  geom_point(aes(size = n)) +
  geom_linerange(aes(ymin = lq, ymax = uq))
```

![](ggplot_ch09,-ch10_files/figure-gfm/unnamed-chunk-44-1.png)<!-- --> 이
데이터에서는 평균이 좋지 않은 요약 통계량임을 알 수 있다. 가격의 분포가 매우 skewed 되어 있기 때문이다.

``` r
data(Batting, package = "Lahman")
ba <- Batting %>% 
  filter(AB > 0) %>% 
  group_by(playerID) %>% 
  summarise(
    ba = sum(H, na.rm = TRUE) / sum(AB, na.rm = TRUE),
     ab = sum(AB, na.rm = TRUE)
)
ggplot(ba, aes(ba)) +
  geom_histogram(binwidth = 0.01)
```

![](ggplot_ch09,-ch10_files/figure-gfm/unnamed-chunk-45-1.png)<!-- -->
베트를 한번 휘둘러서 공을 친 선수가 많다

``` r
ggplot(ba, aes(ab, ba)) +
  geom_hex(bins = 100) +
  geom_smooth()
```

    ## `geom_smooth()` using method = 'gam' and formula 'y ~ s(x, bs = "cs")'

![](ggplot_ch09,-ch10_files/figure-gfm/unnamed-chunk-46-1.png)<!-- -->

10번 이하로 베트를 휘두르면 제거

``` r
ggplot(filter(ba, ab>= 10), aes(ab, ba)) +
  geom_hex() +
  geom_smooth()
```

    ## `geom_smooth()` using method = 'gam' and formula 'y ~ s(x, bs = "cs")'

![](ggplot_ch09,-ch10_files/figure-gfm/unnamed-chunk-47-1.png)<!-- -->

## Exercises

1.  For each year in the ggplot2movies::movies data determine the
    percent of movies with missing budgets. Visualise the result.

각 년도의 예산결측 비율은 다음과 같다. 다른방법 없을까..

``` r
movies_na <- movies %>%
  mutate(count_na = ifelse(is.na(budget), 1, 0)) %>% 
  group_by(year, count_na) %>% 
  summarise(n = n())

movies_na <- movies_na %>% 
  group_by(year) %>% 
  mutate(sum_n = sum(n)) %>% 
  mutate(prop = n / sum_n) %>% 
  filter(count_na == 1)

ggplot(movies_na, aes(year, prop)) +
  geom_line()
```

![](ggplot_ch09,-ch10_files/figure-gfm/unnamed-chunk-48-1.png)<!-- -->

2.  How does the average length of a movie change over time? Display
    your answer with a plot, including some display of uncertainty.

<!-- end list -->

``` r
movie_length <- movies %>% 
  group_by(year) %>% 
  summarise(
    mean = mean(length),
    n = n(),
     lq = quantile(length, 0.25),
    uq = quantile(length, 0.75)
  )

ggplot(movie_length, aes(year, mean)) +
  geom_line() +
  geom_point(aes(size = n)) +
  geom_linerange(aes(ymin = lq, ymax = uq)) +
  scale_size_area() + 
  coord_cartesian(xlim = c(1980, 2005)) 
```

![](ggplot_ch09,-ch10_files/figure-gfm/unnamed-chunk-49-1.png)<!-- -->

3.  For each combination of diamond quality (e.g. cut, colour and
    clarity), count the number of diamonds, the average price and the
    average size. Visualise the results.

다이아 수

``` r
dia_count <- diamonds %>% 
  group_by(cut, color, clarity) %>% 
  summarise(n = n())
ggplot(dia_count, aes(clarity, color)) +
  geom_tile(aes(fill = n), colour = "grey80") +
  scale_fill_gradient(low = "white", high = "steelblue") +
  geom_text(aes(label = n), vjust = "bottom") +
  facet_wrap(~cut) +
  scale_x_discrete(expand = c(0, 0)) +
  scale_y_discrete(expand = c(0, 0)) +
  theme(panel.grid = element_blank(), panel.background = element_rect(fill = "white"))
```

![](ggplot_ch09,-ch10_files/figure-gfm/unnamed-chunk-50-1.png)<!-- -->
평균 가격

``` r
dia_price <- diamonds %>% 
  group_by(cut, color, clarity) %>% 
  summarise(
    mean = mean(price), 
    n = n(),
    lq = quantile(price, 0.25),
    uq = quantile(price, 0.75)
    )

ggplot(dia_price, aes(clarity, mean, colour = color)) + 
  geom_point(aes(size = n)) +
  geom_linerange(aes(ymin = lq, ymax = uq)) +
  facet_wrap(~cut) +
  scale_size_area() +
  theme(panel.background = element_rect(fill = "white"),
        panel.grid.major = element_line("grey80"),
        panel.grid.minor = element_blank())
```

![](ggplot_ch09,-ch10_files/figure-gfm/unnamed-chunk-51-1.png)<!-- -->

여기서 cut의 영향이 크지 않는것 처럼 보이므로 복잡하니까 cut제거.

``` r
dia_price <- diamonds %>% 
  group_by(color, clarity) %>% 
  summarise(
    mean = mean(price), 
    n = n(),
    lq = quantile(price, 0.25),
    uq = quantile(price, 0.75)
    )
ggplot(dia_price, aes(clarity, mean)) + 
  geom_point(aes(size = n)) +
  geom_linerange(aes(ymin = lq, ymax = uq)) +
  facet_wrap(~color) +
  scale_size_area() +
  theme(panel.background = element_rect(fill = "white"),
        panel.grid.major = element_line("grey80"),
        panel.grid.minor = element_blank())
```

![](ggplot_ch09,-ch10_files/figure-gfm/unnamed-chunk-52-1.png)<!-- -->
품질이 낮은 것들이 highly skewed임을 알 수 있다.

4.  Compute a histogram of carat by “hand” using a binwidth of 0.1.
    Display the results with geom bar(stat = “identity”). (Hint: you
    might need to create a new variable first.)

<!-- end list -->

``` r
temp <- diamonds %>% 
  mutate(carat = cut_width(carat, 0.1)) %>% 
  group_by(carat) %>% 
  summarise(carat_n = n())

ggplot(temp, aes(carat, carat_n)) +
  geom_col() 
```

![](ggplot_ch09,-ch10_files/figure-gfm/unnamed-chunk-53-1.png)<!-- -->

5.  In the baseball example, the batting average seems to increase as
    the number of at bats increases. Why?

<!-- end list -->

``` r
ba <- Batting %>% 
  filter(AB > 0) %>% 
  group_by(playerID) %>% 
  summarise(
    ba = sum(H, na.rm = TRUE) / sum(AB, na.rm = TRUE), ab = sum(AB, na.rm = TRUE),
    G = sum(G, na.rm = TRUE)
)
ggplot(filter(ba, ab>= 10), aes(ab, G)) +
  geom_hex() +
  geom_smooth()
```

    ## `geom_smooth()` using method = 'gam' and formula 'y ~ s(x, bs = "cs")'

![](ggplot_ch09,-ch10_files/figure-gfm/unnamed-chunk-54-1.png)<!-- -->

아무래도 팀에서는 잘하는 선수를 내보낼 테니까 배트를 많이 휘두르는 선수가 잘 칠것이다.

6.  Which player in the Batting dataset has had the most consistently
    good performance over the course of their career?

# Summary

  - x, y관계를 더 잘 탐구하기 위해서 데이터를 대각이 아닌 flat하도록 회전시키는 것은 유용하다
  - 로그변환시 곱의 관계를 선형관계로 바꾸어주는데 변환전 볼 수 없었던 관계가 보일 수 있다.
  - 두 변수간의 상대적 차이에 관심이 있다면 `log(x / y)`
  - 때때로 적분하거나 미분하는것이 해석하기 좋다.
  - 수를 `abs(X)`와 `sign(x)`로 나누어 보라.
  - 강한 추세를 볼 수 있다면 패턴과 잔차를 나누기 위해 모델링하는 것은 유용하다다
  - `geom_freqpoly()`는 범주의 레벨간에 걸쳐 분포를 비교할 때 적합하다. `y = ..density..`와
    `binwidth`를 함께 사용하자.
  - 평균이나 중앙값으로 요약할 때는 분산척도를 포함하자.
  - 테이블 형태의 데이터(두개의 범주형)를 시각화 할때는 히트맵
