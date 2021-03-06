[home](http://tiny.cc/fss2016) | [copyright](https://github.com/txt/fss16/blob/master/LICENSE.md) &copy;2016, tim&commat;menzies.us<br>
[<img width=900 src="https://raw.githubusercontent.com/txt/fss16/master/img/fss16.png">](http://tiny.cc/fss2016)   <br>
[overview](https://github.com/txt/fss16/blob/master/doc/overview.md) |
[syllabus](https://github.com/txt/fss16/blob/master/doc/syllabus.md) |
[src](https://github.com/txt/fss16/blob/master/src) |
[submit](http://tiny.cc/fss2016give) |
[chat](https://fss16.slack.com/) 

_______


Decision Trees
--------------

Bayes classifiers *perform* but they do not *explain* their performance.
If you ask "what is going on? how does it make its decisions?", there's
no answer except to browse the complicated frequency count tables.

Q: So, how to learn a *decision tree* whose leaves are classifications
and whose internal nodes are tests on attributes?

        curb-weight <= 2660 : 
        |   curb-weight <= 2290 : 
        |   |   curb-weight <= 2090 : 
        |   |   |   length <= 161 : price=6220
        |   |   |   length >  161 : price=7150
        |   |   curb-weight >  2090 : price=8010
        |   curb-weight >  2290 : 
        |   |   length <= 176 : price=9680
        |   |   length >  176 : 
        |   |   |   normalized-losses <= 157 : price=10200
        |   |   |   normalized-losses >  157 : price=15800
        curb-weight >  2660 : 
        |   width <= 68.9 : price=16100
        |   width >  68.9 : price=25500


This tree is built by recursively partitioning the data:

![](https://cdn.pbrd.co/images/eum8YzNoJ.png)

(The above diagram comes from a very nice, and short, discussion of decision trees:
[LARGE-SCALE MUSIC SIMILARITY SEARCH WITH SPATIAL TREES, Brian McFee, Gert Lanckriet, ISMIRE'11](http://ismir2011.ismir.net/papers/PS1-3.pdf)).

### Preliminaries: Entropy

-   entropy of a bunch of symbols occurring with probability p1, p2, ...
    then   
        - entropy(p1,p2,...) = &Sigma; -p * log2(p)
		- btw, log2 = log base 2
-   Hints:
    -   if your favorite programming language has no "base 2", then use
        log2(x)=log(x)/log(2)
    -   If "x" occurs f times in a sample of size "n", then "p = f/n"
-   Examples:
    -   Entropy of 2 apples and 3 oranges:  
         entropy(2/5,3/5) = -2/5 * log(2/5) - 3/5 * log(3/5) = 0.971
        bits
    -   Entropy of 4 apples and no oranges:    
         entropy(4/4,0/4) = entropy(1) = 0 (i.e. no mixed up)

-   Simplification trick:
    -   entropy([2/9,3/9,4/9])     
         = -2/9 * log(2/9) - 3/9 * log(3/9) - 4/9 * log(4/9)   
         = [ -2 * log(2) - 3 * log(3) - 4 * log(4) + 9 * log(9)]/9

### Iterative Dichotomization

Back to tree learning...

-   Given a bag of mixed-up stuff.
    -   Need a *measure* of "mixed-up-ness" (entropy).

-   *Split*: Find something that divides up the bag in two new sub-bags
    -   And each sub-bag is less mixed-up;
    -   Each split is the root of a sub-tree.

-   *Recurse*: repeat for each sub-bag
    -   i.e. on just the data that falls into each part of the split
        -   Need a Stop rule
        -   Condense the instances that fall into each sub-bag (report
            majority class).

### Example

Which feature *splits* generates symbols that are less mixed up?

<center>
<img width=400 
   src="../img/splits.jpg">
</center>

Which is the best attribute to split on?

-   The one which will result in the smallest tree
-   Heuristic: choose the attribute that produces the "purest" nodes
-   Purity = not-mixed-up
-   Popular impurity criterion: information gain
-   Information gain increases with the average purity \
     of the subsets that an attribute produces
-   Strategy: choose attribute that results in greatest *information
    gain*.

Compare the number of bits required to encode the splits with the number
of bits required to encode the un-split data.

-   Entropy of un-split data = entropy(9/14,5.14) =    
     -5/14 * log(5/14)/log(2) - 9/14 * log(9/14)/log(2) = 0.94
-   Entropy of the split data:
    -   Weighted sum of the entropy of the splits of size, say, 5, 4, 5
    -   5/14 * ent1 + 4/14  * ent2 + 5/14  * ent3

e.g. Outlook= sunny

-   info([2,3])= entropy(2/5,3/5) =    
    -2/5 \* log(2/5) - 3/5 \* log(3/5) = 0.971 bits

Outlook = overcast

-   info([4,0]) = entropy(1,0) =    
    -1 * log(1) - 0 * log(0) = 0 bits

Outlook = rainy

-   info([3,2]) = entropy(3/5, 2/5) =    
    -3/5 * log(3/5) - 2/5 * log(2/5) = 0.971 bits

Expected info for Outlook = Weighted sum of the above

-   info([3,2],[4,0],[3,2]) =    
    5/14 * 0.971 + 4/14 * 0 + 5/14 * 0.971 = 0.693

Computing the information gain

-   e.g. information before splitting minus information after splitting
-   e.g. gain for attributes from weather data:
-   gain("Outlook") = info([9,5]) - info([2,3],[4,0],[3,2]) = 0.940 -
    0.963 = 0.247 bits
-   gain("Temperature") = 0.247 bits
-   gain("Humidity") = 0.152 bits
-   gain("Windy") = 0.048 bits

Repeatedly split recursively:

<center>
<img width=400
    src="../img/finaltree.jpg"
	>
</center>

-   Note, final tree: not all leaves are pure
-   Splitting stops when data can't be split any further
-   Or too few examples left to split (the *-M 2* flag in J48)


## Why Use just one decision tree?

Traditional tree learners like CART and C4.5 cannot scale to Big Data problems
since they assume that data is loaded into main memory and executed within one
thread. There are many ways to address these issues such as the classic “peepholing”
method of Catlett [8]. But one of the most interesting, and simplest, is the random forest
method of Breimann [7]. The motto of random forests is, “If one tree is good, why
not build a whole bunch?” To build one tree in a random forest, pick a number m less
than the number of features. Then, to build a forest, build many trees as follows:

1. select some subset d of the training data;
2. build a tree as above, but at each split, only consider m features (selected at random); and
3. do not bother to post-prune.

Finding the right d and m values for a particular dataset means running the forests,
checking the error rates in the predictions, then applying engineering judgment to
select better values. Note that d cannot be bigger than what can fit into RAM. Also,
a useful default for m is the log of the number of features.

Random forests make predictions by passing test data down each tree. The output
is the most common conclusion made by all trees.

Random forests have certain drawbacks:

- Random forests do not generate a single simple model that users can browse
and understand. On the other hand, the forests can be queried to find the most
important features (by asking what features across all the trees were used most
as a split criteria).

Some commonly used data mining toolkits insist that all the data load into RAM
before running random forests (But it should be emphasized that this is more an issue in the typical toolkit’s implementation than
some fatal flaw with random forests).

Nevertheless, random forests are remarkably effective:

- Random forests generate predictions that are often as good as, or better than,
many other learners [7].
- They are fast. In [7], Breimann reports experiments where running it on a dataset
with 50,000 cases and 100 variables, it produced 100 trees in 11 min on a
800MHz machine. On modern machines, random forest learning is even faster.
- They scale to datasets with very large numbers of rows or features: just repeatedly
sample as much data as can fit into RAM.
- They extend naturally into cloud computing: just build forests on different CPUs.

Like C4.5 and CART, it might be best to think of random forests as a framework
within which we can explore multiple data mining methods. When faced with data that is too big to process:

– Repeat many times: 
– Learn something from subsets of the rows and features.
- Then make conclusions by sampling across that ensemble of learners.

As seen with random forests, this strategy works well for decision tree learning, but
it is useful for many other learners as well (later in this chapter we discuss an analog
of random forests for the naive Bayesian classifier).

Note that for this style of random learning to be practical, each model must be
learned very fast. Hence, when building such a learner, do not “sweat the small
stuff.” If something looks tricky, then just skip it (e.g., random forests do not do
post-pruning). The lesson of random forests is that multiple simple samples can do
better than fewer and more complex methods. Don’t worry, be happy.

A final note on random forests: they are an example of an ensemble learning
method. The motto of ensemble learning is that if one expert is good, then many
are better. While N copies of the same expert is clearly a waste of resources, N
experts all learned from slightly different data can offer N different perspectives on
the same problem. Ensemble learning is an exciting area in data mining—and one
that has proved most productive. For example:
- The annual KDD-cup is an international competition between data mining
research teams. All the first and second-placed winners for 2009–2011 used
ensemble methods.
- In our own work, our current best-of-breed learner for effort estimation is an
ensemble method [34].

References:

- [7]  Breimann, L.: Random forests. Mach. Learn. 45(1), 5–32 (2001).
DOI 10.1023/A:1010933404324
- [8] Catlett, J.: Inductive learning from subsets, or, Disposal ofexcess training data considered
harmful. In: Proceedings of the Australian Workshop on Knowledge Acquisition
forKnowledge-Based Systems, pp. 53–67 (1991)
- [34] . Kocaguneli, E., Menzies, T., Keung, J.: On the value of ensemble effort estimation. IEEE
Trans. Software Eng. 38(6), 1403–1416 (2012b). DOI 10.1109/TSE.2011.111

## Want to know More?

- More on trees http://dms.irb.hr/tutorial/tut_dtrees.php
- More on other learners:

![](../img/manyLearners.png)
