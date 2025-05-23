# Intro/Motivation
## Context *(Furkan)*
Robot Utility Models is a framework for collecting versatile data with portable data collection tools that allow collecting data from different environments in an efficient and quick way, using these data to train an imitation-learning policy that can generalize to new environments, and deploying these policies on Hello Robot Stretch robot. [1]

## Project Objectives *(Jaron)*
Robot Utility Models are able to perform individual short-term tasks learned from expert demonstrations. We seek to investigate and expand the capabilities they provide in regards to multimodality and task complexity.

RUMs uses BeT, which has been shown to be able to learn multimodal actions. We aim to test the extent of this capability with a distinctly multimodal task: sorting items based on their visual appearance into location-specific receptacles. RUMs has not yet been tested on such an explicitly forked task conditioned on visual indicators.

It is plausible that RUMs could handle longer, complex tasks with accordingly longer and more complex expert demonstrations, but each task would require a new dataset, and collecting these demonstrations would take exponentially longer due to the greater degree of total variety required for a robust policy. Longer tasks are often able to be decomposed into shorter, simpler subtasks. The same subtask may also be present in many different complex tasks. Therefore, our second objective is to implement task composition.

Finally, the current deployment procedure requires the user to first carefully position the robot and align the camera with the task scene. We explore methods of reducing the dependence on human intervention, such as automated navigation.

## Expected Contributions
The expected contributions of the project include:

1. A robust visually-conditioned sorting policy.
2. An implementation of task composition.
3. The integration of navigation capabilities with RUMs.

# Experiments / Procedures
## Initial Ideation Stage *(Alex)*
The project started with an idea curation phase, where the current RUM is able to pick up bag and tissue, open door and drawer, and reorient fallen objects on the table. Given our proposal to extend RUM's capability and to explore more complex home tasks, our idea is to first select a task that is adjacent to established possible tasks, and then compose that task with some natural extension. 

In terms of task selection, our first idea is to have a bottle/can/cup pick up policy, and the composition task will be placing cups in cabinet. Here we will train: a) a cup pickup policy, b) a cabinet open policy, c) a cup placing policy. The main obstacle here is navigation and object occlusion: while pick up by itself should be straightforward, interacting with the cabinet requires acting at different height and placing cups require overcoming occlusion caused by the form factor of cups. Other natural tasks, such as taking cans out of the fridge, have similar navigational challenges. We therefore realized that a more suitable task would involve common objects, preferably those small in size so as to reduce occlusion, and a compositional task that requires minimal navigation. Picking fruits naturally came up, and we decided that we will have a pick up policy for lemon and lime, and the compositional task will be sorting them. More detailed description and variations are presented below. 

Besides task selection, we also need to overcome the technical challenge of task composition, as this was not previously implemented. Our goal is to have a general, policy independent way of composing arbitrary policies, as this will allow arbitrary extension of task complexity, as long as the tasks are compositional. There were discussions on techniques to achieve that, below we present two alignment methods, one compares image encoding similarity while the other uses DynaMem.  

<p align="center">
  <img src="images/cup_pick_up_frames/ezgif-frame-001.png" width="150" />
  <img src="images/cup_pick_up_frames/ezgif-frame-039.png" width="150" />
  <img src="images/cup_pick_up_frames/ezgif-frame-059.png" width="150" />
  <img src="images/cup_pick_up_frames/ezgif-frame-079.png" width="150" />
</p>

## Lemon Pickup Policy *(Alex)*
This is the first policy we trained, where we familiarized ourselves with the data collection procedure, and this policy can be seen as replicating existing result, since it is just a simple pick up task, like those already trained. From conversation with Mahi and Haritheja, we understood we needed roughly 500 demos to learn a lemon pick up policy, so we collected lemon pick up in various environments, some of the demos can be seen in the "overview of policy training procedure" section. Below we attach a successful policy rollout demonstration (2x speed).

