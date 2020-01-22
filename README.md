# 6DoF Car Detection

My 184th* place solution for the [6DoF Car Detection competition](https://www.kaggle.com/c/pku-autonomous-driving) hosted on Kaggle by Peking University/Baidu.

## Overview

My solution was pretty simple: use keras centernet[[1]](https://github.com/GreatGameDota/6DoF-Car-Detection-from-RGB-Images#final-thoughts) with heatmap head and regression head. I train the model on preprocessed images with the masks of ignored cars. Focal loss was used for heatmap head and L1 loss (as implemented in the centernet baseline kernal[[2]](https://github.com/GreatGameDota/6DoF-Car-Detection-from-RGB-Images#final-thoughts)) for regression head. Optimizer was Adam with initial lr of 0.001 with ReduceLROnPlateau employed to lower lr based on val_loss.

## Model

I used keras centernet as implemented by @see--[[1]](https://github.com/GreatGameDota/6DoF-Car-Detection-from-RGB-Images#final-thoughts). I have a heatmap head with one output and a regression head with 5 outputs (yaw, roll, z, pitch_sin, pitch_cos). I use the predicted x,y from heatmap in order to get the final x,y,z prediction. I then concatenate the two heads together in order to calculate loss. For decoder, I used the same decoder implemented in @see--'s repo. Trained weights files are avaliable for download in this repo's releases (<link>)

## Input

What to give the model was a big problem to solve which hindered me for basically the entire competition. Initially I used a gaussian heatmap as mask but this caused a great slow down in training (1 epoch took 8 hours alone). And I didn't switch to a simple square mask until a week before the competition deadline. The regression mask didn't have any problems since it is always a square mask. As for RGB image input I experimented a lot with it but settled on the most basic preproccessing. I tried keeping the input consistent with the original implementation of the centernet model[[1]](https://github.com/GreatGameDota/6DoF-Car-Detection-from-RGB-Images#final-thoughts). So I used the same letterbox transformer util to reshape the image to 512x512 with black bars at the top and bottom of the image. I then used the same normalize function to normalize the input image. I also added augmentations: horizontal flip[[4]](https://github.com/GreatGameDota/6DoF-Car-Detection-from-RGB-Images#final-thoughts), and random contrast/brightness.

## Training

This part is where my inexperince hindered me the most. I first trained my best model on a 90-10 train data split, lr=0.001 w/ ReduceLROnPlateau, Adam optimizer, and for about 20-25 epochs (colab disconnects after 9-10 epochs and I didn't keep track). Training took about 20 total hours (about an hour per epoch). This first part of training got a CV of .18 (CV calculated using the 10% validation data and using @its7171's evaluation script[[3]](https://github.com/GreatGameDota/6DoF-Car-Detection-from-RGB-Images#final-thoughts)). **Screenshot of this training below.** I then changed the train data split to 80-20 to try and get a better idea of the performance of the model. I trained separate models and didn't see much difference in public LB performance and CV score. So then I took my later model (which initially scored .16 mAP on the 20% validation data) and continued training with the 80-20 train test split with initial lr set to what it was when training stopped last time. I also implmented a callback which calculated the mAP after every epoch and saved the model with best score. The mAP increased slightly to .167 which scored the best on LB of the models I actually tested.

Other training notes:

- Every experiment had a batch size of 1 due to memory
- For every fresh train I used old pretrained weights from a model I previously trained with just the heatmap head on gaussian heatmaps. This improved convergence for every model.
- Everything was done in Google Colab Notebooks on my laptop
- All trained model weights were saved into my Google Drive

Overall I learned a lot on how to properly train deep models and the main problem was the time constraint on me since I only discoverd this working model structure less than a week before the compeition's end.

What I should have done:

- Use a LR Scheduler to optimize lr.
- Train a new model on entire dataset with the best hyperparameters
- Figure out a faster and better way to experiment inorder to find optimal solutions
- Figure out augmentation (I did use augmentations but I don't think it helped)
- Implement mAP callback while training ASAP

## Final Submission

For my final submission I converted the model to an inference model (add the decoder) and predict on the test images. I then convert the x,y predictions to image coordinates and used the predicted z to get the x,y,z world coordinates. I also used a threshold where there were <23,000 predicted cars as recommended by fellow kagglers (which increased my score a good amount).

```
Public LB: 0.058
Private LB: 0.051
```

## Experiments

I could go on for ages about all the experiments I persued but there are SO many things I tried over the last 4 months that I can't be bothered and none of them worked anyway.

## Final thoughts

Oh boy I ended up writing so much that I doubt anyone will read. If you read all of that thanks, I hope you got something helpful from it. Anyways, I had a lot of fun in my first ever kaggle competition, I learned a lot and I can't wait to apply my new knowledge for upcoming competitions!

---------

- [1] Keras Centernet Github Repo by see-- https://github.com/see--/keras-centernet  
- [2] Awesome Centernet Basline kernal by hocop1 https://www.kaggle.com/hocop1/centernet-baseline  
- [3] Important evalution script by its7171 (based off my evaluation script util functions) https://www.kaggle.com/its7171/metrics-evaluation-script  
- [4] Correct horizontal flip function by gebbissimo https://www.kaggle.com/gebbissimo/correct-horizontal-flipping-during-augmentation  

*This was written before private LB was finalized so placement might change

![](https://github.com/GreatGameDota/6DoF-Car-Detection-from-RGB-Images/blob/master/assets/hourglass%204.png)
