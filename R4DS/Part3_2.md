R for Data Science
================
true
2019년 7월

<style>
mystyle{
    font-family :  Georgia;
    font-size : 26px;
    color : PaleVioletRed  ;
}
</style>

> <mystyle> Part 3 </mystyle>  
> <mystyle> Program </mystyle>

참고: <https://statkclee.github.io/parallel-r/ds-fp-purrr.html>

# 함수형 프로그래밍

함수를 리스트로 저장할 수 있다.

``` r
summary <- function(x) {
  funs <- c(mean, median, sd, mad, IQR)
  lapply(funs, function(f) f(x, na.rm = TRUE))
}
```

여기서 lapply의 함수 인자로 기능(functional) + 무기명 함수를 취했다. 기능이란 입력으로 함수를 취하고 출력으로
벡터를 반환하는 함수다.

이 밖에 formula를 리스트로 생성해서 lapply를 사용할 수 있고 데이터를 부트스트랩으로 설정할 수도 있다. 또 이
모형들에서 \(R^2\)도 추출 가능.(advR 211쪽 참고)

## Minor modifications

base::Negate() and plyr::failwith()는 기능(functional)과 함께 혼용하는 경우 아주 편리하고,
사소하지만 유용한 함수 수정을 제공한다. Negate()는 논리형 벡터를 반환하고, 그 함수의 부정(negation)을 반환하는
함수를 취한다. 이것은 사용자가 필요로 하는 것과 반대되는 것을 함수가 반환했을 때 유용한 바로가기가 될 수 있다.

``` r
Negate <- function(f) {
  force(f) # 인자가 연산되는 것을 보장한다
  function(...) !f(...)
}
(Negate(is.null)) (NULL)
```

    ## [1] FALSE

흔이 모든 null 요소를 리스트에서 제거하는 함수인 compact()를 만드는 데 이 아이디어를 사용한다.

``` r
compact <- function(x) Filter(Negate(is.null), x)
```

plyr::failwith()는 오류를 내는 함수를 오류가 있을 때 기본값을 반환하는 함수로 변환한다. dplyr에도 존재.
단순히 오류를 포착하여 실행을 계속하도록 하는 함수인 try()의 래퍼

현재는 purr::possibly()를 사용한다. 뒤에서 purr패키지 공부하면서 보자.

``` r
failwith(NA, log) ("a")
```

    ## Warning: Deprecated: please use `purrr::possibly()` instead

    ## Error in f(...) : 수학함수에 숫자가 아닌 인자가 전달되었습니다

    ## [1] NA

``` r
failwith(NA, log, quiet = TRUE) ("a")
```

    ## Warning: Deprecated: please use `purrr::possibly()` instead

    ## [1] NA

failwith()는 기능과 결합하여 함께 사용하면 매우 유용하다. 즉, 실패가 전파되어 상위 루프를 종료하는 대신 순환을
완료하고 잘못된 것을 찾을 수 있다. GLM의 집합을 리스트로 된 데이터 프레임에 적합한다고 생각해보라. GLM은
최적화 문제 때문에 가끔 실패할 수 있지만, 사용자는 모든 모형을 계속 적합하고 실패한 것이 어디인지 찾아보길 원할 것이다.

``` r
# 어떤 모형이든지 실패하면 모든 모형이 적합에 실패함
models <- lapply(datasets, glm, formula = y ~ x1 + x2 * x3)
# 모형 하나가 실패하면, 그 실패한 모형은 NULL값을 가짐
models <- lapply(datasets, failwith(NULL, glm), formula = y ~ x1 + x2 * x3)

# 간단히 하기 위해 (NULL 값을 갖는) 실패한 모형들을 제거
ok_models <- compact(models)
# 실패한 모형에 대응하는 데이터 세트를 추출
failed_data <- datasets[vapply(models, is.null, logical(1))]
```

## 상황 처리

  - `try()` : 오류가 발생하더라도 계속 실행할 수 있는 능력을 제공
  - `tryCatch()` : 상황이 발생할 때 일어나는 일들을 제어하는 핸글러(handler)함수를 지정 할 수 있게 해주
    준다.

### try를 이용한 오류 무시

오류를 발생시키는 함수를 실행하면 그 함수는 즉시 종료되고, 값을 반환하지 않는다. 그러나 오류를 생성하는 구문을 try()로
감싸면 오류 메시지가 출력되기는 하지만, 실행은 계속된다.

``` r
f <- function(x) {
  try(log(x))
  10
}
f("a")
```

    ## Error in log(x) : 수학함수에 숫자가 아닌 인자가 전달되었습니다

    ## [1] 10

``` r
# 더 큰 오류를 감싸려면 {}
try({
  a <- 1
  b <- "x"
  a + b
})
```

    ## Error in a + b : 이항연산자에 수치가 아닌 인수입니다

try()함수의 출력을 캡처할 수도 있다. 성공했다면 그 코드 블록에서 평가된 마지막 결과일 것이고, 성공하지 못했자면
*try-error*클래스의 (보이지 않는)객체일 것이다.

``` r
success <- try(1 + 2)
failure <- try("a" + "b")
```

    ## Error in "a" + "b" : 이항연산자에 수치가 아닌 인수입니다

``` r
class(success) ; class(failure)
```

    ## [1] "numeric"

    ## [1] "try-error"

try()는 특히 함수를 리스트의 여러 요소에 적용할 때 유용하다.

``` r
elements <- list(1:10, c(-1, 10), c(TRUE, FALSE), letters)
# results <- lapply(elements, log)
#  NaNs producedError in FUN(X[[i]], ...) : non-numeric argument to mathematical function
results <- lapply(elements, function(x) try(log(x)))
```

    ## Warning in log(x): NaN이 생성되었습니다

    ## Error in log(x) : 수학함수에 숫자가 아닌 인자가 전달되었습니다

``` r
results
```

    ## [[1]]
    ##  [1] 0.0000000 0.6931472 1.0986123 1.3862944 1.6094379 1.7917595 1.9459101
    ##  [8] 2.0794415 2.1972246 2.3025851
    ## 
    ## [[2]]
    ## [1]      NaN 2.302585
    ## 
    ## [[3]]
    ## [1]    0 -Inf
    ## 
    ## [[4]]
    ## [1] "Error in log(x) : 수학함수에 숫자가 아닌 인자가 전달되었습니다\n"
    ## attr(,"class")
    ## [1] "try-error"
    ## attr(,"condition")
    ## <simpleError in log(x): 수학함수에 숫자가 아닌 인자가 전달되었습니다>

try-error클래스를 테스트하자

``` r
is.error <- function(x) inherits(x, "try-error")
succeeded <- !vapply(results, is.error, logical(1))

#성공한 결과
str(results[succeeded])
```

    ## List of 3
    ##  $ : num [1:10] 0 0.693 1.099 1.386 1.609 ...
    ##  $ : num [1:2] NaN 2.3
    ##  $ : num [1:2] 0 -Inf

``` r
#실패한 입력
str(elements[!succeeded])
```

    ## List of 1
    ##  $ : chr [1:26] "a" "b" "c" "d" ...

여기서 `inherits(x, "classname")`은 어떤 객체가 특정 클래스를 상속했는지 확인 할 수 있다.

### tryCatch()를 이용한 상황 처리

error, warning, message, intrrupt, 그리고 condition은 내장된 이름중 유용하다.

``` r
show_condition <- function(code) {
  tryCatch(code,
           error = function(c) "error",
           warning = function(c) "warning",
           message = function(c) "message"
           )
}
show_condition(stop("!"))
```

    ## [1] "error"

``` r
show_condition(warning("?!"))
```

    ## [1] "warning"

``` r
show_condition(message("?"))
```

    ## [1] "message"

# Chapter17: Iteration with purrr

코드의 반복을 줄이는 것은 세가지 주요한 이점이 있다

  - 당신의 눈이 다른 것에 더 이끌리기 때문에 코드의 의도를 쉽게 볼 수 있다.
  - 요구사항의 변경에 쉽게 반응할 수 있다. 변화가 필요할 때, 코드를 복사하고 붙이는 모든 곳에서 변화를 기억하기 보다
    제자리에서 변화를 필요로 한다.
  - 각 코드의 라인이 많은 곳에서 사용되므로 버그를 줄일 수 있다.

