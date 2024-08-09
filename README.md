
# Object Detection using SSD (Single Shot Detection)
---
- SSD is a single-stage detector, meaning it performs object detection in a single forward pass through the network. It directly predicts the bounding boxes and class scores from the feature maps without needing a separate region proposal step.
- Two-Stage Detector: Faster R-CNN is a two-stage detector. The first stage is the Region Proposal Network (RPN) that generates region proposals (candidate bounding boxes that might contain objects). The second stage classifies these proposals and refines their bounding boxes.
- SSD uses a set of predefined anchor boxes (default boxes) with different aspect ratios and scales at multiple locations in the feature maps. The model predicts the offsets and class probabilities for these boxes directly.
- SSD Beats YOLO and Faster RCNN
  
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
- Multiscale Feature Maps: SSD makes use of feature maps from multiple layers of the base network to detect objects at different scales. Shallower layers detect smaller objects, while deeper layers detect larger objects.

  
---

![image](https://github.com/user-attachments/assets/29909d1d-a80e-4f05-af0a-b58c2ff04dcd)

Lets break down the architecture -

**2. Input size:**
- The input to the SSD model is a 300x300 image with 3 channels (typically RGB).

**2. Base Network (VGG-16):**
- VGG-16 Backbone: The first part of the SSD architecture is a truncated VGG-16 network. The diagram shows that the SSD uses VGG-16 up to the Conv5_3 layer.
- Conv4_3: The output of the Conv4_3 layer is used for object detection. This feature map has a size of 38x38 with 512 channels. This large feature map is useful for detecting smaller objects in the image.
- Conv5_3: The final layer from the VGG-16 backbone used is Conv5_3, which has a size of 19x19 with 512 channels.

**3. Extra Feature Layers:**
- The extra feature layers are additional convolutional layers added to the architecture after the VGG-16 backbone. These layers allow the model to detect objects at multiple scales and handle different aspect ratios.
- These layers progressively reduce the size of the feature maps:
- Conv6 (FC6): 19x19x1024
- Conv7 (FC7): 19x19x1024
- Conv8_2: 10x10x512
- Conv9_2: 5x5x256
- Conv10_2: 3x3x256
- Conv11_2: 1x1x256
- Each of these feature layers has specific kernel sizes, stride, and padding to control the dimensions of the output feature maps.
- Smaller the dim, larger the object is detected.

**4. Multiscale Predictions:**
- Detection Predictions: At each feature layer, small convolutional filters (typically 3x3) are applied to predict object classes and bounding box offsets.
  
- Each layer contributes to predictions at different scales:
- Conv4_3: Predicts using a 3x3 filter with a 4x(Classes+4) output. The Classes+4 term accounts for the number of object classes (e.g., 20 for VOC) and 4 for the bounding box coordinates.
The process is repeated for each of the other feature layers, with different numbers of default boxes (anchor boxes) per location.

**5. Anchor Boxes:**
SSD uses a fixed set of default anchor boxes at each location in the feature maps. These boxes vary in scale and aspect ratio, allowing the model to detect objects of various shapes and sizes. For instance, Conv4_3 may use 4 anchor boxes, while Conv8_2 may use 6, as indicated by 4x(Classes+4) and 6x(Classes+4).

**5.1 Variety of Aspect Ratios and Scales:**
- Anchor Boxes (Default Boxes): Anchor boxes are predefined bounding boxes with various aspect ratios and scales that serve as references for the model to predict object locations and sizes. The idea is that the model does not need to predict bounding boxes from scratch but rather adjusts these anchor boxes to fit the objects in the image.

- Aspect Ratios: The anchor boxes are designed with different aspect ratios (e.g., 1:1, 2:1, 1:2) and scales to cover a wide range of possible object shapes. This is important because objects in real-world images can have widely varying sizes and proportions.

**5.2 Number of Anchor Boxes:**
- 4 Anchor Boxes at Conv4_3:
In the Conv4_3 layer, 4 anchor boxes per spatial location are used. This layer has a relatively large feature map (38x38), which is better suited for detecting smaller objects. By using 4 anchor boxes, the model can cover different aspect ratios and scales while keeping the number manageable to avoid overburdening the computation and memory requirements.

- 6 Anchor Boxes at Deeper Layers (e.g., Conv7):
As you move deeper into the network, the feature maps become smaller (e.g., 19x19 at Conv7). These smaller feature maps are more suitable for detecting larger objects. In these layers, 6 anchor boxes per spatial location are used. This increased number of anchor boxes allows the model to accommodate a wider variety of larger objects and different aspect ratios at these scales.

**5.3 Coverage of Object Sizes:**
Smaller Objects: The layers with more feature map locations (like Conv4_3 with 38x38) are tasked with detecting smaller objects. Fewer anchor boxes (e.g., 4) are sufficient here because the higher resolution of the feature map already provides fine-grained spatial coverage.
Larger Objects: For layers with fewer feature map locations (like Conv7 with 19x19), more anchor boxes (e.g., 6) are used to ensure that the model can still detect larger objects, even though the spatial resolution is lower.


**6. Non-Maximum Suppression (NMS):**
After the model generates a large number of bounding box predictions, Non-Maximum Suppression is applied to filter out redundant boxes. This step retains the boxes with the highest confidence scores for each detected object, ensuring that only one bounding box per object remains.


**7. Final Detections:**
The SSD model outputs a total of 8732 detections per class. These are filtered using the confidence scores, and the remaining high-confidence detections are passed to the Non-Maximum Suppression step.
The final detections per image after NMS represent the objects detected in the image along with their bounding boxes and class labels.

**8. Performance Metrics:**
The diagram indicates the performance of the SSD model: it achieves 74.3 mAP (mean Average Precision) and can process images at 59 FPS (Frames Per Second), highlighting its efficiency and effectiveness for real-time object detection tasks.

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
