# The Elements of Statistical Learning: 2. Overview of Supervised Learning

Supervised Learning is the machine elarning task of learning a function tat maps an input to an output based on example input-output pairs.

Supervised learning steps:
1. **Determine the type of training examples**. Before doing anything else, the user should decide what kind of data is to be used as a training set. In the case of handwriting analysis, for example, this might be a single handwritten character, an entire handwritten word, or an entire line of handwriting.
2. **Gather a training set**. The training set needs to be representative of the real-world use of the function. Thus, a set of input objects is gathered and corresponding outputs are also gathered, either from human experts or from measurements.
3. **Determine the input feature representation of the learned function**. The accuracy of the learned function depends strongly on how the input object is represented. Typically, the input object is transformed into a feature vector, which contains a number of features that are descriptive of the object. The number of features should not be too large, because of the curse of dimensionality; but should contain enough information to accurately predict the output.
4. **Determine the structure of the learned function and corresponding learning algorithm**. For example, the engineer may choose to use support vector machines or decision trees.
5. **Complete the design**. Run the learning algorithm on the gathered training set. Some supervised learning algorithms require the user to determine certain control parameters. These parameters may be adjusted by optimizing performance on a subset (called a validation set) of the training set, or via cross-validation.
6. **Evaluate the accuracy of the learned function**. After parameter adjustment and learning, the performance of the resulting function should be measured on a test set that is separate from the training set.

## 2.1 Variables Type and Terminology
- Output type:
	1. Quantitative variable / numeric
	2. Qualitative variable / categorical / discrete variables / factor
	3. Ordered Categorical (ex, small, medium, large), there is an ordering between values, but no metric notion is appropriate.

The distinction of outputs type has led to a naming convention for the prediction tasks: 
	- Regression - when target is quantitative outputs
	- Classification - when target is qualitative outputs.

Qualitative variables are typically represented numerically by codes. For example, 'Success' and 'Failures' are often represented with 1 and 0. Another example, is 'Survived' and 'Died' with 1 and -1. For reasons that will become apparent, such numeric codes are sometimes referred to as *targets*. When there are more than 2 categories, several alternatives are available. The most useful and commonly used coding is via *dummy variables*.

$\text{S}_1(N) = \sum_{p=1}^N \text{E}(p)$
$$y-y_0=m(x-x_0)$$

latexImg = function(latex){
    link = paste0('http://latex.codecogs.com/gif.latex?',
           gsub('\\=','%3D',URLencode(latex)))
    link = gsub("(%..)","\\U\\1",link,perl=TRUE)
    return(paste0('![](',link,')'))
}

$\lambda{}$.
