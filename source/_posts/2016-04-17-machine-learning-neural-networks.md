title: Machine Learning Neural Networks
date: 2016-04-17 21:29:20
tags:
    - Machine Learning
---

This week is about the mysterious Neural Networks. The courses in this week just explain the basics about Neural Networks.

## What is Neural Networks
It's a technique to train our data based on how human brains works. A simple Neural Network has:
- input layer
- hidden layer
- output layer

We use Neural NetWorks to make classification and regression.
We use sigmoid function the map data from input layer to hidden layer then the output layer, the function is called activation function.
<!--more-->
![Neural Network](http://ww3.sinaimg.cn/mw690/761b7938jw1f3019b35a2j21380kcn1d.jpg)

In Neural Network, we add bias unit, x0, a1 to do calculate.
With Neural Network, we build more complex hypothesis function.
![Neural Network](http://ww4.sinaimg.cn/mw690/761b7938jw1f301cv572fj214o0m2teg.jpg)

To play with neural network, you can try google's open source [tensorflow](http://playground.tensorflow.org/#activation=tanh&batchSize=10&dataset=circle&regDataset=reg-plane&learningRate=0.03&regularizationRate=0&noise=0&networkShape=4,2&seed=0.28657&showTestData=false&discretize=false&percTrainData=50&x=true&y=true&xTimesY=false&xSquared=false&ySquared=false&cosX=false&sinX=false&cosY=false&sinY=false&collectStats=false&problem=classification)

## Handwritten Digital Classification
This week's assignment is to do multi classification on handwritten recognize.

**Do multi classification with one-vs-all logistic regression**
For handwritens in 10 lables: 0~9, we do 10 regression regression to calculate 10 group of theta. And then make 10 predications base on these 10 group of theta, choose the lable with max hypothesis value（probaility value）

**1. Cost function**
```matlab
function [J, grad] = lrCostFunction(theta, X, y, lambda)
reg_theta = [0;theta(2:end)];
predictions = sigmoid(X * theta);
J = (-1 / m) * sum((y .* log(predictions) + (1 - y) .* log(1 - predictions))) + (lambda / (2 * m)) * sum(reg_theta .^ 2);
grad = (1 / m) * (X' * (predictions - y)) + (lambda / m) * reg_theta;
end
```
**2. OneVsAll**
make 10 classifications
```matlab
function [all_theta] = oneVsAll(X, y, num_labels, lambda)
% Some useful variables
m = size(X, 1);
n = size(X, 2);

% You need to return the following variables correctly
all_theta = zeros(num_labels, n + 1);

% Add ones to the X data matrix
X = [ones(m, 1) X];

% Note: For this assignment, we recommend using fmincg to optimize the cost
%       function. It is okay to use a for-loop (for c = 1:num_labels) to
%       loop over the different classes.
%
%       fmincg works similarly to fminunc, but is more efficient when we
%       are dealing with large number of parameters.
%
for i=1:num_labels
    initial_theta = zeros(n + 1, 1);
    options = optimset('GradObj', 'on', 'MaxIter', 50);
    % theta is a column vector
    % Run fmincg to obtain the optimal theta
    [theta] = fmincg(@(t)(lrCostFunction(t, X, (y == i), lambda)), ...
                    initial_theta, options);
    all_theta(i, :) = theta';
end

end

```
**3. PredictOneVsAll**
```matlab
function p = predictOneVsAll(all_theta, X)
m = size(X, 1);
num_labels = size(all_theta, 1);

% You need to return the following variables correctly
p = zeros(size(X, 1), 1);

% Add ones to the X data matrix
X = [ones(m, 1) X];
predictions = X * all_theta';
% calculate max of each row
[max_predictions, p] = max(predictions, [], 2);
end
```
The logistic has great accurracy, about 95% in this case, but neural network will have higher accuracy, about 97%.

## Neural Forward Propagation algorithm
In the assignment, it build hypothesis function with 3 layers neural network.
The predications implementation

```matlab
function p = predict(Theta1, Theta2, X)
%PREDICT Predict the label of an input given a trained neural network
%   p = PREDICT(Theta1, Theta2, X) outputs the predicted label of X given the
%   trained weights of a neural network (Theta1, Theta2)

% Useful values
m = size(X, 1);
num_labels = size(Theta2, 1);

% You need to return the following variables correctly 
p = zeros(size(X, 1), 1);

% ====================== YOUR CODE HERE ======================
% Instructions: Complete the following code to make predictions using
%               your learned neural network. You should set p to a 
%               vector containing labels between 1 to num_labels.
%
% Hint: The max function might come in useful. In particular, the max
%       function can also return the index of the max element, for more
%       information see 'help max'. If your examples are in rows, then, you
%       can use max(A, [], 2) to obtain the max for each row.
%
% Theta1 is 25 * 401, X is 5000 * 401
X = [ones(m, 1) X];
% z2 is 5000 * 25
z2 = X * Theta1';
a2 = sigmoid(z2);
% a2 with bias unit is 5000 * 26
a2 = [ones(m, 1) a2];

% Theta2 is 10 * 26
% z3 is 5000 * 10
z3 = a2 * Theta2';
a3 = sigmoid(z3);
[max_valid, p] = max(a3, [], 2);

% =========================================================================

end
```

My question is:
- how to train the Theta1, Theta2
- how to decide how many units in hidden layer
In the later course, I think NG will explain it. Next week, I will learn backpropagation algorithm.

## My assignment
[Week Assignments](https://github.com/lgrcyanny/MachineLearningCoursera/tree/master/assignments)




