Breakout Game (Arduino + Embedded System)

📘 Project Description

This project is developed as part of an Embedded Systems C Programming assignment. We were required to design a C-program that runs an application on an embedded system using at least three inputs and producing at least three different outputs.

Our implementation is a Breakout-style arcade game built on Arduino Uno, where the player controls a paddle to bounce a ball and break bricks.

When the start button is pressed, the game begins — the player uses two control buttons to move the paddle left and right, keeping the ball in play. The ball bounces off the walls, paddle, and bricks, and every collision with a brick increases the score.

A set of bricks is arranged in a grid, and the goal is to clear all bricks without losing all lives. Each time the player misses the ball, one life is lost (out of three). The player wins when all bricks are cleared and loses when all lives are depleted.

Inputs and Outputs

The system utilizes three main input components and four distinct output components to achieve interactive gameplay functionality and feedback control. The table below summarizes each component and its corresponding function within the game.

| **Input Component**       | **Function**                                                                                                                                                    |
| ------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Start / Reset Button**  | Initiates the game if it has not yet begun. When pressed during gameplay, it resets the system and restarts the game from the beginning.                        |
| **Pause / Resume Button** | Temporarily pauses the game when it is running. Pressing it again resumes the game from the previous state, allowing users to continue gameplay seamlessly.     |
| **Analog Joystick**       | Controls the paddle’s movement along the horizontal axis, enabling the player to move the paddle left or right to deflect the ball and prevent it from falling. |

| **Output Component** | **Function**                                                                                                                                                                                                          |
| -------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Yellow LED**       | Lights up to indicate that the player has successfully cleared all bricks, signifying a **game win** condition.                                                                                                       |
| **Red LED**          | Activates when the player loses all available lives, indicating a **game over** condition.                                                                                                                            |
| **Buzzer**           | Produces distinct sound effects corresponding to various events, such as **brick collisions**, **game win**, and **game over**, enhancing player feedback.                                                            |
| **OLED Display**     | Serves as the main visual interface that displays **scores, remaining lives, and gameplay elements** such as the ball, paddle, and bricks. It also shows **status messages** (e.g., “Start”, “Pause”, “Win”, “Lose”). |