복제를 줄이는 한 가지 도구는 함수이고 코드의 반복적인 패턴을 확인하고 쉽게 재사용되고 업데이트 될 수 있는 독립적인 조각으로
추출함으로써 복제를 줄인다. 다른 복제를 줄이는 도구는 ***iteration***이고 다수의 입력으로 같은 일을 하는
것(다른 열과 다른 데이터 셋에 같은 행동을 반복한다.)을 필요로 할 때 당신을 도와준다. 두 가지 중요한 반복 패러다임이
있다: imperative programming and functional programming. imperative에서
for루프와 while루프 같은 도구를 가지며 이것은 매우 명백하므로 시작하기 좋다. 하지만 for루프는 아주 장황하고 모든
for루프를 위해 복제된 코드는 아주 bookkeeping코드를 필요로 한다. 함수형 프로그래밍(FP)는 복제된 코드를 추출하는
도구를 제공하고, 각각의 공통의 for루프 패턴을 자신의 함수로 얻는다. FP의 어휘를 숙달하면 많은 코드를 줄이고 더 쉽고
에러가 작은 흔한 반복 문제를 해결할 수 있을 것이다.

## For Loops

``` r
df <- tibble(
  a = rnorm(10),
  b = rnorm(10),
  c = rnorm(10),
  d = rnorm(10)
)
```

`median(df$x)`로 각 열의 중앙값을 계산 할 수 있다. 하지만 우리의 경험상 부숴버리자. 대신에 for루프를 사용할 수
있다.

``` r
output <- vector("double", ncol(df)) # 1. output
for(i in seq_along(df)) {
  output[[i]] <- median(df[[i]])
}
output
```

    ## [1] -0.03205625  0.63237633 -0.58413226  0.06837661

모든 for루프는 세가지 구성요소를 가진다:

***output*** output \<- vector(“double”, length(x))

<span style="background : yellow">루프를 시작하기 전에, 반드시 항상 충분한 출력값을 위한 공간을
할당해야 한다.</span> 효율성을 위해 매우 중요하다: 만약 당신이 각 반복에 c()를 사용한다면 루프가 매우
느려진다.

