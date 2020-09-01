# Moneyball Analytics


Here the salaries of baseball players were predicted based on the dataset that contains information on 263 players from the MLB in 1986.

Three predictive analysis techniques were used for comparison, discussed in the following structure.

1. Data Exploration
2. Linear Regression
3. Ridge Regression and LASSO
4. XGBOOST


## Data Exploration
Let's first extract only the numerical predictors from the dataset to look at their correlations between each other.<br />
```bash
hitters_raw <- read.csv("Hitters.csv")
hitters_num <-hitters_raw[,2:18]
ggcorrplot(round(cor(hitters_num),1), method="circle", 
           tl.col = "black", tl.cex = 15, lab=TRUE)
```
<p align="center">
<img src="./img/1.a_1.png" width="300" align='middle'>
</p>

The correlation matrix forms two box clusters based on the seasonal (lower-left) and career-long (upper-right) performance stats; the later shows a stronger correlation than the former. Based on the matrix, CRuns (Number of runs in the career) and CRBI (RBI: Number runs enabled in the career) are the most correlated predictors to the salary.

Note that this may not necessarily imply causation. There are several possible explanations: (a) A influences B; (b) B influences A; and (c) A and B are influenced by one or more additional variables.
<br /><br />
Now let's look at the p-values between the salary and other predictors.

```bash
lm.mod <- lm(Salary ~., data = hitters_num)
summary(lm.mod)
```
<p align="center">
<img src="./img/1.a_p.png" width="300" align='middle'>
</p>
The predictors with the 95%-level significance are AtBat, Hits, Walks, CRuns, CWalks, and PutOuts, which could indicate their strong influences on the salary.

## Linear Regression

Based on the EDA (Exploratory Data Analysis), CRBI, AtBat, Hits, Walks, CRuns, CWalks, and PutOuts are considered important to the salary. Before building a linear regression model with these predictors, let's first normalize the data to align data features in a same scale of importance.

```bash
#Normalize and split data
pp <- preProcess(hitters_raw, method=c("center", "scale"))
Hitters <- predict(pp, hitters_raw)
set.seed(15071)
train.obs <- sort(sample(seq_len(nrow(Hitters)), 0.7*nrow(Hitters)))
train <- Hitters[train.obs,2:21]
test <- Hitters[-train.obs,2:21]

#Fit a restricted Linear Regression with variables with significance
head(train)
lin.mod <- lm(Salary ~ CRBI+AtBat+Hits+Walks+CRuns+CWalks+PutOuts,
               data = train)
pred.train = predict(lin.mod, newdata = train)

#Out-of-Sample R-squared
pred.test = predict(lin.mod, newdata = test)
SSE.test = sum((pred.test - test$Salary)^2)
SST.test = sum((test$Salary - mean(train$Salary))^2)
OSR2 = 1 - SSE.test/SST.test
```

### **Prediction Accuracy**
|Model|In-sample R-squared|Out-of-sample R-squared|
|--|--|--|
|Unrestricted model |**0.5603**|0.4003593|
|Restricted model |0.5163| **0.4742655**|

The unrestricted model trained with all variables performs better with in-sample data, whereas the restricted model with variables with significance outperforms in predicting the out-of-sample data by avoiding the overfitting.

## Ridge Regression and LASSO
i) Train Ridge regression and LASSO models with 10-fold cross-validation

Plots of cross-validated Mean Squared Error as a function of l: