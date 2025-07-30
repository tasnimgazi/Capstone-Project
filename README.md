# Capstone-Project
A library Management System by C langauage.


#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <ctype.h>
#include <conio.h>  

#define ADMIN_USERNAME "Admin01"
#define ADMIN_PASSWORD "242"

struct User {
    char fullname[100];
    char username[50];
    char email[100];
    char phone[20];
    char password[50];
};

struct Book {
    int id;
    char title[100];
    char author[100];
};

// --- Utility ---
void trim_newline(char *str) {
    size_t len = strlen(str);
    if (len > 0 && str[len-1] == '\n') str[len-1] = '\0';
}

int username_exists(const char *username) {
    FILE *fp = fopen("users.txt", "r");
    if (!fp) return 0;
    struct User u;
    while (fscanf(fp, "%99[^,],%49[^,],%99[^,],%19[^,],%49[^\n]\n",
                  u.fullname, u.username, u.email, u.phone, u.password) == 5) {
        if (strcmp(u.username, username) == 0) {
            fclose(fp);
            return 1;
        }
    }
    fclose(fp);
    return 0;
}

int is_valid_phone(const char *phone) {
    if (strlen(phone) != 11) return 0;
    for (int i = 0; i < 11; i++) {
        if (!isdigit(phone[i])) return 0;
    }
    return 1;
}

int is_valid_email(const char *email) {
    int has_at = 0;
    for (int i = 0; email[i]; i++) {
        if (!islower(email[i]) && email[i] != '@' && email[i] != '.' && !isdigit(email[i]))
            return 0;
        if (email[i] == '@') has_at = 1;
    }
    return has_at;
}

void get_password(char *password) {
    int idx = 0;
    char ch;
    while ((ch = getch()) != '\n' && ch != '\r') {
        if (ch == 127 || ch == 8) {
            if (idx > 0) {
                idx--;
                printf("\b \b");
            }
        } else if (idx < 49) {
            password[idx++] = ch;
            printf("*");
        }
    }
    password[idx] = '\0';
    printf("\n");
}

// --- User ---
void register_user() {
    struct User u;
    getchar();
    printf("\n--- USER REGISTRATION ---\n");
    printf("Full Name: "); fgets(u.fullname, 100, stdin); trim_newline(u.fullname);

    printf("Username: "); 
    fgets(u.username, 50, stdin); 
    trim_newline(u.username);
    if (username_exists(u.username)) {
        printf("Username already exists!\n");
        return;
    }

    while (1) {
        printf("Email (lowercase only, must include '@'): ");
        fgets(u.email, 100, stdin); 
        trim_newline(u.email);
        if (is_valid_email(u.email)) break;
        printf("Invalid email. Try again.\n");
    }

    while (1) {
        printf("Phone (11 digits only): ");
        fgets(u.phone, 20, stdin);
        trim_newline(u.phone);
        if (is_valid_phone(u.phone)) break;
        printf("Invalid phone number. Try again.\n");
    }

    printf("Password: ");
    get_password(u.password);

    FILE *fp = fopen("users.txt", "a");
    if (!fp) {
        printf("Error saving user!\n");
        return;
    }
    fprintf(fp, "%s,%s,%s,%s,%s\n", u.fullname, u.username, u.email, u.phone, u.password);
    fclose(fp);
    printf("Registration successful!\n");
}

int login_user() {
    char uname[50], pass[50];
    struct User u;
    int found = 0;
    getchar();
    printf("\n--- USER LOGIN ---\n");
    printf("Username: "); fgets(uname, 50, stdin); trim_newline(uname);
    printf("Password: ");
    get_password(pass);  // Hide password input with ***
    
    FILE *fp = fopen("users.txt", "r");
    if (!fp) {
        printf("No users registered!\n");
        return 0;
    }
    while (fscanf(fp, "%99[^,],%49[^,],%99[^,],%19[^,],%49[^\n]\n",
                  u.fullname, u.username, u.email, u.phone, u.password) == 5) {
        if (strcmp(u.username, uname) == 0 && strcmp(u.password, pass) == 0) {
            found = 1;
            break;
        }
    }
    fclose(fp);
    if (found) {
        printf("Login successful! Welcome, %s.\n", u.fullname);
        return 1;
    } else {
        printf("Login failed! Wrong username or password.\n");
        return 0;
    }
}

// --- Book ---
struct Book book;
int generate_book_id() {
    FILE *fp = fopen("books.txt", "r");
    if (!fp) return 1;
    int max_id = 0;
    while (fscanf(fp, "%d,%99[^,],%99[^\n]\n", &book.id, book.title, book.author) == 3) {
        if (book.id > max_id) max_id = book.id;
    }
    fclose(fp);
    return max_id + 1;
}

void add_book() {
    getchar();
    printf("\n--- ADD NEW BOOK ---\n");
    book.id = generate_book_id();
    printf("Book Title: "); fgets(book.title, 100, stdin); trim_newline(book.title);
    printf("Author: "); fgets(book.author, 100, stdin); trim_newline(book.author);

    FILE *fp = fopen("books.txt", "a");
    if (!fp) {
        printf("Error opening books file.\n");
        return;
    }
    fprintf(fp, "%d,%s,%s\n", book.id, book.title, book.author);
    fclose(fp);
    printf("Book added successfully with ID %d.\n", book.id);
}

