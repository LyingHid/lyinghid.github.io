---
layout: post
title:  "Factorization Machines"
date:   2019-08-24 +0800
categories: recommend-system factorization-machines support-vector-machines
---

After days of work, I finally figure out how Factorization Machines (FM) works, and write down this note to explain it with a topdown approach.

# Factorization Machines

FM is introduced by [Rendle](https://www.csie.ntu.edu.tw/~b97053/paper/Rendle2010FM.pdf), the core is the following formular:

$$ \hat{y} \left( \boldsymbol{x} \right) = w_0 + \sum_{i=1}^n w_i x_i + \sum_{i=1}^n \sum_{j=i+1}^n w_{ij} x_ix_j $$

It looks complex. So let's breakdown the formular with an [example](https://zhouyonglong.wordpress.com/2018/05/01/factorization-machines/).

Assume we are interest to know how likely our friend Bob will go shopping in the next few days. Intuitively, the closer Bob lives beside a supermarket, the more often and likely he will go shopping. And if Thanks Giving day is coming and he lives in america, he will likely go shopping.

In the above example, we use location and time to predict Bob's behavior. When we measure the distance between Bob's home and the supermarket, location is used alone. When we figure out it's Thanks Giving day and the fact that he lives in america, time and location is used as a combination.

Time and location are features. They are $x_i$ and $x_j$ in the formular, and $x_ix_j$ represents the combination of two features. The $w$ is the weight of each feature/each combination of features, it decides how strong a feature will contribute to the predict result $y$. As time and location are natural features, transformation is needed to make them machine friendly. One hot encoding is one of the transformations.

The name of FM comes from $w_{ij}$ in the formular.

The $w_{ij}$ for all $i$ and all $j$ form a matrix $W$, $w_{ij}$ is at the i-th row and j-th column of $W$. In FM, $W$ is factored to two another matrices, just like integer factorization, for example $24 = 6 \times 4$.

# Matrix Factorization

Why do we factor the matrix? [Netflix](https://www.youtube.com/watch?v=ZspR5PZemcs) gives us an example.

Consider we have 4 users: Alice, Bob, Carlos, Dana, and 5 movies: Mall-Cop, Twister, Jaws, Observe and Report, sharknado, abbr to M1~M5. Their rating to movies are shown below:

|--:|----|----|----|----|----|
|   | M1 | M2 | M3 | M4 | M5 |
| A | 3  | 1  | 1  | 3  | 1  |
| B | 1  | 2  | 4  | 1  | 3  |
| C | 3  | 1  | 1  | 3  | 1  |
| D | 4  | 3  | 5  | 4  | 4  |

Given such a rating table, how can we predict the rating of a up coming movie?

Well, we can think that users and movies have features, if the features of one user and one movie come close to each other, then the rating will be high.

I'll pick two features for the users and movies. For the users, the two features are whether they like comedy or action movies. For the movies. the two features are how much they belong to comedy or action movies.

Now, the rating table can be factored to two smaller matrices, one for users and another for movies.

The user matrix

|   | comedy | action |
|--:|:------:|:------:|
| A |   1    |   0    |
| B |   0    |   1    |
| C |   1    |   0    |
| D |   1    |   1    |

The movie matrix

|    | comedy | action |
|---:|:------:|:------:|
| M1 |   3    |   1    |
| M2 |   1    |   2    |
| M3 |   1    |   4    |
| M4 |   3    |   1    |
| M5 |   1    |   3    |

Each row of the factored matrices can be treated as a vector. To measure how close two vectors are, we can use cosine distance/dot product.

Let's exam the user matrix and movie matrix that are indeed the factorization of the original matrix. For Alice and M3, the rating is 1, calculated by picking the first row of user matrix and the third row of movie matrix, $\langle 1, 0 \rangle \cdot \langle 1, 4 \rangle = 1$. Done.

So, no matter how many new movies we encounter, we can dot product their feature vectors with uesrs', and get the predicted ratings. Something important is happening, we can predict all other movies by just knowing 4 movies by matrix factorization. Users and movies get connected by the features via matrix factorization.

OK, in Netflix example, we used matrix factorization directly to produce predictions. In FM, matrix factorization is used to produce weights of feature combinations, and then the weights and features are used to produce predctions. By using matrix factorization, we extract the features of weights of feature combinations in FM.

After introducing FM to us, Rendle compared it with Support Vector Machines (SVM). To understand the comparisions, I have to dig into [the details of SVM](https://www.youtube.com/watch?v=_PwhiWxHK8o).

# Support Vector Machines

{% comment %}

import math
from matplotlib import pyplot as plt

fig = plt.figure(figsize=(4, 4))
axes = fig.gca()
axes.set_xlim([0, 5])
axes.set_ylim([0, 5])

root2 = math.sqrt(2) / 2
axes.scatter([1, 2], [2, 1], s=100, color='g', marker='$-$')
axes.scatter([2.5, 4], [2.5, 4], s=100, color='r', marker='+')

axes.plot([2, 2], [0, 5], 'y')
axes.plot([4, 0], [0, 4], 'b')

plt.savefig('graph.svg')

{% endcomment %}

Consider the following figure, we have positive and negetive samples, and we want to draw a line to seperate them. The blue line and yellow line both do the jobs. But the blue one does better as it has larger margins between positive and negetive samples. SVM is used to find such optimized lines.

<!-- Created with matplotlib (https://matplotlib.org/) -->
<svg height="288pt" version="1.1" viewBox="0 0 288 288" width="288pt" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
 <defs>
  <style type="text/css">
*{stroke-linecap:butt;stroke-linejoin:round;}
  </style>
 </defs>
 <g id="figure_1">
  <g id="patch_1">
   <path d="M 0 288 
L 288 288 
L 288 0 
L 0 0 
z
" style="fill:#ffffff;"/>
  </g>
  <g id="axes_1">
   <g id="patch_2">
    <path d="M 36 256.32 
L 259.2 256.32 
L 259.2 34.56 
L 36 34.56 
z
" style="fill:#ffffff;"/>
   </g>
   <g id="PathCollection_1">
    <defs>
     <path d="M -3.552519 -2.425278 
L 5 -2.425278 
L 5 -1.291631 
L -3.552519 -1.291631 
z
" id="mad828fcd99" style="stroke:#008000;"/>
    </defs>
    <g clip-path="url(#p6086f58c3c)">
     <use style="fill:#008000;stroke:#008000;" x="80.64" xlink:href="#mad828fcd99" y="167.616"/>
     <use style="fill:#008000;stroke:#008000;" x="125.28" xlink:href="#mad828fcd99" y="211.968"/>
    </g>
   </g>
   <g id="PathCollection_2">
    <defs>
     <path d="M -5 0 
L 5 0 
M 0 5 
L 0 -5 
" id="m1a39e526b7" style="stroke:#ff0000;stroke-width:1.5;"/>
    </defs>
    <g clip-path="url(#p6086f58c3c)">
     <use style="fill:#ff0000;stroke:#ff0000;stroke-width:1.5;" x="147.6" xlink:href="#m1a39e526b7" y="145.44"/>
     <use style="fill:#ff0000;stroke:#ff0000;stroke-width:1.5;" x="214.56" xlink:href="#m1a39e526b7" y="78.912"/>
    </g>
   </g>
   <g id="matplotlib.axis_1">
    <g id="xtick_1">
     <g id="line2d_1">
      <defs>
       <path d="M 0 0 
L 0 3.5 
" id="md87ec0abb5" style="stroke:#000000;stroke-width:0.8;"/>
      </defs>
      <g>
       <use style="stroke:#000000;stroke-width:0.8;" x="36" xlink:href="#md87ec0abb5" y="256.32"/>
      </g>
     </g>
     <g id="text_1">
      <!-- 0 -->
      <defs>
       <path d="M 31.78125 66.40625 
Q 24.171875 66.40625 20.328125 58.90625 
Q 16.5 51.421875 16.5 36.375 
Q 16.5 21.390625 20.328125 13.890625 
Q 24.171875 6.390625 31.78125 6.390625 
Q 39.453125 6.390625 43.28125 13.890625 
Q 47.125 21.390625 47.125 36.375 
Q 47.125 51.421875 43.28125 58.90625 
Q 39.453125 66.40625 31.78125 66.40625 
z
M 31.78125 74.21875 
Q 44.046875 74.21875 50.515625 64.515625 
Q 56.984375 54.828125 56.984375 36.375 
Q 56.984375 17.96875 50.515625 8.265625 
Q 44.046875 -1.421875 31.78125 -1.421875 
Q 19.53125 -1.421875 13.0625 8.265625 
Q 6.59375 17.96875 6.59375 36.375 
Q 6.59375 54.828125 13.0625 64.515625 
Q 19.53125 74.21875 31.78125 74.21875 
z
" id="DejaVuSans-48"/>
      </defs>
      <g transform="translate(32.81875 270.918437)scale(0.1 -0.1)">
       <use xlink:href="#DejaVuSans-48"/>
      </g>
     </g>
    </g>
    <g id="xtick_2">
     <g id="line2d_2">
      <g>
       <use style="stroke:#000000;stroke-width:0.8;" x="80.64" xlink:href="#md87ec0abb5" y="256.32"/>
      </g>
     </g>
     <g id="text_2">
      <!-- 1 -->
      <defs>
       <path d="M 12.40625 8.296875 
L 28.515625 8.296875 
L 28.515625 63.921875 
L 10.984375 60.40625 
L 10.984375 69.390625 
L 28.421875 72.90625 
L 38.28125 72.90625 
L 38.28125 8.296875 
L 54.390625 8.296875 
L 54.390625 0 
L 12.40625 0 
z
" id="DejaVuSans-49"/>
      </defs>
      <g transform="translate(77.45875 270.918437)scale(0.1 -0.1)">
       <use xlink:href="#DejaVuSans-49"/>
      </g>
     </g>
    </g>
    <g id="xtick_3">
     <g id="line2d_3">
      <g>
       <use style="stroke:#000000;stroke-width:0.8;" x="125.28" xlink:href="#md87ec0abb5" y="256.32"/>
      </g>
     </g>
     <g id="text_3">
      <!-- 2 -->
      <defs>
       <path d="M 19.1875 8.296875 
L 53.609375 8.296875 
L 53.609375 0 
L 7.328125 0 
L 7.328125 8.296875 
Q 12.9375 14.109375 22.625 23.890625 
Q 32.328125 33.6875 34.8125 36.53125 
Q 39.546875 41.84375 41.421875 45.53125 
Q 43.3125 49.21875 43.3125 52.78125 
Q 43.3125 58.59375 39.234375 62.25 
Q 35.15625 65.921875 28.609375 65.921875 
Q 23.96875 65.921875 18.8125 64.3125 
Q 13.671875 62.703125 7.8125 59.421875 
L 7.8125 69.390625 
Q 13.765625 71.78125 18.9375 73 
Q 24.125 74.21875 28.421875 74.21875 
Q 39.75 74.21875 46.484375 68.546875 
Q 53.21875 62.890625 53.21875 53.421875 
Q 53.21875 48.921875 51.53125 44.890625 
Q 49.859375 40.875 45.40625 35.40625 
Q 44.1875 33.984375 37.640625 27.21875 
Q 31.109375 20.453125 19.1875 8.296875 
z
" id="DejaVuSans-50"/>
      </defs>
      <g transform="translate(122.09875 270.918437)scale(0.1 -0.1)">
       <use xlink:href="#DejaVuSans-50"/>
      </g>
     </g>
    </g>
    <g id="xtick_4">
     <g id="line2d_4">
      <g>
       <use style="stroke:#000000;stroke-width:0.8;" x="169.92" xlink:href="#md87ec0abb5" y="256.32"/>
      </g>
     </g>
     <g id="text_4">
      <!-- 3 -->
      <defs>
       <path d="M 40.578125 39.3125 
Q 47.65625 37.796875 51.625 33 
Q 55.609375 28.21875 55.609375 21.1875 
Q 55.609375 10.40625 48.1875 4.484375 
Q 40.765625 -1.421875 27.09375 -1.421875 
Q 22.515625 -1.421875 17.65625 -0.515625 
Q 12.796875 0.390625 7.625 2.203125 
L 7.625 11.71875 
Q 11.71875 9.328125 16.59375 8.109375 
Q 21.484375 6.890625 26.8125 6.890625 
Q 36.078125 6.890625 40.9375 10.546875 
Q 45.796875 14.203125 45.796875 21.1875 
Q 45.796875 27.640625 41.28125 31.265625 
Q 36.765625 34.90625 28.71875 34.90625 
L 20.21875 34.90625 
L 20.21875 43.015625 
L 29.109375 43.015625 
Q 36.375 43.015625 40.234375 45.921875 
Q 44.09375 48.828125 44.09375 54.296875 
Q 44.09375 59.90625 40.109375 62.90625 
Q 36.140625 65.921875 28.71875 65.921875 
Q 24.65625 65.921875 20.015625 65.03125 
Q 15.375 64.15625 9.8125 62.3125 
L 9.8125 71.09375 
Q 15.4375 72.65625 20.34375 73.4375 
Q 25.25 74.21875 29.59375 74.21875 
Q 40.828125 74.21875 47.359375 69.109375 
Q 53.90625 64.015625 53.90625 55.328125 
Q 53.90625 49.265625 50.4375 45.09375 
Q 46.96875 40.921875 40.578125 39.3125 
z
" id="DejaVuSans-51"/>
      </defs>
      <g transform="translate(166.73875 270.918437)scale(0.1 -0.1)">
       <use xlink:href="#DejaVuSans-51"/>
      </g>
     </g>
    </g>
    <g id="xtick_5">
     <g id="line2d_5">
      <g>
       <use style="stroke:#000000;stroke-width:0.8;" x="214.56" xlink:href="#md87ec0abb5" y="256.32"/>
      </g>
     </g>
     <g id="text_5">
      <!-- 4 -->
      <defs>
       <path d="M 37.796875 64.3125 
L 12.890625 25.390625 
L 37.796875 25.390625 
z
M 35.203125 72.90625 
L 47.609375 72.90625 
L 47.609375 25.390625 
L 58.015625 25.390625 
L 58.015625 17.1875 
L 47.609375 17.1875 
L 47.609375 0 
L 37.796875 0 
L 37.796875 17.1875 
L 4.890625 17.1875 
L 4.890625 26.703125 
z
" id="DejaVuSans-52"/>
      </defs>
      <g transform="translate(211.37875 270.918437)scale(0.1 -0.1)">
       <use xlink:href="#DejaVuSans-52"/>
      </g>
     </g>
    </g>
    <g id="xtick_6">
     <g id="line2d_6">
      <g>
       <use style="stroke:#000000;stroke-width:0.8;" x="259.2" xlink:href="#md87ec0abb5" y="256.32"/>
      </g>
     </g>
     <g id="text_6">
      <!-- 5 -->
      <defs>
       <path d="M 10.796875 72.90625 
L 49.515625 72.90625 
L 49.515625 64.59375 
L 19.828125 64.59375 
L 19.828125 46.734375 
Q 21.96875 47.46875 24.109375 47.828125 
Q 26.265625 48.1875 28.421875 48.1875 
Q 40.625 48.1875 47.75 41.5 
Q 54.890625 34.8125 54.890625 23.390625 
Q 54.890625 11.625 47.5625 5.09375 
Q 40.234375 -1.421875 26.90625 -1.421875 
Q 22.3125 -1.421875 17.546875 -0.640625 
Q 12.796875 0.140625 7.71875 1.703125 
L 7.71875 11.625 
Q 12.109375 9.234375 16.796875 8.0625 
Q 21.484375 6.890625 26.703125 6.890625 
Q 35.15625 6.890625 40.078125 11.328125 
Q 45.015625 15.765625 45.015625 23.390625 
Q 45.015625 31 40.078125 35.4375 
Q 35.15625 39.890625 26.703125 39.890625 
Q 22.75 39.890625 18.8125 39.015625 
Q 14.890625 38.140625 10.796875 36.28125 
z
" id="DejaVuSans-53"/>
      </defs>
      <g transform="translate(256.01875 270.918437)scale(0.1 -0.1)">
       <use xlink:href="#DejaVuSans-53"/>
      </g>
     </g>
    </g>
   </g>
   <g id="matplotlib.axis_2">
    <g id="ytick_1">
     <g id="line2d_7">
      <defs>
       <path d="M 0 0 
L -3.5 0 
" id="mb8c7d1ba30" style="stroke:#000000;stroke-width:0.8;"/>
      </defs>
      <g>
       <use style="stroke:#000000;stroke-width:0.8;" x="36" xlink:href="#mb8c7d1ba30" y="256.32"/>
      </g>
     </g>
     <g id="text_7">
      <!-- 0 -->
      <g transform="translate(22.6375 260.119219)scale(0.1 -0.1)">
       <use xlink:href="#DejaVuSans-48"/>
      </g>
     </g>
    </g>
    <g id="ytick_2">
     <g id="line2d_8">
      <g>
       <use style="stroke:#000000;stroke-width:0.8;" x="36" xlink:href="#mb8c7d1ba30" y="211.968"/>
      </g>
     </g>
     <g id="text_8">
      <!-- 1 -->
      <g transform="translate(22.6375 215.767219)scale(0.1 -0.1)">
       <use xlink:href="#DejaVuSans-49"/>
      </g>
     </g>
    </g>
    <g id="ytick_3">
     <g id="line2d_9">
      <g>
       <use style="stroke:#000000;stroke-width:0.8;" x="36" xlink:href="#mb8c7d1ba30" y="167.616"/>
      </g>
     </g>
     <g id="text_9">
      <!-- 2 -->
      <g transform="translate(22.6375 171.415219)scale(0.1 -0.1)">
       <use xlink:href="#DejaVuSans-50"/>
      </g>
     </g>
    </g>
    <g id="ytick_4">
     <g id="line2d_10">
      <g>
       <use style="stroke:#000000;stroke-width:0.8;" x="36" xlink:href="#mb8c7d1ba30" y="123.264"/>
      </g>
     </g>
     <g id="text_10">
      <!-- 3 -->
      <g transform="translate(22.6375 127.063219)scale(0.1 -0.1)">
       <use xlink:href="#DejaVuSans-51"/>
      </g>
     </g>
    </g>
    <g id="ytick_5">
     <g id="line2d_11">
      <g>
       <use style="stroke:#000000;stroke-width:0.8;" x="36" xlink:href="#mb8c7d1ba30" y="78.912"/>
      </g>
     </g>
     <g id="text_11">
      <!-- 4 -->
      <g transform="translate(22.6375 82.711219)scale(0.1 -0.1)">
       <use xlink:href="#DejaVuSans-52"/>
      </g>
     </g>
    </g>
    <g id="ytick_6">
     <g id="line2d_12">
      <g>
       <use style="stroke:#000000;stroke-width:0.8;" x="36" xlink:href="#mb8c7d1ba30" y="34.56"/>
      </g>
     </g>
     <g id="text_12">
      <!-- 5 -->
      <g transform="translate(22.6375 38.359219)scale(0.1 -0.1)">
       <use xlink:href="#DejaVuSans-53"/>
      </g>
     </g>
    </g>
   </g>
   <g id="line2d_13">
    <path clip-path="url(#p6086f58c3c)" d="M 125.28 256.32 
L 125.28 34.56 
" style="fill:none;stroke:#bfbf00;stroke-linecap:square;stroke-width:1.5;"/>
   </g>
   <g id="line2d_14">
    <path clip-path="url(#p6086f58c3c)" d="M 214.56 256.32 
L 36 78.912 
" style="fill:none;stroke:#0000ff;stroke-linecap:square;stroke-width:1.5;"/>
   </g>
   <g id="patch_3">
    <path d="M 36 256.32 
L 36 34.56 
" style="fill:none;stroke:#000000;stroke-linecap:square;stroke-linejoin:miter;stroke-width:0.8;"/>
   </g>
   <g id="patch_4">
    <path d="M 259.2 256.32 
L 259.2 34.56 
" style="fill:none;stroke:#000000;stroke-linecap:square;stroke-linejoin:miter;stroke-width:0.8;"/>
   </g>
   <g id="patch_5">
    <path d="M 36 256.32 
L 259.2 256.32 
" style="fill:none;stroke:#000000;stroke-linecap:square;stroke-linejoin:miter;stroke-width:0.8;"/>
   </g>
   <g id="patch_6">
    <path d="M 36 34.56 
L 259.2 34.56 
" style="fill:none;stroke:#000000;stroke-linecap:square;stroke-linejoin:miter;stroke-width:0.8;"/>
   </g>
  </g>
 </g>
 <defs>
  <clipPath id="p6086f58c3c">
   <rect height="221.76" width="223.2" x="36" y="34.56"/>
  </clipPath>
 </defs>
</svg>

When the optimized line is found, we will use it to classify an unknown dot $\overline{u}$ to positive ones or negetive ones. To do this, $\overline{u}$ is projected to $\overline{w}$ which is perpendicular to the optimized lines. In another word, $\overline{u}$ belongs to positive ones if the dot product $\overline{w} * \overline{u} \geq c$, $c$ is some constant.

{% comment %}

import math
from matplotlib import pyplot as plt

fig = plt.figure(figsize=(4, 4))
axes = fig.gca()
axes.set_xlim([0, 5])
axes.arrow(0, 0, 1, 1, label='123')
axes.set_ylim([0, 5])

root2 = math.sqrt(2) / 2
axes.scatter([1, 2], [2, 1], s=100, color='g', marker='$-$')
axes.scatter([2.5, 4], [2.5, 4], s=100, color='r', marker='+')

axes.arrow(0, 0, 1, 1, head_width=0.1, fill=False)
axes.scatter([1], [1.25], c='k', s=100, marker='$\overline{\omega}$')
axes.arrow(0, 0, 1.5, 3, head_width=0.1, fill=False)
axes.scatter([1.5], [3.25], c='k', s=100, marker='$\overline{u}$')
axes.plot([4, 0], [0, 4], 'b')
axes.plot([5, 0], [0, 5], 'b--')
axes.plot([3, 0], [0, 3], 'b--')

plt.savefig('graph.svg')

{% endcomment %}

<!-- Created with matplotlib (https://matplotlib.org/) -->
<svg height="288pt" version="1.1" viewBox="0 0 288 288" width="288pt" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
 <defs>
  <style type="text/css">
*{stroke-linecap:butt;stroke-linejoin:round;}
  </style>
 </defs>
 <g id="figure_1">
  <g id="patch_1">
   <path d="M 0 288 
L 288 288 
L 288 0 
L 0 0 
z
" style="fill:#ffffff;"/>
  </g>
  <g id="axes_1">
   <g id="patch_2">
    <path d="M 36 256.32 
L 259.2 256.32 
L 259.2 34.56 
L 36 34.56 
z
" style="fill:#ffffff;"/>
   </g>
   <g id="PathCollection_1">
    <defs>
     <path d="M -3.552519 -2.425278 
L 5 -2.425278 
L 5 -1.291631 
L -3.552519 -1.291631 
z
" id="mbc39e2e27e" style="stroke:#008000;"/>
    </defs>
    <g clip-path="url(#p457abc24a6)">
     <use style="fill:#008000;stroke:#008000;" x="80.64" xlink:href="#mbc39e2e27e" y="167.616"/>
     <use style="fill:#008000;stroke:#008000;" x="125.28" xlink:href="#mbc39e2e27e" y="211.968"/>
    </g>
   </g>
   <g id="PathCollection_2">
    <defs>
     <path d="M -5 0 
L 5 0 
M 0 5 
L 0 -5 
" id="mae2194e8e8" style="stroke:#ff0000;stroke-width:1.5;"/>
    </defs>
    <g clip-path="url(#p457abc24a6)">
     <use style="fill:#ff0000;stroke:#ff0000;stroke-width:1.5;" x="147.6" xlink:href="#mae2194e8e8" y="145.44"/>
     <use style="fill:#ff0000;stroke:#ff0000;stroke-width:1.5;" x="214.56" xlink:href="#mae2194e8e8" y="78.912"/>
    </g>
   </g>
   <g id="PathCollection_3">
    <defs>
     <path d="M -2.445598 4.422157 
Q -4.865656 4.422157 -4.190204 0.957201 
Q -3.927114 -0.412362 -2.656443 -2.278251 
L -1.501458 -2.278251 
Q -2.70309 -0.412362 -2.969913 0.987055 
Q -3.471837 3.511603 -2.165714 3.511603 
Q -0.958484 3.511603 -0.376327 0.375044 
L 0.614461 0.375044 
Q -0.019942 3.528397 1.187289 3.511603 
Q 2.487813 3.500408 2.976676 0.987055 
Q 3.245364 -0.412362 2.778892 -2.278251 
L 3.933878 -2.278251 
Q 4.469388 -0.412362 4.206297 0.957201 
Q 3.542041 4.427755 1.116385 4.422157 
Q -0.469621 4.41656 -0.329679 2.677551 
Q -0.906239 4.422157 -2.445598 4.422157 
z
M -5 -3.681399 
L -5 -4.427755 
L 5 -4.427755 
L 5 -3.681399 
L -5 -3.681399 
z
" id="m4e7b8281fb" style="stroke:#000000;"/>
    </defs>
    <g clip-path="url(#p457abc24a6)">
     <use style="stroke:#000000;" x="80.64" xlink:href="#m4e7b8281fb" y="200.88"/>
    </g>
   </g>
   <g id="PathCollection_4">
    <defs>
     <path d="M -3.373656 1.88172 
L -2.509224 -2.571157 
L -1.290586 -2.571157 
L -2.155018 1.837445 
Q -2.220377 2.160025 -2.249895 2.389838 
Q -2.279412 2.61965 -2.279412 2.771453 
Q -2.279412 3.332279 -1.937856 3.640101 
Q -1.594191 3.945815 -0.968005 3.945815 
Q 0.006062 3.945815 0.716582 3.288003 
Q 1.429211 2.628083 1.646374 1.512756 
L 2.458096 -2.571157 
L 3.670409 -2.571157 
L 2.240934 4.808138 
L 1.028621 4.808138 
L 1.271084 3.648535 
Q 0.75875 4.293696 0.050337 4.647902 
Q -0.658075 5 -1.455039 5 
Q -2.424889 5 -2.964632 4.468691 
Q -3.504375 3.937381 -3.504375 2.988615 
Q -3.504375 2.792536 -3.472749 2.495256 
Q -3.439015 2.197976 -3.373656 1.88172 
z
M -4.276038 -4.156652 
L -4.276038 -5 
L 4.276038 -5 
L 4.276038 -4.156652 
L -4.276038 -4.156652 
z
" id="m2f26e5814d" style="stroke:#000000;"/>
    </defs>
    <g clip-path="url(#p457abc24a6)">
     <use style="stroke:#000000;" x="102.96" xlink:href="#m2f26e5814d" y="112.176"/>
    </g>
   </g>
   <g id="patch_3">
    <path clip-path="url(#p457abc24a6)" d="M 80.782044 211.826873 
L 80.687348 212.015042 
L 80.655783 211.983681 
L 36.015783 256.335681 
L 35.984217 256.304319 
L 80.624217 211.952319 
L 80.592652 211.920958 
z
" style="fill:#1f77b4;stroke:#000000;stroke-linejoin:miter;"/>
   </g>
   <g id="patch_4">
    <path clip-path="url(#p457abc24a6)" d="M 85.374787 207.26376 
L 82.218262 213.53608 
L 80.655783 211.983681 
L 36.015783 256.335681 
L 35.984217 256.304319 
L 80.624217 211.952319 
L 79.061738 210.39992 
z
" style="fill:none;stroke:#000000;stroke-linejoin:miter;"/>
   </g>
   <g id="patch_5">
    <path clip-path="url(#p457abc24a6)" d="M 105.954542 117.313555 
L 104.956361 124.255741 
L 102.979964 123.273917 
L 36.019964 256.329917 
L 35.980036 256.310083 
L 102.940036 123.254083 
L 100.963639 122.272259 
z
" style="fill:none;stroke:#000000;stroke-linejoin:miter;"/>
   </g>
   <g id="matplotlib.axis_1">
    <g id="xtick_1">
     <g id="line2d_1">
      <defs>
       <path d="M 0 0 
L 0 3.5 
" id="m42c5734af3" style="stroke:#000000;stroke-width:0.8;"/>
      </defs>
      <g>
       <use style="stroke:#000000;stroke-width:0.8;" x="36" xlink:href="#m42c5734af3" y="256.32"/>
      </g>
     </g>
     <g id="text_1">
      <!-- 0 -->
      <defs>
       <path d="M 31.78125 66.40625 
Q 24.171875 66.40625 20.328125 58.90625 
Q 16.5 51.421875 16.5 36.375 
Q 16.5 21.390625 20.328125 13.890625 
Q 24.171875 6.390625 31.78125 6.390625 
Q 39.453125 6.390625 43.28125 13.890625 
Q 47.125 21.390625 47.125 36.375 
Q 47.125 51.421875 43.28125 58.90625 
Q 39.453125 66.40625 31.78125 66.40625 
z
M 31.78125 74.21875 
Q 44.046875 74.21875 50.515625 64.515625 
Q 56.984375 54.828125 56.984375 36.375 
Q 56.984375 17.96875 50.515625 8.265625 
Q 44.046875 -1.421875 31.78125 -1.421875 
Q 19.53125 -1.421875 13.0625 8.265625 
Q 6.59375 17.96875 6.59375 36.375 
Q 6.59375 54.828125 13.0625 64.515625 
Q 19.53125 74.21875 31.78125 74.21875 
z
" id="DejaVuSans-48"/>
      </defs>
      <g transform="translate(32.81875 270.918437)scale(0.1 -0.1)">
       <use xlink:href="#DejaVuSans-48"/>
      </g>
     </g>
    </g>
    <g id="xtick_2">
     <g id="line2d_2">
      <g>
       <use style="stroke:#000000;stroke-width:0.8;" x="80.64" xlink:href="#m42c5734af3" y="256.32"/>
      </g>
     </g>
     <g id="text_2">
      <!-- 1 -->
      <defs>
       <path d="M 12.40625 8.296875 
L 28.515625 8.296875 
L 28.515625 63.921875 
L 10.984375 60.40625 
L 10.984375 69.390625 
L 28.421875 72.90625 
L 38.28125 72.90625 
L 38.28125 8.296875 
L 54.390625 8.296875 
L 54.390625 0 
L 12.40625 0 
z
" id="DejaVuSans-49"/>
      </defs>
      <g transform="translate(77.45875 270.918437)scale(0.1 -0.1)">
       <use xlink:href="#DejaVuSans-49"/>
      </g>
     </g>
    </g>
    <g id="xtick_3">
     <g id="line2d_3">
      <g>
       <use style="stroke:#000000;stroke-width:0.8;" x="125.28" xlink:href="#m42c5734af3" y="256.32"/>
      </g>
     </g>
     <g id="text_3">
      <!-- 2 -->
      <defs>
       <path d="M 19.1875 8.296875 
L 53.609375 8.296875 
L 53.609375 0 
L 7.328125 0 
L 7.328125 8.296875 
Q 12.9375 14.109375 22.625 23.890625 
Q 32.328125 33.6875 34.8125 36.53125 
Q 39.546875 41.84375 41.421875 45.53125 
Q 43.3125 49.21875 43.3125 52.78125 
Q 43.3125 58.59375 39.234375 62.25 
Q 35.15625 65.921875 28.609375 65.921875 
Q 23.96875 65.921875 18.8125 64.3125 
Q 13.671875 62.703125 7.8125 59.421875 
L 7.8125 69.390625 
Q 13.765625 71.78125 18.9375 73 
Q 24.125 74.21875 28.421875 74.21875 
Q 39.75 74.21875 46.484375 68.546875 
Q 53.21875 62.890625 53.21875 53.421875 
Q 53.21875 48.921875 51.53125 44.890625 
Q 49.859375 40.875 45.40625 35.40625 
Q 44.1875 33.984375 37.640625 27.21875 
Q 31.109375 20.453125 19.1875 8.296875 
z
" id="DejaVuSans-50"/>
      </defs>
      <g transform="translate(122.09875 270.918437)scale(0.1 -0.1)">
       <use xlink:href="#DejaVuSans-50"/>
      </g>
     </g>
    </g>
    <g id="xtick_4">
     <g id="line2d_4">
      <g>
       <use style="stroke:#000000;stroke-width:0.8;" x="169.92" xlink:href="#m42c5734af3" y="256.32"/>
      </g>
     </g>
     <g id="text_4">
      <!-- 3 -->
      <defs>
       <path d="M 40.578125 39.3125 
Q 47.65625 37.796875 51.625 33 
Q 55.609375 28.21875 55.609375 21.1875 
Q 55.609375 10.40625 48.1875 4.484375 
Q 40.765625 -1.421875 27.09375 -1.421875 
Q 22.515625 -1.421875 17.65625 -0.515625 
Q 12.796875 0.390625 7.625 2.203125 
L 7.625 11.71875 
Q 11.71875 9.328125 16.59375 8.109375 
Q 21.484375 6.890625 26.8125 6.890625 
Q 36.078125 6.890625 40.9375 10.546875 
Q 45.796875 14.203125 45.796875 21.1875 
Q 45.796875 27.640625 41.28125 31.265625 
Q 36.765625 34.90625 28.71875 34.90625 
L 20.21875 34.90625 
L 20.21875 43.015625 
L 29.109375 43.015625 
Q 36.375 43.015625 40.234375 45.921875 
Q 44.09375 48.828125 44.09375 54.296875 
Q 44.09375 59.90625 40.109375 62.90625 
Q 36.140625 65.921875 28.71875 65.921875 
Q 24.65625 65.921875 20.015625 65.03125 
Q 15.375 64.15625 9.8125 62.3125 
L 9.8125 71.09375 
Q 15.4375 72.65625 20.34375 73.4375 
Q 25.25 74.21875 29.59375 74.21875 
Q 40.828125 74.21875 47.359375 69.109375 
Q 53.90625 64.015625 53.90625 55.328125 
Q 53.90625 49.265625 50.4375 45.09375 
Q 46.96875 40.921875 40.578125 39.3125 
z
" id="DejaVuSans-51"/>
      </defs>
      <g transform="translate(166.73875 270.918437)scale(0.1 -0.1)">
       <use xlink:href="#DejaVuSans-51"/>
      </g>
     </g>
    </g>
    <g id="xtick_5">
     <g id="line2d_5">
      <g>
       <use style="stroke:#000000;stroke-width:0.8;" x="214.56" xlink:href="#m42c5734af3" y="256.32"/>
      </g>
     </g>
     <g id="text_5">
      <!-- 4 -->
      <defs>
       <path d="M 37.796875 64.3125 
L 12.890625 25.390625 
L 37.796875 25.390625 
z
M 35.203125 72.90625 
L 47.609375 72.90625 
L 47.609375 25.390625 
L 58.015625 25.390625 
L 58.015625 17.1875 
L 47.609375 17.1875 
L 47.609375 0 
L 37.796875 0 
L 37.796875 17.1875 
L 4.890625 17.1875 
L 4.890625 26.703125 
z
" id="DejaVuSans-52"/>
      </defs>
      <g transform="translate(211.37875 270.918437)scale(0.1 -0.1)">
       <use xlink:href="#DejaVuSans-52"/>
      </g>
     </g>
    </g>
    <g id="xtick_6">
     <g id="line2d_6">
      <g>
       <use style="stroke:#000000;stroke-width:0.8;" x="259.2" xlink:href="#m42c5734af3" y="256.32"/>
      </g>
     </g>
     <g id="text_6">
      <!-- 5 -->
      <defs>
       <path d="M 10.796875 72.90625 
L 49.515625 72.90625 
L 49.515625 64.59375 
L 19.828125 64.59375 
L 19.828125 46.734375 
Q 21.96875 47.46875 24.109375 47.828125 
Q 26.265625 48.1875 28.421875 48.1875 
Q 40.625 48.1875 47.75 41.5 
Q 54.890625 34.8125 54.890625 23.390625 
Q 54.890625 11.625 47.5625 5.09375 
Q 40.234375 -1.421875 26.90625 -1.421875 
Q 22.3125 -1.421875 17.546875 -0.640625 
Q 12.796875 0.140625 7.71875 1.703125 
L 7.71875 11.625 
Q 12.109375 9.234375 16.796875 8.0625 
Q 21.484375 6.890625 26.703125 6.890625 
Q 35.15625 6.890625 40.078125 11.328125 
Q 45.015625 15.765625 45.015625 23.390625 
Q 45.015625 31 40.078125 35.4375 
Q 35.15625 39.890625 26.703125 39.890625 
Q 22.75 39.890625 18.8125 39.015625 
Q 14.890625 38.140625 10.796875 36.28125 
z
" id="DejaVuSans-53"/>
      </defs>
      <g transform="translate(256.01875 270.918437)scale(0.1 -0.1)">
       <use xlink:href="#DejaVuSans-53"/>
      </g>
     </g>
    </g>
   </g>
   <g id="matplotlib.axis_2">
    <g id="ytick_1">
     <g id="line2d_7">
      <defs>
       <path d="M 0 0 
L -3.5 0 
" id="m9204a97596" style="stroke:#000000;stroke-width:0.8;"/>
      </defs>
      <g>
       <use style="stroke:#000000;stroke-width:0.8;" x="36" xlink:href="#m9204a97596" y="256.32"/>
      </g>
     </g>
     <g id="text_7">
      <!-- 0 -->
      <g transform="translate(22.6375 260.119219)scale(0.1 -0.1)">
       <use xlink:href="#DejaVuSans-48"/>
      </g>
     </g>
    </g>
    <g id="ytick_2">
     <g id="line2d_8">
      <g>
       <use style="stroke:#000000;stroke-width:0.8;" x="36" xlink:href="#m9204a97596" y="211.968"/>
      </g>
     </g>
     <g id="text_8">
      <!-- 1 -->
      <g transform="translate(22.6375 215.767219)scale(0.1 -0.1)">
       <use xlink:href="#DejaVuSans-49"/>
      </g>
     </g>
    </g>
    <g id="ytick_3">
     <g id="line2d_9">
      <g>
       <use style="stroke:#000000;stroke-width:0.8;" x="36" xlink:href="#m9204a97596" y="167.616"/>
      </g>
     </g>
     <g id="text_9">
      <!-- 2 -->
      <g transform="translate(22.6375 171.415219)scale(0.1 -0.1)">
       <use xlink:href="#DejaVuSans-50"/>
      </g>
     </g>
    </g>
    <g id="ytick_4">
     <g id="line2d_10">
      <g>
       <use style="stroke:#000000;stroke-width:0.8;" x="36" xlink:href="#m9204a97596" y="123.264"/>
      </g>
     </g>
     <g id="text_10">
      <!-- 3 -->
      <g transform="translate(22.6375 127.063219)scale(0.1 -0.1)">
       <use xlink:href="#DejaVuSans-51"/>
      </g>
     </g>
    </g>
    <g id="ytick_5">
     <g id="line2d_11">
      <g>
       <use style="stroke:#000000;stroke-width:0.8;" x="36" xlink:href="#m9204a97596" y="78.912"/>
      </g>
     </g>
     <g id="text_11">
      <!-- 4 -->
      <g transform="translate(22.6375 82.711219)scale(0.1 -0.1)">
       <use xlink:href="#DejaVuSans-52"/>
      </g>
     </g>
    </g>
    <g id="ytick_6">
     <g id="line2d_12">
      <g>
       <use style="stroke:#000000;stroke-width:0.8;" x="36" xlink:href="#m9204a97596" y="34.56"/>
      </g>
     </g>
     <g id="text_12">
      <!-- 5 -->
      <g transform="translate(22.6375 38.359219)scale(0.1 -0.1)">
       <use xlink:href="#DejaVuSans-53"/>
      </g>
     </g>
    </g>
   </g>
   <g id="line2d_13">
    <path clip-path="url(#p457abc24a6)" d="M 214.56 256.32 
L 36 78.912 
" style="fill:none;stroke:#0000ff;stroke-linecap:square;stroke-width:1.5;"/>
   </g>
   <g id="line2d_14">
    <path clip-path="url(#p457abc24a6)" d="M 259.2 256.32 
L 36 34.56 
" style="fill:none;stroke:#0000ff;stroke-dasharray:5.55,2.4;stroke-dashoffset:0;stroke-width:1.5;"/>
   </g>
   <g id="line2d_15">
    <path clip-path="url(#p457abc24a6)" d="M 169.92 256.32 
L 36 123.264 
" style="fill:none;stroke:#0000ff;stroke-dasharray:5.55,2.4;stroke-dashoffset:0;stroke-width:1.5;"/>
   </g>
   <g id="patch_6">
    <path d="M 36 256.32 
L 36 34.56 
" style="fill:none;stroke:#000000;stroke-linecap:square;stroke-linejoin:miter;stroke-width:0.8;"/>
   </g>
   <g id="patch_7">
    <path d="M 259.2 256.32 
L 259.2 34.56 
" style="fill:none;stroke:#000000;stroke-linecap:square;stroke-linejoin:miter;stroke-width:0.8;"/>
   </g>
   <g id="patch_8">
    <path d="M 36 256.32 
L 259.2 256.32 
" style="fill:none;stroke:#000000;stroke-linecap:square;stroke-linejoin:miter;stroke-width:0.8;"/>
   </g>
   <g id="patch_9">
    <path d="M 36 34.56 
L 259.2 34.56 
" style="fill:none;stroke:#000000;stroke-linecap:square;stroke-linejoin:miter;stroke-width:0.8;"/>
   </g>
  </g>
 </g>
 <defs>
  <clipPath id="p457abc24a6">
   <rect height="221.76" width="223.2" x="36" y="34.56"/>
  </clipPath>
 </defs>
</svg>

Now the question is how to find $\overline{w}$ given the training set.

For positive samples, $\overline{w} \cdot \overline{x}\_{+} \geq c$. We rewrite it to $\overline{w} \cdot \overline{x}_+ + b \geq 0$. As it's in training set, we are more strict and want the left side of inequality greater than 1. The same applys to negetive samples. So we get:

$$
\overline{w} \cdot \overline{x}_+ + b \geq 1 \\
\overline{w} \cdot \overline{x}_- + b \leq -1
$$

For math convience, another variable $y$ is introduced such that $y = 1$ for positive samples and $y = -1$ for negetive samples. Then the above inequality gorup can be uniformed as below:

$$
y_i \left(\overline{w} \cdot \overline{x}_i + b \right) \geq 1
$$

For the samples on the margin lines:

$$
y_i \left(\overline{w} \cdot \overline{x}_i + b \right) = 1
$$

Now we need to choose some samples to make the margin as wide as possible. To find the width, we pick on the margin one positive sample $\overline{x}\_+$ and substract it from another negetive sample $\overline{x}_-$, then project the result to a unit vector of $\overline{w}$.

{% comment %}

import math
from matplotlib import pyplot as plt

fig = plt.figure(figsize=(4, 4))
axes = fig.gca()
axes.set_xlim([0, 5])
axes.arrow(0, 0, 1, 1, label='123')
axes.set_ylim([0, 5])

root2 = math.sqrt(2) / 2
axes.scatter([1, 2], [2, 1], s=100, color='g', marker='$-$')
axes.scatter([2.5, 4], [2.5, 4], s=100, color='r', marker='+')

axes.arrow(0, 0, 1, 1, head_width=0.1, fill=False)
axes.scatter([1], [1.25], c='k', s=100, marker='$\overline{\omega}$')
axes.arrow(0, 0, 1, 2, head_width=0.1, fill=False)
axes.scatter([1], [2.25], c='g', s=100, marker='$\overline{x}_-$')
axes.arrow(0, 0, 2.5, 2.5, head_width=0.1, fill=False)
axes.scatter([2.5], [2.75], c='r', s=100, marker='$\overline{x}_+$')
axes.arrow(1, 2, 1.5, 0.5, head_width=0.1, fill=False)
axes.plot([4, 0], [0, 4], 'b')
axes.plot([5, 0], [0, 5], 'b--')
axes.plot([3, 0], [0, 3], 'b--')

plt.savefig('graph.svg')

{% endcomment %}

<!-- Created with matplotlib (https://matplotlib.org/) -->
<svg height="288pt" version="1.1" viewBox="0 0 288 288" width="288pt" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
 <defs>
  <style type="text/css">
*{stroke-linecap:butt;stroke-linejoin:round;}
  </style>
 </defs>
 <g id="figure_1">
  <g id="patch_1">
   <path d="M 0 288 
L 288 288 
L 288 0 
L 0 0 
z
" style="fill:#ffffff;"/>
  </g>
  <g id="axes_1">
   <g id="patch_2">
    <path d="M 36 256.32 
L 259.2 256.32 
L 259.2 34.56 
L 36 34.56 
z
" style="fill:#ffffff;"/>
   </g>
   <g id="PathCollection_1">
    <defs>
     <path d="M -3.552519 -2.425278 
L 5 -2.425278 
L 5 -1.291631 
L -3.552519 -1.291631 
z
" id="m84968927c8" style="stroke:#008000;"/>
    </defs>
    <g clip-path="url(#pe22274eb01)">
     <use style="fill:#008000;stroke:#008000;" x="80.64" xlink:href="#m84968927c8" y="167.616"/>
     <use style="fill:#008000;stroke:#008000;" x="125.28" xlink:href="#m84968927c8" y="211.968"/>
    </g>
   </g>
   <g id="PathCollection_2">
    <defs>
     <path d="M -5 0 
L 5 0 
M 0 5 
L 0 -5 
" id="mdc50240cf8" style="stroke:#ff0000;stroke-width:1.5;"/>
    </defs>
    <g clip-path="url(#pe22274eb01)">
     <use style="fill:#ff0000;stroke:#ff0000;stroke-width:1.5;" x="147.6" xlink:href="#mdc50240cf8" y="145.44"/>
     <use style="fill:#ff0000;stroke:#ff0000;stroke-width:1.5;" x="214.56" xlink:href="#mdc50240cf8" y="78.912"/>
    </g>
   </g>
   <g id="PathCollection_3">
    <defs>
     <path d="M -2.445598 4.422157 
Q -4.865656 4.422157 -4.190204 0.957201 
Q -3.927114 -0.412362 -2.656443 -2.278251 
L -1.501458 -2.278251 
Q -2.70309 -0.412362 -2.969913 0.987055 
Q -3.471837 3.511603 -2.165714 3.511603 
Q -0.958484 3.511603 -0.376327 0.375044 
L 0.614461 0.375044 
Q -0.019942 3.528397 1.187289 3.511603 
Q 2.487813 3.500408 2.976676 0.987055 
Q 3.245364 -0.412362 2.778892 -2.278251 
L 3.933878 -2.278251 
Q 4.469388 -0.412362 4.206297 0.957201 
Q 3.542041 4.427755 1.116385 4.422157 
Q -0.469621 4.41656 -0.329679 2.677551 
Q -0.906239 4.422157 -2.445598 4.422157 
z
M -5 -3.681399 
L -5 -4.427755 
L 5 -4.427755 
L 5 -3.681399 
L -5 -3.681399 
z
" id="m76a9364540" style="stroke:#000000;"/>
    </defs>
    <g clip-path="url(#pe22274eb01)">
     <use style="stroke:#000000;" x="80.64" xlink:href="#m76a9364540" y="200.88"/>
    </g>
   </g>
   <g id="PathCollection_4">
    <defs>
     <path d="M -0.093287 -2.100078 
L -2.061115 0.001225 
L -0.854948 2.185796 
L -1.663141 2.185796 
L -2.569297 0.486141 
L -4.146499 2.185796 
L -5 2.185796 
L -2.89135 -0.072248 
L -4.0118 -2.100078 
L -3.204831 -2.100078 
L -2.381943 -0.549816 
L -0.946788 -2.100078 
z
M 1.566158 1.563242 
L 5 1.563242 
L 5 2.018402 
L 1.566158 2.018402 
z
M -4.796727 -3.020929 
L -4.796727 -3.510743 
L -0.158799 -3.510743 
L -0.158799 -3.020929 
L -4.796727 -3.020929 
z
" id="m41a9ceb05e" style="stroke:#008000;"/>
    </defs>
    <g clip-path="url(#pe22274eb01)">
     <use style="fill:#008000;stroke:#008000;" x="80.64" xlink:href="#m41a9ceb05e" y="156.528"/>
    </g>
   </g>
   <g id="PathCollection_5">
    <defs>
     <path d="M -0.093287 -2.100078 
L -2.061115 0.001225 
L -0.854948 2.185796 
L -1.663141 2.185796 
L -2.569297 0.486141 
L -4.146499 2.185796 
L -5 2.185796 
L -2.89135 -0.072248 
L -4.0118 -2.100078 
L -3.204831 -2.100078 
L -2.381943 -0.549816 
L -0.946788 -2.100078 
z
M 3.508516 0.070901 
L 3.508516 1.563242 
L 5 1.563242 
L 5 2.018402 
L 3.508516 2.018402 
L 3.508516 3.510743 
L 3.058499 3.510743 
L 3.058499 2.018402 
L 1.566158 2.018402 
L 1.566158 1.563242 
L 3.058499 1.563242 
L 3.058499 0.070901 
z
M -4.796727 -3.020929 
L -4.796727 -3.510743 
L -0.158799 -3.510743 
L -0.158799 -3.020929 
L -4.796727 -3.020929 
z
" id="m324ae6bf44" style="stroke:#ff0000;"/>
    </defs>
    <g clip-path="url(#pe22274eb01)">
     <use style="fill:#ff0000;stroke:#ff0000;" x="147.6" xlink:href="#m324ae6bf44" y="134.352"/>
    </g>
   </g>
   <g id="patch_3">
    <path clip-path="url(#pe22274eb01)" d="M 80.782044 211.826873 
L 80.687348 212.015042 
L 80.655783 211.983681 
L 36.015783 256.335681 
L 35.984217 256.304319 
L 80.624217 211.952319 
L 80.592652 211.920958 
z
" style="fill:#1f77b4;stroke:#000000;stroke-linejoin:miter;"/>
   </g>
   <g id="patch_4">
    <path clip-path="url(#pe22274eb01)" d="M 85.374787 207.26376 
L 82.218262 213.53608 
L 80.655783 211.983681 
L 36.015783 256.335681 
L 35.984217 256.304319 
L 80.624217 211.952319 
L 79.061738 210.39992 
z
" style="fill:none;stroke:#000000;stroke-linejoin:miter;"/>
   </g>
   <g id="patch_5">
    <path clip-path="url(#pe22274eb01)" d="M 83.634542 161.665555 
L 82.636361 168.607741 
L 80.659964 167.625917 
L 36.019964 256.329917 
L 35.980036 256.310083 
L 80.620036 167.606083 
L 78.643639 166.624259 
z
" style="fill:none;stroke:#000000;stroke-linejoin:miter;"/>
   </g>
   <g id="patch_6">
    <path clip-path="url(#pe22274eb01)" d="M 152.334787 140.73576 
L 149.178262 147.00808 
L 147.615783 145.455681 
L 36.015783 256.335681 
L 35.984217 256.304319 
L 147.584217 145.424319 
L 146.021738 143.87192 
z
" style="fill:none;stroke:#000000;stroke-linejoin:miter;"/>
   </g>
   <g id="patch_7">
    <path clip-path="url(#pe22274eb01)" d="M 153.952383 143.3362 
L 148.30582 147.5438 
L 147.607058 145.461038 
L 80.647058 167.637038 
L 80.632942 167.594962 
L 147.592942 145.418962 
L 146.89418 143.3362 
z
" style="fill:none;stroke:#000000;stroke-linejoin:miter;"/>
   </g>
   <g id="matplotlib.axis_1">
    <g id="xtick_1">
     <g id="line2d_1">
      <defs>
       <path d="M 0 0 
L 0 3.5 
" id="m2d97570a81" style="stroke:#000000;stroke-width:0.8;"/>
      </defs>
      <g>
       <use style="stroke:#000000;stroke-width:0.8;" x="36" xlink:href="#m2d97570a81" y="256.32"/>
      </g>
     </g>
     <g id="text_1">
      <!-- 0 -->
      <defs>
       <path d="M 31.78125 66.40625 
Q 24.171875 66.40625 20.328125 58.90625 
Q 16.5 51.421875 16.5 36.375 
Q 16.5 21.390625 20.328125 13.890625 
Q 24.171875 6.390625 31.78125 6.390625 
Q 39.453125 6.390625 43.28125 13.890625 
Q 47.125 21.390625 47.125 36.375 
Q 47.125 51.421875 43.28125 58.90625 
Q 39.453125 66.40625 31.78125 66.40625 
z
M 31.78125 74.21875 
Q 44.046875 74.21875 50.515625 64.515625 
Q 56.984375 54.828125 56.984375 36.375 
Q 56.984375 17.96875 50.515625 8.265625 
Q 44.046875 -1.421875 31.78125 -1.421875 
Q 19.53125 -1.421875 13.0625 8.265625 
Q 6.59375 17.96875 6.59375 36.375 
Q 6.59375 54.828125 13.0625 64.515625 
Q 19.53125 74.21875 31.78125 74.21875 
z
" id="DejaVuSans-48"/>
      </defs>
      <g transform="translate(32.81875 270.918437)scale(0.1 -0.1)">
       <use xlink:href="#DejaVuSans-48"/>
      </g>
     </g>
    </g>
    <g id="xtick_2">
     <g id="line2d_2">
      <g>
       <use style="stroke:#000000;stroke-width:0.8;" x="80.64" xlink:href="#m2d97570a81" y="256.32"/>
      </g>
     </g>
     <g id="text_2">
      <!-- 1 -->
      <defs>
       <path d="M 12.40625 8.296875 
L 28.515625 8.296875 
L 28.515625 63.921875 
L 10.984375 60.40625 
L 10.984375 69.390625 
L 28.421875 72.90625 
L 38.28125 72.90625 
L 38.28125 8.296875 
L 54.390625 8.296875 
L 54.390625 0 
L 12.40625 0 
z
" id="DejaVuSans-49"/>
      </defs>
      <g transform="translate(77.45875 270.918437)scale(0.1 -0.1)">
       <use xlink:href="#DejaVuSans-49"/>
      </g>
     </g>
    </g>
    <g id="xtick_3">
     <g id="line2d_3">
      <g>
       <use style="stroke:#000000;stroke-width:0.8;" x="125.28" xlink:href="#m2d97570a81" y="256.32"/>
      </g>
     </g>
     <g id="text_3">
      <!-- 2 -->
      <defs>
       <path d="M 19.1875 8.296875 
L 53.609375 8.296875 
L 53.609375 0 
L 7.328125 0 
L 7.328125 8.296875 
Q 12.9375 14.109375 22.625 23.890625 
Q 32.328125 33.6875 34.8125 36.53125 
Q 39.546875 41.84375 41.421875 45.53125 
Q 43.3125 49.21875 43.3125 52.78125 
Q 43.3125 58.59375 39.234375 62.25 
Q 35.15625 65.921875 28.609375 65.921875 
Q 23.96875 65.921875 18.8125 64.3125 
Q 13.671875 62.703125 7.8125 59.421875 
L 7.8125 69.390625 
Q 13.765625 71.78125 18.9375 73 
Q 24.125 74.21875 28.421875 74.21875 
Q 39.75 74.21875 46.484375 68.546875 
Q 53.21875 62.890625 53.21875 53.421875 
Q 53.21875 48.921875 51.53125 44.890625 
Q 49.859375 40.875 45.40625 35.40625 
Q 44.1875 33.984375 37.640625 27.21875 
Q 31.109375 20.453125 19.1875 8.296875 
z
" id="DejaVuSans-50"/>
      </defs>
      <g transform="translate(122.09875 270.918437)scale(0.1 -0.1)">
       <use xlink:href="#DejaVuSans-50"/>
      </g>
     </g>
    </g>
    <g id="xtick_4">
     <g id="line2d_4">
      <g>
       <use style="stroke:#000000;stroke-width:0.8;" x="169.92" xlink:href="#m2d97570a81" y="256.32"/>
      </g>
     </g>
     <g id="text_4">
      <!-- 3 -->
      <defs>
       <path d="M 40.578125 39.3125 
Q 47.65625 37.796875 51.625 33 
Q 55.609375 28.21875 55.609375 21.1875 
Q 55.609375 10.40625 48.1875 4.484375 
Q 40.765625 -1.421875 27.09375 -1.421875 
Q 22.515625 -1.421875 17.65625 -0.515625 
Q 12.796875 0.390625 7.625 2.203125 
L 7.625 11.71875 
Q 11.71875 9.328125 16.59375 8.109375 
Q 21.484375 6.890625 26.8125 6.890625 
Q 36.078125 6.890625 40.9375 10.546875 
Q 45.796875 14.203125 45.796875 21.1875 
Q 45.796875 27.640625 41.28125 31.265625 
Q 36.765625 34.90625 28.71875 34.90625 
L 20.21875 34.90625 
L 20.21875 43.015625 
L 29.109375 43.015625 
Q 36.375 43.015625 40.234375 45.921875 
Q 44.09375 48.828125 44.09375 54.296875 
Q 44.09375 59.90625 40.109375 62.90625 
Q 36.140625 65.921875 28.71875 65.921875 
Q 24.65625 65.921875 20.015625 65.03125 
Q 15.375 64.15625 9.8125 62.3125 
L 9.8125 71.09375 
Q 15.4375 72.65625 20.34375 73.4375 
Q 25.25 74.21875 29.59375 74.21875 
Q 40.828125 74.21875 47.359375 69.109375 
Q 53.90625 64.015625 53.90625 55.328125 
Q 53.90625 49.265625 50.4375 45.09375 
Q 46.96875 40.921875 40.578125 39.3125 
z
" id="DejaVuSans-51"/>
      </defs>
      <g transform="translate(166.73875 270.918437)scale(0.1 -0.1)">
       <use xlink:href="#DejaVuSans-51"/>
      </g>
     </g>
    </g>
    <g id="xtick_5">
     <g id="line2d_5">
      <g>
       <use style="stroke:#000000;stroke-width:0.8;" x="214.56" xlink:href="#m2d97570a81" y="256.32"/>
      </g>
     </g>
     <g id="text_5">
      <!-- 4 -->
      <defs>
       <path d="M 37.796875 64.3125 
L 12.890625 25.390625 
L 37.796875 25.390625 
z
M 35.203125 72.90625 
L 47.609375 72.90625 
L 47.609375 25.390625 
L 58.015625 25.390625 
L 58.015625 17.1875 
L 47.609375 17.1875 
L 47.609375 0 
L 37.796875 0 
L 37.796875 17.1875 
L 4.890625 17.1875 
L 4.890625 26.703125 
z
" id="DejaVuSans-52"/>
      </defs>
      <g transform="translate(211.37875 270.918437)scale(0.1 -0.1)">
       <use xlink:href="#DejaVuSans-52"/>
      </g>
     </g>
    </g>
    <g id="xtick_6">
     <g id="line2d_6">
      <g>
       <use style="stroke:#000000;stroke-width:0.8;" x="259.2" xlink:href="#m2d97570a81" y="256.32"/>
      </g>
     </g>
     <g id="text_6">
      <!-- 5 -->
      <defs>
       <path d="M 10.796875 72.90625 
L 49.515625 72.90625 
L 49.515625 64.59375 
L 19.828125 64.59375 
L 19.828125 46.734375 
Q 21.96875 47.46875 24.109375 47.828125 
Q 26.265625 48.1875 28.421875 48.1875 
Q 40.625 48.1875 47.75 41.5 
Q 54.890625 34.8125 54.890625 23.390625 
Q 54.890625 11.625 47.5625 5.09375 
Q 40.234375 -1.421875 26.90625 -1.421875 
Q 22.3125 -1.421875 17.546875 -0.640625 
Q 12.796875 0.140625 7.71875 1.703125 
L 7.71875 11.625 
Q 12.109375 9.234375 16.796875 8.0625 
Q 21.484375 6.890625 26.703125 6.890625 
Q 35.15625 6.890625 40.078125 11.328125 
Q 45.015625 15.765625 45.015625 23.390625 
Q 45.015625 31 40.078125 35.4375 
Q 35.15625 39.890625 26.703125 39.890625 
Q 22.75 39.890625 18.8125 39.015625 
Q 14.890625 38.140625 10.796875 36.28125 
z
" id="DejaVuSans-53"/>
      </defs>
      <g transform="translate(256.01875 270.918437)scale(0.1 -0.1)">
       <use xlink:href="#DejaVuSans-53"/>
      </g>
     </g>
    </g>
   </g>
   <g id="matplotlib.axis_2">
    <g id="ytick_1">
     <g id="line2d_7">
      <defs>
       <path d="M 0 0 
L -3.5 0 
" id="meb4b4e43e9" style="stroke:#000000;stroke-width:0.8;"/>
      </defs>
      <g>
       <use style="stroke:#000000;stroke-width:0.8;" x="36" xlink:href="#meb4b4e43e9" y="256.32"/>
      </g>
     </g>
     <g id="text_7">
      <!-- 0 -->
      <g transform="translate(22.6375 260.119219)scale(0.1 -0.1)">
       <use xlink:href="#DejaVuSans-48"/>
      </g>
     </g>
    </g>
    <g id="ytick_2">
     <g id="line2d_8">
      <g>
       <use style="stroke:#000000;stroke-width:0.8;" x="36" xlink:href="#meb4b4e43e9" y="211.968"/>
      </g>
     </g>
     <g id="text_8">
      <!-- 1 -->
      <g transform="translate(22.6375 215.767219)scale(0.1 -0.1)">
       <use xlink:href="#DejaVuSans-49"/>
      </g>
     </g>
    </g>
    <g id="ytick_3">
     <g id="line2d_9">
      <g>
       <use style="stroke:#000000;stroke-width:0.8;" x="36" xlink:href="#meb4b4e43e9" y="167.616"/>
      </g>
     </g>
     <g id="text_9">
      <!-- 2 -->
      <g transform="translate(22.6375 171.415219)scale(0.1 -0.1)">
       <use xlink:href="#DejaVuSans-50"/>
      </g>
     </g>
    </g>
    <g id="ytick_4">
     <g id="line2d_10">
      <g>
       <use style="stroke:#000000;stroke-width:0.8;" x="36" xlink:href="#meb4b4e43e9" y="123.264"/>
      </g>
     </g>
     <g id="text_10">
      <!-- 3 -->
      <g transform="translate(22.6375 127.063219)scale(0.1 -0.1)">
       <use xlink:href="#DejaVuSans-51"/>
      </g>
     </g>
    </g>
    <g id="ytick_5">
     <g id="line2d_11">
      <g>
       <use style="stroke:#000000;stroke-width:0.8;" x="36" xlink:href="#meb4b4e43e9" y="78.912"/>
      </g>
     </g>
     <g id="text_11">
      <!-- 4 -->
      <g transform="translate(22.6375 82.711219)scale(0.1 -0.1)">
       <use xlink:href="#DejaVuSans-52"/>
      </g>
     </g>
    </g>
    <g id="ytick_6">
     <g id="line2d_12">
      <g>
       <use style="stroke:#000000;stroke-width:0.8;" x="36" xlink:href="#meb4b4e43e9" y="34.56"/>
      </g>
     </g>
     <g id="text_12">
      <!-- 5 -->
      <g transform="translate(22.6375 38.359219)scale(0.1 -0.1)">
       <use xlink:href="#DejaVuSans-53"/>
      </g>
     </g>
    </g>
   </g>
   <g id="line2d_13">
    <path clip-path="url(#pe22274eb01)" d="M 214.56 256.32 
L 36 78.912 
" style="fill:none;stroke:#0000ff;stroke-linecap:square;stroke-width:1.5;"/>
   </g>
   <g id="line2d_14">
    <path clip-path="url(#pe22274eb01)" d="M 259.2 256.32 
L 36 34.56 
" style="fill:none;stroke:#0000ff;stroke-dasharray:5.55,2.4;stroke-dashoffset:0;stroke-width:1.5;"/>
   </g>
   <g id="line2d_15">
    <path clip-path="url(#pe22274eb01)" d="M 169.92 256.32 
L 36 123.264 
" style="fill:none;stroke:#0000ff;stroke-dasharray:5.55,2.4;stroke-dashoffset:0;stroke-width:1.5;"/>
   </g>
   <g id="patch_8">
    <path d="M 36 256.32 
L 36 34.56 
" style="fill:none;stroke:#000000;stroke-linecap:square;stroke-linejoin:miter;stroke-width:0.8;"/>
   </g>
   <g id="patch_9">
    <path d="M 259.2 256.32 
L 259.2 34.56 
" style="fill:none;stroke:#000000;stroke-linecap:square;stroke-linejoin:miter;stroke-width:0.8;"/>
   </g>
   <g id="patch_10">
    <path d="M 36 256.32 
L 259.2 256.32 
" style="fill:none;stroke:#000000;stroke-linecap:square;stroke-linejoin:miter;stroke-width:0.8;"/>
   </g>
   <g id="patch_11">
    <path d="M 36 34.56 
L 259.2 34.56 
" style="fill:none;stroke:#000000;stroke-linecap:square;stroke-linejoin:miter;stroke-width:0.8;"/>
   </g>
  </g>
 </g>
 <defs>
  <clipPath id="pe22274eb01">
   <rect height="221.76" width="223.2" x="36" y="34.56"/>
  </clipPath>
 </defs>
</svg>

The projection length is the width of margin, which can be expressed as:

$$
WIDTH = \left( \overline{x}_{+} - \overline{x}_- \right) \cdot \frac{\overline{w}}{\parallel w \parallel}
$$

As the samples is on the margin line, we have 

$$
\overline{x}_+ = \frac{1 - b}{\overline{w}} \\ 
\overline{x}_- = \frac{1 + b}{\overline{w}}
$$

Bring them to the $WIDTH$ equation, we get:

$$
WIDTH = \frac{2}{\parallel w \parallel}
$$

So to find the max width, we need to find min of $\parallel w \parallel$, or for math convience, $\frac{1}{2} {\parallel w \parallel}^2$, and the minimum is under constraint of $y_i \left(\overline{w} \cdot \overline{x}_i + c \right) = 1$.

This is where Lagerange Multiplier shines. The minimum problem is equivalent to find the minimum of the following:

$$
L = \frac{1}{2} {\parallel w \parallel}^2 - sum{\ \alpha_i \left[ y_i \left(\overline{w} \cdot \overline{x}_i + b \right) - 1 \right]}
$$

In the equation, $\alpha_i$ equals 1 if we let margin line cross the sample $\overline{x}_i$, other wise 0. To find the minimum, we use derivatives.

$$
\frac{\partial L}{\partial{\overline{w}}} = \overline{w} - sum{\ \alpha_i y_i \overline{x}_i} = 0
\quad \Rightarrow \quad \overline{w} = sum{\ \alpha_i y_i \overline{x}_i} \\
\frac{\partial L}{\partial{\overline{b}}} = - sum{\ \alpha_i y_i} = 0 \quad \Rightarrow \quad sum{\ \alpha_i y_i} = 0
$$

Choosing the right samples to build the margin line is a quadratic optimization problem, I'll not solve it and let the $\alpha_i$ remains in the solution. Now we find the minimum $\overline{w}$. To classify the unknown $\overline{u}$, we calculate:

$$
\hat{y} = \overline{w} \cdot \overline{u} + b = sum{\ \alpha_i y_i \overline{x}_i} \overline{u} + b
$$

If $\hat{y} > 0$, it's positive else negetive. We can see that in the equation classifying unknown vectors, some samples in the training set remain, they are the 'support vector' of SVM. However, in FM, there is no such 'support vector'. And unlike FM, the weight do not designed to share some features and cannot be factored.  
