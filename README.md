#TL:DL -  AMS-Differential-DC-Voltmeter
Arduino based Automatic Measurement System that can measure voltage and current in the ranges: {-500mV... +500mV}, {-5V... +5V} (changeable)

July 18 2021, Roy Ben Avraham

Goal: Design Arduino - based Automatic Measurement System that can measure voltage and current in the ranges (changable)

[in this specific case:{-500mV...+500mV}, {-5V...+5V} ].

It automatically selects proper range and re-connects proper elements of AMS (to enable optimal accuracy with the Arduino ADC)

The AMS, must grab a value to be measured at least 5 times and present mean value and its standard deviation Mean values and variations are presented on the a serial monitor by using relevant units and must be updated once per 5 seconds.

One must be aware enough to know that sqrt operation is expensive with respect to time - therefore I will calculate here the squared STR (standard deviation).

System's blocks:
Block number 1: 2 input buffers and a differential amplifier

![image](https://user-images.githubusercontent.com/105777016/169131100-3b5fd48b-eedf-4407-9c0c-509eae815653.png)

This is our system input, two input signals are being entered through the buffers (for a large input impedance) and then the differential amplifier

Outputs the voltage: V+ - V-

The blue line represents the V+ and the grey line represents the V- voltage.

![image](https://user-images.githubusercontent.com/105777016/169131304-ebbeace2-dde0-4b9e-a420-486f179e998d.png)

How we chose our resistors for the differential amplifier:

We want to get Vout=V+-V- and therefore we chose our resistors in the following way:

R1=R2=R3=R4=500[kΩ].

We connected the differential amplifier output to a buffer to maintain it’s voltage.

Note:

All operational amplifiers in this block have ±5[V] supply voltages.

In order to implement the ±5[V] we used two power supplies as shown in the figure below:

![image](https://user-images.githubusercontent.com/105777016/169131580-cb256b38-6111-48b3-b66c-207e7131fdb8.png)

Block number 2: Summing amplifier for the range of ±5[V]

![image](https://user-images.githubusercontent.com/105777016/169131758-2dc89aad-85ef-4021-9015-bb83e2c54d4f.png)

This block’s job is to convert the differential voltage in the range of {-5[V]…+5[V]} to {0[V]…5[V]}.

We need to do that conversion because an Arduino can’t be plugged to negative voltages.

The output function of this amplifier has to be Vout=(5[V]+Vin)/2

In order to do this conversion.

![image](https://user-images.githubusercontent.com/105777016/169131983-64c4574d-0972-4aa1-af4e-b6bd1b5c8056.png)

How we chose our resistors for the differential amplifier:

The inputs are V1 and V2, V2=5[V],V1=Vin (this is the output

From the previous block 1) therefore in order to get

Amplifier output function we chose the resistors accordingly.

RA=100[Ω],RB=500[kΩ],R1=R2=500[kΩ].

After we chose all of those resistors, we receive the following output

Function:

Vout=(5[V]+Vin)/2

Because RB<<RA he has a negligible contribution to the amplifiers gain.

Block number 3: Summing amplifier for the range of ±500[mV] with a buffer

![image](https://user-images.githubusercontent.com/105777016/169132316-41de6b1f-61a1-474a-961c-a3df6bc5f237.png)

This block’s job is to convert the differential voltage in the range of {500[mV]…+500[mV]} to {0[V]…500[mV]}.

The output function of this amplifier has to be Vout=(500[mV]+Vin)/2 *10

In order to do this conversion.

We used a voltage divider and a buffer, therefore the point of the voltage divider is going into the non-inverting input of the operational amplifier. All of that was made in order to maintain a voltage of 500[mV] that we need for this function.

How we chose our resistors for the differential amplifier:

The inputs are V1 and V2, V2=500[mV],V1=Vin (this is the output

From the previous block 1) therefore in order to get

Amplifier output function we chose the resistors accordingly.

RA=450[kΩ],RB=50[kΩ],R1=R2=500[kΩ].

After we chose all of those resistors, we receive the following output

Function:

Vout=(500[mV]+Vin)/2 *10

Note1: 1+RA/RB = 10 which we are multiplying with.

Note2: For the voltage divider mentioned on the previous slide, we used the following resistors as shown in the figure below:

Block number 4: Differential voltage rectifiers

![image](https://user-images.githubusercontent.com/105777016/169132740-62e445ed-2333-4fb4-8f82-825386b0222f.png)

Block number 4: Differential voltage rectifiers
               
               ![image](https://user-images.githubusercontent.com/105777016/169133092-ab7a4211-c6be-448b-ba6b-bb68f581f326.png)


Why we are using rectifiers anyway?:

This block is used in the next block (block 5) this block’s output is an input to the non inverting input of the comparator OA.

![image](https://user-images.githubusercontent.com/105777016/169133417-f2ff1609-f415-4007-bf27-4ec50c7a27ca.png)
               
We have implemented the circuit above in the following way:

When our input signal (output from block number 1) is greater than 0, A1 amplifier behaves as a buffer.

Both input of A2 amplifier are at the same potential as the input signal, so A2 actually outputs the positive signal.

When the signal is less than 0, the output of A1 equals to 0[V] and A2 will invert the input signal.

The overall result is the absolute value of the input signal.

All the resistors that were used in this block are equal to R=500[kΩ].

Block number 5: Comperator OA with a relay

               ![image](https://user-images.githubusercontent.com/105777016/169133782-b358bb42-ebf1-4b81-9491-05117e45fce4.png)
              
The comparators input in V+ is the rectified differential output voltage from the previous block number 4.

And on V- we have 500[mV] from the buffer amplifier mention on block 3.

By that we can choose from are two ranges: {-5[V]…+5[V]},{-500[mV]…+500[mV]}.

If the absolute value of the differential voltage is greater than 500[mV] the relay will output the output of block 2 which handle this range of voltages.

And if the absolute value of the differential voltage is less than 500[mV] the relay will output the output of block 3.

The relays output (green wire) goes to A0 and the orange wire goes to pin 2.

System's flowchart:
![image](https://user-images.githubusercontent.com/105777016/169134176-be50a36b-d817-4031-96cc-3ce52328a25f.png)
  
  ![image](https://user-images.githubusercontent.com/105777016/169134324-64964ee1-7149-42e8-8b8b-f14c5b00364c.png)
  
In the attachments above, we can see for different ranges of voltage that no matter what voltage we have, the output current is always 0[A].

Dividing it with 0[A] output current will lead to Rinput = (Voltage(in)[V])/(Current (in)[A])=(Voltage(in)[V})/(0[A]) = ∞

Meaning that in the ranges we were requested to design this AMS, Differential DC Voltmeter our input impedance is practically infinite.

In order to achieve that result we used Buffer operational amplifier (for both inputs) which is known that one of his characteristics is that his Rinput equals to infinity.

Note: even if the voltage==0[v] we get 0/0 = ‘not defined’, but it doesn’t matter since anyway there is no connection (voltage nor current) in this case.

Electrical circuit:
  ![image](https://user-images.githubusercontent.com/105777016/169134628-06383edd-d69f-4218-a46a-667505347302.png)


System Layout:
  ![image](https://user-images.githubusercontent.com/105777016/169134960-bde73ab4-6421-4cbb-b838-450ad47c389a.png)




