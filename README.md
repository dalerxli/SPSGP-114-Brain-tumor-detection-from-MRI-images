# Brain-tumor-detection-from-MRI-images

## Brain tumor detection from MRI images

## Goal and Background  


![pic 1](https://user-images.githubusercontent.com/66870226/102860441-e501b200-4453-11eb-979c-27f3e083d89c.png)


- The goal of this project is to examine the effectiveness of symmetry features in detecting tumors in brain MRI scans.
- The normal human brain exhibits a high degree of symmetry. 
- When a brain tumor is present, however, the brain becomes more asymmetric.
-  symmetry features from the brain and training a classifier on these features should prove useful in locating tumors in the brain.
- This project is an extension of a project I have been working on this and last semester.
- The project involves detection of small and multiple tumors from the proposed
dataset using blob detection. 
- Tumors in the brain tend to be relatively compact and spherical.

## Dataset

- The dataset used for this project consists of 20 patient MRI scans provided by Dr. Dan Nguyen from the Milton S. Hershey Medical Center.
- These scans were obtained using a Phillips Intera
-1.5 Tesla Magnet with a resolution of 231x185x200 pixels and 1mm slice thickness. 
- They are T1 gadolinium-enhanced.
- Each scan contains at least one tumor, with a maximum of 16 tumors in one case.
- There are 86 tumors in all, 2 - 38 mm in diameter (3 - 28,731 cubic mm volume). - - These include both primary and metastatic tumors of various sizes, some with dark necrotic cores. 
- The ground truth, which involves the center and radius of each tumor, was given to us by Dr. Nguyen, the radiologist who provided the data.
- The data has been pre-processed. It has been converted to ANALYZE format.
- Using fully automated midsagittal plane (MSP) extraction software [2], each scan was properly aligned.
- The skull and surrounding tissue were removed from the scans using FSL [3][4].

## The Method

### Blob acquisition and Pruning


![pic 2](https://user-images.githubusercontent.com/66870226/102862591-542cd580-4457-11eb-87b0-14b24a58fd22.png)
### Fig. 1: Slice 136 of brain h38

- We begin with an input volume that is centered and whose midsagittal plane is aligned vertically (Fig. 1).
- Blobs are extracted from this brain using 3D Laplacian of Gaussian filters with varying sigma values, thus allowing us to find blobs of different sizes. 
- The skull and surrounding tissue are not removed at this stage to allow maximum information to be used for blob detection. 
- After filtering, a dilated brain mask is applied, allowing the border of the brain to be padded by true voxel intensities instead of a black background.
-This should make analysis more accurate.
- At every voxel, the maximum achieved LoG value, along with the sigma associated with the filter from which this value was computed, are stored in one maximum LoG volume. 
- Non-maximum suppression is then applied to every 5x5x5 non-zero box in this volume, keeping as a tumor candidate the voxel in each box with the highest LoG score. 
- We record this voxel's coordinates along with the LoG score and the sigma value of the filter used to obtain this score.
- Fig. 2 shows the results of this non-maximum suppression.

![pic 3](https://user-images.githubusercontent.com/66870226/102862922-d5846800-4457-11eb-8212-7d81c569e270.png)
### Fig. 2: The original volume and all the initial blobs after non-maximum suppression.

Following the initial blob detection, the blobs are pruned by a combination of three masks. 
- The first, shown in Fig 3., is the brain-only mask obtained using FSL [3][4]. 
- This mask serves to remove all blobs outside the brain-only region.

![pic 4](https://user-images.githubusercontent.com/66870226/102863130-285e1f80-4458-11eb-9942-26f181e60a63.png)
### Fig 3. The original volume, the brain-only mask, and the brain mask applied to the original volume, respectively

- The second, shown in Fig. 4, takes into account deviations from the mean intensity, whether light or dark. 
- Tumors are visually distinct from their surroundings, and this mask is meant to represent visual distinctiveness. 
- It is obtained by finding the mean and standard deviation of the brain-only intensities (see rightmost image in Fig. 3) and subtracting the mean value from each brain-only voxel's intensity. The absolute value of these differences is then taken to account for voxels that are both lighter and darker than the mean. 
- To create the mask, any voxels whose absolute difference from the mean is within one standard deviation of the mean are set to 0.
- All others are set to 1.

![pic 5](https://user-images.githubusercontent.com/66870226/102863341-76732300-4458-11eb-8fd9-a5e20105b01e.png)
### Fig 4. The original volume, the absolute intensity deviation mask, and the mask applied to the original volume, respectively

- The third mask, shown in Fig. 5, takes into account asymmetry. 
- The brain-only portion of the original volume is reflected along the MSP. 
- The absolute value of the difference between the brain-only portion of the original volume and the resulting image then forms the absolute difference image [5][6]. 
- To create the mask, any voxels where the absolute difference value is below 20% of the maximum value (20% of 255 = 51) are set to 0. 
- All others are set to 1. To allow for blobs that are located along the MSP, which are self symmetric and thus would be pruned away by this mask, values along the MSP and 1 voxel to the right or left of it are set to 1.

![pic 6](https://user-images.githubusercontent.com/66870226/102863499-ae7a6600-4458-11eb-8d73-1dbcc7681092.png)
### Fig 5. The original volume, the asymmetry mask, and the mask applied to the original volume, respectively.

- The three masks are combined (see Fig. 6) to form one mask that is used to perform the first pruning step.

![pic 7](https://user-images.githubusercontent.com/66870226/102863630-dbc71400-4458-11eb-8cc9-077275a99fec.png)
### Fig 6. The original volume, the combined mask, and the mask applied to the original volume, respectively.

- The combined mask is applied to the initial blobs and all blobs whose centers are located outside the mask are removed. 
- The remaining blobs are significantly fewer in number (see Fig. 7 and Table 1).

![pic 8](https://user-images.githubusercontent.com/66870226/102863855-32345280-4459-11eb-91e8-87d29c47fc6a.png)
### Fig 7. The original volume and the blobs remaining after the combined mask is applied.

- The second pruning step removes blobs which are nearly symmetrical. 
- in the brain do not always line up perfectly across the MSP. Gliding along any direction may occur. 
- To account for this, tolerated asymmetry pruning is performed as follows. 
- Let S be the set of blobs that are remaining after mask pruning. 
- Let b be a blob in S.
- We first locate the voxel, vc, that is the symmetrical counterpart of the center of b. 
- We define a search area, A, that is a 21 x 21 x 21 box centered at vc (allowing a movement of 10 voxels in each direction x, y, and z). 
- Let Sb be the set of blobs in S whose centers are in A. 
- If Sb is not empty, blob b is potentially symmetric. 
- Further prune Sb by removing any blobs whose sigma value differs from b's sigma value by more than 2, whose LoG score differs from b's LoG score by more than 0.05, and whose mean intensity differs from b's mean intensity by more than 10.  
- If b is symmetric, the blobs that are symmetric to it should be of similar shape, blobbiness and intensity. 
- If any blobs remain in Sb after this pruning, b is assumed symmetric and is removed, along with these blobs, from S.
- One exception is made if b is self-symmetric. 
- In this case, b and any remaining blobs in Sb are kept since they are also likely self-symmetric.
- The blobs remaining after this second pruning step are shown in Fig. 8. 
- The number of blobs pruned can be seen in Table 1.

![pic 9](https://user-images.githubusercontent.com/66870226/102864065-893a2780-4459-11eb-9539-8a5a19ff3c44.png)
### Fig 8. The original volume and the blobs remaining after nearly symmetrical blobs are removed.

![pic 10](https://user-images.githubusercontent.com/66870226/102864171-ae2e9a80-4459-11eb-9ff1-ab0681b4f49b.png)
### Table 1. The number of blobs in each case after each pruning step. Case #18 corresponds to brain h38.

## 3.2	Feature Extraction

- Usable features are extracted using seven volumes.

### The volumes 

![9](https://user-images.githubusercontent.com/66870226/103407431-af13aa80-4b84-11eb-9a96-9e692a65f836.png)
### Fig. 9: Original Volume

- The first volume is the original volume, masked by a dilated brain mask in order to keep the true voxel values at the edges of the brain (see Fig. 9).
- The second volume is the absolute deviation volume (see Fig. 10). This is obtained the same way as the absolute intensity deviation mask but without thresholding. 
- Thus, each value in this volume represents the absolute difference between the voxel's original intensity and the mean intensity of the brain-only voxels from the original volume. 
- The third volume is obtained by smoothing the original volume with a 3D Gaussian filter. For a blob with sigma value , features for this blob are computed using the volume that was smoothed by a 3D Gaussian filter with the same.
- This allows for features to be computed at the blob's own scale, reducing "noise" from other scales. See Fig. 12 for all smoothed volumes. 

![10](https://user-images.githubusercontent.com/66870226/103407473-d2d6f080-4b84-11eb-8927-afb9d8d791f0.png)
### Fig. 10: Deviation Volume

- The fourth volume had been computed during the blob acquisition stage - the LoG volume. 
- LoG volumes are obtained by filtering the original.
volume with 3D Laplacian of Gaussian filters with 1, 2, 3,...,16 .
- These same sigma values are used in smoothing and are the only sigma values that blobs may have. 
- Once again, features computed for a blob with sigma value  are obtained using the LoG volume that resulted from filtering with a LoG having the same See Fig. 13 for all LoG volumes.
- The next three images are the absolute difference images of the original volume (Fig. 11), the smoothed volumes (Fig. 14, 15), and the LoG volumes (Fig. 16,17), weighted by an intensity deviation prior, and are used as measures of asymmetry. 
- The intensity deviation prior is simply the absolute deviation volume divided by its own maximum value, thus giving values between 0 and 1 at each voxel. 
- It is used to offset the effects of taking the absolute value of the difference image. 
- When a difference image is obtained, a bright asymmetric blob in the original volume will remain bright (although less so).
- A dark asymmetric blob in the original volume will have negative values. To give bright and dark blobs the same footing in the asymmetry image, an absolute value is taken. 
- This, however, leads to symmetric asymmetry the region opposite a light or dark asymmetric blob will have values that are the same as those of the blob, indicating the same degree of asymmetry. 
- Since we are only interested in the blob, this extra region needs to be removed. 
- This is done by applying the prior, which gives more weight to the bright/dark blob and less to the normal brain tissue across the MSP from it.

![11](https://user-images.githubusercontent.com/66870226/103407515-f9952700-4b84-11eb-83ac-2d1da824e130.png)
### Fig. 11 The absolute difference image of the original volume (left) and the same absolute difference image after the intensity deviation prior is applied (right). Only the weighted image was used for feature extraction.

![12](https://user-images.githubusercontent.com/66870226/103407562-1e899a00-4b85-11eb-84a8-1c71f24e1433.png)
### Fig. 12 The smoothed volumes obtained using Gaussian filters with increasing sigma values.

![13](https://user-images.githubusercontent.com/66870226/103407606-3f51ef80-4b85-11eb-98d6-547d1677743e.png)
### Fig. 13 The LoG volumes obtained using LoG filters with increasing sigma values. The LoG values in these images are rescaled to [0,255] to make feature extraction the same for each volume.

![14](https://user-images.githubusercontent.com/66870226/103407644-63153580-4b85-11eb-8c3b-b9f9b895ba70.png)
### Fig. 14 The absolute difference images of the smoothed volumes obtained using Gaussian filters with increasing sigma values. Note that these are nor weighted by the prior and hence are not used for feature extraction.

![15](https://user-images.githubusercontent.com/66870226/103407682-89d36c00-4b85-11eb-91c2-88bce7386b04.png)
### Fig. 15 The absolute difference images of the smoothed volumes obtained using Gaussian filters with increasing sigma values. Note that these are weighted by the prior and are the ones used for feature extraction.

![16](https://user-images.githubusercontent.com/66870226/103407718-ad96b200-4b85-11eb-94e0-c7b730f93061.png)
### Fig. 16 The absolute difference images of the LoG volumes obtained using LoG filters with increasing sigma values.Note that these are nor weighted by the prior and hence are not used for feature extraction.

![17](https://user-images.githubusercontent.com/66870226/103407798-f64e6b00-4b85-11eb-8ae8-3ca1a2baa449.png)
### Fig. 17 The absolute difference images of the LoG volumes obtained using LoG filters with increasing sigma values. Note that these are weighted by the prior and are the ones used for feature extraction.

## The features

- Each blob begins with 5 features from the pruning stage 
- the x, y, and z coordinates of its center, the sigma value of the LoG filter producing the blob, and the LoG value of this blob.
- Of these, only the LoG value is used for classification since the classifier should be invariant to the size and location of tumors.
- The next 91 features are statistical features - 13 features from each of the 7 volumes.
- The mean, median, variance, standard deviation, minimum, maximum, skewness, kurtosis, 5th and 6th central moments, uniformity, entropy and smoothness are computed over the voxels within the volume of a given blob.
- The next 7 features (1 feature from each volume) measure shape.
- The "shape" of each blob is estimated using compactness, computed as follows.
- Let Vb be the set of voxels within the volume of blob b. 

I  I ,
x	x
 
I  I ,
y	y
 
I    I
z	z
 
- Compute at each voxel in Vb . 
- Create the structure tensor for the blob, where the summation is over all voxels in Vb :

 ∑ I 2	∑ I I	∑ I I  
	x	x    y	x  z 
T    ∑ I I	∑ I 2	∑ I  I
	x    y	y	y    z 
 
∑ I  I
 
∑ I I
 
∑ I 2 
 
	x   z	y   z	z     

- Then the compactness, Cb , for the blob is given by:

C   max 1, 2, 3 
min 1, 2, 3 

- Where 1 , 2 , 3 are the eigenvalues of T. 
- Compactness values closer to 1 indicate the blob is more spherical, while values high values indicate the blob is elongated.
- The next 7 features (1 feature from each volume) are Gaussian-weighted intensity measures. 
- Unlike the mean, which gives equal weight to all voxel intensities within the blob, the Gaussian- weighted intensity gives more weight to values closer to the center of the blob. 
- The Gaussian weights are obtained using a Gaussian having a sigma equal to that of the blob.
- The final 7 features (1 feature from each volume) measure strict asymmetry.
- If Vb is the set of voxels within the volume of blob b, and Vb is the set of voxels within an equal-sized volume centered at the symmetrical center of blob b, then the asymmetry of blob b is given by the Mallow's Distance (equivalent to the Earthmover's Distance) :

Ab 
 
∑
all bins
 
cdf  Ib   cdf  Ib 
 
- where
- Ib is the set of intensity values of the voxels in Vb ,
 
- Ib is the set of intensity values of the
 
- voxels in Vb , and cdf gives the cumulative distribution histogram. 
- This histogram has 256 bins, one for each possible intensity value.

### Classification and Evaluation

- The goal of this project was to test the efficacy of the new asymmetry features obtained. 
- As much has changed over the course of this project, comparing current results to previous results would be unfair. 
- The blobs are now obtained using 8 more scales, allowing for more granularity. 
- They are also pruned by a combination of masks and filters that take into account more than just asymmetry. 
- These differences make the dataset entering the classification step different from before.
- Because of this, I decided to test the feature effects on the new dataset only.

### Preprocessing
- Each blob was compared to the ground truth and received a label of tumor or non-tumor.
- To receive a label of tumor, the blob's center must be located within the volume of the ground truth tumor. 
- The blobs from all brains are then pooled into one dataset. 
- At the beginning of each experiment, the features of the training and testing datasets are normalized to [0,1] to avoid bias.

### Data
- Experiments were run on three different datasets.
- The first dataset only contained non- asymmetry features, which excludes the asymmetry feature and any features computed on difference images. 
- The second dataset contained only asymmetry features - the asymmetry feature and those computed on the difference images. 
- The final dataset used all 113 features.

### Feature Selection
- Because the feature set is small - there are only 113 features used - ranking is unnecessary; after normalization, the features are sent directly to Sequential Forward Selection (SFS). 
- Two types of experiments were performed - using SFS wrapped around a QDA classifier, and using SFS wrapped around an LDA classifier. Both produced similar results.

### Cross-Validation and Classification
- Because there are more non-tumors than tumors, the split between training and testing data is determined by the tumors. 60% of the tumor data, and an equal number of non-tumor data is used for training. 
- The remaining data is used for testing. 
_ Each time an experiment is run, the data is randomly split using this proportion. 
= For cross-validation, each experiment is carried out 100 times and the classification results are averaged. 
- LDA, QDA, Naive Bayes (Linear and Quadratic), SVM and a classifier using Mahalanobis distance were used.

### Results
Results of these experiments and feature analysis can be seen in Appendix I.


### 4.	Limitations, Issues, and Future Work
