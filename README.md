
# Object Detection using SSD (Single Shot Detection)
---
- SSD is a single-stage detector, meaning it performs object detection in a single forward pass through the network. It directly predicts the bounding boxes and class scores from the feature maps without needing a separate region proposal step.
- Two-Stage Detector: Faster R-CNN is a two-stage detector. The first stage is the Region Proposal Network (RPN) that generates region proposals (candidate bounding boxes that might contain objects). The second stage classifies these proposals and refines their bounding boxes.
- SSD uses a set of predefined anchor boxes (default boxes) with different aspect ratios and scales at multiple locations in the feature maps. The model predicts the offsets and class probabilities for these boxes directly.
  
### Topics -
1. How SSD is Different
2. Multi-box Concept
3. Predicting Object positions - Predicting Offsets
4. Scale Problem

#### 1. How SSD is Different
![image](https://github.com/user-attachments/assets/993a15cc-f561-470d-9f02-7e540c50abdd)

- Look at image, only once,

---
Introduction - 
- The SSD (Single Shot MultiBox Detector) is a popular architecture for object detection that combines speed and accuracy.
- It is designed to detect objects in images in a single pass, making it faster compared to two-stage detectors like Faster R-CNN.


#### 3. Predicting Object positions - Predicting Offsets
![image](https://github.com/user-attachments/assets/65fe18a0-5364-4710-9b55-90c5be91ec89)

-These anchor boxes serve as references for detecting objects. The idea is that the model does not predict bounding boxes from scratch; instead, it predicts adjustments (offsets) to these anchor boxes to fit the objects in the image better.
- During training, each ground truth object is matched to the anchor box that has the highest Intersection over Union (IoU) with it. The model learns to predict the correct offsets that minimize the difference between the predicted bounding box (after applying the offsets) and the ground truth bounding box.
- At each location on the feature map, the model applies a small convolutional filter (often 3x3 in size) to predict the offsets for the anchor boxes associated with that location.
The output of this convolutional filter is a set of values corresponding to the offsets for each anchor box. If there are 𝑘 anchor boxes per location, the filter predicts 4×k values: Δx, Δy, Δw, and Δh for each of the 
k anchor boxes.
- The model uses backpropagation to adjust the weights of the convolutional layers that predict the offsets. The gradients of the localization loss with respect to the predicted offsets are calculated, and these gradients are used to update the weights.
- Over time, through many iterations of training, the model learns to make more accurate predictions of the offsets, enabling it to better fit the anchor boxes to the objects in the image.

#### 4. Scale Problem
![image](https://github.com/user-attachments/assets/ae7fa36a-8479-46f3-bbf2-470962f43f86)

- horse in front is not detected.
- will resize the image keeping the anchor box size as same and then perform CNN on top of it to classify the object.
- will need multiple resized boxes for this. 
- CNN is used to reduce the image size.
---

![image](https://github.com/user-attachments/assets/29909d1d-a80e-4f05-af0a-b58c2ff04dcd)

Lets break down the architecture -

- 



---
 #### Below is an overview of the SSD architecture:

1. Base Network (Feature Extractor):
The SSD architecture starts with a base network that is usually a pre-trained convolutional neural network (CNN) like VGG16. This network acts as a feature extractor, providing a rich set of features at different scales. The intermediate feature maps from this network are used for detecting objects.

2. Multiscale Feature Maps:
Unlike many traditional models, SSD utilizes feature maps from multiple layers of the base network, which allows the model to detect objects at various scales. These feature maps decrease in size as they go deeper into the network, which enables SSD to detect both large and small objects.

3. Convolutional Layers for Predictions:
SSD applies a set of small convolutional filters (3x3) on these feature maps to predict the class scores and bounding box offsets for a fixed set of default anchor boxes (or priors) of different aspect ratios.
Each location in the feature map is associated with several anchor boxes, and the network predicts:
The probability of each class for each anchor box.
The offset of each anchor box to better match the object it encloses.

4. Anchor Boxes (Default Boxes):
SSD uses a technique known as anchor boxes or default boxes at each location of the feature maps. These boxes have different aspect ratios and sizes to handle objects of various shapes and sizes. The model predicts how these boxes should be adjusted to best fit the objects in the image.

5. Loss Function:
The SSD loss function is a combination of:
Localization Loss: Measures the difference between the predicted bounding box and the ground truth box using Smooth L1 loss.
Confidence Loss: Measures the difference between the predicted class probability distribution and the true distribution using a softmax cross-entropy loss.

6. Non-Maximum Suppression (NMS):
After making predictions, SSD applies Non-Maximum Suppression (NMS) to remove redundant bounding boxes for the same object. NMS retains the bounding box with the highest confidence score and suppresses the others.

7. Output:
The final output of the SSD architecture includes the bounding boxes and class probabilities for the detected objects. The bounding boxes are adjusted according to the predicted offsets and the most confident predictions are selected after NMS.

- Advantages of SSD:
Speed: SSD is significantly faster than two-stage detectors like Faster R-CNN because it processes the image in a single pass.
Multiscale Detection: By using multiple feature maps of different resolutions, SSD can handle objects of various sizes effectively.
Simple Design: SSD has a relatively simple design compared to other detection models, making it easier to implement and train.
Limitations:
Accuracy on Small Objects: SSD may struggle with detecting small objects in images due to the lower resolution of the deeper feature maps.