void view_books() {
    FILE *fp = fopen("books.txt", "r");
    if (!fp) {
        printf("No books found.\n");
        return;
    }
    printf("\n--- BOOK LIST ---\n");
    printf("%-5s %-30s %-30s\n", "ID", "Title", "Author");
    while (fscanf(fp, "%d,%99[^,],%99[^\n]\n", &book.id, book.title, book.author) == 3) {
        printf("%-5d %-30s %-30s\n", book.id, book.title, book.author);
    }
    fclose(fp);
}

void search_book() {
    char keyword[100];
    int found = 0;
    getchar();
    printf("\nEnter keyword to search (title/author): ");
    fgets(keyword, 100, stdin); trim_newline(keyword);

    FILE *fp = fopen("books.txt", "r");
    if (!fp) {
        printf("No books found.\n");
        return;
    }

    printf("\n--- SEARCH RESULTS ---\n");
    while (fscanf(fp, "%d,%99[^,],%99[^\n]\n", &book.id, book.title, book.author) == 3) {
        if (strstr(book.title, keyword) || strstr(book.author, keyword)) {
            printf("ID: %d | Title: %s | Author: %s\n", book.id, book.title, book.author);
            found = 1;
        }
    }
    if (!found) printf("No matching books found.\n");
    fclose(fp);
}

void edit_book() {
    int id, found = 0;
    printf("\nEnter Book ID to edit: ");
    scanf("%d", &id);
    FILE *fp = fopen("books.txt", "r");
    FILE *temp = fopen("temp_books.txt", "w");

    if (!fp || !temp) {
        printf("Error opening files.\n");
        if (fp) fclose(fp);
        if (temp) fclose(temp);
        return;
    }

    while (fscanf(fp, "%d,%99[^,],%99[^\n]\n", &book.id, book.title, book.author) == 3) {
        if (book.id == id) {
            getchar();
            printf("New Title: "); fgets(book.title, 100, stdin); trim_newline(book.title);
            printf("New Author: "); fgets(book.author, 100, stdin); trim_newline(book.author);
            found = 1;
        }
        fprintf(temp, "%d,%s,%s\n", book.id, book.title, book.author);
    }

    fclose(fp);
    fclose(temp);

    if (!found) {
        printf("Book ID not found.\n");
        remove("temp_books.txt");
    } else {
        remove("books.txt");
        rename("temp_books.txt", "books.txt");
        printf("Book updated.\n");
    }
}

void delete_book() {
    int id, found = 0;
    printf("\nEnter Book ID to delete: ");
    scanf("%d", &id);
    FILE *fp = fopen("books.txt", "r");
    FILE *temp = fopen("temp_books.txt", "w");

    if (!fp || !temp) {
        printf("Error opening files.\n");
        if (fp) fclose(fp);
        if (temp) fclose(temp);
        return;
    }

    while (fscanf(fp, "%d,%99[^,],%99[^\n]\n", &book.id, book.title, book.author) == 3) {
        if (book.id == id) {
            found = 1;
            continue;
        }
        fprintf(temp, "%d,%s,%s\n", book.id, book.title, book.author);
    }

    fclose(fp);
    fclose(temp);

    if (!found) {
        printf("Book ID not found.\n");
        remove("temp_books.txt");
    } else {
        remove("books.txt");
        rename("temp_books.txt", "books.txt");
        printf("Book deleted.\n");
    }
}

// --- Admin Menu ---
void admin_main_menu() {
    int choice;
    do {
        printf("\n--- ADMIN PANEL ---\n");
        printf("1. Add Book\n2. View Books\n3. Search Book\n4. Edit Book\n5. Delete Book\n6. Logout\nEnter choice: ");
        scanf("%d", &choice);
        switch (choice) {
            case 1: add_book(); break;
            case 2: view_books(); break;
            case 3: search_book(); break;
            case 4: edit_book(); break;
            case 5: delete_book(); break;
            case 6: printf("Logging out...\n"); break;
            default: printf("Invalid choice!\n");
        }
    } while (choice != 6);
}

void admin_panel() {
    char uname[50], pass[50];
    getchar();
    printf("\n--- ADMIN LOGIN ---\n");
    printf("Admin Username: "); fgets(uname, 50, stdin); trim_newline(uname);
    printf("Admin Password: ");
    get_password(pass);  // Password hidden with ***
    
    if (strcmp(uname, ADMIN_USERNAME) == 0 && strcmp(pass, ADMIN_PASSWORD) == 0) {
        printf("Welcome Admin!\n");
        admin_main_menu();
    } else {
        printf("Invalid admin credentials!\n");
    }
}

int main() {
    int choice;
    while (1) {
        printf("\n=== LIBRARY MANAGEMENT SYSTEM ===\n");
        printf("1. Register\n2. Login\n3. Admin Panel\n4. Exit\nEnter choice: ");
        scanf("%d", &choice);

        switch (choice) {
            case 1: register_user(); break;
            case 2:
                if (login_user()) {
                    printf("You are logged in as a user. Limited access.\n");
                }
                break;
            case 3: admin_panel(); break;
            case 4: printf("Goodbye!\n"); exit(0);
            default: printf("Invalid choice!\n");
        }
    }
    return 0;
}



[by tasnim gazi]
