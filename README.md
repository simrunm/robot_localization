# Robot Localization Project

*ENGR3590: A Computational Introduction to Robotics, Olin College of Engineering, FA2022*

*Simrun Mutha and Melody Chiu*

*14 October 2022*

# Project overview

The goal of our project is to implement a particle filter in order to localize the robot. This means that in any given map, we can identify where the robot is relative to the map. Another way to think about this is that we are using a particle filter in order to find the transform between the robot’s odometry frame relative to the map frame so that we know where it is on the map. For our project, we used the gauntlet map and tried to localize the robot on that map.

<p align="center">
<img src="particle_filter.gif" width="400"/>
</p>
<center> Video of our particle filter working in RViz </center>

* [Implementation](#implementation)
* [Design Decisions](#design-decisions)
* [Challenges](#challenges)
* [Next Steps](#next-steps)
* [Lessons Learned](#lessons-learned)

# Implementation

The particle filter can be broken down into 4 operations that are called in a loop with each update to the filter. An update occurs whenever the robot has moved past a certain threshold. We began by initializing the particle cloud which is a cluster of particles around the Neato, with each particle object holding position and orientation information.

## Updating particle positions
The first step of the particle filter update loop is to reflect the Neato’s change in position for each particle in the particle cloud. This involves finding the Neato odometry's position and orientation change (delta) and mirroring that change for each particle within a frame centered at its initial position. To perform this operation, we used matrix multiplication to transform delta from the Neato odometry frame to the local particle frame before applying it to the particle.

## Updating particle weights
Next, each particle's weight is updated based on how likely the Neato lidar readings are from its position and orientation. In other words, the likelihood that a particle is an accurate estimate can be determined by whether lidar scan data from the Neato matches the surroundings of the particle, represented as an occupancy grid. First, we convert r and theta values, which represent obstacles from the scan data, to the corresponding x and y coordinates where the obstacles would be located relative to the particle. Then we query the occupancy grid at those coordinates to see if there is an obstacle there. If there is, then it indicates that the particle is a likely estimate of the Neato position and should be weighted with a higher confidence. If querying the occupancy grid shows that the nearest obstacle is at some distance, then the particle weight is determined by how far off it is.

## Updating robot position estimate
Next, we take the weighted average of all of the particle's poses to generate an estimated robot pose which is then used to update the odom to map transform. 

## Resampling particles
Lastly, the entire particle cloud is resampled based on the new probability distribution. In other words, we use the weight distribution in the particle cloud to draw a random sample of new particles located in areas with high confidence.

# Design Decisions

One of the design decisions we made was to use matrices to compute the update for each particle. Although we found this method more complicated to understand, we thought it was worth it because the computation would be faster. Our implementation was already fairly inefficient overall and so we thought it was worth it to understand how to make the matrix math work. 

Another design decision we made was to add noise to every particle. Initially, we found that the particles were converging too quickly and the localization would be significantly inaccurate. By adding noise, it would take more time for the particles to converge which made the localization better. 

# Challenges

One challenge we faced was finding a way against the particles converging too quickly. Even though the localization was still unaligned, the particles would all converge to where the NEATO was positioned and the localization would not improve. To overcome this challenge, we decided to add noise to the particles in order to prevent them from converging too quickly. We played around with the different amounts of noise and the places we would add them to improve the localization.

# Next Steps

Currently, a lot of particles return NaN for the closest distance which affects the mean closest distance we get for each particle. NaN is returned when the distance isn’t valid and I would expect it to happen for a few particles but currently it happens for a large number of particles. If I had more time, I would figure out why that is happening and try to understand how to account for it such that it doesn’t compromise the distribution of weights. Currently, the standard deviation of the weights is very small and I think that is because of the large number of NaN values that are being returned.

Currently we are adding noise to every particle, but I think we could improve the particle filter by only adding noise to particles that are already relatively confident, or those that have a higher weight. I think adding noise to particles with low weights doesn’t help improve the accuracy of the filter.

# Lessons Learned

One thing I learned more about were the tradeoffs involved in working on a robotics project. The debugging strategies for this particle filter were more involved than typical strategies. The simplest strategy, using print statements, is not very effective because there are too many messages to process. Using rviz visualizations or matplotlib graphs is much more helpful but it can take a long time to implement. While working on this project, there were some places where it would have been helpful to have nicer visualizations but in the interest of time, I just tried to make do with prints. In other places, it was worth it to have a visual. The starter code in this project provided us with a way to visualize our particle filter without much effort from our end but on future projects that I work on from scratch, I think it will be helpful to put in the time to get a good rviz visual even though it can be frustrating to work on that part of the code.

Another lesson I learned was to allocate more time towards testing the code rather than implementing it. I think the nature of robotics work is that it can always be better than what it is. In this project, we did not leave a lot of time for testing and so we were not able to try out ways to incrementally improve the filter. I think getting to a first pass implementation faster would be helpful in future projects. 
