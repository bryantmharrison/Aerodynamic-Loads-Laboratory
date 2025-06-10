# Aerodynamic Loads Laboratory
Lab 2 of Introduction to Experimental Methods for Spring 2025

## Objectives
This lab was a way of introducing me to the workings of a subsonic wind tunnel. The maximum speed of the wind tunnel is only about 35 mph. It draws in air with giant fan blades in the front, normalizes the air against turbulence via filters and speeds it up by constricting the volume of the tunnel.

![Wind tunnel](https://github.com/user-attachments/assets/be167121-3061-4ba0-b93b-d1ddc98b0e25)

Within the wind tunnel is a load cell oriented so that its y-axis was along the direction of the wind, the x-axis pointing towards the observation room, and the z-axis pointing vertically upwards. This load cell measures six items: the forces in the x, y, and z directions and the moments in the x, y, and z directions. It is important to note that this load cell only works when an item (typically a wing model) is placed upon its mount.

The load cell is placed upon a stepper motor below it. This device rotates the load cell and its attached items by degrees at a time. This is to simulate an aircraft's angle of attack - the pitch of the nose of the aircraft during flight.

The wing tested during this experiment was a symmetric, rectangular wing that had a removeable end plate designed to reduce turbulence.

![Wing](https://github.com/user-attachments/assets/036a694b-aa1b-4a78-860f-cbd0b40ff281)

For calculation purposes, we note that the load cell was placed at the quarter-chord along the airfoil (the cross section of the wing). The quarter-chord is the aerodynamic center of the airfoil and is 1/4 of the distance from the leading edge of the airfoil to its trailing edge.

## Experimental Setup
For this experiment:
* The tunnel will be operated at a "high speed" of ~16 m/s and a "low speed" of ~12 m/s
* A pitot-static tube on/in the wind tunnel connects to a Baratron device that converts pressure differences into electric signals via capacitors with a conversion rate of 1.016 mmHg/V
* A FlowKinetics Manometer measures ambient pressure, temperature, and density
* The load cell measured the data (approximately 100 samples per second) and was sent to a LabView VI to compile the data into a large spreadsheet
* Data was taken at each whole-degree angle of attack up to 22 deg for the high speed and 20 deg for the low speed
* This last point was done for both the wing with its end plate and without

## Results and Discussion
The experiment produced two raw text (.csv) files with all of the data. One file was for the high speed and low speed measurements with the end plate attached, and the other contained the high speed and low speed measurements wihtout the end plate attached.

The first step of analyzing this data is transforming the load cell frame of reference to the aerodynamic frame of reference of the wing. The following diagram clues us into the method of transforming from one frame to the other:

![Airfoil diagram](https://github.com/user-attachments/assets/061c5bd9-03aa-4391-bcba-68f8721de7eb)

Using basic linear algebra the conversion formulas are as follows:
1. L = -Fx*cos(a) - Fy*sin(a)
2. D = Fy*cos(a) - Fx*sin(a)
3. M = -Mz,

where L is the lift force, Fx is the force along the x-axis, a is the angle of attack, Fy is the force along the y-axis, D is the drag force, M is the pitching moment, and Mz is the torque about the z-axis.
