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