주어진 길이의 빈 벡터를 만드는 일반적인 방법은 `vector()`함수이다. 두 가지 인자를 가진다. 벡터의
타입((“logical,” “integer,” “double,” “character,” “list”,
“expression”, etc.) 그리고 벡터의 길이다.

***sequence*** i in seq\_along(df)

루프의 끝을 결정한다. 각 for루프의 실행은 seq\_along(df)로부터 다른 값에 i를 할당한다. i를 대명사로 생각하면
유용하다. length(df)는 열의 수를 반환한다.

`seq_along()`을 이전에 보지 못했을 수도 있다. 1:length(l)의 안전한 버전인데 중요한 차이가 있다. 만약
0길이의 벡터를 가진다면 seq\_along은 다음과 같은 행동을 취한다.

``` r
y <- vector("double", 0)
seq_along(y)
```

    ## integer(0)

``` r
1:length(y)
```

    ## [1] 1 0

당신은 아마 0길이의 벡터를 의도적으로 만들지 않으려고 하겠지만 우연히 만들기는 쉽다. 에러를 발생시키지 않도록
`seq_along(x)`를 사용하자.

***body*** output\[\[i\]\] \<- median (df\[\[i\]\])

매번 다른 i값이 반복적으로 실행된다.

## Exercises

1.  Write for loops to:

<!-- end list -->

1.  Compute the mean of every column in mtcars.

<!-- end list -->

``` r
output <- vector("double", ncol(mtcars))
for (i in seq_along(mtcars)) {
  output[[i]] <- mean(mtcars[[i]])
}
names(output) <- colnames(mtcars)
output
```

    ##        mpg        cyl       disp         hp       drat         wt 
    ##  20.090625   6.187500 230.721875 146.687500   3.596563   3.217250 
    ##       qsec         vs         am       gear       carb 
    ##  17.848750   0.437500   0.406250   3.687500   2.812500

2.  Determine the type of each column in nyc flights13::flights.

<!-- end list -->

``` r
df_type <- vector("list", ncol(flights)) # posix때문에 list
for (i in seq_along(flights)) {
  df_type[[i]] <- class(flights[[i]])
}
names(df_type) <- colnames(flights)
df_type
```

    ## $year
    ## [1] "integer"
    ## 
    ## $month
    ## [1] "integer"
    ## 
    ## $day
    ## [1] "integer"
    ## 
    ## $dep_time
    ## [1] "integer"
    ## 
    ## $sched_dep_time
    ## [1] "integer"
    ## 
    ## $dep_delay
    ## [1] "numeric"
    ## 
    ## $arr_time
    ## [1] "integer"
    ## 
    ## $sched_arr_time
    ## [1] "integer"
    ## 
    ## $arr_delay
    ## [1] "numeric"
    ## 
    ## $carrier
    ## [1] "character"
    ## 
    ## $flight
    ## [1] "integer"
    ## 
    ## $tailnum
    ## [1] "character"
    ## 
    ## $origin
    ## [1] "character"
    ## 
    ## $dest
    ## [1] "character"
    ## 
    ## $air_time
    ## [1] "numeric"
    ## 
    ## $distance
    ## [1] "numeric"
    ## 
    ## $hour
    ## [1] "numeric"
    ## 
    ## $minute
    ## [1] "numeric"
    ## 
    ## $time_hour
    ## [1] "POSIXct" "POSIXt"

3.  Compute the number of unique values in each column of iris.

<!-- end list -->

``` r
iris_uniqueCnt <- vector("integer", ncol(iris))
for (i in seq_along(iris)) {
  iris_uniqueCnt[[i]] <- n_distinct(iris[[i]])
}
names(iris_uniqueCnt) <- colnames(iris)
iris_uniqueCnt
```

    ## Sepal.Length  Sepal.Width Petal.Length  Petal.Width      Species 
    ##           35           23           43           22            3

4.  Generate 10 random normals for each of μ = –10, 0, 10, and

<!-- end list -->

100. Think about the output, sequence, and body before you start writing
     the loop.

<!-- end list -->

``` r
mu <- c(-10, 0, 10, 100)
output <- matrix(0, ncol = length(mu), nrow = 10)
for(i in seq_along(mu)) {
  output[ ,i] <- rnorm(10, mu[i])
}
colnames(output) <- paste("mu = ", mu)
output
```

    ##        mu =  -10     mu =  0  mu =  10 mu =  100
    ##  [1,]  -9.644024 -1.33156161  9.335007 101.64276
    ##  [2,]  -9.170095 -1.04822574  8.199870  99.84234
    ##  [3,]  -9.312178  1.68765977 11.503126 100.07217
    ##  [4,] -10.865780  0.65222634 10.173874  99.60306
    ##  [5,]  -9.139811 -0.81888954  9.168529  98.12317
    ##  [6,] -10.630711 -0.71532275 10.611100  99.33551
    ##  [7,]  -8.353956  0.88235160 11.137439  99.33873
    ##  [8,] -10.891815 -0.06008791 11.056407  98.03213
    ##  [9,] -10.255189 -1.15569634 10.164637  98.33648
    ## [10,]  -9.623687 -1.32519504  9.599302 100.29568

``` r
# 이게 최고!
matrix(rnorm(n * length(mu), mean = mu), ncol = n)
```

2.  Eliminate the for loop in each of the following examples by taking
    advantage of an existing function that works with vectors:

<!-- end list -->

``` r
out <- ""
for (x in letters) {
  out <- stringr::str_c(out, x)
}

x <- sample(100)
sd <- 0
for (i in seq_along(x)) {
  sd <- sd + (x[i] - mean(x)) ^ 2
}
sd <- sqrt(sd / (length(x) - 1))

x <- runif(100)
out <- vector("numeric", length(x))
out[1] <- x[1]
for (i in 2:length(x)) {
  out[i] <- out[i - 1] + x[i]
}
```

``` r
out <- str_c(letters, collapse = "")

x <- sample(100)
sd2 <- sd(x)

x <- runif(100)
out <- cumsum(x)
```

3번은.. 시간이 남아돌 때 풀자.. 3. Combine your function writing and for loop
skills: a. Write a for loop that prints() the lyrics to the children’s
song “Alice the Camel.” b. Convert the nursery rhyme “Ten in the Bed” to
a function. Generalize it to any number of people in any sleeping
structure. c. Convert the song “99 Bottles of Beer on the Wall” to a
function. Generalize to any number of any vessel containing any liquid
on any surface.

## For Loop Variations

  - 새로운 객체를 생성하는 대신에 기존의 객체를 수정한다.
  - 인덱스 대신에 이름과 값으로 루프
  - 모르는 길이로 출력값 다루기
  - 모르는 길이로 수열 다루기

### Modifying an Existing Object

때때로 가끔 기존의 객체를 수정하는 루프를 사용하기를 원할 수 있다. 데이터프레임의 모든 열을 rescale하기를 원할 수 있다.

``` r
df <- tibble(
  a = rnorm(10),
  b = rnorm(10),
  c = rnorm(10),
  d = rnorm(10)
)
rescale01 <- function(x) {
  rng <- range(x, na.rm = TRUE)
  (x - rng[1]) / (rng[2] - rng[1])
}
```

이 문제를 for루프로 해결하기 위해 세가지 구성요소를 고려하자.

Output 이미 출력값을 가지고 있다.

Sequence 열의 리스트로서 데이터프레임을 고려해서 우리는 각 열을 seq\_along(df)로 반복

Body rescale01()적용

``` r
for (i in seq_along(df)) {
  df[[i]] <- rescale01(df[[i]])
}
```

전형적으로 리스트나 데이터프레임을 이런 종류의 루프로 수정하므로 \[가 아니라 \[\[ 사용을 기억하라. \[\[를 원자형 벡터에
쓰는것이 단 하나의 성분으로 작업한다는 것을 분명하게 해준다.

### Looping Patterns

벡터에 대해 루프를 돌리는 기본적인 세 가지 방법이 있다. 현재까지 가장 일반적인 걸 보여줬다. : looping over the
numeric indices with for (i in seq\_along(xs)), and extracting the value
with x\[\[i\]\]. There are two other forms

  - 성분에 대한 루프 : `for (x in xs)`. 플랏을 그리거나 파일 저장과 같은 부작용을 조심한다면 가장 유용하다.
  - 이름에 대한 루프 : `for (nm in names(xs))`. 이름을 주고 x\[\[nm\]\]로 값에 접근할 수
    있다. 플랏의 제목이나 파일 이름에 이름을 사용할 때 유용하다. 이름있는 출력값을 생성한다면, 벡터의 결과를
    이름짓는 것을 확실히 하라.

<!-- end list -->

``` r
results <- vector("list", length(x))
names(results) <- names(x)
```

수치 인덱스에 대한 반복은 가장 일반적은 형태다. 왜냐하면 이름과 값 둘 다 추출할 수 있기 때문이다.

``` r
for (i in seq_along(x)) {
  name <- names(x)[[i]]
  value <- x[[i]]
}
```

### Unknown Output Length

벡터의 길이가 랜덤일 때 벡터의 길이를 키우는 방법으로 문제를 풀고 싶을 것이다.

``` r
means <- c(0, 1, 2)
output <- double()
for (i in seq_along(means)) {
  n <- sample(100, 1)
  output <- c(output, rnorm(n, means[[i]]))
}
str(output)
```

    ##  num [1:194] -0.334 0.585 1.174 -1.357 -0.535 ...

더 나은 방법은 리스트에 결과를 저장해서 루프를 쓴 후에 단하나의 벡터로 조합하는 것이다.

``` r
out <- vector("list", length(means))
for (i in seq_along(means)) {
  n <- sample(100, 1)
  out[[i]] <- rnorm(n, means[[i]])
}
str(out)
```

    ## List of 3
    ##  $ : num [1:63] -0.0409 2.0596 -1.5297 -1.1819 -0.7457 ...
    ##  $ : num 1.87
    ##  $ : num [1:31] 0.304 2.787 1.849 2.871 0.384 ...

``` r
str(unlist(out))
```

    ##  num [1:95] -0.0409 2.0596 -1.5297 -1.1819 -0.7457 ...

  - paste()를 각 반복에 사용하는 대신에 <span style="background : yellow">문자형 벡터를
    저장하고 벡터를 하나의 문자열로 조합하라.</span> `paste(output, collapse = "")`
  - rbind로 각 반복마나 데이터 프레임을 합치지 말고 <span style="background : yellow">리스트로
    저장해서 `bind_rows(output)`을 사용해서 단 하나의 데이트프레임으로 조합하라.</span>

### Unknown Sequence Length

시뮬레이션을 할 때는 수열의 길이도 모를 수 있다.

한 행의 세 head를 얻는데 취하는 반복 횟수를 while루프를 사용

``` r
flip <- function() sample(c("T", "H"), 1)

flips <- 0
nheads <- 0

while (nheads < 3) {
  if (flip() == "H") {
    nheads <- nheads + 1
  } else {
    nheads <- 0
  }
  flips <- flips + 1
}
flips
```

    ## [1] 7

### cat과 print의 차이

`cat()`은 원자형 벡터와 이름만 유효하다. 빈 리스트나 다른 객체 타입을 호출할 수 없다.

`print()`는 제네릭 함수여서 특정 S3 class의 구체적 이행을 정의할 수 있다.

``` r
foo <- "foo"
print(foo)
```

    ## [1] "foo"

``` r
attributes(foo)$class <- "foo"
print(foo)
```

    ## [1] "foo"
    ## attr(,"class")
    ## [1] "foo"

``` r
attr(foo,"class")
```

    ## [1] "foo"

``` r
print.foo <- function(x) print("This is foo") # 메소드정의
print(foo)
```

    ## [1] "This is foo"

또 다른 차이는 반환값이다. `cat()`은 보이지 않게 NULL을 반환하지만 `print()`는 인자를 반환한다. 이런
`print()`의 특성은 pipe와 조합하면 특히 유용하다.

``` r
coefs <- lm(Sepal.Width ~  Petal.Length, iris) %>%
         print() %>%
         coefficients()
```

    ## 
    ## Call:
    ## lm(formula = Sepal.Width ~ Petal.Length, data = iris)
    ## 
    ## Coefficients:
    ##  (Intercept)  Petal.Length  
    ##       3.4549       -0.1058

`cat()`은 파일에 문자열을 쓸 때 유용하다

``` r
sink("foobar.txt")
cat('"foo"\n')
cat('"bar"')
sink()
```

print()나 sprintf()는 결과를 출력한 뒤 개행이 일어난다. 반면 cat()은 주어진 입력을 출력하고 행을 바꾸지 않는
특징이 있다. 또한 cat()에는 여러 인자를 나열해 쓰면 해당 인자들이 계속 연결되어 출력된다는 특징이 있다. 개행을 하려면
직접 “”을 써주면 된다.

## Exercises

1.  Imagine you have a directory full of CSV files that you want to read
    in. You have their paths in a vector, files \<- dir(“data/”, pattern
    = “\\.csv$”, full.names = TRUE), and now want to read each one with
    read\_csv(). Write the for loop that will load them into a single
    data frame.

<!-- end list -->

``` r
files <- dir(pattern = "\\.csv$", full.names = TRUE)
df <- vector("list", length(files))
for (i in seq_along(files)) {
  df[[i]] <- read.csv(files[[i]])
}
df <- bind_rows(df)
```

참고로 여기서 .앞에 \\가 붙은 이유는 정규표현식 때문.

2.  What happens if you use for (nm in names(x)) and x has no names?
    What if only some of the elements are named? What if the names are
    not unique?

<!-- end list -->

``` r
x <- 1:10
for (nm in names(x)) {
  print(nm)
}

names(x) <- letters[1:5]
for (nm in names(x)) {
  print(nm)
}
```

    ## [1] "a"
    ## [1] "b"
    ## [1] "c"
    ## [1] "d"
    ## [1] "e"
    ## [1] NA
    ## [1] NA
    ## [1] NA
    ## [1] NA
    ## [1] NA

``` r
nm
```

    ## [1] NA

``` r
names(x) <- rep("a", 10)
for (nm in names(x)) {
  print(nm)
}
```

    ## [1] "a"
    ## [1] "a"
    ## [1] "a"
    ## [1] "a"
    ## [1] "a"
    ## [1] "a"
    ## [1] "a"
    ## [1] "a"
    ## [1] "a"
    ## [1] "a"

``` r
x["a"] # 가장 앞만 매칭
```

    ## a 
    ## 1

3.  Write a function that prints the mean of each numeric column in a
    data frame, along with its name. For example,

format 함수

첫번째 인자: x, 수치 R 객체 세번째 인자: digits, 유의한 자릿수 네번째 인자: nsmall, 소수점의 최소 자릿수

``` r
format(pi*100, digits = 2, nsmall = 6)
```

    ## [1] "314.159265"

한국어는 str\_pad에서 2배만큼 띄어야 한다.

``` r
show_mean <- function(df, digits = 2) {
  # Get mas Length of any variable in the dataset
  maxstr <- max(str_length(names(df)))
  for (nm in names(df)) {
    if (is_numeric(df[[nm]])) {
      cat(str_c(str_pad(str_c(nm, ":"), maxstr + 1L, side = "right"),
                format(mean(df[[nm]]), digits = digits, nsmall = digits),
                sep = " "),
         "\n")
    }
  }
}
show_mean(iris)
```

    ## Warning: Deprecated

    ## Sepal.Length: 5.84

    ## Warning: Deprecated

    ## Sepal.Width:  3.06

    ## Warning: Deprecated

    ## Petal.Length: 3.76

    ## Warning: Deprecated

    ## Petal.Width:  1.20

    ## Warning: Deprecated

    ## Warning in mean.default(df[[nm]]): argument is not numeric or logical:
    ## returning NA

    ## Species:      NA

would print: (Extra challenge: what function did I use to make sure that
the numbers lined up nicely, even though the variable names had
different lengths?)

*기억해두자.* 4. What does this code do? How does it work?

``` r
trans <- list(
  disp = function(x) x * 0.0163871,
  am = function(x) {
    factor(x, labels = c("auto", "manual"))
  }
)

for (var in names(trans)) {
  mtcars[[var]] <- trans[[var]](mtcars[[var]])
}
```

함수 이름을 리스트의 원소로 넣어서 for문을 돌려 기존 데이터프레임을 변형하였다.

`invoke_map()`으로 구현

``` r
trans_df <- data_frame(trans, var = names(trans))
```

``` r
formulas <- list(
  mpg ~ disp,
  mpg ~ I(1 / disp),
  mpg ~ disp + wt,
  mpg ~ I(1 / disp) + wt
)
mtcars %>% 
  map(formulas,lm, data = .) %>% 
  map(summary) %>% 
  map("r.squared")
```

## For Loops Versus Functionals

for루프는 다른 언어에서 만큼 R에서는 중요하지 않다. 왜냐하면 R은 함수형프로그래밍 언어이기 때문이다. 이건 for루프를
함수로 둘러쌀 수 있고 함수를 for루프 대신에 호출할 수 있다.

``` r
df <- tibble(
  a = rnorm(10),
  b = rnorm(10),
  c = rnorm(10),
  d = rnorm(10)
)
```

모든 열의 평균을 계산할 때 for루프를 쓰면

``` r
output <- vector("double", length(df))
for (i in seq_along(df)) {
  output[[i]] <- mean(df[[i]])
}
```

아주 빈전히 모든 열의 평균을 계산하기를 원해서 함수로 추출한다.

``` r
col_mean <- function(df) {
  output <- vector("double", length(df))
  for (i in seq_along(df)) {
    output[i] <- mean(df[[i]])
  }
  output
}
```

중앙값과 표준편차도 계산 하면 도움이 될 것 같다.

``` r
col_median <- function(df) {
  output <- vector("double", length(df))
  for (i in seq_along(df)) {
    output[i] <- median(df[[i]])
  }
  output
}
col_sd <- function(df) {
  output <- vector("double", length(df))
  for (i in seq_along(df)) {
    output[i] <- sd(df[[i]])
  }
  output
}
```

2번이나 코드를 복붙했으므로 일반화 할 방법을 생각해보자.

만약 함수의 집합이 이렇다면 무엇을 할까?

``` r
f1 <- function(x) abs(x - mean(x)) ^ 1
f2 <- function(x) abs(x - mean(x)) ^ 2
f3 <- function(x) abs(x - mean(x)) ^ 3
```

``` r
f <- function(x, i) abs(x - mean(x)) ^ i
```

우리는 이것을 col\_mean(), col\_median(), and col\_sd()에도 각 열에 적용을 위해 함수를 제공하는
인자를 추가함으로써 같은 일을 할 수 있다.

``` r
col_summary <- function(df, fun) {
  out <- vector("double", length(df))
  for (i in seq_along(df)) {
    out[i] <- fun(df[[i]])
  }
  out
}
col_summary(df, median)
```

    ## [1]  0.610849425 -0.246453111 -0.333448369  0.003837985

``` r
col_summary(df, mean)
```

    ## [1]  0.63410350  0.03540099 -0.04638160 -0.12671054

다른 함수에 함수를 전달하는 것은 굉장히 강략한 아이디어이고, R을 함수형 프로그래밍 언어로 만드는 행동 중 하나이다.

purrr 함수를 for루프 대신에 사용하는 목적은 공통의 리스트 조작 문제를 독립적 조각으로 나누게 하기 위함이다.

  - 리스트의 단 하나의 요소에 대한 문제를 어떻게 해결할 것인가? 일단 문제를 해결하면 purrr은 리스트의 모든 성분에
    당신의 해답을 일반화한다.
  - 복잡한 문제를 푼다면, 해답을 향한 약간의 단계로 나아가기 위해 어떻게 조각을 나눌것인가? purrr과 함께 pipe를
    가지고 함께 구성할 수 있는 작은 조각을 얻는다.

## The Map Functions

벡터에 대한 루프의 패턴은 각 성분에 행동을 취하고 결과를 저장하는 것은 흔해서 purrr패키지는 이것을 할 함수족을 제공한다.

  - `map()`: 리스트를 만든다.
  - `map_lgl()`: 논리형 벡터를 만든다.
  - `map_int()`: 정수형 벡터를 만든다.
  - `map_dbl()`: double형 벡터를 만든다.
  - `map_chr()`: 문자형 벡터를 만든다.

각 함수는 벡터를 입력값으로 취하고, 각 조각에 함수를 적용하며 입력값과 같은 길이의 새로운 벡터를 반환한다. 벡터의 타입은
접미사로 결정된다.

이러한 함수들을 숙달했을 때, 반복 문제를 해결한 좀 더 짧은 시간을 소요하게 될 것이다. 하지만 map함수 대신에 for루프를
사용하는 것에 대해 기분 나쁘게 생각하지 마라. map함수는 추상화의 타워를 강화하고 어떻게 작동하는지 머리 주위에서 긴
시간이 걸릴 수 있다. 중요한 것은 당신이 작업하는 문제를 해결하는 것이지 가장 간결하고 우아한 코드를 작성하는 것이
아니다.

for루프는 느리다고 피하라고 하지만 잘못된 것이다. map()함수 사용의 주된 이점은 속도가 아니라 명료함이다. 코드를 읽고
쓰기 쉽게 만든다.

마지막 for루프와 같은 계산을 수행하는 함수를 사용할 수 있다. double을 반환하므로 `map_dbl()`을 사용한다.

``` r
map_dbl(df, mean)
```

    ##           a           b           c           d 
    ##  0.63410350  0.03540099 -0.04638160 -0.12671054

``` r
map_dbl(df, median)
```

    ##            a            b            c            d 
    ##  0.610849425 -0.246453111 -0.333448369  0.003837985

``` r
map_dbl(df, sd)
```

    ##        a        b        c        d 
    ## 1.054878 1.061262 1.097137 0.724126

좀 더 분명하게 하기 위해

``` r
df %>% map_dbl(mean)
df %>% map_dbl(median)
df %>% map_dbl(sd)
```

map\_\*()와 col\_summary()사이의 몇 가지 차이가 있다.

  - 모든 purrr 함수는 C에서 시행된다. 가독성을 희생하면서 조금 더 빠르다.
  - 두번째 인자인 .f는 formula가 될 수 있다.
  - map\_\*()는 가변인자를 호출 될 때마다 .f에 추가적 인자로 전달한다.

<!-- end list -->

``` r
map_dbl(df, mean, trim = 0.5)
```

    ##            a            b            c            d 
    ##  0.610849425 -0.246453111 -0.333448369  0.003837985

  - map함수는 또한 이름을 보존한다.

<!-- end list -->

``` r
z <- list(x = 1:3, y = 4:5)
map_int(z, length)
```

    ## x y 
    ## 3 2

### Shortcuts

.f와 함께 타이핑을 줄일 수 있는 몇 가지 숏컷이 있다. 데이터 셋의 각 그룹에 선형 모형을 적합시킨다고 상상해보자.

``` r
models <- mtcars %>% 
  split(.$cyl) %>%  # df를 열로 나눈다
  map(function(df) lm(mpg ~ wt, data = df))
```

R에서 익명함수를 만드는 문법은 장황해서 purrr은 편리한 숏컷을 제공한다.

``` r
models <- mtcars %>% 
  split(.$cyl) %>% 
  map(~lm(mpg ~ wt, data = .))
```

여기서 대명사로서 .을 사용했다. 이것은 현재 리스트 성분을 참조한다.(마찬가지로 for루프의 현재 인덱스를 i가 참조한다.)

\(R^2\)를 추출하기를 원할 수 있다.

``` r
models %>% 
  map(summary) %>% 
  map_dbl(~.$r.squared) # ~사용을 잊지 말자.
```

하지만 이름있는 성분을 추출하는 것은 흔한 연산이므로 purrr은 심지어 더 짧은 숏컷을 제공한다. : 문자열을 사용할 수 있다.

``` r
models %>% 
  map(summary) %>% 
  map_dbl("r.squared")
```

또한 위치에 성분을 선택하기 위해 정수를 사용할 수 있다.

``` r
x <- list(list(1, 2, 3), list(4, 5, 6), list(7, 8, 9))
x %>% map_dbl(2)
```

    ## [1] 2 5 8

map\_\*의 첫번째 인자가 꼭 데이터가 될 필요는 없다. “각 함수가 입력값으로서 벡터를 취하고 각 조각에 함수를 적용해서
입력값과 같은 갗은 길이의 새로운 벡터를 반환한다”.

~는 .f인자에 함수의 이름을 제외한 무언가를 적어야 하는 함수일 때 써준다고 생각하면 편하다. 성분 추출은
map(“성분이름”)을 사용한다.

### Base R

  - `lapply()`는 기본적으로 `map()`과 비슷한데 `map()`이 purrr의 다른 함수와 일관된다는 것과 .f를
    위한 숏컷을 제외하면 말이다.
  - `sapply()`는 lapply()의 출력을 자동적으로 간단화하는 래퍼이다. 상호작용하는 작업이지만 출력의 종류를 절대
    알 수 없다.

<!-- end list -->

``` r
x1 <- list(
c(0.27, 0.37, 0.57, 0.91, 0.20),
c(0.90, 0.94, 0.66, 0.63, 0.06),
c(0.21, 0.18, 0.69, 0.38, 0.77)
)
x2 <- list(
c(0.50, 0.72, 0.99, 0.38, 0.78),
c(0.93, 0.21, 0.65, 0.13, 0.27),
c(0.39, 0.01, 0.38, 0.87, 0.34)
)
threshold <- function(x, cutoff = 0.8) x[x > cutoff]
x1 %>% sapply(threshold) %>% str()
```

    ## List of 3
    ##  $ : num 0.91
    ##  $ : num [1:2] 0.9 0.94
    ##  $ : num(0)

``` r
x2 %>% sapply(threshold) %>% str()
```

    ##  num [1:3] 0.99 0.93 0.87

  - `vapply()` `sapply()`의 안전한 대안이다. 왜냐하면 타입을 정의하는 추가적 인자를 제공한다.
    `vapply()`의 유일한 문제는 타이핑이 길다. vapply(df, is.numeric, logical(1)) is
    equivalent to map\_lgl(df, is.numeric). `vapply()`의 purrr의 map 함수에
    대해 한가지 이점은 행렬을 또한 생성 할 수 있다는 것이다. (map 함수는 오직 벡터만 생성한다.)

## Exercises

1.  Write code that uses one of the map functions to:

<!-- end list -->

1.  Compute the mean of every column in
    mtcars.

<!-- end list -->

``` r
mtcars %>% map_dbl(mean)
```

    ## Warning in mean.default(.x[[i]], ...): argument is not numeric or logical:
    ## returning NA

    ##        mpg        cyl       disp         hp       drat         wt 
    ##  20.090625   6.187500   3.780862 146.687500   3.596563   3.217250 
    ##       qsec         vs         am       gear       carb 
    ##  17.848750   0.437500         NA   3.687500   2.812500

2.  Determine the type of each column in nyc flights13::flights

<!-- end list -->

``` r
flights %>% map(class)
```

    ## $year
    ## [1] "integer"
    ## 
    ## $month
    ## [1] "integer"
    ## 
    ## $day
    ## [1] "integer"
    ## 
    ## $dep_time
    ## [1] "integer"
    ## 
    ## $sched_dep_time
    ## [1] "integer"
    ## 
    ## $dep_delay
    ## [1] "numeric"
    ## 
    ## $arr_time
    ## [1] "integer"
    ## 
    ## $sched_arr_time
    ## [1] "integer"
    ## 
    ## $arr_delay
    ## [1] "numeric"
    ## 
    ## $carrier
    ## [1] "character"
    ## 
    ## $flight
    ## [1] "integer"
    ## 
    ## $tailnum
    ## [1] "character"
    ## 
    ## $origin
    ## [1] "character"
    ## 
    ## $dest
    ## [1] "character"
    ## 
    ## $air_time
    ## [1] "numeric"
    ## 
    ## $distance
    ## [1] "numeric"
    ## 
    ## $hour
    ## [1] "numeric"
    ## 
    ## $minute
    ## [1] "numeric"
    ## 
    ## $time_hour
    ## [1] "POSIXct" "POSIXt"

3.  Compute the number of unique values in each column of iris.

<!-- end list -->

``` r
iris %>% map_int(n_distinct)
```

    ## Sepal.Length  Sepal.Width Petal.Length  Petal.Width      Species 
    ##           35           23           43           22            3

``` r
# iris %>% map_int(~length(unique(.)))
```

4.  Generate 10 random normals for each of μ = –10, 0, 10, and

<!-- end list -->

100. 
여기서는 map만 사용가능

``` r
map(c(-10, 0, 10, 100), rnorm, n = 10)
```

    ## [[1]]
    ##  [1]  -9.992633 -10.412290 -10.829424 -11.117653  -9.290339 -10.999837
    ##  [7] -10.486853 -10.379637  -8.134538  -9.385523
    ## 
    ## [[2]]
    ##  [1]  0.55648749 -0.39442887  0.09856163  0.45801629  1.09489026
    ##  [6]  1.27210771 -2.17757989 -1.46126343 -1.17805570  1.21809564
    ## 
    ## [[3]]
    ##  [1] 11.738337 10.903606  8.409627 10.652368  9.577640 10.096269  8.393419
    ##  [8]  9.436382  8.233485 10.508041
    ## 
    ## [[4]]
    ##  [1] 100.12367  99.88357  98.86302  99.30595 100.09458 100.36345  99.99120
    ##  [8]  98.21275 101.55303 100.42807

2.  How can you create a single vector that for each column in a data
    frame indicates whether or not it’s a
    factor?

<!-- end list -->

``` r
diamonds %>% map_lgl(is.factor)
```

    ##   carat     cut   color clarity   depth   table   price       x       y 
    ##   FALSE    TRUE    TRUE    TRUE   FALSE   FALSE   FALSE   FALSE   FALSE 
    ##       z 
    ##   FALSE

3.  What happens when you use the map functions on vectors that aren’t
    lists? What does map(1:5, runif) do? Why?

<!-- end list -->

``` r
map(1:5, runif)
```

    ## [[1]]
    ## [1] 0.7141421
    ## 
    ## [[2]]
    ## [1] 0.06124215 0.25583886
    ## 
    ## [[3]]
    ## [1] 0.43540756 0.09376931 0.41797629
    ## 
    ## [[4]]
    ## [1] 0.05885303 0.64381831 0.90294089 0.18016962
    ## 
    ## [[5]]
    ## [1] 0.60305331 0.94659313 0.50641054 0.05748896 0.61676581

4.  What does map(-2:2, rnorm, n = 5) do? Why? What does map\_dbl(-2:2,
    rnorm, n = 5) do? Why?

<!-- end list -->

``` r
map(-2:2, rnorm, n =5)
```

    ## [[1]]
    ## [1] -1.376804 -1.327119 -2.806854 -1.413772 -2.865952
    ## 
    ## [[2]]
    ## [1] -1.085787 -2.044906 -1.605284 -1.156763 -1.083867
    ## 
    ## [[3]]
    ## [1]  0.5279501  0.2288033  0.1016783 -0.5238340  1.2059315
    ## 
    ## [[4]]
    ## [1] 2.29284463 0.27634175 0.90950726 1.38070392 0.07280937
    ## 
    ## [[5]]
    ## [1] 3.019624 2.780235 2.325414 1.848048 1.937846

``` r
map_dbl(-2:2, rnorm, n =5)
```

    ## Result 1 must be a single double, not a double vector of length 5

`map_dbl()`은 길이 1의 double벡터를 반환한다.

5.  Rewrite map(x, function(df) lm(mpg ~ wt, data = df)) to eliminate
    the anonymous function.

예제에서 split함수를 사용하면 데이터프레임이 리스트로 변환된다.

``` r
mtcars %>%
  split(.$cyl) %>% 
  map(4) # 리스트의 각 성분의 4번째 열 추출
```

    ## $`4`
    ##  [1]  93  62  95  66  52  65  97  66  91 113 109
    ## 
    ## $`6`
    ## [1] 110 110 110 105 123 123 175
    ## 
    ## $`8`
    ##  [1] 175 245 180 180 180 205 215 230 150 150 245 175 264 335

*기억\!* .x의 데이터 타입은 원자형 벡터이거나 list임을 기억하자.

``` r
list(mtcars) %>% map(~lm(mpg~wt, data = .))
```

    ## [[1]]
    ## 
    ## Call:
    ## lm(formula = mpg ~ wt, data = .)
    ## 
    ## Coefficients:
    ## (Intercept)           wt  
    ##      37.285       -5.344

## Dealing with Failure

연산을 반복하기 위해 map함수를 사용할 때, 연산중 하나가 실패할 확률은 커진다. 이것이 일어나면, 에러를 발생시키고 어떠한
출력값도 나타나지 않는다. 이것은 짜증난다. 왜 하나의 실패가 모든 다른 성공을 막는가?

`safely()`는 부사다 : 함수를 취해서 수정된 버전을 반환한다. 이 경우에 수정된 함수는 에러를 발생시키지 않는다.
대신에, 항상 두 가지 성분을 가지는 리스트를 반환한다.

results 원본 결과. 에러가 있다면 NULL일 것이다.

error 에러 객체. 연산이 성공적이면 이것은 NULL일 것이다.

만약 base R의 `try()`에 익숙할 수도 있다. 이것은 비슷하지만 원래의 객체를 반환하고 에러 객체를 반환하기 때문에
작업하기 좀 더 어렵다.

``` r
safe_log <- safely(log)
str(safe_log(10))
```

    ## List of 2
    ##  $ result: num 2.3
    ##  $ error : NULL

``` r
str(safe_log("a"))
```

    ## List of 2
    ##  $ result: NULL
    ##  $ error :List of 2
    ##   ..$ message: chr "수학함수에 숫자가 아닌 인자가 전달되었습니다"
    ##   ..$ call   : language .Primitive("log")(x, base)
    ##   ..- attr(*, "class")= chr [1:3] "simpleError" "error" "condition"

함수가 result 성분을 성공할 때 result를 포함하고 error 성분은 NULL이다. 함수가 실패하면 반대로다.

`safely()`은 map과 함께 작업하기 위해 설계되어 있다.

``` r
x <- list(1, 10, "a")
y <- x %>% map(safely(log))
str(y)
```

    ## List of 3
    ##  $ :List of 2
    ##   ..$ result: num 0
    ##   ..$ error : NULL
    ##  $ :List of 2
    ##   ..$ result: num 2.3
    ##   ..$ error : NULL
    ##  $ :List of 2
    ##   ..$ result: NULL
    ##   ..$ error :List of 2
    ##   .. ..$ message: chr "수학함수에 숫자가 아닌 인자가 전달되었습니다"
    ##   .. ..$ call   : language .Primitive("log")(x, base)
    ##   .. ..- attr(*, "class")= chr [1:3] "simpleError" "error" "condition"

만약 두가지 리스트를 가진다면 함께 작업하기 쉬울 것이다.모든 에러중 하나와 모든 값들 중 하나.

``` r
y <- y %>% transpose()
str(y)
```

    ## List of 2
    ##  $ result:List of 3
    ##   ..$ : num 0
    ##   ..$ : num 2.3
    ##   ..$ : NULL
    ##  $ error :List of 3
    ##   ..$ : NULL
    ##   ..$ : NULL
    ##   ..$ :List of 2
    ##   .. ..$ message: chr "수학함수에 숫자가 아닌 인자가 전달되었습니다"
    ##   .. ..$ call   : language .Primitive("log")(x, base)
    ##   .. ..- attr(*, "class")= chr [1:3] "simpleError" "error" "condition"

에러를 어떻게 처리하는 가는 당신에게 달렸지만 일반적으로 y가 에러인 x의 값을 보거나 y가 올바른 값으로 작업한다.

``` r
is_ok <- y$error %>% map_lgl(is_null)
x[!is_ok]
```

    ## [[1]]
    ## [1] "a"

``` r
y$result[is_ok] %>% flatten_dbl() #  unlist()와 비슷한 함수
```

    ## [1] 0.000000 2.302585

purrr은 다른 유용한 부사를 두 가지 제공한다.

  - `safely()`처럼 `possibly()`는 항상 성공한다. `safely()`보다 간단한데 에러가 있을 때 디폴트
    값을 반환하기 때문이다.

<!-- end list -->

``` r
x <- list(1, 10, "a")
x %>% map_dbl(possibly(log, NA_real_))
```

    ## [1] 0.000000 2.302585       NA

  - `quietly()`는 `safely()`와 비슷한 역할을 수행하지만 에러를 찾는 대신에, 결과, 메시지, 그리고 경고를
    출력한다.

<!-- end list -->

``` r
x <- list(1, -1)
x %>% map(quietly(log)) %>% str()
```

    ## List of 2
    ##  $ :List of 4
    ##   ..$ result  : num 0
    ##   ..$ output  : chr ""
    ##   ..$ warnings: chr(0) 
    ##   ..$ messages: chr(0) 
    ##  $ :List of 4
    ##   ..$ result  : num NaN
    ##   ..$ output  : chr ""
    ##   ..$ warnings: chr "NaN이 생성되었습니다"
    ##   ..$ messages: chr(0)

### transpose()와 rerun()

`rerun()`은 표본 데이터 생성에 편리하다. `transpose()`는 list-of-lists “inside-out”

``` r
10 %>%
  rerun(x = rnorm(5), y = rnorm(5)) %>%
  map_dbl(~ cor(.$x, .$y))
```

    ##  [1] -0.76928366  0.51951648  0.02128613  0.53185637 -0.21829024
    ##  [6] -0.72717227  0.50398541 -0.25577812 -0.67455260 -0.70252000

``` r
x <- rerun(5, x = runif(1), y = runif(5))
x %>% str()
```

    ## List of 5
    ##  $ :List of 2
    ##   ..$ x: num 0.36
    ##   ..$ y: num [1:5] 0.2223 0.553 0.4737 0.0205 0.517
    ##  $ :List of 2
    ##   ..$ x: num 0.647
    ##   ..$ y: num [1:5] 0.9965 0.0214 0.9223 0.5651 0.171
    ##  $ :List of 2
    ##   ..$ x: num 0.115
    ##   ..$ y: num [1:5] 0.0741 0.7577 0.692 0.5091 0.2098
    ##  $ :List of 2
    ##   ..$ x: num 0.846
    ##   ..$ y: num [1:5] 0.175 0.288 0.961 0.254 0.969
    ##  $ :List of 2
    ##   ..$ x: num 0.253
    ##   ..$ y: num [1:5] 0.149 0.955 0.878 0.135 0.296

``` r
x %>% transpose() %>% str()
```

    ## List of 2
    ##  $ x:List of 5
    ##   ..$ : num 0.36
    ##   ..$ : num 0.647
    ##   ..$ : num 0.115
    ##   ..$ : num 0.846
    ##   ..$ : num 0.253
    ##  $ y:List of 5
    ##   ..$ : num [1:5] 0.2223 0.553 0.4737 0.0205 0.517
    ##   ..$ : num [1:5] 0.9965 0.0214 0.9223 0.5651 0.171
    ##   ..$ : num [1:5] 0.0741 0.7577 0.692 0.5091 0.2098
    ##   ..$ : num [1:5] 0.175 0.288 0.961 0.254 0.969
    ##   ..$ : num [1:5] 0.149 0.955 0.878 0.135 0.296

## Mapping over Multiple Arguments

현재까지 단 하나의 입력값에 따라 매핑했다. 하지만 동시에 반복할 필요가 있는 다수의 관련된 입력값을 가질 수 있다. 이것이
`map2()`와 `pmap()`함수의 일이다. 예를 들어, 다른 평균을 가진 임의의 정규분포를 실험하기를 원한다고 하자.

``` r
mu <- list(5, 10, -3)
mu %>%
  map(rnorm, n = 5) %>%
  str()
```

    ## List of 3
    ##  $ : num [1:5] 5.6 6.08 6.1 5.56 6.7
    ##  $ : num [1:5] 10.57 10.12 10.44 9.01 9.38
    ##  $ : num [1:5] -3.73 -1.04 -3.6 -3.61 -2.5

표준편차를 다양화하기를 원하면 어떨까? 한 가지 방법은 인덱스에 대해서 반복하고 인덱스를 평균과 표준편차의 벡터로 반복

``` r
sigma <- list(1, 5, 10)
seq_along(mu) %>% 
  map(~rnorm(5, mu[[.]], sigma[[.]])) %>% 
  str()
```

    ## List of 3
    ##  $ : num [1:5] 4.07 5.09 4.24 5 3.87
    ##  $ : num [1:5] 4.14 10.65 7.18 12.44 14.9
    ##  $ : num [1:5] 7.52 -6.69 -2.69 -8.9 -2.33

하지만 코드가 혼란스럽다. 대신에 `map2()`를 사용하고 동시에 두 벡터를 반복한다.

``` r
map2(mu, sigma, rnorm, n = 5) %>% str()
```

    ## List of 3
    ##  $ : num [1:5] 4.64 4.5 4.66 3.91 3.66
    ##  $ : num [1:5] 21.28 8.53 9.35 13.07 12.62
    ##  $ : num [1:5] -2.52 -11.95 -6.84 1.33 -16.42

각 호출에 다양한 인자가 함수 이전에 오는 것을 주의하라. 모든 호출에 같은 인자는 그 후에 온다.

`map()`처럼 `map2()`는 단지 for루프의 래퍼이다.

``` r
map2 <- function(x, y, f, ...) {
  out <- vector("list", length(x))
  for (i in seq_along(x)) {
    out[[i]] <- f(x[[i]], y[[i]], ...)
  }
  out
}
```

map3(), map4(), … 과 같은 함수보다는 purrr은 인자의 리스트를 취하는 pmap()을 제공한다.

``` r
n <- list(1, 3, 5)
args1 <- list(n, mu, sigma)
args1 %>% 
  pmap(rnorm) %>% 
  str()
```

    ## List of 3
    ##  $ : num 4.16
    ##  $ : num [1:3] 3.54 14.08 8.34
    ##  $ : num [1:5] -5.02 -0.26 -4.31 18.07 10.95

list의 성분을 이름 짓지 않는다면, `pmap()`은 positional matching을 함수 호출 시 사용할 것이다. 이건
읽기 어려우므로 인자를 이름 짓는 것이 더 좋다.

``` r
args2 <- list(mean = mu, sd = sigma, n = n)
args2 %>% 
  pmap(rnorm) %>% 
  str()
```

    ## List of 3
    ##  $ : num 4.61
    ##  $ : num [1:3] 9.47 4.66 8.87
    ##  $ : num [1:5] -3.108 -0.533 9.842 11.485 -2.436

더 길지만 안전한 호출이다.

모든 같은 길이의 인자이기 때문에, 데이터 프레임에 저장하는 것이 이해하기 쉽다.

``` r
parms <- tribble(
  ~mean, ~se, ~n,
  5, 1, 1,
  10, 5, 3,
  -3, 10, 5
)
params <- tibble(mean = unlist(mu), sd = unlist(sigma), n = unlist(n)) # 조금 전에 리스트로 지정해서
```

``` r
params %>% 
  pmap(rnorm)
```

    ## [[1]]
    ## [1] 6.042481
    ## 
    ## [[2]]
    ## [1]  9.9750379 12.8059692  0.1298868
    ## 
    ## [[3]]
    ## [1]   5.840664 -11.926128  -1.345248   7.936713   2.031719

코드가 복잡해지자마자 데이터프레임이 좋은 접근법이다. 각 열이 이름을 가지고 모든 다른 열과 같은 길이임을 보장하기 때문이다.

## Invoking Different Functions

복잡함에 한단계 더 나아가자. 함수의 인자를 다양화 할 뿐만 아니라 함수 그 자체를 다양화 해보자.

``` r
f <- c("runif", "rnorm", "rpois")

param <- list(
  list(min = -1, max = 1),
  list(sd = 5),
  list(lambda = 10)
)
```

이런 경우에 `invoke_map()`를 사용할 수 있다.

``` r
invoke_map(f, param, n = 5) %>% str()
```

    ## List of 3
    ##  $ : num [1:5] 0.645 -0.74 0.9 0.97 0.181
    ##  $ : num [1:5] 6.801 3.923 -3.509 3.75 0.883
    ##  $ : int [1:5] 9 9 11 12 14

첫번째 인자는 함수의 리스트이거나 함수의 문자형 벡터이다. 두번째 인자는 각 함수에 달라지는 인자의 리스트의 리스트이다. 이어지는
인자는 모든 함수에 전달된다.

``` r
sim <- tribble(
  ~f, ~params,
  "runif", list(min = -1, max = 1),
  "rnorm", list(sd = 5),
  "rpois", list(lambda = 10)
)
sim <- tibble(f, params = param)
sim %>% 
  mutate(sim = invoke_map(f, params, n = 10))
```

    ## # A tibble: 3 x 3
    ##   f     params           sim       
    ##   <chr> <list>           <list>    
    ## 1 runif <named list [2]> <dbl [10]>
    ## 2 rnorm <named list [1]> <dbl [10]>
    ## 3 rpois <named list [1]> <int [10]>

## Walk

Walk는 반환값 대신에 side-effect을 위한 함수를 호출하기를 원할 때 사용하는 map의 대안이다. 당신은 일반적으로
이것을 하는데 왜냐하면 스크린에 output을 render하고 싶어하거나 디스크에 파일을 저장하고 싶어한다. 중요한 것은
행동이지 반환 값이 아니다. 주의 : r 변수에 저장하는 것은 walk사용 X

287쪽에서 pipe가 가능한 주요한 두 가지 함수가 transformation함수와 side-effect함수라고 언급하였다.
여기서 side-effect함수는 주로 플랏을 그리고 파일을 저장하는 것과 같이 주로 action을 수행하기 위해
호출된다. 그래서 디폴트로는 출력되지 않지만 pipeline에 이용될 수 있다.

``` r
x <- list(1, "a", 3)
x %>% 
  walk(print)
```

    ## [1] 1
    ## [1] "a"
    ## [1] 3

`walk()`는 일반적으로 `walk2()`와 `pwalk()`와 비교해서 유용하지는 않다. 예를 들면, 플랏의 리스트와
파일이름의 벡터를 가진다면 `pwalk()`를 디스크에 위치에 상응하는 각 파일을 저장하기 위해 사용할 수 있다.

``` r
plots <- mtcars %>% 
  split(.$cyl) %>% 
  map(~ggplot(., aes(mpg, wt)) + geom_point())
paths <- str_c(names(plots), ".pdf")

pwalk(list(paths, plots), ggsave, path = tempdir())
```

`walk()`, `walk2()`, and `pwalk()` 모두 보이지 않게 .x를 반환한다. 이것은 pipeline의 중간에
사용하기 적합하도록 한다.

`pmap()`, `invoke_map()`, `pwalk()`모두 인자의 순서에 주의한다.

## Other Patterns of For Loops

purrr은 다른 유형의 for루프에 대해 추출하는 다른 함수의 수를 제공한다. 당신은 map함수보다 덜 사용할 것이지만 알면
유용하다. 목적은 간결하게 각 함수를 설명해서 희망적으로 미래에 비슷한 문제를 봤을 때 생각이 떠오를 것이다.
documentation에서 세부사항을 보아라.

### Predicate Functions

많은 함수는 TRUE 또는 FALSE를 반환하는 ***predicate***함수와 함께 작업한다.

`keep()`과 `discard()`는 predicate가 TRUE 또는 FALSE인 입력값의 요소를 각각 유지한다.

``` r
iris %>% 
  keep(is.factor) %>% 
  str()
```

    ## 'data.frame':    150 obs. of  1 variable:
    ##  $ Species: Factor w/ 3 levels "setosa","versicolor",..: 1 1 1 1 1 1 1 1 1 1 ...

``` r
iris %>% 
  discard(is.factor) %>% 
  str()
```

    ## 'data.frame':    150 obs. of  4 variables:
    ##  $ Sepal.Length: num  5.1 4.9 4.7 4.6 5 5.4 4.6 5 4.4 4.9 ...
    ##  $ Sepal.Width : num  3.5 3 3.2 3.1 3.6 3.9 3.4 3.4 2.9 3.1 ...
    ##  $ Petal.Length: num  1.4 1.4 1.3 1.5 1.4 1.7 1.4 1.5 1.4 1.5 ...
    ##  $ Petal.Width : num  0.2 0.2 0.2 0.2 0.2 0.4 0.3 0.2 0.2 0.1 ...

map과 \[\]를 섞은 느낌인듯.

``` r
iris[map_lgl(iris, is.factor)] %>% str()
```

    ## 'data.frame':    150 obs. of  1 variable:
    ##  $ Species: Factor w/ 3 levels "setosa","versicolor",..: 1 1 1 1 1 1 1 1 1 1 ...

<font color = "red">아래 코드는 기억해두자.</font>

``` r
# 파이프라인이 최고 ㅎㅎ
iris %>% 
  `[`(map_lgl(., is.factor)) %>% 
  car::some()
```

    ##        Species
    ## 26      setosa
    ## 27      setosa
    ## 46      setosa
    ## 47      setosa
    ## 66  versicolor
    ## 99  versicolor
    ## 113  virginica
    ## 117  virginica
    ## 121  virginica
    ## 124  virginica

`some()`과 `every()`는 predicate가 요소에서 사실인지 결정한다.

``` r
x <- list(1:5, letters, list(10))

x %>% 
  some(is_character)
```

    ## [1] TRUE

``` r
x %>% 
  every(is_character)
```

    ## [1] FALSE

``` r
# 차이점 이해
x %>% 
  map_lgl(is.character)
```

    ## [1] FALSE  TRUE FALSE

`detect()`는 predicate가 true인 첫번째 성분을 찾는다. `detect_index()`는 그 위치를 찾는다.

``` r
x <- sample(10)
x
```

    ##  [1]  2  7  3  6 10  4  8  1  9  5

``` r
x %>% 
  detect(~ . > 5)
```

    ## [1] 7

``` r
x %>% 
  detect_index(~ . > 5)
```

    ## [1] 2

`head_while()`과 `tail_while()`은 predicate가 true인 동안 벡터의 시작 또는 끝으로부터 성분을
취한다.

``` r
x %>% 
  head_while(~ . > 5)
```

    ## integer(0)

``` r
x %>% 
  tail_while(~ . > 5)
```

    ## integer(0)

## Reduce and Accumulate

때때로 쌍을 단독 개체로 축소시키는 함수를 반복적으로 적용함으로써 간단한 리스트를 줄이기를 원하는 복잡한 리스트를 가질 수 있다.
two-table dplyr 동사를 다수의 테이블에 적용하기를 원한다면 유용하다. 예를 들어, 데이터프레임의 리스트를 가질수도
있고, 성분을 함께 join함으로써 하나의 데이터 프레임으로 줄이기를 원할 수 있다.

``` r
dfs <- list(
  age = tibble(name = "John", age = 30),
  sex = tibble(name = c("John", "Mary"), sex = c("M", "F")),
  trt = tibble(name = "Mary", treatment = "A")
)

dfs %>% reduce(full_join)
```

    ## Joining, by = "name"
    ## Joining, by = "name"

    ## # A tibble: 2 x 4
    ##   name    age sex   treatment
    ##   <chr> <dbl> <chr> <chr>    
    ## 1 John     30 M     <NA>     
    ## 2 Mary     NA F     A

``` r
# reduce를 사용하지 않으면
# full_join(full_join(dfs[[1]], dfs[[2]]), dfs[[3]])
```

벡터의 리스트가 있다면 교집합을 찾기 원할 수 있다.

``` r
vs <- list(
  c(1, 3, 5, 6, 10),
  c(1, 2, 3, 7, 8, 10),
  c(1, 2, 3, 4, 8, 9, 10)
)
vs %>% reduce(intersect)
```

    ## [1]  1  3 10

reduce함수는 이진 함수를 취해서 리스트가 단 하나의 성분만 가질 때 까지 반복한다.

``` r
x <- sample(10)
x %>% accumulate(`+`)
```

    ##  [1] 10 16 20 23 30 39 47 48 50 55

## Exercises

1.  Implement your own version of every() using a for loop. Compare it
    with purrr::every(). What does purrr’s version do that your version
    doesn’t?

<!-- end list -->

``` r
# Use ... to pass arguments to the function
every2 <- function(.x, .p, ...) {
  for (i in .x) {
    if (!.p(i, ...)) {
      # If any is FALSE we know not all of then were TRUE
      return(FALSE)
    }
  }
  # if nothing was FALSE, then it is TRUE
  TRUE  
}
```

2.  Create an enhanced col\_sum() that applies a summary function to
    every numeric column in a data frame.

<!-- end list -->

``` r
col_sum2 <- function(df, f, ...) {
  map(keep(df, is_numeric), f, ...)
}
col_sum3 <- function(df, f, ...) {
  is_num <- vapply(df, is_numeric, logical(1))
  df_num <- df[, is_num]
  sapply(df_num, f, ...)
}

col_sum4 <- function(df, f, ...) {
  df %>% 
    `[`(map_lgl(., is_numeric)) %>% 
    map_dbl(f, ...)
}

df <- tibble(
  x = 1:3,
  y = 3:1,
  z = c("a", "b", "c")
)
# OK
col_sum3(df, mean)
```

    ## Warning: Deprecated
    
    ## Warning: Deprecated
    
    ## Warning: Deprecated

    ## x y 
    ## 2 2

``` r
# Has problems: don't always return numeric vector
col_sum3(df[1:2], mean)
```

    ## Warning: Deprecated
    
    ## Warning: Deprecated

    ## x y 
    ## 2 2

``` r
col_sum3(df[1], mean)
```

    ## Warning: Deprecated

    ## x 
    ## 2

``` r
col_sum3(df[0], mean)
```

    ## named list()

<https://stackoverflow.com/questions/17499013/how-do-i-make-a-list-of-data-frames/24376207#24376207>
를 읽고 공부해보자.

# Summary

  - 입력으로 함수를 취하고(리스트로) 출력으로 벡터를 반환: functional
  - `inherits(x, "classname")`: 어떤 객체가 특정 클래스를 상속했는가
  - 루프 시작 전 `output <- vector("list", length(df))`로 사전할당.
  - `for(nm in names(xs))`
  - `paste()`를 각 반복에 사용하는 대신에 문자형 벡터를 저장하고 벡터를 하나의 문자열로 조합하라.
  - `cat()`은 보이지 않게 NULL을 반환하지만 `print()`는 인자를 반환하므로 pipe와 조합하면 유용하다.
  - `mtcars %>% split(.$cyl) %>% map(~lm(mpg ~ wt, data = .)) %>%
    map(summary) %>% map_dbl("r.squared")`
  - `map(params, safely(fun)) %>% transpose()` 에서 `y$error %>%
    map_lgl(is_null)` `x[!is_ok]`로 오류를 찾을 수 있다. 올바른 값은 `y$result[is_ok]
    %>% flatten_dbl()`로 볼 수 있다.대신 `possibly()`나 `quietly()`도 있다.
  - map2나 pmap의 인자가 복잡해지면 어차피 길이는 모두 같으므로 데이터프레임으로 만들어버리자.
  - 함수 그 자체를 다양화하기 위해 `invoke_map()`을 사용
  - 리스트를 디스트에 저장할 때는 `walk()`계열의 함수를 이용하자.
  - map과 \[\]를 같이 사용하는 대신에 `keep()`이나 `discard()`를 사용할 수 있다.
  - `some()`과 `every()`, `detect()`, `detect_index()`, `head_while()`,
    `tail_while()` 있다.
  - join함수와 `reduce()`를 결합하자
  - `list.files(pattern = "\\.csv$")`로 데이터를 불러와서 리스트로 저장
  - `split()`로 CV셋 나누기
  - 데이터를 리스트로 만들지 않았다면 `mget(ls(pattern = "df[0-9]"))`로 데이터를 리스트로 만들기
  - `data.table::rbindlist()`나 `dplyr::bind_rows()`를 사용해서 하나의 데이터프레임으로
    합치기
