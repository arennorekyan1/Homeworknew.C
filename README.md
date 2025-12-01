# Homeworknew.C[Homeworknew.c](https://github.com/user-attachments/files/23860672/Homeworknew.c)
//HW1

#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define NAME_LEN 50
#define SURNAME_LEN 50
#define FILENAME "group_details.csv"

typedef struct Student {
    char name[NAME_LEN];
    char surname[SURNAME_LEN];
    int id;
    float grade;
    struct Student *next;
} Student;

Student *head = NULL;

/* ---------- Helper functions ---------- */

void flush_input(void) {
    int c;
    while ((c = getchar()) != '\n' && c != EOF) { }
}

Student *create_student(const char *name, const char *surname, int id, float grade) {
    Student *s = (Student *)malloc(sizeof(Student));
    if (!s) {
        printf("Memory allocation failed!\n");
        exit(1);
    }
    strncpy(s->name, name, NAME_LEN - 1);
    s->name[NAME_LEN - 1] = '\0';

    strncpy(s->surname, surname, SURNAME_LEN - 1);
    s->surname[SURNAME_LEN - 1] = '\0';

    s->id = id;
    s->grade = grade;
    s->next = NULL;
    return s;
}

void add_student(void) {
    char name[NAME_LEN];
    char surname[SURNAME_LEN];
    int id;
    float grade;

    printf("Enter student name: ");
    scanf("%49s", name);

    printf("Enter student surname: ");
    scanf("%49s", surname);

    printf("Enter student ID: ");
    scanf("%d", &id);

    printf("Enter student grade: ");
    scanf("%f", &grade);

    Student *new_student = create_student(name, surname, id, grade);

    // Insert at the end of the list
    if (head == NULL) {
        head = new_student;
    } else {
        Student *curr = head;
        while (curr->next != NULL) {
            curr = curr->next;
        }
        curr->next = new_student;
    }

    printf("Student added successfully.\n");
}

void remove_student_by_id(void) {
    if (head == NULL) {
        printf("No students in the list.\n");
        return;
    }

    int id;
    printf("Enter ID of student to remove: ");
    scanf("%d", &id);

    Student *curr = head;
    Student *prev = NULL;

    while (curr != NULL && curr->id != id) {
        prev = curr;
        curr = curr->next;
    }

    if (curr == NULL) {
        printf("Student with ID %d not found.\n", id);
        return;
    }

    if (prev == NULL) { // removing head
        head = curr->next;
    } else {
        prev->next = curr->next;
    }

    free(curr);
    printf("Student removed successfully.\n");
}

Student *search_student_by_id_internal(int id) {
    Student *curr = head;
    while (curr != NULL) {
        if (curr->id == id)
            return curr;
        curr = curr->next;
    }
    return NULL;
}

void search_student_by_id(void) {
    if (head == NULL) {
        printf("No students in the list.\n");
        return;
    }

    int id;
    printf("Enter ID of student to search: ");
    scanf("%d", &id);

    Student *s = search_student_by_id_internal(id);
    if (s == NULL) {
        printf("Student with ID %d not found.\n", id);
    } else {
        printf("Student found:\n");
        printf("Name: %s\n", s->name);
        printf("Surname: %s\n", s->surname);
        printf("ID: %d\n", s->id);
        printf("Grade: %.2f\n", s->grade);
    }
}

void display_all_students(void) {
    if (head == NULL) {
        printf("No students in the list.\n");
        return;
    }

    Student *curr = head;
    printf("\n--- Student List ---\n");
    while (curr != NULL) {
        printf("Name: %-15s  Surname: %-15s  ID: %6d  Grade: %.2f\n",
               curr->name, curr->surname, curr->id, curr->grade);
        curr = curr->next;
    }
    printf("--------------------\n\n");
}

void display_average_grade(void) {
    if (head == NULL) {
        printf("No students in the list.\n");
        return;
    }

    int count = 0;
    float sum = 0.0f;
    Student *curr = head;

    while (curr != NULL) {
        sum += curr->grade;
        count++;
        curr = curr->next;
    }

    printf("Average grade of all students: %.2f\n", sum / count);
}

