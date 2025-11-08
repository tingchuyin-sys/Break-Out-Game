#include <SPI.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1331.h>
#include "pitches.h" // Include pitches.h for note frequencies

// Pins for the OLED display
#define sclk 13
#define mosi 11
#define cs   10
#define rst  9
#define dc   8

// Pins for the joystick
#define JOYSTICK_X A0
#define JOYSTICK_Y A1
#define JOYSTICK_BTN 2

// Pins for the active buzzer
const int buzzerPin = 7;

// Pins for LEDs
const int ledRedPin = 5;   // Red LED for loss
const int ledYellowPin = 6; // Yellow LED for win

// Pins for the buttons
const int startResetButtonPin = 3;
const int pauseResumeButtonPin = 4;

// Define colors
#define BLACK     0x0000
#define WHITE     0xFFFF
#define RED       0xF800
#define GREEN     0x07E0
#define BLUE      0x001F
#define CYAN      0x07FF
#define MAGENTA   0xF81F
#define YELLOW    0xFFE0
#define ORANGE    0xFD20

Adafruit_SSD1331 display = Adafruit_SSD1331(cs, dc, rst);

const int JOYSTICK_X_PIN = A0; // X-axis connected to analog pin A0
const int JOYSTICK_Y_PIN = A1; // Y-axis connected to analog pin A1 (not used in this example)

int paddleX;
const int paddleWidth = 20;
const int paddleHeight = 4;
const int paddleY = 58;

int ballX, ballY;
int ballDX = 1.3;
int ballDY = 1.3;
const int ballSize = 3;

const int brickRows = 3;
const int brickCols = 8;
const int brickWidth = 14;
const int brickHeight = 5;
const int brickStartY = 12; // Starting Y position for bricks
bool bricks[brickRows][brickCols];
bool allBricksDestroyed;

int score = 0;
int lives = 3;
bool gameOver = false;
bool gameWon = false;
bool gameStarted = false; // Flag to track if the game has started
bool gamePaused = false;  // Flag to track if the game is paused

// Define "Congratulations" music notes and durations
int congratsMelody[] = {
  NOTE_C5, NOTE_D5, NOTE_E5, NOTE_C5, NOTE_D5, NOTE_E5,
  NOTE_F5, NOTE_E5, NOTE_D5, NOTE_C5
};

int congratsNoteDurations[] = {
  4, 4, 4, 4, 4, 4,
  4, 4, 4, 4
};

// Define "Game Over" music notes and durations
int gameOverMelody[] = {
  NOTE_C4, NOTE_G3, NOTE_E3, NOTE_A3, NOTE_B3, NOTE_A3, NOTE_A3,
  NOTE_GS3, NOTE_AS3, NOTE_GS3, NOTE_G3, NOTE_F3, NOTE_G3
};

int gameOverNoteDurations[] = {
  8, 8, 8, 8, 4, 4, 4,
  8, 8, 8, 8, 8, 8
};

// Define "Brick Hit" sound note and duration
const int brickHitNote = NOTE_C6;
const int brickHitDuration = 100; // Duration in milliseconds

void setup() {
  Serial.begin(9600);
  pinMode(JOYSTICK_X_PIN, INPUT);
  pinMode(JOYSTICK_Y_PIN, INPUT);
  pinMode(buzzerPin, OUTPUT);
  pinMode(ledRedPin, OUTPUT);     // Initialize Red LED pin as output
  pinMode(ledYellowPin, OUTPUT);  // Initialize Yellow LED pin as output

  // Initialize button pins as input with pull-up resistors
  pinMode(startResetButtonPin, INPUT_PULLUP); 
  pinMode(pauseResumeButtonPin, INPUT_PULLUP); 

  display.begin();
  display.fillScreen(BLACK); // Clear the display with black color
  
  paddleX = (display.width() - paddleWidth) / 2;
  ballX = display.width() / 2;
  ballY = display.height() / 2;

  resetGame(); // Initialize the game state
}

