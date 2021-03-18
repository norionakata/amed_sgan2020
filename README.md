# Generation of supervised training dataset for AI for diagnostic imaging by GAN: Datasets for supervised learning of breast ultrasound images by StyleGAN2

Norio Nakata, MD.

Division of Artificial Intelligence in Medicine, Jikei University, School of Medicine
### Table of contents
0. Image preparation
1. Training for image classification

     1-1. InceptionResNetV2 training with real images

     1-2. Generation of synthetic images by StyleGAN2

     1-3. Selection of synthetic images by trained model of real images

     1-4. InceptionResNetV2 training with synthetic images

2. InceptionRexNetV2 test and comparison of two models & statistical analysis

![プレゼンテーション1](https://user-images.githubusercontent.com/47726033/111586385-15af7880-8804-11eb-95d3-d82cddf24235.jpg)
Figure1. Overall workflow

## 0. Image preparation
First, the images are divided into "Train" and "Test".
150 benign and malignant images for the "Test", and the others are randomly selected and classified for "Train". The image format uses BMP for classification and PNG for image generation. All image sizes are 256x256.
Ideally, the number of images should be 10,000 or more, but if the number of images is 1000 or more, composition is possible. I used StyleGAN2 this time, but it seems that the successor version, StyleGAN2-ADA, can synthesize high-quality images with thousands of images. However, in our experience, in the case of the mammary gland ultrasound image used this time, StyleGAN2 was more suitable as supervised learning data for two-class classification than StyleGAN2-ADA, so StyleGAN2 was used this time. Information on this matter will be disclosed separately in the future.
## 1. Training for image classification
### 1-1. InceptionResNetV2 training with real images
Build the Anaconda virtual environment as follows.
- cudatoolkit               10.1.243             
- cudnn                     7.6.5                   
- keras                     2.3.1                          
- keras-applications        1.0.8                 
- keras-base                2.3.1                   
- keras-preprocessing       1.1.0                    
- matplotlib                3.1.3                    
- matplotlib-base           3.1.3             
- numpy                     1.18.1             
- numpy-base                1.18.1           
- pandas                    1.0.1            
- pillow                    7.0.0             
- python                    3.7.6          
- scikit-learn              0.22.1        
- scipy                     1.4.1           
- tensorboard               2.2.1          
- tensorboard-plugin-wit    1.6.0        
- tensorflow                2.2.0          
- tensorflow-base           2.2.0           
- tensorflow-estimator      2.2.0             
- tensorflow-gpu            2.2.0    

Download all the Codes for this GituHub.
Copy the selected 150 images to "BreastBenign" and "BreastMalignancy" in the "Test" folder in the "hikakudata" folder in the "real" folder.
Then zip the hikakudata folder.

Run classification training: 
```
python IRV2_755_real_breastUS.py
```
Test after training.`
```
python IRV2_755_real_breastUS_test.py
```
The test result (example) is output as follows.
```
Real 
2x2_IncRenNetV2_775
Benign
[[138 12]
[ 18 132]]
Benign precision: 0.9166666666666666 ( 132 / 144.0 )
Benign recall(sensitivity): 0.88 ( 132 / 150.0 )
Benign specificity: 0.92 ( 138 / 150.0 )
Benign f_1 score: 0.8979591836734694 ( 1.6133333333333333 / 1.7966666666666666 )

Malignancy
[[132 18]
[ 12 138]]
Malig precision: 0.8846153846153846 ( 138 / 156.0 )
Malig recall(sensitivity): 0.92 ( 138 / 150.0 )
Malig specificity: 0.88 ( 132 / 150.0 )
Malig f_1 score: 0.9019607843137256 ( 1.6276923076923078 / 1.8046153846153845 )

Accuracy: 0.9 ( 270 / 300.0 )

precision recall f1-score support

Benign 0.917 0.880 0.898 150
Malignant 0.885 0.920 0.902 150

accuracy 0.900 300
macro avg 0.901 0.900 0.900 300
weighted avg 0.901 0.900 0.900 300

300/300 [==============================] - 1s 4ms/step
AUC: 0.9650666666666666
```    
References:
1. Keras Applications
https://keras.io/api/applications/
2. 【Python】画像認識 - kerasで InceptionResNetV2をfine-tuningしてみる 【DeepLearning】 - PythonMania　（Japanese）
https://www.pythonmania.work/entry/2019/04/17/154153
## 1-2. Generation of synthetic images by StyleGAN2
Regarding the implementation of StyleGAN2, it conformed to the official GitHub.
The operating system used was Ubuntu 18.04.

// NVlabs/stylegan2: StyleGAN2 - Official TensorFlow Implementation
https://github.com/NVlabs/stylegan2

After implementing the Anaconnda environment first, T was based on the official GitHub. We used two NVIDIA GV100 GPUs. For StyleGAN, GPUs of Volta generation and above are recommended.

After implementing the Anaconnda environment, the environment was constructed as follows.

- cudatoolkit               10.1.243             
- cudnn                     7.6.5       
- keras-applications        1.0.8                     
- keras-preprocessing       1.1.0                  
- numpy-base                1.19.1          
- python                    3.6.12                
- tensorboard               1.14.0         
- tensorflow                1.14.0        
- tensorflow-base           1.14.0         
- tensorflow-estimator      1.14.0             
- tensorflow-gpu            1.14.0         
- 
### Run dataset_tool.py
Example:　
```    
python dataset_tool.py create_from_images ~/stylegan2/datasets/benign-dataset ~/BreastBenign
```    
### Traing: run_train.py
Example: 
```    
python run_training.py --num-gpus=2 --total-kimg=100000 --data-dir=datasets --config=config-e --dataset=benign-dataset --mirror-augment=true
```    
### Monitor training:　Run tensorboard
Open another terminal and run tensorboard.
Reference: TensorBoard: TensorFlow's visualization toolkit
https://www.tensorflow.org/tensorboard

Example:
```    
python tensorboard --logdir results/00010-stylegan2-benign-dataset-2gpu-config-e
```    
In our environment, training takes 3-4 days or more. Select the network with the lowest FID after the training and use it for image generation.

### Image generation: run_generator.py
Generates 25,000 images for both benign and malignant.
Example :
```    
python run_generator.py generate-images --network=results/00010-stylegan2-benign-dataset-2gpu-config-e/network-snapshot-012499.pkl --seeds=1-25000 --truncation-psi=1.6
python run_generator.py generate-images --network=results/00011-stylegan2-cancer-dataset-2gpu-config-e/network-snapshot-009918.pkl --seeds=1-25000 --truncation-psi=1.6
```    
## 1-3. Selection of synthetic images by trained model of real images
Copy the created 25,000 images to the / sg2t16_28000 / sgan_out / BreastBenign and BreastMalignancy folders, respectively.
Then zip the sgan_out foldger.
Run the select the images of sg2t16_28000 folder.
```
python IRNV2_755_28000_breast_select_all.py
```
The result is when input to the malignant synthetic image sgan_out
/ sg2t16_28000 / select_malignancy_IRNV2 /
When input to benign composite image sgan_out
/ sg2t16_28000 / select_benign_IRNV2 /

Note１) It is better to select benign and malignant composite images separately.
Note２) This process is performed to improve the accuracy of the image by screening the composite image with a two-class classification model preliminarily trained in real image when using it as supervised learning data.
## 1-4. InceptionResNetV2 training with synthetic images
Randomly select 14,000 benign and malignant images output in 1-3 above, and copy them to Breast Benign and Breast Malignancy of Train Falda in / sg2t16_28000 / hikakudata /, respectively.
Also, the images of Breast Benign and Breast Malignancy in the Test folder are
Data in / real / hikakudata / Test / Copy and use 150 real images for each benign and malignant.
Then zip  / sg2t16_28000 / hikakudatahikakudata folder.

Run classification training: 
```
python IRV2_755_28000_breastUS.py
```
Test after training.`
```
python IRV2_755_28000_breastUS_test.py
```
The test result (example) is output as follows.
```
2x2_IncRenNetV2_775
Benign
[[130  20]
 [ 34 116]]
Benign precision: 0.8529411764705882 ( 116 / 136.0 )
Benign recall(sensitivity): 0.7733333333333333 ( 116 / 150.0 )
Benign specificity: 0.8666666666666667 ( 130 / 150.0 )
Benign f_1 score: 0.8111888111888111 ( 1.3192156862745097 / 1.6262745098039215 )

Malignancy
[[116  34]
 [ 20 130]]
Malig precision: 0.7926829268292683 ( 130 / 164.0 )
Malig recall(sensitivity): 0.8666666666666667 ( 130 / 150.0 )
Malig specificity: 0.7733333333333333 ( 116 / 150.0 )
Malig f_1 score: 0.8280254777070064 ( 1.3739837398373984 / 1.659349593495935 )

Accuracy: 0.82 ( 246 / 300.0 )

              precision    recall  f1-score   support

      Benign      0.853     0.773     0.811       150
   Malignant      0.793     0.867     0.828       150

    accuracy                          0.820       300
   macro avg      0.823     0.820     0.820       300
weighted avg      0.823     0.820     0.820       300

300/300 [=================
```
##  2. InceptionRexNetV2 test and comparison of two models & statistical analysis
From the test results of 1-1. InceptionResNetV2 training with real images and 1-4. InceptionResNetV2 training with synthetic images, it was analyzed by the Mcnemar test whether there was a significant difference between them.
Reference: statsmodels.stats.contingency_tables.mcnemar — statsmodels
https://www.statsmodels.org/dev/generated/statsmodels.stats.contingency_tables.mcnemar.html
Example:mcnemar.py
```
import numpy as np
from statsmodels.stats import contingency_tables

ar = np.array([[223,10],[38,29]])
a = contingency_tables.mcnemar(ar).pvalue
print(a)　
```
Output：p-value
```
6.169640777642373e-05
```
