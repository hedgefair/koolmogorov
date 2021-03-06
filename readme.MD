# Koolmogorov

Koolmogorov is a hierarchical clustering and visualization library in Python based on [CompLearn](http://complearn.org/). 

Koolmogorov uses approaches, techniques, and data structures from:

- Kolmogorov Complexity (https://www.jstor.org/stable/25049284)
- Normalized Information Distance (https://arxiv.org/abs/0809.2553)
- Normalized Compression Distance (https://arxiv.org/abs/cs/0312044)
- Normalized Web Distance (https://arxiv.org/abs/0905.4039)
- Normalized Count-Min Sketch Distance (Unpublished research)
- Normalized Compression Neighbors (Unpublished research)
- Topological Data Analysis (http://brown.edu/Research/kalisnik/mapperPBG.pdf)
- Count-Min Sketch (http://dl.acm.org/citation.cfm?id=1073718)
- A New Quartet Tree Heuristic for Hierarchical Clustering (https://arxiv.org/abs/cs/0606048)
- Force-Directed Graphs (http://www.cs.cmu.edu/afs/cs/academic/class/15462-s13/www/lec_slides/Jakobsen.pdf)
- K-means Clustering based on Voronoi iteration (http://www.kanungo.com/pubs/tpami02-kmeans.pdf)

Koolmogorov is licensed under MIT.

## Installation

```
pip install koolmogorov
```

### Dependancies on external libraries

*Required*
- matplotlib
- networkx
- numpy

*Optional*
- pylzma
- snappy
- scikit-learn

*Externally loaded*
- D3.js
- Open Sans web font

## Creating distance matrices

The basis for Koolmogorov is a distance matrix. You can create your own distance matrices or use the MakeDistanceMatrix class.

```
from koolmogorov import MakeDistanceMatrix

mdm = MakeDistanceMatrix(verbose=1)
dm = mdm.ncd("data/*.*", compressor="snappy")
```

This makes a distance matrix `dm` with the normalized compression distances (using Snappy compressor) for all files in the `data/` directory.

See MakeDistanceMatrix documentation for more information.

## Creating trees

The distance matrix is used to create a tree (also called: unrooted dendrogram or phylogenetic tree).

```
from koolmogorov import MakeTree

mt = MakeTree(verbose=1, n_jobs=8)
T, score = mt.fit_transform(dm)
```

This makes a tree `T` (networkX object) and gives a fitness score `score`, using multi-threading with a Pool-size of 8.

See MakeTree documentation for more information.

## Creating visualizations

Finally we visualize the tree.

```
from koolmogorov import MakeViz

mv = MakeViz(verbose=1, method="d3")
mv.visualize(T, file_name="my_visualization")
```

This creates an HTML5/CSS3 file `my_visualization.html`, using d3.js to model the tree `T` as a force-directed graph.

See MakeViz documentation for more information.

## Examples

### Chain Letters

We download a small collection of 11 chain letters, which are all variations of a common form, called the "Good Luck" or "Saint Jude" chain letter.

Sample:
```
With love all things are possible.

This paper was sent to you for good luck.  The original is in New England.
It has been around the world nine times.  The luck has been sent to you.
You will receive good luck in the mail.  Send no money as faith as no 
price.  Do not keep this letter.  It must leave your hands within 96 hours.
```

We use Koolmogorov to create an evolutionary tree:

```
from koolmogorov import MakeDistanceMatrix, MakeTree, MakeViz

mdm = MakeDistanceMatrix()
dm = mdm.ncd("chain/*.*", compressor="snappy")

mt = MakeTree(n_jobs=80)
G, score = mt.fit_transform(dm)

mv = MakeViz()
mt.visualize(G, method="matplotlib")
```

Result:

![Chain Letter Dendrogram](http://i.imgur.com/DobvcIY.png)

### Malware families

We download a few malware families from the theZoo malware database:

- Zeus
- EquationGroup
- Cryptolocker
- Stuxnet

```
from koolmogorov import MakeDistanceMatrix, MakeTree, MakeViz

mdm = MakeDistanceMatrix()
dm = mdm.ncd("malware/*.*", compressor="bz2")

mt = MakeTree(n_iters=200, n_jobs=80)
G, score = mt.fit_transform(dm)

mv = MakeViz()
mv.visualize(G)
```

Result:

![Malware Dendrogram](http://i.imgur.com/vjuFHTB.png)

### Semantic Word Similarity with Lord of the Rings

We look at co-occurence of word tokens inside sentences of the Lord of the Rings trilogy.

```
from koolmogorov import MakeIndex, MakeDistanceMatrix, MakeTree, MakeViz

tokens = ["two",
          "three",
          "four",
          "red",
          "blue",
          "green",
          "smjagol",
          "gollum",
          "frodo",
          "ring"]

mi = MakeIndex()
ind = mi.index(mi.sentence_splitter("lotr.txt"), 
               ignore_all_but=tokens)

mdm = MakeDistanceMatrix()
dm = mdm.ncmsd(tokens, compressor=ind)

mt = MakeTree(method="native", n_jobs=8, n_iters=450)
G, score = mt.fit_transform(dm)

mv = MakeViz()
mv.visualize(G, 
             method="3Djs", 
             score=score, 
             compressor="Lord of the Rings", 
             title="Semantic Word Analysis on LOTR Trilogy")
```

Result:

![LOTR dendrogram](http://i.imgur.com/jUNd4dO.png)

Live demo: http://mlwave.github.io/tda/Semantic-Word-Analysis-on-LOTR-Trilogy.html

### Word Semantics

We use Bing search engine as a compressor,

```
from koolmogorov import MakeDistanceMatrix, MakeTree, MakeViz

mdm = MakeDistanceMatrix()
dm = mdm.nwd(["red", "white", "blue", "one", "two", "three"], compressor="bing")

mt = MakeTree(n_jobs=80)
G, score = mt.fit_transform(dm)

mv = MakeViz()
mt.visualize(G, method="d3js")
```

### Prime Number Classification

We use Yahoo search engine as a compressor to classify prime numbers.

```
import numpy as np
from sklearn import svm, cross_validation
from koolmogorov import MakeAnchorVector

# We have a train set with 10 prime numbers and 10 non-prime numbers
train = ["6029", "5953", "499", "157", "137", "71", "977", "5323", "7919", "3851",
         "4028", "5334", "501", "1780", "1020", "3173", "4121", "6215", "7023", "7407"]
y = [1] * 10
y += [0] * 10
y = np.array(y)

# For anchors we use 6 prime numbers
anchors = ["433", "919", "1123", "2099", "5233", "7589"]

# We create anchor vectors for the train set with Yahoo search engine
mav = MakeAnchorVector()
X = np.array(mav.nwd(train, anchors, compressor="yahoo"))

# We check leave-one-out performance with a Support Vector Classifier
clf = svm.SVC(probability=True, random_state=42)
kf = cross_validation.KFold(20, n_folds=20, random_state=1, shuffle=False)
for i, (train_fold, test_fold) in enumerate(kf):
	X_train = X[train_fold]
	y_train = y[train_fold]
	X_test = X[test_fold]
	y_test = y[test_fold]
	sample_test = train[test_fold]
	clf.fit(X_train, y_train)
	print(i+1, sample_test, "%f"%clf.predict_proba(X_test)[:,1][0], y_test[0])
```
We get 100% accuracy with 0.5 as a threshold:
```
(1,  '6029', '0.581687', 1)
(2,  '5953', '0.587490', 1)
(3,   '499', '0.563200', 1)
(4,   '157', '0.562242', 1)
(5,   '137', '0.559748', 1)
(6,    '71', '0.559738', 1)
(7,   '977', '0.569535', 1)
(8,  '5323', '0.585396', 1)
(9,  '7919', '0.593069', 1)
(10, '3851', '0.586171', 1)
(11, '4028', '0.447882', 0)
(12, '5334', '0.448476', 0)
(13,  '501', '0.441458', 0)
(14, '1780', '0.449435', 0)
(15, '1020', '0.447405', 0)
(16, '3173', '0.450983', 0)
(17, '4121', '0.449847', 0)
(18, '6215', '0.449977', 0)
(19, '7023', '0.450984', 0)
(20, '7407', '0.451069', 0)
```

Note how non-primes ending with an uneven number get a slightly higher probability of being a prime.

## Documentation

### MakeIndex(verbose=0, w=1000000, d=5)
| Attribute | Value |
| --- | --- |
| verbose | a non-negative integer denoting level of verbosity. Default `0` |
| w | Int. Width of the Count-Min Sketch (number of buckets). Higher leads to more accurate count estimates. Default `1000000`. |
| d | Int. Height of the Count-Min Sketch (number of rows). Higher leads to more accurate count estimates, but increases number of computations. Default `5`. |

| Function | Description |
| --- | --- |
| index(l, ignore_all_but=[]) | Indexes a corpus into a count-min sketch.<br><br>l: list of strings. List of document contents you want to index for co-occurrence.<br><br>ignore_all_but: list of tokens. When non-empty, will only look at these tokens. Default `[]`.<br><br>Returns: ind. Count-min sketch to use as a compressor for MakeDistanceMatrix.ncmsd(). |
| sentence_splitter(loc_corpus) | Helper function to turn a file directly into a list of sentences.<br><br>loc_corpus: String. Location of the file you want to split up into sentences.|
| paragraph_splitter(loc_corpus) | Helper function to turn a file directly into a list of paragraphs.<br><br>loc_corpus: String. Location of the file you want to split up into paragraphs.|

### MakeDistanceMatrix(verbose=0)

| Attribute | Value |
| --- | --- |
| verbose | a non-negative integer denoting level of verbosity. Default `0` |

| Function | Description |
| --- | --- |
| ncd(loc_files, compressor="bz2", compression_level=9) | Computes Normalized Compression Distance between a collection of files.<br><br>loc_files: glob. Location to the files under consideration. For instance: `data/*.*` for all files in the `data`-directory. <br><br> compressor: String. Any of `bz2`, `zlib`, `snappy`, `pylzma`. Using `snappy` or `pylzma` requires installation of those libraries. Default: `bz2`.<br><br>compression_level: Int. The compression level (higher is usually better for approaching Kolmogorov Complexity). When using `snappy`-compressor, the `compression_level` is ignored. Default: `9`.<br><br>Returns: distance_matrix: Dict. A Dictionary with tuples for keys and distances for values.|
| nwd(l, compressor="bing") | Computes Normalized Web Distance between a list of strings.<br><br>l: list of strings.<br><br> compressor: String. Any of `bing`, `google`, `duckduckgo`, `wikipedia`, `hackernews`. **Note:** Most search engines do not allow automated access in their Terms of Service. Though Koolmogorov does not hammer the search engines (it uses sleep to limit requests per second), search engines may not appreciate that many automated requests from a single IP. Use at your own discretion and calculate risk of a (temporary) ban. Default: `bing`.<br><br>Returns: distance_matrix: Dict. A Dictionary with tuples for keys and distances for values.|
| ncmsd(l, compressor=ind) | Computer Normalized Count-Min Sketch Distance between a list of strings.<br><br>l: list of strings.<br><br> compressor: ind. A Count-Min Sketch indexed on certain corpora using MakeIndex.<br><br>Returns: distance_matrix: Dict. A Dictionary with tuples for keys and distances for values.|


### MakeTree(verbose=0, random_state=None, n_jobs=1, n_iters=100, method="native")

| Attribute | Value |
| --- | --- |
| verbose | a non-negative integer denoting level of verbosity. Default `0` |
| random_state | Int or None. If Int set a random seed for replication. Default `None`. |
| n_jobs | Int. Number of multiple processes at once. Default `1`. |
| n_iters | Int. Number of iterations before halting. Default `100`. |
| method | String. Methodology for making the tree. `native` for Python. `MCMC` for original MCMC MQTC. The original method is both faster and more accurate, allowing you to build bigger trees in less time. Set `koolmogorov.LOC_MAKETREE` variable to the location of the MakeTree executable. Original executables can be downloaded and build from the official [CompLearn site](http://complearn.org/download.html). Note: When using `MCMC`-method other attributes like `random_state`, `n_jobs`, and `n_iters` are ignored. Default `native`. |

| Function | Description |
| --- | --- |
| fit_transform(dm) | Builds a tree from a distance matrix. <br><br>Given a dictionary with tuples for keys and distance for values, computes a networkx tree object. <br><br> dm: Dict. A Dictionary with tuples for keys and distances for values. Distances between objects themselves are not stored. Both directions should be stored. For instance, with two objects "bill" and "gates", the distance_matrix may look like: `{("bill", "gates"): 0.15, ("gates", "bill"): 0.15}` <br><br> Returns: G. NetworkX Graph Object. |

