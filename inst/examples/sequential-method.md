

## Sequential method


```r
require(MASS)
require(ggplot2)
require(kernlab)
opts_knit$set(upload.fun = socialR::notebook.url)
```


Parameterization-specific



```r
X <- 1
obs <- data.frame(x = c(-3, -1, 3),
                  y = c(0,  1, -1))
#x <- -5:5
#obs <- data.frame(x = -5:5, y = sin(x) + rnorm(length(x),sd=.1))
l <- 1
sigma_n <- 0.
```



Radial basis function/Gaussian kernel:


```r
  SE <- function(Xi,Xj, l=1) exp(-0.5 * (Xi - Xj) ^ 2 / l ^ 2)
  cov <- function(X, Y) outer(X, Y, SE, l) 
```



### Cholesky method
  

```r
  n <- length(obs$x)
  K <- cov(obs$x, obs$x)
  I <-  diag(1, n)

  L <- chol(K + sigma_n^2 * I)
  alpha <- solve(t(L), solve(L, obs$y))
  k_star <- cov(obs$x, X)
  Y <- t(k_star) %*% alpha
  v <- solve(L, k_star)
  Var <- cov(X,X) - t(v) %*% v
  loglik <- -.5 * t(obs$y) %*% alpha - sum(log(diag(L))) - n * log(2 * pi) / 2
```

  
### Direct method 


```r
  cov_xx_inv <- solve(K + sigma_n^2*I)
  Ef <- cov(X, obs$x) %*% cov_xx_inv %*% obs$y
  Cf <- cov(X, X) - cov(X, obs$x)  %*% cov_xx_inv %*% cov(obs$x, X)
```


### Direct sequential method, avoids matrix inverse instability


```r
ef <- numeric(length(X))
cf <- matrix(0, nrow=length(X), ncol=length(X))

A <- as.numeric( cov(obs$x[1], obs$x[1]) )
mu <- obs$y[1] 

for(i in 2:length(obs$x)){
  mu <- obs$y[i] + cov(obs$x[i], obs$x[1:(i-1)]) %*% (obs$y[1:(i-1)] - mu) / (A + sigma_n^2)
  A <- as.numeric( cov(obs$x[i], obs$x[i]) - cov(obs$x[i], obs$x[1:(i-1)]) %*% cov(obs$x[1:(i-1)], obs$x[i]) / (A + sigma_n^2) )
}
  
ef <- cov(X, obs$x) %*% (obs$y - mu) / A
cf <- cov(X,X) - cov(X, obs$x) %*% cov(obs$x, X) / A
```



See if this iteration scheme is correct?


```r
V1 <- cov(x[1], x[1])
V2 <- cov(x[2], x[2]) - cov(x[2], x[1]) %*% solve(cov(x[1], x[1])) %*% cov(x[1], x[2])
V2b <- cov(x[2], x[2]) - cov(x[2], x[1]) %*% cov(x[1], x[2]) / V1
V3 <- cov(x[3], x[3]) - cov(x[3], x[1:2]) %*% solve(cov(x[1:2], x[1:2])) %*% cov(x[1:2], x[3])
V3b <- cov(x[3], x[3]) - cov(x[3], x[1:2]) %*% cov(x[1:2], x[3]) / V2
c(V3, V3b)
```

```
[1]  0.0429 -9.9074
```
