//User-Friendly-System-Call-Interface-for-Enhanced-Security
//Design an intuitive interface for system calls that enhances security by preventing unauthorized access. The interface should include authentication mechanisms and provide detailed logs of system call usage


#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>

// User Registration
void registerUser() {
    char newUsername[50], newPassword[50];
    printf("Register\nEnter username: ");
    scanf("%s", newUsername);
    printf("Enter password: ");
    scanf("%s", newPassword);

    FILE *userFile = fopen("users.db", "a");
    if (!userFile) {
        printf("Error opening user file.\n");
        return;
    }

    fprintf(userFile, "%s:%s\n", newUsername, newPassword);
    fclose(userFile);
    printf("✅ Registered successfully!\n");
}

// User Login
int loginUser() {
    char inputUsername[50], inputPassword[50], storedUsername[50], storedPassword[50];
    int loginSuccess = 0;

    printf("Login\nEnter username: ");
    scanf("%s", inputUsername);
    printf("Enter password: ");
    scanf("%s", inputPassword);

    FILE *userFile = fopen("users.db", "r");
    if (!userFile) {
        printf("Error: Could not open users.db\n");
        return 0;
    }

    while (fscanf(userFile, "%[^:]:%s\n", storedUsername, storedPassword) != EOF) {
        if (strcmp(inputUsername, storedUsername) == 0 && strcmp(inputPassword, storedPassword) == 0) {
            loginSuccess = 1;
            break;
        }
    }
    fclose(userFile);

    if (loginSuccess) {
        printf("✅ Login successful!\n");
        return 1;
    } else {
        printf("❌ Login failed.\n");
        return 0;
    }
}

// Logging function
void logEvent(const char *userNameLog, const char *userAction, const char *fileNameLog, const char *resultStatus) {
    FILE *logFile = fopen("logs.txt", "a");
    if (!logFile) return;

    time_t currentTime = time(NULL);
    struct tm *localTime = localtime(&currentTime);

    fprintf(logFile, "[%02d-%02d-%d %02d:%02d:%02d] User: %s | Action: %s | File: %s | Status: %s\n",
            localTime->tm_mday, localTime->tm_mon + 1, localTime->tm_year + 1900,
            localTime->tm_hour, localTime->tm_min, localTime->tm_sec,
            userNameLog, userAction, fileNameLog, resultStatus);
    fclose(logFile);
}

// AI Warning Check
void detectSuspiciousFile(const char *checkFileName) {
    if (strstr(checkFileName, "secret") != NULL) {
        printf("⚠️ WARNING: Suspicious filename detected! Admin will be notified.\n");
    }
}

// File Operations
void createFile(const char *user) {
    char newFileName[100];
    printf("Enter filename to create: ");
    scanf("%s", newFileName);

    detectSuspiciousFile(newFileName);

    FILE *newFile = fopen(newFileName, "w");
    if (newFile) {
        fclose(newFile);
        printf("✅ File '%s' created.\n", newFileName);
        logEvent(user, "CREATE", newFileName, "SUCCESS");
    } else {
        printf("❌ Could not create file.\n");
        logEvent(user, "CREATE", newFileName, "FAIL");
    }
}

void readFile(const char *user) {
    char readFileName[100], character;
    printf("Enter filename to read: ");
    scanf("%s", readFileName);

    detectSuspiciousFile(readFileName);

    FILE *readFile = fopen(readFileName, "r");
    if (readFile) {
        printf("📄 Contents of '%s':\n", readFileName);
        while ((character = fgetc(readFile)) != EOF) {
            putchar(character);
        }
        fclose(readFile);
        printf("\n");
        logEvent(user, "READ", readFileName, "SUCCESS");
    } else {
        printf("❌ Could not open file.\n");
        logEvent(user, "READ", readFileName, "FAIL");
    }
}

void writeFile(const char *user) {
    char writeFileName[100], fileContent[100];
    printf("Enter filename to write to: ");
    scanf("%s", writeFileName);

    detectSuspiciousFile(writeFileName);

    printf("Enter content (no spaces): ");
    scanf("%s", fileContent);

    FILE *writeFile = fopen(writeFileName, "a");
    if (writeFile) {
        fprintf(writeFile, "%s\n", fileContent);
        fclose(writeFile);
        printf("✍️ Content written to '%s'.\n", writeFileName);
        logEvent(user, "WRITE", writeFileName, "SUCCESS");
    } else {
        printf("❌ Could not write to file.\n");
        logEvent(user, "WRITE", writeFileName, "FAIL");
    }
}

// Menu after login
void syscallMenu(const char *loggedUser) {
    int menuChoice;
    while (1) {
        printf("\n--- System Call Interface ---\n");
        printf("1. Create File\n2. Read File\n3. Write File\n4. Exit\nEnter choice: ");
        scanf("%d", &menuChoice);

        if (menuChoice == 1) createFile(loggedUser);
        else if (menuChoice == 2) readFile(loggedUser);
        else if (menuChoice == 3) writeFile(loggedUser);
        else if (menuChoice == 4) break;
        else printf("Invalid choice.\n");
    }
}

// Main Function
int main() {
    int mainChoice;
    char activeUser[50];

    printf("=== Secure System Call Interface ===\n");

    while (1) {
        printf("\n1. Register\n2. Login\n3. Exit\nEnter choice: ");
        scanf("%d", &mainChoice);

        if (mainChoice == 1) registerUser();
        else if (mainChoice == 2) {
            if (loginUser()) {
                printf("Enter your username again (for logging): ");
                scanf("%s", activeUser);
                syscallMenu(activeUser);
            }
        }
        else if (mainChoice == 3) {
            printf("Goodbye!\n");
            break;
        }
        else printf("Invalid choice.\n");
    }

    return 0;
}
