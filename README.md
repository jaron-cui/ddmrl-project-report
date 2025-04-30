# Intro/Motivation
## Context {Furkan}
TODO: <Flowcharts/graphs>
Robot Utility Models are... <brief and concise description of RUMs>

## Project Objectives
Robot Utility Models are able to complete individual short-term tasks by cloning expert demonstrations. We seek to investigate and expand the capabilities they provide in regards to multimodality and task complexity.

RUMs uses BeT, which has been shown to be able to learn multimodal actions. We aim to test the extent of this capability with a distinctly multimodal task: sorting items based on their visual appearance into location-specific receptacles. RUMs has not yet been tested on such an explicitly forked task conditioned on visual indicators.

It is plausible that RUMs could handle longer, complex tasks with accordingly longer and more complex expert demonstrations, but each task would require a new dataset, and collecting these demonstrations would take exponentially longer due to the greater degree of total variety required for a robust policy. Longer tasks are often able to be decomposed into shorter, simpler subtasks. The same subtask may also be present in many different complex tasks. Therefore, our second objective is to implement task composition.

Finally, the current deployment procedure requires the user to first carefully position the robot and align the camera with the task scene. We explore methods of reducing the dependence on human intervention, such as automated navigation.

## Expected Contributions
The expected contributions of the project include:

1. A robust visually-conditioned sorting policy.
2. An implementation of task composition.
3. The integration of navigation capabilities with RUMs

# Experiments/Processes
## Describe initial ideation stage (cups/cabinet stuff) {Alex}
## Lemon pickup policy {Alex}
## Lemon/lime sorting (left/right) {Alex/Jaron}
## Lemon/Lime Sorting with Labeled Bowls {Jaron}
The first sorting policy is visually conditioned to direct an object to a set relative location (left/right). An interesting alternative is where the sorting destination is not fixed. Rather than sorting lemons and limes into the left and right bowls, respectively, lemons and limes are more flexibly sorted into corresponding labeled bowls. Picture 'lemon' and 'lime' signs affixed to each bowl.

Two problems with the implementation of this task are that 1: It appears to require double the training data as the previous version, as we now need to demonstrate sorting each fruit into the other bowl, and 2: There are still significant inflexibilities - what if we need to make modifications to the labels we select for lemons and limes, or would also like to sort oranges?

### Tailored Image Augmentations
We suggest that the increased training data acquisition may be avoided by using targeted image augmentations.
The training demonstrations can be performed such that we are able to substitute the labels and the sorted item after-the-fact using image processing operations.

From these "generic" demonstrations, we can produce demonstrations of sorting lemons by changing the sign on the destination bowl to appear as the lemon label and color-shifting the held fruit to resemble a lemon. We can likewise produce demonstrations of sorting limes by changing the sign to the lime label and color-shifting the held fruit to resemble a lime.

To facilitate label substitution, the physical labels used while taking training demonstrations are ARUCO fiducials. The held item is always a lemon, and during demonstration is placed into either the left or right bowl. The same ARUCO marker is always placed on the bowl in which the lemon is placed. With this procedure, lemon-sorting demonstrations can be produced by drawing the lemon label atop the ARUCO marker, and lime-sorting demonstrations can be produced by inserting the lime label and color-shifting the lemon.

## Image Encoding-based alignment {Jaron}
### Motivation
Given working RUM policies for various simple tasks, the objective of composing compound tasks requires a robust method of chaining them together. The behavior of each individual task policy is sensitive to the deployment environment. If the model receives out-of-distribution observations, we can expect it to perform poorly. Thus, it is important that the robot realign itself after completing a chained task such that the following chained task is presented with an environment compatible with its training.

For example, suppose the robot is instructed to pick up a lemon and place it in a bowl. First, the robot should align its camera so that a lemon is in view. Second, deploy the lemon-pickup policy. Third, align the camera with a bowl. Fourth, deploy the place-lemon-in-bowl policy.

We attempt to devise an implementation of the alignment function that does not require additional data collection and can be run locally on the robot.

### Demonstration
<p align="center">
  <img src="images\alignment\alignment-demo.gif" alt="Encoding-based Alignment Demo" height="400vh"/>
</p>

### Implementation
Our image encoding-based alignment strategy is as follows:
1. Distill a representation of the start state of a task from the first few frames of each training demonstrations.
2. During deployment, scan the surroundings for the camera frames that most closely match the distilled start state representation.
3. Maneuver the robot towards the orientation that produced the closest match.

