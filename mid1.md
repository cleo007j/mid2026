#1번 답안
```
<img width="893" height="691" alt="image" src="https://github.com/user-attachments/assets/922505d0-0223-4b52-a802-fcf6e3c68361" />
#include "raylib.h"
#include <stdbool.h>
#include <stdio.h>

#define SCREEN_WIDTH 800
#define SCREEN_HEIGHT 600

#define PLAYER_WIDTH 100
#define PLAYER_HEIGHT 20
#define PLAYER_SPEED 8.0f

#define BALL_RADIUS 10.0f
#define INITIAL_BALL_SPEED 5.0f

#define ROWS 7
#define COLS 10
#define BLOCK_WIDTH 70
#define BLOCK_HEIGHT 20
#define BLOCK_PADDING 10
#define OFFSET_X 45
#define OFFSET_Y 60

typedef struct Player {
    Rectangle rect;
    float speed;
} Player;

typedef struct Ball {
    Vector2 position;
    Vector2 speed;
    float radius;
    bool active;
} Ball;

typedef struct Block {
    Rectangle rect;
    Color color;
    bool active;
} Block;

void ResetBlocks(Block blocks[ROWS][COLS], Color rainbowColors[ROWS]) {
    for (int i = 0; i < ROWS; i++) {
        for (int j = 0; j < COLS; j++) {
            blocks[i][j].rect = (Rectangle){
                (float)(OFFSET_X + j * (BLOCK_WIDTH + BLOCK_PADDING)),
                (float)(OFFSET_Y + i * (BLOCK_HEIGHT + BLOCK_PADDING)),
                (float)BLOCK_WIDTH,
                (float)BLOCK_HEIGHT
            };
            blocks[i][j].color = rainbowColors[i];
            blocks[i][j].active = true;
        }
    }
}

int main(void) {
    // Initialization
    InitWindow(SCREEN_WIDTH, SCREEN_HEIGHT, "Rainbow Block Breaker");
    SetTargetFPS(60);

    // Game States: 0 = Menu, 1 = Playing, 2 = GameOver, 3 = Victory
    int gameState = 0;
    int score = 0;
    int lives = 3;

    Player player = {
        { (SCREEN_WIDTH - PLAYER_WIDTH) / 2.0f, SCREEN_HEIGHT - 50.0f, (float)PLAYER_WIDTH, (float)PLAYER_HEIGHT },
        PLAYER_SPEED
    };

    Ball ball = {
        { SCREEN_WIDTH / 2.0f, SCREEN_HEIGHT - 70.0f },
        { 0.0f, 0.0f },
        BALL_RADIUS,
        false
    };

    // Rainbow colors mapping 7 rows
    Color rainbowColors[ROWS] = {
        RED,
        ORANGE,
        YELLOW,
        GREEN,
        BLUE,
        DARKBLUE,
        PURPLE
    };

    Block blocks[ROWS][COLS];
    ResetBlocks(blocks, rainbowColors);

    while (!WindowShouldClose()) {
        // Update Logic
        switch (gameState) {
            case 0: // Menu State
                if (IsKeyPressed(KEY_ENTER)) {
                    gameState = 1;
                    score = 0;
                    lives = 3;
                    ResetBlocks(blocks, rainbowColors);
                    ball.position = (Vector2){ player.rect.x + player.rect.width / 2.0f, player.rect.y - BALL_RADIUS - 2.0f };
                    ball.speed = (Vector2){ 0.0f, 0.0f };
                    ball.active = false;
                }
                break;

            case 1: // Playing State
                // Player movement (Arrow keys or A/D keys)
                if (IsKeyDown(KEY_LEFT) || IsKeyDown(KEY_A)) player.rect.x -= player.speed;
                if (IsKeyDown(KEY_RIGHT) || IsKeyDown(KEY_D)) player.rect.x += player.speed;

                // Screen boundaries for player
                if (player.rect.x < 0) player.rect.x = 0;
                if (player.rect.x + player.rect.width > SCREEN_WIDTH) player.rect.x = SCREEN_WIDTH - player.rect.width;

                // Ball launch / positioning
                if (!ball.active) {
                    ball.position.x = player.rect.x + player.rect.width / 2.0f;
                    ball.position.y = player.rect.y - ball.radius - 2.0f;
                    if (IsKeyPressed(KEY_SPACE)) {
                        ball.active = true;
                        ball.speed = (Vector2){ INITIAL_BALL_SPEED, -INITIAL_BALL_SPEED };
                    }
                } else {
                    // Ball movement
                    ball.position.x += ball.speed.x;
                    ball.position.y += ball.speed.y;

                    // Wall collisions (Left, Right, Top)
                    if (ball.position.x - ball.radius < 0) {
                        ball.position.x = ball.radius;
                        ball.speed.x *= -1.0f;
                    }
                    if (ball.position.x + ball.radius > SCREEN_WIDTH) {
                        ball.position.x = SCREEN_WIDTH - ball.radius;
                        ball.speed.x *= -1.0f;
                    }
                    if (ball.position.y - ball.radius < 0) {
                        ball.position.y = ball.radius;
                        ball.speed.y *= -1.0f;
                    }

                    // Bottom boundary collision (Lose life)
                    if (ball.position.y + ball.radius > SCREEN_HEIGHT) {
                        lives--;
                        ball.active = false;
                        if (lives <= 0) {
                            gameState = 2; // Game Over
                        }
                    }

                    // Paddle collision
                    if (CheckCollisionCircleRec(ball.position, ball.radius, player.rect)) {
                        ball.speed.y *= -1.0f;
                        float hitFactor = (ball.position.x - (player.rect.x + player.rect.width / 2.0f)) / (player.rect.width / 2.0f);
                        ball.speed.x = hitFactor * INITIAL_BALL_SPEED * 1.5f;
                        ball.position.y = player.rect.y - ball.radius - 1.0f;
                    }

                    // Block collisions & victory check
                    bool allCleared = true;
                    for (int i = 0; i < ROWS; i++) {
                        for (int j = 0; j < COLS; j++) {
                            if (blocks[i][j].active) {
                                allCleared = false;
                                if (CheckCollisionCircleRec(ball.position, ball.radius, blocks[i][j].rect)) {
                                    blocks[i][j].active = false;
                                    ball.speed.y *= -1.0f;
                                    score += (ROWS - i) * 10;
                                }
                            }
                        }
                    }

                    if (allCleared) {
                        gameState = 3; // Victory
                    }
                }
                break;

            case 2: // Game Over State
            case 3: // Victory State
                if (IsKeyPressed(KEY_ENTER)) {
                    gameState = 0;
                }
                break;
        }

        // Render Graphics
        BeginDrawing();
        ClearBackground(RAYWHITE);

        if (gameState == 0) {
            DrawText("RAINBOW BLOCK BREAKER", SCREEN_WIDTH / 2 - MeasureText("RAINBOW BLOCK BREAKER", 30) / 2, SCREEN_HEIGHT / 2 - 60, 30, DARKGRAY);
            DrawText("PRESS ENTER TO START", SCREEN_WIDTH / 2 - MeasureText("PRESS ENTER TO START", 20) / 2, SCREEN_HEIGHT / 2, 20, GRAY);
            DrawText("USE LEFT/RIGHT ARROWS OR A/D TO MOVE", SCREEN_WIDTH / 2 - MeasureText("USE LEFT/RIGHT ARROWS OR A/D TO MOVE", 16) / 2, SCREEN_HEIGHT / 2 + 50, 16, LIGHTGRAY);
            DrawText("PRESS SPACE TO LAUNCH BALL", SCREEN_WIDTH / 2 - MeasureText("PRESS SPACE TO LAUNCH BALL", 16) / 2, SCREEN_HEIGHT / 2 + 80, 16, LIGHTGRAY);
        } 
        else if (gameState == 1) {
            DrawRectangleRec(player.rect, DARKBLUE);
            DrawCircleV(ball.position, ball.radius, RED);

            for (int i = 0; i < ROWS; i++) {
                for (int j = 0; j < COLS; j++) {
                    if (blocks[i][j].active) {
                        DrawRectangleRec(blocks[i][j].rect, blocks[i][j].color);
                        DrawRectangleLinesEx(blocks[i][j].rect, 1, Fade(BLACK, 0.3f));
                    }
                }
            }

            DrawText(TextFormat("SCORE: %d", score), 20, 20, 20, DARKGRAY);
            DrawText(TextFormat("LIVES: %d", lives), SCREEN_WIDTH - 120, 20, 20, DARKGRAY);
        } 
        else if (gameState == 2) {
            DrawText("GAME OVER", SCREEN_WIDTH / 2 - MeasureText("GAME OVER", 40) / 2, SCREEN_HEIGHT / 2 - 40, 40, RED);
            DrawText(TextFormat("FINAL SCORE: %d", score), SCREEN_WIDTH / 2 - MeasureText(TextFormat("FINAL SCORE: %d", score), 20) / 2, SCREEN_HEIGHT / 2 + 10, 20, DARKGRAY);
            DrawText("PRESS ENTER TO RETURN TO MENU", SCREEN_WIDTH / 2 - MeasureText("PRESS ENTER TO RETURN TO MENU", 16) / 2, SCREEN_HEIGHT / 2 + 50, 16, GRAY);
        } 
        else if (gameState == 3) {
            DrawText("VICTORY!", SCREEN_WIDTH / 2 - MeasureText("VICTORY!", 40) / 2, SCREEN_HEIGHT / 2 - 40, 40, GREEN);
            DrawText(TextFormat("FINAL SCORE: %d", score), SCREEN_WIDTH / 2 - MeasureText(TextFormat("FINAL SCORE: %d", score), 20) / 2, SCREEN_HEIGHT / 2 + 10, 20, DARKGRAY);
            DrawText("PRESS ENTER TO RETURN TO MENU", SCREEN_WIDTH / 2 - MeasureText("PRESS ENTER TO RETURN TO MENU", 16) / 2, SCREEN_HEIGHT / 2 + 50, 16, GRAY);
        }

        EndDrawing();
    }

    CloseWindow();
    return 0;
}
