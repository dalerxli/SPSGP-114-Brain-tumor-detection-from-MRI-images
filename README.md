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


