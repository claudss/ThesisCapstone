# ThesisCapstone
Assorted files from my capstone project for my MS Thesis at UCLA.

## Table of Contents

## Summary
My final year MS thesis examined automation of the study of nonverbal communication through computer vision and machine learning, allowing for better speed and precision. The developed automated classification pipeline was run on video footage of the first and third presidential debates between Donald Trump and Hillary Clinton to gauge its accuracy. Results showed the automated pipeline is viable as an easily upscaled replacement for human work, able to both reproduce the results of human labeling of the footage and allow for insight into the various nonverbal idiosyncrasies of the speakers with a high average accuracy (approx. 84% across both candidates).

## Background and Data Setup
The usual methods of classifying facial expressions are manual and tedious, not to mention difficult to scale up to larger projects (at least not without investing in more manpower) due to these problems. Essentially, human labelers and their intuition can better understand nuance and ambiguity, resulting in more “accurate” labeling of presented emotions, but quality control of those human labelers is often a large time sink in the research process. This makes automation of the endeavor via machine something of interest to explore.

Data was collected from debate videos via the open-source tools of ![OpenFace](https://cmusatyalab.github.io/openface/) and ![OpenPose](https://github.com/CMU-Perceptual-Computing-Lab/openpose), which analyzed each frame of video to detect facial features and body positioning. Frames were broken down into coordinate planes, with keypoints denoted by their positions within that space. For faces, the analysis looked like this:

![OpenFace](https://github.com/claudss/ThesisCapstone/blob/main/openface_example.png)

Meanwhile, analysis of a frame for body language and positioning looked like this:
![OpenPose2]((https://github.com/claudss/ThesisCapstone/blob/main/openpose_example2.png)

In the below image, we can see how many body points that OpenPose breaks the human body into. The analysis of hands in detail like in the image is optional, but was enabled in this situation because it would be helpful data to take into account various sentiment-conveying gestures. Data points were also only considered from the waist up to eliminate any extraneous data, since the podiums present in debate footage blocked out the lower body.

![OpenPose1](https://github.com/claudss/ThesisCapstone/blob/main/openpose_example.png)

Original annotations for classification (which sentiments were to be checked for, which gestures could be meaningful, etc.) ere determined by experienced academics in the field of communication, and were aggregated based on scores given by annotators who were vetted for impartiality to American politics. 

## Classifier information
The classifier used for this research was a recurrent neural network (RNN), specifically with long short-term memory (LSTM) units which then returned predicted index values for each of the nonverbal cues initially coded for. A standard RNN consists of internal state vectors, referred to as hidden states, that are updated upon processing of an input sequence. When the RNN incorporates LSTMs, another state variable called a cell state occurs in each LSTM. The cell state stores information required over the entire duration of processing of the input sequence, with an LSTM maintaining control over this cell state and updating it when necessary.

In this work, the input at any given timestep is a feature vector of statistics from OpenFace and OpenPose, corresponding to a frame of the analyzed video. The model begins with the first frame of the video and updates cell states and hidden states as it goes through timesteps, storing and passing in those values to the next LSTM for further computation. The values in these states are stored and passed to the computation for the next time step until every element in the input sequence has been processed. 

Here is a diagram to visually demonstrate how the output of a previous frame is fed into the model once more as the input to the next time step of classification:

![Diagram](https://github.com/claudss/ThesisCapstone/blob/main/model_diagram.png)

## Conclusion
The results of the model were analyzed via 10-fold cross validation. This yielded clear overall statistics about the classifier’s accuracy, which averaged .825 across all statistics for Trump (with a range of .646 to .952) and .858 for Clinton (range of .774 to .991), leading to a candidate-wide average of approx. 0.84, and had the highest accuracy for both candidates when examining when they looked at their opponents as well as their overall level of activity. The classifier accuracy numbers also offered insights about both candidates’ speaking styles. Trump’s nonverbal cues and behavior stand out as significantly more demonstrative than Clinton. When factoring in what was observed during manual annotation, Trump did not show any cues that would convey a more placating impression, such as lowered mouth corners, head turned down towards body, or averted eye orientation, hence the lack of data for the sad-face and affinity-gest annotations above. In contrast, the classifier showed especially high accuracy when analyzing his defiant gestures or expressions, such as defiance-gest and disagreement. This demonstrates a high frequency of them that corresponds to his bombastic speaking style, which has been described as “transgressive” and “populist political performance” by other scholars in similar study. 

However, the model's scale was also a bit small. It needed some more extensive analysis to evaluate its performance against potential alternatives, and gauge if it could generalize into application to other debates. A basis of comparison was acquired by applying a separate model trained on a larger dataset, the Expression in-the-Wild (ExpW) Dataset, a public set of 91,793 faces manually labeled with each of seven fundamental expression categories: angry, disgust, fear, happy, sad, surprise, or neutral. This was done with a convolutional neural network (CNN) that takes image rather than video input; as such, the frames of the input video for the RNN were used as the CNN’s classification input.

Using the CNN classifier also gave a chance to single out flaws or inconsistencies in the automated process. For example, despite Trump’s wider range of emoting, he did not actually smile often, with the expression usually occurring in attempts to “laugh off” speaking points by Clinton. However, in his already low-accuracy classification in the CNN’s “happy” category, there were also evident *false positives*: incorrect classifications of frames as “happy” when they were actually depicting other emotions. Because facial datasets like ExpW often have examples of “happy”-categorized expressions that show cues such like visible teeth and a widely opened mouth, those cues can be wrongly interpreted in different situations by classifiers that overlook context.

