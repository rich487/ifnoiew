#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define MAX_TASKS 50

struct Task {
    char description[200];
    char dueDate[20];
    int completed;
};

struct Task tasks[MAX_TASKS];
int taskCount = 0;

void scheduleTask(char *description, char *dueDate) {
    if (taskCount < MAX_TASKS) {
        strncpy(tasks[taskCount].description, description, sizeof(tasks[taskCount].description) - 1);
        tasks[taskCount].description[sizeof(tasks[taskCount].description) - 1] = '\0'; // Ensure null-terminated
        strncpy(tasks[taskCount].dueDate, dueDate, sizeof(tasks[taskCount].dueDate) - 1);
        tasks[taskCount].dueDate[sizeof(tasks[taskCount].dueDate) - 1] = '\0'; // Ensure null-terminated
        tasks[taskCount].completed = 0; // 0 for not completed
        taskCount++;
        printf("Content-type: text/plain\n\n");
        printf("Task scheduled successfully!\n");
    } else {
        printf("Content-type: text/plain\n\n");
        printf("Maximum number of tasks reached. Cannot schedule more tasks.\n");
    }
}

int main() {
    char *method = getenv("REQUEST_METHOD");

    if (method != NULL && strcmp(method, "POST") == 0) {
        // Handle form submission
        char *lengthStr = getenv("CONTENT_LENGTH");
        int contentLength = lengthStr ? atoi(lengthStr) : 0;

        if (contentLength > 0) {
            char *postData = malloc(contentLength + 1);
            fread(postData, 1, contentLength, stdin);
            postData[contentLength] = '\0';

            // Parse the form data
            char *description = strstr(postData, "taskDescription=");
            char *dueDate = strstr(postData, "dueDate=");

            if (description && dueDate) {
                description += strlen("taskDescription=");
                dueDate += strlen("dueDate=");

                // Replace '+' with space in form data
                for (char *ptr = description; *ptr; ++ptr) {
                    if (*ptr == '+') *ptr = ' ';
                }

                for (char *ptr = dueDate; *ptr; ++ptr) {
                    if (*ptr == '+') *ptr = ' ';
                }

                scheduleTask(description, dueDate);
            }

            free(postData);
        }
    }

    return 0;
}
