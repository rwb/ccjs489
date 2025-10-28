* An estimator is a formula or way of calculating some scientifically interesting quantity.
* Estimators are used to calculate estimates (sometimes people say "estimand").
* Scientists must attend to the properties of the estimators they use for describing phenomena and developing inferences.
* When we conduct a statistical analysis we are usually doing one of the following things: (1) point estimation; (2) interval estimation; and/or (3) hypothesis testing.
* It is natural to be concerned with how accurately we are performing these tasks.
* To motivate this discussion, let's consider the case of a normally distributed random variable, *y*, for a large population.
* This generates the population distribution of y (mean of about 100 and a standard deviation of about 10):

```R
y <- rnorm(n=1000000,mean=100,sd=10)
mean(y)
sd(y)
hist(y)
```

* Here is our output:

```Rout
> y <- rnorm(n=1000000,mean=100,sd=10)
> mean(y)
[1] 100.0005
> sd(y)
[1] 10.00185
> hist(y)
> 
```

* So far, all we've done is consider the population.
* Now, let's draw a single sample, yss, of size 100 from the population -- with replacement.

```R
yss <- sample(y,size=100,replace=T)
```

* The y values for the single sample are included in the vector, *yss*.
* Now, let's suppose our objective is to estimate the *population mean* using the *sample* information.
* We need to choose an estimator to produce the estimate.
* An obvious choice for an estimator would be the *sample mean*: $\overline{y} = \frac{1}{n} \sum_{i=1}^n y_i$
* But another choice for an estimator could be the *sample median* or the middle score of the distribution.
* Let's calculate both quantities for our single sample:

```
mean(yss)
median(yss)
```

* Here is our output:

```Rout
> mean(yss)
[1] 99.91584
> median(yss)
[1] 99.46716
>
```

* Let's draw another sample and see what happens:

```R
yss <- sample(y,size=100,replace=T)
mean(yss)
median(yss)
```

* Here is our next output:

```Rout
> yss <- sample(y,size=100,replace=T)
> mean(yss)
[1] 101.0809
> median(yss)
[1] 100.2906
```

* In both samples the mean and median are close to the population mean
* In the first sample, the mean is closer; in the second sample the median is closer.
* How should we choose which estimator to use?
* To investigate this, let's draw repeated samples of size n = 100 from this population.

```R
ymean <- vector()
ymedian <- vector()

for(i in 1:1e5){
  ys <- sample(y,size=100,replace=T)
  ymean[i] <- mean(ys)
  ymedian[i] <- median(ys)
  }

mean(ymean)
mean(ymedian)
boxplot(ymean,ymedian,names=c("sample mean","sample median"))
```

* Here is our output:

```Rout
> ymean <- vector()
> ymedian <- vector()
> 
> for(i in 1:1e5){
+   ys <- sample(y,size=100,replace=T)
+   ymean[i] <- mean(ys)
+   ymedian[i] <- median(ys)
+   }
> 
> mean(ymean)
[1] 99.99852
> mean(ymedian)
[1] 99.9985
> boxplot(ymean,ymedian,names=c("sample mean","sample median"))
>
```

* This repeated sampling exercise yields an approximation to the *sampling distribution* of the two sets of estimates.
* Both the sample mean and median seem to be *unbiased* (right on average) estimators of the population mean.
* But the sample median has greater dispersion than the sample mean -- it is less *efficient*.
* So, we would prefer the sample mean over the sample median on efficiency grounds
* We can confirm this by looking at the standard deviation of each distribution of sample means and sample medians.

```R
sd(ymean)
sd(ymedian)
```

* which gives the following output:

```Rout
> sd(ymean)
[1] 1.000298
> sd(ymedian)
[1] 1.243241
>
```

* Not by coincidence, the *mean squared error* for the two estimators gives us the same result (this is just the square of the standard deviations or the variance of the sampling distribution).

```R
dmean <- ymean-100
sum(dmean^2)/1e5
dmedian <- ymedian-100
sum(dmedian^2)/1e5
```

* Here is the output:

```Rout
> dmean <- ymean-100
> sum(dmean^2)/1e5
[1] 1.000589
> dmedian <- ymedian-100
> sum(dmedian^2)/1e5
[1] 1.545636
> 
```

* Now, let's think about a 90% confidence interval for the sample mean.
* We can calculate this interval for a single sample using the following approach:

```R
yss <- sample(y,size=100,replace=T)
mean(yss)
sd(yss)
se.meanyss <- sd(yss)/sqrt(100)
se.meanyss
p5 <- qnorm(p=0.05,mean=0,sd=1)
p5
p95 <- qnorm(p=0.95,mean=0,sd=1)
p95
lcl <- mean(yss)+p5*se.meanyss
lcl
ucl <- mean(yss)+p95*se.meanyss
ucl
```

* Here is our output:

```Rout
> yss <- sample(y,size=100,replace=T)
> mean(yss)
[1] 99.35032
> sd(yss)
[1] 10.99164
> se.meanyss <- sd(yss)/sqrt(100)
> se.meanyss
[1] 1.099164
> p5 <- qnorm(p=0.05,mean=0,sd=1)
> p5
[1] -1.644854
> p95 <- qnorm(p=0.95,mean=0,sd=1)
> p95
[1] 1.644854
> lcl <- mean(yss)+p5*se.meanyss
> lcl
[1] 97.54235
> ucl <- mean(yss)+p95*se.meanyss
> ucl
[1] 101.1583
>
```

* So, the calculated 90% confidence interval for this particular sample is [97.542,101.1583] -- which traps the true population value of 100.
* The creation of a valid 90% confidence interval means that our interval has *at least* a 90% chance of trapping the true population value of 100. What does this mean?
* Here is some code to check on whether it really works:

```R
ymean <- vector()
se.ymean <- vector()
lcl90 <- vector()
ucl90 <- vector()

for(i in 1:1e5){
  ys <- sample(y,size=100,replace=T)
  ymean[i] <- mean(ys)
  se.ymean[i] <- sd(ys)/sqrt(100)
  lcl90[i] <- ymean[i]+qnorm(p=0.05,mean=0,sd=1)*se.ymean[i]
  ucl90[i] <- ymean[i]+qnorm(p=0.95,mean=0,sd=1)*se.ymean[i]
  }

mean(ymean)
sd(ymean)
mean(se.ymean)
mean(ucl90-lcl90)
trap <- ifelse(lcl90<100 & ucl90>100,"hit","miss")
table(trap)
```

* Here is the output:
  
```Rout
> mean(ymean)
[1] 99.99113
> sd(ymean)
[1] 1.004598
> mean(se.ymean)
[1] 0.9969187
> mean(ucl90-lcl90)
[1] 3.279571
> trap <- ifelse(lcl90<100 & ucl90>100,"hit","miss")
> table(trap)
trap
  hit  miss 
89468 10532 
> 
```

* As you can see, it's pretty close but not quite right. It turns out this is *not* a valid 90% confidence interval.
* We should be using the t-distribution instead of the normal distribution with a sample size of 100.
* Let's try the t-distribution and see what we get:

```R
ymean <- vector()
se.ymean <- vector()
lcl90 <- vector()
ucl90 <- vector()

for(i in 1:1e5){
  ys <- sample(y,size=100,replace=T)
  ymean[i] <- mean(ys)
  se.ymean[i] <- sd(ys)/sqrt(100)
  lcl90[i] <- ymean[i]+qt(p=0.05,df=100-1)*se.ymean[i]
  ucl90[i] <- ymean[i]+qt(p=0.95,df=100-1)*se.ymean[i]
  }

mean(ymean)
mean(se.ymean)
mean(ucl90-lcl90)
trap <- ifelse(lcl90<100 & ucl90>100,"hit","miss")
table(trap)
```

* That's better!

```Rout
> ymean <- vector()
> se.ymean <- vector()
> lcl90 <- vector()
> ucl90 <- vector()
> 
> for(i in 1:1e5){
+   ys <- sample(y,size=100,replace=T)
+   ymean[i] <- mean(ys)
+   se.ymean[i] <- sd(ys)/sqrt(100)
+   lcl90[i] <- ymean[i]+qt(p=0.05,df=100-1)*se.ymean[i]
+   ucl90[i] <- ymean[i]+qt(p=0.95,df=100-1)*se.ymean[i]
+   }
> 
> mean(ymean)
[1] 99.99331
> mean(se.ymean)
[1] 0.9976558
> mean(ucl90-lcl90)
[1] 3.312998
> trap <- ifelse(lcl90<100 & ucl90>100,"hit","miss")
> table(trap)
trap
  hit  miss 
90076  9924 
> 
```
