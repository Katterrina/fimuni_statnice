# 7. Dobývání znalostí

> Předzpracování dat. Učení častých vzorů a asociačních pravidel. Nástroje pro strojové učení a dolování z dat (obecně + popis jednoho podrobně). Analýza temporálních dat. (PV056)

# PV056 Strojové učení a dobývání znalostí (Popelínský)

- M. Straka (MFF UK): https://ufal.mff.cuni.cz/courses/npfl129/2223-winter
- some notes from Discord (thanks!): https://docs.google.com/document/d/16RK6mwhNoIh0xQOas09_S3aeuE_eVqgDhOR3JU22jrU/edit

## Tree Learning

### Classification tree

- classifiers for instances represented as feature-vectors
    - "explainable"
- nodes for division by feature value, leaves specify the category
- continuous features split based on threshold
- https://www.youtube.com/watch?v=_L39rN6gz7Y

### Complexity of finding an optimal decision tree

- finding a minimal decision tree (nodes, leaves, or depth) is NP-hard 
- heuristic algorithms used to build a tree

### Tree induction pseudocode

```python
def decision_tree(examples,features):
    if all examples in one category:
        return leaf node (that category)
    elif features == []:
        return leaf node (most common category within examples)
    else:
        F = choice(features) # various options how to choose
        R_F = create a node
        for each possible value f_i in F:
            examples_i = [e[F] == f_i for e in examples] # subset of examples with value f_i of F
            if examples_i == []:
                N = leaf node (most common category within examples)
                E[R_F,N] = create edge marked f_i
            else:
                N = decision_tree(examples_i,features-F)
                E[R_F,N] = create edge marked f_i
        return R
```

### Impurity. Splitting criteria for discrete and continuous features

- goal: the resulting tree as small as possible
    - pick a feature that creates subsets of examples that are relatively “pure” in a single class so they are “closer” to being leaf nodes
    - impurity = leaf node with examples from more than one class
- splitting criteria
    - impurity for classes $D_1,...,D_c$: $Imp(D_1,...,D_c)=\sum_{j=1}^c\frac{|D_j|}{|D|}Imp(D_j)$
        - $Imp(D_j)$ is impurity if we choose the class $j$
        - denote $p_j$ relative frequency of label $j$ in node
        - minority class (error rate) $min(p_j,1-p_j)$
            - the proportion of misclassified examples if the node majority class is chosen
        - Gini index $p_j(1-p_j)$ 
            - ($2p_j(1-p_j)$ in slides by Popelínský, no idea why)
        - entropy $-p_j\log_2 p_j$

### Computational complexity of tree learning

- worst case: each branch tests all features
    - n examples, m features -> tree depth m
    - at each level examine the remaining m-i features for each example at the level to calculate info gains ($\sum_{i=1}^m = O(nm^2)$)
- in practice
    - learned tree is rarely complete
    - complexity linear in both m and n
    - set max depth, min number of examples in node to split, max number of leaf nodes

### Pre-pruning and post-pruning, which subtrees to prune

- used as overfitting prevention
- pre-pruning: stop growing trees at some point during top-down construction
    - when there is no longer sufficient data to make reliable decisions
    - set max depth, min number of examples in node to split, max number of leaf nodes, ...
- post-pruning: grow the full tree, then remove subtrees that do not have sufficient evidence
    - which subtrees
        - cross-validation: use validation set (part of training data) to evaluate the utility of subtrees
            - reduced error pruning: build tree, try to remove subtrees until accuracy on validation set decreases
                - problems: wasting training data on the validation set (depends on how much data we have)
            - without losing training data: several trials of reduced error pruning, remember the average complexity of the trees, build the final tree breadth-first, stop when defined complexity reached
        - statistical test: use a statistical test on the training data to determine if any observed regularity can be dismissed as likely due to random chance
        - minimum description length (MDL): is the additional complexity of the hypothesis less complex than just explicitly remembering any exceptions resulting from pruning?

### Kolmogorov complexity and Minimal description length (MDL) principle

- Kolmogorov complexity = for an object, the length of the shortest computer program that produces the object as output
    - a measure of the computational resources needed to specify the object
- minimum description length (MDL) = given data and possible theories to explain them, we want to choose a theory such that the length of the theory and data encoded using the theory is minimal
- we can use MDL for the best tree selection

### What is a regression tree

- similar to classification trees
- prediction in a leaf is an average of the examples belonging to this leaf
- performance measured e.g. by RMSE 
- https://www.youtube.com/watch?v=g9c66TUylZ4

### Regression tree impurity measures

- we want minimum variation in the nodes after the split
    - impurity of a node -> variance $Var(Y)=\frac{1}{|Y|}\sum_{y\in Y}(y-\bar{y})^2$
- the most common split measure used is just the weighted variance of the nodes
    - $Var(Y_1,...,Y_l) = \sum_{i=1}^l \frac{|Y_i|}{|Y|} Var(Y_i)$

### Evaluation. Performance measures for numeric prediction

- prediction: follow a path from the root to a leaf
- how accurate the prediction is? measure a difference between a correct value and the value in the leaf

### C4.5 vs CART

- CART binary, C4.5 not

## Bias-variance tradeoff

### Idea. Formalization