<p align="center">
  <img src="images/lemon_pick_up.gif" alt="lemon_pick_up" height="181px"/>
</p>

## Lemon Lime Sorting (Left/Right) *(Alex)*
This task is the first variant of sorting lemon and lime, where the idea is heavily inspired from Mahi. It has the advantage of conceptually simple and easy to collect data while compared to the labeled bowls sorting it is less data efficient, see next section for more detail.

The task is as follows: the stretch robot starts with a lemon/lime in its gripper, with two containers in front of it, and the task is to put the lemon/lime in the left/right container respectively. Alternatively, this task can be seen as a placing policy that is condition on fruit variety where the placing locaiton is fixed, in the next variant the we relaxes the second constraint. This task can also be seen as a combination of two separate task where each sorts lemon and lime respectively, and in that case one could imagine using a higher level policy to decide which sub-policy to trigger. The environmental diversity/out of distribution generalization comes in the form of a) different containers, b) different table top, c) different approach directions. While lemon and lime also comes in variations, the variations are much more minor than that of the other factors. In our data collection, to insure robustness, we made sure to consider variation and combination of the containers, such as swapping the left and right container, and we made sure to use different table top to record our demo. We also had different appraoch directions, though at the present moment whether we have under-collected the number of demos given our dataset diversity, more discussion in the results/conclusion section. 

Our first training involved around 1.5k demos, however that seems to not be enough, a further 1k demos is collected and the training is currently underway; we also had some hiccups during trainig because of data preprocessing and gpu availability, more discussion in the results/conclusion section. Below we show some test environment rollout and some sample of collected demos.

<p align="center">
  <img src="images/collected_demo.gif" alt="pick up and sorting" height="240px"/>
  <img src="images/chaining_sorting.gif" alt="pick up and sorting" height="240px"/>
</p>

For the second round of training we added ~500 demos, we collected more but weren't able to use all of it because of data consistency worries. The larger dataset now contain mostly plates, and the demos are now more conscious of object occlusion. The policy is much better, below are tests performed on two different table, with plates not seen before (though the demo does contain white plates). The success rate for sorting alone is around 50% or higher, and the main failure mode is a failure to release lemon/lime at the plate. We also notice the curious behavior of a "slow start" where the stretch robot moves minutely in the first few steps, and then seemingly finding it's target, moves quickly and confidently. This slow start does not seem to correlate with policy success rate, although the test video below shows a failure case.

<p align="center">
  <img src="images/lemon-sofa-success.gif" height="240px"/>
  <img src="images/lime-sofa-release-failure.gif" height="240px"/>
  <img src="images/lime-desk-success.gif" height="240px"/>
  <img src="images/lemon-desk-slowstart-failure.gif" height="240px"/>
</p>


We see that the first rollout is a success, the second rollout demonstrates the failed to release failure mode, the third rollout is a success with some slow start behavior, and the fourth one shows a slow start behavior and also a failed to release. All vidoe are 2x speed up.

## Lemon/Lime Sorting with Labeled Bowls *(Jaron)*
### Idea
The first sorting policy is visually conditioned to direct an object to a set relative location (left/right). We consider a version of this task where the sorting destination is not fixed. Picture 'lemon' and 'lime' signs affixed to each bowl.

Two problems with the implementation of this task are that 1: It appears to require double the training data as the previous version (we now need to demonstrate sorting each fruit into both bowls), and 2: There are still significant inflexibilities - what if we need to make modifications to the labels we select for lemons and limes, or would also like to sort oranges?

### Tailored Image Transformations
We suggest that the need for increased data acquisition may be avoided by applying targeted image transformations to a smaller dataset.
Training demonstrations can be performed such that we are able to modify the appearance of the labels and the sorted item in post.

From these "generic" demonstrations, we can produce lemon-sorting demonstrations by changing the sign on the destination bowl to appear as the lemon label and color-shifting the held fruit to resemble a lemon. We can likewise produce lime-sorting demonstrations by changing the sign to the lime label and color-shifting the held fruit to resemble a lime.

Training demonstrations are performed with ARUCO fiducials that facilitate label substitution. A lemon is placed into either the left or right bowl. The same ARUCO marker is always associated with the destination bowl. With this procedure, lemon-sorting demonstrations can be produced by drawing the lemon label atop the ARUCO marker, and lime-sorting demonstrations can be produced by inserting the lime label and color-shifting the lemon.

The raw sample (left), the derived lemon sorting sample (center), and the derived lime sorting sample (right).
<p align="center">
  <img src="images/aruco/pre-transform-sample.gif" alt="Raw sample" height="200vh"/>
  <img src="images/aruco/lemon-transformed-sample.gif" alt="Derived lemon sample" height="200vh"/>
  <img src="images/aruco/lime-transformed-sample.gif" alt="Derived lime sample" height="200vh"/>
</p>

### Implementation
OpenCV allows us to intermittently extract the position of ARUCO markers from videos frame-by-frame.
We tested three methods of turning these sporadic signals into persistent ones:
1. Kalman filters
2. Forward optical flow tracking
3. Interpolated forward and reverse optical flow tracking

Below, we provide annotated signal tracking visualizations. First, the raw frame-by-frame detections. Second, with Kalman filtering. Third, using forward optical flow. Lastly and most robust, using interpolated dual optical flow.
<p align="center">
  <img src="images/aruco/aruco-raw.gif" alt="Raw sample" height="150vh"/>
  <img src="images/aruco/aruco-kalman.gif" alt="Kalman filter" height="150vh"/>
  <img src="images/aruco/aruco-forward-optical-flow.gif" alt="Forward optical flow" height="150vh"/>
  <img src="images/aruco/aruco-dual-optical-flow.gif" alt="Dual optical flow" height="150vh"/>
</p>

We map colored labels with blue borders and random variation (in hue, border width, and noise level) onto the signal stabilized by dual optical flow interpolation.

Note that our 728 "generic" samples all use lemons. The plan was to mask by lemons by location and hue, and then color-shift in order to transmute lemons to limes. Alas, yellow is too varied a color category to easily filter. The example given above is mottled with only mild leakage of green to the bowl in which it is placed, but many other samples taken on light wooden tables have proven infeasible to reliably constrain green within the fruit. We ought to have implemented and tested the image transforms on a few samples before collecting the full dataset. In that case, we would have used a more easily masked color - perhaps by painting a lemon neon pink.

