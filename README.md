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
* Data was taken at each whole-degree angle of attack up to 22° for the high speed and 20° for the low speed
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

This conversion must be done to the averages of every angle of attack for each of the four scenarios. This was done through a MATLAB script (attached to this repository). The following lines will walkthrough the script and explain the process being done behind the scenes:

    plateData = readmatrix('A09-X_Endplate.csv');
    attackAngle = plateData(:,2);

These lines pull the data from the .csv file into a MATLAB matrix (plateData). Then, a vector of the angles of attack (attackAngle) is pulled from this plateData matrix.

    Fx = [];
    Fy = [];
    Fz = [];
    Tx = [];
    Ty = [];
    Tz = [];
    V = [];
    data = [];
    for i = 1:length(attackAngle)

These lines prepare for then initalize a for loop that goes through each angle of attack.

    a = attackAngle(i);
    if a == 0 && i <= 78800
        Fx = [Fx plateData(i,3)];
        Fy = [Fy plateData(i,4)];
        Tz = [Tz plateData(i,8)];
        V = [V plateData(i,9)];
        if attackAngle(i+1) >= 1
            avgFx = mean(Fx);
            stdFx = std(Fx);
            avgFy = mean(Fy);
            stdFy = std(Fy);
            avgTz = mean(Tz);
            stdTz = mean(Tz);
            avgV = mean(V);
            stdV = std(V);
            instData = [a avgFx stdFx avgFy stdFy avgTz stdTz avgV stdV];
            data = [data;instData];
            Fx = [];
            Fy = [];
            Fz = [];
            Tx = [];
            Ty = [];
            Tz = [];
            V = [];
            instData = [];
        end
    end

These lines check what angle of attack and which scenario (high speed versus low speed) is present. Then, it pulls out the x and y forces, the moment about the z-axis and the baratron voltage for this instance in the scenario. It stores those values into a vector for later. Then, the code checks to ensure the angle of attack changes on the next iteration and averages the vectors for the x and y forces, the moment, and the voltage. It also calculates the standard deviations of these means. It stores all of this data in a vector called instData and then a larger matrix called data that will hold all of the vectors for each instance of the larger for loop. It then resets the looping variables and moves to the next line.

There are many if statements that verify the attack angle and whether it is high or low speed. All of these statements follow the same premise as this chunk of code.

    aH = dataHigh(:,1);
    xH = dataHigh(:,2);
    yH = dataHigh(:,4);
    TzH = dataHigh(:,6);
    VH = dataHigh(:,8);

These lines separate out the aerodynamic values from the data matrix.

Note that the "H" subscript represents the high speed instance of the data with the endplate present.

    dragH = -xH.*sind(aH)+yH.*cosd(aH);
    liftH = -xH.*cosd(aH)-yH.*sind(aH);
    PH = -TzH;

These lines use equations (1), (2), and (3) to turn the raw data into a drag force vector, a lift force vector, and a pitching moment vector.

    qinf = VH*1.016*133.3223684; %Pa
    vinf = (qinf*2/rho0).^(1/2); %m/s
    v = mean(vinf)

These lines serve as a verification of the mathematical and data-taking processes. They first convert the Baratron voltage from volts to the dynamic pressure in Pascals. Then, using the static density collected by the manometer and that dynamic pressure to find the free-stream velocity of the air. Averaging these values yielded a value of v = 15.8926 m/s for the high speed, plated instance. The average speed noted during this trial was about 15.7 m/s.

    a1 = aH;
    CL1 = liftH./(qinf*Sp);
    CD1 = dragH./(qinf*Sp);
    CM1 = PH./(qinf*Sp*c);

Now, using the free-stream dynamic pressure and geometries of the wing (S being the span of the wing from one tip to the other, c being the chord of the wing from leading edge to trailing edge), these lines normalize the lift, drag, and moment vectors into vectors of coefficients of lift, drag, and moment, respectively.

    uCL1 = sqrt((-cosd(aH)./qinf * uL).^2 + (-sind(aH)./qinf * uL).^2 + ((xH.*cosd(aH)+yH.*sind(aH))./(qinf.*VH) * uV).^2);
    uCD1 = sqrt((cosd(aH)./qinf * uL).^2 + (-sind(aH)./qinf * uL).^2 + ((xH.*sind(aH)-yH.*cosd(aH))./(qinf.*VH) * uV).^2);
    uCM1 = sqrt((-1./qinf * uT).^2 + (TzH./(qinf .* VH * c) * uV).^2);

These lines generate an estimated error based on the precision and systematic bias theory of uncertainty. It uses partial derivatives of the detemining equations of each variable multiplied by the uncertainty in the raw measurement of the determining variable to give a systematic bias uncertainty. The precision uncertainty is chosen based on the 95% certainty yielding a factor of 1.96 times the standard deviation of a data set normalized by the root of the number of data in the set. The exact values of the uncertainty are not of concern, but are represented by error bars on the following graphs.

    figure
    hold on
    errorbar(a1, CL1, uCL1, 'horizontal')
    errorbar(a2, CL2, uCL2, 'horizontal')
    errorbar(a3, CL3, uCL3, 'horizontal')
    errorbar(a4, CL4, uCL4, 'horizontal')
    title('C_L for High/Low Speed With/Without End Plate')
    xlabel('Angle of Attack, \alpha (deg)')
    ylabel('Coefficient of Lift, C_L')
    legend('High Speed, Plate','Low Speed, Plate','High Speed, No Plate','Low Speed, No Plate','Location','Northwest','FontSize',6)

