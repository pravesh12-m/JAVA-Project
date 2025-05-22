# JAVA-Project
SBI BANKING SYSTEM USING JAVA 
com.sbi.banking/
├── models/
│   ├── Customer.java
│   ├── Account.java
│   ├── Transaction.java
│   └── Admin.java
├── dao/
│   ├── CustomerDAO.java
│   ├── AccountDAO.java
│   ├── TransactionDAO.java
│   └── DatabaseConnection.java
├── gui/
│   ├── LoginFrame.java
│   ├── MainDashboard.java
│   ├── CreateAccountFrame.java
│   ├── TransactionFrame.java
│   └── BalanceInquiryFrame.java
├── utils/
│   ├── ValidationUtils.java
│   ├── PasswordEncoder.java
│   └── Constants.java
└── main/
    └── SBIBankingApp.java


---

## 3. Design Database Schema 

### Database Schema Design:
sql
-- Database: sbi_banking_db

-- Customers Table
CREATE TABLE customers (
    customer_id INT PRIMARY KEY AUTO_INCREMENT,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    phone VARCHAR(15) NOT NULL,
    address TEXT,
    date_of_birth DATE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Accounts Table
CREATE TABLE accounts (
    account_number BIGINT PRIMARY KEY,
    customer_id INT,
    account_type ENUM('SAVINGS', 'CURRENT', 'FIXED_DEPOSIT') DEFAULT 'SAVINGS',
    balance DECIMAL(15,2) DEFAULT 0.00,
    pin VARCHAR(4) NOT NULL,
    status ENUM('ACTIVE', 'INACTIVE', 'BLOCKED') DEFAULT 'ACTIVE',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

-- Transactions Table
CREATE TABLE transactions (
    transaction_id INT PRIMARY KEY AUTO_INCREMENT,
    from_account BIGINT,
    to_account BIGINT,
    transaction_type ENUM('DEPOSIT', 'WITHDRAWAL', 'TRANSFER', 'BALANCE_INQUIRY'),
    amount DECIMAL(15,2),
    description TEXT,
    transaction_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status ENUM('SUCCESS', 'FAILED', 'PENDING') DEFAULT 'SUCCESS',
    FOREIGN KEY (from_account) REFERENCES accounts(account_number),
    FOREIGN KEY (to_account) REFERENCES accounts(account_number)
);

-- Admin Table
CREATE TABLE admin (
    admin_id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    full_name VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);


---

## 4. Create MySQL Tables 

### Table Creation Script:
sql
-- Execute these commands in MySQL Workbench

USE sbi_banking_db;

-- Create all tables as per schema above

-- Insert sample admin user
INSERT INTO admin (username, password, full_name) 
VALUES ('admin', 'admin123', 'SBI Administrator');

-- Insert sample customer data
INSERT INTO customers (first_name, last_name, email, phone, address, date_of_birth)
VALUES 
('Rajesh', 'Kumar', 'rajesh.kumar@email.com', '9876543210', 'Delhi, India', '1990-05-15'),
('Priya', 'Sharma', 'priya.sharma@email.com', '9876543211', 'Mumbai, India', '1985-08-20');

-- Insert sample accounts
INSERT INTO accounts (account_number, customer_id, account_type, balance, pin)
VALUES 
(1234567890, 1, 'SAVINGS', 50000.00, '1234'),
(1234567891, 2, 'CURRENT', 75000.00, '5678');


---

## 5. Implement JDBC for Database Connectivity 

### DatabaseConnection.java:
java
package com.sbi.banking.dao;

import java.sql.*;
import java.util.Properties;

public class DatabaseConnection {
    private static final String URL = "jdbc:mysql://localhost:3306/sbi_banking_db";
    private static final String USERNAME = "root";
    private static final String PASSWORD = "your_mysql_password";
    
    private static Connection connection = null;
    
    public static Connection getConnection() throws SQLException {
        try {
            if (connection == null || connection.isClosed()) {
                Properties props = new Properties();
                props.setProperty("user", USERNAME);
                props.setProperty("password", PASSWORD);
                props.setProperty("useSSL", "false");
                props.setProperty("serverTimezone", "UTC");
                
                Class.forName("com.mysql.cj.jdbc.Driver");
                connection = DriverManager.getConnection(URL, props);
                System.out.println("Database connected successfully!");
            }
        } catch (ClassNotFoundException e) {
            System.err.println("MySQL Driver not found: " + e.getMessage());
            throw new SQLException("Database driver not found", e);
        } catch (SQLException e) {
            System.err.println("Database connection failed: " + e.getMessage());
            throw e;
        }
        return connection;
    }
    
    public static void closeConnection() {
        try {
            if (connection != null && !connection.isClosed()) {
                connection.close();
                System.out.println("Database connection closed.");
            }
        } catch (SQLException e) {
            System.err.println("Error closing connection: " + e.getMessage());
        }
    }
    
    // Test connection method
    public static boolean testConnection() {
        try (Connection conn = getConnection()) {
            return conn != null && !conn.isClosed();
        } catch (SQLException e) {
            System.err.println("Connection test failed: " + e.getMessage());
            return false;
        }
    }
}


---

## 6. Create Model & DAO Classes 

### Customer.java (Model):
java
package com.sbi.banking.models;

import java.time.LocalDate;
import java.time.LocalDateTime;

public class Customer {
    private int customerId;
    private String firstName;
    private String lastName;
    private String email;
    private String phone;
    private String address;
    private LocalDate dateOfBirth;
    private LocalDateTime createdAt;
    
    // Constructors
    public Customer() {}
    
    public Customer(String firstName, String lastName, String email, 
                   String phone, String address, LocalDate dateOfBirth) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.email = email;
        this.phone = phone;
        this.address = address;
        this.dateOfBirth = dateOfBirth;
    }
    
    // Getters and Setters
    public int getCustomerId() { return customerId; }
    public void setCustomerId(int customerId) { this.customerId = customerId; }
    
    public String getFirstName() { return firstName; }
    public void setFirstName(String firstName) { this.firstName = firstName; }
    
    public String getLastName() { return lastName; }
    public void setLastName(String lastName) { this.lastName = lastName; }
    
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
    
    public String getPhone() { return phone; }
    public void setPhone(String phone) { this.phone = phone; }
    
    public String getAddress() { return address; }
    public void setAddress(String address) { this.address = address; }
    
    public LocalDate getDateOfBirth() { return dateOfBirth; }
    public void setDateOfBirth(LocalDate dateOfBirth) { this.dateOfBirth = dateOfBirth; }
    
    public LocalDateTime getCreatedAt() { return createdAt; }
    public void setCreatedAt(LocalDateTime createdAt) { this.createdAt = createdAt; }
    
    public String getFullName() {
        return firstName + " " + lastName;
    }
    
    @Override
    public String toString() {
        return "Customer{" +
                "customerId=" + customerId +
                ", name='" + getFullName() + '\'' +
                ", email='" + email + '\'' +
                ", phone='" + phone + '\'' +
                '}';
    }
}


### Account.java (Model):
java
package com.sbi.banking.models;

import java.math.BigDecimal;
import java.time.LocalDateTime;

public class Account {
    private long accountNumber;
    private int customerId;
    private String accountType;
    private BigDecimal balance;
    private String pin;
    private String status;
    private LocalDateTime createdAt;
    
    // Constructors
    public Account() {}
    
    public Account(long accountNumber, int customerId, String accountType, 
                  BigDecimal balance, String pin) {
        this.accountNumber = accountNumber;
        this.customerId = customerId;
        this.accountType = accountType;
        this.balance = balance;
        this.pin = pin;
        this.status = "ACTIVE";
    }
    
    // Getters and Setters
    public long getAccountNumber() { return accountNumber; }
    public void setAccountNumber(long accountNumber) { this.accountNumber = accountNumber; }
    
    public int getCustomerId() { return customerId; }
    public void setCustomerId(int customerId) { this.customerId = customerId; }
    
    public String getAccountType() { return accountType; }
    public void setAccountType(String accountType) { this.accountType = accountType; }
    
    public BigDecimal getBalance() { return balance; }
    public void setBalance(BigDecimal balance) { this.balance = balance; }
    
    public String getPin() { return pin; }
    public void setPin(String pin) { this.pin = pin; }
    
    public String getStatus() { return status; }
    public void setStatus(String status) { this.status = status; }
    
    public LocalDateTime getCreatedAt() { return createdAt; }
    public void setCreatedAt(LocalDateTime createdAt) { this.createdAt = createdAt; }
    
    @Override
    public String toString() {
        return "Account{" +
                "accountNumber=" + accountNumber +
                ", customerId=" + customerId +
                ", accountType='" + accountType + '\'' +
                ", balance=" + balance +
                ", status='" + status + '\'' +
                '}';
    }
}


### AccountDAO.java:
java
package com.sbi.banking.dao;

import com.sbi.banking.models.Account;
import java.math.BigDecimal;
import java.sql.*;
import java.util.ArrayList;
import java.util.List;

public class AccountDAO {
    
    public boolean validateAccount(long accountNumber, String pin) {
        String sql = "SELECT COUNT(*) FROM accounts WHERE account_number = ? AND pin = ? AND status = 'ACTIVE'";
        
        try (Connection conn = DatabaseConnection.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            pstmt.setLong(1, accountNumber);
            pstmt.setString(2, pin);
            
            ResultSet rs = pstmt.executeQuery();
            if (rs.next()) {
                return rs.getInt(1) > 0;
            }
        } catch (SQLException e) {
            System.err.println("Error validating account: " + e.getMessage());
        }
        return false;
    }
    
    public Account getAccountByNumber(long accountNumber) {
        String sql = "SELECT * FROM accounts WHERE account_number = ?";
        
        try (Connection conn = DatabaseConnection.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            pstmt.setLong(1, accountNumber);
            ResultSet rs = pstmt.executeQuery();
            
            if (rs.next()) {
                Account account = new Account();
                account.setAccountNumber(rs.getLong("account_number"));
                account.setCustomerId(rs.getInt("customer_id"));
                account.setAccountType(rs.getString("account_type"));
                account.setBalance(rs.getBigDecimal("balance"));
                account.setPin(rs.getString("pin"));
                account.setStatus(rs.getString("status"));
                return account;
            }
        } catch (SQLException e) {
            System.err.println("Error retrieving account: " + e.getMessage());
        }
        return null;
    }
    
    public boolean updateBalance(long accountNumber, BigDecimal newBalance) {
        String sql = "UPDATE accounts SET balance = ? WHERE account_number = ?";
        
        try (Connection conn = DatabaseConnection.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            pstmt.setBigDecimal(1, newBalance);
            pstmt.setLong(2, accountNumber);
            
            return pstmt.executeUpdate() > 0;
        } catch (SQLException e) {
            System.err.println("Error updating balance: " + e.getMessage());
            return false;
        }
    }
    
    public boolean createAccount(Account account) {
        String sql = "INSERT INTO accounts (account_number, customer_id, account_type, balance, pin) VALUES (?, ?, ?, ?, ?)";
        
        try (Connection conn = DatabaseConnection.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            pstmt.setLong(1, account.getAccountNumber());
            pstmt.setInt(2, account.getCustomerId());
            pstmt.setString(3, account.getAccountType());
            pstmt.setBigDecimal(4, account.getBalance());
            pstmt.setString(5, account.getPin());
            
            return pstmt.executeUpdate() > 0;
        } catch (SQLException e) {
            System.err.println("Error creating account: " + e.getMessage());
            return false;
        }
    }
    
    public List<Account> getAccountsByCustomerId(int customerId) {
        List<Account> accounts = new ArrayList<>();
        String sql = "SELECT * FROM accounts WHERE customer_id = ?";
        
        try (Connection conn = DatabaseConnection.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            pstmt.setInt(1, customerId);
            ResultSet rs = pstmt.executeQuery();
            
            while (rs.next()) {
                Account account = new Account();
                account.setAccountNumber(rs.getLong("account_number"));
                account.setCustomerId(rs.getInt("customer_id"));
                account.setAccountType(rs.getString("account_type"));
                account.setBalance(rs.getBigDecimal("balance"));
                account.setPin(rs.getString("pin"));
                account.setStatus(rs.getString("status"));
                accounts.add(account);
            }
        } catch (SQLException e) {
            System.err.println("Error retrieving accounts: " + e.getMessage());
        }
        return accounts;
    }
}


---

## 7. Aesthetics and Visual Appeal of UI 

### LoginFrame.java:
java
package com.sbi.banking.gui;

import com.sbi.banking.dao.AccountDAO;
import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;

public class LoginFrame extends JFrame {
    private JTextField accountNumberField;
    private JPasswordField pinField;
    private JButton loginButton;
    private JButton createAccountButton;
    private AccountDAO accountDAO;
    
    public LoginFrame() {
        accountDAO = new AccountDAO();
        initializeComponents();
        setupLayout();
        setupEventHandlers();
    }
    
    private void initializeComponents() {
        setTitle("SBI Banking System - Login");
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setSize(500, 400);
        setLocationRelativeTo(null);
        setResizable(false);
        
        // Set Bank Colors - SBI Blue Theme
        getContentPane().setBackground(new Color(0, 51, 102));
        
        // Create components with enhanced styling
        accountNumberField = new JTextField(15);
        pinField = new JPasswordField(15);
        loginButton = new JButton("LOGIN");
        createAccountButton = new JButton("CREATE NEW ACCOUNT");
        
        // Style text fields
        styleTextField(accountNumberField);
        styleTextField(pinField);
        
        // Style buttons
        styleButton(loginButton, new Color(46, 125, 50));
        styleButton(createAccountButton, new Color(25, 118, 210));
    }
    
    private void styleTextField(JTextField field) {
        field.setFont(new Font("Arial", Font.PLAIN, 14));
        field.setBorder(BorderFactory.createCompoundBorder(
            BorderFactory.createLineBorder(new Color(200, 200, 200), 1),
            BorderFactory.createEmptyBorder(8, 8, 8, 8)
        ));
        field.setBackground(Color.WHITE);
    }
    
    private void styleButton(JButton button, Color backgroundColor) {
        button.setFont(new Font("Arial", Font.BOLD, 12));
        button.setBackground(backgroundColor);
        button.setForeground(Color.WHITE);
        button.setBorder(BorderFactory.createEmptyBorder(10, 20, 10, 20));
        button.setFocusPainted(false);
        button.setCursor(new Cursor(Cursor.HAND_CURSOR));
    }
    
    private void setupLayout() {
        setLayout(new BorderLayout());
        
        // Header Panel with SBI Logo and Title
        JPanel headerPanel = new JPanel();
        headerPanel.setBackground(new Color(0, 51, 102));
        headerPanel.setPreferredSize(new Dimension(500, 80));
        
        JLabel titleLabel = new JLabel("STATE BANK OF INDIA");
        titleLabel.setFont(new Font("Arial", Font.BOLD, 24));
        titleLabel.setForeground(Color.WHITE);
        titleLabel.setHorizontalAlignment(SwingConstants.CENTER);
        
        JLabel subtitleLabel = new JLabel("Banking Management System");
        subtitleLabel.setFont(new Font("Arial", Font.PLAIN, 14));
        subtitleLabel.setForeground(new Color(200, 200, 200));
        subtitleLabel.setHorizontalAlignment(SwingConstants.CENTER);
        
        headerPanel.setLayout(new GridLayout(2, 1));
        headerPanel.add(titleLabel);
        headerPanel.add(subtitleLabel);
        
        // Main Panel
        JPanel mainPanel = new JPanel();
        mainPanel.setBackground(Color.WHITE);
        mainPanel.setLayout(new GridBagLayout());
        GridBagConstraints gbc = new GridBagConstraints();
        
        // Login Form
        JLabel loginLabel = new JLabel("Account Login");
        loginLabel.setFont(new Font("Arial", Font.BOLD, 18));
        loginLabel.setForeground(new Color(0, 51, 102));
        
        gbc.gridx = 0; gbc.gridy = 0;
        gbc.gridwidth = 2;
        gbc.insets = new Insets(20, 0, 30, 0);
        mainPanel.add(loginLabel, gbc);
        
        // Account Number
        JLabel accountLabel = new JLabel("Account Number:");
        accountLabel.setFont(new Font("Arial", Font.PLAIN, 12));
        gbc.gridx = 0; gbc.gridy = 1;
        gbc.gridwidth = 1;
        gbc.insets = new Insets(5, 0, 5, 10);
        gbc.anchor = GridBagConstraints.EAST;
        mainPanel.add(accountLabel, gbc);
        
        gbc.gridx = 1; gbc.gridy = 1;
        gbc.anchor = GridBagConstraints.WEST;
        mainPanel.add(accountNumberField, gbc);
        
        // PIN
        JLabel pinLabel = new JLabel("PIN:");
        pinLabel.setFont(new Font("Arial", Font.PLAIN, 12));
        gbc.gridx = 0; gbc.gridy = 2;
        gbc.anchor = GridBagConstraints.EAST;
        mainPanel.add(pinLabel, gbc);
        
        gbc.gridx = 1; gbc.gridy = 2;
        gbc.anchor = GridBagConstraints.WEST;
        mainPanel.add(pinField, gbc);
        
        // Buttons
        gbc.gridx = 0; gbc.gridy = 3;
        gbc.gridwidth = 2;
        gbc.insets = new Insets(20, 0, 10, 0);
        gbc.anchor = GridBagConstraints.CENTER;
        mainPanel.add(loginButton, gbc);
        
        gbc.gridy = 4;
        gbc.insets = new Insets(5, 0, 20, 0);
        mainPanel.add(createAccountButton, gbc);
        
        add(headerPanel, BorderLayout.NORTH);
        add(mainPanel, BorderLayout.CENTER);
        
        // Footer
        JPanel footerPanel = new JPanel();
        footerPanel.setBackground(new Color(240, 240, 240));
        footerPanel.setPreferredSize(new Dimension(500, 30));
        JLabel footerLabel = new JLabel("© 2024 State Bank of India. All rights reserved.");
        footerLabel.setFont(new Font("Arial", Font.PLAIN, 10));
        footerLabel.setHorizontalAlignment(SwingConstants.CENTER);
        footerPanel.add(footerLabel);
        add(footerPanel, BorderLayout.SOUTH);
    }
    
    private void setupEventHandlers() {
        loginButton.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                performLogin();
            }
        });
        
        createAccountButton.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                openCreateAccountFrame();
            }
        });
        
        // Enter key support
        getRootPane().setDefaultButton(loginButton);
    }
    
    private void performLogin() {
        String accountNumberText = accountNumberField.getText().trim();
        String pin = new String(pinField.getPassword());
        
        if (accountNumberText.isEmpty() || pin.isEmpty()) {
            showErrorMessage("Please enter both account number and PIN");
            return;
        }
        
        try {
            long accountNumber = Long.parseLong(accountNumberText);
            
            if (accountDAO.validateAccount(accountNumber, pin)) {
                // Success - open main dashboard
                new MainDashboard(accountNumber).setVisible(true);
                this.dispose();
            } else {
                showErrorMessage("Invalid account number or PIN");
                pinField.setText("");
            }
        } catch (NumberFormatException e) {
            showErrorMessage("Please enter a valid account number");
        }
    }
    
    private void openCreateAccountFrame() {
        new CreateAccountFrame().setVisible(true);
    }
    
    private void showErrorMessage(String message) {
        JOptionPane.showMessageDialog(this, message, "Login Error", JOptionPane.ERROR_MESSAGE);
    }
    
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            try {
                UIManager.setLookAndFeel(UIManager.getSystemLookAndFeel());
            } catch (Exception e) {
                e.printStackTrace();
            }
            new LoginFrame().setVisible(true);
        });
    }
}


