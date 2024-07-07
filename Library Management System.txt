#include <iostream>
#include <vector>
#include <algorithm>
#include <ctime>

struct Book {
    std::string title;
    std::string author;
    std::string ISBN;
    bool isAvailable;
    time_t dueDate;
};

struct Borrower {
    std::string name;
    std::vector<Book*> borrowedBooks;
};

// Function prototypes
void addBook(std::vector<Book>& books);
void searchBooks(const std::vector<Book>& books);
void checkoutBook(std::vector<Book>& books, std::vector<Borrower>& borrowers);
void returnBook(std::vector<Book>& books, std::vector<Borrower>& borrowers);
void calculateFine(const std::vector<Book>& books, const std::vector<Borrower>& borrowers);

int main() {
    std::vector<Book> books;
    std::vector<Borrower> borrowers;

    int choice;

    do {
        std::cout << "LIBRARY MANAGEMENT SYSTEM" << std::endl;
        std::cout << "1. Add Book" << std::endl;
        std::cout << "2. Search Books" << std::endl;
        std::cout << "3. Checkout Book" << std::endl;
        std::cout << "4. Return Book" << std::endl;
        std::cout << "5. Calculate Fine" << std::endl;
        std::cout << "6. Exit" << std::endl;
        std::cout << "Enter your choice: ";
        std::cin >> choice;

        switch (choice) {
            case 1:
                addBook(books);
                break;
            case 2:
                searchBooks(books);
                break;
            case 3:
                checkoutBook(books, borrowers);
                break;
            case 4:
                returnBook(books, borrowers);
                break;
            case 5:
                calculateFine(books, borrowers);
                break;
            case 6:
                std::cout << "Exiting the Library Management System. Goodbye!" << std::endl;
                break;
            default:
                std::cout << "Invalid choice. Please enter a valid option." << std::endl;
        }

    } while (choice != 6);

    return 0;
}

void addBook(std::vector<Book>& books) {
    Book newBook;
    std::cin.ignore(); // Clear the input buffer
    std::cout << "Enter the title of the book: ";
    std::getline(std::cin, newBook.title);
    std::cout << "Enter the author of the book: ";
    std::getline(std::cin, newBook.author);
    std::cout << "Enter the ISBN of the book: ";
    std::getline(std::cin, newBook.ISBN);
    newBook.isAvailable = true;
    newBook.dueDate = 0; // Due date is initially set to 0

    books.push_back(newBook);

    std::cout << "Book added successfully!" << std::endl;
}

void searchBooks(const std::vector<Book>& books) {
    std::string searchTerm;
    std::cin.ignore(); // Clear the input buffer
    std::cout << "Enter search term (title, author, or ISBN): ";
    std::getline(std::cin, searchTerm);

    std::cout << "SEARCH RESULTS" << std::endl;
    std::cout << "-----------------------------------------" << std::endl;

    for (const auto& book : books) {
        if (book.title.find(searchTerm) != std::string::npos ||
            book.author.find(searchTerm) != std::string::npos ||
            book.ISBN.find(searchTerm) != std::string::npos) {
            std::cout << "Title: " << book.title << std::endl;
            std::cout << "Author: " << book.author << std::endl;
            std::cout << "ISBN: " << book.ISBN << std::endl;
            std::cout << "Availability: " << (book.isAvailable ? "Available" : "Checked Out") << std::endl;
            if (!book.isAvailable) {
                std::cout << "Due Date: " << std::asctime(std::localtime(&book.dueDate));
            }
            std::cout << "-----------------------------------------" << std::endl;
        }
    }
}

void checkoutBook(std::vector<Book>& books, std::vector<Borrower>& borrowers) {
    std::string borrowerName;
    std::cin.ignore(); // Clear the input buffer
    std::cout << "Enter your name: ";
    std::getline(std::cin, borrowerName);

    auto borrowerIt = std::find_if(borrowers.begin(), borrowers.end(),
        [borrowerName](const Borrower& b) { return b.name == borrowerName; });

    if (borrowerIt == borrowers.end()) {
        Borrower newBorrower{borrowerName, {}};
        borrowers.push_back(newBorrower);
        borrowerIt = borrowers.end() - 1;
    }

    int bookIndex;
    std::cout << "Enter the index of the book to checkout: ";
    std::cin >> bookIndex;

    if (bookIndex >= 1 && bookIndex <= static_cast<int>(books.size())) {
        Book& selectedBook = books[bookIndex - 1];
        if (selectedBook.isAvailable) {
            selectedBook.isAvailable = false;
            time_t now = time(0);
            selectedBook.dueDate = now + 14 * 24 * 60 * 60; // Due in 14 days
            borrowerIt->borrowedBooks.push_back(&selectedBook);
            std::cout << "Book checked out successfully!" << std::endl;
        } else {
            std::cout << "Book is not available for checkout." << std::endl;
        }
    } else {
        std::cout << "Invalid book index. Please enter a valid index." << std::endl;
    }
}

void returnBook(std::vector<Book>& books, std::vector<Borrower>& borrowers) {
    std::string borrowerName;
    std::cin.ignore(); // Clear the input buffer
    std::cout << "Enter your name: ";
    std::getline(std::cin, borrowerName);

    auto borrowerIt = std::find_if(borrowers.begin(), borrowers.end(),
        [borrowerName](const Borrower& b) { return b.name == borrowerName; });

    if (borrowerIt != borrowers.end()) {
        int bookIndex;
        std::cout << "Enter the index of the book to return: ";
        std::cin >> bookIndex;

        if (bookIndex >= 1 && bookIndex <= static_cast<int>(borrowerIt->borrowedBooks.size())) {
            Book* returnedBook = borrowerIt->borrowedBooks[bookIndex - 1];
            returnedBook->isAvailable = true;
            borrowerIt->borrowedBooks.erase(borrowerIt->borrowedBooks.begin() + bookIndex - 1);
            std::cout << "Book returned successfully!" << std::endl;
        } else {
            std::cout << "Invalid book index. Please enter a valid index." << std::endl;
        }
    } else {
        std::cout << "Borrower not found. Please check your name." << std::endl;
    }
}

void calculateFine(const std::vector<Book>& books, const std::vector<Borrower>& borrowers) {
    time_t now = time(0);

    std::cout << "FINE CALCULATION" << std::endl;
    std::cout << "-----------------------------------------" << std::endl;

    for (const auto& borrower : borrowers) {
        for (const auto& borrowedBook : borrower.borrowedBooks) {
            if (borrowedBook->dueDate < now && !borrowedBook->isAvailable) {
                // Calculate fine (assume a fixed fine for simplicity)
                double fine = difftime(now, borrowedBook->dueDate) / (24 * 60 * 60) * 0.50;
                std::cout << "Borrower: " << borrower.name << std::endl;
                std::cout << "Book: " << borrowedBook->title << std::endl;
                std::cout << "Fine: $" << fine << std::endl;
                std::cout << "-----------------------------------------" << std::endl;
            }
        }
    }
}