These lines graph the coefficients of lift for each of the four scenarios depicted over the angles of attack:

![Lift data](https://github.com/user-attachments/assets/46ce222d-ddcd-4422-9603-4b2b06c22fd9)

Notice the drops in each graph. This is where stalling, the phenomenon where lift is no longer generated due to a wing holding too high of an angle of attack, happens. This gives a pilot a general idea of the safe operating angles of the wing being studied. It is also important to note that the higher speeds stall at higher angles of attack.

    figure
    hold on
    errorbar(a1, CD1, uCD1, 'horizontal')
    errorbar(a2, CD2, uCD2, 'horizontal')
    errorbar(a3, CD3, uCD3, 'horizontal')
    errorbar(a4, CD4, uCD4, 'horizontal')
    title('C_D for High/Low Speed With/Without End Plate')
    xlabel('Angle of Attack, \alpha (deg)')
    ylabel('Coefficient of Drag, C_D')
    legend('High Speed, Plate','Low Speed, Plate','High Speed, No Plate','Low Speed, No Plate','Location','Northwest','FontSize',6)

Similarly, these lines graph the coefficients of drag for each scenario:

![Drag data](https://github.com/user-attachments/assets/c96d360c-2468-4b12-a78f-d19818b59302)

This figure demonstrates that drag increases at stalling at the same time that lift decreases.

    figure
    hold on
    errorbar(CD1, CL1, -uCD1, uCD1, -uCL1, uCL1)
    errorbar(CD2, CL2, -uCD2, uCD2, -uCL2, uCL2)
    errorbar(CD3, CL3, -uCD3, uCD3, -uCL3, uCL3)
    errorbar(CD4, CL4, -uCD4, uCD4, -uCL4, uCL4)
    title('Drag Polar for High/Low Speed With/Without End Plate')
    xlabel('Coefficient of Drag, C_D')
    ylabel('Coefficient of Lift, C_L')
    legend('High Speed, Plate','Low Speed, Plate','High Speed, No Plate','Low Speed, No Plate','Location','Northwest','FontSize',6)

These lines graph the coefficients of lift versus the corresponding coeffiicents of drag. This type of graph is called the drag polar, as these values against one another tend to make a polar-type graph with heavy curvature. The experiments in particular yielded:

![Drag polar](https://github.com/user-attachments/assets/0ba03ea0-ca1f-4672-a2c8-b528cbefb371)

The biggest point of note out of this graph is that the end plate lines are higher coefficients of lift at each coefficient of drag compared to the lines of no end plate. This demonstrates that the end plate of the wing generates more lift than drag, an important notion for flight.

    figure
    hold on
    errorbar(a1, CM1, uCM1, 'horizontal')
    errorbar(a2, CM2, uCM2, 'horizontal')
    errorbar(a3, CM3, uCM3, 'horizontal')
    errorbar(a4, CM4, uCM4, 'horizontal')
    title('C_M for High/Low Speed With/Without End Plate')
    xlabel('Angle of Attack, \alpha (deg)')
    ylabel('Coefficient of the Pitching Moment, C_M')
    legend('High Speed, Plate','Low Speed, Plate','High Speed, No Plate','Low Speed, No Plate','Location','Northwest','FontSize',6)

Finally, these lines plot the coefficient of the pitching moment against the angle of attack:

![Pitching moment](https://github.com/user-attachments/assets/57ad3403-7152-485c-8e2e-6262e65b2be0)

This graph demonstrates the degree of pitch present at different angles of attack. This is important to be accounted for during flight, and the discovery of this data through testing will allow a pilot during flight to account for such additional moments on the aircraft's wings.

## Conclusion
Thus, this experiment demonstrated that higher speeds and the presence of a turblence-preventing end plate provides benefits to a wing in the form of additional lift, moreso than the additional drag that also may be induced. This is important if the test wing is to then be used in actual flight, as an idea of wing behavior at particular angles of attack was determined. This behavior was most notable in the stalling that occured at 19.152°, 13.104°, 18.144°, and 12.096° for the high and low speeds with the end plate and the high and low speeds without the end plate, respectively. And, thus, I had successfully analyzed the effects of aerodynamic loads on a wing.

The results of this experiment are found in more detail in the lab report that I submitted for class (posted to this repository). Similarly, the raw MATLAB script is posted in this repository.

**Overall, this experiment was a fantastic exercise. Out of the three laboratories this AE2610 class mandated, aerodynamic loads was above and beyond the most challenging and rewarding. It required me to familiarize myself with proper lab equipment relevant to the aerospace field. Then, I had to obtain data from this equipment and properly analyze it in a coding base like MATLAB. The MATLAB script was easily the longest I had written before, requiring over 2000 lines of code. Granted, there are many improvements that can be made to this code. One could simply loop through half the code a second time with the other excel file and cut out ~800 lines of code. The function would be the same, but it would create a more efficient script. Moreover, there exist ways to avoid excessive conditionals throughout the program like using a boolean check and having the code look for changes in the angles of attack. However, in the creation of this script, I was under a time crunch. Thus, I was not only focused on code efficiency. I instead focused on function first over form. If I was given some time to revise this code, though, I would certainly make it more efficient. All in all, this experiment was a wonderful chance to hone my class skills and learn a lot about the field of aerospace engineering, particularly regarding laboratory research.**
