# A Method for Animating Children's Drawings of the Human Figure
[arXiv](https://arxiv.org/abs/2303.12741)

[Paper](https://arxiv.org/pdf/2303.12741.pdf)

## Abstract
- Children's human figure drawings are interesting, but different from other types of drawings and are not usually discussed
- Animated Drawings Demo - the result of their work, animates children's drawings, and is widely received around the world by millions
- In addition, they also contribute the Amateur Drawings Dataset, which is the first dataset to contain children's drawings exclusively.

## Background
- Children's drawings has large variations, even as the child develops. These variations make it hard to automate animation. Children also represents
3D objects uniquely, 'with attributes ...similar to those of 3D objects they are meant to represent'. Thus, there's no such complex concept as foreshortening.
There is also twisted perspective (object is drawn from different perspectives - this sounds a lot like cubism :) ).
- 2D image to animation: some tools require users for additional input - too complex for children. Some require users to draw multiple poses for the character -
this is also too hard for a child. Some only animates specific forms - this is not suited for this task.
- Detection, segmentation &amp; pose estimation - but for non-photorealistic images: there's a lack of datasets. The dataset they created serve
  this specific purpose.

## Method
1. Figure Detection: detect bounding box, using Mask R-CNN. Fine-tuned the model to only detect human figures.
2. Figure Segmentation (separate figure from background): Mask R-CNN's segmentation proves to be inadequate, so they decided to use an
   image processing-based approach: resize (width = 400px) -> grayscale -> adaptive thresholding -> morphological closing + dilating (remove noise
   &amp; connect foreground pixels) -> floodfill (no closed groups have any holes) -> retain polygon with largest area. This step might be easier
   to understand with the help of figure 5 in the paper.
3. Pose Estimation: due to large variations in strokes' meanings, they minimize number of keypoints to be used as joins - only 17 points.
   Train ResNet-50 to estimate joint locations (for each joint a heatmap is produced, the highest value pixel is taken as the joint location).
4. Animation: Generate skeletal rig from the estimated pose, which will be used to follow a mocap. The rig is divided into upper body and lower body.
   Each of them either follow the mocap from the front or the side (sagittal) plane. The decision to follow which plane depends on the similarity between
   the body's forward vector &amp; the plane's normal. This method results in twisted perspective &amp; enhances the visual effect.
5. UI: user takes a picture of their drawing. The program presents a bounding box, a segmentation mask &amp; joint estimations in order, user can modify them all.
   Finally user chooses from a gallery of mocap &amp; watch their drawing comes to life :)

## Evaluation
- Good public reception
- The more data, the higher the success rate (look into paper for more details)
- The majority of users prefer twisted perspective

## How do I rate this paper?
- 8.5/10! Easy to read, and very cute, but I didn't learn as much as when I read EgoLocate