void loop() {
  if (digitalRead(startResetButtonPin) == LOW) {
    if (gameStarted) {
      resetGame();
    } else {
      gameStarted = true;
    }
    delay(200); // Debounce delay
  }

  if (digitalRead(pauseResumeButtonPin) == LOW) {
    gamePaused = !gamePaused;
    delay(200); // Debounce delay
  }

  if (!gameStarted) {
    displayStartMessage();
    return;
  }

  if (gamePaused) {
    displayPauseMessage();
    return;
  }

  if (score == 180) {
    displayYOUWON();
    digitalWrite(ledRedPin, LOW); // Turn off red LED
    digitalWrite(ledYellowPin, HIGH); // Turn on yellow LED for win
    playCongratsTune(); // Play the congratulatory tune
    digitalWrite(ledYellowPin, LOW); 
    resetGame();
    return; // Exit loop after playing the
  }
  
  if(gameOver){
      digitalWrite(ledRedPin, HIGH); // Turn on red LED for loss
      digitalWrite(ledYellowPin, LOW); // Turn off yellow LED
      displayGameOver();
      playGameOverTune(); // Play the "Game Over" tune
      digitalWrite(ledRedPin, LOW);
      resetGame();
      return;
  }
      
  // Read joystick input
  int joystickX = analogRead(JOYSTICK_X_PIN);

  // Map joystick input to paddle position
  paddleX = map(joystickX, 0, 1023, 0, display.width() - paddleWidth);

  // Move the ball
  ballX += ballDX;
  ballY += ballDY;

  // Ball collision with walls
  if (ballX <= 0 || ballX >= display.width() - ballSize) {
    ballDX = -ballDX;
  }
  if (ballY <= 0) {
    ballDY = -ballDY;
  }

  if (ballY >= paddleY - ballSize && ballX + ballSize >= paddleX && ballX <= paddleX + paddleWidth) {
    ballDY = -ballDY;
  }

  // Ball out of bounds (lose condition)
  if (ballY > display.height()) {
    ballX = display.width() / 2;
    ballY = display.height() / 2;
    ballDX = 1.3;
    ballDY = 1.3;
    lives--; // Decrement lives when ball is lost
    if (lives == 0) {
      gameOver = true;
    }
  }

  // Ball collision with bricks
  for (int i = 0; i < brickRows; i++) {
    for (int j = 0; j < brickCols; j++) {
      if (bricks[i][j]) {
        int brickX = j * (brickWidth + 2);
        int brickY = brickStartY + i * (brickHeight + 2);
        if (ballX + ballSize > brickX && ballX < brickX + brickWidth && ballY + ballSize > brickY && ballY < brickY + brickHeight) {
          ballDY = -ballDY;
          bricks[i][j] = false; // Remove the brick
          score += 10; // Increment score for each brick hit
          tone(buzzerPin, brickHitNote, brickHitDuration); // Play the brick hit sound
        }
      }
    }
  }

  // Clear display
  display.fillScreen(BLACK); // Clear the screen with black color

  // Draw game elements
  drawPaddle();
  drawBall();
  drawBricks(); // Draw the bricks
  drawScore();
  drawLives();

  // Delay to control game speed
  delay(30);
}

void drawPaddle() {
  display.fillRect(paddleX, paddleY, paddleWidth, paddleHeight, WHITE);
}

void drawLives() {
  int heartSize = 5; // Size of each heart
  for (int i = 0; i < lives; i++) {
    drawHeart(display.width() - (i + 1) * (heartSize + 2), 0, heartSize, RED);
  }
}

void drawBall() {
  display.fillRect(ballX, ballY, ballSize, ballSize, WHITE);
}

void drawBricks() {
  for (int i = 0; i < brickRows; i++) {
    for (int j = 0; j < brickCols; j++) {
      if (bricks[i][j]) {
        int brickX = j * (brickWidth + 2);
        int brickY = brickStartY + i * (brickHeight + 2);
        display.fillRect(brickX, brickY, brickWidth, brickHeight, WHITE);
      }
    }
  }
}

void drawScore() {
  display.setTextColor(WHITE);
  display.setCursor(0, 0);
  display.print("Score: ");
  display.print(score);
}

void drawHeart(int x, int y, int size, uint16_t color) {
  // Draw a simple heart shape
  display.fillTriangle(x, y, x + size / 2, y + size / 2, x - size / 2, y + size / 2, color);
  display.fillTriangle(x, y + size / 2, x + size / 2, y + size, x - size / 2, y + size, color);
  display.fillRect(x - size / 2, y + size / 4, size, size / 2, color);
}

void displayGameOver() {
  display.fillScreen(BLACK);
  display.setTextColor(RED);
  display.setCursor(display.width() / 2 - 30, display.height() / 2 - 10);
  display.setTextSize(1);
  display.print("Game Over");
}

void displayYOUWON() {
  display.fillScreen(BLACK);
  display.setTextColor(GREEN);
  display.setCursor(display.width() / 2 - 50, display.height() / 2 - 10);
  display.setTextSize(1);
  display.print("YOU WON!!!");
}

void playCongratsTune() {
  int pauseBetweenNotes = 100;

  for (int thisNote = 0; thisNote < 10; thisNote++) {
    int noteDuration = 1000 / congratsNoteDurations[thisNote];
    tone(7, congratsMelody[thisNote], noteDuration);
    delay(noteDuration * 1.30);
    noTone(7);
    delay(pauseBetweenNotes);
  }
}

void playGameOverTune() {
  int pauseBetweenNotes = 100;

  for (int thisNote = 0; thisNote < 13; thisNote++) {
    int noteDuration = 1000 / gameOverNoteDurations[thisNote];
    tone(7, gameOverMelody[thisNote], noteDuration);
    delay(noteDuration * 1.30);
    noTone(7);
    delay(pauseBetweenNotes);
  }
}

void displayStartMessage() {
  display.fillScreen(BLACK);
  display.setTextColor(WHITE);
  display.setCursor(display.width() / 2 - 30, display.height() / 2 - 10);
  display.setTextSize(1);
  display.print("Press Start");
}

void displayPauseMessage() {
  display.fillScreen(BLACK);
  display.setTextColor(WHITE);
  display.setCursor(display.width() / 2 - 30, display.height() / 2 - 10);
  display.setTextSize(1);
  display.print("Paused");
}

void resetGame() {
  paddleX = (display.width() - paddleWidth) / 2;
  ballX = display.width() / 2;
  ballY = display.height() / 2;
  ballDX = 1.3;
  ballDY = 1.3;
  score = 0;
  lives = 3;
  gameOver = false;
  gameWon = false;
  gameStarted = false;
  gamePaused = false;

  // Initialize bricks array
  for (int i = 0; i < brickRows; i++) {
    for (int j = 0; j < brickCols; j++) {
      bricks[i][j] = true; // All bricks start as active
    }
  }
}
