#2번 문
```
#include <stdio.h>
#include <stdlib.h>
#include "sqlite3.h"

int main(void) {
    sqlite3 *db;
    char *err_msg = 0;
    int rc;

    // Open database
    rc = sqlite3_open("school.db", &db);
    if (rc != SQLITE_OK) {
        fprintf(stderr, "Cannot open database: %s\n", sqlite3_errmsg(db));
        sqlite3_close(db);
        return 1;
    }

    // Create table for student scenario
    const char *sql_create = "DROP TABLE IF EXISTS STUDENTS;"
                             "CREATE TABLE STUDENTS(ID INT, NAME TEXT, MAJOR TEXT, SCORE INT);";
    
    rc = sqlite3_exec(db, sql_create, 0, 0, &err_msg);
    if (rc != SQLITE_OK) {
        fprintf(stderr, "SQL error: %s\n", err_msg);
        sqlite3_free(err_msg);
        sqlite3_close(db);
        return 1;
    }

    // Insert data for 3 or more people
    const char *sql_insert = "INSERT INTO STUDENTS VALUES(1, 'Alice', 'Computer Science', 95);"
                             "INSERT INTO STUDENTS VALUES(2, 'Bob', 'Mathematics', 88);"
                             "INSERT INTO STUDENTS VALUES(3, 'Charlie', 'Physics', 92);"
                             "INSERT INTO STUDENTS VALUES(4, 'Diana', 'Chemistry', 90);";

    rc = sqlite3_exec(db, sql_insert, 0, 0, &err_msg);
    if (rc != SQLITE_OK) {
        fprintf(stderr, "SQL error: %s\n", err_msg);
        sqlite3_free(err_msg);
        sqlite3_close(db);
        return 1;
    }

    printf("=== All Students in Database ===\n");
    const char *sql_select_all = "SELECT * FROM STUDENTS;";
    sqlite3_stmt *stmt_all;
    
    rc = sqlite3_prepare_v2(db, sql_select_all, -1, &stmt_all, 0);
    if (rc == SQLITE_OK) {
        while (sqlite3_step(stmt_all) == SQLITE_ROW) {
            printf("ID: %d | Name: %s | Major: %s | Score: %d\n",
                   sqlite3_column_int(stmt_all, 0),
                   sqlite3_column_text(stmt_all, 1),
                   sqlite3_column_text(stmt_all, 2),
                   sqlite3_column_int(stmt_all, 3));
        }
    }
    sqlite3_finalize(stmt_all);

    // Search feature implementation
    printf("\n=== Search Result (Name: Bob) ===\n");
    const char *search_name = "Bob";
    char sql_search[256];
    snprintf(sql_search, sizeof(sql_search), "SELECT * FROM STUDENTS WHERE NAME = '%s';", search_name);

    sqlite3_stmt *stmt_search;
    rc = sqlite3_prepare_v2(db, sql_search, -1, &stmt_search, 0);
    if (rc == SQLITE_OK) {
        while (sqlite3_step(stmt_search) == SQLITE_ROW) {
            printf("Found -> ID: %d | Name: %s | Major: %s | Score: %d\n",
                   sqlite3_column_int(stmt_search, 0),
                   sqlite3_column_text(stmt_search, 1),
                   sqlite3_column_text(stmt_search, 2),
                   sqlite3_column_int(stmt_search, 3));
        }
    } else {
        fprintf(stderr, "Failed to execute search: %s\n", sqlite3_errmsg(db));
    }
    sqlite3_finalize(stmt_search);

    // Close database
    sqlite3_close(db);
    return 0;
}
