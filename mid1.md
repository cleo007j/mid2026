#1번 답안
```
/*
 * Rainbow Block Breaker
 * C language + raylib (latest version) graphics library
 *
 * Build with Visual Studio Community + NuGet:
 *   1. Create a new "Console App" C++ project (or C project)
 *   2. Right-click project -> Manage NuGet Packages
 *   3. Search and install "raylib" (or use the raylib NuGet package)
 *   4. Build and run
 *
 * Alternative: use vcpkg: vcpkg install raylib
 * Or download prebuilt binaries from raylib.com
 */

#include "raylib.h"
#include <stdio.h>
#include <stdlib.h>
#include <math.h>

/* ───────────────────────────────────────────
   Constants
   ─────────────────────────────────────────── */
#define WINDOW_WIDTH    800
#define WINDOW_HEIGHT   600
#define PADDLE_WIDTH    120
#define PADDLE_HEIGHT   14
#define PADDLE_Y        (WINDOW_HEIGHT - 40)
#define BALL_RADIUS     10
#define BALL_SPEED      4.5
#define BRICK_ROWS      6
#define BRICK_COLS      8
#define BRICK_WIDTH     80
#define BRICK_HEIGHT    25
#define BRICK_PADDING   6
#define BRICK_TOP       60
#define BRICK_LEFT      ((WINDOW_WIDTH - (BRICK_COLS * (BRICK_WIDTH + BRICK_PADDING))) / 2)
#define MAX_BALLS       3

/* ───────────────────────────────────────────
   Rainbow colors (ROYGBIV)
   ─────────────────────────────────────────── */
typedef enum {
    COLOR_RED,
    COLOR_ORANGE,
    COLOR_YELLOW,
    COLOR_GREEN,
    COLOR_CYAN,
    COLOR_BLUE,
    COLOR_VIOLET,
    COLOR_COUNT
} RainbowColor;

static Color rainbow_colors[COLOR_COUNT] = {
    { 220,  50,  50, 255 },  /* Red     */
    { 230, 140,  30, 255 },  /* Orange  */
    { 240, 220,  40, 255 },  /* Yellow  */
    {  60, 190,  80, 255 },  /* Green   */
    {  30, 180, 200, 255 },  /* Cyan    */
    {  50,  80, 200, 255 },  /* Blue    */
    { 170,  60, 190, 255 },  /* Violet  */
};

static Color bg_color = { 20, 20, 35, 255 };
static Color text_color = { 255, 255, 255, 255 };

/* ───────────────────────────────────────────
   Structures
   ─────────────────────────────────────────── */
typedef struct {
    double x, y;        /* Center position */
    double vx, vy;      /* Velocity vector */
} Ball;

typedef struct {
    int x, y;           /* Top-left corner */
    int w, h;           /* Size */
    int alive;          /* 1: alive, 0: destroyed */
    RainbowColor color; /* Rainbow color */
} Brick;

typedef struct {
    int lives;          /* Remaining balls */
    int score;
    int level;          /* Current level */
    int game_over;
    int win;
} GameState;

/* ───────────────────────────────────────────
   Global variables
   ─────────────────────────────────────────── */
static Ball         g_balls[MAX_BALLS];
static int          g_ball_count  = 0;
static Brick        g_bricks[BRICK_ROWS][BRICK_COLS];
static int          g_paddle_x    = 0;  /* Top-left x */
static GameState    g_state;

/* ───────────────────────────────────────────
   Game object initialization
   ─────────────────────────────────────────── */
static void InitGame(void)
{
    /* Ball initialization: 1 ball at center-top */
    g_balls[0].x = (double)WINDOW_WIDTH / 2.0;
    g_balls[0].y = (double)PADDLE_Y - BALL_RADIUS - 1.0;
    g_balls[0].vx = (double)BALL_SPEED;
    g_balls[0].vy = -(double)BALL_SPEED;
    g_ball_count = 1;

    /* Paddle initialization: center bottom */
    g_paddle_x = (WINDOW_WIDTH - PADDLE_WIDTH) / 2;

    /* Brick initialization: fill with rainbow colors */
    for (int r = 0; r < BRICK_ROWS; r++) {
        for (int c = 0; c < BRICK_COLS; c++) {
            g_bricks[r][c].x = BRICK_LEFT + c * (BRICK_WIDTH + BRICK_PADDING);
            g_bricks[r][c].y = BRICK_TOP + r * (BRICK_HEIGHT + BRICK_PADDING);
            g_bricks[r][c].w = BRICK_WIDTH;
            g_bricks[r][c].h = BRICK_HEIGHT;
            g_bricks[r][c].alive = 1;
            /* Cycle rainbow colors per row */
            g_bricks[r][c].color = (RainbowColor)(r % COLOR_COUNT);
        }
    }

    /* Game state initialization */
    g_state.lives = MAX_BALLS;
    g_state.score = 0;
    g_state.level = 1;
    g_state.game_over = 0;
    g_state.win = 0;
}

/* ───────────────────────────────────────────
   Check if all bricks are destroyed
   ─────────────────────────────────────────── */
static int AllBricksDestroyed(void)
{
    for (int r = 0; r < BRICK_ROWS; r++) {
        for (int c = 0; c < BRICK_COLS; c++) {
            if (g_bricks[r][c].alive) {
                return 0;
            }
        }
    }
    return 1;
}

/* ───────────────────────────────────────────
   Ball-brick collision detection and handling
   ─────────────────────────────────────────── */
static void CheckBallBrickCollision(Ball *ball)
{
    double bx = ball->x;
    double by = ball->y;
    double r  = (double)BALL_RADIUS;

    for (int row = 0; row < BRICK_ROWS; row++) {
        for (int col = 0; col < BRICK_COLS; col++) {
            Brick *brick = &g_bricks[row][col];
            if (!brick->alive) {
                continue;
            }

            /* AABB bounds */
            double left   = (double)brick->x;
            double right  = (double)brick->x + brick->w;
            double top    = (double)brick->y;
            double bottom = (double)brick->y + brick->h;

            /* Check if ball overlaps brick rectangle (approximation) */
            if (bx + r > left && bx - r < right &&
                by + r > top  && by - r < bottom) {

                /* Find closest point on rectangle to ball center */
                double closest_x = fmax(left, fmin(bx, right));
                double closest_y = fmax(top,  fmin(by, bottom));
                double dx = bx - closest_x;
                double dy = by - closest_y;

                /* Collision face determination */
                if (fabs(dx) * fabs(dx) + fabs(dy) * fabs(dy) <= r * r) {
                    /* Determine collision face: the axis with larger distance */
                    if (fabs(dx) > fabs(dy)) {
                        ball->vx = -ball->vx;
                    } else {
                        ball->vy = -ball->vy;
                    }

                    /* Destroy brick */
                    brick->alive = 0;
                    g_state.score += 10;
                }
            }
        }
    }
}

/* ───────────────────────────────────────────
   Ball-paddle collision
   ─────────────────────────────────────────── */
static void CheckBallPaddleCollision(Ball *ball)
{
    double bx = ball->x;
    double by = ball->y;
    double r  = (double)BALL_RADIUS;
    int paddle_left   = g_paddle_x;
    int paddle_right  = g_paddle_x + PADDLE_WIDTH;
    int paddle_top    = PADDLE_Y;
    int paddle_bottom = PADDLE_Y + PADDLE_HEIGHT;

    if (bx + r > (double)paddle_left && bx - r < (double)paddle_right &&
        by + r > (double)paddle_top  && by - r < (double)paddle_bottom) {

        /* Calculate where the ball hit the paddle (0.0 ~ 1.0 ratio) */
        double ratio = (bx - (double)paddle_left) / (double)PADDLE_WIDTH;  /* 0~1 */
        /* Map to angle: -60 degrees to +60 degrees (in radians) */
        double angle = (ratio - 0.5) * (M_PI / 3.0);  /* -PI/6 ~ PI/6 */

        double speed = sqrt(ball->vx * ball->vx + ball->vy * ball->vy);
        ball->vx =  speed * sin(angle);
        ball->vy = -speed * cos(angle);

        /* Push ball above paddle to prevent tunneling */
        ball->y = (double)paddle_top - r - 1.0;
    }
}

/* ───────────────────────────────────────────
   Ball-wall collision
   ─────────────────────────────────────────── */
static void CheckBallWallCollision(Ball *ball)
{
    /* Left and right walls */
    if (ball->x - BALL_RADIUS < 0) {
        ball->x = (double)BALL_RADIUS;
        ball->vx = fabs(ball->vx);
    } else if (ball->x + BALL_RADIUS > WINDOW_WIDTH) {
        ball->x = (double)(WINDOW_WIDTH - BALL_RADIUS);
        ball->vx = -fabs(ball->vx);
    }

    /* Ceiling */
    if (ball->y - BALL_RADIUS < 0) {
        ball->y = (double)BALL_RADIUS;
        ball->vy = fabs(ball->vy);
    }

    /* Floor -> ball lost */
    if (ball->y + BALL_RADIUS > WINDOW_HEIGHT) {
        ball->vy = 0.0;
        ball->vx = 0.0;
    }
}

/* ───────────────────────────────────────────
   Ball movement update
   ─────────────────────────────────────────── */
static void UpdateBalls(void)
{
    for (int i = 0; i < g_ball_count; i++) {
        Ball *ball = &g_balls[i];

        /* Balls with zero velocity are waiting on paddle -> skip */
        if (ball->vx == 0.0 && ball->vy == 0.0) {
            continue;
        }

        ball->x += ball->vx;
        ball->y += ball->vy;

        CheckBallWallCollision(ball);
        CheckBallPaddleCollision(ball);
        CheckBallBrickCollision(ball);
    }
}

/* ───────────────────────────────────────────
   Ball loss handling
   ─────────────────────────────────────────── */
static void LoseABall(void)
{
    g_state.lives--;

    if (g_state.lives <= 0) {
        g_state.game_over = 1;
        return;
    }

    /* Place a new ball on the paddle */
    if (g_ball_count < MAX_BALLS) {
        g_balls[g_ball_count].x = (double)WINDOW_WIDTH / 2.0;
        g_balls[g_ball_count].y = (double)PADDLE_Y - BALL_RADIUS - 1.0;
        g_balls[g_ball_count].vx = 0.0;
        g_balls[g_ball_count].vy = 0.0;
        g_ball_count++;
    }
}

/* ───────────────────────────────────────────
   Input handling
   ─────────────────────────────────────────── */
static void HandleInput(void)
{
    /* Left/Right arrow keys or A/D */
    if (IsKeyDown(KEY_LEFT)  || IsKeyDown(KEY_A)) {
        g_paddle_x -= 7;
    }
    if (IsKeyDown(KEY_RIGHT) || IsKeyDown(KEY_D)) {
        g_paddle_x += 7;
    }

    /* Clamp paddle within screen bounds */
    if (g_paddle_x < 0) {
        g_paddle_x = 0;
    }
    if (g_paddle_x + PADDLE_WIDTH > WINDOW_WIDTH) {
        g_paddle_x = WINDOW_WIDTH - PADDLE_WIDTH;
    }

    /* Spacebar: launch ball if it is on paddle and not launched */
    if (IsKeyDown(KEY_SPACE)) {
        for (int i = 0; i < g_ball_count; i++) {
            Ball *ball = &g_balls[i];
            if (ball->vx == 0.0 && ball->vy == 0.0) {
                ball->vx = (double)BALL_SPEED;
                ball->vy = -(double)BALL_SPEED;
                break;
            }
        }
    }
}

/* ───────────────────────────────────────────
   Main rendering
   ─────────────────────────────────────────── */
static void Render(void)
{
    /* Background */
    ClearBackground(bg_color);

    /* ── Brick rendering ── */
    for (int r = 0; r < BRICK_ROWS; r++) {
        for (int c = 0; c < BRICK_COLS; c++) {
            Brick *b = &g_bricks[r][c];
            if (!b->alive) {
                continue;
            }

            Color brick_color = rainbow_colors[b->color];
            /* Draw brick with slight inner highlight for depth */
            DrawRectangle(b->x, b->y, b->w, b->h, brick_color);

            /* Border */
            DrawRectangleLines(b->x, b->y, b->w, b->h, WHITE);
        }
    }

    /* ── Paddle rendering ── */
    DrawRectangle(g_paddle_x, PADDLE_Y, PADDLE_WIDTH, PADDLE_HEIGHT, WHITE);
    DrawRectangleLines(g_paddle_x, PADDLE_Y, PADDLE_WIDTH, PADDLE_HEIGHT, LIGHTGRAY);

    /* ── Ball rendering ── */
    for (int i = 0; i < g_ball_count; i++) {
        Ball *ball = &g_balls[i];
        if (ball->vx == 0.0 && ball->vy == 0.0) {
            continue;
        }

        int cx = (int)ball->x;
        int cy = (int)ball->y;
        int r  = BALL_RADIUS;

        /* Ball shadow */
        DrawCircle(cx + 2, cy + 2, r, BLACK);

        /* Ball body */
        DrawCircle(cx, cy, r, WHITE);

        /* Ball highlight */
        DrawCircle(cx - 2, cy - 2, r / 2, LIGHTGRAY);
    }

    /* ── HUD (score, remaining balls, level) ── */
    DrawText(FormatText("Score: %d", g_state.score), 10, 10, 20, text_color);
    DrawText(FormatText("Balls: %d / %d", g_state.lives, MAX_BALLS), 10, 38, 20, text_color);
    DrawText(FormatText("Level: %d", g_state.level), WINDOW_WIDTH - 130, 10, 20, text_color);

    /* ── Game Over / Win message ── */
    if (g_state.game_over) {
        DrawRectangle(0, 0, WINDOW_WIDTH, WINDOW_HEIGHT, BLACK);

        DrawText("GAME OVER!", WINDOW_WIDTH / 2 - 100, WINDOW_HEIGHT / 2 - 50, 40, WHITE);
        DrawText(FormatText("Final Score: %d", g_state.score), WINDOW_WIDTH / 2 - 110, WINDOW_HEIGHT / 2, 24, WHITE);
        DrawText("Space: Restart | ESC: Quit", WINDOW_WIDTH / 2 - 150, WINDOW_HEIGHT / 2 + 50, 20, LIGHTGRAY);
    }

    if (g_state.win) {
        DrawRectangle(0, 0, WINDOW_WIDTH, WINDOW_HEIGHT, BLACK);

        DrawText("ALL BRICKS DESTROYED!", WINDOW_WIDTH / 2 - 170, WINDOW_HEIGHT / 2 - 40, 32, WHITE);
        DrawText(FormatText("Score: %d", g_state.score), WINDOW_WIDTH / 2 - 70, WINDOW_HEIGHT / 2 + 10, 24, WHITE);
        DrawText("Space: Next Level | ESC: Quit", WINDOW_WIDTH / 2 - 140, WINDOW_HEIGHT / 2 + 50, 20, LIGHTGRAY);
    }

    /* Prompt when all balls are waiting on paddle */
    int all_waiting = 1;
    for (int i = 0; i < g_ball_count; i++) {
        if (g_balls[i].vx != 0.0 || g_balls[i].vy != 0.0) {
            all_waiting = 0;
            break;
        }
    }
    if (all_waiting && !g_state.game_over && g_state.lives > 0) {
        DrawText("Press SPACE to launch ball", WINDOW_WIDTH / 2 - 160, WINDOW_HEIGHT / 2, 24, LIGHTGRAY);
    }

    EndDrawing();
}

/* ───────────────────────────────────────────
   Game logic update
   ─────────────────────────────────────────── */
static void Update(void)
{
    if (g_state.game_over || g_state.win) {
        return;
    }

    UpdateBalls();

    /* Check for balls that fell to the floor */
    for (int i = 0; i < g_ball_count; i++) {
        Ball *ball = &g_balls[i];
        if (ball->vx != 0.0 && ball->y + BALL_RADIUS >= (double)WINDOW_HEIGHT) {
            ball->vx = 0.0;
            ball->vy = 0.0;
            LoseABall();
            break;
        }
    }

    /* All bricks destroyed -> win */
    if (AllBricksDestroyed() && !g_state.win) {
        g_state.win = 1;
    }
}

/* ───────────────────────────────────────────
   Restart / Next level
   ─────────────────────────────────────────── */
static void RestartOrNextLevel(void)
{
    if (g_state.game_over) {
        /* Game over -> restart from beginning */
        g_ball_count = 0;
        g_state.lives = MAX_BALLS;
        g_state.score = 0;
        g_state.level = 1;
        g_state.game_over = 0;
        g_state.win = 0;
        InitGame();
    } else if (g_state.win) {
        /* Win -> next level */
        g_state.level++;
        g_state.win = 0;
        g_ball_count = 0;
        g_state.lives = MAX_BALLS;
        InitGame();
    }
}

/* ───────────────────────────────────────────
   Main
   ─────────────────────────────────────────── */
int main(void)
{
    /* Initialize raylib window */
    const int screen_width = WINDOW_WIDTH;
    const int screen_height = WINDOW_HEIGHT;
    const char *window_title = "Rainbow Block Breaker";

    InitWindow(screen_width, screen_height, window_title);
    SetTargetFPS(60);

    /* Initialize game */
    InitGame();

    /* Game loop */
    while (!WindowShouldClose()) {
        HandleInput();

        /* Game over / win state: space to restart / next level */
        if (g_state.game_over || g_state.win) {
            if (IsKeyDown(KEY_SPACE)) {
                /* Short delay to prevent repeated triggering */
                WaitTime(0.2);
                RestartOrNextLevel();
            }
            Update();
            Render();
            continue;
        }

        Update();
        Render();
    }

    /* Cleanup */
    CloseWindow();

    return 0;
}
