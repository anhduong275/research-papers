# Learning Physically Simulated Tennis Skills from Broadcast Videos
[Website](https://cs.stanford.edu/~haotianz/research/vid2player3d/) | [Paper](https://cs.stanford.edu/~haotianz/research/vid2player3d/data/tennis_skills_main.pdf)

## Abstract
The system can learn to simulate tennis playing through widely available broadcast videos of tennis. The system consists of a low level imitation policy and a high level motion planning policy.
When trained on large datasets, it seamlessly connects shots into actual games, where it simulates 2 characters playing with each other. The strokes and learning styles are diverse, and motions are
accurate enough to hit the ball.

## Related work
- Monocular human pose/motion estimation: kinematic pose estimators had issues with artifacts. Some previous works attempted to fix this (more details in section 2 of paper). However, this paper not only seeks to estimate poses correctly, but also synthesixe simulating controllers.
- Physics based character control: deep reinforcement learning (DRL) proved some success. To accomplish this, they use a VAE - the motion embedding will be constructed using a conditional VAE.
- Learning skills from videos: mocap is nice, but data is hard to find. Previous works showed some success, but trained each controller on a short video clip. They instead choose to train one controller on a big dataset.
- Character animation for sports: mostly used mocap. They don't.
- Inspiration: Vid2Player, a 2D version of this.

## Method
* Note: this can be messy, as the content is rearranged from the original paper.
* Before we start: I don't understand what they mean by 'root'. Center of the character? Or projection point of character center on tennis court?
- Overview is in figure 2 of paper.
- Stage 1: Video annotation
  - 13 US open match videos, with 3 specific players, chosen by appearance frequency
  - Kinematic motion estimation done by automatic machine annotations (using off the shelf models, and some data processing in between)
  - Player's identity (to learn style of player) & racket-ball contact times done manually

- Stage 2: Low level imitation policy
  - Purpose: correct the physically-incorrect estimation from estimated kinematic motion (done in stage 1) (and also used in stage 4 to control movement of simulated character)
  - Define task: control character agent to mimic reference motions, in a physically simulated environment
  - The task is formulated as a Markov decision process: M = (S - states, A - actions, T - transition dynamics, r - reward function, gamma - discount factor. Generally:
    - Initial state of character s<sub>0</sub> = initial state of reference motion.
    - Agent iteratively samples actions from A, according to a policy at each state s<sub>t</sub> $\in$ S. Environment transitions to next state s<sub>t+1</sub> according to transition dynamics T, then output a scalar reward r<sub>t</sub> for the transition.
    - In the end, pick the best policy (highest return, which is some function of the reward scalar r<sub>t</sub> and discount factor gamma)
  - In specific details:
    - State is defined with several features, like rotations/positions/velocities of joints or root. Look at paper for more details
    - Action is defined through some function I don't quite understand, which are based on the joints (non-root and root) and the torques at those joints
    - Rewards: minimize energy, closely track reference motion. There are several rewards function, which measures based on joint rotation, velocity, joint position, keypoint (projected 2D points vs actual detected 2D points), and power penalty (the more energy, the less reward).
  - The training part talks about residual forces, which I do not understand

- Stage 3: Motion Embedding
  - Input: the corrected motions from stage 2
  - Motion embedding model is based on motion VAE model (MVAE), while adapting like so:
    - Condition on pose features in global coords => prevent global drift
    - Besides predict pose, also predict phase (bc tennis motion is cyclic) (why do we predict - don't we just output the simulation? maybe they do mean outputting some kind of pose)
  - Pose representation is also defined in a similar manner to stage 2's state or action. Motion phase is represented using a cyclic variable theta = \[0, 2pi\].

- Stage 4: High-level motion planning policy
