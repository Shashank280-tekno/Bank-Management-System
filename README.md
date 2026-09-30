#include <iostream>
#include <fstream>
#include <iomanip>
#include <limits>

using namespace std;

// Class representing a Bank Account
class Account {
private:
    int accountNumber;
    char name[50];
    double balance;

public:
    // Function to take user input for a new account
    void createAccount() {
        cout << "\nEnter Account Number: ";
        cin >> accountNumber;
        cin.ignore(numeric_limits<streamsize>::max(), '\n'); // Clear input buffer
        
        cout << "Enter Account Holder Name: ";
        cin.getline(name, 50);
        
        cout << "Enter Initial Balance ($): ";
        cin >> balance;
        
        cout << "\n[Success] Account successfully created!\n";
    }

    // Function to display account details
    void displayAccount() const {
        cout << "\n-----------------------------------";
        cout << "\nAccount Number : " << accountNumber;
        cout << "\nAccount Holder : " << name;
        cout << "\nCurrent Balance: $" << fixed << setprecision(2) << balance;
        cout << "\n-----------------------------------\n";
    }

    // Getter for account number
    int getAccountNumber() const {
        return accountNumber;
    }

    // Getter for balance
    double getBalance() const {
        return balance;
    }

    // Function to deposit money
    void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
            cout << "\n[Success] $" << fixed << setprecision(2) << amount << " deposited successfully.";
        } else {
            cout << "\n[Error] Invalid deposit amount.";
        }
    }

    // Function to withdraw money
    bool withdraw(double amount) {
        if (amount <= 0) {
            cout << "\n[Error] Invalid withdrawal amount.";
            return false;
        }
        if (amount > balance) {
            cout << "\n[Error] Insufficient balance!";
            return false;
        }
        balance -= amount;
        cout << "\n[Success] $" << fixed << setprecision(2) << amount << " withdrawn successfully.";
        return true;
    }
};

// --- File Handling Functions ---

// Function to write a new record to the file
void writeAccountToFile() {
    Account acc;
    acc.createAccount();

    ofstream outFile("bank_records.dat", ios::binary | ios::app);
    if (!outFile) {
        cout << "\n[Error] File could not be opened!";
        return;
    }
    
    // Write object data to binary file
    outFile.write(reinterpret_cast<char*>(&acc), sizeof(Account));
    outFile.close();
}

// Function to check balance / display account details by Account Number
void displayRecord(int n) {
    Account acc;
    ifstream inFile("bank_records.dat", ios::binary);
    bool found = false;

    if (!inFile) {
        cout << "\n[Error] No records found / File could not be opened!";
        return;
    }

    while (inFile.read(reinterpret_cast<char*>(&acc), sizeof(Account))) {
        if (acc.getAccountNumber() == n) {
            acc.displayAccount();
            found = true;
            break;
        }
    }
    inFile.close();

    if (!found) {
        cout << "\n[Error] Account number " << n << " does not exist.";
    }
}

// Function to deposit or withdraw funds in a specific account
void modifyAccountBalance(int n, int option) {
    Account acc;
    fstream file("bank_records.dat", ios::binary | ios::in | ios::out);
    bool found = false;

    if (!file) {
        cout << "\n[Error] File could not be opened!";
        return;
    }

    while (!file.eof() && file.read(reinterpret_cast<char*>(&acc), sizeof(Account))) {
        if (acc.getAccountNumber() == n) {
            found = true;
            double amount;

            if (option == 1) { // Deposit
                cout << "\nCurrent Balance: $" << fixed << setprecision(2) << acc.getBalance();
                cout << "\nEnter amount to deposit: $";
                cin >> amount;
                acc.deposit(amount);
            } 
            else if (option == 2) { // Withdrawal
                cout << "\nCurrent Balance: $" << fixed << setprecision(2) << acc.getBalance();
                cout << "\nEnter amount to withdraw: $";
                cin >> amount;
                acc.withdraw(amount);
            }

            // Move the file write pointer back by the size of one Account object to overwrite
            int pos = (-1) * static_cast<int>(sizeof(Account));
            file.seekp(pos, ios::cur);
            file.write(reinterpret_cast<char*>(&acc), sizeof(Account));
            
            cout << "\n[Success] Record updated successfully!";
            break;
        }
    }
    file.close();

    if (!found) {
        cout << "\n[Error] Account number " << n << " does not exist.";
    }
}

// --- Main Program Loop ---

int main() {
    int choice;
    int accNum;

    do {
        cout << "\n===================================";
        cout << "\n     BANK MANAGEMENT SYSTEM";
        cout << "\n===================================";
        cout << "\n1. Open New Account";
        cout << "\n2. Deposit Amount";
        cout << "\n3. Withdraw Amount";
        cout << "\n4. Balance Inquiry / Display Account";
        cout << "\n5. Exit";
        cout << "\n-----------------------------------";
        cout << "\nSelect an option (1-5): ";
        
        while (!(cin >> choice)) { // Input validation
            cout << "Invalid input. Please enter a number (1-5): ";
            cin.clear();
            cin.ignore(numeric_limits<streamsize>::max(), '\n');
        }

        switch (choice) {
            case 1:
                writeAccountToFile();
                break;
            case 2:
                cout << "\nEnter Account Number: ";
                cin >> accNum;
                modifyAccountBalance(accNum, 1);
                break;
            case 3:
                cout << "\nEnter Account Number: ";
                cin >> accNum;
                modifyAccountBalance(accNum, 2);
                break;
            case 4:
                cout << "\nEnter Account Number: ";
                cin >> accNum;
                displayRecord(accNum);
                break;
            case 5:
                cout << "\nThank you for using the Bank Management System!\n";
                break;
            default:
                cout << "\n[Error] Invalid choice! Please select between 1 and 5.";
        }
    } while (choice != 5);

    return 0;
}