void save_to_file(void) {
    FILE *fp = fopen(FILENAME, "w");
    if (!fp) {
        printf("Could not open file \"%s\" for writing.\n", FILENAME);
        return;
    }

    Student *curr = head;
    while (curr != NULL) {
        // CSV: name,surname,id,grade
        fprintf(fp, "%s,%s,%d,%.2f\n",
                curr->name, curr->surname, curr->id, curr->grade);
        curr = curr->next;
    }

    fclose(fp);
    printf("Student details saved to file \"%s\".\n", FILENAME);
}

void read_from_file(void) {
    FILE *fp = fopen(FILENAME, "r");
    if (!fp) {
        printf("Could not open file \"%s\" for reading.\n", FILENAME);
        return;
    }

    // First free any existing list
    Student *curr = head;
    while (curr != NULL) {
        Student *tmp = curr;
        curr = curr->next;
        free(tmp);
    }
    head = NULL;

    char line[256];
    while (fgets(line, sizeof(line), fp)) {
        char name[NAME_LEN];
        char surname[SURNAME_LEN];
        int id;
        float grade;

        // read name,surname,id,grade
        if (sscanf(line, "%49[^,],%49[^,],%d,%f",
                   name, surname, &id, &grade) == 4) {

            Student *new_student = create_student(name, surname, id, grade);

            if (head == NULL) {
                head = new_student;
            } else {
                Student *tail = head;
                while (tail->next != NULL) {
                    tail = tail->next;
                }
                tail->next = new_student;
            }
        }
    }

    fclose(fp);
    printf("Student details loaded from file \"%s\".\n", FILENAME);
}

void free_all_students(void) {
    Student *curr = head;
    while (curr != NULL) {
        Student *tmp = curr;
        curr = curr->next;
        free(tmp);
    }
    head = NULL;
}

/* ---------- Main menu ---------- */

void print_menu(void) {
    printf("\n===== Student Management System =====\n");
    printf("1. Add a new student\n");
    printf("2. Remove a student by ID\n");
    printf("3. Search a student by ID\n");
    printf("4. Display all students\n");
    printf("5. Display average grade of all students\n");
    printf("6. Save student details to file (%s)\n", FILENAME);
    printf("7. Read student details from file (%s)\n", FILENAME);
    printf("0. Exit\n");
    printf("Choose an option: ");
}

int main(void) {
    int choice;

    do {
        print_menu();
        if (scanf("%d", &choice) != 1) {
            printf("Invalid input. Please enter a number.\n");
            flush_input();
            continue;
        }

        switch (choice) {
        case 1:
            add_student();
            break;
        case 2:
            remove_student_by_id();
            break;
        case 3:
            search_student_by_id();
            break;
        case 4:
            display_all_students();
            break;
        case 5:
            display_average_grade();
            break;
        case 6:
            save_to_file();
            break;
        case 7:
            read_from_file();
            break;
        case 0:
            printf("Exiting program...\n");
            break;
        default:
            printf("Invalid choice. Try again.\n");
        }
    } while (choice != 0);

    free_all_students();  // free dynamic memory before exit
    return 0;
}


//HW2

#include <stdio.h>
#include <stdlib.h>

void sort_array(int *arr, size_t n)
{
    size_t i, j;
    for (i = 0; i < n; ++i) {
        for (j = 0; j + 1 < n - i; ++j) {
            if (arr[j] > arr[j + 1]) {
                int tmp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = tmp;
            }
        }
    }
}

int main(void)
{
    size_t n;
    int *array = NULL;
    size_t i;

    printf("How many integers do you want to sort? ");
    if (scanf("%zu", &n) != 1 || n == 0)
        return 1;

    array = (int *)malloc(n * sizeof(int));
    if (array == NULL)
        return 1;

    printf("Enter %zu integers:\n", n);
    for (i = 0; i < n; ++i) {
        if (scanf("%d", &array[i]) != 1) {
            free(array);
            return 1;
        }
    }

    sort_array(array, n);

    printf("Sorted numbers:\n");
    for (i = 0; i < n; ++i)
        printf("%d ", array[i]);
    printf("\n");

    free(array);
    return 0;
}