We use the pretrained dino-vits16 image encoder to encode the reference and scan images. The start state representation is simply the average of the encodings of the first few frames of each training demonstration. This process is repeated for both the RGB and depth images to produce two independent references which have weighted relative importance.

During a scan, images and the angles at which they were taken are recorded in a buffer class. Then, the images are encoded and scored by their cosine similarity to the references. The angle selection process involves binning the closest matches at small angle increments and selecting the bin with the highest match rate (normalized to account for variable reading density).

An example of a 360° scan that passes over a lemon and the frame taken at the scan angle closest to the selected angle of 315°.
<p align="center">
  <img src="images\alignment\lab-scan-lemon.gif" alt="Example of a scanning video" height="250vh"/>
  <img src="images\alignment\nearest-scan-frame.png" alt="Nearest scan frame" height="250vh"/>
</p>

The selected angle was chosen after scoring each frame and then performing the binning procedure.
<p align="center">
  <img src="images\alignment\raw-scores.png" alt="Raw frame scores" height="250vh"/>
  <img src="images\alignment\binned-scores.png" alt="Binned scores" height="250vh"/>
</p>

The encoder choice and methodology were chosen after experimenting with multiple options. A standalone repository containing the experimental code is located at https://github.com/jaron-cui/camera-frame-alignment.

## DynaMem-based alignment {Akshat}
## Navigation {Akshat}
## Overview of policy training procedure (data collection -> training -> deployment) {Alex/Furkan}

<p align="center">
  <img src="images/training-and-deployment.png" alt="Training and Deployment" />
</p>

<p align="center">
  Figure 1: Training and Deployment
</p>

In the 1st step we collected videos for for picking up the lemons, sorting the lemons, and sorting the limes in different environments. 

An example of the video for picking up the lemon and sorting the lemon be seen in the pictures below. 

<p align="center">
  <img src="images/lemon-pickup-frames/frame-1.png" width="150" />
  <img src="images/lemon-pickup-frames/frame-2.png" width="150" />
  <img src="images/lemon-pickup-frames/frame-3.png" width="150" />
  <img src="images/lemon-pickup-frames/frame-4.png" width="150" />
  <img src="images/lemon-pickup-frames/frame-5.png" width="150" />
</p>

<p align="center">
   Figure 2: Lemon Pick Up
</p>

<p align="center">
  <img src="images/lemon-sorting-frames/frame-1.png" width="150" />
  <img src="images/lemon-sorting-frames/frame-2.png" width="150" />
  <img src="images/lemon-sorting-frames/frame-3.png" width="150" />
  <img src="images/lemon-sorting-frames/frame-4.png" width="150" />
  <img src="images/lemon-sorting-frames/frame-5.png" width="150" />
</p>

<p align="center">
  Figure 3: Lemon Sorting
</p>


## Data collection {Jaron}

## Training {Furkan}


<p align="center">
  <img src="images/vq-bet.png" alt="VQ-BeT Training" />
</p>

<p align="center">
  Figure 4: VQ-BeT Training
</p>


After the videos are collected, they are compressed, and translation (linear movement of the robot's gripper), rotation (how the gripper is rotated in space relative to a reference frame), gripper values (representing how open or closed the gripper is while holding a lemon or lime) that were collected during the video recording are extracted. 

### Stage 1. Action Tokenization

The translation, rotation, and gripper values extracted from each frame in each video are used as actions. When we want to perform a certain task, there are often multiple ways to do it. Transformer-based models are very good at capturing this multi-modality and associating different and relevant parts of the input with each other. However, transformer-based models usually work with discrete data. To train a language model, for instance, the text is split into discrete units named tokens, these tokens are converted into embedding vectors to represent each token with a vector (list of numbers), and these embedding vectors are transformed further with another linear layer. In our case, the action values are all continuous. So, to make these continuous actions compatible with a transformer model, we need to tokenize them. 

There are different ways to do this. One way is to simply use K-Means clustering to cluster the continous actions and treat each cluster as a discrete unit/token. However,this method is inefficient for high dimensional action spaces. It doesn't scale well for long action sequences. It lacks gradient information and it struggles with modeling long range dependencies in action sequences In addition, K-Means create hard boundaries between clusters. []

One other method we can use to tokenize the continous actions is Vector Quantization. 

### Stage 2. Learning VQ-BeT




# Results/Conclusions
## Recap successes/failures of individual experiments/ideas + analysis
TODO: Add respective notes into this section
## Implications of these successes/failures
## Future work?

