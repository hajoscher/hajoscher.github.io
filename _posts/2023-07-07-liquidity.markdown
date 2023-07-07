---
layout: post
title:  "the Constant Product Formula for Liquidity Pools"
date:   2023-07-07
categories: crypto
usemathjax: true
---


The basic formula for liquidity pools

$$x  y = k$$

is well-known.  But how to derive it?

## TLDR

Having assets $x$ and $y$ in a pool, the price is $p=y/x$. Taking out an amount $\partial x$ one has to deposit $\partial y= - p  \partial x = -{\displaystyle \frac{y}{x}} \partial x$. Arrange like ${\displaystyle \frac{\partial x}{x}} = - {\displaystyle \frac{\partial y}{y}}$. Solving this differential equation yields $\ln(x) + \ln(y) = \ln (x y) = C$, and after exponentiation $x y = e^C = k$.

## Long Version

Let's start out by stating the obvious, the price is the fraction of the assets in the pool.
E.g. $x$ is BTC and $y$ is USDC. The price per Bitcoin is the number of USDC in the pool
divided by the number of BTC:

$$ p = y/x $$

Now, if someone wants to buy/sell some amount $\Delta x$ BTC, and assuming there are no fees, someone removes/puts in BTC and puts/removes the amount multiplied by the price in USDC.

$$
\Delta y = - (\Delta x)  p = - \Delta x \frac{y}{x}
$$

$\Delta x$ is negative if someone buys BTC , since the BTC amount in the pool gets reduced. The number $\Delta y$ is then positive, since the trader has to deposit USDC, the amount in the pool increases. And vice versa for a sell.

Now, for this to work we must ensure that the price changes consistently with each transaction. We cannot just calculate the price and do some transaction.

The easiest way to see why this is, is that if someone buys 1 BTC out of the pool, then sells the BTC back in the pool should be in the same state as in the beginning, with the same price. But, since the price is higher after the first transaction,
the trader would gain money buy doing this.

Let's do an example. $x_0=10$ and $y_0=200 \\, 000$. So the price initially is  $p=20 \\, 000$ USDC per BTC.

Now buy one BTC from the pool:

$$
\begin{align}
 \Delta x &= -1 \\
 x_1 &= x_0 + \Delta x = 9    \\
 \Delta y &= - (-1) \times 20\\,000 = 20\\,000 \\
 y_1 &= y_0 + \Delta y = 200 \\, 000 + 20\\,000 = 220\\,000 \\
 p_1 &= y_1 / x_1 =  24\\,444.44 \\
\end{align}
$$

The index indicates the state after transaction one. The pool has now $x_1=9$ BTC , and $y_1=220\\,000$ USDC, and the new price would be $p_1 = y_1/x_1 = 24\\,444.44$ USD per BTC .

Now we sell one BTC back to the pool at $p_1=24\\,444.44$, so $\Delta x = +1$.

$$
\begin{align}
 \Delta x &= 1    \\
  x_2 &= x_1 + \Delta x = 10    \\
 \Delta y &= \Delta x \times p_1 = - (1) \times 24\\,444.44 = -24\\,444.44 \\
 y_2 &= y_1 + \Delta y = 220 \\, 000 - 24\\,444.44 = 195\\,555.56 \\
\end{align}
$$

Wow, the trader paid $20\\,000$ USDC for one BTC and sold it back to the pool for $24\\,444.44$,
gaining $4\\,444.44$ USDC in the process, which is missing from the pool now.

The mistake is, we calculate the price independently from how large the transaction is. The price is really a function of $x$ and  $y$. The solution is to divide the transaction into many smaller ones, to give the price a chance to change.

Here is a little python script to simulate buying 1 BTC in $n$ steps, then selling it back to the pool, also in $n$ steps.

```python
x = 10
y = 200_000
n = 1   # try various n

# buy 1 BTC from pool in n steps
for i in range(n):
    p = y/x
    dx = -1/n
    x = x + dx
    y = y - dx * p
    print(f"x={round(x,2)}, y={round(y,2)}")

# sell 1 BTC to pool in n steps
for i in range(n):
    p = y/x
    dx = 1/n
    x = x + dx
    y = y - dx * p
    print(f"x={round(x,2)}, y={round(y,2)}")
print(f"missing y = {round(200_000 - y,2)}")
```

Let's put the missing money in a table for various number of steps $n$.
| n  | missing y  |
| ---- | ------------- |
| 1  | 4444.44  |
| 10  | 444.00  |
| 100 | 44.44 |
| 10000 | 0.44 |

So the price must change along with every transaction, as if dividing large transactions into many small ones.
The equation only holds for very small, infinitesimal transactions, then you need to re-evaluate the price and continue. A classic case for differential calculus.


$$ \partial y = - p \partial x $$

Price is $y/x$

$$ \partial y = - \frac{y}{x} \partial x $$

Rearranging $x$ and $y$ together:

$$ {\displaystyle \frac{\partial x}{x} = - \frac{\partial y}{y} } $$

Now, remember, the integral of $1/x$ is $\ln(x) + \mathrm{constant}$. The integration is exactly what the above script tries to achieve numerically.

$$ \ln(x) = - \ln(y) + C  $$

$$ \ln(x) + \ln (y) = C   $$

Adding logarithms by multiplying their arguments:

$$ \ln ( x  y ) = C $$

Exponentiation:

$$ x  y = e^C $$

Defining $k = e^C$ leads to:

$$ x  y = k $$

Done.

The constant $k$ is really just a constant. You calculate it in the beginning and keep it the same after each transaction. It is not the total value on the pool or such things. It has the un-intuitive unit BTC times USDC.

Now let's try the 1 BTC round trip trade again:

We have $k = 10 * 200\\,000 = 2\\,000\\,000 $

Buy 1 BTC yields $x_1 = 10-1 = 9 $,  $y_1 = k / 9 = 222\\,222.22$.
The trader has to pay $\Delta y = y_1 - y_0 = 222\\,222.22 - 200\\,000 =  22\\,222.22$.

Selling 1 BTC back to the pool gives $x_2 = x_1 + \Delta x = 9 + 1 = 10$,  $y_2 = k / 10 = 200\\,000$,
and $\Delta y = y_2 - y_1 = -22\\,222.22$, which is negative so the pool gets reduced by this amount and goes back to the trader.

Everything works now.