---

## 8. Component Placement and Alignment 

### MainDashboard.java:
```java
package com.sbi.banking.gui;

import com.sbi.banking.dao.AccountDAO;
import com.sbi.banking.dao.CustomerDAO;
import com.sbi.banking.models.Account;
import com.sbi.banking.models.Customer;
import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.text.NumberFormat;
import java.util.Locale;

public class MainDashboard extends JFrame {
    private long accountNumber;
    private Account currentAccount;
    private Customer currentCustomer;
    private AccountDAO accountDAO;
    private CustomerDAO customerDAO;
    
    // UI Components
    private JLabel welcomeLabel;
    private JLabel balanceLabel;
    private JLabel accountInfoLabel;
    private JButton balanceInquiryButton;
    private JButton transferMoneyButton;
    private JButton transactionHistoryButton;
    private JButton logoutButton;
    
    public MainDashboard(long accountNumber) {
        this.accountNumber = accountNumber;
        this.accountDAO = new AccountDAO();
        this.customerDAO = new CustomerDAO();
        
        loadAccountData();
        initializeComponents();
        setupLayout();
        setupEventHandlers();
    }
    
    private void loadAccountData() {
        currentAccount = accountDAO.getAccountByNumber(accountNumber);
        if (currentAccount != null) {
            currentCustomer = customerDAO.getCustomerById(currentAccount.getCustomerId());
        }
    }
    
    private void initializeComponents() {
        setTitle("SBI Banking Dashboard");
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setSize(800, 600);
        setLocationRelativeTo(null);
        setResizable(false);
        
        // Set background
        getContentPane().setBackground(new Color(245, 245, 245));
        
        // Initialize components
        String customerName = currentCustomer != null ? currentCustomer.getFullName() : "Customer";
        welcomeLabel = new JLabel("Welcome, " + customerName);
        welcomeLabel.setFont(new Font("Arial", Font.BOLD, 24));
        welcomeLabel.setForeground(new Color(0, 51, 102));
        
        // Format balance
        NumberFormat currencyFormat = NumberFormat.getCurrencyInstance(new Locale("en", "IN"));
        String formattedBalance = currencyFormat.format(currentAccount.getBalance());
        
        balanceLabel = new JLabel("Current Balance: " + formattedBalance);
        balanceLabel.setFont(new Font("Arial", Font.BOLD, 18));
        balanceLabel.setForeground(new Color(46, 125, 50));
        
        accountInfoLabel = new JLabel("Account: " + accountNumber + " | Type: " + currentAccount.getAccountType());
        accountInfoLabel.setFont(new Font("Arial", Font.PLAIN, 12));
        accountInfoLabel.setForeground(new Color(100, 100, 100));
        
        // Create buttons with proper styling
        balanceInquiryButton = createStyledButton("Balance Inquiry", new Color(33, 150, 243));
        transferMoneyButton = createStyledButton("Transfer Money", new Color(76, 175, 80));
        transactionHistoryButton = createStyledButton("Transaction History", new Color(156, 39, 176));
        logoutButton = createStyledButton("Logout", new Color(244, 67, 54));
    }
    
    private JButton createStyledButton(String text, Color backgroundColor) {
        JButton button = new JButton(text);
        button.setFont(new Font("Arial", Font.BOLD, 14));
        button.setBackground(backgroundColor);
        button.setForeground(Color.WHITE);
        button.setBorder(BorderFactory.createEmptyBorder(15, 25, 15, 25));
        button.setFocusPainted(false);
        button.setCursor(new Cursor(Cursor.HAND_CURSOR));
        button.setPreferredSize(new Dimension(200, 50));
        return button;
    }
    
    private void setupLayout() {
        setLayout(new BorderLayout());
        
        // Header Panel
        JPanel headerPanel = new JPanel();
        headerPanel.setBackground(new Color(0, 51, 102));
        headerPanel.setPreferredSize(new Dimension(800, 100));
        headerPanel.setLayout(new GridBagLayout());
        
        GridBagConstraints gbc = new GridBagConstraints();
        
        JLabel titleLabel = new JLabel("SBI BANKING DASHBOARD");
        titleLabel.setFont(new Font("Arial", Font.BOLD, 20));
        titleLabel.setForeground(Color.WHITE);
        
        gbc.gridx = 0; gbc.gridy = 0;
        gbc.insets = new Insets(10, 0, 5, 0);
        headerPanel.add(titleLabel, gbc);
        
        JLabel dateLabel = new JLabel(new java.util.Date().toString());
        dateLabel.setFont(new Font("Arial", Font.PLAIN, 12));
        dateLabel.setForeground(new Color(200, 200, 200));
        
        gbc.gridy = 1;
        gbc.insets = new Insets(0, 0, 10, 0);
        headerPanel.add(dateLabel, gbc);
        
        // Main Content Panel
        JPanel mainPanel = new JPanel();
        mainPanel.setBackground(Color.WHITE);
        mainPanel.setLayout(new GridBagLayout());
        mainPanel.setBorder(BorderFactory.createEmptyBorder(30, 30, 30, 30));
        
        gbc = new GridBagConstraints();
        
        // Welcome section
        gbc.gridx = 0; gbc.gridy = 0;
        gbc.gridwidth = 2;
        gbc.insets = new Insets(0, 0, 20, 0);
        gbc.anchor = GridBagConstraints.CENTER;
        mainPanel.add(welcomeLabel,
Responsive Design Implementation:
javapublic class ResponsivePanel extends JPanel {
    private int minWidth = 320;
    private int maxWidth = 1200;
    
    public ResponsivePanel() {
        addComponentListener(new ComponentAdapter() {
            @Override
            public void componentResized(ComponentEvent e) {
                adjustLayout();
            }
        });
    }
    
    private void adjustLayout() {
        int width = getWidth();
        
        if (width < 600) {
            // Mobile layout - single column
            setLayout(new BoxLayout(this, BoxLayout.Y_AXIS));
        } else if (width < 900) {
            // Tablet layout - two columns
            setLayout(new GridLayout(0, 2, 10, 10));
        } else {
            // Desktop layout - multiple columns
            setLayout(new GridLayout(0, 3, 15, 15));
        }
        
        revalidate();
        repaint();
    }
}
**Accessibility Implementation:**
java// Screen reader support
JButton submitButton = new JButton("Submit Form");
submitButton.getAccessibleContext().setAccessibleDescription(
    "Submit the user registration form with entered information"
);

// Keyboard navigation
submitButton.setMnemonic(KeyEvent.VK_S);
submitButton.setToolTipText("Alt+S to submit form");

// Focus management
JTextField usernameField = new JTextField();
usernameField.getAccessibleContext().setAccessibleName("Username");
usernameField.getAccessibleContext().setAccessibleDescription(
    "Enter your username, 3-20 characters, letters and numbers only"
);

// High contrast support
public void applyHighContrastTheme() {
    Color backgroundColor = new Color(0, 0, 0);
    Color foregroundColor = new Color(255, 255, 255);
    Color highlightColor = new Color(255, 255, 0);
    
    UIManager.put("Panel.background", backgroundColor);
    UIManager.put("Label.foreground", foregroundColor);
    UIManager.put("Button.background", backgroundColor);
    UIManager.put("Button.foreground", foregroundColor);
    UIManager.put("TextField.selectionBackground", highlightColor);
}
Keyboard Navigation Implementation:

Tab Order: Logical sequence through interactive elements
Keyboard Shortcuts: Mnemonics and accelerators for common actions
Focus Indicators: Visual feedback for currently focused element
Skip Navigation: Quick access to main content areas
Modal Management: Proper focus trapping in dialogs

Internationalization (i18n) Support:
java// Resource bundle implementation
public class MessageBundle {
    private static ResourceBundle bundle;
    
    static {
        Locale currentLocale = Locale.getDefault();
        bundle = ResourceBundle.getBundle("messages", currentLocale);
    }
    
    public static String getMessage(String key) {
        try {
            return bundle.getString(key);
        } catch (MissingResourceException e) {
            return "!" + key + "!";
        }
    }
    
    public static void setLocale(Locale locale) {
        bundle = ResourceBundle.getBundle("messages", locale);
    }
}

// Dynamic text direction support
public void applyTextDirection(ComponentOrientation orientation) {
    this.setComponentOrientation(orientation);
    
    // Recursively apply to all child components
    applyOrientationToChildren(this, orientation);
}
