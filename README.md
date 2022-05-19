# Seismometer

## by Yair Elbaum & Idan Alperin

In this project we have an MPU6050 accelerometer communicating with our
MCU over I2C. Whenever A noise is detected above a programmable threshold,
an interrupt is raised and the mcu starts sending accelerometer data.

------------------------------------------------------------

Calibration
------------------------------------------------------------

The calibration takes a predefined number of samples (1024) and computes an average
for each of the axis (denoted as $E[x], E[y], E[z]$).
In order to make sure the samples are good, we make sure the variance of the samples
is below a certain threshold.
After sampling, we take the vector $\begin{bmatrix} E[x] \\ E[y]\\ E[z] \end{bmatrix} $and consider it our true Z vector.
In order to compute our true X & Y vectors we need to compute 2 orthogonal vectors to Z.
The X vector can be achieved easily via taking the vector:

$$
\begin{bmatrix} E[y] \\ -E[x] \\ 0 \end{bmatrix}
$$

The Y vector can be computed as the cross product of Z & X, which is 

$$
\begin{bmatrix} E[x] * E[z]\\ E[y] * E[y] \\ -(E[x]^2 + E[y]^2)\end{bmatrix}
$$

Now all that is left to do is create a basis change matrix and multiply any reading we
have with this matrix.

Earthquake handling
------------------------------------------------------------

On register 0x1F of MPU6050, we can define a threshold for the MPU's interrupt pin to
turn high, subsequently causing an interrupt in the MCU. once the interrupt is received,
the MCU takes samples in a predefined interval, processes them, and batches them up
using collector.h. Once a batch is full, it is sent via mqtt to a broker.

Once the samples variance falls below a threshold, the event is over, the MPU's interrupt
is cleared and the MCU goes idle.

Remote configuration
------------------------------------------------------------

The MCU is subscribed to a certain topic (tonto2/set), and upon receiving a message
will update MPU6050's threshold register (0x1F) to the specified value. Thus, the
sensitivity can be remotely adjusted.

Pinging
------------------------------------------------------------

Every 30 seconds of inactivity, a ping is sent to the server in order to keep the
connection alive. This is implemented using a timer & an interrupt.

Buttons
------------------------------------------------------------

Pressing BTN0 will force a calibration event.
Pressing BTN1 will make the cat mewo.