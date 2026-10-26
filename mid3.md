<img width="554" height="234" alt="image" src="https://github.com/user-attachments/assets/32a37f82-b7db-4c8d-9c11-92047939156f" />

# 3번문제


```


#include <stdio.h>
#include <stdlib.h>
#include <time.h>

int main(void) {
    int answer[3];
    int user[3];
    int strikes = 0;
    int balls = 0;
    int attempts = 0;

    // Initialize random seed
    srand((unsigned int)time(NULL));

    // Generate 3 unique random digits (0 - 9)
    answer[0] = rand() % 10;
    do {
        answer[1] = rand() % 10;
    } while (answer[1] == answer[0]);

    do {
        answer[2] = rand() % 10;
    } while (answer[2] == answer[0] || answer[2] == answer[1]);

    printf("=== NUMBER BASEBALL GAME ===\n");
    printf("Guess 3 unique digits between 0 and 9.\n\n");

    while (1) {
        strikes = 0;
        balls = 0;

        printf("Attempt %d - Enter 3 digits (e.g., 1 2 3): ", attempts + 1);
        if (scanf_s("%d %d %d", &user[0], &user[1], &user[2]) != 3) {
            printf("Invalid input. Please enter numbers correctly.\n");
            while (getchar() != '\n'); // Clear input buffer
            continue;
        }

        attempts++;

        // Calculate strikes and balls
        for (int i = 0; i < 3; i++) {
            for (int j = 0; j < 3; j++) {
                if (user[i] == answer[j]) {
                    if (i == j) {
                        strikes++;
                    }
                    else {
                        balls++;
                    }
                }
            }
        }

        // Print result
        if (strikes == 3) {
            printf("Homerun! You win in %d attempts.\n", attempts);
            break;
        }
        else if (strikes == 0 && balls == 0) {
            printf("Out!\n\n");
        }
        else {
            printf("Result: %d Strike(s), %d Ball(s)\n\n", strikes, balls);
        }
    }

    return 0;
}
