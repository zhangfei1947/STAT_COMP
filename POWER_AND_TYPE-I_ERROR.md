### Power & Type-I Error Rate


Consider testing a hypothesis (e.g, in a linear model `y=mu+b*x+e`, H0: b=0 Vs Ha: b!=0). Based on a test statistic (e.g., p-value) and a decision rule (e.g., reject if p-value<0.05) we may reject or do not reject H0. Thus, we have two possible states of nature (H0 and Ha) and  two possible decisions; the table below classifies each of these cases


|           | Do not reject H0  | Reject H0          |
|-----------|-------------------|---------------------|
| H0 holds  | True Negative  | False Positive |
| Ha holds  | False Negative | True positive  |

**Types of error**: In the table above there are two decisions that are incorrect: the False Positives (also called Type-I errors) the False Negatvies (called the Type-II errors).

Suppose we repeat the experiment a large number of times, each time rejecting or not rejecting H0 based on a sample collected from the population (i.e., repeated sampling from a population). Imagine we have an oracle and know hweather H0 or Ha holds and we count how many TN (N1), FP (N2), FN (N3) and TP (N4) we get, 


|           | Do not reject H0  | Reject H0          |
|-----------|-------------------|---------------------|
| H0 holds  | N1 | N2 |
| Ha holds  | N3 | N4  |


If H0 holds (firsst row in the above-table), the false discovery proportion is: N2/(N1+N2)

The **type-I error rate** is the probability of rejecting the null given that the null is true, that is p(rejecting|H0 holds)= E[N2/(N1+N2)].

The **power** of an experiment is the probability of rejecting the null given that the alternative holds, that is power=p(rejecting|Ha)=E[N4/(N3+N4)].

In cases where we know the sampling distribution of the test statistic, power and Type-I error rate can be computed anlythically. 
However, in many cases we don't know the sampling distribution of the test statistic; in these cases we can estimate power and type-I error rate using Monte Carlo simulations. 

In a MC study we replicate the sampling process (in this case the process that generates the test statistic) a very large number of times, each time applying  the decision rules. We count how many times we reject. If we are simulating under H0, the rejection rate estimates type-I error rate. If we are simulating under Ha, the empirical rejection rate is an estimate of hte power of the experiment. 

The example below illustrates how to estimate power using a simpe linear regression model as an example.

#### Example

Power and error rates depend on three main factors: (i) sample size, (ii) the size of the effect relative to the variance of the noise (i.e., signal-to-noise ratio), and (iii) the test statistics and the decision rule used to reject H0.


The following example evaluates rejection rate for a simple linear model. The R2 parameter controls the signal to noise ratio (R2/(1-R2)), R2=0 simulates from the null, R2>0 simulates from Ha.

```r
  R2=0.01 # Model R-sq.
  N=50 # sample size
  nRep=10000 # number of Monte Carlo replicates
   
  b=sqrt(R2)
  pValues=rep(NA,nRep)
  
  for(i in 1:nRep){
      x=rnorm(N)
      signal=x*b # var(xb)=var(x)*var(b)=var(x)b^2=R2
      error=rnorm(sd=sqrt(1-R2),n=N) 
      y=signal+error
      fm=lsfit(y=y,x=x) # equivalent to lm (i.e., fits model via OLS) but faster
      pValues[i]=ls.print(fm,print.it = FALSE)$coef[[1]][2,4]
  }
  
  reject=pValues<.05 # decision rule
  mean(reject) # since we are simulating under Ha this estimates power
  
```

If you run the above example with R2=0, the estimated rejection rate should be close to 0.05. Why? Because we are rejecting if p-value<0.05, if the p-values are correct (in the above case they are correct because we simulate data under the same normal assumptions that are used to derive the test-statistic) then we expect a rejection rate equal to the significance level used for rejection. But p-values are not always correct. The following example illustrates this.


## What if assumptions don't hold?

If the assumptions made to derive the p-values do not hold, p-values are not necesarily correct and we may have a type-I error rate smaller or higher than the significance level used. The first example uses exponential residuals, this type of violation of assumptions may ahve an effect on type-I error rate, but the effect dissapears with sample size because asymptotically, OLS estimates follows normal distrivbution regardless of the distribution of the errors.

```r
  
  N=10 # sample size
  nRep=100000 # number of Monte Carlo replicates

  pValues=rep(NA,nRep)
  N0=floor(N/2)
  x=c(rep(0,N0),rep(1,N-N0))
  
  for(i in 1:nRep){   
      error=rexp(rate=1,n=N)  # this violates the normality assumption
      y=error # i.e., simulating under H0
      fm=lsfit(y=y,x=x) # equivalent to lm (i.e., fits model via OLS) but faster
      pValues[i]=ls.print(fm,print.it = FALSE)$coef[[1]][2,4]
      if(i%%5000==0){ message(i) }
  }
  
  reject=pValues<.05 # decision rule
  mean(reject,na.rm=T) # Is the test is slightly conservative?, Change N=100
   
```  

**Model miss-specification**: Other violations of assumptions are more complicated, for instance, imagine `Y` is affected by `Z`, which is correlated with `X` and suppose we regress `Y` on `X`. `X` does not have an effect but if we miss-specified the model by ignoring `Z`, then we will get inflated type-I error.

```r 
  N=100
  nRep=10000 # number of Monte Carlo replicates

  pValues=rep(NA,nRep)
  N0=floor(N/2)
  bZ=.1
  
  for(i in 1:nRep){   
      error=rnorm(n=N)  # this violates the normality assumption
      z=rnorm(N)
      x=rnorm(N)+z
      signal=z*bZ
      y=signal + error 
      fm=lsfit(y=y,x=x) # equivalent to lm (i.e., fits model via OLS) but faster
      pValues[i]=ls.print(fm,print.it = FALSE)$coef[[1]][2,4]
      if(i%%1000==0){ message(i) }
  }
  
  reject=pValues<.05 # decision rule
  mean(reject,na.rm=T) 
  
```  


Including `Z` in the model fixes the problem.


```r
  
  N=50
  nRep=10000 # number of Monte Carlo replicates

  pValues=rep(NA,nRep)
  N0=floor(N/2)
  bZ=.1
  
  
  for(i in 1:nRep){   
      error=rnorm(n=N)  # this violates the normality assumption
      z=rnorm(N)
      x=rnorm(N)+z
      signal=z*b
      y=signal + error 
      fm=lsfit(y=y,x=cbind(x,z)) # equivalent to lm (i.e., fits model via OLS) but faster
      pValues[i]=ls.print(fm,print.it = FALSE)$coef[[1]][2,4]
      if(i%%1000==0){ message(i) }
  }
  
  reject=pValues<.05 # decision rule
  mean(reject,na.rm=T) 
  
```  
[Main]( https://github.com/gdlc/STAT_COMP/blob/master/README.md )
