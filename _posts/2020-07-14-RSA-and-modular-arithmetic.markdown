---
layout: post
title:  "RSA and modular arithmetic"
date:   2020-07-14 +0800
categories: RSA modular-arithmetic Euler-Theorem Chinese-Remainder-Theorem
---

Recently, I took the course of [Applied Cryptography](https://www.udacity.com/course/applied-cryptography--cs387) and learned more details about RSA. The math behind the scene is complicated and attractive for me, so I decided to figure it out.

There are lots of correctness proofs for RSA on the internet, and the most elegant one comes from [original paper](https://people.csail.mit.edu/rivest/Rsapaper.pdf). This article serves as a note or index about the algorithm, and related math theorems.

# First look

RSA uses exponent calculation based on modular arithmetic. A message $M$ is encrypted by raising it with some number $e$, eg, the cipher $C = M ^ e$; and decryption by raising the cipher to another number $d$, eg, $M = C ^ d = (M ^ e) ^ d = M ^ {ed}$.

The original paper has an example, $e = 17$, $d = 157$, modular $n = 2773$, and the message $M = 920$. Using python, we can easily check that $C \equiv M ^ e \equiv 920 ^ {17} \equiv 948 \mod 2773$, and for decryption, $M \equiv C ^ d \equiv 948 ^ {157} \equiv 920 \mod 2773$. it works.

```
>>> 920 ** 17
242322122805021463846350650290995200000000000000000
>>> _ % 2773
948
>>> 948 ** 157
228511974050288606356776902344003250786765339280429567483703996687679918754310808116762581946500309044935063895813994903862607889435410969372665687695332125577189642781886048649411339280291847319790295052238061530036066382680768550813673301548951552880234295829603371076580111293376807996489026124001520858798504685685090226812534126526485193603752385784652819655258004954411544721976443792145433128756776848416542537051430937496263760556207832371853899712995186966528
>>> _ % 2773
920
```

# How $e$ and $d$ are chosen?

Let's first walk through the process generating $e$ and $d$.

RSA firstly chooses two prime numbers $p$ and $q$, making the modular $n = pq$. Then $e$ and $d$ are generated with the constraint that $ed - 1$ is a multiple of $\varphi(n)$. Note that $\varphi()$ is Euler's totient function, in our case, $\varphi(n) = \varphi(pq) = (p - 1)(q - 1)$.

# Why does it work?

In the above example, we see that $M \equiv M ^ {ed} \mod n$. If both sides can be divided by $M$, we have:

$$ M ^ {ed - 1} \equiv 1 \mod n \tag{1} $$.

This looks quite like Euler’s Theorem, which states:

$$ M ^ {\varphi(n)} \equiv 1 \mod n $$

As $ed - 1$ is a multiple of $\varphi(n)$, we have $ed - 1 = k \varphi(n)$ for some k. By using Euler’s Theorem, we can conduct that:

$$ \begin{align}
M ^ {ed} &\equiv M ^ {ed - 1 + 1} \\
         &\equiv M ^ {k \varphi(n)} \times M \\
         &\equiv M ^ {\varphi(n)} \times M ^ {\varphi(n)} \times … M ^ {\varphi(n)} \times M \\
         &\equiv （M ^ {\varphi(n)} \mod n) \times (M ^ {\varphi(n)} \mod n) \times … (M ^ {\varphi(n)} \mod n) \times M \\
         &\equiv 1 \times 1 \times … \times 1 \times M \\
         &\equiv M \mod n
\tag{2}
\end{align} $$

The plain text $M$ and modular $n$ should be coprime by the precondition of Euler’s Theorem. However, this precondition is [not necessary](https://crypto.stackexchange.com/a/1008) if conducted by Fermat's Little Theorem.

By Fermat's Little Theorem, if $M$ and $p$ are coprime, we have $M ^ {p - 1} \equiv 1 \mod p$, which leads to:

$$ \begin{align}
M ^ {ed} &= M ^ {k \varphi(n)} \times M \\
         &= M ^ {k(p -1)(q - 1)} \times M \\
         &= (M ^ {p - 1}) ^ {k(q - 1)} \times M \\
         &= (M ^ {p - 1} \mod p) ^ {k(q - 1)} \times M \\
         &= 1 ^ {k(q - 1)} \times M \\
         &= 1 \times M \\
         &= M \mod p
\tag{3}
\end{align} $$

And the same applies for $q$: $M ^ {ed} \equiv M \mod q$.

Since $M ^ {ed}$ has the same remainder when divided by $p$ and $q$, we have $M ^ {ed} = M \mod pq = M \mod n$ by Chinese Remainder Theorem.

Since $p$ and $q$ are prime numbers, any other number is naturally coprime to both of them. We don't need coprime precondition any more.

# Modular Arithmetic

Equation (1) is derived by division, which we take for granted in real number arithmetic; and equation (2)(3) utilized a multiplication property: $ a \times b \mod n \equiv (a \mod n) \times (b \mod n) \mod n $. Things become different as we are not doing usual arithmetic. We need to prove those properties before taking advantages of them. Let's begin with intuition.

## Intuition

Old textbook told me that math is abstract. Indeed modular arithmetic is the abstract of periodic things in the real world.
Clock is one of the periodic things. When the clock handle goes past number 12 in the plate, it’s round back to number 1. The addition and subtraction on the clock is abstract from moving the clock handle clockwise or counter clockwise.

For example:

$$ 9 + 5 \equiv 2 \mod 12 $$

It means to move 5 units clockwise to the clock handle pointed at number 9, as the result, the handler now points to number 2.
For normal arithmetic, $ 5 * 3 = 5 + 5 + 5 = 15 $, the same applies to modular arithmetic:

$$ 5 * 3 \equiv 5 + 5 + 5 \equiv 15 \equiv 3 \mod 12 $$

And, division is the inverse of multiplication. For normal arithmetic, divisor zero is meaningless. Because division is a one to one map, all numbers map to zero when multiplied by zero, so the inverse is all numbers when divided by zero. For modular arithmetic, we have $ 4 \times 5 \equiv 8 \mod 12 $, so $ 8 \div 5 \equiv 4 \mod 12 $. However, $ 8 \div 4 \mod 12 $ is meaningless, as $ 4 \times 2 \equiv 8 \mod 12 $, $ 4 \times 5 \equiv 8 \mod 12 $, $ 4 \times 8 \equiv 8 \mod 12 $ and $ 4 \times 11 \equiv 8 \mod 12 $, so when divided by $4$, we get four results $2$, $5$, $8$ and $11$ which is not a one to one map.

## Properties

After the abstraction, it's time to find the properties of modular arithmetic.

The first comes to the addition:

If $A \equiv a \mod n$, and $B \equiv b \mod n$, then $A + B \equiv a + b \mod n$.

We can prove it by visualizing $A$ and $B$ to two lines. As the figure below shows, $A$ and $B$ both can be broken down into divisible parts and remainders, the remainders are $a$ and $b$. When $A$ and $B$ are concatenated together, the divisible parts are still divisible, so the result depends on the remainders of $A$ and $B$, which is $a + b$.

<svg
   xmlns:dc="http://purl.org/dc/elements/1.1/"
   xmlns:cc="http://creativecommons.org/ns#"
   xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"
   xmlns:svg="http://www.w3.org/2000/svg"
   xmlns="http://www.w3.org/2000/svg"
   xmlns:sodipodi="http://sodipodi.sourceforge.net/DTD/sodipodi-0.dtd"
   xmlns:inkscape="http://www.inkscape.org/namespaces/inkscape"
   sodipodi:docname="drawing.svg"
   inkscape:version="1.0 (4035a4fb49, 2020-05-01)"
   id="svg8"
   version="1.1"
   viewBox="0 0 75.521049 57.336212"
   height="57.336212mm"
   width="75.521049mm">
  <defs
     id="defs2">
    <marker
       inkscape:isstock="true"
       style="overflow:visible"
       id="StopS"
       refX="0"
       refY="0"
       orient="auto"
       inkscape:stockid="StopS">
      <path
         transform="scale(0.2)"
         style="fill:#00ffff;fill-opacity:1;fill-rule:evenodd;stroke:#0000ff;stroke-width:1pt;stroke-opacity:1"
         d="M 0,5.65 V -5.65"
         id="path1196" />
    </marker>
  </defs>
  <sodipodi:namedview
     inkscape:window-maximized="1"
     inkscape:window-y="0"
     inkscape:window-x="0"
     inkscape:window-height="740"
     inkscape:window-width="1366"
     showborder="false"
     fit-margin-bottom="0"
     fit-margin-right="0"
     fit-margin-left="0"
     fit-margin-top="0"
     inkscape:snap-bbox="false"
     inkscape:snap-midpoints="false"
     showgrid="false"
     inkscape:document-rotation="0"
     inkscape:current-layer="layer1"
     inkscape:document-units="mm"
     inkscape:cy="115.99115"
     inkscape:cx="147.22567"
     inkscape:zoom="1.4283557"
     inkscape:pageshadow="2"
     inkscape:pageopacity="0.0"
     borderopacity="1.0"
     bordercolor="#666666"
     pagecolor="#ffffff"
     id="base" />
  <metadata
     id="metadata5">
    <rdf:RDF>
      <cc:Work
         rdf:about="">
        <dc:format>image/svg+xml</dc:format>
        <dc:type
           rdf:resource="http://purl.org/dc/dcmitype/StillImage" />
        <dc:title></dc:title>
      </cc:Work>
    </rdf:RDF>
  </metadata>
  <g
     transform="translate(-49.684091,-14.103735)"
     id="layer1"
     inkscape:groupmode="layer"
     inkscape:label="Layer 1">
    <path
       id="path32"
       d="M 50.184091,25.035692 H 71.624204"
       style="fill:#00ffff;stroke:#0000ff;stroke-width:1;stroke-linecap:round;stroke-linejoin:miter;stroke-miterlimit:7.2;stroke-dasharray:none;stroke-opacity:1;paint-order:markers fill stroke" />
    <path
       style="fill:#0000ff;stroke:#0000ff;stroke-width:1;stroke-linecap:butt;stroke-linejoin:miter;stroke-miterlimit:4;stroke-dasharray:none;stroke-opacity:1"
       d="M 89.122538,25.036 H 110.56265"
       id="path32-3" />
    <path
       id="path49"
       d="M 71.624204,25.035692 H 82.774858"
       style="fill:none;stroke:#008000;stroke-width:1;stroke-linecap:round;stroke-linejoin:miter;stroke-miterlimit:4;stroke-dasharray:none;stroke-opacity:1;paint-order:stroke markers fill" />
    <path
       id="path51"
       d="m 110.56265,25.036 h 14.64249"
       style="fill:none;stroke:#ff0000;stroke-width:1;stroke-linecap:butt;stroke-linejoin:miter;stroke-miterlimit:4;stroke-dasharray:none;stroke-opacity:1" />
    <path
       style="fill:#0000ff;stroke:#0000ff;stroke-width:1;stroke-linecap:butt;stroke-linejoin:miter;stroke-miterlimit:4;stroke-dasharray:none;stroke-opacity:1"
       d="m 71.624113,62.733208 h 21.44011"
       id="path32-6" />
    <path
       style="fill:#00ffff;stroke:#0000ff;stroke-width:1;stroke-linecap:butt;stroke-linejoin:miter;stroke-miterlimit:4;stroke-dasharray:none;stroke-opacity:1;marker-end:url(#StopS)"
       d="M 50.184,62.733208 H 71.624113"
       id="path32-7" />
    <path
       style="fill:#00ffff;stroke:#0000ff;stroke-width:1;stroke-linecap:butt;stroke-linejoin:miter;stroke-miterlimit:4;stroke-dasharray:none;stroke-opacity:1"
       d="M 50.184,48.603547 H 71.624113"
       id="path32-5" />
    <path
       style="fill:none;stroke:#008000;stroke-width:1;stroke-linecap:butt;stroke-linejoin:miter;stroke-miterlimit:4;stroke-dasharray:none;stroke-opacity:1"
       d="M 71.624113,48.603547 H 82.774766"
       id="path49-3" />
    <text
       id="text917"
       y="31.681204"
       x="74.621719"
       style="font-style:normal;font-weight:normal;font-size:9.20893px;line-height:1.25;font-family:sans-serif;fill:#008000;fill-opacity:1;stroke:none;stroke-width:0.230224"
       xml:space="preserve"><tspan
         style="fill:#008000;stroke-width:0.230224"
         y="31.681204"
         x="74.621719"
         id="tspan915"
         sodipodi:role="line">a</tspan></text>
    <text
       xml:space="preserve"
       style="font-style:normal;font-weight:normal;font-size:9.20893px;line-height:1.25;font-family:sans-serif;fill:#ff0000;fill-opacity:1;stroke:none;stroke-width:0.230224"
       x="115.28797"
       y="32.558121"
       id="text917-5"><tspan
         sodipodi:role="line"
         id="tspan915-6"
         x="115.28797"
         y="32.558121"
         style="fill:#ff0000;stroke-width:0.230224">b</tspan></text>
    <text
       xml:space="preserve"
       style="font-style:normal;font-weight:normal;font-size:9.20893px;line-height:1.25;font-family:sans-serif;fill:#000000;fill-opacity:1;stroke:none;stroke-width:0.230224"
       x="63.869789"
       y="20.981277"
       id="text917-2"><tspan
         sodipodi:role="line"
         id="tspan915-9"
         x="63.869789"
         y="20.981277"
         style="fill:#000000;stroke-width:0.230224">A</tspan></text>
    <text
       id="text917-2-1"
       y="20.817045"
       x="103.80862"
       style="font-style:normal;font-weight:normal;font-size:9.20893px;line-height:1.25;font-family:sans-serif;fill:#000000;fill-opacity:1;stroke:none;stroke-width:0.230224"
       xml:space="preserve"><tspan
         style="fill:#000000;stroke-width:0.230224"
         y="20.817045"
         x="103.80862"
         id="tspan915-9-2"
         sodipodi:role="line">B</tspan></text>
    <path
       id="path32-3-7"
       d="M 82.774766,48.603547 H 104.21488"
       style="fill:#0000ff;stroke:#0000ff;stroke-width:1;stroke-linecap:butt;stroke-linejoin:miter;stroke-miterlimit:4;stroke-dasharray:none;stroke-opacity:1" />
    <path
       style="fill:none;stroke:#ff0000;stroke-width:1;stroke-linecap:butt;stroke-linejoin:miter;stroke-miterlimit:4;stroke-dasharray:none;stroke-opacity:1"
       d="m 104.21488,48.603547 h 14.64249"
       id="path51-0" />
    <path
       id="path49-3-9"
       d="M 93.064223,62.733208 H 104.21487"
       style="fill:none;stroke:#008000;stroke-width:1;stroke-linecap:butt;stroke-linejoin:miter;stroke-miterlimit:4;stroke-dasharray:none;stroke-opacity:1" />
    <path
       id="path51-0-3"
       d="m 104.21487,62.733208 h 14.64249"
       style="fill:none;stroke:#ff0000;stroke-width:1;stroke-linecap:butt;stroke-linejoin:miter;stroke-miterlimit:4;stroke-dasharray:none;stroke-opacity:1" />
    <text
       xml:space="preserve"
       style="font-style:normal;font-weight:normal;font-size:9.20893px;line-height:1.25;font-family:sans-serif;fill:#008000;fill-opacity:1;stroke:none;stroke-width:0.230224"
       x="97.422539"
       y="71.228142"
       id="text917-6"><tspan
         sodipodi:role="line"
         id="tspan915-0"
         x="97.422539"
         y="71.228142"
         style="fill:#008000;stroke-width:0.230224">a<tspan
   id="tspan1445"
   style="fill:#000000">+</tspan><tspan
   id="tspan1443"
   style="fill:#ff0000">b</tspan></tspan></text>
  </g>
</svg>

Then is the multiplication:

If $ A \equiv a \mod n $, and $ B \equiv b \mod n $, then $ AB \equiv (A \mod n)(B \mod n) \equiv ab \mod n $.

This time we can use a rectangle to visualize the proof, of which the width and length is $A$ and $B$. The area of the rectangle can be broken down into four parts by the divisible parts and remainder parts of $A$ and $B$. Three of them are divisible, and the non-divisible part is equal to ab, which proves the property.

<svg
   xmlns:dc="http://purl.org/dc/elements/1.1/"
   xmlns:cc="http://creativecommons.org/ns#"
   xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"
   xmlns:svg="http://www.w3.org/2000/svg"
   xmlns="http://www.w3.org/2000/svg"
   xmlns:sodipodi="http://sodipodi.sourceforge.net/DTD/sodipodi-0.dtd"
   xmlns:inkscape="http://www.inkscape.org/namespaces/inkscape"
   sodipodi:docname="drawing1.svg"
   inkscape:version="1.0 (4035a4fb49, 2020-05-01)"
   id="svg8"
   version="1.1"
   viewBox="0 0 79.49505 69.618225"
   height="69.618225mm"
   width="79.495056mm">
  <defs
     id="defs2" />
  <sodipodi:namedview
     width="296mm"
     units="mm"
     showborder="true"
     inkscape:showpageshadow="false"
     fit-margin-bottom="0"
     fit-margin-right="0"
     fit-margin-left="0"
     fit-margin-top="0"
     inkscape:window-maximized="1"
     inkscape:window-y="0"
     inkscape:window-x="0"
     inkscape:window-height="740"
     inkscape:window-width="1366"
     showgrid="false"
     inkscape:document-rotation="0"
     inkscape:current-layer="layer1"
     inkscape:document-units="mm"
     inkscape:cy="116.35224"
     inkscape:cx="222.87865"
     inkscape:zoom="1.01"
     inkscape:pageshadow="2"
     inkscape:pageopacity="0.0"
     borderopacity="1.0"
     bordercolor="#666666"
     pagecolor="#ffffff"
     id="base" />
  <metadata
     id="metadata5">
    <rdf:RDF>
      <cc:Work
         rdf:about="">
        <dc:format>image/svg+xml</dc:format>
        <dc:type
           rdf:resource="http://purl.org/dc/dcmitype/StillImage" />
        <dc:title></dc:title>
      </cc:Work>
    </rdf:RDF>
  </metadata>
  <g
     transform="translate(-62.390415,-72.84311)"
     id="layer1"
     inkscape:groupmode="layer"
     inkscape:label="Layer 1">
    <rect
       y="89.902641"
       x="74.363403"
       height="40.199265"
       width="40.199265"
       id="rect837"
       style="fill:#0000ff;fill-opacity:0;stroke:#0000ff;stroke-width:1;stroke-miterlimit:4;stroke-dasharray:none" />
    <path
       id="path843"
       d="M 74.363402,89.90264 V 73.34311"
       style="fill:#008000;stroke:#008000;stroke-width:1;stroke-linecap:butt;stroke-linejoin:miter;stroke-miterlimit:4;stroke-dasharray:none;stroke-opacity:1" />
    <path
       id="path845"
       d="m 114.56267,130.1019 h 26.8228"
       style="fill:#ff0000;stroke:#ff0000;stroke-width:1;stroke-linecap:butt;stroke-linejoin:miter;stroke-miterlimit:4;stroke-dasharray:none;stroke-opacity:1" />
    <path
       style="fill:#008000;stroke:#008000;stroke-width:1;stroke-linecap:butt;stroke-linejoin:miter;stroke-miterlimit:4;stroke-dasharray:none;stroke-opacity:1"
       d="M 114.56267,89.90264 V 73.34311"
       id="path843-3" />
    <path
       style="fill:#ff0000;stroke:#ff0000;stroke-width:1;stroke-linecap:butt;stroke-linejoin:miter;stroke-miterlimit:4;stroke-dasharray:none;stroke-opacity:1"
       d="m 114.56267,89.90264 h 26.8228"
       id="path845-6" />
    <path
       id="path843-3-7"
       d="M 141.38547,89.90264 V 73.34311"
       style="fill:#008000;stroke:#008000;stroke-width:1;stroke-linecap:butt;stroke-linejoin:miter;stroke-miterlimit:4;stroke-dasharray:none;stroke-opacity:1" />
    <path
       id="path845-6-5"
       d="m 114.56267,73.34311 h 26.8228"
       style="fill:#ff0000;stroke:#ff0000;stroke-width:1;stroke-linecap:butt;stroke-linejoin:miter;stroke-miterlimit:4;stroke-dasharray:none;stroke-opacity:1" />
    <path
       id="path923"
       d="M 74.363402,73.34311 H 114.56267"
       style="fill:#0000ff;stroke:#0000ff;stroke-width:1;stroke-linecap:butt;stroke-linejoin:miter;stroke-miterlimit:4;stroke-dasharray:none;stroke-opacity:1" />
    <path
       id="path925"
       d="M 141.38547,89.90264 V 130.1019"
       style="fill:#0000ff;stroke:#0000ff;stroke-width:1;stroke-linecap:butt;stroke-linejoin:miter;stroke-miterlimit:4;stroke-dasharray:none;stroke-opacity:1" />
    <text
       id="text978"
       y="84.297691"
       x="75.418777"
       style="font-style:normal;font-weight:normal;font-size:10.5833px;line-height:1.25;font-family:sans-serif;fill:#008000;fill-opacity:1;stroke:#008000;stroke-width:0.264583"
       xml:space="preserve"><tspan
         style="fill:#008000;stroke:#008000;stroke-width:0.264583"
         y="84.297691"
         x="75.418777"
         id="tspan976"
         sodipodi:role="line">a</tspan></text>
    <text
       id="text982"
       y="127.90557"
       x="125.79942"
       style="font-style:normal;font-weight:normal;font-size:10.5833px;line-height:1.25;font-family:sans-serif;fill:#ff0000;fill-opacity:1;stroke:#ff0000;stroke-width:0.264583"
       xml:space="preserve"><tspan
         style="fill:#ff0000;stroke:#ff0000;stroke-width:0.264583"
         y="127.90557"
         x="125.79942"
         id="tspan980"
         sodipodi:role="line">b</tspan></text>
    <text
       id="text986"
       y="85.952957"
       x="122.10623"
       style="font-style:normal;font-weight:normal;font-size:10.5833px;line-height:1.25;font-family:sans-serif;fill:#000000;fill-opacity:1;stroke:none;stroke-width:0.264583"
       xml:space="preserve"><tspan
         style="stroke-width:0.264583"
         y="85.952957"
         x="122.10623"
         id="tspan984"
         sodipodi:role="line">ab</tspan></text>
    <text
       id="text990"
       y="108.14462"
       x="62.210499"
       style="font-style:normal;font-weight:normal;font-size:10.5833px;line-height:1.25;font-family:sans-serif;fill:#000000;fill-opacity:1;stroke:none;stroke-width:0.264583"
       xml:space="preserve"><tspan
         style="stroke-width:0.264583"
         y="108.14462"
         x="62.210499"
         id="tspan988"
         sodipodi:role="line">A</tspan></text>
    <text
       id="text994"
       y="142.46133"
       x="107.04762"
       style="font-style:normal;font-weight:normal;font-size:10.5833px;line-height:1.25;font-family:sans-serif;fill:#000000;fill-opacity:1;stroke:none;stroke-width:0.264583"
       xml:space="preserve"><tspan
         style="stroke-width:0.264583"
         y="142.46133"
         x="107.04762"
         id="tspan992"
         sodipodi:role="line">B</tspan></text>
  </g>
</svg>

Thanks for this multiplication property, the transformations are valid in equations (2)(3).

And by these properties, we can conclude that modular arithmetic only cares about the remainder part.

Then is the division:

For real number, divided by some number is equivalent to multiply with its inverse. The same applies for modular division, so it is valid if and only if the inverse is unique. It turns out that the unique divisor exists if and only if divisor and modular are coprime. The sufficient and necessary conditions are proved in [here](https://stanford.edu/~dntse/classes/cs70_fall09/cs70_fall09_5.pdf) and [here](https://math.stackexchange.com/questions/2101189/proving-that-modular-inverse-only-exists-when-gcdn-x-1). Now we understand why the original paper requires message and modular are coprime and the precondition of equation (1).

# Theorems leveraged by RSA

Now we start proving theorems leveraged by RSA.

## Fermat’ Little Theorem

This theorem states that $ a ^ {p - 1} \equiv 1 \mod p $ if $a$ and $p$ are coprime. Wikipedia uses an intuitive way to prove it by [counting necklaces](https://en.wikipedia.org/wiki/Proofs_of_Fermat%27s_little_theorem). Section Fermat/Euler Theorem in [another paper](https://aip.scitation.org/doi/pdf/10.1063/1.3526259) proves it by congruent and permutation, which defines two sets $P$ and $Q$ under prime modular $p$. Elements in $P$ are from $1$ to $p - 1$, which are all coprime to $p$. And $Q$ is made by multiplying each element in $P$ with some number. The theorem is proved by observing that $Q$ and $P$ are the same since the result of multiplications of elements in $P$ is a permutation of $P$ as elements in $Q$ are distinct to each other and the size/range of $P$ and $Q$ are the same. However, it’s insufficient to support the Euler's Theorem using this approach provided by the paper. By the way, Fermat's Little Theorem is a special case in Eulur's Theorem.

## Euler's Theorem:

Now the modular is not required to be prime, so it's a general case of Fermat's little theorem. The theorem is also proved by congruent and permutation. An Indian brother [explained the proof](https://www.youtube.com/watch?v=cnYit1nQ12U) with his heavy accent. The set $P$ still contains all the numbers coprime to modular and $Q$ is still made by multiplying all the numbers in $P$. Note that as the modular is not prime, some numbers from $1$ to the $modular - 1$ may not belong to set $P$, which distincts it from Fermat’s Little Theorem. After multiplying, numbers in $Q$ have the possibility of not coprime with modular any more. So extra steps are taken to show that elements in $Q$ are still coprime, by which we get a precondition of permutation. Other processes are the same as Fermat’s Little Theorem.

Euler's totient function is proved in a series of posts beginning at [here](https://mathlesstraveled.com/2019/05/09/computing-the-euler-totient-function-part-1/) by leveraging the multiplicative property of it and factorization.

## Chinese Remainder Theorem

RSA uses the corollary or special case of Chinese Remainder Theorem. Let’s walk through the general case first. This theorem says if $ x \equiv a \mod p $ and $ x \equiv b \mod q $, then we have a unique solution for $ x \mod pq $. [Here](https://crypto.stanford.edu/pbc/notes/numbertheory/crt.html) the theorem is proved by constructing $x$ via modular inverse mentioned before.

The corollary says that if $a$ and $b$ are the same, we have $ x \equiv a \mod pq $.
[Here](https://exploringnumbertheory.wordpress.com/2013/11/23/proving-chinese-remainder-theorem/) is the proof of the corollary by letting $ x = a = b $. The corollary is also proved directly by [the paper](https://aip.scitation.org/doi/pdf/10.1063/1.3526259) used before to prove Fermat’s Little Theorem.
