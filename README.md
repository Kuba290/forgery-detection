# Forgery detection

## Constants declaration

*   IMAGE_SIZE - resolution for all images,
*   FEATURE_NAMES:
    *  ELA:
        * mean x3 colors,
        * standard deviation x3 colors.
    *  mean for base image x3 colors,
    *  standard deviation for base image x3 colors,
    *  skewness x3 colors,
    *  Local Binary Patterns for 10 bins.
 
## Error Level Analysis

It saves the image at a specified JPEG quality (default 90%) and then calculates the difference between the original and the compressed version to detect areas with different compression levels. If a fragment of the image was pasted from another file, it will have a different noise structure than the rest of the image. Max Pooling is used to summarize these differences and derive statistics: mean and standard deviation.

## Image Features Function

Combines the results of several methods:

* retrieves data from the ELA function,
* calculates color statistics (mean, standard deviation, skewness) in RGB space,
* uses Local Binary Patterns to analyze image texture.

## Visualization function for a modified image with a mask applied

Displays the image side-by-side, a black-and-white mask indicating the manipulation, and applies the mask to the image (as a red area).

## Confusion Table Function

Represents the performance of the classification algorithm (classifier). Each row of the table represents the possible actual labels of the units being tested, and each column represents the labels predicted by the algorithm.

## Image Resolution Histogram Function

The image collection is analyzed for image height and width.

## Image Resolution Histograms in CASIA2

The histograms feature bars grouping similar values, and a kernel density estimator (KDE), a type of nonparametric estimator designed to determine the density distribution of a random variable based on the obtained sample. It determines the values ​​of the variable under study during previous measurements.

The conclusion from this study is that the images have different resolutions, and the distribution is divergent between authentic and faked images, which requires rescaling the entire set.

## Main for CASIA2

At the beginning, the RANDOM_NUMBER variable determines the randomness of the selected images, split into training and test sets. MAX_IMAGES was used only during program development to avoid working with the entire set every time. Removing it will run all images.

Exploratory Data Analysis is used to evaluate which set the model will work on. An 80/20 split (training/testing) was applied.

Next, for `PCA_USED = True`, principal component analysis (PCA) was used to reduce features that did not meet the 95% most important criteria.

Three different classifiers are trained:

* Random Forest, for 100 decision trees, depending on the random number declared at the beginning.

It bases its decisions on the collective wisdom of many individual decision trees. During the training process, the algorithm creates a large number of trees from random subsets of data and features, so each of them views the problem slightly differently. The final classification is the result of a democratic vote, as the system selects the class indicated by the majority of the trees.

* Support Vector Machine (SVM)

Strives to find the optimal boundary (hyperplane) separating different data groups. It searches for a dividing line that provides the maximum margin between classes. The closest points to these classes are called support vectors. It can project data to higher dimensions, allowing for the efficient separation of very complex and nonlinear sets of information.

* k-Nearest Neighbors, for 5 neighbors.

Based on the assumption that similar objects are typically located close to each other in feature space, it stores all input data. When a new point appears, k-NN calculates its distance from all known examples and assigns it the class that dominates among its k nearest neighbors.

---

**Finally, we receive a report on the model's performance on the test data:**

* Accuracy - % of correct decisions, compared in a column chart depending on the classifier used,
* Recall - sensitivity to modified photos,
* Precision - how many suspected fake photos were correctly detected,
* F1-Score - harmonic mean between Precision and Recall sensitivity,
* Support - number of occurrences.

---

## Pipeline

1. Before the image is analyzed, it is loaded and transformed to a fixed resolution of `IMAGE_SIZE = (250, 250)`. This prevents the model from learning noise resulting from variable image dimensions.

2. Feature extraction - 25 per image:

    * Compression artifacts (ELA - 6 features): The image is compressed to JPEG format with 90% quality. The pixel-level difference between the original and the compressed image is calculated. The difference image is passed through Max Pooling with a 5x5 window. From this, the mean and standard deviation are calculated for each of the 3 RGB channels.

    * Base image statistics (9 features): The mean brightness, standard deviation, and skewness of the color distribution are calculated separately for each channel.

    * Local Binary Patterns texture (10 features): The image is converted to grayscale. LBP is run for a radius of 1 and 8 neighbors. The result is compressed into a 10-bin histogram illustrating the frequency of edges, corners, and flat areas.

3. The dataset is split into a training and test set 80/20. In the test phase, the algorithm uses fake and real images in a 1:1 ratio, which protects the Accuracy metric from being biased by the dominance of one class.

4. The extracted features have drastically different ranges (e.g., skewness can have values ​​around 1-2, while the sum of the LBP bins can have completely different values). They are rescaled around zero with a standard deviation of 1, which is required, especially for the correct performance of k-NN and SVM classifiers.

5. **(two versions: with and without this point)** PCA is performed on 25 features to reduce those that fall outside the 95% variance.

6. Three classifiers: Random Forest, SVM, and k-NN learn the relationships between features and the Au/Tp label. They are then tested on an independent, balanced test set.

7. Finally, the algorithm evaluates the Random Forest based on why it made a given decision. SHAP displays all features from most important to least important. For example, a dot shifted to the right for a given feature indicates that its high value strongly biased the result toward classifying the image as fake. Due to its computational complexity, this classifier was chosen over SHAP.

## SHAP

Shapley Additive exPlanations is a method for explaining model results that allows us to visualize which features play a key role in classification. It is based on game theory (Shapley values), treating each image feature as a player in a team working towards a final result.
