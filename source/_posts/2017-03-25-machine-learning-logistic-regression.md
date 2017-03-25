title: Machine Learning Logistic Regression
date: 2017-03-25 21:48:51
tags:
    - Machine Learning
---
Logistic Regression is for classification problem, and the predication value is fixed descrete values, such as 1 for positive or 0 for negative. The essence of logistic regression is:
- hypothesis function is sigmoid function
- cost function: J(theta)
- gradient descent and algorithms
- advantanced optimization with regularization to solve overfitting problem.
<!--more-->
## Basics about logistic regression
hypothesis function = 1 / (1 + exp(-htheta(x))),
where htheta(x) = theta' * x(theta' is transpose theta)
![Sigmoid Function or Logistic Function](http://ww2.sinaimg.cn/mw690/761b7938jw1f2rxxio8x0j20v80nit9x.jpg)
htheta(x) mean **Probalitiy that y=1, given x parameterized by theta P(y=1 | x; theta)**,
```matlab
if htheta(x) >= 0.5, then y = 1
if htheta(x) < 0.5, then y = 0

```
## Descision Boundary
![descision boundary](http://ww3.sinaimg.cn/mw690/761b7938jw1f2rxxhyf4ij20v00ngtbs.jpg)
Our goal is the calculate theta, can classify our traing data with descision boundary.
In the example, the traning data can be classified into 2 categories by a straight line.
```matlab
if (theta'x) >= 0, then htheta(x) >= 0.5, then y = 1
if (theta'x) < 0, then htheta(x) < 0.5, then y = 0
```

## Cost function implementation
For the assignment of week3, predicate the adimission by university with 2 exams grade data.
I optimize the implementation with vectoriaztion

```matlab
function [J, grad] = costFunction(theta, X, y)
%COSTFUNCTION Compute cost and gradient for logistic regression
%   J = COSTFUNCTION(theta, X, y) computes the cost of using theta as the
%   parameter for logistic regression and the gradient of the cost
%   w.r.t. to the parameters.

% Initialize some useful values
m = length(y); % number of training examples

% You need to return the following variables correctly
J = 0;
grad = zeros(size(theta));

% ====================== YOUR CODE HERE ======================
% Instructions: Compute the cost of a particular choice of theta.
%               You should set J to the cost.
%               Compute the partial derivatives and set grad to the partial
%               derivatives of the cost w.r.t. each parameter in theta
%
% Note: grad should have the same dimensions as theta
%
% Predications: h_theta(x)
predications = sigmoid(X * theta);
cost_items = y .* log(predications) + (1 - y) .* log(1 - predications);
J = (-1 / m) * sum(cost_items);

grad = (1 / m) * (X' * (hypothesis - y));

% =============================================================

end
```

## Cost function with regularization
Regularzation is for overfitting problem.
- underfit: not fit the training data, with high bias between predications and actual value
- Just Right: great fit
- Overfitting:  often with too many features, not so much traning data, fit traing data well, but with hight variance, predict new data not very well

```matlab
function [J, grad] = costFunctionReg(theta, X, y, lambda)
%COSTFUNCTIONREG Compute cost and gradient for logistic regression with regularization
%   J = COSTFUNCTIONREG(theta, X, y, lambda) computes the cost of using
%   theta as the parameter for regularized logistic regression and the
%   gradient of the cost w.r.t. to the parameters.

% Initialize some useful values
m = length(y); % number of training examples

% You need to return the following variables correctly
J = 0;
grad = zeros(size(theta));

% ====================== YOUR CODE HERE ======================
% Instructions: Compute the cost of a particular choice of theta.
%               You should set J to the cost.
%               Compute the partial derivatives and set grad to the partial
%               derivatives of the cost w.r.t. each parameter in theta
hypothesis = sigmoid(X * theta);
cost_items = (y .* log(hypothesis)) + (1 - y) .* log(1 - hypothesis);
% don't penalize theta0
reg_theta = [0; theta(2:length(theta))];
J = (-1 / m) * sum(cost_items) + (lambda / (2 * m)) * sum(reg_theta .^ 2);
%grad = (1 / m) * sum((predications - y) .* X)' + (lambda / m) * penalize_theta;
grad = (1 / m) * (X' * (hypothesis - y)) + (lambda / m) * reg_theta;

% =============================================================

end

```

the lambda for regularization can't be too large:
- large lamba will got very small theta value, and underfit.
- small lambda will got large theta velue, and overfit.
- the lambda for the exerise is 1


## Github assignments
[Week 3 Assignments](https://github.com/lgrcyanny/MachineLearningCoursera/tree/master/assignments/ex2-logistic-regression)

## Write on the last
After one year, I learn the logistic regression again. Last week, Andrew NG left Baidu. Maybe, these great people thought Baidu is not worth to fight for. Now I still decidated on a Spark project and focus on Spark Streaming. As team leader, I am bearing a great burden and is stressful. It's a great chance to train my leadership. I am also wondering next opportunity. Learning Machine Learning is right and worth to do. Anyway, even though mist is on the path, just go forward and fight~