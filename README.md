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

<img width="389" height="411" alt="obraz" src="https://github.com/user-attachments/assets/fc0d249a-4b45-4a3f-b583-69e521be3032" />

## Image Features Function

Combines the results of several methods:

* retrieves data from the ELA function,
* calculates color statistics (mean, standard deviation, skewness) in RGB space,
* uses Local Binary Patterns to analyze image texture.

## Visualization function for a modified image with a mask applied

Displays the image side-by-side, a black-and-white mask indicating the manipulation, and applies the mask to the image (as a red area).

<img width="1183" height="483" alt="obraz" src="https://github.com/user-attachments/assets/4a05931c-187b-49dd-8009-688a109f7e8a" />
<img width="562" height="404" alt="obraz" src="https://github.com/user-attachments/assets/82a39454-9048-4ae9-b837-c36e21236578" />

## Confusion Table Function

Represents the performance of the classification algorithm (classifier). Each row of the table represents the possible actual labels of the units being tested, and each column represents the labels predicted by the algorithm.

<img width="1784" height="484" alt="obraz" src="https://github.com/user-attachments/assets/080a3c0e-c2e5-4b43-9991-89dc323cdaa9" />

## Image Resolution Histograms in CASIA2

The histograms feature bars grouping similar values, and a kernel density estimator (KDE), a type of nonparametric estimator designed to determine the density distribution of a random variable based on the obtained sample. It determines the values ​​of the variable under study during previous measurements.

The conclusion from this study is that the images have different resolutions, and the distribution is divergent between authentic and faked images, which requires rescaling the entire set.

<img width="1384" height="584" alt="obraz" src="https://github.com/user-attachments/assets/d1f5a0d7-2f1c-40f5-8d01-946895f463a9" />

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

<img width="567" height="440" alt="obraz" src="https://github.com/user-attachments/assets/09ed2b66-8726-4e69-a766-972c82d8bc2a" />
<img width="701" height="533" alt="obraz" src="https://github.com/user-attachments/assets/04229511-712a-47c7-802d-41acf3d335bf" />

## SHAP

Shapley Additive exPlanations is a method for explaining model results that allows us to visualize which features play a key role in classification. It is based on game theory (Shapley values), treating each image feature as a player in a team working towards a final result.

<img width="756" height="614" alt="obraz" src="https://github.com/user-attachments/assets/69087b77-492d-4fe8-b2c5-055121ceb37c" />

## Main for CoMoFod

### Visualisation
<img width="1009" height="509" alt="obraz" src="https://github.com/user-attachments/assets/d3a0e65d-4520-45e8-91e5-638868e805e3" />
<img width="425" height="428" alt="obraz" src="https://github.com/user-attachments/assets/bbb516d6-2612-4ff5-a38b-e68f0a1d28cd" />
<img width="1010" height="509" alt="obraz" src="https://github.com/user-attachments/assets/d4c20b84-5887-4547-978c-af90924af499" />

### Error Level Analysis
<img width="389" height="411" alt="obraz" src="https://github.com/user-attachments/assets/3fe8ffad-b651-4962-91bb-8bc720b7aa38" />

### Class image count
<img width="552" height="435" alt="obraz" src="https://github.com/user-attachments/assets/527a31ab-ca08-4a58-be99-bf29ccc0fff4" />

### Accuracy
<img width="691" height="528" alt="obraz" src="https://github.com/user-attachments/assets/2acf5550-cc1b-447e-98ad-d5b2d91a78e9" />

### Confusion matrices
<img width="1790" height="490" alt="obraz" src="https://github.com/user-attachments/assets/27cd041d-523f-4dbc-9a5f-8a7d827522f5" />

### SHAP
<img width="765" height="940" alt="obraz" src="https://github.com/user-attachments/assets/889c1df6-e999-430c-b370-0f69980a8050" />








