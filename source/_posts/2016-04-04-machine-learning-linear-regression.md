title: Machine Learning Linear Regression
date: 2016-04-04 15:55:31
tags:
    - Machine Learning
---

I have been learning the coursera Machine Learning Course by Andrew Ng for two weeks now. Machine Learning is fun and different. For the coursera assignment1 of linear regression, I want to share something.
<!--more-->

## Using matlab

I think matlab is better than octave, please use coursera account. [Install matlab](https://www.coursera.org/learn/machine-learning/supplement/rANSM/installing-matlab)

## Octave Install
The course use Octave/Matlab for programming practice. I learned octave basics in two days. I don't have too much time, can just doing these homework in weekends. For Octavel installed on mac, I encounter some problems and solved it. Now octave is 4.2.0, I think ocatve is better now.

```shell
    brew install octave
```

if you encounter some problem, you can solve it as follows:
- brew update && brew upgrade
- brew tap --repair
- brew install octave
- install xserver(seems no need to install)
    - http://www.xquartz.org/
- font can’t find when plot
    - export FONTCONFIG_PATH=/opt/X11/lib/X11/fontconfig
- can’t plot unknown or ambiguous terminal type; type just 'set terminal' for a list
    - brew uninstall gnuplot
    - download and install aquaterm: https://sourceforge.net/projects/aquaterm/?source=typ_redirect
    - brew install gnuplot --with-aquaterm --with-qt4
- add start config to /usr/local/share/octave/site/m/startup/octaverc
    - PS1('>> ')

## Gradient Descent Algorithm
Implementing gradient desenct algorithm in vectorization style was more efficient than iteration algorithm. Here is my implementation:
No for loop looks elegant.

```matlab
function [theta, J_history] = gradientDescent(X, y, theta, alpha, num_iters)
%GRADIENTDESCENT Performs gradient descent to learn theta
%   theta = GRADIENTDESENT(X, y, theta, alpha, num_iters) updates theta by
%   taking num_iters gradient steps with learning rate alpha

% Initialize some useful values
m = length(y); % number of training examples
J_history = zeros(num_iters, 1);

for iter = 1:num_iters

    % ====================== YOUR CODE HERE ======================
    % Instructions: Perform a single gradient step on the parameter vector
    %               theta.
    %
    % Hint: While debugging, it can be useful to print out the values
    %       of the cost function (computeCost) and gradient here.
    %
    predications = X * theta;
    errors = predications - y; % m by 1 vector
    % sum_delta = (alpha / m) * sum(errors .* X, 1); % sum by column, which is 1 by n + 1 matrix
    % transpose X, no need sum(errors .* X, 1) here
    sum_delta = (alpha / m) .* (X' * errors);
    theta = theta - sum_delta;

    % ============================================================

    % Save the cost J in every iteration
    J_history(iter) = computeCost(X, y, theta);
end

end


```

**Another implementation by my wwzyhao**
[by wwzyhao]
```matlab
function [theta, J_history] = gradientDescent(X, y, theta, alpha, num_iters)
%GRADIENTDESCENT Performs gradient descent to learn theta
%   theta = GRADIENTDESENT(X, y, theta, alpha, num_iters) updates theta by
%   taking num_iters gradient steps with learning rate alpha

% Initialize some useful values
m = length(y); % number of training examples
J_history = zeros(num_iters, 1);

for iter = 1:num_iters

    delta = zeros(size(X, 2), 1);
    for j = 1:m
        x = (X(j,:))';
        delta = delta + (1 / m) * (theta' * x - y(j)) * x;
    end;

    theta = theta - alpha * delta;

    % ============================================================

    % Save the cost J in every iteration
    J_history(iter) = computeCost(X, y, theta);

end

end
```
Not better than me! haha~

## My assignments on github
[Assignments1](https://github.com/lgrcyanny/MachineLearningCoursera/tree/master/assignments/ex1/ex1)
For submition errors, please refer to[Jacob Middag](https://learner.coursera.help/hc/en-us/community/posts/204693179-linear-regression-submit-error)