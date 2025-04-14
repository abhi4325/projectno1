#include <iostream>
#include <fstream>
#include <vector>
using namespace std;

class BankAccount {
public:
    int accountNumber;
    string name;
    string accountType;
    float balance;

    void createAccount() {
        cout << "\nEnter Account Number: ";
        cin >> accountNumber;
        cout << "Enter Name: ";
        cin.ignore();
        getline(cin, name);
        cout << "Enter Account Type (Saving/Current): ";
        cin >> accountType;
        cout << "Enter Initial Deposit: ";
        cin >> balance;
    }

    void display() const {
        cout << "\nAccount No.: " << accountNumber;
        cout << "\nName: " << name;
        cout << "\nType: " << accountType;
        cout << "\nBalance: Rs. " << balance << "\n";
    }

    void deposit(float amount) {
        balance += amount;
    }

    bool withdraw(float amount) {
        if (amount <= balance) {
            balance -= amount;
            return true;
        }
        return false;
    }
};

void saveToFile(const BankAccount& acc) {
    ofstream outFile("accounts.dat", ios::app | ios::binary);
    outFile.write((char*)&acc, sizeof(BankAccount));
    outFile.close();
}

void displayAllAccounts() {
    BankAccount acc;
    ifstream inFile("accounts.dat", ios::binary);
    cout << "\nAll Account Details:\n";
    while (inFile.read((char*)&acc, sizeof(BankAccount))) {
        acc.display();
        cout << "--------------------------" << endl;
    }
    inFile.close();
}

void searchAccount(int accNo) {
    BankAccount acc;
    bool found = false;
    ifstream inFile("accounts.dat", ios::binary);
    while (inFile.read((char*)&acc, sizeof(BankAccount))) {
        if (acc.accountNumber == accNo) {
            acc.display();
            found = true;
            break;
        }
    }
    inFile.close();
    if (!found)
        cout << "\nAccount not found!\n";
}

void updateAccount(int accNo, float amount, bool isDeposit) {
    BankAccount acc;
    fstream file("accounts.dat", ios::in | ios::out | ios::binary);
    bool found = false;

    while (file.read((char*)&acc, sizeof(BankAccount))) {
        if (acc.accountNumber == accNo) {
            long pos = file.tellg() - sizeof(BankAccount);
            if (isDeposit)
                acc.deposit(amount);
            else if (!acc.withdraw(amount)) {
                cout << "\nInsufficient balance!\n";
                return;
            }
            file.seekp(pos);
            file.write((char*)&acc, sizeof(BankAccount));
            cout << "\nTransaction successful.\n";
            found = true;
            break;
        }
    }
    file.close();
    if (!found)
        cout << "\nAccount not found!\n";
}

int main() {
    int choice, accNo;
    float amount;
    BankAccount acc;

    do {
        cout << "\n===== Bank Management System =====\n";
        cout << "1. Create New Account\n";
        cout << "2. Display All Accounts\n";
        cout << "3. Search Account by Number\n";
        cout << "4. Deposit Money\n";
        cout << "5. Withdraw Money\n";
        cout << "6. Exit\n";
        cout << "Enter your choice: ";
        cin >> choice;

        switch (choice) {
        case 1:
            acc.createAccount();
            saveToFile(acc);
            break;
        case 2:
            displayAllAccounts();
            break;
        case 3:
            cout << "\nEnter Account Number: ";
            cin >> accNo;
            searchAccount(accNo);
            break;
        case 4:
            cout << "\nEnter Account Number: ";
            cin >> accNo;
            cout << "Enter amount to deposit: ";
            cin >> amount;
            updateAccount(accNo, amount, true);
            break;
        case 5:
            cout << "\nEnter Account Number: ";
            cin >> accNo;
            cout << "Enter amount to withdraw: ";
            cin >> amount;
            updateAccount(accNo, amount, false);
            break;
        case 6:
            cout << "\nThank you for using our system.\n";
            break;
        default:
            cout << "\nInvalid choice!\n";
        }
    } while (choice != 6);

    return 0;
}


