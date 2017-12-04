---
title: Advent of Code Day 3 
excerpt: My approach to solving today's problem
header: 
image: advent-1-625x352.jpg
---

It's that time of year again! Messing around with toy problems to help santa deliver all his presents on time! 
I probably won't end up posting my solution for every day, but I had a lot of fun solving this problem so I wanted
to get my solution out there. [Advent of Code](http://adventofcode.com/) is a super fun way to hack away at problems, so I highly recommend it! I will provide code but it is very rough so don't judge me on it!

## Part 1

In this problem you must calculate the Manhattan distance from a point in a matrix to the center of the matrix.
The matrix looks like this: ![matrix](https://imgur.com/T1GTvcX.png).

You are given a number, and expected to find the distance from that point to the center. 
I decided to not actually generate the matrix in this example, since that sounded complicated.
You can calculate the distance using some simple algebra. First, we must get the dimensions of the matrix. 
The matrix must always be square, and because of how the matrix is constructed, it must always have odd dimensions.
In the picture above, you can see that the matrix is 5x5. If we were to remove the outer layer it would be a 3x3. 
If we were to add an additional layer, it would be 7x7. 

We can calculate the minimum dimensions of the matrix by taking the square root of our given number, and rounding up to the nearest odd number. 
If our input point was 67, the smallest matrix it could fit in would be a 9x9 matrix. A 7x7 matrix could only fit 47 numbers, an 8x8 matrix is invalid, and a 9x9 could fit 81 numbers. 
So we know 67 would exist on the outer shell of a 9x9 matrix. This means that there is a minimum of 9 steps to get to the center. Just like that, we have calculated one half of our answer.

Next we need to calculate how far our point is from the closest midpoint. From our given point, we need to walk to the midpoint of our side, and then we need to walk directly from that midpoint to the matrix's center.
We can calculate each midpoint, and determine which one is the closest to our point. After doing that, we can add it to our first half and get the answer!

```python

def distance_from_closest(my_spot, center_spots):
    return min(list(map(lambda x: abs(my_spot - x), center_spots)))

# my input
raw = 312051
# it's a dimensionality x dimensionality sized matrix
dimensionality = math.ceil(raw ** (1 / 2))

if dimensionality % 2 == 0:
    dimensionality += 1
# the last number in the matrix
end = dimensionality**2

# width/2 
thickness = math.floor(dimensionality / 2)
center_spots = [end - thickness, end - (dimensionality - 1 + thickness), end - (2 * (dimensionality - 1) + thickness), end - (3 * (dimensionality - 1) + thickness)]
dist = distance_from_closest(raw, center_spots)

res = 0
res += thickness

print(thickness + dist)
```

## Part 2

This part was a little harder. Now, instead of each element in the matrix incrementing by one in a spiral pattern, it is instead the result of the addition of everything adjacent to it. 
The matrix should look like this then: ![matrix2](https://i.imgur.com/u80uxwl.png)

Now you have to find the first number in the matrix which is larger than your input.

This part was way harder for me. I couldn't figure out a smarter way to do this, so I walked through and generated the matrix. 
You don't need to store the entire matrix in memory, which is a neat optimization. In order to calculate the next 'layer' of the matrix, you only need the previous layer. 
In this way, you only have to store two arrays, with a set max length. That's nice. But the question is, how do we generate this matrix? 

It was easier for me to reason about this starting at the second layer, so I started my matrix generation with the first layer already completed:

```python
    previous_ring = [1,2,4,5,10,11,23,25]
```

I had a problem trying to conceptualize this problem. Working from the outer layer/ring, it's hard to figure out which values in the inner layer/ring must be used. 
Instead, I worked from the inside out. Using the picture as my guide, I determined which inner indices would be adjacent to outer indices. 

![colored-matrix](https://i.imgur.com/5cpHhnD.png)

You can visualize it here. I worked from the orange layer, trying to figure out which indices in the red layer corresponded.  

If you go around the inner orange layer, you will find a pattern. A number which is not on the corner will be adjacent to exactly 3 values on the outside. A number on the corner, however, will be adjacent to exactly 5 values on the outside. 
I stored these adjacencies as a dictionary keyed by the outer ring's index. We can calculate this dictionary like this:

```python
adj = {}
base = 0
    for ele in ring:
	# 5 possible adjacencies
        if ele in corners:
            unclamped_indices = [base, base + 1, base + 2, base+3, base+4]
	    # edge case: At the end of our ring, we have to wrap back around
            # The end of this ring is adjacent to the beginning of the next ring
            indices = list(map(lambda x: x % max_size, unclamped_indices))
            base += 3 
	# 3 possible adjacencies
        else:
            indices = [base, base + 1, base + 2]
            base += 1

        for ind in indices:
            if ind in adj:
                adj[ind] += ele
            else:
                adj[ind] = ele

    return adj

```


Given this dictionary, calculating the next value for the matrix should be straightforward. As we're creating the next value, we can just look it up in the dictionary. So all the work is done in building the dictionary.
Periodically, when we have completed our outer shell, we must transition to a new shell, and re-calculate the dictionary. The logic to generate a new entry in the matrix looks like this:

```python
    # my number
    while next < 312051:
	# fetch next
        next = adj[index]
	# fill this value in for later entries in the ring
        adj[(index+1) % max_size] += next
        new_ring.append(next)
        index += 1

        if len(new_ring) == max_size:
            previous_ring = new_ring
            new_ring = []
            index = 0
	    # 3x3 -> 5x5 -> 7x7 -> 9x9 etc		
            dimensionality += 2
	    # max size of our current ring
            max_size = ((dimensionality+2) ** 2) - (dimensionality ** 2)
	    # recalculate adjacencies 
            adj = calculate_adjs(previous_ring, max_size)

```