The full implementation of ARUCO tracking and custom image transformations resides at [https://github.com/jaron-cui/aruco-label-tracking](https://github.com/jaron-cui/aruco-label-tracking).

### Outcome
Due to time constraints and problems with the vanilla left/right lemon/lime sorting policy, we were unable to train and test a policy on the transformed data. However, we were able to learn techniques for data processing and the importance of small-scale pre-testing.

## Image Encoding-based alignment *(Jaron)*
### Motivation
Given RUM policies for various simple tasks, composing compound tasks requires a robust chaining method. The behavior of each individual task policy is sensitive to the deployment environment. If the model receives out-of-distribution observations, we can expect it to perform poorly. Thus, it is important that the robot realigns itself after completing a subtask such that the following is presented with an appropriate initial observation.

For example, suppose the robot is instructed to pick up a lemon and place it in a bowl. First, the robot should align its camera so that a lemon is in view. Second, deploy the lemon-pickup policy. Third, align the camera with a bowl. Fourth, deploy the place-lemon-in-bowl policy.

We propose an alignment function that does not require additional data collection and that can be run entirely locally.

### Demonstration
<p align="center">
  <img src="images/alignment/alignment-demo.gif" alt="Encoding-based alignment demo" height="400vh"/>
</p>

### Implementation
Our image encoding-based alignment strategy is as follows:
1. Distill a representation of the start state of a task from the first few frames of each training demonstrations.
2. During deployment, scan the surroundings for the camera frames that most closely match the distilled start state representation.
3. Maneuver the robot toward the orientation that produced the closest match.

We use the pretrained dino-vits16 image encoder to encode the reference and scan images. The start state representation is simply the average of the encodings of the first few frames of each training demonstration. This process is repeated for both the RGB and depth images to produce two independent references which have weighted relative importance.

During a scan, images and the angles at which they were taken are recorded in a buffer class. Then, the images are encoded and scored by their cosine similarity to the references. The angle selection process involves binning the closest matches at small angle increments and selecting the bin with the highest match rate (normalized to account for variable reading density).

An example of a 360° scan that passes over a lemon and the frame taken at the scan angle closest to the selected angle of 315°.
<p align="center">
  <img src="images/alignment/lab-scan-lemon.gif" alt="Example of a scanning video" height="250vh"/>
  <img src="images/alignment/nearest-scan-frame.png" alt="Nearest scan frame" height="250vh"/>
</p>

The selected angle was chosen after scoring each frame and then performing the binning procedure.
<p align="center">
  <img src="images/alignment/raw-scores.png" alt="Raw frame scores" height="250vh"/>
  <img src="images/alignment/binned-scores.png" alt="Binned scores" height="250vh"/>
</p>

The encoder choice and methodology were chosen after experimenting with multiple options. A standalone repository containing the experimental code is located at [https://github.com/jaron-cui/camera-frame-alignment](https://github.com/jaron-cui/camera-frame-alignment).

### Outcome
The resultant alignment function works to a limited degree in a controlled environment with the lemon pickup data. It was not tested with other policies. The behavior of the function is very stable in a given environment, but not always correct.

One problem with the procedure lies in the generality of the pretrained image encoder. The encoder captures features of the reference images that appear frequently, including those which are not the intended subject. In the case of lemon pickup, there is almost always a proximal, flat surface, such as a table, dominantly present in the scene. Through testing, it is clear that the alignment function will almost always prefer to point towards a large, empty table rather than a lemon in a cluttered pile if given the choice.

The most important improvement needed to make this alignment function practical is a way to prioritize matching to the features most pertinent to the task over incidental correlations.

## Step Towards - Compositional RUM *(Akshat)*
### Motivation

Robot Utility Models (RUM) are powerful tools for executing semantic tasks like "pick up the lemon" or "sort the fruits," but their success is highly dependent on the initial scene configuration. In most standard deployments, RUM policies are trained and tested in fixed environments where the relevant objects are already visible and well-aligned within the robot’s field of view.

However, for RUM policies to generalize to real-world, compositional tasks, the robot must go beyond passive execution—it must first navigate and align itself to the appropriate position where a policy can be triggered. This means:

- Locating the right region in the scene (e.g., a sorting station with two bowls),

- Ensuring the relevant objects (e.g., lemons, limes) are in view,

- Then executing the correct policy based on this setup.

Chaining multiple RUM policies—for instance, find lemon -> move it to bowl -> check for next item—requires persistent object memory, spatial awareness, and active alignment. Without these capabilities, RUM policies remain brittle and constrained to curated setups.

This motivated us to develop a lightweight memory and navigation system, which enables the robot to search, align, and act based on natural language queries. The system we present here is a first step toward making RUM policies modular, composable, and environment-aware.

### Object-Based Alignment *(Akshat)*


To enable scene level understanding for object-centric alignment, we integrated the Intel RealSense D435i depth camera with the Stretch 3 robot. We developed a dynamic memory module that processes the RGB-D stream from the head-mounted camera using SAMv2 for segmentation and CLIP for generating semantic embeddings. These embeddings, along with the estimated 3D positions of each segmented object, are stored in a voxel-based memory map

<img src="./images/dynamem/memory-store.png" alt="Memory store setup" height="250vh"/>

 The memory module continuously updates itself with new objects as the robot moves around, allowing it to later observed objects using natural language queries. The above pipeline shows, when prompted with “Monitor” the robot searches the memory for matching embeddings and retrieves the 3D location of the closest match.

 Intial test setups on publically available RGBD datasets - [ https://github.com/Tulsani/voxel-maps ]

### Navigation for Small Horizon Tasks *(Akshat)*

One of the key success factors in executing RUM policies is having the correct starting orientations of the robot

To take action on visual memory, we implemented a navigation routine that aligns robot movement with queried object locations. The function accounts for the fact that the robot’s base movement is perpendicular to its camera’s field of view. It rotates the robot by 90°, moves toward the stored object location, and then rotates back—ensuring accurate object reachability.

<img src="./images/dynamem/robot-navigation.png" alt="re-alignment" height="250vh"/>

 Further, we develop a `find` function enables iterative search for objects beyond the current scene. If the queried object is not found immediately, the robot rotates in 72° increments (just under the 87° FoV of the RealSense D435i) and scans the new field of view. This loop continues until the object is located or a full 360° rotation is complete.

 This iterative alignment and search allow the robot to visually locate targets and orient itself for downstream manipulation tasks.

<img src="./images/dynamem/navigation-control.png" alt="navigations control" height="250vh"/>

### Outcome 

<img src="./images/dynamem/Object-nav-faster-gif.gif" alt="small horizon navigation" height="250vh"/>

We successfully demonstrated small-horizon object-centric navigation using vision-language memory. This sets the stage for integrating compositional RUM policies that act on semantic references in the environment—e.g., "pick up the lemon and place it in the left bowl."

A key area for improvement, the voxel memory stores observations locally from the headcam's frame which limits persistence and spatial consistency. Objects observed in earlier rotations are forgotten. Integrating SLAM to build a persistent world model would allow the robot to remember and revisit locations. We have tried adding ORB-SLAM3 but we were yet to test out its capabilities

We also look to improve our alignment function with active perception, with scene priors. As when attempting practical compositional RUM tasks, where similar task would be aligned at similar locations / setup.

Integeration for seperate thread for processing the RGBD headcam image stream - updates to robot server code to support headcam - setting up the voxel memory can be found here for future reference ( https://github.com/NYU-robot-learning/min-stretch/tree/jafa-ftr-dynamem )

## Overview of Policy Training Procedure *(Furkan)*

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

After collecting the demos, which roughly takes $\mathcal{O}(1)$ hours, it is the data preprocessing and training stage.

During data preprocessing, the video is compressed, and a action vector is extracted, which includes three that parameterizes linear motion and four that parameterizes rotation (quaternions) and a gripper value. It turns out that extrating the gripper width can be difficult at times and not entirely robust to novel situations, we encountered some difficulties there.

The training stage is two fold, the first step is using a clustering algorithm to discretize the action (vqvae), and the second stage is training the behavior transformer (vqbet) using this discretization. Notebly, the first stage have a gpu speed up but have very low gpu usage, so it is not Greene friendly. The second stage roughly takes 4 rtx8000 in <5 days.

During deployment, we move the model weights to the stretch robot and run the deployment code. There is a nice UI that we have used and modified for our project.

## Data Collection *(Jaron)*
The majority of collected data is thoroughly documented in the Google Doc [Task Data Descriptions](https://docs.google.com/document/d/1YCe_gprSHMkfH2Gd2knWuvQBa5XBqS2Zd7mPv_3KrRY/edit?usp=sharing) (visible to NYU accounts).

### Lemon Pickup
The lemon pickup task starts with an open gripper and a lemon present in the scene, usually about 2-4 ft away. The gripper approaches, grasps, and lifts the lemon.

400 samples were collected using the long-fingered gripper, while 668 demos were collected using the short-fingered gripper.
The lemon pickup policy was ultimately trained on the short-fingered samples.

### Lemon/Lime Sorting
The lemon/lime sorting task starts with a gripper grasping a lemon or lime in front of two side-by-side bowls. The gripper places lemons in the left bowl and limes in the right bowl.

1104 + 800 samples were collected and used to train the policy. 1000 additional unused samples were collected at a later date.

### ARUCO Lemon/Lime Sorting
The ARUCO lemon/lime sorting task starts with a gripper grasping a lemon in front of two or more bowls, each with an ARUCO marker placed behind it. The gripper places the lemon into the bowl accompanied by the light green-backed marker.

728 samples were collected, but no policy was trained due to delays in image processing implementation and focus on the left/right lemon/lime sorting task.

## Training *(Furkan)*


<p align="center">
  <img src="images/vq-bet.png" alt="VQ-BeT Training" />
</p>

<p align="center">
  Figure 4: VQ-BeT Training
</p>


During the data collection process, RGB videos, translation (linear movement of the robot's gripper), and rotation (how the gripper is rotated in space relative to a reference frame) are are collected with an iPhone when the lemons are picked up and lemons and limes are sorted with the portable data collection tool. 

After this, the gripper values (representing how open or closed the gripper is while holding a lemon or lime) are predicted from the video frames with an RGB-based model. 

In addition, translation, rotation, and gripper values are automatically synchronized and timestamped by the iPhone without needing any calibration [1]. 

After the data is collected, the videos are compressed, and translation, rotation, and gripper values are extracted and these are used as actions. 

### Stage 1. Action Tokenization

In daily life, when we want to perform a certain task, there are often multiple ways to do it. Transformer-based models are very good at capturing this multi-modality and associating different and relevant parts of the input with each other. However, they usually work with discrete data. To train a language model, for instance, the text is split into discrete units named tokens, these tokens are converted into embedding vectors to represent each token with a vector (list of numbers), and these embedding vectors are transformed further with another layer. 

In our case, the sequence of translation (x, y, z), rotation ($q_x$, $q_y$, $q_z$, $q_w$), and gripper (g) values that are extracted from each video are used as actions and these values are all continuous. So, to make these continuous actions compatible with a transformer model, they need to be tokenized. 

There are different ways to do this. One way is to use K-Means clustering [2] to represent similar continuous actions in different groups/clusters and treat each cluster as a discrete unit/token. However, this method is inefficient for high-dimensional action spaces. It doesn't scale well for long action sequences. It lacks gradient information and it struggles with modeling long-range dependencies in action sequences In addition, K-Means creates hard boundaries between clusters. [4]

One other method that can be used to tokenize the continuous actions is Vector Quantization with Variational Autoencoders [3]. In this method, a neural network model is used to extract the features of the continuous action values. This is called a Residual VQ Encoder. In addition, a list of vectors is initialized randomly and these are called codebook vectors. There is also a process of quantization which is basically mapping the outputs of the RVQ Encoder to the nearest codebook vector.  

In VQ-BeT, multiple layers of quantization are used in such a way that the codebook in each layer captures more details about the input (continuous action) that were missed in the previous layers. Assuming that we use $N$ layers, here is how the process works: 

1) After a continuous action is encoded into a latent vector $x$ by RVQ Encoder, this latent vector is compared with all vectors of the 1st codebook.
2) The codebook vector that is most similar to the latent vector $x$ is found and this codebook vector becomes the quantized output of the action in the 1st layer. The index of this codebook vector is called the "primary code" for the input action.
3) The difference (residuals) between the latent vector $x$ and the quantized output of the 1st layer is passed to the 2nd layer.
4) The codebook vector in the 2nd codebook that is closest to the residuals is found and the index of this codebook vector is called "secondary code" for the input action. 
4) This process continues until quantizing the residuals $N-1$ times.

Because the difference (residuals) between the continuous action $x$ and the quantized outputs from the previous layer(s) are used in each layer, this is called Residual Vector Quantization (RVQ). 

After this process is done, another neural network model tries to reconstruct the original continuous action from the quantized representations of each layer. This model is called the Residual VQ Decoder [4]. 

During the training of RVQ, the encoder outputs of different actions can be assigned to the same codebook vector. For updating that codebook vector, the average of all encoder outputs that are assigned to that codebook vector is taken and the codebook vector is updated as a weighted average. By using this approach, codebook vectors adapt to represent the distribution of actions over time. 

At the end of RVQ training, the codebooks are updated and a codebook vector can be seen as the discrete representation of the continuous action.

### Stage 2. Learning VQ-BeT

MinGPT [5] is a minimal representation of the GPT that is used to predict the next token. In VQ-BeT, the sequence of image frames (observation sequence) extracted from the collected videos is used as input to MinGPT. The output of the MinGPT is fed to a layer (code predictor head) and this layer produces a probability distribution over all possible primary codes (index of the codebook vector in the 1st codebook that is closest to the input action), a probability distribution over all possible secondary codes (index of the codebook in the 2nd codebook that is closest to the input action), etc. In other words, a probability distribution of the indices of the code vectors is predicted for each quantization layer.

Instead of choosing the code vector that is assigned the highest probability, a code vector is sampled from the probability distribution over all code vectors for each quantization layer. This introduces controlled randomness that allows exploration and helps the model avoid getting stuck in repetitive behaviors. Then the difference between 1) the code vector that is predicted by the code predictor head based on the observation sequence and 2) the code vector that is assigned to the observation sequence by the RVQ is minimized for each quantization layer. 

