# Dining Philosophers
## Dining Philosophers
Lastly, another common problem which demonstrates inter-process communication is the **dining philosophers** problem. Much like the producer-consumer problem, it is frequently used to demonstrate the effectiveness of a new synchronization methods.

> Imagine that five philosophers who spend their lives just thinking and eating. In the middle of the dining room is a circular table with five chairs. The table has a big plate of spaghetti. However, there are only five chopsticks available, as shown in the following figure. Each philosopher thinks. When he gets hungry, he sits down and picks up the two chopsticks that are closest to him. If a philosopher can pick up both chopsticks, he eats for a while. After a philosopher finishes eating, he puts down the chopsticks and starts to think.
This problem also demonstrates the issue of deadlock which could occur if every philosopher holds a left chopstick and waits perpetually for a right chopstick (or vice versa).

Other similar classical IPC problems include the **sleeping barber** problem and the **readers and writers** problem.
