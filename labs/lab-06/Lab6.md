# Lab 6

### Question 1

The word game implementation you downloaded already takes words of length 5. Modify the base code to test the following words:

1. `chaos` to `order`
2. `nodes` to `graph`
3. `moron` to `smart`
4. `flies` to `swims`
5. `mango` to `peach`
6. `pound` to `marks`

To make this change, I modified the list of tuples starting on line 78 from this:

```python
for (source, target) in [('chaos', 'order'),
                             ('nodes', 'graph'),
                             ('pound', 'marks')]:
```

to this:

```python
for (source, target) in [('chaos', 'order'),
                             ('nodes', 'graph'),
                             ('moron', 'smart'),
                             ('flies', 'swims'),
                             ('mango', 'peach'),
                             ('pound', 'marks')]:
```



Running the script with our expanded list of pairs gives us the following output transformation:

```
Loaded words_dat.txt containing 5757 five-letter English words.
Two words are connected if they differ in one letter.
Graph has 5757 nodes with 14135 edges
853 connected components
Shortest path between chaos and order is
chaos
choos
shoos
shoes
shoed
shred
sired
sided
aided
added
adder
odder
order
Shortest path between nodes and graph is
nodes
lodes
lores
lords
loads
goads
grads
grade
grape
graph
Shortest path between moron and smart is
moron
boron
baron
caron
capon
capos
capes
canes
banes
bands
bends
beads
bears
sears
stars
start
smart
Shortest path between flies and swims is
flies
flips
slips
slims
swims
Shortest path between mango and peach is
mango
mange
marge
merge
merse
terse
tease
pease
peace
peach
Shortest path between pound and marks is
None
```



### Question 3

Now generate a new version that takes words of length 4 and test with: 4. `cold` to `warm` 5. `love` to `hate` 5. `good` to `evil` 5. `pear` to `beef` 5. `make` to `take`



To make the change to allow us to process 4 letter words I updated the code to the following:

```python
"""
=====
Words
=====
Words/Ladder Graph
------------------
Generate  an undirected graph over the 5757 5-letter words in the
datafile `words_dat.txt.gz`.  Two words are connected by an edge
if they differ in one letter, resulting in 14,135 edges. This example
is described in Section 1.1 in Knuth's book (see [1]_ and [2]_).
References
----------
.. [1] Donald E. Knuth,
   "The Stanford GraphBase: A Platform for Combinatorial Computing",
   ACM Press, New York, 1993.
.. [2] http://www-cs-faculty.stanford.edu/~knuth/sgb.html
"""
# Authors: Aric Hagberg (hagberg@lanl.gov),
#          Brendt Wohlberg,
#          hughdbrown@yahoo.com

#    Copyright (C) 2004-2019 by
#    Aric Hagberg <hagberg@lanl.gov>
#    Dan Schult <dschult@colgate.edu>
#    Pieter Swart <swart@lanl.gov>
#    All rights reserved.
#    BSD license.

import gzip
from string import ascii_lowercase as lowercase

import networkx as nx

#-------------------------------------------------------------------
#   The Words/Ladder graph of Section 1.1
#-------------------------------------------------------------------


def generate_graph(words):
    G = nx.Graph(name="words")
    lookup = dict((c, lowercase.index(c)) for c in lowercase)

    def edit_distance_one(word):
        for i in range(len(word)):
            left, c, right = word[0:i], word[i], word[i + 1:]
            j = ord(c) - ord('a')  # lookup[lowercase.index(c)]
            for cc in lowercase[j + 1:]:
                yield left + cc + right
    candgen = ((word, cand) for word in sorted(words)
               for cand in edit_distance_one(word) if cand in words)
    G.add_nodes_from(words)
    for word, cand in candgen:
        G.add_edge(word, cand)
    return G


def words_graph():
    """Return the words example graph from the Stanford GraphBase"""
    fh = gzip.open('words4_dat.txt.gz', 'r')
    words = set()
    for line in fh.readlines():
        line = line.decode()
        if line.startswith('*'):
            continue
        w = str(line[0:4])
        words.add(w)
    return generate_graph(words)


if __name__ == '__main__':
    G = words_graph()
    print("Loaded words4_dat.txt containing 2174 four-letter English words.")
    print("Two words are connected if they differ in one letter.")
    print("Graph has %d nodes with %d edges"
          % (nx.number_of_nodes(G), nx.number_of_edges(G)))
    print("%d connected components" % nx.number_connected_components(G))

    for (source, target) in [('cold', 'warm'),
                             ('love', 'hate'),
                             ('good', 'evil'),
                             ('pear', 'beef'),
                             ('make', 'take')]:
        print("Shortest path between %s and %s is" % (source, target))
        try:
            sp = nx.shortest_path(G, source, target)
            for n in sp:
                print(n)
        except nx.NetworkXNoPath:
            print("None")
```

The main differences were:

1. The source/target list was updated to the 4 letter words we want to find the difference between.
2. The path to the word list updated
3. The string indices changed from [0:4] for the word to [0:5]



This script generates the following output:

```
Loaded words4_dat.txt containing 2174 four-letter English words.
Two words are connected if they differ in one letter.
Graph has 2174 nodes with 8040 edges
129 connected components
Shortest path between cold and warm is
cold
wold
word
ward
warm
Shortest path between love and hate is
love
hove
have
hate
Shortest path between good and evil is
None
Shortest path between pear and beef is
pear
bear
beer
beef
Shortest path between make and take is
make
take
```

 ### Question 4

Implement a variation where we consider two words (nodes) to be  adjacent if there is a one letter difference without regard to ordering. You will need to change the edit_distance_one function to disregard  letter position. Test with the 5 letter examples above. There are  several ways to attack this. One way is to use multisets [(Counter)](https://docs.python.org/3.5/library/collections.html#collections.Counter) from the collections module, another is to use permutations from [itertools](https://docs.python.org/3/library/itertools.html?highlight=permutations#itertools.permutations). Of the two, I ***highly*** recommend itertools. The multiset implementation is far more difficult to get correct.



I updated the `edit_distance_one` function to the following:

```python
    def edit_distance_one(word):
        for perm in permutations(word):
            perm_word = ''.join(perm)
            for i in range(len(perm_word)):
                left, c, right = perm_word[0:i], perm_word[i], perm_word[i + 1:]
                j = lookup[c]  # lowercase.index(c)
                for cc in lowercase[j + 1:]:
                    yield left + cc + right
```

which iterates over every permutation of a word, and changes out one letter at a time. Running this updated `edit_distance_one` function with the original list of words of length 5 gives the following result:

```
Loaded words_dat.txt containing 5757 five-letter English words.
Two words are connected if they differ in one letter.
Graph has 5757 nodes with 112278 edges
16 connected components
Shortest path between chaos and order is
chaos
chose
chore
coder
order
Shortest path between nodes and graph is
nodes
anode
agone
anger
gaper
graph
Shortest path between pound and marks is
pound
mound
monad
moans
roams
marks
```