In addition, the codebook vectors that are predicted by the code predictor head for each quantization layer based on the sequence of observations are combined and decoded to an action with the RVQ Decoder that was trained in Stage 1.

However, a discrete unit can only represent a limited amount of information. Therefore, when a continuous action is converted into discrete tokens (code vectors), and these discrete tokens are used to reconstruct the continuous action, the loss of information in the continuous action is inevitable during the reconstruction process. To solve this issue, the output of the MinGPT is fed to another layer that is used to produce small continuous adjustments for each possible code vector in each codebook in all quantization layers. These small continuous adjustments are called offsets. 

After the probability distribution of the indices of the codebook vectors are predicted by the code predictor head for each quantization layer and a codebook vector is sampled from these distributions, the offsets that are predicted by the offset head for each quantization layer and that belong to the selected codebook vectors are combined and then added to the decoded action. This decoded and adjusted action is then compared with the ground truth action and the difference between them is minimized during the training process.


# Results/Conclusions
## Contributions
### Lemon/Lime Left/Right Sorting Policy
During this project, we tested the capabilities of RUMs by training on a more complex task than done previously - sorting.
While the deployment of the policy had a low success rate, the majority of the failures are related to fine control.
The sorting policy reliably guided the lemons and limes in the correct directions, but had trouble with deposition.
This may have to do with the difficulty of locating the rims of bowls when very close - especially considering the visual obstruction of the grasped object.

