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

struct BorrowedBook {
    char username[50];
    int book_id;
    char borrow_date[20];
};

// --- Utility Functions ---
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

// --- User Registration & Login ---
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

int login_user(char *username) {
    char pass[50];
    struct User u;
    int found = 0;
    getchar();
    printf("\n--- USER LOGIN ---\n");
    printf("Username: "); fgets(username, 50, stdin); trim_newline(username);
    printf("Password: ");
    get_password(pass);

    FILE *fp = fopen("users.txt", "r");
    if (!fp) {
        printf("No users registered!\n");
        return 0;
    }
    while (fscanf(fp, "%99[^,],%49[^,],%99[^,],%19[^,],%49[^\n]\n",
                  u.fullname, u.username, u.email, u.phone, u.password) == 5) {
        if (strcmp(u.username, username) == 0 && strcmp(u.password, pass) == 0) {
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

// --- Book Management ---
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
            printf("New Author: ");
            fgets(book.author, 100, stdin); trim_newline(book.author);
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

// --- Borrow Book ---
void borrow_book(const char *username) {
    int id, found = 0;
    struct BorrowedBook bb;
    printf("\nEnter Book ID to borrow: ");
    scanf("%d", &id);

    FILE *fp = fopen("books.txt", "r");
    while (fscanf(fp, "%d,%99[^,],%99[^\n]\n", &book.id, book.title, book.author) == 3) {
        if (book.id == id) {
            found = 1;
            break;
        }
    }
    fclose(fp);

    if (!found) {
        printf("Book ID not found.\n");
        return;
    }

    // Get the current date (placeholder)
    strcpy(bb.borrow_date, "2023-10-01");
    strcpy(bb.username, username);
    bb.book_id = id;

    FILE *bf = fopen("borrowed_books.txt", "a");
    if (!bf) {
        printf("Error saving borrowed book!\n");
        return;
    }
    fprintf(bf, "%s,%d,%s\n", bb.username, bb.book_id, bb.borrow_date);
    fclose(bf);
    printf("Book borrowed successfully!\n");
}

// --- View Borrow History ---
void view_borrow_history(const char *username) {
    FILE *fp = fopen("borrowed_books.txt", "r");
    if (!fp) {
        printf("No borrow history found.\n");
        return;
    }

    printf("\n--- BORROW HISTORY ---\n");
    printf("%-20s %-5s %-20s\n", "Username", "Book ID", "Borrow Date");
    struct BorrowedBook bb;
    int found = 0;
    while (fscanf(fp, "%49[^,],%d,%19[^\n]\n", bb.username, &bb.book_id, bb.borrow_date) == 3) {
        if (strcmp(bb.username, username) == 0) {
            printf("%-20s %-5d %-20s\n", bb.username, bb.book_id, bb.borrow_date);
            found = 1;
        }
    }
    if (!found) {
        printf("No borrow history found for user %s.\n", username);
    }
    fclose(fp);
}

// --- Admin Menu ---
void book_management_menu() {
    int choice;
    do {
        printf("\n--- BOOK MANAGEMENT ---\n");
        printf("1. View Books\n2. Add Book\n3. Delete Book\n4. Search Book\n5. Edit Book\n6. Back to Admin Panel\nEnter choice: ");
        scanf("%d", &choice);
        switch (choice) {
            case 1: view_books(); break;
            case 2: add_book(); break;
            case 3: delete_book(); break;
            case 4: search_book(); break;
            case 5: edit_book(); break;
            case 6: printf("Returning to Admin Panel...\n"); break;
            default: printf("Invalid choice!\n");
        }
    } while (choice != 6);
}

void view_all_users() {
    FILE *fp = fopen("users.txt", "r");
    if (!fp) {
        printf("No users registered!\n");
        return;
    }

    printf("\n--- USER DETAILS ---\n");
    printf("%-20s %-15s %-25s %-15s\n", "Full Name", "Username", "Email", "Phone");
    
    struct User u;
    while (fscanf(fp, "%99[^,],%49[^,],%99[^,],%19[^,],%49[^\n]\n",
                  u.fullname, u.username, u.email, u.phone, u.password) == 5) {
        printf("%-20s %-15s %-25s %-15s\n", u.fullname, u.username, u.email, u.phone);
    }
    fclose(fp);
}

void delete_user() {
    char username[50];
    printf("\nEnter username to delete: ");
    getchar();
    fgets(username, 50, stdin);
    trim_newline(username);

    FILE *fp = fopen("users.txt", "r");
    FILE *temp = fopen("temp_users.txt", "w");

    if (!fp || !temp) {
        printf("Error opening files.\n");
        if (fp) fclose(fp);
        if (temp) fclose(temp);
        return;
    }

    struct User u;
    int found = 0;
    while (fscanf(fp, "%99[^,],%49[^,],%99[^,],%19[^,],%49[^\n]\n",
                  u.fullname, u.username, u.email, u.phone, u.password) == 5) {
        if (strcmp(u.username, username) == 0) {
            found = 1;
            continue;
        }
        fprintf(temp, "%s,%s,%s,%s,%s\n", u.fullname, u.username, u.email, u.phone, u.password);
    }

    fclose(fp);
    fclose(temp);

    if (!found) {
        printf("Username not found.\n");
        remove("temp_users.txt");
    } else {
        remove("users.txt");
        rename("temp_users.txt", "users.txt");
        printf("User deleted successfully.\n");
    }
}

void admin_main_menu() {
    int choice;
    do {
        printf("\n=== ADMIN PANEL ===\n");
        printf("1. Book Management\n2. Borrow History\n3. User Details\n4. Delete User\n5. Logout\nEnter choice: ");
        scanf("%d", &choice);
        switch (choice) {
            case 1: book_management_menu(); break;
            case 2: {
                char username[50];
                printf("Enter username to check borrow history: ");
                getchar();
                fgets(username, 50, stdin);
                trim_newline(username);
                view_borrow_history(username);
                break;
            }
            case 3: view_all_users(); break;
            case 4: delete_user(); break;
            case 5: printf("Logging out...\n"); break;
            default: printf("Invalid choice!\n");
        }
    } while (choice != 5);
}

void admin_panel() {
    char uname[50], pass[50];
    getchar();
    printf("\n--- ADMIN LOGIN ---\n");
    printf("Admin Username: "); fgets(uname, 50, stdin); trim_newline(uname);
    printf("Admin Password: ");
    get_password(pass);

    if (strcmp(uname, ADMIN_USERNAME) == 0 && strcmp(pass, ADMIN_PASSWORD) == 0) {
        printf("Welcome Admin!\n");
        admin_main_menu();
    } else {
        printf("Invalid admin credentials!\n");
    }
}

// --- Main Function ---
int main() {
    int choice;
    char username[50];
    while (1) {
        printf("\n=== LIBRARY MANAGEMENT SYSTEM ===\n");
        printf("1. Register\n2. Login\n3. Admin Panel\n4. Exit\nEnter choice: ");
        scanf("%d", &choice);

        switch (choice) {
            case 1: register_user(); break;
            case 2:
                if (login_user(username)) {
                    printf("You are logged in as a user. Limited access.\n");
                    int user_choice;
                    do {
                        printf("\n1. View Books\n2. Search Book\n3. Borrow Book\n4. View Borrow History\n5. Logout\nEnter choice: ");
                        scanf("%d", &user_choice);
                        switch (user_choice) {
                            case 1: view_books(); break;
                            case 2: search_book(); break;
                            case 3: borrow_book(username); break;
                            case 4: view_borrow_history(username); break;
                            case 5: printf("Logging out...\n"); break;
                            default: printf("Invalid choice!\n");
                        }
                    } while (user_choice != 5);
                }
                break;
            case 3: admin_panel(); break;
            case 4: printf("Goodbye!\n"); exit(0);
            default: printf("Invalid choice!\n");
        }
    }
    return 0;
}
