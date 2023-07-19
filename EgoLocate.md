# EgoLocate
- [Site](https://xinyu-yi.github.io/EgoLocate/)
- [Paper](https://arxiv.org/pdf/2305.01599.pdf#page13)

## Abstract
- Motivation: Humans &amp; environments sensing are important topics, but are often done separately, which leads to high errors.
  Without surrounding environments, (inertial) human sensing - mocap - will drift. Without environment sensing, basing on visual alone is not
  enough for techniques like SLAM in situations of poor visual
- EgoLocate: combine inertial mocap &amp; SLAM for REAL-TIME mocap

## Compared to previous works?
- Some does not have global positioning signals => cannot base calculations on ground truth (?)
- Some with completely visual SLAM, or SLAM with only 1 IMU, still fail terribly when faced with poor visual
- None had real time mocap (&amp; simultaneous mapping)

## Method
Pipeline in (simple) words:
1. Preparation: 1 head-mounted camera &amp; 6 IMUs (& some ground truth points, seems like they use around 2-4 points)
2. Calibration: T-pose for mocap; camera is calibrated and initial map points are triangulated
3. Input: inertial measurements (into 4)  & image (into 5)
4. Inertial Mocap:
   - Determine camera pose (thanks to the camera calibration) & global acceleration
   - Camera pose is passed as input to 5
5. Camera Tracking:
   - Match features between keyframes (aka important frames)
   - Optimization: error = E<sub>projection</sub> + E<sub>mocap</sub>, where E<sub>projection</sub> = error of 2D-3D matches, E<sub>mocap</sub>
   = distance/difference (?) between current camera pose (in the optimization process) &amp; the mocap-based camera pose
6. Mapping  &amp; Looping:
   - Input: keyframes (from the camera tracking - step 6)
   - Calculate map point confidence (look at Fig. 3 for best understanding)
   - Mocap-aware Bundle Adjustment (TBH I don't understand what's going on here, but basically to optimize camera pose with mocap constraints, using
     recent keyframes &amp; map points?)
   - There's also loop detection &amp; closing (based on recognizing seen keyframes), but I don't understand how this works, because what happens if the error is huge and they
     want to close the loop? Would the person just... fly? (Edit: yes, the person does just fly. Read [limitations](#limitations)
7. Body Translation Updater:
   - Input: human pose &amp; global acceleration (from inertial mocap - step 4) and optimized camera pose &amp; confidence (output of step 5)
   - Update human translation - using a lot of math I don't understand
8. Output:
   - Input: human pose &amp; global position (step 7); environment map (step 6)
   - We now have a human in an environment

## Results
Thanks to the combination of IMUs and camera, the mocap is incredibly accurate (almost no drifting). In the video showing a person dancing,
the closing loop step does a great job at keeping the person in the correct position, when the camera sees an old frame.

Across metrics, EgoLocate almost always outperforms other methods.

## Limitations
- Online Loop Closure: since the online module does not allow editing of previous keyframes where loop is detected, the person unfortunately has
  to fly (or teleport)
- Body Shape &amp; Scene Scale: the proportions might not be correct because they assume mean body shape. This is a minor problem though.
- Calibration Errors: There are some requirements for calibrations at the beginning.
- Degenerated Cases: edge case where texture is spare or person moving too fast, or the person performing weird poses

## What I rate this paper
9.5/10!