### Simple Image Encoding-Based Alignment Function
The image encoding-based alignment function has the advantages of being generalizable and being simple to run.
No additional data collection is required, since it is trained on the data already collected for a given policy.
No external context or information about the task is required, for the same reason.
However, this also means that it is only applicable to tasks for which data has already been collected.

The performance of the alignment function is fairly consistent within a given environment, but is not always correct. The correctness of the encoding similarity scoring function is an area that needs improvement for practical application of the process.

### Text-Prompted Visual Memory Alignment Function
The text-prompted alignment function uses a more sophisticated encoder than the simple image encoding-based alignment function, as well as a voxel memory module. The use of a wide-range headcam grants greater visibility of the scene than the gripper perspective camera. One advantage of the system is its increased flexibility and tunability, as alignment prompts can be tailored for each policy.

A future improvement would be the integration of persistent memory, such as via the integration of SLAM.

### RUMs Database Expansion
Overall, our team increased the RUMs V2 Data count by 4,737 samples for various tasks.

## Notable Obstacles
### Gripper opening/closing threshold hyperparameter tuning
During deployment, the model predicts a gripper value, and if that value passes a threshold then the gripper acts to close/open. The policy performance crucially depends on this threshold, which is a hyperparameter in deployment, and one observes in practice that sometimes failure occurs because the threshold is $\epsilon$ above the predicted gripper value and that the gripper value does not necessarily moves in a smooth manner. If this is not due to the policy being undertrained, I wonder if there is a more nature, smoother way to gate gripper behavior. 
### Gripper value problem
It turns out Greene is gpu poor for the students, and since all steps in the training stage requires GPU disjointly it adds to the overhead and wait time. While we had minor problems with quaternion parameterization, the biggest inconsistency seems to be gripper value extraction. It seems to me that using a smaller gripper did not change gripper value extraction, but having the gripper started off open, as in the sorting policy, really messed things up. One temporary fix is to invert the video in the gripper value extraction function as that would make it look like a pick up policy. However this seems to not work as can be seen in the image below. We did seem to always have data preprocessing problems.
<p align="center">
  <img src="images/H_reprocessed.png" width="500"/>
  <img src="images/t_reverse.png" width="500" />
