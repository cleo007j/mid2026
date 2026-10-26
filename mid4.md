<img width="1152" height="272" alt="image" src="https://github.com/user-attachments/assets/8cbd9be7-dcc4-4f2a-8a36-ffe18607f6bf" />

# 4번문제





```
#include <stdio.h>

int main(void) {
    int n;

    printf("Enter the height (N): ");
    if (scanf_s("%d", &n) != 1 || n <= 0) {
        printf("Invalid input.\n");
        return 1;
    }

    for (int i = 1; i <= n; i++) {
        // Print leading spaces for center alignment
        for (int j = 0; j < n - i; j++) {
            printf(" ");
        }

        // Determine character: odd rows use '*', even rows use '#'
        char ch = (i % 2 != 0) ? '*' : '#';

        // Print characters for the current row (2 * i - 1 characters)
        for (int j = 0; j < (2 * i - 1); j++) {
            printf("%c", ch);
        }

        printf("\n");
    }

    return 0;
}
