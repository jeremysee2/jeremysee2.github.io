---
layout: page
title: Brainhack - NLP and CV in Robotics
date: 2020-06-25
---
## DSTA Brainhack 2020: NLP/CV in Robotics

I engineered a cooperative robotics platform where a drone flies to a designated point to take a photo of the arena and plot a course around the obstacles for the ground robot using OpenCV. The robot would then use dead reckoning to navigate the course. In the second stage, we trained a Natural Language Processing (NLP) model to identify a doll based on its description and a Computer Vision (CV) model using [YoloV3](https://github.com/ultralytics/yolov3) in PyTorch to identify said doll in the arena. Once identified, the robot would pick up the doll with its grabber and score points for the competition. My team and I placed 4th overall.

![robot we used](/uploads/dji-robomaster.png)

Cooperative robotics was no issue for us, as we were easily able to characterise the shape of the track and start/end points using feature detection in OpenCV. The drone would fly above the track, grab a picture, send it back to the computer for processing, which would issue instructions to the robot via the vendor API over TCP.

The main difficulty that we encountered was the system integration between the NLP and CV parts of the competition. Our NLP was good enough to extract the correct description of the doll from the sentence, but was unable to successfully identify the doll in the arena. We had previously tested these two functions separately, but never as a whole. Unfortunately, this mistake ultimately cost us the trophy.