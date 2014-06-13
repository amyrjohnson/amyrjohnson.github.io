---
layout: post
title: "Matrices in Ruby"
date: 2014-06-11 13:05:08 -0400
comments: true
categories: "Flatiron&nbsp;School"
---


Ruby has a matrix library that can be used to do simple linear algebra. The Matrix class defines useful methods like transpose, rank, and determinant. This is nice, because a lot of practical problems can be formulated in terms of linear algebra. Say that we have the following data about recently sold homes:

```
| ---- Table -- |  --  Square Footage -- |  -- # Bathrooms --  |  -- Price ---- |
|:-------------:|:----------------------:|:-------------------:|:--------------:|
|  House 0      |4000                    |3                    |400000          |
|  House 1      |3000                    |2                    |310000          |
|  House 2      |3800                    |3                    |375000          |
|  House 3      |3200                    |2                    |315000          |
|  House 4      |3542                    |4                    |350000          |
|  House 5      |2348                    |2                    |250000          |
|  House 6      |2987                    |3                    |300000          |
|  House 7      |4300                    |4                    |450000          |
|  House 8      |3342                    |4                    |360000          |
|  House 9      |3000                    |3                    |320000          |
```
	
The data has, for each house, the square footage, number of bathrooms, and the price that the house sold for. If you wanted to predict what price another house (not in this data set) might sell for, and you knew its size and bathroom count, you could use the Matrix class to perform a linear regression on the data.  

In linear regression we attempt to express our dependent variable (in this case, housing price), as a linear combination of the independent variables (bathrooms and square footage) plus some constant.


**<center>f(x,y) = ax + by + c</center>**

For any given triplet of datapoints (square footage, number of bathrooms, and price), the difference between the actual price and the price predicted by our equation, squared is the pointwise error. In order to determine the values of a, b, and c that make the most sense, we choose the values that minimize the total average squared error. If the data can be expressed as an matrix X such that X<sup>T</sup>X is invertible, there is an easy formula to get the right values: 

**<center>w = (X<sup>T</sup>X)<sup>-1</sup>X<sup>T</sup>b</center>**

where w is the matrix containing the coefficients `a`, `b`, and `c`, `X` represents the dependent variable matrix, and `b` represents the independent variable matrix.

With the Ruby matrix class these manipulations will be easy. Creating matrices in Ruby is simple. First add a line to require the matrix library, and you're ready to get started. Ruby stores each matrix as an array of rows. Just like arrays, the indices of matrix rows and columns start at zero. Don't forget that all the rows must be the same length for the matrix to be valid.
 

Here we enter the data from the table into two matrices. Matrices can be instantiated in a variety of ways. I chose to simply list out the rows as nested arrays, and unless specified otherwise that's what Ruby assumes the input will be. Each row represents a house with square footage in column 1 (the second column since ruby indexes columns from 0) and the number of bathrooms in column 2. The constant value in column 0 is necessary for the calculation of the constant c from our previous equation. The second matrix contains the housing prices. These two matrices are all we need for our calculations. 

```
require 'matrix'

housing_data = Matrix[[1,4000,3], 
                      [1,3000,2], 
                      [1,3800,3], 
                      [1,3200,2], 
                      [1,3542,4], 
                      [1,2348,2],
                      [1,2987,3],
                      [1,4300,4],
                      [1,3342,4],
                      [1,3000,3]
   					]

housing_prices = Matrix[[400000],
						[310000],
						[375000],
						[315000],
						[350000],
						[250000],
						[300000],
						[450000],
						[360000],
						[320000]
						]
```
 So, to get started on our handy equation, let's transpose `X`, our housing data matrix. Ruby has a useful function that will transpose a matrix for you, as you can see above. 

```ruby
x_t = housing_data.transpose

 => Matrix[[1, 1, 1, 1, 1, 1, 1, 1, 1, 1], 
 			[4000, 3000, 3800, 3200, 3542, 2348, 2987, 4300, 3342, 3000], 
 			[3, 2, 3, 2, 4, 2, 3, 4, 4, 3]] 
```
 Next we multiply the trasposed matrix by the original housing data matrix. The standard operators of `*, +, -, /,` and `**` perform the corresponding operations on matrices.

```ruby
 x_t_x = x_t * housing_data

  => Matrix[[10, 33519, 30], 
  			[33519, 115320001, 103193], 
  			[30, 103193, 96]] 
```

We can invert the resulting matrix using the inverse method.

```ruby
 x_t_x.inverse

  => Matrix[[(421924847/108574934), (-61017/54287467), (-673863/108574934)], 
            [(-61017/54287467), (30/54287467), (-13180/54287467)], 
            [(-673863/108574934), (-13180/54287467), (29676649/108574934)]] 
```
Finally, we can get the coefficients matrix with two more multiplication operations. Those fractions look pretty unwieldy, so let's iterate through the matrix and change them to floating point numbers to get a better idea of what's going on.

```ruby
w = (x_t_x.inverse * x_t) * housing_prices

 => Matrix[[(1018615352500/54287467)], 
 		    [(4850790000/54287467)], 
 		    [(447540942500/54287467)]] 
```
`Matrix.each` reads left to right, then down the rows by default, but you can also specify options, for instance only reading down the diagonal (`row index == column index`). Since here we only have a simple [3x1] matrix, the default method works just fine. Rather than initializing and returning a new matrix, we can also just use the `collect` method in place of `each` to return the new matrix automatically.

```
w.collect do |i|
	i.to_f.round(2)
end

=> Matrix[[18763.36], 
          [89.35], 
          [8243.91]] 
```

And there we have the coefficients in a matrix. The constant, c, is in row 0, the square footage coefficient, b, is in row 1, and the bathroom coeffient, a, is in row 2 (note that this mirrors the column structure of the original data matrix). As we would expect, both a and b are positive since higher square footage and more bathrooms are both associated with a higher price. Additionally, the magnitude of the coefficient for square footage (row 1) is much smaller than the one for bathrooms, since square footage is a much bigger number than bathroom count. Let's revisit our original example of a 3500 square foot house with 3 bathrooms and determine the predicted price.

```
f(x,y) = 8243.91*3 + 89.35*3500 + 18763.36

 => 356220.08999999997 
```
Our equation predicts that this house would sell for $356,220, which makes sense compared to the other points in our data set. 

For more information on using matrices in Ruby checkout the [Ruby Documentation](http://www.ruby-doc.org/stdlib-2.0/libdoc/matrix/rdoc/Matrix.html) and this [helpful post](http://rubylearning.com/blog/2013/04/04/ruby-matrix-the-forgotten-library/) from RubyLearning Blog.