- generalization error = bias^2 + variance + irreducible error
    - formalization: $E[(y-f'(x))^2]=Bias[f'(x)]^2+Var[f'(x)]+\sigma^2$ where $f'(x)$ is model prediction and $y$ is real value
        - $Bias[f'(x)]=E[f'(x)]-f(x)$
        - $Var[f'(x)]=E[(f'(x)-E[f'(x)])^2]$
        - $\sigma^2$ irreducible error
- bias: systematic error of the model, for example, caused by erroneous assumptions in the learning algorithm
    - e.g. approximating a non-linear function using a learning method for linear models
    - high bias can cause underfitting
- variance: error caused by sensitivity to small fluctuations in the training set
    - high variance can cause overfitting

### Examples. Bias-variance for a particular learning algorithm (classification/regression)

- kNN - formula to relate bias-variance decomposition to the parameter k
        - bias - monotonically increasing function of k, 0 for k=1
        - variance - monotonically decreasing function of k
- decision trees - depth determines variance, pruned to control variance 

## Ensemble learning

- ensemble construction can be defined as a learning problem

### An ensemble of learning algorithms. Main idea

- model combination
    - train several independent models, combine results 
    - how to combine
        - regression - average
        - classification
            - voting - predicting the class predicted most often by the models (ties - randomly)
            - soft voting (averaging) - average the returned model distributions and predict the class with the highest probability
- idea - if models are independent (uncorrelated errors) and at least more accurate than random, then by averaging model predictions the errors will cancel out
    - combined decision better than individual models
        - reasonable ~10 models, for more models the gain is smaller and smaller

### Stacking

- heterogenous weak learners, trained in parallel
- combine by training a meta-model - prediction based on the weak models' predictions

### Homogenous ensembles

- same learners, changed data

#### Bagging = bootstrap aggeregation

- resample training data
- given a training set of size n, create m samples of size n by drawing n examples from the original data with replacement
- combine the m models
- decreases error for unstable learners algorithms (like decision trees) whose output can change dramatically when the training data is slightly changed
    - training for "different" data and voting is more robust

#### Boosting

- reweight training data
- learners learned sequentially in an adaptative way
    - examples have equal uniform weights in the beginning
    - model depends on the previous ones
        - focus on classifying the most highly weighted examples
        - decrease weights of correctly classified examples
    - models combined following a deterministic strategy

#### AdaBoost = adaptive boosting

```python
def train_AdaBoost(examples, base_learner, ensemble size T):
    for e_i in examples:
        w_i = 1/|examples| # weight
    H = [] # hypotheses
    for t in range(1,T):
        h_t = base_learner(examples) # learn hypothesis h_t from weighted examples
        H.append(h_t)
        error_t = sum(w_i if e_i misclassified) # error of the hypothesis h_t
        if error_t > 0.5:
            break
        beta_t = error_t/(1-error_t)
        for e_i in examples:
            if e_i classified correctly by h_t:
                w_i *= beta_t
        normalize weights # sum remains 1
    return H

def test_AdaBoost(example,H):
    for h_t in H:
        vote_t = h_t classification weighted by log(1/beta_t)
    return weighted total vote
```

### Learning with weighted examples

- possible to replicate examples in the training set proportional to their weights
- incorporate weights into a learning algorithm
    - e.g. just multiply the loss of the examples by weights
    - classification tree - count example as weight, not just 1
- weights - set manually, an inverse number of samples of a specific class, ...

### Random Forest

- an ensemble of classification or regression random trees
- one tree
    - different bootstrap sample
    - a subset of features
- majority voting 

### Ensembles and bias-variance

- bagging decreases the variance
    - variance -> variance / number of ensemble members
- boosting decreases bias
    - hypothesis complexity increasing

## Automated machine learning (AutoML)

- automating the tasks of applying machine learning to real-world problems
- goal: identify the best algorithm and hyperparameters for a given task and data

### OpenML. Meta-attributes (= meta-features) and meta-model. Landmarking

- meta-learning
    - take experiments on various datasets described by meta-features (attributes: number of examples, number of features, statistics, ...)
        - build a meta-model (e.g. decision tree)
    - characterize dataset (= compute meta-features), use it as a test example, get "the best algorithm for this task"
- landmarking - another way how to find which algorithm is the best
    - two options
        - use some fast variant of an algorithm
        - run on a small sample
    - results can be used as meta-features 
- OpenML
    - a source of datasets with meta-attributes
    - information on what people did with the data (whole pipelines)

### Average ranking (AR)

- run several algorithms on several datasets and count how often was an algorithm first, second, ...
    - compute average rank, sort, choose the best (or more of them and try 3-4)
- possible to incorporate learning time

### Bayesian optimization for AutoML

- https://towardsdatascience.com/bayesian-optimization-concept-explained-in-layman-terms-1d2bcdeaf12f
- build a probability model of the objective function 
    - use it to select hyperparameters to evaluate the true objective function
    - surrogate model = probability representation of the objective function

### Auto-sklearn

- AutoML system based on scikit-learn
- *automatically taking into account past performance on similar datasets, and by constructing ensembles from the models evaluated during the optimization* (AUTO-SKLEARN paper)
    - first step - meta-learning
        - quickly suggest some instantiations of the ML framework that are likely to perform quite well
    - Bayesian optimizer
        - start for hyperparameter spaces as large as those of entire ML frameworks, but can fine-tune performance over time
    - ensembles

## Inductive logic programming (ILP)

- https://www.doc.ic.ac.uk/~shm/ilp_theory.html
- https://people.cs.vt.edu/ramakris/Courses/CS6604/lectures/lec12/lec12.pdf
- https://www.sti-innsbruck.at/sites/default/files/courses/fileadmin/documents/intelsys09-10/11_Intelligent_Systems-InductiveLogicProgramming.pdf

### Main task, dis/advantages of ILP. When ILP is useful?

- idea: combines logic with machine learning
- main task: given a finite set of positive E+ and negative E- examples and domain knowledge B, find a general description (H, s set of hypotheses) of the whole set 
    - we want to cover (almost) all positive and no negative examples
    - we want to describe the whole set and also use the description for other examples
- advantage: flexible, does not depend on the attribute-value description of the data, background knowledge
    - attribute-value representation is insufficient when examples do not have a uniform description (e.g. different length), a structure of examples is important, ...
- disadvantage: time-consuming
- useful: example - bioinformatics (carcinogenicity)

### Logical consequence and generalization/specialization

- G |= S
    - hypothesis G is more general than hypothesis S - generalization
    - hypothesis S is more special than hypothesis G - specialization
    - any model of G is also a model of S
    - S is a logical consequence of G
    - S follows deductively from G, G follows inductively from S
- specialization/generalization operator assigns a formula to a set of all its specializations/generalizations
- deductive inference rule R maps a conjunction of clauses G onto a conjunction of clauses S such that G |= S
    - specialization rule
    - for example resolution
- inductive inference rule r in R maps a conjunction of clauses S onto a conjunction of clauses G such that G |= S 
    - generalization rule
    - not sound
    - can be obtained by inverting deductive inference rules
    - for example inverse resolution

### General refinement (specialization) operator

- specialization operator assigns to a clause a set of all its specializations
- ideal specialization operator
    - apply a substitution { X / Y } where X, Y already appear in an atom
        - binding variables
    - apply a substitution { X / f(Y1, ... , Yn)} where Yi are new variables
    - apply a substitution { X / c } where c is a constant
- the minimal set of specialization operations for logic programs without function symbols:
    - binding of two distinct variables
    - adding a most general atom into a clause body
    - with function symbols:
        - substitution of a variable with a most general term

### Generic algorithm for ILP. Covering paradigm

```python
def ILP():
    QH = initialize candidate hypotheses
    while not stop_criteria(QH):
        H = QH.pop() # how to choose H from QH influences the search strategy
        # expands that hypotheses using inference rules
        r_0, ..., r_k = choose inference rules from R
        # r_0, ..., r_k inductive or deductive rules (generalization or specialization)
        for i in range(k+1):
            H_i = apply r_i to H
            QH.add(H_i)
        prune QH # discard unpromising hypotheses from further consideration 
```

### Inverse resolution. Idea, example

- idea: we have C1 (A∨B), and resolvent (B∨C), the aim is to find C2 such that we can get the resolvent from C1 and C2
- main drawback: nondeterminism
- an example: learning family relationships (daughter, grandma, ...) from examples, e.g. female(Eva), parent(Eva, Marie), ...

### θ-subsumption

- https://cw.fel.cvut.cz/b192/_media/courses/smu/smu20_ilp_1_with_solutions.pdf
- clause G subsumes clause F if and only if G |= F or, equivalently G ⊆ F
- G θ-subsumes F iff there exists a substitution θ such that Gθ = F
- the most important framework for inductive logic programming
- the background knowledge is supposed to be empty, and the deductive inference rule corresponds to θ-subsumption among single clauses
- sound: if c1 theta-subsumes c2 then c1 |= c2

### The basic Aleph algorithm

- from https://www.cs.ox.ac.uk/activities/programinduction/Aleph/aleph.html
- select an example to be generalized (if none, stop)
- saturation: build the most specific clause (bottom clause) that entails the selected example
    - usually a definite clause with many literals
- reduction: search for a clause more general than the bottom clause
    - searching for some subset of the literals in the bottom clause that has "the best score"
- cover removal: the best-found clause is added to the current theory and all examples made redundant are removed

### Aleph input files

- positive (file extension .f) and negative (.n) examples and determinations (.b) in separate files

```
eastbound(east1).
eastbound(west1).
:- determination(eastbound/1,has_car/2).
```

---

## Vnitro

---

## Association rule mining

- https://www.upgrad.com/blog/association-rule-mining-an-overview-and-its-applications/
- association rules = simple If/Then statements
    - A ⇒ B[support, confidence]
    - A antecedent
    - B consequent
- suitable for non-numeric, categorical data
- observe frequently occurring patterns, correlations, or associations
- example: **If** a customer buys bread, **then** he’s 70% likely of buying milk.
    - antecedent: milk, consequent: bread

### Association mining: Frequent pattern (large itemset), Support, confidence, association rule.

- support
    - how frequently the if/then relationship appears in the 
    - % of transactions that contain both antecedent and consequent
- confidence
    - the number of times these relationships have been found to be true
    - % of transactions in D from those containing antecedent that contain also consequent
- frequent pattern
    - large itemset such that support ≥ minsup (predefined threshold)
- applications 
    - Market Basket Analysis: The database consists of records on past transactions - a single record lists all the items bought by a customer in one sale. Knowing which groups are inclined towards which set of items allows shops to adjust the store layout and the store catalog.
    - Medical Diagnosis: Using relational association rule mining, we can identify the probability of the occurrence of illness concerning various factors and symptoms.


### Association mining: Apriori algorithm

- task: find rules such as support >= minsup && confidence >= minconf
- https://is.muni.cz/th/aawbh/dp.pdf

```python
Algorithm:
[input: data D, support threshold minsup, confidence threshold minconf]
[output: all frequent itemsets F1,...,Fk]
generate frequent 1-itemsets F1 (support >= minsup, only counting items)
k := 1
while Fk not empty: # finding frequent itemsets
    generate Ck+1 candidates from Fk - merge elements from Fk which differ only in one item  
    remove Ck+1 element if it contains is any subset that is not in Fk
    count support for Ck+1
    Fk+1 := Ck+1 elements with support >= minsup
    k := k+1
if ABCD, AB frequent and support(ABCD)/support(AB) >= minconf
    AB -> CD[support(ABCD), support(ABCD)/support(AB)]
```

### Class-association rules

- only a single item in the consequent, a value of the target attribute
- can be used for classification
- majority voting or any other (more sophisticated) way can be used

### Association rules for pre-processing and for learning.

- pre-processing for feature construction, a frequent pattern = new feature
- for classification: class association rules

## Imbalanced data

- https://www.jeremyjordan.me/imbalanced-data/
- classification: majority and minority classes, much fewer examples in the minority classes
- imbalanced domain
    - not all values of the target variable are equally important
    - the more important values are scarcely represented in the training data
- relevance function
    - A relevance function $\phi(Y): domain(Y) \rightarrow [0,1]$ is a function that expresses the application-specific bias concerning the target variable Y domain by mapping it into a $[0, 1]$ scale of relevance, where 0 and 1 represent the minimum and maximum relevance, respectively
    - applicable in both classification and regression

### Performance measures for imbalanced data.

- standard performance metrics (e.g. accuracy, error rate) assume that all instances are equally relevant for the model performance
- F-measure
    - $F_{\beta}=\frac{(\beta^2+1)\cdot Prec\cdot Rec}{\beta^2\cdot Prec+Rec}$
    - $Prec=\frac{TP}{TP+FP}$, $Rec=\frac{TP}{TP+FN}$
- AUC
    - area under the ROC curve (FPRate vs TPRate for various decision thresholds)
- Gmean
    - $Gm = \sqrt{sensitivity \times specificity}$
    - $sensitivity = \frac{TP}{TP+FN}$, $specificity = \frac{TN}{TN+FP}$

### Pre-processing imbalanced data. Undersampling, oversampling.

- we can alter the dataset to remove an imbalance
    - advantages
        - possible to use any learning algorithm
        - the obtained model will be biased to the goals of the domain
        - interpretable model
    - disadvantages
        - the difficulty of relating the modifications in the data distribution and domain preferences
        - mapping the given data distribution into an optimal new distribution according to domain goals is not easy 
- undersampling
    - random
        - remove some random examples from the majority class to achieve balanced classes
    - near miss
        - keep the points from the majority class necessary to distinguish between other classes
    - removing Tomeks links
        - Tomek’s link exists if two observations of different classes are the nearest neighbors of each other
        - remove any observations from the majority class for which a Tomek's link is identified
        - won't achieve a balance among the classes - but still useful for cleaning
    - edited nearest neighbors 
        - run the nearest-neighbors algorithm and “edit” the dataset by removing samples that do not agree “enough” with their neighborhood
- oversampling
    - random
        - randomly sample the minority classes and simply duplicate the sampled observations
        - ! artificially reducing the variance of the dataset
    - Synthetic Minority Over-sampling Technique (SMOTE)
        - generates new observations for the minority classes by interpolating between observations in the original dataset
    - Adaptive Synthetic (ADASYN) sampling
        - the number of samples generated for a given x is proportional to the number of nearby samples which do not belong to the same class as x
- it is possible to combine oversampling and undersampling


### Modification of classifiers to be able to handle imbalanced data. Cost-sensitive learning.

- change the learning algorithms so they can learn from imbalanced data
    - advantages
        - domain goals are incorporated directly into the models by setting an appropriate preference criterion
        - interpretable model
    - disadvantages
        - restricted to that specific set of modified learning algorithms
        - requires a deep knowledge of algorithms
        - if the preference criterion changes, models have to be relearned and, possibly the algorithm has to be re-adapted
        - not easy to map the domain preferences with a suitable preference criterion
- **cost-sensitive learning**
    - instead of creating balanced data distributions through different sampling strategies, cost-sensitive learning targets the imbalanced learning problem by using different cost matrices that describe the costs for misclassifying any particular data example
    - class weight - different weights for different classes, just scale loss function

## Outliers 

- applications: fraud detection, medicine (unusual symptoms), measurement errors detection
- point outliers
    - individual/ small groups are very different from the others (e.g. credit card fraud)
- contextual 
    - outlier only when taking the context into account (e.g. temperature time series)
- collective 
    - individual cases cannot be considered strange, but together with other associated cases are clearly outliers (e.g. human cardiogram)

### Detection methods

- statistical
    - normal data objects generated by a statistical (stochastic) model, data not following the model are outliers
- proximity-based
    - the proximity of an object to its neighbors significantly deviates from the proximity of most other objects to their neighbors in the same dataset 
    - distance-based, density-based, clustering-based
- ABOD – angle-based outlier degree 
    - object o is an outlier if most other objects are located in similar directions
    - object o is no outlier if many other objects are located in varying directions

### Outlier detection method types

- Supervised 
    - building a predictive model for normal vs. anomaly classes (problem is transformed to classification problem) e.g.: any supervised learning algorithm (decision tree)
    - problems:
        - far fewer anomalous instances than normal instances
        - it is challenging to obtain accurate labels for the anomaly class

- Semi-supervised 
    - training data has labeled instances only for the normal class e.g.: one-class learning: one-class SVM, clustering: EM algorithm (Normal data instances are close to their closest cluster centroid. Anomalies are far from their closest cluster centroid.)

- Unsupervised 
    - no labels, most widely used 
    - assumption: normal instances are far more frequent than anomalies in the test dataset and they make clusters 
    - e.g.: proximity-based methods, clustering LOF, Local Outlier Factor

### LOF (Local Outlier Factor)

- LOF >> 1 -> outlier
- $dist_k(o)$ - distance from o to its k-th nearest neighbor
- $reachdist_k(o,p) = max(dist_k(p), d(o,p))$
- local reachability distance - $lrd$
    - an inverse of the average reachability-distance of k-neighborhood
    - $lrd(o) = 1/(\sum_{p \in kNN(o)} reachdist_k(o,p)/k)$
- $LOF(o)=(\frac{1}{k}\sum_{p\in kNN(o)}lrd(p))/lrd(o)$ 

## Class-based outliers

- An outlier is an observation that deviates so much from the other observations as to arouse suspicions that it was generated by a different mechanism. [Hawkins 1980]
- two needs:
    - detect, then remove and run again
    - detect, then analyze
- detection methods
    - multi-class outlier detection
        - learn a model for each normal class, if the data point doesn’t fit any model, it is declared an outlier
        - advantage: easy to use, disadvantage: some outliers can’t be detected
    - semantic outliers
        - similarity between example and other examples in the class

### Class-based outliers (CBO). Idea. Example of a task solved by CBO.

- CBO
    - look anomalous when the class labels are taken into account
    - do not have to be anomalous when the class labels are ignored
    - sometimes called ‘semantic outliers’
- class-based outlier factor
    - COF = OF w.r.t. own class (+ const * ) OF w.r.t. the other class/es
        - OF - various definitions

### Multi-class outlier detection

- learn a model for each normal class (one model detecting one class), if the data point doesn’t fit any model, it is declared an outlier
- advantage: easy to use
- disadvantage: some outliers can’t be detected

### CODB (Class Outlier Distance Based)

- a combination of distance-based and density-based approach w.r.t class attribute, no need for clustering
- $COF(T) = K \cdot P_{class}(T,K) + \alpha \cdot 1/dev(T) + \beta \cdot dist_k(T)$
    - $P_{class}(T,K)$ probability of the class label of T w.r.t. the K nearest neighbors (similarity to the K nearest neighbors)
    - $dev(T)$ sum of the distance from all other elements from the same class

### Random Forest for outlier detection. Proximity matrix. RF-OEX

- https://www.stat.berkeley.edu/~breiman/RandomForests/cc_home.htm#outliers
- an approach described below is called RF-OEX
- proximity matrix calculation
    - creating a random forest
    - after each tree is built, all of the data are run down the tree, and proximity values are computed for each pair of cases
        - two cases occupy the same terminal node -> proximity +1
    - proximity values normalized by dividing by the number of trees
    - the average proximity is computed for each instance
- outlier factor for an instance p is computed as a sum of three different measures of outlierness – inverse proximity to the members of the same class OF1, misclassification measure (proximity to the members of other classes) OF2, and ambiguity measure OF3
    - OF(p) = OF1(p)+OF2(p)+OF3(p)
    - OF1(p) = 1/sum(if class(p)==class(q) proximity(p,q))
    - OF2(p) how many close points are not in the same class as p (scaled)
    - OF3(p) increases the importance of outliers that are far from all points

### Outlier explanation. Choose two methods.

- understand why an instance is detected as an outlier
- what is a meaningful explanation
    - helpful for a user, namely easy to understand
        - e.g. the smallest subset of attributes
    - efficient, scalable
- explaining outliers by subspace separability
    - look for a subspace A where the outlier factor is high and the dimension of A is low
    - separability - instance outlierness is related to its separability from the rest of the data
    - separability as an error in classification
        - assume that the data follows a distribution f
        - original data = inlierclass; outlier + artificial points = outlierclass
        - use standard feature selection methods to find explanatory subspaces - selected features explainable
- RF-OEX: Analysis of Random Forest
    - instead of classes use just non-outlier/outlier
    - first option:
        - search for frequent branches 
    - second option:
        - reduction of trees

## Dataset hardness. 

- level of difficulty in accurately modeling or predicting outcomes using the dataset
- a variety of factors
    - the size of the dataset
    - the quality of the data
    - distribution of the data
    - complexity of the relationships between the variables in the dataset

### Dataset hardness measures: idea, 3 main factors of dataset hardness.

- idea: an estimate of the difficulty in separating the data points into their expected classes

**Main factors on dataset hardness**

- ambiguity of the classes
    - the classes can not be distinguished using the data, regardless of the classification algorithm employed
- sparsity and dimensionality of the data
    - an incomplete or sparse dataset -> some input space regions underconstrained
        - data in those regions are classified arbitrarily
- complexity of the boundary separating the classes 
    - relates to the size of the smallest description needed to represent the classes
    - worst case: list all the objects along with their labels

### 6 groups of measures. What measures do you know? Describe one from each group in detail.

1. Feature overlapping measures
    - evaluate the discriminative power of the features
    - **Volume of the overlapping region measure (F2)** calculates the overlap of the distributions of the values of the features within the classes. F2 can be determined by finding, for each feature fi, its minimum and maximum values in the classes. The range of the overlapping interval is then calculated, and normalized by the range of the values in both classes. The higher the F2 value, the greater the amount of overlap between the problem classes. Therefore, the problem’s complexity is also higher. And if there is at least one non-overlapping feature, the F2 value should be zero.
2. Linearity measures 
    - quantify whether the classes can be linearly separated
    - **Error rate of linear classifier measure (L2)** computes the error rate of the linear SVM classifier.
3. Neighborhood measures 
    - characterize the presence and density of the same or different classes in local neighborhoods
    - **Error rate of the nearest neighbor classifier (N3)** refers to the error rate of a 1NN classifier that is estimated using a leave-one-out procedure. High N3 values indicate that many examples are close to examples of other classes, making the problem more complex.
4. Network measures
    - extract structural information from the dataset by modeling it as a graph
    - **Average density of the network (Density)** measure gives the number of edges that are retained in the graph built from the dataset normalized by the maximum number of edges between n pairs of data points. Graph building: Each example from the dataset corresponds to a node of the graph, whilst undirected edges connect pairs of examples and are weighted by the distances between the examples - nodes i and j are connected only if dist(i, j) < e, weight = 1/distance.
        - density = 2|E|/(|V|(|V|-1))
5. Dimensionality measures
    - data sparsity based on the number of samples relative to the data dimensionality
    - **Average number of points per dimension measure (T2)** divides the number of examples in the dataset by their dimensionality, T2=n/m.
6. Class balance measures
    - a ratio of the number of examples between classes
    - **Entropy of class proportions measure (C1)** captures the imbalance in a dataset. 
        - $C1=-\frac{1}{log(n_c)}\sum_{i=1}^{n_c}p_i log(p_i)$

## Instance hardness. 

- refers to the difficulty of correctly classifying individual instances in a dataset
    - some instances may be easy to classify correctly, while others may be more difficult 
- reasons:
    - noise in the data
    - overlap between classes
    - the presence of outliers
- which instances and why are misclassified, how they contribute to data set complexity
    -  improve the learning process and could guide the future development of learning algorithms and data analysis methods

### Instance hardness measures: idea, theoretical model. Classifier output difference.

- theoretical model 
    - each instance in a data set has a hardness property that indicates the likelihood that it will be misclassified
    - e.g. outliers and mislabeled instances are expected to have high instance hardness since a learning algorithm will have to overfit to classify them correctly
    - what is the probability of instance misclassification?
- instance $<x_i,y_i>$, quantity $p(y_i | x_i, h)$ measures the probability that $h$ assigns the label $y_i$ to the input feature vector $x_i$
    - larger $p(y_i | x_i, h)$ -> higher probability of correct classification
    - definition of instance hardness with respect to h: $IH_h(〈x_i, y_i〉) = 1 − p(y_i|x_i, h)$
        - simply an inverse of the probability of assigning the correct label
-  Classifier Output Difference (COD) 
    - unsupervised meta-learning
    - the distance between two learning algorithms - the probability that the learning algorithms make different predictions 

### What measures you know, one in detail. Indicator function and classifier scores.

- we want an approximation of $p(y_i | x_i, h)$
    - Indicator function - frequency of an instance being misclassified
    - Classifier scores - aggregation of some scores from the used classifiers (i.e. accuracy of samples reaching the leaf node in decision trees)
- measures 
    - k-Disagreeing Neighbors 
        - measures the local overlap of an instance in the original task space in relation to its nearest neighbors
        - the percentage of the k nearest neighbors (using Euclidean distance) for an instance that does not share its target class value
    - Tree Depth 
        - description length of an instance in an induced C4.5 decision tree


## Active learning

### Active learning. Definition. Uncertainty sampling. Query by committee. Applications

- https://en.wikipedia.org/wiki/Active_learning_(machine_learning)
- goal: reduce the number of labeled examples needed for learning
- idea: learner actively chooses which examples to label
    - how to choose?
        - query synthesis
        - selective sampling - for each point in the stream, label or discard
        - pool-based - a pool of examples, the learner chooses for labeling

**Uncertainty sampling**
- pool-based active learning
    - given a pool of unlabeled data, the learner chooses from the pool which to label next
- query the sample x that the learner is most uncertain about
    - uncertainty measures:
        - maximum entropy
        - the smallest margin between the most likely and second most likely label
        - least confidence

**Query by committee**
- set (=commitee) of classifiers 
- query the instance that the committee members disagree
- how to get a committee
    - ensemble
    - sample model from P(tau|L)
- how to measure disagreement
    - number of disagreeing pairs
    - vote distribution as probabilities - use uncertainty measure

**Applications**
- if manual labeling is complicated
    - time-consuming, e.g., document classification
    - expensive, e.g., medical decisions (need doctors)
    - dangerous, e.g., landmine detection

### Semi-supervised learning. Definition. How useful it is. Applications

- goal: using both labeled and unlabeled data to build better learners
- unlabeled data is cheap, labeled data can be hard to get
- *Does unlabeled data always help?* Unfortunately, this is not the case, yet.

**Self-training algorithm**
- assumption: high confidence predictions are correct
- train on labeled -> predict unlabeled -> add those predictions as labeled data to training set -> repeat. 
- variations: add only most confident/add all/add all and weight by confidence
- advantages: simple, applies to existing classifiers, often used in NLP
- disadvantages: early mistakes can reinforce themselves, and we don't know if it will converge

**Generative models**
- figure out from which distribution the data come
- add unlabeled data to the model with the highest probability
- examples
    - Mixture of Gaussian distributions (GMM)
    - Mixture of multinomial distributions (Naive Bayes)
    - Hidden Markov Models (HMM)
- advantages: well-studied probabilistic framework, can be extremely effective
- disadvantages: difficult to verify the correctness of the model, and unlabeled data may hurt if the generative model is wrong

**Cluster-and-label**
- run clustering, label all points within a cluster by the majority of labeled points in that cluster
- advantages: simple
- disadvantages: can be difficult to analyze

**S3VM**

- assumption: Unlabeled data from different classes are separated by a large margin.
- enumerate all possible labelings -> build SVM -> choose SVM with the largest margin (avoid unlabeled data in the margin)
- advantages: applicable wherever SVMs are applicable, clear mathematical framework
- disadvantages: optimization is difficult, and can be trapped in bad local optima

### Supervised, semi-supervised and active learning. Comparison.

- semi-supervised and active learning goal is the same: good learning performance (e.g., classification accuracy) without demanding too many labeled examples
- supervised: easy - we have labeled examples
- semi-supervised: use unlabeled data
- active learning: choose examples to label

## Sequence mining

**Definitions**
- let $L=\{i_1,i_2,...,i_n\}$ be a set of items, itemset is a subset of L
- sequence is defined as an ordered list of itemsets
    - $s=s_1s_2...s_l$ where each $s_i$ is an itemset
    - $s_i$ is also called an element or a transaction
        - denoted $(x_1x_2...x_m)$ where each $x_j$ is an item 
        - denoted $x_1$ is single-element transaction
        - string = sequence with transaction length 1
- $a=a_1a2...an$ is a subsequence of sequence $b=b_1b_2...bm$ if there exist integers $1\leq j_1<j_2<...<j_n<m$ such that $a_i \subseteq b_{j_i}$
    - denoted $a\sqsubseteq b$, $b$ called super-sequence of $a$
- sequence database $S$ is set of tuples $(id,s)$ where $s$ is a sequence
- support of sequece $a$ in database $S$ is defined as $support_S(a)=|\{(id,s)|(id,s)\in S \land a \sqsubseteq s\}|$
- sequence motifs - short distinctive sequence pattern shared by multiple related sequences

### Sequence mining. Idea. Frequent and closed frequent patterns. GSP algorithm.

- idea: finding statistically relevant patterns between data examples where the values are delivered in a sequence
    - discrete steps, different from time series
- sequence $a$ is called a sequential pattern in sequence database $S$ if $support_S(a) \geq minsup$
    - predefined support threshold $minsup$
    - a sequential pattern $s$ is called closed if there is no super-pattern $s'$ such that $s \sqsubset s'$ and $support(s)=support(s')$

**GSP algorithm** (apriori-based)
- initial candidates: all singleton sequences, initial support: count sequences
- generate length-2 candidates
- repeat for each length:
    - scan DB to find support
    - generate length+1 candidates
     

### Sequence features and distance functions. Representation of sequences

**Sequence features**
- possibilities for how to describe/represent sequence
- k-grams
- k-gapped pairs
    - pair of items with a gap of length k
- frequent/closed patterns
- boolean features – presence / absence
- numerical features - counts of distinct items 

**Distance functions**
- edit distance (Levenshtein distance)
    - minimum number of edit operations to transform one sequence to another
    - operations: change, delete, insert
- Hamming distance
    - number of positions where the two sequences differ
    - (sequences of the same length)

### Constraints in sequence mining.

- searching for sequences according to the constraints
- categories of constraints
    - item constraint – which items should or should not be present in patterns
    - length constraint – requirements on the number of items or transactions in patterns
    - super-pattern constraint – which patterns should be contained in the resulting patterns
    - aggregate constraint – requirements on sum, avg, max, min, ... over items of each pattern
    - regular expression constraint – using operators such as disjunction or Kleene closure
    - duration constraint – requirements on a time difference between the first and the last transaction in the pattern
    - gap constraint – min and max gap between transactions in patterns
- constraint-based sequential pattern mining 
    - How to mine sequences? Some useful properties of constraints:
        - anti-monotonicity
            - if S violates c, the super-sequences of S also violate c
            - examples: sum(S) < 150, min(S) > 10
        - monotonicity
            - if S satisfies c, the super-sequences of S also do so
            - examples: count(S)>5, there is an element x in S
        - succincity
            - we can enumerate all and only those sequences that are guaranteed to satisfy the constraint, even before support counting begins
            - example: there are elements x,y in S

### Mining partial orders (only for an oral exam)

## Time series

### Main Time Series Data Mining Tasks

- indexing (query by content)
    - given a query time series and a similarity measure, find the most similar time series in the database
- clustering
    - Find the natural groupings of a set of time series in a database using some similarity measure 
- classification
    - given an unlabelled time series, assign it a label from a set of pre-defined labels 
- prediction
- anomaly detection
- segmentation 

### Types of Variation. Stationarity

- **seasonal variation**
    - Some time series exhibit a variation that is annual in the period, e.g. demand for ice cream.
- **other cyclic variation**
    - Some time series have periodic variations that are not related to seasons but to other factors, e.g. some economic time series.
- **trends**
    - A trend is a long-term change in the mean level of the time series.

**Stationarity**

A time series is said to be stationary if there is no systematic change in mean (no trend), if there is no systematic change in variance, and if strictly periodic variations have been removed.
- In these cases, statistics like mean, standard deviation, variance, etc., bring relevant information!

**Time plots**
- show important features like trends, seasonality, outliers, and discontinuities
- transformations 
    - might be useful to reveal new information from time plots
    - variance stabilization
        - when: trend with the variance increasing with the mean
        - solution: logarithm
    - make the seasonal effects additive
        - when: there is a trend and the size of the seasonal effect increases with the mean (multiplicative seasonality) 
        - solution: logarithm
    - remove trend
        - when: there is a systematic change in the mean
        - solution 1: first-order differentiation
        - solution 2: model the trend and subtract it from the original series

### Tests of Randomness

- test the hypothesis that the observed time series is random
    - correlogram
    - runs test https://home.ubalt.edu/ntsbarsh/business-stat/opre504.htm#rrunstest

**Runs test algorithm**
- let's have sample S of length n
- compute the mean of the sample
- going through the sample sequence, replace any observation with +, or - depending on whether it is above or below the mean (ties discarded)
- compute
    - $R$ - number of runs (run = maximal subsequence of like elements)
    - $n^+$ - number of +
    - $n^-$ - number of -
- compute the expected mean and variance of R
    - $\mu = 1 + \frac{2n^+n^-}{n}$
    - $\sigma^2 = \frac{(\mu-1)(\mu-2)}{n-1}$
- compute $z=(R-\mu)/sigma$
- conclusion
    - $z>Z_{\alpha}$: there might be cyclic, seasonality behavior (under-mixing)
    - $z<-Z_{\alpha}$: there might be a trend
    - $z>Z_{\alpha/2}$ or $z<-Z_{\alpha/2}$: reject the randomness

### Measuring Similarity. Types of matching. Types of Distance Functions

- most time series data mining tasks require the similarity between series to be asserted
- types of matching
    - whole matching
        - the whole query matched against all time series in the database
    - subsequence matching
        - all time series in the database searched for a subsection matching the query
- metric
    - symmetry: $D(X,Y)=D(Y,X)$
    - self-similarity: $D(X,X)=0$
    - positivity: $D(X,Y)=0 \iff X=Y$
    - triangular inequality: $D(X,Y) \geq D(X,Z)+D(Z,Y)$
- types of distance functions
    - metric
        - Euclidean metric, correlation, ...
        - $D(X,Y)=(\sum_{i=1}^n|x_i-y_i|^p)^{(1/p)}$
            - $p=1$ Manhattan, $p=2$ Euclidean
    - non-metric
        - time warping, LCSS, ... 

### Dynamic Time Warping. LCSS Longest common subsequence

**Dynamic Time Warping**
- main ideas
    - allows local deformations (stretch and shrink) along the time
    - able to handle series of different lengths
- rules
    - first index to the first index, last to last (but possible to map more indices ant one)
    - all indices in both sequences must be matched
    - the mapping of the indices from the first sequence to indices from the other sequence must be monotonically increasing 

**DTWDistance algorithm**
```python 
[input: s: array [1..n], t: array [1..m]]
[output: int]

DTW := array [0..n, 0..m]
for i := 0 to n
    for j := 0 to m
        DTW[i, j] := infinity
DTW[0, 0] := 0
    
for i := 1 to n
    for j := 1 to m
        cost := d(s[i], t[j]) # distance between the values
        DTW[i, j] := cost + minimum(DTW[i-1, j  ],    // insertion
                                    DTW[i  , j-1],    // deletion
                                    DTW[i-1, j-1])    // match
    
return DTW[n, m]
```

**LCSS - Longest common subsequence algorithm**
```python 
lcs(X,Y):
    if (len(X) == 0 || len(Y) == 0)
        return 0;
    if (X[0] == Y[0])
        return 1 + lcs(X[1:], Y[1:]);
    else
        return max(lcs(X, Y[1:]),
                   lcs(X[1:], Y));
```

### Evaluation Methods. Correct Evaluation of Time Series Models. Metrics for Evaluating Predictive Performance

- Why evaluation? 
    - obtain a reliable estimate of some evaluation measure
        - a high probability of achieving the same score on other samples of the same population
- evaluation measures
    - **predictive accuracy**
    - model size
    - computational complexity
- time series data problem: Any form of resampling changes the natural order of the data!

**Correct Evaluation of Time Series Models**
- do not "forget" the time tags of the observations
- do not evaluate the model on past data
- possible methods
    - fixed learning window
        - fixed split, past observations for training, "fake future" data for testing
    - growing window
        - train, test, add the tested window to the model, train again...
    - sliding window 
        - train on a window, test on the next window, train on the tested window, test on the next...

**Metrics for Evaluating Predictive Performance**

- absolute measures
    - MSE (mean squared error)
        - $\frac{1}{n}\sum_{i=1}^n(pred_i-y_i)^2$
    - MAD (mean absolute deviation)
        - $\frac{1}{n}\sum_{i=1}^n|pred_i-y_i|$
- relative measures
    - Thiel coefficient
        - $\sqrt{\sum_{i=1}^n(pred_i-y_i)^2}/\sqrt{\sum_{i=1}^n(y_i-y_{i-1})^2}$
    - MAPE (mean absolute percentage error)
        - $\frac{1}{n}\sum_{i=1}^n|(pred_i-y_i)/y_i|$

### Assumptions of Classical Linear Approaches. Moving Average Models. Exponential Moving Average Models.

- "Classical" Linear Approaches
    - assumptions
        - linearity - the model of the time series behavior is linear on its inputs
        - stationarity - the underlying equations governing the behavior of the system do not change with time
- Moving Average Models
    - https://www.projectpro.io/article/moving-average-time-series-model/716
    - a moving average of order $q$, MA(q) is time series defined as
        - $Y_t = \sum_{i=0}^q \beta_i X_{t-i}$
- Exponential Moving Average Models
    - $EMA_{\alpha}$
        - $Y_1 = X_1$
        - $Y_t = \alpha X_t + (1-\alpha) EMA_{\alpha}(X_{t-1})$

### Autoregressive (AR) Models. ARMA. ARIMA

- Autoregressive (AR) Models
    - an autoregressive model of order p is a series given by $Y_t=\sum_{i=0}^p\alpha_iY_{t-i}$
- Mixed Autoregressive and Moving Average Models (ARMA)
    - a mixed ARMA model of order p, q is a series given by $Y_t = \sum_{i=0}^p\alpha_iY_{t-i} + \sum_{i=0}^q \beta_i X_{t-i}$
- An integrated ARMA (ARIMA)
    - does not assume stationary series (the previous ones do)
    - https://en.wikipedia.org/wiki/Autoregressive_integrated_moving_average
    - https://www.youtube.com/watch?v=3UmyHed0iYE
    - integrated - 1st order $z_t=q_{t+1}-q_t$

### Clustering.

- types
    - whole time-series clustering
        - clustering a set of individual time series with respect to their similarity
    - subsequence clustering
        - clustering a set of subsequences of a time series that are extracted via sliding window 
        - clustering segments of a single long time-series
    - time point clustering
        - clustering time points based on a combination of their temporal proximity and similarity of the corresponding values
        - not all points need to be clustered, some are considered noise 
- https://www.heka.ai/en/our-publications/time-series-clustering
- grouping raw sequences based on distances
    - Euclidean distance, DTW
    - rarely used
- preprocessing and distance clustering
- simplified representation clustering 
    - for example FFT, "most important" coefficients
- model-based clustering
- feature (characteristics) based clustering

**Algorithms**

- Agglomerative hierarchical clustering: considers each item as a cluster, and then gradually merges the clusters (bottom-up)
- Divisive hierarchical clustering: starts with all objects as a single cluster and then splits the cluster to reach the clusters with one object (top-down)


### Discretization

- PAA: Piecewise Aggregate Approximation
    - divide the original time series into M equally sized frames and compute the mean values for each frame
- SAX: Symbolic Aggregate approXimation
    - https://jmotif.github.io/sax-vsm_site/morea/algorithm/SAX.html
    - transform the original time series into the PAA representation, and converts it into a string using a lookup table
- TVA: Combining Trend-Based and Value-Based Approximations

## Theory of ML. Five main tasks

We seek theory to relate:
- the probability of successful learning
- number of training examples
- complexity of the hypothesis space
- accuracy to which the target concept is approximated
- how are training examples presented

### Two roles of Bayesian learning

- practical learning algorithm
- conceptual framework
    - the gold standard for evaluating other learning algorithms

### VC-dimension

- a measure of the capacity (complexity, expressive power, richness, or flexibility) of a set of functions that can be learned by a statistical binary classification algorithm
- the size of the largest set of instances that can be shattered by particular hypotheses language or model class
    - examples: 
        - linear classifier in d dimensions has VC-dimension d+1
        - kNN for k=1 has infinite VC-dimension

### Probably approximately correct (PAC) learning

- the concept class C is PCA learnable iff the VC-dimension of C is finite
- https://en.wikipedia.org/wiki/Probably_approximately_correct_learning

### Kolmogorov complexity and ML

- https://theorangeduck.com/page/machine-learning-kolmogorov-complexity-squishy-bunnies
- Kolmogorov complexity of an object, such as a piece of text, is the length of the shortest computer program (in a predetermined programming language) that produces the object as output. It is a measure of the computational resources needed to specify the object.
- Neural Networks are not just good for things we don't know how to solve, they can provide massive performance gains on problems we already know how to solve. In fact, we can use the concept of Kolmogorov Complexity to get a kind of intuition (and even use PCA for a simple kind of measure) for how well we expect Neural Networks to perform on a task.

### Monte Carlo methods in ML

- what is it used for
    - resampling
    - random hyperparameter tunning
    - stochastic optimization
        - simulated annealing
