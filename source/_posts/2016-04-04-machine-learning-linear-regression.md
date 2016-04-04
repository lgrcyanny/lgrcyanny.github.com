title: Machine Learning Linear Regression
date: 2016-04-04 15:55:31
tags:
    - Machine Learning
    - Coursera
id: 495
categories:
      - Machine Learning
      - Linear Regression
---

I have been learning the coursera Machine Learning Course by Andrew Ng for two weeks now. Machine Learning is fun and different. For the coursera assignment1 of linear regression, I want to share something.
<!--more-->
## Octave Install
The course use Octave/Matlab for programming practice. I learned octave basics in two days. I don't have too much time, can just doing these homework in weekends. For Octavel installed on mac, I encounter some problems and solve it.

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
Implementing gradient desenct algorithm in vectorization style is what I am proud of. Here is my implementation:
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
    sum_delta = (alpha / m) * sum(errors .* X, 1); % sum by column, which is 1 by n + 1 matrix
    theta = theta - sum_delta';

    % ============================================================

    % Save the cost J in every iteration    
    J_history(iter) = computeCost(X, y, theta);

end

end

```

## My assignments on github
[Assignments1](https://github.com/lgrcyanny/MachineLearningCoursera/tree/master/assignments/ex1/ex1)