</p>
<p align="center">
  Correct (Left) vs Attempted Fix (Right)
</p>

### Occlusion problem
This is not the necessary reason that the policy is performing less than idea, but it is a speculation. Where in the sorting demos, we could see that a) the gripper object is out of focus and b) the target visual cue is sometimes occluded. While problem a does not affect us, one could image situations in which the blurred out texture of gripper object is not enough to differentiate it for the task (perhaps sorting bad lemon from good lemon). Problem b is more serious, where for example in the sorting policy, in the later steps, the information of the location of the gripper is only contained in the rim of the image. A potential solution is to use a longer effective observation horizon in inference, so that the policy can infer the location of the gripper using "memory". Another solution is during demo collection we can collect in such a way so as to tilt the gripper downwards, revealing the occluded object, This will require many more hours of data collection, but mostly it is unclear whether this would improve the policy. Due to time constraint, neither suggested solution is tested.

# Acknowledgement
Special thanks to Mahi Shafiullah, Haritheja Eturuku, and Lerrel Pinto for making this project possible.

# References
[1] Etukuru, H., Naka, N., Hu, Z., Mehu, J., Edsinger, A., Paxton, C., Chintala, S., Pinto, L., & Shafiullah, N. M. M. (2024). *General Policies for Zero-Shot Deployment in New Environments*. arXiv preprint arXiv:2409.05865. [https://arxiv.org/abs/2409.05865](https://arxiv.org/abs/2409.05865)

[2] Shafiullah, N. M. M., Cui, Z. J., Altanzaya, A., & Pinto, L. (2022). *Behavior Transformers: Cloning k modes with one stone*. In *Thirty-Sixth Conference on Neural Information Processing Systems (NeurIPS 2022)*. [https://openreview.net/forum?id=agTr-vRQsa](https://openreview.net/forum?id=agTr-vRQsa)

[3] Van den Oord, A., Vinyals, O., & Kavukcuoglu, K. (2017). *Neural Discrete Representation Learning*. In *Advances in Neural Information Processing Systems*, 30. [https://proceedings.neurips.cc/paper_files/paper/2017/file/7a98af17e63a0ac09ce2e96d03992fbc-Paper.pdf](https://proceedings.neurips.cc/paper_files/paper/2017/file/7a98af17e63a0ac09ce2e96d03992fbc-Paper.pdf)

[4] Lee, S., Wang, Y., Etukuru, H., Kim, H. J., Shafiullah, N. M. M., & Pinto, L. (2024). *Behavior Generation with Latent Actions*. arXiv preprint arXiv:2403.03181. [https://arxiv.org/abs/2403.03181](https://arxiv.org/abs/2403.03181)

[5] Karpathy, A. (2020). MinGPT. GitHub repository. https://github.com/karpathy/minGPT
