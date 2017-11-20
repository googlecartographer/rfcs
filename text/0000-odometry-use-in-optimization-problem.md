# Reconsider the role of odometry in the 2D optimization problem

## Summary
[summary]: #summary

An odometry term has been added to the optimization problem in [this pull request](https://github.com/googlecartographer/cartographer/pull/456).
The term is used if odometry data is available and acts as a relative pose constraint that penalizes deviations from the relative odometry measurements during the optimization.
If no odometry data is available, the relative pose constraint is formed in the conventional way from consecutive initial pose estimates.

Since this approach seems to be problematic in some cases where odometry data is available (e.g. [here](https://github.com/googlecartographer/cartographer/issues/534), our own experiments), we should reconsider the *implicit* use of odometry-based constraints in the optimization.

## Motivation
[motivation]: #motivation

* Odometry-based constraints imply a certain level of trust in the quality of the odometry, which could be problematic due to the usually noisy nature of (wheel) odometry.
* Initial pose estimation in the front-end combines different sensor modalities (pose extrapolation, scan matching) - in contrast, adding a pure odometry constraint in the back-end does not account for multiple sensors and may introduce a bias.

## Approach
[approach]: #approach

We didn't come up with a master plan for this yet, but two possible approaches are:

1. Revert the changes introduced by the PR mentioned above if the discussion yields that the odometry constraint is *generally* not beneficial.

2. A simple compromise would be to introduce a boolean option that allows to use the initial pose estimates for the relative constraints instead of the odometry even if odometry data is available (as it was before the change).

We implemented 2. for testing and saw a qualitative improvement with our system.

## Discussion Points
[discussion]: #discussion

1. A discussion about the pro's and con's of the current implementation would be good since there might be different views due to different experiences with this topic.

2. Because of 1., the possible approaches are of course also up to discussion.