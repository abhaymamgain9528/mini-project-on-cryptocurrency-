# mini-project-on-cryptocurrency-
#include <iostream>
#include <vector>
#include <string>
#include <ctime>
#include <fstream>
using namespace std;

/* ---------------- TRANSACTION ---------------- */
struct Transaction
{
    string sender;
    string receiver;
    double amount;
};

/* ---------------- HASH FUNCTION ---------------- */
string generateHash(string input)
{
    int hash = 0;
    for (char c : input)
        hash = (hash + c) * 31;
    return to_string(hash);
}

/* ---------------- BLOCK ---------------- */
class Block
{
public:
    int index;
    string timestamp;
    vector<Transaction> transactions;
    string previousHash;
    string hash;

    Block(int idx, vector<Transaction> tx, string prevHash)
    {
        index = idx;
        transactions = tx;
        previousHash = prevHash;

        time_t now = time(0);
        timestamp = ctime(&now);

        hash = calculateHash();
    }

    string calculateHash()
    {
        string data = to_string(index) + timestamp + previousHash;

        for (auto &tx : transactions)
            data += tx.sender + tx.receiver + to_string(tx.amount);

        return generateHash(data);
    }
};

/* ---------------- BLOCKCHAIN ---------------- */
class Blockchain
{
public:
    vector<Block> chain;

    Blockchain()
    {
        chain.push_back(Block(0, {}, "0"));
    }

    Block getLatestBlock()
    {
        return chain.back();
    }

    void addBlock(vector<Transaction> tx)
    {
        Block newBlock(chain.size(), tx, getLatestBlock().hash);
        chain.push_back(newBlock);
    }

    double getBalance(string address)
    {
        double balance = 0;

        for (auto &block : chain)
        {
            for (auto &tx : block.transactions)
            {
                if (tx.sender == address)
                    balance -= tx.amount;
                if (tx.receiver == address)
                    balance += tx.amount;
            }
        }
        return balance;
    }

    void showHistory(string user)
    {
        cout << "\nTransaction History:\n";
        for (auto &block : chain)
        {
            for (auto &tx : block.transactions)
            {
                if (tx.sender == user || tx.receiver == user)
                {
                    cout << tx.sender << " -> "
                         << tx.receiver << " : "
                         << tx.amount << endl;
                }
            }
        }
    }

    /* -------- SAVE ONE TRANSACTION -------- */
    void saveTransaction(Transaction tx)
    {
        ofstream file("blockchain.txt", ios::app); // append mode

        file << tx.sender << " "
             << tx.receiver << " "
             << tx.amount << endl;
    }

    /* -------- LOAD ALL DATA -------- */
    void loadFromFile()
    {
        ifstream file("blockchain.txt");

        string s, r;
        double amt;

        while (file >> s >> r >> amt)
        {
            Transaction tx = {s, r, amt};
            addBlock({tx});
        }
    }
};

/* ---------------- WALLET ---------------- */
struct Wallet
{
    string name;
    string password;
};

/* -------- SAVE WALLET -------- */
void saveWallet(Wallet w)
{
    ofstream file("wallets.txt", ios::app); // append
    file << w.name << " " << w.password << endl;
}

/* -------- LOAD WALLETS -------- */
void loadWallets(vector<Wallet> &wallets)
{
    ifstream file("wallets.txt");

    Wallet w;
    while (file >> w.name >> w.password)
    {
        wallets.push_back(w);
    }
}

/* ---------------- MAIN ---------------- */
int main()
{
    Blockchain crypto;
    vector<Wallet> wallets;

    // LOAD OLD DATA
    loadWallets(wallets);
    crypto.loadFromFile();

    int choice;
    string name, pass;

    while (true)
    {
        cout << "\n===== CRYPTO SYSTEM =====\n";
        cout << "1. Create Wallet\n";
        cout << "2. Login\n";
        cout << "3. Add Balance\n";
        cout << "4. Send Coins\n";
        cout << "5. Check Balance\n";
        cout << "6. Transaction History\n";
        cout << "7. Exit\n";

        cout << "Enter choice: ";
        cin >> choice;

        switch (choice)
        {
        case 1:
        {
            Wallet w;
            cout << "Enter wallet name: ";
            cin >> w.name;
            cout << "Set password: ";
            cin >> w.password;

            wallets.push_back(w);
            saveWallet(w);

            cout << "Wallet created!\n";
            break;
        }

        case 2:
        {
            cout << "Enter wallet name: ";
            cin >> name;
            cout << "Enter password: ";
            cin >> pass;

            bool found = false;
            for (auto &w : wallets)
            {
                if (w.name == name && w.password == pass)
                {
                    cout << "Login successful!\n";
                    found = true;
                    break;
                }
            }

            if (!found)
                cout << "Invalid credentials!\n";

            break;
        }

        case 3:
        {
            cout << "Enter wallet name: ";
            cin >> name;

            double amount;
            cout << "Enter amount: ";
            cin >> amount;

            Transaction tx = {"System", name, amount};

            crypto.addBlock({tx});
            crypto.saveTransaction(tx);

            cout << "Balance added!\n";
            break;
        }

        case 4:
        {
            string sender, receiver;
            double amount;

            cout << "Sender: ";
            cin >> sender;
            cout << "Receiver: ";
            cin >> receiver;
            cout << "Amount: ";
            cin >> amount;

            if (crypto.getBalance(sender) < amount)
            {
                cout << "Insufficient balance!\n";
                break;
            }

            Transaction tx = {sender, receiver, amount};

            crypto.addBlock({tx});
            crypto.saveTransaction(tx);

            cout << "Transaction successful!\n";
            break;
        }

        case 5:
        {
            cout << "Enter wallet name: ";
            cin >> name;

            cout << "Balance: "
                 << crypto.getBalance(name)
                 << " coins\n";
            break;
        }

        case 6:
        {
            cout << "Enter wallet name: ";
            cin >> name;

            crypto.showHistory(name);
            break;
        }

        case 7:
            exit(0);

        default:
            cout << "Invalid choice\n";
        }
    }
}
