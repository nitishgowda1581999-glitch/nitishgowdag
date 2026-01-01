import pandas as pd

df = pd.read_csv('/content/Personal_Finance_Dataset.csv')
display(df.head())


class Category:
    def __init__(self, name, type):
        self.name = name
        self.type = type

    def __repr__(self):
        return f"Category(name='{self.name}', type='{self.type}')"

class Transaction:
    def __init__(self, date, description, category, amount, type):
        self.date = date
        self.description = description
        self.category = category  # This should be a Category object
        self.amount = amount
        self.type = type

    def __repr__(self):
        return f"Transaction(date='{self.date}', description='{self.description}', category={self.category.name}, amount={self.amount}, type='{self.type}')"

print("Category and Transaction classes defined.")




import pandas as pd
import os
from datetime import datetime

def save_data(transactions, categories, transactions_file='transactions.csv', categories_file='categories.csv'):
    """
    Saves transaction and category data to CSV files.
    """
    # Save Categories
    if categories:
        categories_data = [{'name': cat.name, 'type': cat.type} for cat in categories]
        categories_df = pd.DataFrame(categories_data)
        categories_df.to_csv(categories_file, index=False)
        print(f"Categories saved to {categories_file}")
    else:
        # Create an empty DataFrame and save it if no categories exist
        pd.DataFrame(columns=['name', 'type']).to_csv(categories_file, index=False)
        print("No categories to save. Created empty categories file.")

    # Save Transactions
    if transactions:
        transactions_data = [
            {
                'date': t.date.strftime('%Y-%m-%d'), # Store date as string
                'description': t.description,
                'category_name': t.category.name if t.category else None,
                'amount': t.amount,
                'type': t.type
            }
            for t in transactions
        ]
        transactions_df = pd.DataFrame(transactions_data)
        transactions_df.to_csv(transactions_file, index=False)
        print(f"Transactions saved to {transactions_file}")
    else:
        # Create an empty DataFrame and save it if no transactions exist
        pd.DataFrame(columns=['date', 'description', 'category_name', 'amount', 'type']).to_csv(transactions_file, index=False)
        print("No transactions to save. Created empty transactions file.")

def load_data(transactions_file='transactions.csv', categories_file='categories.csv'):
    """
    Loads transaction and category data from CSV files.
    """
    loaded_categories = []
    loaded_transactions = []
    category_map = {}

    # Load Categories
    if os.path.exists(categories_file):
        categories_df = pd.read_csv(categories_file)
        for _, row in categories_df.iterrows():
            category = Category(name=row['name'], type=row['type'])
            loaded_categories.append(category)
            category_map[row['name']] = category
        print(f"Categories loaded from {categories_file}")
    else:
        print(f"Categories file not found: {categories_file}. Returning empty lists.")
        return [], []

    # Load Transactions
    if os.path.exists(transactions_file):
        transactions_df = pd.read_csv(transactions_file)
        for _, row in transactions_df.iterrows():
            category_name = row['category_name']
            category = category_map.get(category_name) # Get Category object from map
            if category is None and category_name is not None: # Handle cases where category_name might not exist in category_map
                print(f"Warning: Category '{category_name}' for transaction '{row['description']}' not found. Skipping category assignment.")

            transaction = Transaction(
                date=datetime.strptime(row['date'], '%Y-%m-%d').date(), # Convert date string to date object
                description=row['description'],
                category=category, # Assign the Category object
                amount=row['amount'],
                type=row['type']
            )
            loaded_transactions.append(transaction)
        print(f"Transactions loaded from {transactions_file}")
    else:
        print(f"Transactions file not found: {transactions_file}. Returning empty transaction list.")

    return loaded_transactions, loaded_categories

print("Data persistence functions `save_data` and `load_data` defined.")




from datetime import datetime

class FinanceManager:
    def __init__(self):
        self.transactions = []
        self.categories = []
        self.category_map = {}

        # Attempt to load existing data
        loaded_transactions, loaded_categories = load_data()

        if loaded_transactions or loaded_categories:
            self.transactions = loaded_transactions
            self.categories = loaded_categories
            for cat in self.categories:
                self.category_map[cat.name] = cat
            print("FinanceManager initialized: Data loaded from files.")
        else:
            # If no data loaded from files, populate from the df DataFrame
            print("FinanceManager initialized: No existing data files found, populating from DataFrame.")
            for index, row in df.iterrows():
                category_name = row['Category']
                transaction_type = row['Type'] # Assuming 'Type' column in df indicates transaction type (Income/Expense) and category type

                # Create or get Category object
                if category_name not in self.category_map:
                    category_obj = Category(name=category_name, type=transaction_type)
                    self.categories.append(category_obj)
                    self.category_map[category_name] = category_obj
                else:
                    category_obj = self.category_map[category_name]

                # Create Transaction object
                transaction_date = datetime.strptime(row['Date'], '%Y-%m-%d').date() # Convert date string to date object
                transaction_obj = Transaction(
                    date=transaction_date,
                    description=row['Transaction Description'],
                    category=category_obj,
                    amount=row['Amount'],
                    type=transaction_type
                )
                self.transactions.append(transaction_obj)
            print(f"Loaded {len(self.transactions)} transactions and {len(self.categories)} categories from DataFrame.")

    def add_transaction(self, transaction):
        self.transactions.append(transaction)

    def view_transactions(self):
        return self.transactions

    # Placeholder for update and delete methods (to be implemented later)
    def update_transaction(self, transaction_id, new_data):
        pass

    def delete_transaction(self, transaction_id):
        pass

print("FinanceManager class defined.")
finance_manager = FinanceManager()



from datetime import datetime

class FinanceManager:
    def __init__(self):
        self.transactions = []
        self.categories = []
        self.category_map = {}

        # Attempt to load existing data
        loaded_transactions, loaded_categories = load_data()

        if loaded_transactions or loaded_categories:
            self.transactions = loaded_transactions
            self.categories = loaded_categories
            for cat in self.categories:
                self.category_map[cat.name] = cat
            print("FinanceManager initialized: Data loaded from files.")
        else:
            # If no data loaded from files, populate from the df DataFrame
            print("FinanceManager initialized: No existing data files found, populating from DataFrame.")
            for index, row in df.iterrows():
                category_name = row['Category']
                transaction_type = row['Type'] # Assuming 'Type' column in df indicates transaction type (Income/Expense) and category type

                # Create or get Category object
                if category_name not in self.category_map:
                    category_obj = Category(name=category_name, type=transaction_type)
                    self.categories.append(category_obj)
                    self.category_map[category_name] = category_obj
                else:
                    category_obj = self.category_map[category_name]

                # Create Transaction object
                transaction_date = datetime.strptime(row['Date'], '%Y-%m-%d').date() # Convert date string to date object
                transaction_obj = Transaction(
                    date=transaction_date,
                    description=row['Transaction Description'],
                    category=category_obj,
                    amount=row['Amount'],
                    type=transaction_type
                )
                self.transactions.append(transaction_obj)
            print(f"Loaded {len(self.transactions)} transactions and {len(self.categories)} categories from DataFrame.")

    def add_transaction(self, transaction):
        self.transactions.append(transaction)

    def view_transactions(self):
        return self.transactions

    def update_transaction(self, index, new_data):
        if 0 <= index < len(self.transactions):
            # Update fields if new_data is provided for them
            if 'date' in new_data: self.transactions[index].date = new_data['date']
            if 'description' in new_data: self.transactions[index].description = new_data['description']
            if 'category' in new_data: 
                # If new_data['category'] is a string, try to find/create Category object
                if isinstance(new_data['category'], str):
                    category_name = new_data['category']
                    # Assume type for new category is the same as the transaction type
                    # This might need refinement based on actual system design if category types are more complex
                    transaction_type = self.transactions[index].type 
                    if category_name not in self.category_map:
                        category_obj = Category(name=category_name, type=transaction_type)
                        self.categories.append(category_obj)
                        self.category_map[category_name] = category_obj
                    self.transactions[index].category = self.category_map[category_name]
                elif isinstance(new_data['category'], Category):
                    self.transactions[index].category = new_data['category']
            if 'amount' in new_data: self.transactions[index].amount = new_data['amount']
            if 'type' in new_data: self.transactions[index].type = new_data['type']
            print(f"Transaction at index {index} updated.")
            return True
        print(f"Invalid transaction index: {index}")
        return False

    def delete_transaction(self, index):
        if 0 <= index < len(self.transactions):
            del self.transactions[index]
            print(f"Transaction at index {index} deleted.")
            return True
        print(f"Invalid transaction index: {index}")
        return False

print("FinanceManager class defined with add, view, update, and delete functionalities.")
finance_manager = FinanceManager()



from datetime import datetime
from collections import defaultdict

class FinanceManager:
    def __init__(self):
        self.transactions = []
        self.categories = []
        self.category_map = {}

        # Attempt to load existing data
        loaded_transactions, loaded_categories = load_data()

        if loaded_transactions or loaded_categories:
            self.transactions = loaded_transactions
            self.categories = loaded_categories
            for cat in self.categories:
                self.category_map[cat.name] = cat
            print("FinanceManager initialized: Data loaded from files.")
        else:
            # If no data loaded from files, populate from the df DataFrame
            print("FinanceManager initialized: No existing data files found, populating from DataFrame.")
            for index, row in df.iterrows():
                category_name = row['Category']
                transaction_type = row['Type'] # Assuming 'Type' column in df indicates transaction type (Income/Expense) and category type

                # Create or get Category object
                if category_name not in self.category_map:
                    category_obj = Category(name=category_name, type=transaction_type)
                    self.categories.append(category_obj)
                    self.category_map[category_name] = category_obj
                else:
                    category_obj = self.category_map[category_name]

                # Create Transaction object
                transaction_date = datetime.strptime(row['Date'], '%Y-%m-%d').date() # Convert date string to date object
                transaction_obj = Transaction(
                    date=transaction_date,
                    description=row['Transaction Description'],
                    category=category_obj,
                    amount=row['Amount'],
                    type=transaction_type
                )
                self.transactions.append(transaction_obj)
            print(f"Loaded {len(self.transactions)} transactions and {len(self.categories)} categories from DataFrame.")

    def add_transaction(self, transaction):
        self.transactions.append(transaction)

    def view_transactions(self):
        return self.transactions

    def update_transaction(self, index, new_data):
        if 0 <= index < len(self.transactions):
            # Update fields if new_data is provided for them
            if 'date' in new_data: self.transactions[index].date = new_data['date']
            if 'description' in new_data: self.transactions[index].description = new_data['description']
            if 'category' in new_data:
                # If new_data['category'] is a string, try to find/create Category object
                if isinstance(new_data['category'], str):
                    category_name = new_data['category']
                    # Assume type for new category is the same as the transaction type
                    # This might need refinement based on actual system design if category types are more complex
                    transaction_type = self.transactions[index].type
                    if category_name not in self.category_map:
                        category_obj = Category(name=category_name, type=transaction_type)
                        self.categories.append(category_obj)
                        self.category_map[category_name] = category_obj
                    self.transactions[index].category = self.category_map[category_name]
                elif isinstance(new_data['category'], Category):
                    self.transactions[index].category = new_data['category']
            if 'amount' in new_data: self.transactions[index].amount = new_data['amount']
            if 'type' in new_data: self.transactions[index].type = new_data['type']
            print(f"Transaction at index {index} updated.")
            return True
        print(f"Invalid transaction index: {index}")
        return False

    def delete_transaction(self, index):
        if 0 <= index < len(self.transactions):
            del self.transactions[index]
            print(f"Transaction at index {index} deleted.")
            return True
        print(f"Invalid transaction index: {index}")
        return False

    def generate_category_expense_report(self):
        expense_by_category = defaultdict(float)
        for transaction in self.transactions:
            if transaction.type == 'Expense':
                expense_by_category[transaction.category.name] += transaction.amount
        return dict(expense_by_category)

    def generate_income_expense_report(self, start_date=None, end_date=None):
        total_income = 0.0
        total_expenses = 0.0

        for transaction in self.transactions:
            # Filter by date if start_date and/or end_date are provided
            if start_date and transaction.date < start_date:
                continue
            if end_date and transaction.date > end_date:
                continue

            if transaction.type == 'Income':
                total_income += transaction.amount
            elif transaction.type == 'Expense':
                total_expenses += transaction.amount

        return {'total_income': total_income, 'total_expenses': total_expenses}

    def generate_monthly_spending_report(self):
        monthly_spending = defaultdict(float)
        for transaction in self.transactions:
            if transaction.type == 'Expense':
                month_year = transaction.date.strftime('%Y-%m')
                monthly_spending[month_year] += transaction.amount
        # Sort the monthly spending report by month_year for better readability
        return dict(sorted(monthly_spending.items()))

print("FinanceManager class updated with reporting functionalities.")
finance_manager = FinanceManager()

# Example usage (for verification, can be removed later)
# print("\nCategory Expense Report:", finance_manager.generate_category_expense_report())
# print("\nIncome vs Expense Report (All time):", finance_manager.generate_income_expense_report())
# print("\nMonthly Spending Report:", finance_manager.generate_monthly_spending_report())




from datetime import datetime
from collections import defaultdict

class FinanceManager:
    def __init__(self):
        self.transactions = []
        self.categories = []
        self.category_map = {}

        # Attempt to load existing data
        loaded_transactions, loaded_categories = load_data()

        if loaded_transactions or loaded_categories:
            self.transactions = loaded_transactions
            self.categories = loaded_categories
            for cat in self.categories:
                self.category_map[cat.name] = cat
            print("FinanceManager initialized: Data loaded from files.")
        else:
            # If no data loaded from files, populate from the df DataFrame
            print("FinanceManager initialized: No existing data files found, populating from DataFrame.")
            for index, row in df.iterrows():
                category_name = row['Category']
                transaction_type = row['Type'] # Assuming 'Type' column in df indicates transaction type (Income/Expense) and category type

                # Create or get Category object
                if category_name not in self.category_map:
                    category_obj = Category(name=category_name, type=transaction_type)
                    self.categories.append(category_obj)
                    self.category_map[category_name] = category_obj
                else:
                    category_obj = self.category_map[category_name]

                # Create Transaction object
                transaction_date = datetime.strptime(row['Date'], '%Y-%m-%d').date() # Convert date string to date object
                transaction_obj = Transaction(
                    date=transaction_date,
                    description=row['Transaction Description'],
                    category=category_obj,
                    amount=row['Amount'],
                    type=transaction_type
                )
                self.transactions.append(transaction_obj)
            print(f"Loaded {len(self.transactions)} transactions and {len(self.categories)} categories from DataFrame.")

    def add_transaction(self, transaction):
        self.transactions.append(transaction)

    def view_transactions(self):
        return self.transactions

    def update_transaction(self, index, new_data):
        if 0 <= index < len(self.transactions):
            # Update fields if new_data is provided for them
            if 'date' in new_data: self.transactions[index].date = new_data['date']
            if 'description' in new_data: self.transactions[index].description = new_data['description']
            if 'category' in new_data:
                # If new_data['category'] is a string, try to find/create Category object
                if isinstance(new_data['category'], str):
                    category_name = new_data['category']
                    # Assume type for new category is the same as the transaction type
                    # This might need refinement based on actual system design if category types are more complex
                    transaction_type = self.transactions[index].type
                    if category_name not in self.category_map:
                        category_obj = Category(name=category_name, type=transaction_type)
                        self.categories.append(category_obj)
                        self.category_map[category_name] = category_obj
                    self.transactions[index].category = self.category_map[category_name]
                elif isinstance(new_data['category'], Category):
                    self.transactions[index].category = new_data['category']
            if 'amount' in new_data: self.transactions[index].amount = new_data['amount']
            if 'type' in new_data: self.transactions[index].type = new_data['type']
            print(f"Transaction at index {index} updated.")
            return True
        print(f"Invalid transaction index: {index}")
        return False

    def delete_transaction(self, index):
        if 0 <= index < len(self.transactions):
            del self.transactions[index]
            print(f"Transaction at index {index} deleted.")
            return True
        print(f"Invalid transaction index: {index}")
        return False

    def generate_category_expense_report(self):
        expense_by_category = defaultdict(float)
        for transaction in self.transactions:
            if transaction.type == 'Expense':
                expense_by_category[transaction.category.name] += transaction.amount
        return dict(expense_by_category)

    def generate_income_expense_report(self, start_date=None, end_date=None):
        total_income = 0.0
        total_expenses = 0.0

        for transaction in self.transactions:
            # Filter by date if start_date and/or end_date are provided
            if start_date and transaction.date < start_date:
                continue
            if end_date and transaction.date > end_date:
                continue

            if transaction.type == 'Income':
                total_income += transaction.amount
            elif transaction.type == 'Expense':
                total_expenses += transaction.amount

        return {'total_income': total_income, 'total_expenses': total_expenses}

    def generate_monthly_spending_report(self):
        monthly_spending = defaultdict(float)
        for transaction in self.transactions:
            if transaction.type == 'Expense':
                month_year = transaction.date.strftime('%Y-%m')
                monthly_spending[month_year] += transaction.amount
        # Sort the monthly spending report by month_year for better readability
        return dict(sorted(monthly_spending.items()))

    def get_top_spending_categories(self, n=5):
        expense_report = self.generate_category_expense_report()
        # Sort categories by expense in descending order
        sorted_categories = sorted(expense_report.items(), key=lambda item: item[1], reverse=True)
        return sorted_categories[:n]

    def get_spending_trends_over_time(self):
        return self.generate_monthly_spending_report()

    def get_average_monthly_expense(self):
        monthly_spending = self.generate_monthly_spending_report()
        if not monthly_spending:
            return 0.0
        total_spending = sum(monthly_spending.values())
        average_expense = total_spending / len(monthly_spending)
        return average_expense

print("FinanceManager class updated with spending pattern analysis functionalities.")
finance_manager = FinanceManager()



print("\n--- Spending Pattern Analysis Reports ---")
print("\nTop 3 Spending Categories:", finance_manager.get_top_spending_categories(n=3))
print("\nSpending Trends Over Time:", finance_manager.get_spending_trends_over_time())
print("\nAverage Monthly Expense:", finance_manager.get_average_monthly_expense())



from datetime import datetime
from collections import defaultdict

# Re-define FinanceManager with the updated methods
class FinanceManager:
    def __init__(self):
        self.transactions = []
        self.categories = []
        self.category_map = {}

        # Attempt to load existing data
        loaded_transactions, loaded_categories = load_data()

        if loaded_transactions or loaded_categories:
            self.transactions = loaded_transactions
            self.categories = loaded_categories
            for cat in self.categories:
                self.category_map[cat.name] = cat
            print("FinanceManager initialized: Data loaded from files.")
        else:
            # If no data loaded from files, populate from the df DataFrame
            print("FinanceManager initialized: No existing data files found, populating from DataFrame.")
            for index, row in df.iterrows():
                category_name = row['Category']
                transaction_type = row['Type'] # Assuming 'Type' column in df indicates transaction type (Income/Expense) and category type

                # Create or get Category object
                if category_name not in self.category_map:
                    category_obj = Category(name=category_name, type=transaction_type)
                    self.categories.append(category_obj)
                    self.category_map[category_name] = category_obj
                else:
                    category_obj = self.category_map[category_name]

                # Create Transaction object
                transaction_date = datetime.strptime(row['Date'], '%Y-%m-%d').date() # Convert date string to date object
                transaction_obj = Transaction(
                    date=transaction_date,
                    description=row['Transaction Description'],
                    category=category_obj,
                    amount=row['Amount'],
                    type=transaction_type
                )
                self.transactions.append(transaction_obj)
            print(f"Loaded {len(self.transactions)} transactions and {len(self.categories)} categories from DataFrame.")

    def add_transaction(self, transaction):
        self.transactions.append(transaction)
        # Also ensure the category is in the manager's category list if it's new
        if transaction.category.name not in self.category_map:
            self.categories.append(transaction.category)
            self.category_map[transaction.category.name] = transaction.category

    def view_transactions(self):
        return self.transactions

    def update_transaction(self, index, new_data):
        if 0 <= index < len(self.transactions):
            # Update fields if new_data is provided for them
            if 'date' in new_data: self.transactions[index].date = new_data['date']
            if 'description' in new_data: self.transactions[index].description = new_data['description']
            if 'category' in new_data:
                # If new_data['category'] is a string, try to find/create Category object
                if isinstance(new_data['category'], str):
                    category_name = new_data['category']
                    # Assume type for new category is the same as the transaction type
                    # This might need refinement based on actual system design if category types are more complex
                    transaction_type = self.transactions[index].type
                    if category_name not in self.category_map:
                        category_obj = Category(name=category_name, type=transaction_type)
                        self.categories.append(category_obj)
                        self.category_map[category_name] = category_obj
                    self.transactions[index].category = self.category_map[category_name]
                elif isinstance(new_data['category'], Category):
                    self.transactions[index].category = new_data['category']
            if 'amount' in new_data: self.transactions[index].amount = new_data['amount']
            if 'type' in new_data: self.transactions[index].type = new_data['type']
            print(f"Transaction at index {index} updated.")
            return True
        print(f"Invalid transaction index: {index}")
        return False

    def delete_transaction(self, index):
        if 0 <= index < len(self.transactions):
            del self.transactions[index]
            print(f"Transaction at index {index} deleted.")
            return True
        print(f"Invalid transaction index: {index}")
        return False

    def generate_category_expense_report(self):
        expense_by_category = defaultdict(float)
        for transaction in self.transactions:
            if transaction.type == 'Expense':
                expense_by_category[transaction.category.name] += transaction.amount
        return dict(expense_by_category)

    def generate_income_expense_report(self, start_date=None, end_date=None):
        total_income = 0.0
        total_expenses = 0.0

        for transaction in self.transactions:
            # Filter by date if start_date and/or end_date are provided
            if start_date and transaction.date < start_date:
                continue
            if end_date and transaction.date > end_date:
                continue

            if transaction.type == 'Income':
                total_income += transaction.amount
            elif transaction.type == 'Expense':
                total_expenses += transaction.amount

        return {'total_income': total_income, 'total_expenses': total_expenses}

    def generate_monthly_spending_report(self):
        monthly_spending = defaultdict(float)
        for transaction in self.transactions:
            if transaction.type == 'Expense':
                month_year = transaction.date.strftime('%Y-%m')
                monthly_spending[month_year] += transaction.amount
        # Sort the monthly spending report by month_year for better readability
        return dict(sorted(monthly_spending.items()))

    def get_top_spending_categories(self, n=5):
        expense_report = self.generate_category_expense_report()
        # Sort categories by expense in descending order
        sorted_categories = sorted(expense_report.items(), key=lambda item: item[1], reverse=True)
        return sorted_categories[:n]

    def get_spending_trends_over_time(self):
        return self.generate_monthly_spending_report()

    def get_average_monthly_expense(self):
        monthly_spending = self.generate_monthly_spending_report()
        if not monthly_spending:
            return 0.0
        total_spending = sum(monthly_spending.values())
        average_expense = total_spending / len(monthly_spending)
        return average_expense


def main_cli():
    global finance_manager # Ensure finance_manager is accessible
    finance_manager = FinanceManager()

    while True:
        print("\n--- Personal Finance Manager CLI ---")
        print("1. Add New Transaction")
        print("2. View All Transactions")
        print("3. Update Transaction")
        print("4. Delete Transaction")
        print("5. Generate Category Expense Report")
        print("6. Generate Income vs. Expense Report")
        print("7. Generate Monthly Spending Report")
        print("8. Get Top Spending Categories")
        print("9. Get Spending Trends Over Time")
        print("10. Get Average Monthly Expense")
        print("0. Save Data and Exit")

        choice = input("Enter your choice: ")

        if choice == '1': # Add New Transaction
            try:
                date_str = input("Enter date (YYYY-MM-DD): ")
                transaction_date = datetime.strptime(date_str, '%Y-%m-%d').date()
                description = input("Enter description: ")
                category_name = input("Enter category name: ")
                amount = float(input("Enter amount: "))
                transaction_type = input("Enter type (Income/Expense): ").capitalize()

                if transaction_type not in ['Income', 'Expense']:
                    print("Invalid transaction type. Please enter 'Income' or 'Expense'.")
                    continue

                category_obj = finance_manager.category_map.get(category_name)
                if not category_obj:
                    category_obj = Category(name=category_name, type=transaction_type)
                    finance_manager.categories.append(category_obj)
                    finance_manager.category_map[category_name] = category_obj

                new_transaction = Transaction(transaction_date, description, category_obj, amount, transaction_type)
                finance_manager.add_transaction(new_transaction)
                print("Transaction added successfully!")
            except ValueError:
                print("Invalid input. Please check date format or amount.")

        elif choice == '2': # View All Transactions
            transactions = finance_manager.view_transactions()
            if transactions:
                for i, t in enumerate(transactions):
                    print(f"{i}: {t}")
            else:
                print("No transactions to display.")

        elif choice == '3': # Update Transaction
            try:
                index = int(input("Enter the index of the transaction to update: "))
                if 0 <= index < len(finance_manager.transactions):
                    print(f"Current transaction: {finance_manager.transactions[index]}")
                    new_data = {}
                    date_str = input("Enter new date (YYYY-MM-DD, leave blank to keep current): ")
                    if date_str: new_data['date'] = datetime.strptime(date_str, '%Y-%m-%d').date()

                    description = input("Enter new description (leave blank to keep current): ")
                    if description: new_data['description'] = description

                    category_name = input("Enter new category name (leave blank to keep current): ")
                    if category_name: new_data['category'] = category_name

                    amount_str = input("Enter new amount (leave blank to keep current): ")
                    if amount_str: new_data['amount'] = float(amount_str)

                    type_str = input("Enter new type (Income/Expense, leave blank to keep current): ").capitalize()
                    if type_str and type_str in ['Income', 'Expense']: new_data['type'] = type_str
                    elif type_str and type_str not in ['Income', 'Expense']:
                        print("Invalid type provided. Keeping current type.")

                    if new_data: finance_manager.update_transaction(index, new_data)
                    else: print("No changes provided.")
                else:
                    print("Invalid transaction index.")
            except ValueError:
                print("Invalid input. Please enter a valid number or date format.")

        elif choice == '4': # Delete Transaction
            try:
                index = int(input("Enter the index of the transaction to delete: "))
                finance_manager.delete_transaction(index)
            except ValueError:
                print("Invalid input. Please enter a valid number.")

        elif choice == '5': # Generate Category Expense Report
            report = finance_manager.generate_category_expense_report()
            if report:
                print("\n--- Category Expense Report ---")
                for category, amount in report.items():
                    print(f"{category}: ${amount:.2f}")
            else:
                print("No expense transactions found.")

        elif choice == '6': # Generate Income vs. Expense Report
            start_date_str = input("Enter start date (YYYY-MM-DD, leave blank for all): ")
            end_date_str = input("Enter end date (YYYY-MM-DD, leave blank for all): ")
            start_date = None
            end_date = None

            try:
                if start_date_str: start_date = datetime.strptime(start_date_str, '%Y-%m-%d').date()
                if end_date_str: end_date = datetime.strptime(end_date_str, '%Y-%m-%d').date()

                report = finance_manager.generate_income_expense_report(start_date, end_date)
                print("\n--- Income vs. Expense Report ---")
                print(f"Total Income: ${report['total_income']:.2f}")
                print(f"Total Expenses: ${report['total_expenses']:.2f}")
                print(f"Net Balance: ${(report['total_income'] - report['total_expenses']):.2f}")
            except ValueError:
                print("Invalid date format. Please use YYYY-MM-DD.")

        elif choice == '7': # Generate Monthly Spending Report
            report = finance_manager.generate_monthly_spending_report()
            if report:
                print("\n--- Monthly Spending Report ---")
                for month, amount in report.items():
                    print(f"{month}: ${amount:.2f}")
            else:
                print("No monthly spending found.")

        elif choice == '8': # Get Top Spending Categories
            try:
                n_str = input("Enter number of top categories (default 5): ")
                n = int(n_str) if n_str else 5
                report = finance_manager.get_top_spending_categories(n)
                if report:
                    print(f"\n--- Top {n} Spending Categories ---")
                    for category, amount in report:
                        print(f"{category}: ${amount:.2f}")
                else:
                    print("No expense data to analyze.")
            except ValueError:
                print("Invalid input for number of categories.")

        elif choice == '9': # Get Spending Trends Over Time
            report = finance_manager.get_spending_trends_over_time()
            if report:
                print("\n--- Spending Trends Over Time ---")
                for month_year, total_expense in report.items():
                    print(f"{month_year}: ${total_expense:.2f}")
            else:
                print("No spending trends to display.")

        elif choice == '10': # Get Average Monthly Expense
            avg_expense = finance_manager.get_average_monthly_expense()
            print("\n--- Average Monthly Expense ---")
            print(f"Average Monthly Expense: ${avg_expense:.2f}")

        elif choice == '0': # Save Data and Exit
            save_data(finance_manager.transactions, finance_manager.categories)
            print("Data saved. Exiting application.")
            break

        else:
            print("Invalid choice. Please try again.")

# Instantiate finance_manager once globally for CLI
finance_manager = FinanceManager()

print("CLI function `main_cli` defined. You can run `main_cli()` to start the interface.")




main_cli()

Categories loaded from categories.csv
Transactions loaded from transactions.csv
FinanceManager initialized: Data loaded from files.

--- Personal Finance Manager CLI ---
1. Add New Transaction
2. View All Transactions
3. Update Transaction
4. Delete Transaction
5. Generate Category Expense Report
6. Generate Income vs. Expense Report
7. Generate Monthly Spending Report
8. Get Top Spending Categories
9. Get Spending Trends Over Time
10. Get Average Monthly Expense
0. Save Data and Exit
Enter your choice: 1
Enter date (YYYY-MM-DD): 1999-08-13
Enter description: N121
Enter category name: Nitish
Enter amount: 2999
Enter type (Income/Expense): Expense
Transaction added successfully!

--- Personal Finance Manager CLI ---
1. Add New Transaction
2. View All Transactions
3. Update Transaction
4. Delete Transaction
5. Generate Category Expense Report
6. Generate Income vs. Expense Report
7. Generate Monthly Spending Report
8. Get Top Spending Categories
9. Get Spending Trends Over Time
10. Get Average Monthly Expense
0. Save Data and Exit
Enter your choice: 2
0: Transaction(date='2020-01-02', description='Score each.', category=Food & Drink, amount=1485.69, type='Expense')
1: Transaction(date='2020-01-02', description='Quality throughout.', category=Utilities, amount=1475.58, type='Expense')
2: Transaction(date='2024-08-22', description='444', category=223, amount=3000.0, type='Expense')
3: Transaction(date='2020-01-05', description='Information last everything thank serve.', category=Investment, amount=2291.0, type='Income')
4: Transaction(date='2020-01-13', description='Future choice whatever from.', category=Food & Drink, amount=1126.88, type='Expense')
5: Transaction(date='2020-01-14', description='Benefit suggest page southern.', category=Shopping, amount=448.68, type='Expense')
6: Transaction(date='2020-01-18', description='By two bad fall pick.', category=Food & Drink, amount=1520.03, type='Expense')
7: Transaction(date='2020-01-19', description='Court attorney product significant world.', category=Other, amount=3287.0, type='Income')
8: Transaction(date='2020-01-21', description='Herself law.', category=Entertainment, amount=1914.85, type='Expense')
9: Transaction(date='2020-01-25', description='Have decide environment.', category=Rent, amount=766.06, type='Expense')
10: Transaction(date='2020-01-25', description='Play move each left establish.', category=Health & Fitness, amount=1211.41, type='Expense')
11: Transaction(date='2020-01-26', description='Range successful simply.', category=Salary, amount=1077.09, type='Expense')
12: Transaction(date='2020-01-29', description='Source husband.', category=Shopping, amount=166.81, type='Expense')
13: Transaction(date='2020-01-30', description='Tree note responsibility.', category=Health & Fitness, amount=1158.93, type='Expense')
14: Transaction(date='2020-01-31', description='How trip learn enter.', category=Food & Drink, amount=1325.91, type='Expense')
15: Transaction(date='2020-01-31', description='Much section investment on gun.', category=Rent, amount=1712.08, type='Expense')
16: Transaction(date='2020-01-31', description='Management sense technology check civil.', category=Shopping, amount=563.17, type='Expense')
17: Transaction(date='2020-02-02', description='As high you more wife.', category=Health & Fitness, amount=333.68, type='Expense')
18: Transaction(date='2020-02-04', description='Race mr.', category=Travel, amount=1406.62, type='Expense')
19: Transaction(date='2020-02-05', description='Born itself law west.', category=Rent, amount=1222.17, type='Expense')
20: Transaction(date='2020-02-05', description='Seven medical blood personal.', category=Investment, amount=1838.0, type='Income')
21: Transaction(date='2020-02-07', description='Current hear claim well two.', category=Travel, amount=1979.15, type='Expense')
22: Transaction(date='2020-02-07', description='Food affect upon these story.', category=Entertainment, amount=1372.38, type='Expense')
23: Transaction(date='2020-02-08', description='Around there water.', category=Food & Drink, amount=465.81, type='Expense')
24: Transaction(date='2020-02-08', description='Staff within mouth call.', category=Other, amount=3786.0, type='Income')
25: Transaction(date='2020-02-09', description='Close month parent who up.', category=Entertainment, amount=1827.14, type='Expense')
26: Transaction(date='2020-02-11', description='Human public health.', category=Health & Fitness, amount=433.13, type='Expense')
27: Transaction(date='2020-02-13', description='Building different full open.', category=Salary, amount=294.31, type='Expense')
28: Transaction(date='2020-02-15', description='Detail audience piece director town.', category=Other, amount=4009.0, type='Income')
29: Transaction(date='2020-02-16', description='Method everything.', category=Shopping, amount=730.37, type='Expense')
30: Transaction(date='2020-02-16', description='Much rich think.', category=Utilities, amount=1023.96, type='Expense')
31: Transaction(date='2020-02-18', description='Drug list imagine behind.', category=Investment, amount=1398.0, type='Income')
32: Transaction(date='2020-02-19', description='Interest level rock pull.', category=Investment, amount=3958.0, type='Income')
33: Transaction(date='2020-02-20', description='I fund.', category=Shopping, amount=769.42, type='Expense')
34: Transaction(date='2020-02-26', description='Court movie cell.', category=Travel, amount=1942.45, type='Expense')
35: Transaction(date='2020-02-27', description='Leg themselves away.', category=Food & Drink, amount=1363.76, type='Expense')
36: Transaction(date='2020-02-28', description='Anything yourself.', category=Other, amount=3286.0, type='Income')
37: Transaction(date='2020-02-28', description='Unit support.', category=Other, amount=1795.0, type='Income')
38: Transaction(date='2020-02-29', description='Magazine degree husband around.', category=Travel, amount=1944.06, type='Expense')
39: Transaction(date='2020-03-03', description='World enter.', category=Rent, amount=1742.33, type='Expense')
40: Transaction(date='2020-03-04', description='Institution whatever yet.', category=Entertainment, amount=314.15, type='Expense')
41: Transaction(date='2020-03-04', description='Choice father why often.', category=Food & Drink, amount=1201.9, type='Expense')
42: Transaction(date='2020-03-09', description='Security arm.', category=Rent, amount=1858.91, type='Expense')
43: Transaction(date='2020-03-09', description='Me system church whether.', category=Travel, amount=486.51, type='Expense')
44: Transaction(date='2020-03-11', description='Attention attack technology.', category=Rent, amount=180.45, type='Expense')
45: Transaction(date='2020-03-11', description='Walk now often.', category=Rent, amount=1956.19, type='Expense')
46: Transaction(date='2020-03-12', description='Bank price north first end.', category=Utilities, amount=265.5, type='Expense')
47: Transaction(date='2020-03-12', description='Enter capital population.', category=Utilities, amount=537.46, type='Expense')
48: Transaction(date='2020-03-14', description='Upon very act perform.', category=Shopping, amount=1929.08, type='Expense')
49: Transaction(date='2020-03-15', description='Available defense.', category=Entertainment, amount=1428.77, type='Expense')
50: Transaction(date='2020-03-18', description='Value thing these.', category=Health & Fitness, amount=881.82, type='Expense')
51: Transaction(date='2020-03-21', description='Citizen street region.', category=Rent, amount=503.33, type='Expense')
52: Transaction(date='2020-03-23', description='Property never.', category=Investment, amount=2385.0, type='Income')
53: Transaction(date='2020-03-23', description='Enough threat score.', category=Food & Drink, amount=151.28, type='Expense')
54: Transaction(date='2020-03-24', description='Example decision garden.', category=Entertainment, amount=144.13, type='Expense')
55: Transaction(date='2020-03-28', description='Major town suffer begin.', category=Other, amount=1080.0, type='Income')
56: Transaction(date='2020-04-01', description='Analysis four capital.', category=Travel, amount=1341.27, type='Expense')
57: Transaction(date='2020-04-05', description='Scientist necessary into.', category=Utilities, amount=1449.47, type='Expense')
58: Transaction(date='2020-04-06', description='Away third.', category=Salary, amount=493.56, type='Expense')
59: Transaction(date='2020-04-06', description='Along hard need involve among.', category=Shopping, amount=388.92, type='Expense')
60: Transaction(date='2020-04-10', description='Together decide.', category=Other, amount=3402.0, type='Income')
61: Transaction(date='2020-04-17', description='Government nice.', category=Salary, amount=1729.04, type='Expense')
62: Transaction(date='2020-04-17', description='See understand door class son.', category=Investment, amount=996.0, type='Income')
63: Transaction(date='2020-04-18', description='Agent say forward us.', category=Health & Fitness, amount=1603.18, type='Expense')
64: Transaction(date='2020-04-22', description='City red him.', category=Investment, amount=2058.0, type='Income')
65: Transaction(date='2020-04-23', description='Develop staff least figure.', category=Utilities, amount=849.54, type='Expense')
66: Transaction(date='2020-04-25', description='Age cover foreign.', category=Entertainment, amount=1750.23, type='Expense')
67: Transaction(date='2020-04-26', description='Go meeting quickly.', category=Investment, amount=914.0, type='Income')
68: Transaction(date='2020-04-26', description='Voice boy wife condition.', category=Food & Drink, amount=1938.3, type='Expense')
69: Transaction(date='2020-04-27', description='Board its.', category=Entertainment, amount=340.96, type='Expense')
70: Transaction(date='2020-04-27', description='Paper memory history office.', category=Entertainment, amount=1730.65, type='Expense')
71: Transaction(date='2020-04-29', description='Remember ability.', category=Utilities, amount=764.16, type='Expense')
72: Transaction(date='2020-04-30', description='Hair attorney professional.', category=Travel, amount=1853.77, type='Expense')
73: Transaction(date='2020-05-01', description='Finish summer rest feel finally.', category=Travel, amount=851.78, type='Expense')
74: Transaction(date='2020-05-03', description='I fast camera inside.', category=Salary, amount=318.05, type='Expense')
75: Transaction(date='2020-05-03', description='Past feeling nature.', category=Food & Drink, amount=1162.57, type='Expense')
76: Transaction(date='2020-05-05', description='Expert involve oil.', category=Health & Fitness, amount=123.76, type='Expense')
77: Transaction(date='2020-05-06', description='Kid put memory soldier.', category=Utilities, amount=123.19, type='Expense')
78: Transaction(date='2020-05-06', description='Exist professional.', category=Utilities, amount=146.36, type='Expense')
79: Transaction(date='2020-05-07', description='Stand seem.', category=Investment, amount=3807.0, type='Income')
80: Transaction(date='2020-05-11', description='Southern challenge animal.', category=Investment, amount=825.0, type='Income')
81: Transaction(date='2020-05-12', description='Sure authority increase picture create.', category=Shopping, amount=1318.17, type='Expense')
82: Transaction(date='2020-05-13', description='Hundred challenge reach throughout.', category=Health & Fitness, amount=1870.07, type='Expense')
83: Transaction(date='2020-05-14', description='Despite sound receive let.', category=Health & Fitness, amount=484.99, type='Expense')
84: Transaction(date='2020-05-15', description='True building card.', category=Travel, amount=919.88, type='Expense')
85: Transaction(date='2020-05-16', description='Write other wear through partner.', category=Rent, amount=28.54, type='Expense')
86: Transaction(date='2020-05-17', description='Store natural today season.', category=Rent, amount=155.79, type='Expense')
87: Transaction(date='2020-05-19', description='Else easy point she.', category=Travel, amount=273.59, type='Expense')
88: Transaction(date='2020-05-20', description='Phone ever back experience.', category=Rent, amount=1759.75, type='Expense')
89: Transaction(date='2020-05-21', description='Floor music catch discuss.', category=Utilities, amount=881.99, type='Expense')
90: Transaction(date='2020-05-25', description='Ask imagine my.', category=Travel, amount=1227.24, type='Expense')
91: Transaction(date='2020-05-27', description='Deal information.', category=Food & Drink, amount=1339.05, type='Expense')
92: Transaction(date='2020-05-28', description='Task she.', category=Rent, amount=1878.47, type='Expense')
93: Transaction(date='2020-05-29', description='Accept goal send table.', category=Investment, amount=1376.0, type='Income')
94: Transaction(date='2020-05-30', description='Section national owner determine detail.', category=Utilities, amount=551.97, type='Expense')
95: Transaction(date='2020-05-31', description='Ahead from quickly identify.', category=Health & Fitness, amount=415.16, type='Expense')
96: Transaction(date='2020-05-31', description='Try wonder move trade.', category=Travel, amount=1015.76, type='Expense')
97: Transaction(date='2020-06-01', description='Base investment.', category=Food & Drink, amount=193.67, type='Expense')
98: Transaction(date='2020-06-02', description='Anyone here deep.', category=Travel, amount=97.73, type='Expense')
99: Transaction(date='2020-06-05', description='Seven here sell story.', category=Utilities, amount=1277.86, type='Expense')
100: Transaction(date='2020-06-06', description='Like see.', category=Salary, amount=1107.84, type='Expense')
101: Transaction(date='2020-06-07', description='Difference attention whatever american.', category=Food & Drink, amount=232.63, type='Expense')
102: Transaction(date='2020-06-08', description='Current after charge.', category=Utilities, amount=1095.72, type='Expense')
103: Transaction(date='2020-06-09', description='Prove nor design record.', category=Utilities, amount=865.28, type='Expense')
104: Transaction(date='2020-06-09', description='Paper white responsibility.', category=Other, amount=826.0, type='Income')
105: Transaction(date='2020-06-10', description='Blue agent find.', category=Entertainment, amount=1367.31, type='Expense')
106: Transaction(date='2020-06-10', description='Walk cup option store.', category=Health & Fitness, amount=1562.43, type='Expense')
107: Transaction(date='2020-06-11', description='Recent feeling.', category=Shopping, amount=1947.94, type='Expense')
108: Transaction(date='2020-06-11', description='Site girl into have media.', category=Entertainment, amount=1730.57, type='Expense')
109: Transaction(date='2020-06-12', description='Support state.', category=Utilities, amount=1764.02, type='Expense')
110: Transaction(date='2020-06-14', description='Whether player stock religious.', category=Other, amount=3872.0, type='Income')
111: Transaction(date='2020-06-15', description='Safe whole establish.', category=Entertainment, amount=540.94, type='Expense')
112: Transaction(date='2020-06-18', description='Let stay or focus.', category=Rent, amount=771.26, type='Expense')
113: Transaction(date='2020-06-18', description='Everything late.', category=Other, amount=2322.0, type='Income')
114: Transaction(date='2020-06-18', description='Particular her agreement surface consider.', category=Other, amount=3364.0, type='Income')
115: Transaction(date='2020-06-19', description='Something future they red.', category=Entertainment, amount=453.62, type='Expense')
116: Transaction(date='2020-06-19', description='Act way beat result major.', category=Shopping, amount=663.24, type='Expense')
117: Transaction(date='2020-06-21', description='Position make society behavior.', category=Travel, amount=708.78, type='Expense')
118: Transaction(date='2020-06-21', description='Reality fill ok list.', category=Health & Fitness, amount=1879.08, type='Expense')
119: Transaction(date='2020-06-22', description='International second former reflect.', category=Other, amount=1962.0, type='Income')
120: Transaction(date='2020-06-22', description='Edge building court.', category=Travel, amount=86.13, type='Expense')
121: Transaction(date='2020-06-24', description='Week real course school everybody.', category=Health & Fitness, amount=1459.8, type='Expense')
122: Transaction(date='2020-06-30', description='Executive beyond create dinner conference.', category=Rent, amount=776.54, type='Expense')
123: Transaction(date='2020-06-30', description='Call animal approach factor want.', category=Travel, amount=98.33, type='Expense')
124: Transaction(date='2020-07-02', description='Their off.', category=Entertainment, amount=734.8, type='Expense')
125: Transaction(date='2020-07-03', description='Key continue anything wait.', category=Other, amount=3071.0, type='Income')
126: Transaction(date='2020-07-04', description='State husband.', category=Rent, amount=1442.31, type='Expense')
127: Transaction(date='2020-07-05', description='Picture approach chance none fight.', category=Travel, amount=1337.12, type='Expense')
128: Transaction(date='2020-07-06', description='Newspaper indicate other.', category=Travel, amount=1113.28, type='Expense')
129: Transaction(date='2020-07-06', description='Herself training father open investment.', category=Other, amount=1925.0, type='Income')
130: Transaction(date='2020-07-08', description='Art whether between several personal.', category=Travel, amount=818.09, type='Expense')
131: Transaction(date='2020-07-11', description='Church community.', category=Travel, amount=580.96, type='Expense')
132: Transaction(date='2020-07-12', description='Able late order.', category=Health & Fitness, amount=935.33, type='Expense')
133: Transaction(date='2020-07-12', description='Discuss religious.', category=Entertainment, amount=1027.24, type='Expense')
134: Transaction(date='2020-07-13', description='Reach under.', category=Utilities, amount=1321.11, type='Expense')
135: Transaction(date='2020-07-17', description='Boy different chance enter.', category=Health & Fitness, amount=195.84, type='Expense')
136: Transaction(date='2020-07-20', description='Arrive society.', category=Entertainment, amount=1348.86, type='Expense')
137: Transaction(date='2020-07-21', description='Keep light fight.', category=Entertainment, amount=303.23, type='Expense')
138: Transaction(date='2020-07-21', description='Matter management ball.', category=Other, amount=1096.0, type='Income')
139: Transaction(date='2020-07-22', description='Same page ago director.', category=Entertainment, amount=1439.51, type='Expense')
140: Transaction(date='2020-07-22', description='Week yourself public especially american.', category=Shopping, amount=495.53, type='Expense')
141: Transaction(date='2020-07-22', description='Million kind everything young.', category=Food & Drink, amount=1786.39, type='Expense')
142: Transaction(date='2020-07-23', description='Why like impact fly.', category=Rent, amount=1559.06, type='Expense')
143: Transaction(date='2020-07-24', description='Form really explain.', category=Salary, amount=109.93, type='Expense')
144: Transaction(date='2020-07-28', description='Already because education break significant.', category=Rent, amount=918.36, type='Expense')
145: Transaction(date='2020-07-28', description='Price my.', category=Health & Fitness, amount=1900.58, type='Expense')
146: Transaction(date='2020-07-29', description='Could gun.', category=Shopping, amount=1662.83, type='Expense')
147: Transaction(date='2020-07-30', description='Kind appear environment capital explain.', category=Utilities, amount=1489.87, type='Expense')
148: Transaction(date='2020-07-31', description='Ahead picture son.', category=Travel, amount=1506.04, type='Expense')
149: Transaction(date='2020-08-01', description='Nearly need behavior.', category=Travel, amount=1533.97, type='Expense')
150: Transaction(date='2020-08-01', description='Water positive child usually.', category=Entertainment, amount=556.45, type='Expense')
151: Transaction(date='2020-08-01', description='Relate indeed lot line lead.', category=Other, amount=2420.0, type='Income')
152: Transaction(date='2020-08-02', description='Course out second carry.', category=Health & Fitness, amount=1787.12, type='Expense')
153: Transaction(date='2020-08-03', description='Significant take future.', category=Investment, amount=2394.0, type='Income')
154: Transaction(date='2020-08-04', description='Star executive fear.', category=Utilities, amount=1415.77, type='Expense')
155: Transaction(date='2020-08-04', description='Your majority chance mrs generation.', category=Other, amount=3210.0, type='Income')
156: Transaction(date='2020-08-04', description='Help usually thank wonder draw.', category=Shopping, amount=133.91, type='Expense')
157: Transaction(date='2020-08-05', description='Task improve fish.', category=Shopping, amount=1810.63, type='Expense')
158: Transaction(date='2020-08-05', description='History professional star wonder.', category=Food & Drink, amount=1714.9, type='Expense')
159: Transaction(date='2020-08-06', description='Call whole.', category=Shopping, amount=959.19, type='Expense')
160: Transaction(date='2020-08-08', description='Them key.', category=Travel, amount=1509.42, type='Expense')
161: Transaction(date='2020-08-08', description='Address century that wide.', category=Shopping, amount=1081.06, type='Expense')
162: Transaction(date='2020-08-09', description='Sit enter perhaps however.', category=Entertainment, amount=981.58, type='Expense')
163: Transaction(date='2020-08-13', description='System teacher here first.', category=Salary, amount=67.76, type='Expense')
164: Transaction(date='2020-08-14', description='Along attention piece tv young.', category=Shopping, amount=1451.12, type='Expense')
165: Transaction(date='2020-08-16', description='Within set region beyond.', category=Utilities, amount=1961.64, type='Expense')
166: Transaction(date='2020-08-17', description='Tough animal someone.', category=Shopping, amount=1187.87, type='Expense')
167: Transaction(date='2020-08-18', description='Hear certainly most not.', category=Rent, amount=1289.07, type='Expense')
168: Transaction(date='2020-08-19', description='Company participant him.', category=Other, amount=1988.0, type='Income')
169: Transaction(date='2020-08-20', description='Tree central leave effect.', category=Other, amount=3181.0, type='Income')
170: Transaction(date='2020-08-21', description='Together let.', category=Health & Fitness, amount=681.63, type='Expense')
171: Transaction(date='2020-08-21', description='Citizen kid generation onto police.', category=Travel, amount=1506.5, type='Expense')
172: Transaction(date='2020-08-22', description='Lead few yourself.', category=Travel, amount=1671.47, type='Expense')
173: Transaction(date='2020-08-22', description='Down occur.', category=Food & Drink, amount=1999.82, type='Expense')
174: Transaction(date='2020-08-26', description='Yeah far within.', category=Rent, amount=1564.65, type='Expense')
175: Transaction(date='2020-08-27', description='Account ten sing.', category=Food & Drink, amount=1899.73, type='Expense')
176: Transaction(date='2020-08-27', description='Seat strategy total simply discover.', category=Investment, amount=1748.0, type='Income')
177: Transaction(date='2020-08-29', description='Pm per question.', category=Salary, amount=1342.22, type='Expense')
178: Transaction(date='2020-08-30', description='Best physical always small almost.', category=Entertainment, amount=935.42, type='Expense')
179: Transaction(date='2020-08-31', description='Bed far section.', category=Health & Fitness, amount=343.9, type='Expense')
180: Transaction(date='2020-09-01', description='Car much will.', category=Rent, amount=1557.81, type='Expense')
181: Transaction(date='2020-09-05', description='Tree store either station.', category=Other, amount=1385.0, type='Income')
182: Transaction(date='2020-09-05', description='Second full boy.', category=Travel, amount=1155.84, type='Expense')
183: Transaction(date='2020-09-06', description='Senior difficult put.', category=Shopping, amount=799.34, type='Expense')
184: Transaction(date='2020-09-07', description='Hand so add mr lawyer.', category=Rent, amount=1188.27, type='Expense')
185: Transaction(date='2020-09-08', description='Herself police he push likely.', category=Entertainment, amount=212.78, type='Expense')
186: Transaction(date='2020-09-11', description='Trip determine as statement.', category=Rent, amount=1594.49, type='Expense')
187: Transaction(date='2020-09-11', description='Bring animal also you.', category=Food & Drink, amount=700.94, type='Expense')
188: Transaction(date='2020-09-11', description='Doctor mr.', category=Health & Fitness, amount=147.22, type='Expense')
189: Transaction(date='2020-09-16', description='He we recent music.', category=Food & Drink, amount=1700.66, type='Expense')
190: Transaction(date='2020-09-17', description='Those pressure street.', category=Rent, amount=872.71, type='Expense')
191: Transaction(date='2020-09-17', description='Allow have kitchen.', category=Salary, amount=1417.54, type='Expense')
192: Transaction(date='2020-09-18', description='Between rate name within.', category=Travel, amount=1235.67, type='Expense')
193: Transaction(date='2020-09-18', description='Huge couple.', category=Salary, amount=935.1, type='Expense')
194: Transaction(date='2020-09-22', description='Begin deep police wife.', category=Travel, amount=651.39, type='Expense')
195: Transaction(date='2020-09-22', description='During call north attention.', category=Rent, amount=565.07, type='Expense')
196: Transaction(date='2020-09-22', description='Only spend do.', category=Salary, amount=1144.0, type='Expense')
197: Transaction(date='2020-09-23', description='Able teach certain candidate economy.', category=Health & Fitness, amount=67.11, type='Expense')
198: Transaction(date='2020-09-23', description='Produce ago.', category=Utilities, amount=980.22, type='Expense')
199: Transaction(date='2020-09-25', description='So draw food company.', category=Travel, amount=687.27, type='Expense')
200: Transaction(date='2020-09-27', description='Score moment second.', category=Travel, amount=1116.02, type='Expense')
201: Transaction(date='2020-09-27', description='Sing message board mean.', category=Entertainment, amount=180.37, type='Expense')
202: Transaction(date='2020-09-27', description='Feeling poor all your suggest.', category=Salary, amount=1114.8, type='Expense')
203: Transaction(date='2020-09-29', description='Make travel.', category=Salary, amount=1295.35, type='Expense')
204: Transaction(date='2020-09-30', description='Focus chance.', category=Food & Drink, amount=195.18, type='Expense')
205: Transaction(date='2020-10-01', description='Person one republican.', category=Entertainment, amount=619.35, type='Expense')
206: Transaction(date='2020-10-03', description='Most law painting between reduce.', category=Salary, amount=1111.41, type='Expense')
207: Transaction(date='2020-10-04', description='Involve thousand including.', category=Health & Fitness, amount=710.1, type='Expense')
208: Transaction(date='2020-10-07', description='Indeed believe interview professor.', category=Travel, amount=510.29, type='Expense')
209: Transaction(date='2020-10-07', description='Rather listen before.', category=Investment, amount=3084.0, type='Income')
210: Transaction(date='2020-10-07', description='It season.', category=Investment, amount=2069.0, type='Income')
211: Transaction(date='2020-10-08', description='News only no.', category=Salary, amount=560.23, type='Expense')
212: Transaction(date='2020-10-08', description='Improve them wait.', category=Travel, amount=1961.22, type='Expense')
213: Transaction(date='2020-10-10', description='Why outside.', category=Travel, amount=462.69, type='Expense')
214: Transaction(date='2020-10-10', description='Official defense.', category=Investment, amount=4875.0, type='Income')
215: Transaction(date='2020-10-10', description='Suggest glass news.', category=Investment, amount=946.0, type='Income')
216: Transaction(date='2020-10-12', description='Off southern suddenly.', category=Utilities, amount=1279.36, type='Expense')
217: Transaction(date='2020-10-14', description='Participant price.', category=Rent, amount=1746.71, type='Expense')
218: Transaction(date='2020-10-15', description='Congress reflect.', category=Salary, amount=962.64, type='Expense')
219: Transaction(date='2020-10-16', description='Go consider century price.', category=Food & Drink, amount=512.43, type='Expense')
220: Transaction(date='2020-10-16', description='Scientist will begin.', category=Rent, amount=1646.01, type='Expense')
221: Transaction(date='2020-10-16', description='Heavy law church.', category=Rent, amount=1158.27, type='Expense')
222: Transaction(date='2020-10-18', description='Since nothing be apply.', category=Utilities, amount=306.9, type='Expense')
223: Transaction(date='2020-10-19', description='Anyone bank kitchen interest parent.', category=Travel, amount=179.51, type='Expense')
224: Transaction(date='2020-10-23', description='Democratic mention generation.', category=Shopping, amount=1216.57, type='Expense')
225: Transaction(date='2020-10-24', description='Question or money.', category=Entertainment, amount=1553.63, type='Expense')
226: Transaction(date='2020-10-26', description='Determine worker.', category=Salary, amount=601.73, type='Expense')
227: Transaction(date='2020-10-27', description='Daughter war.', category=Shopping, amount=617.71, type='Expense')
228: Transaction(date='2020-10-27', description='Face build market.', category=Rent, amount=1895.7, type='Expense')
229: Transaction(date='2020-10-27', description='Can mouth discover.', category=Entertainment, amount=536.65, type='Expense')
230: Transaction(date='2020-10-28', description='Author technology amount affect.', category=Investment, amount=1923.0, type='Income')
231: Transaction(date='2020-10-29', description='Office of remember individual.', category=Utilities, amount=15.32, type='Expense')
232: Transaction(date='2020-10-30', description='Again foot soon peace.', category=Salary, amount=589.61, type='Expense')
233: Transaction(date='2020-10-31', description='Join black.', category=Travel, amount=1408.95, type='Expense')
234: Transaction(date='2020-11-01', description='Hundred bad that second develop.', category=Entertainment, amount=1848.58, type='Expense')
235: Transaction(date='2020-11-01', description='Member box.', category=Entertainment, amount=856.03, type='Expense')
236: Transaction(date='2020-11-02', description='Town glass road.', category=Utilities, amount=1817.74, type='Expense')
237: Transaction(date='2020-11-04', description='Bill eight community.', category=Rent, amount=128.68, type='Expense')
238: Transaction(date='2020-11-06', description='Concern significant.', category=Travel, amount=883.92, type='Expense')
239: Transaction(date='2020-11-07', description='Senior service large under.', category=Travel, amount=1402.47, type='Expense')
240: Transaction(date='2020-11-09', description='Follow wife firm key light.', category=Salary, amount=881.14, type='Expense')
241: Transaction(date='2020-11-10', description='Option goal avoid.', category=Shopping, amount=1471.72, type='Expense')
242: Transaction(date='2020-11-10', description='Also hard expert popular.', category=Food & Drink, amount=191.74, type='Expense')
243: Transaction(date='2020-11-12', description='Order his oil west school.', category=Food & Drink, amount=1999.15, type='Expense')
244: Transaction(date='2020-11-12', description='Training occur could painting.', category=Travel, amount=1156.73, type='Expense')
245: Transaction(date='2020-11-14', description='Whatever late specific.', category=Utilities, amount=946.3, type='Expense')
246: Transaction(date='2020-11-14', description='Mean common.', category=Travel, amount=371.16, type='Expense')
247: Transaction(date='2020-11-14', description='Yard model particular hair.', category=Salary, amount=1938.42, type='Expense')
248: Transaction(date='2020-11-14', description='Impact full our smile.', category=Shopping, amount=673.28, type='Expense')
249: Transaction(date='2020-11-15', description='Form newspaper.', category=Utilities, amount=666.31, type='Expense')
250: Transaction(date='2020-11-15', description='Edge surface who.', category=Travel, amount=1328.54, type='Expense')
251: Transaction(date='2020-11-16', description='Bank professional true financial.', category=Food & Drink, amount=915.1, type='Expense')
252: Transaction(date='2020-11-16', description='Organization visit finally.', category=Health & Fitness, amount=240.68, type='Expense')
253: Transaction(date='2020-11-17', description='Spend prove stock school.', category=Food & Drink, amount=1318.74, type='Expense')
254: Transaction(date='2020-11-20', description='Position interest.', category=Shopping, amount=117.87, type='Expense')
255: Transaction(date='2020-11-22', description='Guy congress.', category=Salary, amount=1254.56, type='Expense')
256: Transaction(date='2020-11-23', description='Way main difficult case.', category=Entertainment, amount=541.44, type='Expense')
257: Transaction(date='2020-11-25', description='Such big side.', category=Other, amount=4089.0, type='Income')
258: Transaction(date='2020-11-25', description='Product then.', category=Salary, amount=251.64, type='Expense')
259: Transaction(date='2020-11-26', description='Heavy class story side speak.', category=Entertainment, amount=1422.47, type='Expense')
260: Transaction(date='2020-11-27', description='Analysis hair rest wide.', category=Food & Drink, amount=1108.98, type='Expense')
261: Transaction(date='2020-11-27', description='Doctor dream whole six.', category=Investment, amount=4280.0, type='Income')
262: Transaction(date='2020-11-29', description='Question now key show.', category=Utilities, amount=1001.77, type='Expense')
263: Transaction(date='2020-12-01', description='Month whom stage soon.', category=Travel, amount=836.83, type='Expense')
264: Transaction(date='2020-12-05', description='Appear lose he.', category=Salary, amount=495.02, type='Expense')
265: Transaction(date='2020-12-05', description='Surface expect several evening town.', category=Shopping, amount=389.31, type='Expense')
266: Transaction(date='2020-12-06', description='Begin treat stage this.', category=Utilities, amount=1729.41, type='Expense')
267: Transaction(date='2020-12-10', description='Apply maybe dog yes.', category=Shopping, amount=686.35, type='Expense')
268: Transaction(date='2020-12-10', description='Agreement us stuff.', category=Travel, amount=1642.89, type='Expense')
269: Transaction(date='2020-12-11', description='Case expert stop.', category=Travel, amount=1676.58, type='Expense')
270: Transaction(date='2020-12-12', description='Large accept bad eight.', category=Salary, amount=1731.87, type='Expense')
271: Transaction(date='2020-12-13', description='Vote participant institution simply down.', category=Salary, amount=696.8, type='Expense')
272: Transaction(date='2020-12-14', description='Draw police.', category=Shopping, amount=916.14, type='Expense')
273: Transaction(date='2020-12-14', description='Book toward.', category=Entertainment, amount=1961.74, type='Expense')
274: Transaction(date='2020-12-16', description='Place offer.', category=Shopping, amount=474.79, type='Expense')
275: Transaction(date='2020-12-16', description='Investment pass drive.', category=Food & Drink, amount=643.17, type='Expense')
276: Transaction(date='2020-12-16', description='Professor beat window.', category=Shopping, amount=778.3, type='Expense')
277: Transaction(date='2020-12-18', description='Name skill medical after them.', category=Utilities, amount=995.7, type='Expense')
278: Transaction(date='2020-12-18', description='Wonder effect administration small.', category=Other, amount=1322.0, type='Income')
279: Transaction(date='2020-12-20', description='Run pick sign.', category=Salary, amount=208.43, type='Expense')
280: Transaction(date='2020-12-20', description='And national spend.', category=Food & Drink, amount=1447.66, type='Expense')
281: Transaction(date='2020-12-22', description='Wonder long consider care.', category=Utilities, amount=158.91, type='Expense')
282: Transaction(date='2020-12-23', description='Fall analysis.', category=Travel, amount=683.74, type='Expense')
283: Transaction(date='2020-12-23', description='Firm financial.', category=Rent, amount=1705.01, type='Expense')
284: Transaction(date='2020-12-24', description='List already positive experience television.', category=Shopping, amount=1908.74, type='Expense')
285: Transaction(date='2020-12-25', description='Event son join hundred.', category=Salary, amount=1746.04, type='Expense')
286: Transaction(date='2020-12-26', description='Fund cost reality happen.', category=Investment, amount=2423.0, type='Income')
287: Transaction(date='2020-12-28', description='Song risk bad.', category=Travel, amount=1994.88, type='Expense')
288: Transaction(date='2020-12-30', description='Family bill foreign fast knowledge.', category=Shopping, amount=1956.89, type='Expense')
289: Transaction(date='2020-12-31', description='Ready goal amount.', category=Rent, amount=892.92, type='Expense')
290: Transaction(date='2021-01-01', description='Your ever woman way avoid.', category=Food & Drink, amount=101.52, type='Expense')
291: Transaction(date='2021-01-06', description='Message seek civil all.', category=Travel, amount=723.34, type='Expense')
292: Transaction(date='2021-01-06', description='Call door.', category=Entertainment, amount=1067.02, type='Expense')
293: Transaction(date='2021-01-08', description='Him note painting quickly.', category=Utilities, amount=348.27, type='Expense')
294: Transaction(date='2021-01-10', description='Know baby case happen.', category=Other, amount=2472.0, type='Income')
295: Transaction(date='2021-01-10', description='Economic box save.', category=Utilities, amount=472.06, type='Expense')
296: Transaction(date='2021-01-11', description='Hit no energy employee land.', category=Salary, amount=518.1, type='Expense')
297: Transaction(date='2021-01-12', description='Customer career available common.', category=Other, amount=2856.0, type='Income')
298: Transaction(date='2021-01-13', description='Specific whose.', category=Utilities, amount=156.99, type='Expense')
299: Transaction(date='2021-01-13', description='Remember nearly face feel.', category=Travel, amount=1281.49, type='Expense')
300: Transaction(date='2021-01-14', description='Soldier meeting building.', category=Travel, amount=919.1, type='Expense')
301: Transaction(date='2021-01-14', description='Door management guess.', category=Shopping, amount=1708.14, type='Expense')
302: Transaction(date='2021-01-15', description='Level work candidate this.', category=Other, amount=3440.0, type='Income')
303: Transaction(date='2021-01-15', description='Huge response long.', category=Travel, amount=53.66, type='Expense')
304: Transaction(date='2021-01-17', description='Statement available win politics.', category=Shopping, amount=556.22, type='Expense')
305: Transaction(date='2021-01-17', description='Quite general there sister.', category=Food & Drink, amount=1822.13, type='Expense')
306: Transaction(date='2021-01-19', description='Item treat area.', category=Travel, amount=1553.72, type='Expense')
307: Transaction(date='2021-01-20', description='Check clearly follow remember generation.', category=Health & Fitness, amount=445.82, type='Expense')
308: Transaction(date='2021-01-24', description='Suffer economy play.', category=Other, amount=1620.0, type='Income')
309: Transaction(date='2021-01-26', description='By field move.', category=Food & Drink, amount=624.78, type='Expense')
310: Transaction(date='2021-01-27', description='Option oil commercial.', category=Health & Fitness, amount=1467.4, type='Expense')
311: Transaction(date='2021-01-27', description='American tell ball we side.', category=Other, amount=3176.0, type='Income')
312: Transaction(date='2021-01-27', description='Short indicate police.', category=Utilities, amount=409.58, type='Expense')
313: Transaction(date='2021-01-27', description='Nation politics few each southern.', category=Health & Fitness, amount=1066.4, type='Expense')
314: Transaction(date='2021-01-29', description='Law read citizen.', category=Utilities, amount=521.37, type='Expense')
315: Transaction(date='2021-02-02', description='Less future century technology.', category=Salary, amount=1935.09, type='Expense')
316: Transaction(date='2021-02-05', description='Wish candidate have no.', category=Health & Fitness, amount=1610.42, type='Expense')
317: Transaction(date='2021-02-05', description='Letter environment easy.', category=Rent, amount=290.11, type='Expense')
318: Transaction(date='2021-02-08', description='Face his.', category=Shopping, amount=1934.87, type='Expense')
319: Transaction(date='2021-02-08', description='While total spend.', category=Health & Fitness, amount=189.7, type='Expense')
320: Transaction(date='2021-02-09', description='Couple city you level.', category=Travel, amount=1077.78, type='Expense')
321: Transaction(date='2021-02-09', description='Age war measure whom.', category=Travel, amount=1173.27, type='Expense')
322: Transaction(date='2021-02-11', description='Good anything manager think pressure.', category=Health & Fitness, amount=225.61, type='Expense')
323: Transaction(date='2021-02-11', description='Mr suggest issue.', category=Food & Drink, amount=1242.9, type='Expense')
324: Transaction(date='2021-02-13', description='Shake hand hundred now.', category=Health & Fitness, amount=1831.36, type='Expense')
325: Transaction(date='2021-02-16', description='Network available mean share.', category=Rent, amount=1274.45, type='Expense')
326: Transaction(date='2021-02-18', description='Quite budget window hour some.', category=Travel, amount=1301.99, type='Expense')
327: Transaction(date='2021-02-19', description='Voice sense current meeting matter.', category=Investment, amount=804.0, type='Income')
328: Transaction(date='2021-02-20', description='Case four.', category=Salary, amount=241.08, type='Expense')
329: Transaction(date='2021-02-21', description='Lead paper middle foreign party.', category=Utilities, amount=783.41, type='Expense')
330: Transaction(date='2021-02-23', description='Wrong school federal from.', category=Shopping, amount=1178.81, type='Expense')
331: Transaction(date='2021-02-24', description='Majority none little admit former.', category=Shopping, amount=1313.29, type='Expense')
332: Transaction(date='2021-02-27', description='On someone rise read.', category=Shopping, amount=1878.72, type='Expense')
333: Transaction(date='2021-02-28', description='Listen whose situation simply officer.', category=Health & Fitness, amount=442.37, type='Expense')
334: Transaction(date='2021-03-02', description='Lose modern return simple herself.', category=Entertainment, amount=1712.35, type='Expense')
335: Transaction(date='2021-03-02', description='Occur red.', category=Other, amount=4960.0, type='Income')
336: Transaction(date='2021-03-02', description='Kid industry.', category=Health & Fitness, amount=130.49, type='Expense')
337: Transaction(date='2021-03-04', description='Improve simple turn.', category=Rent, amount=1898.53, type='Expense')
338: Transaction(date='2021-03-05', description='Million night your.', category=Rent, amount=1328.91, type='Expense')
339: Transaction(date='2021-03-05', description='Heavy what least mouth national.', category=Food & Drink, amount=110.65, type='Expense')
340: Transaction(date='2021-03-07', description='Machine whatever everything fear walk.', category=Utilities, amount=1576.09, type='Expense')
341: Transaction(date='2021-03-07', description='Relate real major.', category=Entertainment, amount=1176.9, type='Expense')
342: Transaction(date='2021-03-08', description='Night various return explain.', category=Entertainment, amount=663.83, type='Expense')
343: Transaction(date='2021-03-09', description='Here woman stand west source.', category=Investment, amount=2771.0, type='Income')
344: Transaction(date='2021-03-09', description='Explain research.', category=Utilities, amount=1975.18, type='Expense')
345: Transaction(date='2021-03-10', description='Pretty section degree still.', category=Utilities, amount=228.75, type='Expense')
346: Transaction(date='2021-03-12', description='Receive case past only drug.', category=Utilities, amount=39.57, type='Expense')
347: Transaction(date='2021-03-15', description='Point appear including.', category=Entertainment, amount=1181.82, type='Expense')
348: Transaction(date='2021-03-15', description='Race case.', category=Other, amount=929.0, type='Income')
349: Transaction(date='2021-03-16', description='Imagine various there.', category=Other, amount=4809.0, type='Income')
350: Transaction(date='2021-03-16', description='Sense expert experience.', category=Investment, amount=4401.0, type='Income')
351: Transaction(date='2021-03-17', description='Present discussion.', category=Health & Fitness, amount=1031.34, type='Expense')
352: Transaction(date='2021-03-17', description='Individual study value.', category=Investment, amount=855.0, type='Income')
353: Transaction(date='2021-03-18', description='Small control see the.', category=Travel, amount=921.5, type='Expense')
354: Transaction(date='2021-03-19', description='Across nothing.', category=Food & Drink, amount=1997.19, type='Expense')
355: Transaction(date='2021-03-20', description='Expect writer myself management.', category=Shopping, amount=1375.33, type='Expense')
356: Transaction(date='2021-03-21', description='Life cover both class.', category=Salary, amount=156.25, type='Expense')
357: Transaction(date='2021-03-22', description='Responsibility add front.', category=Investment, amount=1038.0, type='Income')
358: Transaction(date='2021-03-22', description='Purpose cover particularly police if.', category=Other, amount=3620.0, type='Income')
359: Transaction(date='2021-03-22', description='Foreign husband believe word local.', category=Travel, amount=912.89, type='Expense')
360: Transaction(date='2021-03-23', description='End dog.', category=Rent, amount=1588.26, type='Expense')
361: Transaction(date='2021-03-23', description='Few reveal activity president.', category=Investment, amount=4023.0, type='Income')
362: Transaction(date='2021-03-24', description='Property let.', category=Entertainment, amount=833.49, type='Expense')
363: Transaction(date='2021-03-25', description='Rather type thousand show.', category=Shopping, amount=837.82, type='Expense')
364: Transaction(date='2021-03-26', description='Happen determine whatever.', category=Other, amount=3060.0, type='Income')
365: Transaction(date='2021-03-27', description='Lawyer writer health.', category=Health & Fitness, amount=1907.65, type='Expense')
366: Transaction(date='2021-03-28', description='Serious soon stay seven.', category=Salary, amount=143.6, type='Expense')
367: Transaction(date='2021-03-29', description='Land region back nor article.', category=Rent, amount=869.43, type='Expense')
368: Transaction(date='2021-03-30', description='Dark mr clearly take.', category=Health & Fitness, amount=1625.42, type='Expense')
369: Transaction(date='2021-04-01', description='Quite response.', category=Health & Fitness, amount=1343.33, type='Expense')
370: Transaction(date='2021-04-02', description='Together knowledge.', category=Shopping, amount=1737.5, type='Expense')
371: Transaction(date='2021-04-03', description='Car indeed nor next.', category=Travel, amount=1204.82, type='Expense')
372: Transaction(date='2021-04-03', description='How staff.', category=Entertainment, amount=317.91, type='Expense')
373: Transaction(date='2021-04-03', description='Authority interest red must art.', category=Rent, amount=706.73, type='Expense')
374: Transaction(date='2021-04-08', description='All important shoulder she.', category=Rent, amount=1527.59, type='Expense')
375: Transaction(date='2021-04-08', description='Year him thank trade.', category=Shopping, amount=1691.86, type='Expense')
376: Transaction(date='2021-04-09', description='Radio product.', category=Food & Drink, amount=1221.9, type='Expense')
377: Transaction(date='2021-04-11', description='President girl condition.', category=Travel, amount=67.62, type='Expense')
378: Transaction(date='2021-04-11', description='More road development about.', category=Travel, amount=1843.86, type='Expense')
379: Transaction(date='2021-04-12', description='Leg rest interest.', category=Utilities, amount=1741.03, type='Expense')
380: Transaction(date='2021-04-13', description='Easy get every.', category=Shopping, amount=148.47, type='Expense')
381: Transaction(date='2021-04-18', description='Special information.', category=Food & Drink, amount=192.6, type='Expense')
382: Transaction(date='2021-04-19', description='Deal beyond almost.', category=Shopping, amount=845.47, type='Expense')
383: Transaction(date='2021-04-19', description='Scientist i doctor.', category=Health & Fitness, amount=630.16, type='Expense')
384: Transaction(date='2021-04-19', description='Cell year.', category=Rent, amount=1767.85, type='Expense')
385: Transaction(date='2021-04-19', description='Five our pull fly few.', category=Investment, amount=1168.0, type='Income')
386: Transaction(date='2021-04-21', description='Determine feel article we they.', category=Shopping, amount=976.47, type='Expense')
387: Transaction(date='2021-04-23', description='Quickly new another general poor.', category=Travel, amount=438.98, type='Expense')
388: Transaction(date='2021-04-23', description='Five indicate chance heavy senior.', category=Health & Fitness, amount=865.53, type='Expense')
389: Transaction(date='2021-04-24', description='Support feeling remain.', category=Salary, amount=1058.6, type='Expense')
390: Transaction(date='2021-04-27', description='None whose.', category=Entertainment, amount=796.57, type='Expense')
391: Transaction(date='2021-04-27', description='Care drug data position two.', category=Food & Drink, amount=416.8, type='Expense')
392: Transaction(date='2021-04-27', description='Movie stop.', category=Utilities, amount=1530.68, type='Expense')
393: Transaction(date='2021-04-30', description='Help painting always.', category=Rent, amount=25.38, type='Expense')
394: Transaction(date='2021-05-02', description='Source onto provide.', category=Utilities, amount=267.08, type='Expense')
395: Transaction(date='2021-05-04', description='Decade trade field training.', category=Entertainment, amount=1005.52, type='Expense')
396: Transaction(date='2021-05-04', description='While bring yourself.', category=Health & Fitness, amount=153.43, type='Expense')
397: Transaction(date='2021-05-06', description='Receive region however.', category=Food & Drink, amount=878.08, type='Expense')
398: Transaction(date='2021-05-07', description='Focus executive letter possible final.', category=Rent, amount=1725.6, type='Expense')
399: Transaction(date='2021-05-07', description='Third letter sort reveal seven.', category=Shopping, amount=1421.9, type='Expense')
400: Transaction(date='2021-05-07', description='Have including none.', category=Rent, amount=816.01, type='Expense')
401: Transaction(date='2021-05-10', description='Administration network once.', category=Utilities, amount=1604.63, type='Expense')
402: Transaction(date='2021-05-11', description='Charge form here.', category=Health & Fitness, amount=185.38, type='Expense')
403: Transaction(date='2021-05-11', description='Nearly want front shake.', category=Entertainment, amount=877.0, type='Expense')
404: Transaction(date='2021-05-12', description='Leave run goal cause.', category=Rent, amount=797.58, type='Expense')
405: Transaction(date='2021-05-14', description='Contain threat wrong whatever.', category=Health & Fitness, amount=450.93, type='Expense')
406: Transaction(date='2021-05-14', description='Stuff avoid.', category=Rent, amount=1025.86, type='Expense')
407: Transaction(date='2021-05-15', description='Include successful.', category=Investment, amount=3362.0, type='Income')
408: Transaction(date='2021-05-17', description='Religious across.', category=Utilities, amount=480.18, type='Expense')
409: Transaction(date='2021-05-19', description='Health pay project pattern.', category=Investment, amount=1921.0, type='Income')
410: Transaction(date='2021-05-22', description='Administration mother in admit.', category=Rent, amount=362.52, type='Expense')
411: Transaction(date='2021-05-23', description='Within admit spend purpose south.', category=Salary, amount=933.27, type='Expense')
412: Transaction(date='2021-05-24', description='Truth modern.', category=Salary, amount=1365.46, type='Expense')
413: Transaction(date='2021-05-24', description='People education great.', category=Health & Fitness, amount=1729.85, type='Expense')
414: Transaction(date='2021-05-24', description='Health image.', category=Utilities, amount=885.13, type='Expense')
415: Transaction(date='2021-05-25', description='Wrong difficult range.', category=Travel, amount=1594.7, type='Expense')
416: Transaction(date='2021-05-25', description='Behavior here.', category=Health & Fitness, amount=1019.6, type='Expense')
417: Transaction(date='2021-05-27', description='Argue act commercial matter.', category=Salary, amount=84.84, type='Expense')
418: Transaction(date='2021-05-27', description='Not teach believe month.', category=Travel, amount=162.69, type='Expense')
419: Transaction(date='2021-05-29', description='Deep test thought.', category=Rent, amount=1233.87, type='Expense')
420: Transaction(date='2021-05-30', description='Another avoid understand.', category=Salary, amount=1164.86, type='Expense')
421: Transaction(date='2021-05-31', description='Probably conference sure administration debate.', category=Food & Drink, amount=904.97, type='Expense')
422: Transaction(date='2021-05-31', description='Up federal nor note.', category=Entertainment, amount=649.84, type='Expense')
423: Transaction(date='2021-06-02', description='Forward sense cause write right.', category=Utilities, amount=1915.87, type='Expense')
424: Transaction(date='2021-06-02', description='Approach join stuff future.', category=Health & Fitness, amount=1964.95, type='Expense')
425: Transaction(date='2021-06-03', description='Ahead event yeah make.', category=Investment, amount=820.0, type='Income')
426: Transaction(date='2021-06-06', description='Wait it quickly produce beat.', category=Salary, amount=1982.06, type='Expense')
427: Transaction(date='2021-06-06', description='Require bank child.', category=Utilities, amount=734.16, type='Expense')
428: Transaction(date='2021-06-06', description='First blood accept final.', category=Shopping, amount=823.45, type='Expense')
429: Transaction(date='2021-06-09', description='Bill beautiful issue.', category=Food & Drink, amount=1580.56, type='Expense')
430: Transaction(date='2021-06-11', description='Mention billion bed never others.', category=Rent, amount=666.08, type='Expense')
431: Transaction(date='2021-06-11', description='Activity including single right.', category=Shopping, amount=575.45, type='Expense')
432: Transaction(date='2021-06-14', description='Occur college herself.', category=Utilities, amount=673.39, type='Expense')
433: Transaction(date='2021-06-15', description='Feeling manage as.', category=Utilities, amount=1834.33, type='Expense')
434: Transaction(date='2021-06-16', description='Inside machine baby.', category=Shopping, amount=266.61, type='Expense')
435: Transaction(date='2021-06-20', description='Result many pattern.', category=Rent, amount=626.07, type='Expense')
436: Transaction(date='2021-06-20', description='Career according.', category=Health & Fitness, amount=1628.24, type='Expense')
437: Transaction(date='2021-06-22', description='Success here television key.', category=Rent, amount=1295.59, type='Expense')
438: Transaction(date='2021-06-23', description='Carry fly water cut fire.', category=Health & Fitness, amount=46.28, type='Expense')
439: Transaction(date='2021-06-24', description='Fact reason make office.', category=Entertainment, amount=690.05, type='Expense')
440: Transaction(date='2021-06-26', description='Heavy staff hotel box.', category=Entertainment, amount=460.82, type='Expense')
441: Transaction(date='2021-06-27', description='Or nation race.', category=Investment, amount=2923.0, type='Income')
442: Transaction(date='2021-06-28', description='Well left.', category=Rent, amount=1020.3, type='Expense')
443: Transaction(date='2021-06-28', description='Something story attorney summer.', category=Food & Drink, amount=1327.24, type='Expense')
444: Transaction(date='2021-06-28', description='Cause field education child.', category=Utilities, amount=1198.52, type='Expense')
445: Transaction(date='2021-06-28', description='Available wrong look husband.', category=Investment, amount=1856.0, type='Income')
446: Transaction(date='2021-06-29', description='Myself so growth time.', category=Food & Drink, amount=827.61, type='Expense')
447: Transaction(date='2021-07-04', description='Point be shoulder price great.', category=Entertainment, amount=1939.48, type='Expense')
448: Transaction(date='2021-07-04', description='Worker quality hundred sell whole.', category=Salary, amount=475.67, type='Expense')
449: Transaction(date='2021-07-04', description='Action else.', category=Salary, amount=1808.91, type='Expense')
450: Transaction(date='2021-07-06', description='Newspaper reach determine hear leg.', category=Other, amount=4282.0, type='Income')
451: Transaction(date='2021-07-07', description='General agency bit.', category=Shopping, amount=1010.78, type='Expense')
452: Transaction(date='2021-07-09', description='Break wait.', category=Utilities, amount=1635.61, type='Expense')
453: Transaction(date='2021-07-09', description='Find lay deal lot.', category=Utilities, amount=1747.27, type='Expense')
454: Transaction(date='2021-07-10', description='Just recent.', category=Other, amount=3540.0, type='Income')
455: Transaction(date='2021-07-10', description='Feel special.', category=Rent, amount=1425.69, type='Expense')
456: Transaction(date='2021-07-11', description='Support possible.', category=Rent, amount=577.28, type='Expense')
457: Transaction(date='2021-07-11', description='Might necessary former.', category=Travel, amount=904.15, type='Expense')
458: Transaction(date='2021-07-11', description='Discussion if continue policy.', category=Shopping, amount=192.51, type='Expense')
459: Transaction(date='2021-07-12', description='Ahead pass culture or.', category=Salary, amount=1770.41, type='Expense')
460: Transaction(date='2021-07-13', description='Relate day.', category=Shopping, amount=115.98, type='Expense')
461: Transaction(date='2021-07-14', description='Big the cost for leader.', category=Entertainment, amount=778.47, type='Expense')
462: Transaction(date='2021-07-15', description='Month police others particularly.', category=Investment, amount=730.0, type='Income')
463: Transaction(date='2021-07-19', description='Girl suddenly.', category=Rent, amount=1679.96, type='Expense')
464: Transaction(date='2021-07-19', description='Try opportunity public.', category=Utilities, amount=283.82, type='Expense')
465: Transaction(date='2021-07-23', description='Draw bring health.', category=Salary, amount=1395.18, type='Expense')
466: Transaction(date='2021-07-23', description='Home others offer institution main.', category=Other, amount=4174.0, type='Income')
467: Transaction(date='2021-07-24', description='Never majority cell.', category=Rent, amount=47.82, type='Expense')
468: Transaction(date='2021-07-28', description='People tend weight machine.', category=Utilities, amount=1102.12, type='Expense')
469: Transaction(date='2021-07-29', description='White measure manager range.', category=Shopping, amount=231.26, type='Expense')
470: Transaction(date='2021-08-02', description='Major bar air set.', category=Travel, amount=252.48, type='Expense')
471: Transaction(date='2021-08-06', description='Ever mrs.', category=Salary, amount=134.99, type='Expense')
472: Transaction(date='2021-08-07', description='Not significant manager member.', category=Salary, amount=1197.51, type='Expense')
473: Transaction(date='2021-08-07', description='Environmental myself.', category=Investment, amount=1676.0, type='Income')
474: Transaction(date='2021-08-08', description='State among rest national wrong.', category=Food & Drink, amount=1233.47, type='Expense')
475: Transaction(date='2021-08-09', description='Fear yourself last.', category=Shopping, amount=382.75, type='Expense')
476: Transaction(date='2021-08-11', description='Keep yes.', category=Health & Fitness, amount=144.68, type='Expense')
477: Transaction(date='2021-08-13', description='Specific onto go.', category=Food & Drink, amount=787.0, type='Expense')
478: Transaction(date='2021-08-13', description='Pattern size spend.', category=Shopping, amount=1962.27, type='Expense')
479: Transaction(date='2021-08-15', description='Go exactly much food region.', category=Food & Drink, amount=720.62, type='Expense')
480: Transaction(date='2021-08-16', description='Environment teach world quickly believe.', category=Investment, amount=1349.0, type='Income')
481: Transaction(date='2021-08-18', description='Yeah term.', category=Health & Fitness, amount=275.49, type='Expense')
482: Transaction(date='2021-08-20', description='Economic represent stock.', category=Health & Fitness, amount=1628.88, type='Expense')
483: Transaction(date='2021-08-21', description='Central plan physical heart.', category=Other, amount=4420.0, type='Income')
484: Transaction(date='2021-08-21', description='Idea seek.', category=Utilities, amount=135.57, type='Expense')
485: Transaction(date='2021-08-22', description='Wear force able similar.', category=Salary, amount=83.6, type='Expense')
486: Transaction(date='2021-08-23', description='Play recently sure somebody.', category=Entertainment, amount=1772.72, type='Expense')
487: Transaction(date='2021-08-24', description='Why station manager.', category=Travel, amount=1035.61, type='Expense')
488: Transaction(date='2021-08-25', description='Front official not.', category=Salary, amount=514.09, type='Expense')
489: Transaction(date='2021-08-27', description='Thus several animal phone.', category=Entertainment, amount=579.36, type='Expense')
490: Transaction(date='2021-08-29', description='Buy tax.', category=Food & Drink, amount=1732.94, type='Expense')
491: Transaction(date='2021-09-05', description='A when dark read.', category=Rent, amount=1600.69, type='Expense')
492: Transaction(date='2021-09-06', description='City usually ever.', category=Shopping, amount=1489.18, type='Expense')
493: Transaction(date='2021-09-09', description='Bar professor power.', category=Shopping, amount=684.68, type='Expense')
494: Transaction(date='2021-09-09', description='Personal rise sense investment.', category=Other, amount=3509.0, type='Income')
495: Transaction(date='2021-09-09', description='Ability live.', category=Travel, amount=1604.28, type='Expense')
496: Transaction(date='2021-09-10', description='Difficult quite act receive stage.', category=Rent, amount=866.91, type='Expense')
497: Transaction(date='2021-09-11', description='Local director become.', category=Utilities, amount=1095.49, type='Expense')
498: Transaction(date='2021-09-11', description='Finish thing.', category=Rent, amount=662.56, type='Expense')
499: Transaction(date='2021-09-11', description='Similar send card year.', category=Salary, amount=1209.22, type='Expense')
500: Transaction(date='2021-09-11', description='Most across.', category=Salary, amount=709.58, type='Expense')
501: Transaction(date='2021-09-15', description='Goal defense security turn.', category=Other, amount=4063.0, type='Income')
502: Transaction(date='2021-09-16', description='Beautiful beyond sell.', category=Food & Drink, amount=159.33, type='Expense')
503: Transaction(date='2021-09-16', description='White fire cause everybody.', category=Health & Fitness, amount=1031.04, type='Expense')
504: Transaction(date='2021-09-17', description='Network outside parent.', category=Utilities, amount=1906.05, type='Expense')
505: Transaction(date='2021-09-17', description='Day parent country.', category=Other, amount=785.0, type='Income')
506: Transaction(date='2021-09-19', description='Position six soldier.', category=Investment, amount=3500.0, type='Income')
507: Transaction(date='2021-09-19', description='History drop point audience.', category=Food & Drink, amount=1214.14, type='Expense')
508: Transaction(date='2021-09-19', description='Value car team nothing.', category=Health & Fitness, amount=750.31, type='Expense')
509: Transaction(date='2021-09-19', description='Raise several west side admit.', category=Utilities, amount=1063.63, type='Expense')
510: Transaction(date='2021-09-20', description='Energy always.', category=Travel, amount=506.73, type='Expense')
511: Transaction(date='2021-09-21', description='Media pick may indeed.', category=Investment, amount=4588.0, type='Income')
512: Transaction(date='2021-09-28', description='Policy mrs.', category=Rent, amount=530.94, type='Expense')
513: Transaction(date='2021-10-01', description='Base bad tv.', category=Salary, amount=437.0, type='Expense')
514: Transaction(date='2021-10-07', description='Force only shake power poor.', category=Salary, amount=408.35, type='Expense')
515: Transaction(date='2021-10-08', description='Role suffer around beat.', category=Investment, amount=4203.0, type='Income')
516: Transaction(date='2021-10-13', description='Law participant.', category=Other, amount=1219.0, type='Income')
517: Transaction(date='2021-10-13', description='Budget remain reduce.', category=Health & Fitness, amount=1339.0, type='Expense')
518: Transaction(date='2021-10-13', description='Well mission situation result.', category=Travel, amount=1783.75, type='Expense')
519: Transaction(date='2021-10-16', description='Camera sit environmental though.', category=Utilities, amount=1589.46, type='Expense')
520: Transaction(date='2021-10-20', description='Southern evening ask republican.', category=Rent, amount=1970.41, type='Expense')
521: Transaction(date='2021-10-21', description='Baby lawyer growth matter note.', category=Entertainment, amount=1582.84, type='Expense')
522: Transaction(date='2021-10-22', description='Effort bill off.', category=Other, amount=4328.0, type='Income')
523: Transaction(date='2021-10-24', description='Scientist able teach.', category=Utilities, amount=1227.78, type='Expense')
524: Transaction(date='2021-10-24', description='Animal hair turn.', category=Other, amount=3761.0, type='Income')
525: Transaction(date='2021-10-27', description='Need trial.', category=Salary, amount=1056.13, type='Expense')
526: Transaction(date='2021-10-27', description='Focus indicate result.', category=Rent, amount=259.4, type='Expense')
527: Transaction(date='2021-10-30', description='Number budget.', category=Rent, amount=905.45, type='Expense')
528: Transaction(date='2021-11-03', description='Development middle animal.', category=Health & Fitness, amount=265.39, type='Expense')
529: Transaction(date='2021-11-04', description='Theory tend.', category=Utilities, amount=1537.89, type='Expense')
530: Transaction(date='2021-11-05', description='Beat lead hard factor.', category=Investment, amount=1517.0, type='Income')
531: Transaction(date='2021-11-05', description='Drug happy will young simply.', category=Travel, amount=337.67, type='Expense')
532: Transaction(date='2021-11-06', description='Character usually agency.', category=Entertainment, amount=698.59, type='Expense')
533: Transaction(date='2021-11-07', description='Player really act friend.', category=Travel, amount=1693.41, type='Expense')
534: Transaction(date='2021-11-10', description='Billion morning draw.', category=Travel, amount=259.09, type='Expense')
535: Transaction(date='2021-11-15', description='Win central.', category=Rent, amount=1010.12, type='Expense')
536: Transaction(date='2021-11-16', description='Analysis discuss.', category=Investment, amount=1902.0, type='Income')
537: Transaction(date='2021-11-17', description='Red head window be.', category=Health & Fitness, amount=1686.63, type='Expense')
538: Transaction(date='2021-11-18', description='Card star avoid nothing.', category=Food & Drink, amount=171.63, type='Expense')
539: Transaction(date='2021-11-18', description='Door end fire.', category=Travel, amount=1305.75, type='Expense')
540: Transaction(date='2021-11-19', description='Fish evening avoid dark.', category=Shopping, amount=1240.04, type='Expense')
541: Transaction(date='2021-11-21', description='Choice clearly.', category=Other, amount=2974.0, type='Income')
542: Transaction(date='2021-11-21', description='Image movie who debate.', category=Shopping, amount=601.89, type='Expense')
543: Transaction(date='2021-11-21', description='Enter build citizen rather.', category=Investment, amount=1794.0, type='Income')
544: Transaction(date='2021-11-23', description='Structure foreign before.', category=Salary, amount=934.24, type='Expense')
545: Transaction(date='2021-11-26', description='How any federal star.', category=Utilities, amount=632.04, type='Expense')
546: Transaction(date='2021-11-27', description='Weight take.', category=Health & Fitness, amount=1950.99, type='Expense')
547: Transaction(date='2021-11-28', description='Economic left sound cause activity.', category=Other, amount=4718.0, type='Income')
548: Transaction(date='2021-12-01', description='Cup hundred rather.', category=Health & Fitness, amount=491.06, type='Expense')
549: Transaction(date='2021-12-02', description='Only true avoid young cup.', category=Other, amount=2530.0, type='Income')
550: Transaction(date='2021-12-03', description='Begin suggest speak.', category=Investment, amount=2877.0, type='Income')
551: Transaction(date='2021-12-03', description='Walk anything.', category=Shopping, amount=504.06, type='Expense')
552: Transaction(date='2021-12-08', description='Start house ahead raise.', category=Utilities, amount=1628.82, type='Expense')
553: Transaction(date='2021-12-09', description='Democrat cost evidence third.', category=Health & Fitness, amount=387.77, type='Expense')
554: Transaction(date='2021-12-09', description='People early far include nearly.', category=Other, amount=4585.0, type='Income')
555: Transaction(date='2021-12-11', description='Evidence case.', category=Salary, amount=56.12, type='Expense')
556: Transaction(date='2021-12-12', description='Eat yes myself affect him.', category=Salary, amount=1933.71, type='Expense')
557: Transaction(date='2021-12-13', description='Name machine tree maybe.', category=Health & Fitness, amount=106.83, type='Expense')
558: Transaction(date='2021-12-13', description='Conference five despite.', category=Salary, amount=578.64, type='Expense')
559: Transaction(date='2021-12-15', description='Speak line.', category=Investment, amount=4030.0, type='Income')
560: Transaction(date='2021-12-15', description='Yourself wind beyond prevent entire.', category=Other, amount=3496.0, type='Income')
561: Transaction(date='2021-12-16', description='Offer work home.', category=Health & Fitness, amount=100.03, type='Expense')
562: Transaction(date='2021-12-19', description='Door fish scene.', category=Investment, amount=3469.0, type='Income')
563: Transaction(date='2021-12-20', description='Agreement red way nor none.', category=Rent, amount=778.83, type='Expense')
564: Transaction(date='2021-12-20', description='My compare also argue.', category=Travel, amount=1652.37, type='Expense')
565: Transaction(date='2021-12-23', description='Money a.', category=Rent, amount=266.04, type='Expense')
566: Transaction(date='2021-12-24', description='Future concern sort.', category=Other, amount=3277.0, type='Income')
567: Transaction(date='2021-12-26', description='Against only.', category=Health & Fitness, amount=1512.38, type='Expense')
568: Transaction(date='2021-12-28', description='Suffer team.', category=Other, amount=4596.0, type='Income')
569: Transaction(date='2021-12-31', description='Relate product only from.', category=Investment, amount=1622.0, type='Income')
570: Transaction(date='2022-01-01', description='Wish run join.', category=Salary, amount=1043.47, type='Expense')
571: Transaction(date='2022-01-03', description='Company why really up.', category=Investment, amount=3186.0, type='Income')
572: Transaction(date='2022-01-03', description='Program sport her.', category=Health & Fitness, amount=333.38, type='Expense')
573: Transaction(date='2022-01-03', description='Concern ability reduce.', category=Travel, amount=1191.51, type='Expense')
574: Transaction(date='2022-01-04', description='Surface side nothing consider.', category=Salary, amount=1420.39, type='Expense')
575: Transaction(date='2022-01-05', description='Determine not view cell seat.', category=Food & Drink, amount=743.03, type='Expense')
576: Transaction(date='2022-01-06', description='Word collection those become.', category=Shopping, amount=1171.57, type='Expense')
577: Transaction(date='2022-01-08', description='Activity decide really go.', category=Food & Drink, amount=1197.85, type='Expense')
578: Transaction(date='2022-01-09', description='Answer just.', category=Entertainment, amount=1445.59, type='Expense')
579: Transaction(date='2022-01-13', description='This manage her.', category=Utilities, amount=1053.22, type='Expense')
580: Transaction(date='2022-01-13', description='Big look tell prepare.', category=Shopping, amount=304.07, type='Expense')
581: Transaction(date='2022-01-14', description='Later son almost.', category=Food & Drink, amount=1149.41, type='Expense')
582: Transaction(date='2022-01-16', description='End bit game thousand.', category=Entertainment, amount=47.69, type='Expense')
583: Transaction(date='2022-01-16', description='Natural together.', category=Utilities, amount=831.79, type='Expense')
584: Transaction(date='2022-01-18', description='Order party lot all.', category=Salary, amount=1746.91, type='Expense')
585: Transaction(date='2022-01-18', description='Threat church big day couple.', category=Food & Drink, amount=1414.49, type='Expense')
586: Transaction(date='2022-01-21', description='Enter example down anyone occur.', category=Health & Fitness, amount=1996.26, type='Expense')
587: Transaction(date='2022-01-21', description='Approach about because.', category=Shopping, amount=634.37, type='Expense')
588: Transaction(date='2022-01-22', description='Best deal point.', category=Other, amount=4975.0, type='Income')
589: Transaction(date='2022-01-23', description='Such final however major.', category=Travel, amount=1224.05, type='Expense')
590: Transaction(date='2022-01-24', description='Politics hold.', category=Investment, amount=2785.0, type='Income')
591: Transaction(date='2022-01-28', description='Hard knowledge enjoy let.', category=Entertainment, amount=1883.56, type='Expense')
592: Transaction(date='2022-02-01', description='Nor character recent.', category=Travel, amount=1412.75, type='Expense')
593: Transaction(date='2022-02-02', description='Property space.', category=Salary, amount=640.91, type='Expense')
594: Transaction(date='2022-02-05', description='Dinner article hit mission whole.', category=Travel, amount=582.7, type='Expense')
595: Transaction(date='2022-02-05', description='Think live bad total research.', category=Shopping, amount=1792.56, type='Expense')
596: Transaction(date='2022-02-06', description='Too song quality per build.', category=Health & Fitness, amount=1909.67, type='Expense')
597: Transaction(date='2022-02-08', description='We cold deep.', category=Travel, amount=93.71, type='Expense')
598: Transaction(date='2022-02-08', description='Amount bad raise.', category=Rent, amount=699.26, type='Expense')
599: Transaction(date='2022-02-09', description='Challenge organization floor road southern.', category=Travel, amount=1497.2, type='Expense')
600: Transaction(date='2022-02-10', description='News need task mind.', category=Travel, amount=1866.72, type='Expense')
601: Transaction(date='2022-02-10', description='Onto site both change note.', category=Utilities, amount=227.36, type='Expense')
602: Transaction(date='2022-02-10', description='Understand none.', category=Entertainment, amount=492.49, type='Expense')
603: Transaction(date='2022-02-10', description='Test they there.', category=Entertainment, amount=1279.51, type='Expense')
604: Transaction(date='2022-02-11', description='Pressure occur list common.', category=Other, amount=3205.0, type='Income')
605: Transaction(date='2022-02-14', description='Peace race i.', category=Rent, amount=1365.57, type='Expense')
606: Transaction(date='2022-02-17', description='Real any little environmental.', category=Investment, amount=3833.0, type='Income')
607: Transaction(date='2022-02-17', description='Full soldier.', category=Salary, amount=960.12, type='Expense')
608: Transaction(date='2022-02-17', description='Successful teach range.', category=Other, amount=3131.0, type='Income')
609: Transaction(date='2022-02-17', description='Feel season similar.', category=Food & Drink, amount=1816.79, type='Expense')
610: Transaction(date='2022-02-18', description='Rock painting become.', category=Investment, amount=4883.0, type='Income')
611: Transaction(date='2022-02-21', description='Tree imagine machine.', category=Food & Drink, amount=1827.33, type='Expense')
612: Transaction(date='2022-02-22', description='Start middle city find.', category=Investment, amount=797.0, type='Income')
613: Transaction(date='2022-02-22', description='Career surface these ahead.', category=Shopping, amount=1578.04, type='Expense')
614: Transaction(date='2022-02-25', description='Student economic lead.', category=Other, amount=806.0, type='Income')
615: Transaction(date='2022-02-27', description='Ever statement measure.', category=Investment, amount=3244.0, type='Income')
616: Transaction(date='2022-03-01', description='Drop major.', category=Salary, amount=1779.63, type='Expense')
617: Transaction(date='2022-03-02', description='Whether listen necessary.', category=Other, amount=2735.0, type='Income')
618: Transaction(date='2022-03-04', description='Study else store.', category=Investment, amount=3208.0, type='Income')
619: Transaction(date='2022-03-05', description='Agree anyone take.', category=Other, amount=2607.0, type='Income')
620: Transaction(date='2022-03-07', description='Positive question especially.', category=Food & Drink, amount=1475.87, type='Expense')
621: Transaction(date='2022-03-07', description='After sister should across treatment.', category=Entertainment, amount=359.23, type='Expense')
622: Transaction(date='2022-03-12', description='Song difference part wall require.', category=Shopping, amount=1655.71, type='Expense')
623: Transaction(date='2022-03-14', description='Time surface learn design.', category=Rent, amount=807.66, type='Expense')
624: Transaction(date='2022-03-17', description='Station however bill memory production.', category=Investment, amount=4411.0, type='Income')
625: Transaction(date='2022-03-17', description='Positive participant fear blue as.', category=Entertainment, amount=23.59, type='Expense')
626: Transaction(date='2022-03-18', description='Technology end represent throughout.', category=Salary, amount=1509.98, type='Expense')
627: Transaction(date='2022-03-18', description='Change she tonight.', category=Travel, amount=1169.1, type='Expense')
628: Transaction(date='2022-03-18', description='Race return pretty young.', category=Travel, amount=1310.83, type='Expense')
629: Transaction(date='2022-03-20', description='Identify say on yes.', category=Food & Drink, amount=1338.89, type='Expense')
630: Transaction(date='2022-03-20', description='Risk develop movement size.', category=Utilities, amount=816.78, type='Expense')
631: Transaction(date='2022-03-22', description='Feel color price next.', category=Travel, amount=1383.68, type='Expense')
632: Transaction(date='2022-03-23', description='Likely scientist.', category=Travel, amount=741.41, type='Expense')
633: Transaction(date='2022-03-23', description='Water tonight pressure.', category=Utilities, amount=961.58, type='Expense')
634: Transaction(date='2022-03-24', description='Development the inside thought.', category=Health & Fitness, amount=837.53, type='Expense')
635: Transaction(date='2022-03-25', description='One less brother.', category=Salary, amount=1517.13, type='Expense')
636: Transaction(date='2022-03-26', description='Oil per line create build.', category=Entertainment, amount=1526.1, type='Expense')
637: Transaction(date='2022-03-27', description='Fear approach take case customer.', category=Rent, amount=1055.76, type='Expense')
638: Transaction(date='2022-03-28', description='Throughout would century television congress.', category=Health & Fitness, amount=165.42, type='Expense')
639: Transaction(date='2022-03-28', description='Forward would finish around.', category=Food & Drink, amount=1659.26, type='Expense')
640: Transaction(date='2022-03-28', description='Decide similar office.', category=Entertainment, amount=1149.68, type='Expense')
641: Transaction(date='2022-03-28', description='Size sister can its.', category=Other, amount=4762.0, type='Income')
642: Transaction(date='2022-03-29', description='Investment local above move.', category=Entertainment, amount=1436.67, type='Expense')
643: Transaction(date='2022-03-29', description='Throughout series.', category=Salary, amount=1619.3, type='Expense')
644: Transaction(date='2022-03-31', description='Adult above safe.', category=Shopping, amount=919.03, type='Expense')
645: Transaction(date='2022-03-31', description='Fine bill me fill.', category=Other, amount=3024.0, type='Income')
646: Transaction(date='2022-03-31', description='Fear fact.', category=Shopping, amount=516.37, type='Expense')
647: Transaction(date='2022-04-01', description='Model its rather card you.', category=Investment, amount=1098.0, type='Income')
648: Transaction(date='2022-04-01', description='Seek close trade speech.', category=Other, amount=1022.0, type='Income')
649: Transaction(date='2022-04-01', description='Difference world deep.', category=Food & Drink, amount=1869.82, type='Expense')
650: Transaction(date='2022-04-01', description='Mean but represent society heart.', category=Other, amount=759.0, type='Income')
651: Transaction(date='2022-04-02', description='Perhaps order answer blue structure.', category=Utilities, amount=1540.25, type='Expense')
652: Transaction(date='2022-04-02', description='Indicate herself.', category=Shopping, amount=755.18, type='Expense')
653: Transaction(date='2022-04-03', description='Listen check receive.', category=Salary, amount=1740.54, type='Expense')
654: Transaction(date='2022-04-03', description='Inside big minute.', category=Shopping, amount=169.7, type='Expense')
655: Transaction(date='2022-04-04', description='Store bit responsibility brother.', category=Utilities, amount=1310.53, type='Expense')
656: Transaction(date='2022-04-05', description='Marriage participant.', category=Utilities, amount=1985.77, type='Expense')
657: Transaction(date='2022-04-08', description='Win group important civil nature.', category=Utilities, amount=1458.44, type='Expense')
658: Transaction(date='2022-04-09', description='Former able public trade.', category=Food & Drink, amount=1514.8, type='Expense')
659: Transaction(date='2022-04-09', description='Chance bed memory rich.', category=Salary, amount=1350.26, type='Expense')
660: Transaction(date='2022-04-10', description='Popular lead with.', category=Shopping, amount=1648.49, type='Expense')
661: Transaction(date='2022-04-10', description='Anyone spend them system.', category=Health & Fitness, amount=318.69, type='Expense')
662: Transaction(date='2022-04-16', description='International break surface three.', category=Investment, amount=763.0, type='Income')
663: Transaction(date='2022-04-18', description='Professional color group.', category=Shopping, amount=1233.7, type='Expense')
664: Transaction(date='2022-04-18', description='Environmental run head garden box.', category=Food & Drink, amount=488.93, type='Expense')
665: Transaction(date='2022-04-21', description='Even history surface son.', category=Investment, amount=4157.0, type='Income')
666: Transaction(date='2022-04-22', description='Car risk.', category=Food & Drink, amount=497.02, type='Expense')
667: Transaction(date='2022-04-22', description='Indeed opportunity.', category=Other, amount=2419.0, type='Income')
668: Transaction(date='2022-04-24', description='Consider here grow agreement.', category=Food & Drink, amount=288.87, type='Expense')
669: Transaction(date='2022-04-24', description='Turn both in animal.', category=Entertainment, amount=1633.41, type='Expense')
670: Transaction(date='2022-04-29', description='Actually than modern season.', category=Health & Fitness, amount=1159.17, type='Expense')
671: Transaction(date='2022-04-30', description='Within prove full.', category=Health & Fitness, amount=481.44, type='Expense')
672: Transaction(date='2022-05-02', description='Believe product eat generation put.', category=Entertainment, amount=832.98, type='Expense')
673: Transaction(date='2022-05-02', description='Stuff next write.', category=Utilities, amount=1255.73, type='Expense')
674: Transaction(date='2022-05-03', description='Poor church.', category=Salary, amount=103.3, type='Expense')
675: Transaction(date='2022-05-05', description='Thank avoid man drive.', category=Shopping, amount=1573.75, type='Expense')
676: Transaction(date='2022-05-05', description='Medical nearly.', category=Shopping, amount=822.44, type='Expense')
677: Transaction(date='2022-05-06', description='Stage bad know of.', category=Utilities, amount=1511.29, type='Expense')
678: Transaction(date='2022-05-07', description='Language budget market president.', category=Entertainment, amount=608.47, type='Expense')
679: Transaction(date='2022-05-07', description='Include news a before.', category=Salary, amount=1830.33, type='Expense')
680: Transaction(date='2022-05-07', description='None pressure lay future stay.', category=Shopping, amount=1119.0, type='Expense')
681: Transaction(date='2022-05-08', description='Forward should outside respond.', category=Investment, amount=2588.0, type='Income')
682: Transaction(date='2022-05-08', description='Box service develop game.', category=Rent, amount=1845.23, type='Expense')
683: Transaction(date='2022-05-09', description='Miss reduce animal clearly cell.', category=Rent, amount=1075.91, type='Expense')
684: Transaction(date='2022-05-09', description='Effect cultural.', category=Food & Drink, amount=544.24, type='Expense')
685: Transaction(date='2022-05-11', description='Democratic station course full.', category=Food & Drink, amount=1748.5, type='Expense')
686: Transaction(date='2022-05-13', description='Unit study.', category=Investment, amount=4427.0, type='Income')
687: Transaction(date='2022-05-13', description='Leader operation mean white.', category=Health & Fitness, amount=613.34, type='Expense')
688: Transaction(date='2022-05-14', description='Save network item one under.', category=Entertainment, amount=1874.95, type='Expense')
689: Transaction(date='2022-05-19', description='Community spring billion price paper.', category=Rent, amount=1496.29, type='Expense')
690: Transaction(date='2022-05-19', description='Thing industry participant realize.', category=Entertainment, amount=1394.3, type='Expense')
691: Transaction(date='2022-05-20', description='Huge garden family.', category=Shopping, amount=1831.1, type='Expense')
692: Transaction(date='2022-05-20', description='Include record security.', category=Travel, amount=767.52, type='Expense')
693: Transaction(date='2022-05-22', description='Team republican very plan cut.', category=Health & Fitness, amount=623.76, type='Expense')
694: Transaction(date='2022-05-23', description='Tv car report window.', category=Salary, amount=937.35, type='Expense')
695: Transaction(date='2022-05-24', description='My pm report set.', category=Salary, amount=1525.64, type='Expense')
696: Transaction(date='2022-05-24', description='Old wall report purpose.', category=Entertainment, amount=747.7, type='Expense')
697: Transaction(date='2022-05-25', description='There institution actually.', category=Entertainment, amount=1485.09, type='Expense')
698: Transaction(date='2022-05-27', description='Herself apply.', category=Other, amount=2918.0, type='Income')
699: Transaction(date='2022-05-28', description='Environment size worker.', category=Other, amount=2384.0, type='Income')
700: Transaction(date='2022-05-29', description='Style real far.', category=Entertainment, amount=1003.61, type='Expense')
701: Transaction(date='2022-05-29', description='Grow laugh.', category=Health & Fitness, amount=522.08, type='Expense')
702: Transaction(date='2022-05-30', description='Score night down.', category=Salary, amount=927.05, type='Expense')
703: Transaction(date='2022-05-30', description='Unit hold.', category=Health & Fitness, amount=346.84, type='Expense')
704: Transaction(date='2022-05-31', description='Approach season data than team.', category=Salary, amount=376.26, type='Expense')
705: Transaction(date='2022-06-01', description='Treatment relate positive.', category=Food & Drink, amount=1052.65, type='Expense')
706: Transaction(date='2022-06-01', description='Cultural right cover large work.', category=Rent, amount=1890.92, type='Expense')
707: Transaction(date='2022-06-04', description='While same budget.', category=Food & Drink, amount=830.66, type='Expense')
708: Transaction(date='2022-06-04', description='Evidence enough claim suffer.', category=Entertainment, amount=145.26, type='Expense')
709: Transaction(date='2022-06-06', description='Letter visit always up.', category=Investment, amount=4642.0, type='Income')
710: Transaction(date='2022-06-07', description='Past product make of group.', category=Food & Drink, amount=1239.49, type='Expense')
711: Transaction(date='2022-06-07', description='Detail contain attention measure indeed.', category=Salary, amount=1321.98, type='Expense')
712: Transaction(date='2022-06-11', description='Begin determine degree forward.', category=Other, amount=597.0, type='Income')
713: Transaction(date='2022-06-11', description='Despite sell most.', category=Other, amount=4888.0, type='Income')
714: Transaction(date='2022-06-12', description='Boy generation.', category=Food & Drink, amount=1009.41, type='Expense')
715: Transaction(date='2022-06-12', description='Truth during always those high.', category=Shopping, amount=1614.27, type='Expense')
716: Transaction(date='2022-06-12', description='Either rich agency might.', category=Rent, amount=1836.92, type='Expense')
717: Transaction(date='2022-06-13', description='Sort knowledge analysis together court.', category=Entertainment, amount=391.94, type='Expense')
718: Transaction(date='2022-06-17', description='Way cold.', category=Health & Fitness, amount=541.98, type='Expense')
719: Transaction(date='2022-06-17', description='Account assume prove similar ten.', category=Health & Fitness, amount=1919.0, type='Expense')
720: Transaction(date='2022-06-18', description='Day stock.', category=Entertainment, amount=1980.56, type='Expense')
721: Transaction(date='2022-06-19', description='Section child.', category=Rent, amount=1310.93, type='Expense')
722: Transaction(date='2022-06-20', description='Also available concern edge.', category=Food & Drink, amount=196.07, type='Expense')
723: Transaction(date='2022-06-21', description='Start treat imagine later identify.', category=Salary, amount=121.67, type='Expense')
724: Transaction(date='2022-06-21', description='Hope brother.', category=Investment, amount=1177.0, type='Income')
725: Transaction(date='2022-06-23', description='As second animal summer group.', category=Shopping, amount=1293.37, type='Expense')
726: Transaction(date='2022-06-23', description='Face rate.', category=Rent, amount=1790.63, type='Expense')
727: Transaction(date='2022-06-25', description='She join house.', category=Entertainment, amount=430.72, type='Expense')
728: Transaction(date='2022-06-25', description='Choose detail section someone.', category=Salary, amount=549.48, type='Expense')
729: Transaction(date='2022-06-26', description='Whether century quality.', category=Travel, amount=1799.63, type='Expense')
730: Transaction(date='2022-06-26', description='Network never.', category=Food & Drink, amount=366.92, type='Expense')
731: Transaction(date='2022-06-30', description='Above others.', category=Food & Drink, amount=1923.32, type='Expense')
732: Transaction(date='2022-07-01', description='Water fight security tough gas.', category=Shopping, amount=1858.46, type='Expense')
733: Transaction(date='2022-07-02', description='Central contain pattern education boy.', category=Entertainment, amount=1137.72, type='Expense')
734: Transaction(date='2022-07-04', description='Surface usually.', category=Health & Fitness, amount=773.85, type='Expense')
735: Transaction(date='2022-07-04', description='Popular available public economy.', category=Rent, amount=1006.58, type='Expense')
736: Transaction(date='2022-07-04', description='Maintain former campaign pretty beyond.', category=Food & Drink, amount=204.39, type='Expense')
737: Transaction(date='2022-07-05', description='Fish late pattern.', category=Rent, amount=687.78, type='Expense')
738: Transaction(date='2022-07-06', description='Charge note space when collection.', category=Shopping, amount=1938.15, type='Expense')
739: Transaction(date='2022-07-13', description='Size republican political.', category=Travel, amount=1569.26, type='Expense')
740: Transaction(date='2022-07-17', description='Care run there leg.', category=Entertainment, amount=718.06, type='Expense')
741: Transaction(date='2022-07-17', description='Main board population.', category=Shopping, amount=379.62, type='Expense')
742: Transaction(date='2022-07-17', description='Teach cause mrs.', category=Shopping, amount=180.53, type='Expense')
743: Transaction(date='2022-07-18', description='Baby away affect medical all.', category=Entertainment, amount=1429.09, type='Expense')
744: Transaction(date='2022-07-18', description='If operation trade necessary myself.', category=Investment, amount=3612.0, type='Income')
745: Transaction(date='2022-07-19', description='President chance hotel through.', category=Utilities, amount=1481.91, type='Expense')
746: Transaction(date='2022-07-20', description='Let member however.', category=Rent, amount=192.13, type='Expense')
747: Transaction(date='2022-07-22', description='Production summer.', category=Salary, amount=726.24, type='Expense')
748: Transaction(date='2022-07-25', description='Cell city not not the.', category=Utilities, amount=183.72, type='Expense')
749: Transaction(date='2022-07-27', description='Voice american seven discussion.', category=Shopping, amount=215.55, type='Expense')
750: Transaction(date='2022-07-27', description='Doctor protect military social.', category=Investment, amount=3157.0, type='Income')
751: Transaction(date='2022-07-27', description='Truth thing always possible.', category=Food & Drink, amount=590.78, type='Expense')
752: Transaction(date='2022-07-27', description='Plant event score official now.', category=Rent, amount=1452.44, type='Expense')
753: Transaction(date='2022-07-28', description='Half president structure might.', category=Entertainment, amount=1148.2, type='Expense')
754: Transaction(date='2022-07-28', description='Might good test.', category=Other, amount=1888.0, type='Income')
755: Transaction(date='2022-07-29', description='Thought according think key.', category=Other, amount=2698.0, type='Income')
756: Transaction(date='2022-07-30', description='Method exactly.', category=Rent, amount=342.74, type='Expense')
757: Transaction(date='2022-07-31', description='Church own stage.', category=Travel, amount=973.62, type='Expense')
758: Transaction(date='2022-08-02', description='Specific result loss long.', category=Other, amount=2520.0, type='Income')
759: Transaction(date='2022-08-02', description='Claim street against amount of.', category=Salary, amount=1191.67, type='Expense')
760: Transaction(date='2022-08-03', description='Picture always occur.', category=Investment, amount=1609.0, type='Income')
761: Transaction(date='2022-08-04', description='Election conference when.', category=Shopping, amount=671.11, type='Expense')
762: Transaction(date='2022-08-06', description='These road.', category=Health & Fitness, amount=886.04, type='Expense')
763: Transaction(date='2022-08-07', description='Describe paper.', category=Utilities, amount=607.87, type='Expense')
764: Transaction(date='2022-08-12', description='Color store over some.', category=Entertainment, amount=963.23, type='Expense')
765: Transaction(date='2022-08-13', description='Process enough right garden law.', category=Other, amount=3114.0, type='Income')
766: Transaction(date='2022-08-15', description='Her car admit.', category=Salary, amount=1935.85, type='Expense')
767: Transaction(date='2022-08-15', description='Range step painting concern.', category=Health & Fitness, amount=758.35, type='Expense')
768: Transaction(date='2022-08-16', description='Me brother lot candidate owner.', category=Shopping, amount=736.38, type='Expense')
769: Transaction(date='2022-08-16', description='Miss have analysis morning.', category=Entertainment, amount=1188.53, type='Expense')
770: Transaction(date='2022-08-17', description='Smile high open scientist.', category=Food & Drink, amount=1906.48, type='Expense')
771: Transaction(date='2022-08-18', description='International effort.', category=Entertainment, amount=882.53, type='Expense')
772: Transaction(date='2022-08-19', description='Exist theory.', category=Rent, amount=1639.74, type='Expense')
773: Transaction(date='2022-08-21', description='Country remember smile.', category=Shopping, amount=1349.23, type='Expense')
774: Transaction(date='2022-08-21', description='Brother according pressure reach firm.', category=Other, amount=2482.0, type='Income')
775: Transaction(date='2022-08-22', description='Choice prepare.', category=Utilities, amount=1433.32, type='Expense')
776: Transaction(date='2022-08-22', description='Talk interest hand.', category=Rent, amount=903.64, type='Expense')
777: Transaction(date='2022-08-23', description='Sense including six lot.', category=Shopping, amount=1073.39, type='Expense')
778: Transaction(date='2022-08-24', description='Never effect fill general relationship.', category=Shopping, amount=1091.85, type='Expense')
779: Transaction(date='2022-08-26', description='Security land record.', category=Health & Fitness, amount=1406.25, type='Expense')
780: Transaction(date='2022-08-26', description='Democrat hundred.', category=Rent, amount=227.65, type='Expense')
781: Transaction(date='2022-08-28', description='Pass range appear home player.', category=Health & Fitness, amount=341.01, type='Expense')
782: Transaction(date='2022-08-29', description='Likely outside because.', category=Shopping, amount=1933.95, type='Expense')
783: Transaction(date='2022-08-30', description='He participant right.', category=Shopping, amount=220.83, type='Expense')
784: Transaction(date='2022-08-31', description='Prevent expert white arm similar.', category=Investment, amount=4683.0, type='Income')
785: Transaction(date='2022-08-31', description='Keep cut prevent sound.', category=Entertainment, amount=897.1, type='Expense')
786: Transaction(date='2022-09-01', description='Almost outside middle.', category=Utilities, amount=696.49, type='Expense')
787: Transaction(date='2022-09-02', description='College already dream a church.', category=Investment, amount=2920.0, type='Income')
788: Transaction(date='2022-09-03', description='Good house only military.', category=Rent, amount=240.98, type='Expense')
789: Transaction(date='2022-09-03', description='Near risk.', category=Utilities, amount=703.17, type='Expense')
790: Transaction(date='2022-09-03', description='Give white everybody paper create.', category=Entertainment, amount=1047.02, type='Expense')
791: Transaction(date='2022-09-04', description='Above company media next me.', category=Rent, amount=881.95, type='Expense')
792: Transaction(date='2022-09-04', description='Sort what.', category=Health & Fitness, amount=143.68, type='Expense')
793: Transaction(date='2022-09-05', description='Economic kind help low during.', category=Investment, amount=685.0, type='Income')
794: Transaction(date='2022-09-06', description='Race to television loss.', category=Rent, amount=1357.51, type='Expense')
795: Transaction(date='2022-09-07', description='Him small.', category=Investment, amount=4727.0, type='Income')
796: Transaction(date='2022-09-08', description='Son include good.', category=Investment, amount=3211.0, type='Income')
797: Transaction(date='2022-09-10', description='Red enjoy.', category=Salary, amount=471.39, type='Expense')
798: Transaction(date='2022-09-11', description='Human see.', category=Utilities, amount=380.91, type='Expense')
799: Transaction(date='2022-09-11', description='Never condition.', category=Shopping, amount=68.09, type='Expense')
800: Transaction(date='2022-09-12', description='Despite off consumer second.', category=Entertainment, amount=904.65, type='Expense')
801: Transaction(date='2022-09-14', description='Say dog drug enter.', category=Food & Drink, amount=1412.42, type='Expense')
802: Transaction(date='2022-09-15', description='Strong student green.', category=Rent, amount=1158.92, type='Expense')
803: Transaction(date='2022-09-15', description='Turn put professional.', category=Utilities, amount=737.93, type='Expense')
804: Transaction(date='2022-09-17', description='When summer he firm together.', category=Investment, amount=2649.0, type='Income')
805: Transaction(date='2022-09-18', description='Make worker trouble.', category=Salary, amount=1060.15, type='Expense')
806: Transaction(date='2022-09-19', description='Develop office approach son.', category=Shopping, amount=1226.5, type='Expense')
807: Transaction(date='2022-09-19', description='Allow girl team simply lead.', category=Health & Fitness, amount=710.45, type='Expense')
808: Transaction(date='2022-09-21', description='Least ready activity decision ok.', category=Utilities, amount=1629.34, type='Expense')
809: Transaction(date='2022-09-23', description='Party employee nature down.', category=Travel, amount=1838.18, type='Expense')
810: Transaction(date='2022-09-26', description='Gas contain interest industry sell.', category=Utilities, amount=633.24, type='Expense')
811: Transaction(date='2022-09-26', description='Evidence conference capital office.', category=Rent, amount=374.26, type='Expense')
812: Transaction(date='2022-09-28', description='Result sound.', category=Shopping, amount=842.3, type='Expense')
813: Transaction(date='2022-09-29', description='Task cultural son school.', category=Other, amount=3965.0, type='Income')
814: Transaction(date='2022-09-29', description='Computer matter.', category=Investment, amount=1866.0, type='Income')
815: Transaction(date='2022-09-30', description='Result size feel no nothing.', category=Utilities, amount=382.33, type='Expense')
816: Transaction(date='2022-10-01', description='Real art.', category=Food & Drink, amount=1732.39, type='Expense')
817: Transaction(date='2022-10-02', description='Stuff benefit seat employee.', category=Salary, amount=281.7, type='Expense')
818: Transaction(date='2022-10-03', description='Newspaper special general in go.', category=Salary, amount=824.18, type='Expense')
819: Transaction(date='2022-10-04', description='Office power company sister great.', category=Salary, amount=982.51, type='Expense')
820: Transaction(date='2022-10-04', description='Simple land bad party.', category=Investment, amount=2306.0, type='Income')
821: Transaction(date='2022-10-05', description='What than political up instead.', category=Travel, amount=457.44, type='Expense')
822: Transaction(date='2022-10-08', description='Support author class direction.', category=Salary, amount=1135.64, type='Expense')
823: Transaction(date='2022-10-10', description='Report painting house.', category=Salary, amount=1103.96, type='Expense')
824: Transaction(date='2022-10-12', description='Security minute late cold.', category=Investment, amount=2694.0, type='Income')
825: Transaction(date='2022-10-12', description='Road product wait.', category=Health & Fitness, amount=1088.67, type='Expense')
826: Transaction(date='2022-10-13', description='Cold billion as.', category=Salary, amount=1097.24, type='Expense')
827: Transaction(date='2022-10-19', description='Approach side model you billion.', category=Rent, amount=1474.62, type='Expense')
828: Transaction(date='2022-10-19', description='Pressure those.', category=Entertainment, amount=602.44, type='Expense')
829: Transaction(date='2022-10-20', description='Easy exist phone.', category=Salary, amount=533.0, type='Expense')
830: Transaction(date='2022-10-21', description='Stuff agreement professional team foot.', category=Rent, amount=883.75, type='Expense')
831: Transaction(date='2022-10-22', description='Window music evening know.', category=Entertainment, amount=427.66, type='Expense')
832: Transaction(date='2022-10-23', description='Opportunity lead see.', category=Health & Fitness, amount=1723.49, type='Expense')
833: Transaction(date='2022-10-25', description='Model police finally girl.', category=Shopping, amount=340.3, type='Expense')
834: Transaction(date='2022-10-28', description='Source kind hand.', category=Investment, amount=2195.0, type='Income')
835: Transaction(date='2022-10-30', description='Field a spring.', category=Health & Fitness, amount=1069.24, type='Expense')
836: Transaction(date='2022-10-31', description='Expert movement reach man.', category=Utilities, amount=654.07, type='Expense')
837: Transaction(date='2022-11-03', description='Across involve fall.', category=Travel, amount=1149.74, type='Expense')
838: Transaction(date='2022-11-04', description='Nature country.', category=Rent, amount=279.82, type='Expense')
839: Transaction(date='2022-11-05', description='House clear say hope.', category=Shopping, amount=1838.35, type='Expense')
840: Transaction(date='2022-11-07', description='Enjoy instead dream.', category=Utilities, amount=1405.84, type='Expense')
841: Transaction(date='2022-11-07', description='Possible mother especially.', category=Investment, amount=3138.0, type='Income')
842: Transaction(date='2022-11-09', description='Consumer face.', category=Shopping, amount=984.02, type='Expense')
843: Transaction(date='2022-11-10', description='Laugh red man fly charge.', category=Travel, amount=1259.35, type='Expense')
844: Transaction(date='2022-11-13', description='Control bill.', category=Rent, amount=1270.92, type='Expense')
845: Transaction(date='2022-11-14', description='Resource road do.', category=Rent, amount=811.67, type='Expense')
846: Transaction(date='2022-11-14', description='Management game.', category=Health & Fitness, amount=914.84, type='Expense')
847: Transaction(date='2022-11-15', description='Your less social form.', category=Food & Drink, amount=1739.13, type='Expense')
848: Transaction(date='2022-11-17', description='Entire hotel.', category=Rent, amount=1464.55, type='Expense')
849: Transaction(date='2022-11-17', description='Consider time economic high.', category=Health & Fitness, amount=1701.71, type='Expense')
850: Transaction(date='2022-11-18', description='Window research his.', category=Shopping, amount=443.48, type='Expense')
851: Transaction(date='2022-11-19', description='Scene almost leave.', category=Entertainment, amount=994.48, type='Expense')
852: Transaction(date='2022-11-21', description='Program decade home which view.', category=Travel, amount=682.13, type='Expense')
853: Transaction(date='2022-11-22', description='Rock seat near business loss.', category=Salary, amount=1578.72, type='Expense')
854: Transaction(date='2022-11-26', description='Growth appear project.', category=Food & Drink, amount=101.43, type='Expense')
855: Transaction(date='2022-11-26', description='Plan foot cold.', category=Salary, amount=42.4, type='Expense')
856: Transaction(date='2022-11-28', description='Build smile across among situation.', category=Utilities, amount=888.16, type='Expense')
857: Transaction(date='2022-11-28', description='Choice side eat matter rather.', category=Other, amount=2157.0, type='Income')
858: Transaction(date='2022-12-01', description='Art green child movie blue.', category=Utilities, amount=1765.47, type='Expense')
859: Transaction(date='2022-12-03', description='First next popular.', category=Travel, amount=195.75, type='Expense')
860: Transaction(date='2022-12-04', description='Run again help right.', category=Rent, amount=749.97, type='Expense')
861: Transaction(date='2022-12-05', description='Magazine join.', category=Utilities, amount=113.06, type='Expense')
862: Transaction(date='2022-12-05', description='Recent couple cost girl run.', category=Entertainment, amount=864.17, type='Expense')
863: Transaction(date='2022-12-07', description='Trial never attention choice fast.', category=Investment, amount=508.0, type='Income')
864: Transaction(date='2022-12-12', description='Happen a point here control.', category=Rent, amount=274.07, type='Expense')
865: Transaction(date='2022-12-13', description='Task simply draw return probably.', category=Salary, amount=28.48, type='Expense')
866: Transaction(date='2022-12-15', description='Couple raise she child.', category=Rent, amount=1976.43, type='Expense')
867: Transaction(date='2022-12-16', description='Important focus benefit probably.', category=Utilities, amount=960.62, type='Expense')
868: Transaction(date='2022-12-18', description='Condition all blood.', category=Rent, amount=1464.22, type='Expense')
869: Transaction(date='2022-12-22', description='All politics.', category=Other, amount=3511.0, type='Income')
870: Transaction(date='2022-12-28', description='Moment red eight hand.', category=Utilities, amount=771.61, type='Expense')
871: Transaction(date='2022-12-30', description='Us decade.', category=Utilities, amount=1286.74, type='Expense')
872: Transaction(date='2022-12-31', description='Life police.', category=Rent, amount=1065.71, type='Expense')
873: Transaction(date='2023-01-03', description='Sea three state some around.', category=Investment, amount=4105.0, type='Income')
874: Transaction(date='2023-01-04', description='American audience fight war whether.', category=Entertainment, amount=1259.88, type='Expense')
875: Transaction(date='2023-01-04', description='Rise mother.', category=Other, amount=4124.0, type='Income')
876: Transaction(date='2023-01-06', description='Moment eat personal.', category=Rent, amount=1794.22, type='Expense')
877: Transaction(date='2023-01-09', description='See indeed black police group.', category=Investment, amount=3645.0, type='Income')
878: Transaction(date='2023-01-10', description='Third coach congress husband.', category=Shopping, amount=635.86, type='Expense')
879: Transaction(date='2023-01-11', description='Move outside value.', category=Other, amount=1190.0, type='Income')
880: Transaction(date='2023-01-13', description='Yard foot he share issue.', category=Travel, amount=1560.15, type='Expense')
881: Transaction(date='2023-01-15', description='Drive nation heart nearly season.', category=Travel, amount=418.84, type='Expense')
882: Transaction(date='2023-01-16', description='Language however risk work.', category=Entertainment, amount=1498.21, type='Expense')
883: Transaction(date='2023-01-17', description='Career three according worker democrat.', category=Food & Drink, amount=1569.77, type='Expense')
884: Transaction(date='2023-01-18', description='Sign wall.', category=Entertainment, amount=994.83, type='Expense')
885: Transaction(date='2023-01-19', description='Memory number occur behind leave.', category=Other, amount=1081.0, type='Income')
886: Transaction(date='2023-01-19', description='Enter threat economic.', category=Food & Drink, amount=1973.9, type='Expense')
887: Transaction(date='2023-01-19', description='Begin hundred truth.', category=Utilities, amount=779.81, type='Expense')
888: Transaction(date='2023-01-19', description='Society common stay ready.', category=Shopping, amount=737.17, type='Expense')
889: Transaction(date='2023-01-19', description='Organization feel.', category=Salary, amount=1909.36, type='Expense')
890: Transaction(date='2023-01-20', description='Bad argue side girl.', category=Food & Drink, amount=1178.85, type='Expense')
891: Transaction(date='2023-01-21', description='Us building side section.', category=Investment, amount=826.0, type='Income')
892: Transaction(date='2023-01-21', description='Him fight american music change.', category=Shopping, amount=1018.16, type='Expense')
893: Transaction(date='2023-01-22', description='Check purpose young.', category=Shopping, amount=852.91, type='Expense')
894: Transaction(date='2023-01-23', description='Offer bed it skill challenge.', category=Utilities, amount=562.16, type='Expense')
895: Transaction(date='2023-01-23', description='Live seat.', category=Entertainment, amount=1948.88, type='Expense')
896: Transaction(date='2023-01-23', description='Energy huge its recently.', category=Other, amount=3700.0, type='Income')
897: Transaction(date='2023-01-24', description='Nice maintain present cultural.', category=Rent, amount=627.24, type='Expense')
898: Transaction(date='2023-01-24', description='Drug entire involve.', category=Entertainment, amount=125.23, type='Expense')
899: Transaction(date='2023-01-24', description='Both including.', category=Other, amount=1450.0, type='Income')
900: Transaction(date='2023-01-24', description='Couple general pretty show.', category=Food & Drink, amount=625.49, type='Expense')
901: Transaction(date='2023-01-24', description='Pattern wrong kind certainly.', category=Utilities, amount=247.69, type='Expense')
902: Transaction(date='2023-01-24', description='Current health serious.', category=Food & Drink, amount=336.03, type='Expense')
903: Transaction(date='2023-01-25', description='Science drug.', category=Travel, amount=342.26, type='Expense')
904: Transaction(date='2023-01-26', description='Add really born.', category=Other, amount=1467.0, type='Income')
905: Transaction(date='2023-01-27', description='Age decade mrs data.', category=Food & Drink, amount=676.67, type='Expense')
906: Transaction(date='2023-01-27', description='Nature performance yeah.', category=Rent, amount=533.27, type='Expense')
907: Transaction(date='2023-01-28', description='On professional analysis color.', category=Rent, amount=997.33, type='Expense')
908: Transaction(date='2023-01-29', description='Smile peace apply.', category=Food & Drink, amount=1004.22, type='Expense')
909: Transaction(date='2023-01-29', description='Home even.', category=Health & Fitness, amount=323.05, type='Expense')
910: Transaction(date='2023-01-30', description='Affect test.', category=Rent, amount=464.52, type='Expense')
911: Transaction(date='2023-01-31', description='Woman meet report.', category=Food & Drink, amount=1718.79, type='Expense')
912: Transaction(date='2023-01-31', description='Term mother cost may.', category=Investment, amount=4364.0, type='Income')
913: Transaction(date='2023-02-01', description='Unit again easy morning four.', category=Utilities, amount=366.13, type='Expense')
914: Transaction(date='2023-02-02', description='Adult range green.', category=Food & Drink, amount=1110.2, type='Expense')
915: Transaction(date='2023-02-03', description='You without every.', category=Shopping, amount=456.79, type='Expense')
916: Transaction(date='2023-02-06', description='Election hit yourself wrong want.', category=Health & Fitness, amount=549.8, type='Expense')
917: Transaction(date='2023-02-07', description='Charge today prevent.', category=Other, amount=1478.0, type='Income')
918: Transaction(date='2023-02-07', description='Minute question their rather.', category=Food & Drink, amount=366.9, type='Expense')
919: Transaction(date='2023-02-08', description='Of interview fine.', category=Food & Drink, amount=1908.62, type='Expense')
920: Transaction(date='2023-02-08', description='My deal catch must.', category=Other, amount=2812.0, type='Income')
921: Transaction(date='2023-02-09', description='Finally relate model.', category=Travel, amount=661.33, type='Expense')
922: Transaction(date='2023-02-12', description='Yet wish certain usually.', category=Food & Drink, amount=744.15, type='Expense')
923: Transaction(date='2023-02-13', description='Attack mouth rest contain.', category=Travel, amount=1481.76, type='Expense')
924: Transaction(date='2023-02-15', description='She they.', category=Health & Fitness, amount=1187.46, type='Expense')
925: Transaction(date='2023-02-16', description='Field seat start pm.', category=Entertainment, amount=1467.6, type='Expense')
926: Transaction(date='2023-02-18', description='Single blood continue.', category=Entertainment, amount=309.63, type='Expense')
927: Transaction(date='2023-02-20', description='Note discuss discussion reach grow.', category=Food & Drink, amount=473.67, type='Expense')
928: Transaction(date='2023-02-20', description='Maybe painting republican.', category=Food & Drink, amount=1902.47, type='Expense')
929: Transaction(date='2023-02-22', description='System man case.', category=Other, amount=3019.0, type='Income')
930: Transaction(date='2023-02-22', description='Allow share address.', category=Investment, amount=3939.0, type='Income')
931: Transaction(date='2023-02-24', description='Development provide dark daughter.', category=Food & Drink, amount=1945.39, type='Expense')
932: Transaction(date='2023-02-26', description='Term majority foot leader.', category=Other, amount=1262.0, type='Income')
933: Transaction(date='2023-02-26', description='If among save herself.', category=Investment, amount=4757.0, type='Income')
934: Transaction(date='2023-02-26', description='Particularly score order modern.', category=Health & Fitness, amount=922.33, type='Expense')
935: Transaction(date='2023-02-28', description='Body lead there there early.', category=Rent, amount=1015.8, type='Expense')
936: Transaction(date='2023-03-03', description='Worker always hold share.', category=Shopping, amount=158.91, type='Expense')
937: Transaction(date='2023-03-03', description='Fact huge fight.', category=Shopping, amount=1328.26, type='Expense')
938: Transaction(date='2023-03-05', description='Coach family try ready.', category=Other, amount=2638.0, type='Income')
939: Transaction(date='2023-03-05', description='Stop later reveal both.', category=Shopping, amount=1404.44, type='Expense')
940: Transaction(date='2023-03-05', description='Career including late condition opportunity.', category=Health & Fitness, amount=910.99, type='Expense')
941: Transaction(date='2023-03-07', description='Bed food possible represent.', category=Entertainment, amount=280.86, type='Expense')
942: Transaction(date='2023-03-08', description='Before relationship plant.', category=Rent, amount=319.27, type='Expense')
943: Transaction(date='2023-03-08', description='Religious risk window partner civil.', category=Other, amount=3072.0, type='Income')
944: Transaction(date='2023-03-08', description='Analysis color friend the.', category=Health & Fitness, amount=1929.62, type='Expense')
945: Transaction(date='2023-03-08', description='Than condition republican.', category=Travel, amount=932.06, type='Expense')
946: Transaction(date='2023-03-10', description='Finally shoulder land.', category=Rent, amount=323.65, type='Expense')
947: Transaction(date='2023-03-10', description='Together whether about.', category=Food & Drink, amount=437.89, type='Expense')
948: Transaction(date='2023-03-12', description='Learn talk.', category=Utilities, amount=191.51, type='Expense')
949: Transaction(date='2023-03-13', description='Part management.', category=Health & Fitness, amount=798.53, type='Expense')
950: Transaction(date='2023-03-13', description='Process law kind over according.', category=Other, amount=2631.0, type='Income')
951: Transaction(date='2023-03-13', description='People newspaper stay.', category=Utilities, amount=71.85, type='Expense')
952: Transaction(date='2023-03-15', description='Challenge president already.', category=Salary, amount=1116.35, type='Expense')
953: Transaction(date='2023-03-15', description='Study fear turn any indicate.', category=Rent, amount=929.47, type='Expense')
954: Transaction(date='2023-03-16', description='Public particularly reality.', category=Utilities, amount=251.69, type='Expense')
955: Transaction(date='2023-03-16', description='Concern lay often use behavior.', category=Investment, amount=1570.0, type='Income')
956: Transaction(date='2023-03-18', description='Happy her attack interview.', category=Other, amount=3549.0, type='Income')
957: Transaction(date='2023-03-18', description='Fall care.', category=Rent, amount=1168.82, type='Expense')
958: Transaction(date='2023-03-21', description='Money modern receive partner.', category=Rent, amount=53.38, type='Expense')
959: Transaction(date='2023-03-22', description='Role pick would low read.', category=Other, amount=1555.0, type='Income')
960: Transaction(date='2023-03-23', description='Participant first moment yet.', category=Other, amount=1503.0, type='Income')
961: Transaction(date='2023-03-23', description='Pass market role.', category=Other, amount=914.0, type='Income')
962: Transaction(date='2023-03-23', description='Space this discussion fact.', category=Other, amount=3735.0, type='Income')
963: Transaction(date='2023-03-24', description='Brother let.', category=Shopping, amount=1369.59, type='Expense')
964: Transaction(date='2023-03-27', description='Son yourself increase.', category=Investment, amount=2033.0, type='Income')
965: Transaction(date='2023-03-29', description='Night point action cut.', category=Salary, amount=1768.01, type='Expense')
966: Transaction(date='2023-03-30', description='So film media.', category=Investment, amount=2917.0, type='Income')
967: Transaction(date='2023-03-31', description='Box four.', category=Salary, amount=131.35, type='Expense')
968: Transaction(date='2023-03-31', description='Pattern ahead compare form full.', category=Utilities, amount=844.28, type='Expense')
969: Transaction(date='2023-04-01', description='Something population loss physical.', category=Entertainment, amount=1265.2, type='Expense')
970: Transaction(date='2023-04-01', description='Leg sport until argue.', category=Food & Drink, amount=1978.91, type='Expense')
971: Transaction(date='2023-04-04', description='North question new.', category=Health & Fitness, amount=964.57, type='Expense')
972: Transaction(date='2023-04-07', description='Officer forward trouble.', category=Health & Fitness, amount=1826.23, type='Expense')
973: Transaction(date='2023-04-08', description='Wide exactly.', category=Shopping, amount=548.17, type='Expense')
974: Transaction(date='2023-04-09', description='Worker phone success number.', category=Investment, amount=740.0, type='Income')
975: Transaction(date='2023-04-10', description='Organization ten seem.', category=Travel, amount=126.6, type='Expense')
976: Transaction(date='2023-04-10', description='Even beautiful.', category=Health & Fitness, amount=1536.02, type='Expense')
977: Transaction(date='2023-04-13', description='Later week democratic few.', category=Utilities, amount=209.27, type='Expense')
978: Transaction(date='2023-04-15', description='Size sing tree kitchen.', category=Travel, amount=1072.04, type='Expense')
979: Transaction(date='2023-04-16', description='Toward child future low.', category=Food & Drink, amount=1530.46, type='Expense')
980: Transaction(date='2023-04-20', description='Simply century the.', category=Shopping, amount=716.06, type='Expense')
981: Transaction(date='2023-04-20', description='Day already hold.', category=Utilities, amount=360.44, type='Expense')
982: Transaction(date='2023-04-21', description='About usually.', category=Health & Fitness, amount=1828.82, type='Expense')
983: Transaction(date='2023-04-22', description='Level animal minute probably.', category=Other, amount=1664.0, type='Income')
984: Transaction(date='2023-04-22', description='Officer fine.', category=Entertainment, amount=1675.03, type='Expense')
985: Transaction(date='2023-04-23', description='Customer expert mother significant.', category=Salary, amount=1088.83, type='Expense')
986: Transaction(date='2023-04-24', description='Chance event address.', category=Investment, amount=1239.0, type='Income')
987: Transaction(date='2023-04-25', description='Bill my near even wall.', category=Investment, amount=663.0, type='Income')
988: Transaction(date='2023-05-02', description='Each author party full kitchen.', category=Food & Drink, amount=670.15, type='Expense')
989: Transaction(date='2023-05-02', description='Likely push.', category=Utilities, amount=688.11, type='Expense')
990: Transaction(date='2023-05-03', description='Approach interview condition other.', category=Other, amount=987.0, type='Income')
991: Transaction(date='2023-05-05', description='Green forward cup serve speak.', category=Investment, amount=1408.0, type='Income')
992: Transaction(date='2023-05-05', description='Nothing behavior sort recognize.', category=Health & Fitness, amount=1921.51, type='Expense')
993: Transaction(date='2023-05-05', description='Financial popular short like.', category=Shopping, amount=1176.17, type='Expense')
994: Transaction(date='2023-05-07', description='Technology pay tree.', category=Health & Fitness, amount=281.54, type='Expense')
995: Transaction(date='2023-05-12', description='Record professor small.', category=Other, amount=4491.0, type='Income')
996: Transaction(date='2023-05-14', description='Maintain assume both long.', category=Utilities, amount=1417.29, type='Expense')
997: Transaction(date='2023-05-14', description='Should page camera.', category=Health & Fitness, amount=454.24, type='Expense')
998: Transaction(date='2023-05-14', description='Stage someone for power.', category=Utilities, amount=762.98, type='Expense')
999: Transaction(date='2023-05-15', description='Leave opportunity draw during city.', category=Travel, amount=263.42, type='Expense')
1000: Transaction(date='2023-05-15', description='At decade really issue.', category=Shopping, amount=1703.01, type='Expense')
1001: Transaction(date='2023-05-16', description='Skin name door production back.', category=Food & Drink, amount=372.34, type='Expense')
1002: Transaction(date='2023-05-17', description='Boy other foot best.', category=Entertainment, amount=1282.9, type='Expense')
1003: Transaction(date='2023-05-18', description='Final image our.', category=Other, amount=1652.0, type='Income')
1004: Transaction(date='2023-05-19', description='Management produce bed stage.', category=Rent, amount=483.61, type='Expense')
1005: Transaction(date='2023-05-20', description='Serious produce.', category=Utilities, amount=1276.01, type='Expense')
1006: Transaction(date='2023-05-20', description='Factor past language day theory.', category=Health & Fitness, amount=898.79, type='Expense')
1007: Transaction(date='2023-05-20', description='Some quickly truth view response.', category=Entertainment, amount=1389.42, type='Expense')
1008: Transaction(date='2023-05-22', description='Old particularly.', category=Rent, amount=1443.62, type='Expense')
1009: Transaction(date='2023-05-22', description='Summer system artist too.', category=Other, amount=3937.0, type='Income')
1010: Transaction(date='2023-05-22', description='Reflect star.', category=Shopping, amount=991.23, type='Expense')
1011: Transaction(date='2023-05-25', description='Technology tend.', category=Shopping, amount=211.84, type='Expense')
1012: Transaction(date='2023-05-28', description='Reduce hour fish him.', category=Entertainment, amount=333.69, type='Expense')
1013: Transaction(date='2023-05-29', description='Nation production little could.', category=Investment, amount=2915.0, type='Income')
1014: Transaction(date='2023-05-29', description='Style child young prove.', category=Other, amount=1357.0, type='Income')
1015: Transaction(date='2023-05-30', description='Author administration.', category=Investment, amount=3575.0, type='Income')
1016: Transaction(date='2023-06-02', description='Window career case south.', category=Investment, amount=4981.0, type='Income')
1017: Transaction(date='2023-06-03', description='Us perhaps lawyer interest star.', category=Rent, amount=682.52, type='Expense')
1018: Transaction(date='2023-06-04', description='Difficult interesting account marriage.', category=Salary, amount=1411.96, type='Expense')
1019: Transaction(date='2023-06-05', description='Especially unit office almost.', category=Utilities, amount=1218.56, type='Expense')
1020: Transaction(date='2023-06-08', description='Low happy oil which bag.', category=Food & Drink, amount=211.5, type='Expense')
1021: Transaction(date='2023-06-08', description='Let back three.', category=Entertainment, amount=824.47, type='Expense')
1022: Transaction(date='2023-06-09', description='Line score whether.', category=Travel, amount=1324.59, type='Expense')
1023: Transaction(date='2023-06-09', description='Something fine apply gas sing.', category=Rent, amount=108.77, type='Expense')
1024: Transaction(date='2023-06-10', description='Mention light safe inside.', category=Salary, amount=315.64, type='Expense')
1025: Transaction(date='2023-06-10', description='Cold reason our.', category=Shopping, amount=184.38, type='Expense')
1026: Transaction(date='2023-06-13', description='Employee picture official of prevent.', category=Salary, amount=407.35, type='Expense')
1027: Transaction(date='2023-06-15', description='Agree machine camera baby six.', category=Investment, amount=2890.0, type='Income')
1028: Transaction(date='2023-06-16', description='Past every either.', category=Salary, amount=1689.1, type='Expense')
1029: Transaction(date='2023-06-16', description='Company option character land space.', category=Shopping, amount=947.18, type='Expense')
1030: Transaction(date='2023-06-17', description='Machine son television.', category=Travel, amount=191.36, type='Expense')
1031: Transaction(date='2023-06-17', description='Whose other.', category=Utilities, amount=748.33, type='Expense')
1032: Transaction(date='2023-06-19', description='Sell include himself produce.', category=Utilities, amount=897.09, type='Expense')
1033: Transaction(date='2023-06-21', description='Official success approach.', category=Salary, amount=534.17, type='Expense')
1034: Transaction(date='2023-06-22', description='Apply less.', category=Food & Drink, amount=310.75, type='Expense')
1035: Transaction(date='2023-06-22', description='Live way two.', category=Rent, amount=180.56, type='Expense')
1036: Transaction(date='2023-06-23', description='Raise hope enjoy most.', category=Shopping, amount=1941.7, type='Expense')
1037: Transaction(date='2023-06-25', description='Eye something.', category=Food & Drink, amount=1683.19, type='Expense')
1038: Transaction(date='2023-06-25', description='Which raise american truth.', category=Shopping, amount=1931.6, type='Expense')
1039: Transaction(date='2023-06-25', description='Particular present evening establish.', category=Salary, amount=1149.7, type='Expense')
1040: Transaction(date='2023-06-26', description='Pressure hospital significant.', category=Travel, amount=167.12, type='Expense')
1041: Transaction(date='2023-06-30', description='Side mind question year finish.', category=Food & Drink, amount=1009.18, type='Expense')
1042: Transaction(date='2023-07-01', description='Story activity.', category=Utilities, amount=249.36, type='Expense')
1043: Transaction(date='2023-07-02', description='Billion land detail exist cover.', category=Utilities, amount=1898.63, type='Expense')
1044: Transaction(date='2023-07-05', description='Begin voice prevent structure wear.', category=Investment, amount=2078.0, type='Income')
1045: Transaction(date='2023-07-05', description='Beat painting heavy.', category=Other, amount=1141.0, type='Income')
1046: Transaction(date='2023-07-06', description='To receive education.', category=Shopping, amount=411.82, type='Expense')
1047: Transaction(date='2023-07-07', description='Police two.', category=Utilities, amount=1669.88, type='Expense')
1048: Transaction(date='2023-07-07', description='Realize property show either.', category=Rent, amount=183.18, type='Expense')
1049: Transaction(date='2023-07-10', description='Since him four.', category=Health & Fitness, amount=1714.68, type='Expense')
1050: Transaction(date='2023-07-10', description='Western enough key.', category=Food & Drink, amount=1575.24, type='Expense')
1051: Transaction(date='2023-07-10', description='Particular condition man.', category=Shopping, amount=233.91, type='Expense')
1052: Transaction(date='2023-07-10', description='Shoulder despite poor tree resource.', category=Investment, amount=1365.0, type='Income')
1053: Transaction(date='2023-07-11', description='Authority number.', category=Rent, amount=1573.63, type='Expense')
1054: Transaction(date='2023-07-13', description='Affect marriage present a.', category=Health & Fitness, amount=1757.01, type='Expense')
1055: Transaction(date='2023-07-15', description='Entire record toward off.', category=Salary, amount=1148.33, type='Expense')
1056: Transaction(date='2023-07-15', description='Long identify road speech.', category=Utilities, amount=558.07, type='Expense')
1057: Transaction(date='2023-07-15', description='Difficult his ready sister building.', category=Other, amount=817.0, type='Income')
1058: Transaction(date='2023-07-15', description='Four field area young.', category=Health & Fitness, amount=1384.12, type='Expense')
1059: Transaction(date='2023-07-15', description='Movement which ball least.', category=Other, amount=2700.0, type='Income')
1060: Transaction(date='2023-07-16', description='Computer always his.', category=Food & Drink, amount=682.35, type='Expense')
1061: Transaction(date='2023-07-17', description='Receive to piece.', category=Travel, amount=567.89, type='Expense')
1062: Transaction(date='2023-07-18', description='Be plan same cause.', category=Rent, amount=1945.27, type='Expense')
1063: Transaction(date='2023-07-19', description='Rest or.', category=Travel, amount=1948.7, type='Expense')
1064: Transaction(date='2023-07-19', description='As music film move.', category=Health & Fitness, amount=546.15, type='Expense')
1065: Transaction(date='2023-07-20', description='Already argue.', category=Rent, amount=383.99, type='Expense')
1066: Transaction(date='2023-07-21', description='Brother southern project learn.', category=Shopping, amount=1484.48, type='Expense')
1067: Transaction(date='2023-07-23', description='Scientist do author.', category=Health & Fitness, amount=1643.82, type='Expense')
1068: Transaction(date='2023-07-25', description='President likely truth animal.', category=Utilities, amount=140.58, type='Expense')
1069: Transaction(date='2023-07-26', description='Common anyone.', category=Shopping, amount=279.94, type='Expense')
1070: Transaction(date='2023-07-27', description='Lay color.', category=Food & Drink, amount=355.74, type='Expense')
1071: Transaction(date='2023-07-27', description='Box organization.', category=Entertainment, amount=1675.58, type='Expense')
1072: Transaction(date='2023-07-30', description='Source international.', category=Salary, amount=110.61, type='Expense')
1073: Transaction(date='2023-08-01', description='Building range national focus.', category=Entertainment, amount=555.22, type='Expense')
1074: Transaction(date='2023-08-04', description='Or friend.', category=Shopping, amount=1274.47, type='Expense')
1075: Transaction(date='2023-08-05', description='Center indeed recent.', category=Investment, amount=2481.0, type='Income')
1076: Transaction(date='2023-08-07', description='Just understand american.', category=Salary, amount=363.39, type='Expense')
1077: Transaction(date='2023-08-07', description='Partner sing administration main according.', category=Health & Fitness, amount=695.06, type='Expense')
1078: Transaction(date='2023-08-09', description='Religious order.', category=Travel, amount=1500.43, type='Expense')
1079: Transaction(date='2023-08-09', description='Really society live camera.', category=Travel, amount=64.63, type='Expense')
1080: Transaction(date='2023-08-10', description='Mission include visit.', category=Investment, amount=2313.0, type='Income')
1081: Transaction(date='2023-08-10', description='Conference matter wear physical site.', category=Entertainment, amount=545.48, type='Expense')
1082: Transaction(date='2023-08-11', description='Above dog brother morning.', category=Food & Drink, amount=1730.61, type='Expense')
1083: Transaction(date='2023-08-16', description='Gun seven music despite show.', category=Food & Drink, amount=953.45, type='Expense')
1084: Transaction(date='2023-08-17', description='Apply both chance.', category=Investment, amount=569.0, type='Income')
1085: Transaction(date='2023-08-17', description='Computer prove amount population.', category=Travel, amount=1428.35, type='Expense')
1086: Transaction(date='2023-08-21', description='Itself free source.', category=Travel, amount=853.38, type='Expense')
1087: Transaction(date='2023-08-22', description='Bit appear.', category=Health & Fitness, amount=923.41, type='Expense')
1088: Transaction(date='2023-08-23', description='Citizen ahead method.', category=Travel, amount=1357.62, type='Expense')
1089: Transaction(date='2023-08-25', description='Return or for.', category=Shopping, amount=1173.13, type='Expense')
1090: Transaction(date='2023-08-27', description='Discover body direction down.', category=Other, amount=2815.0, type='Income')
1091: Transaction(date='2023-08-30', description='Heart determine likely.', category=Rent, amount=968.66, type='Expense')
1092: Transaction(date='2023-08-30', description='Three certain.', category=Travel, amount=1284.78, type='Expense')
1093: Transaction(date='2023-08-30', description='Hold carry rest.', category=Travel, amount=1315.68, type='Expense')
1094: Transaction(date='2023-08-30', description='Dark camera realize.', category=Travel, amount=384.84, type='Expense')
1095: Transaction(date='2023-09-01', description='Carry surface learn.', category=Entertainment, amount=1800.38, type='Expense')
1096: Transaction(date='2023-09-01', description='Detail far church some.', category=Shopping, amount=1324.46, type='Expense')
1097: Transaction(date='2023-09-03', description='Wife talk beat tax majority.', category=Food & Drink, amount=1314.25, type='Expense')
1098: Transaction(date='2023-09-04', description='List leg address into their.', category=Travel, amount=1814.75, type='Expense')
1099: Transaction(date='2023-09-04', description='Its develop until watch.', category=Food & Drink, amount=1170.22, type='Expense')
1100: Transaction(date='2023-09-04', description='Individual analysis.', category=Entertainment, amount=692.9, type='Expense')
1101: Transaction(date='2023-09-07', description='Almost most look something.', category=Salary, amount=1813.99, type='Expense')
1102: Transaction(date='2023-09-10', description='Manager bar send really.', category=Travel, amount=1692.38, type='Expense')
1103: Transaction(date='2023-09-11', description='Car fast positive.', category=Utilities, amount=594.63, type='Expense')
1104: Transaction(date='2023-09-11', description='Look fight.', category=Travel, amount=1453.12, type='Expense')
1105: Transaction(date='2023-09-12', description='Section agency lawyer sign them.', category=Utilities, amount=1894.38, type='Expense')
1106: Transaction(date='2023-09-12', description='Smile have less.', category=Rent, amount=452.28, type='Expense')
1107: Transaction(date='2023-09-13', description='Audience throw four allow international.', category=Travel, amount=846.74, type='Expense')
1108: Transaction(date='2023-09-14', description='My some skin ball thing.', category=Health & Fitness, amount=528.51, type='Expense')
1109: Transaction(date='2023-09-14', description='War push thousand book half.', category=Rent, amount=946.37, type='Expense')
1110: Transaction(date='2023-09-14', description='Hand possible relate night.', category=Rent, amount=949.72, type='Expense')
1111: Transaction(date='2023-09-15', description='Wonder draw box.', category=Investment, amount=3626.0, type='Income')
1112: Transaction(date='2023-09-15', description='That reflect send set fly.', category=Shopping, amount=117.58, type='Expense')
1113: Transaction(date='2023-09-16', description='Quite everyone as generation.', category=Utilities, amount=1984.77, type='Expense')
1114: Transaction(date='2023-09-16', description='Money but report question.', category=Salary, amount=295.02, type='Expense')
1115: Transaction(date='2023-09-21', description='Later happy pull.', category=Utilities, amount=1560.09, type='Expense')
1116: Transaction(date='2023-09-21', description='Throughout leader bag.', category=Utilities, amount=633.69, type='Expense')
1117: Transaction(date='2023-09-23', description='When baby clearly.', category=Other, amount=2740.0, type='Income')
1118: Transaction(date='2023-09-24', description='Many themselves minute.', category=Utilities, amount=1156.8, type='Expense')
1119: Transaction(date='2023-09-28', description='Indicate radio use.', category=Investment, amount=1560.0, type='Income')
1120: Transaction(date='2023-09-29', description='Information scene again country.', category=Shopping, amount=812.75, type='Expense')
1121: Transaction(date='2023-10-03', description='Strong mind from.', category=Shopping, amount=19.29, type='Expense')
1122: Transaction(date='2023-10-04', description='Pretty skin main program.', category=Health & Fitness, amount=1900.64, type='Expense')
1123: Transaction(date='2023-10-04', description='Knowledge fight car smile commercial.', category=Entertainment, amount=45.65, type='Expense')
1124: Transaction(date='2023-10-05', description='Machine wife show.', category=Investment, amount=2353.0, type='Income')
1125: Transaction(date='2023-10-06', description='Good lay coach.', category=Health & Fitness, amount=631.06, type='Expense')
1126: Transaction(date='2023-10-08', description='Expect win stuff realize project.', category=Other, amount=1868.0, type='Income')
1127: Transaction(date='2023-10-08', description='Draw american first.', category=Investment, amount=2942.0, type='Income')
1128: Transaction(date='2023-10-08', description='Employee continue none alone.', category=Salary, amount=1458.39, type='Expense')
1129: Transaction(date='2023-10-09', description='Challenge large wish move.', category=Health & Fitness, amount=872.99, type='Expense')
1130: Transaction(date='2023-10-09', description='Identify material.', category=Utilities, amount=692.04, type='Expense')
1131: Transaction(date='2023-10-11', description='Until choose wife national.', category=Health & Fitness, amount=1502.73, type='Expense')
1132: Transaction(date='2023-10-13', description='Positive far time wide.', category=Other, amount=656.0, type='Income')
1133: Transaction(date='2023-10-14', description='Son rock two.', category=Utilities, amount=430.27, type='Expense')
1134: Transaction(date='2023-10-14', description='Rule too.', category=Investment, amount=1508.0, type='Income')
1135: Transaction(date='2023-10-15', description='Bring agent pay detail group.', category=Food & Drink, amount=756.63, type='Expense')
1136: Transaction(date='2023-10-15', description='Part crime.', category=Food & Drink, amount=1191.91, type='Expense')
1137: Transaction(date='2023-10-16', description='Sit short cultural.', category=Investment, amount=661.0, type='Income')
1138: Transaction(date='2023-10-17', description='Support quality whatever wish hour.', category=Investment, amount=3914.0, type='Income')
1139: Transaction(date='2023-10-17', description='Owner memory argue project.', category=Rent, amount=1099.09, type='Expense')
1140: Transaction(date='2023-10-27', description='Respond into history.', category=Utilities, amount=418.93, type='Expense')
1141: Transaction(date='2023-10-27', description='General final knowledge.', category=Food & Drink, amount=609.22, type='Expense')
1142: Transaction(date='2023-10-29', description='Democratic thousand.', category=Travel, amount=1249.37, type='Expense')
1143: Transaction(date='2023-10-30', description='Eye south wife interest.', category=Shopping, amount=1663.0, type='Expense')
1144: Transaction(date='2023-10-30', description='Structure our century.', category=Investment, amount=2815.0, type='Income')
1145: Transaction(date='2023-11-04', description='Write moment friend value.', category=Investment, amount=4577.0, type='Income')
1146: Transaction(date='2023-11-05', description='According police.', category=Investment, amount=2654.0, type='Income')
1147: Transaction(date='2023-11-08', description='Company left nothing trip.', category=Entertainment, amount=722.26, type='Expense')
1148: Transaction(date='2023-11-08', description='Strategy face make.', category=Utilities, amount=88.42, type='Expense')
1149: Transaction(date='2023-11-08', description='Fight nice less agree able.', category=Entertainment, amount=1379.53, type='Expense')
1150: Transaction(date='2023-11-09', description='What strong line.', category=Entertainment, amount=15.45, type='Expense')
1151: Transaction(date='2023-11-09', description='Conference trade population agreement.', category=Rent, amount=1672.11, type='Expense')
1152: Transaction(date='2023-11-11', description='Yes at seat ok.', category=Utilities, amount=1899.4, type='Expense')
1153: Transaction(date='2023-11-11', description='Than physical live reason.', category=Rent, amount=1449.89, type='Expense')
1154: Transaction(date='2023-11-13', description='Mention offer suddenly clear.', category=Other, amount=2393.0, type='Income')
1155: Transaction(date='2023-11-14', description='Floor there career.', category=Travel, amount=535.61, type='Expense')
1156: Transaction(date='2023-11-14', description='Evening information.', category=Investment, amount=823.0, type='Income')
1157: Transaction(date='2023-11-15', description='Face morning.', category=Health & Fitness, amount=1605.18, type='Expense')
1158: Transaction(date='2023-11-17', description='Create physical share out cold.', category=Rent, amount=199.05, type='Expense')
1159: Transaction(date='2023-11-18', description='Step rate feeling rate.', category=Food & Drink, amount=29.25, type='Expense')
1160: Transaction(date='2023-11-21', description='Move goal.', category=Investment, amount=3192.0, type='Income')
1161: Transaction(date='2023-11-21', description='Carry example.', category=Salary, amount=1910.63, type='Expense')
1162: Transaction(date='2023-11-22', description='Future treat bill choice.', category=Rent, amount=751.52, type='Expense')
1163: Transaction(date='2023-11-23', description='Somebody next ready.', category=Rent, amount=807.55, type='Expense')
1164: Transaction(date='2023-11-23', description='Republican reduce sort individual trouble.', category=Travel, amount=1673.06, type='Expense')
1165: Transaction(date='2023-11-26', description='Type look find.', category=Salary, amount=1601.8, type='Expense')
1166: Transaction(date='2023-11-26', description='Stage threat our.', category=Utilities, amount=234.98, type='Expense')
1167: Transaction(date='2023-11-26', description='Report agreement sport.', category=Food & Drink, amount=1963.32, type='Expense')
1168: Transaction(date='2023-11-26', description='Task central dinner.', category=Rent, amount=837.9, type='Expense')
1169: Transaction(date='2023-11-27', description='Candidate bad real.', category=Salary, amount=370.71, type='Expense')
1170: Transaction(date='2023-11-27', description='Affect machine.', category=Health & Fitness, amount=1226.6, type='Expense')
1171: Transaction(date='2023-11-28', description='Cultural allow.', category=Utilities, amount=971.33, type='Expense')
1172: Transaction(date='2023-11-28', description='Arrive budget education at property.', category=Shopping, amount=886.97, type='Expense')
1173: Transaction(date='2023-11-29', description='Turn day top.', category=Other, amount=601.0, type='Income')
1174: Transaction(date='2023-12-05', description='Ahead certainly second everyone.', category=Entertainment, amount=1956.01, type='Expense')
1175: Transaction(date='2023-12-05', description='Goal talk sit party.', category=Health & Fitness, amount=1043.01, type='Expense')
1176: Transaction(date='2023-12-06', description='Attention standard shake more respond.', category=Rent, amount=871.37, type='Expense')
1177: Transaction(date='2023-12-08', description='Top mr wind.', category=Salary, amount=698.04, type='Expense')
1178: Transaction(date='2023-12-09', description='Turn surface listen.', category=Shopping, amount=305.82, type='Expense')
1179: Transaction(date='2023-12-09', description='Pull blue strong wonder.', category=Food & Drink, amount=33.17, type='Expense')
1180: Transaction(date='2023-12-10', description='Method imagine seven.', category=Salary, amount=1705.22, type='Expense')
1181: Transaction(date='2023-12-11', description='Letter mind many.', category=Rent, amount=1505.35, type='Expense')
1182: Transaction(date='2023-12-13', description='News american foreign.', category=Health & Fitness, amount=1940.59, type='Expense')
1183: Transaction(date='2023-12-14', description='Exist outside find wish.', category=Investment, amount=3601.0, type='Income')
1184: Transaction(date='2023-12-14', description='Network fact talk.', category=Entertainment, amount=166.87, type='Expense')
1185: Transaction(date='2023-12-18', description='Thus sort.', category=Salary, amount=185.16, type='Expense')
1186: Transaction(date='2023-12-20', description='Actually purpose public.', category=Health & Fitness, amount=246.23, type='Expense')
1187: Transaction(date='2023-12-21', description='Mission put budget house.', category=Food & Drink, amount=214.26, type='Expense')
1188: Transaction(date='2023-12-22', description='Ask window full.', category=Food & Drink, amount=1653.56, type='Expense')
1189: Transaction(date='2023-12-23', description='Do interesting material.', category=Travel, amount=1781.32, type='Expense')
1190: Transaction(date='2023-12-25', description='Event smile popular hundred live.', category=Salary, amount=1901.21, type='Expense')
1191: Transaction(date='2023-12-26', description='Memory challenge lawyer business.', category=Food & Drink, amount=1635.01, type='Expense')
1192: Transaction(date='2023-12-27', description='Discuss wall ahead scene.', category=Health & Fitness, amount=1124.7, type='Expense')
1193: Transaction(date='2023-12-29', description='Question will him quickly.', category=Salary, amount=1436.15, type='Expense')
1194: Transaction(date='2023-12-29', description='Perform benefit baby.', category=Salary, amount=236.91, type='Expense')
1195: Transaction(date='2023-12-29', description='Society dark product make.', category=Entertainment, amount=14.37, type='Expense')
1196: Transaction(date='2024-01-02', description='Gas teacher national weight all.', category=Travel, amount=1099.9, type='Expense')
1197: Transaction(date='2024-01-02', description='Himself fire heavy.', category=Rent, amount=101.1, type='Expense')
1198: Transaction(date='2024-01-04', description='Allow news.', category=Investment, amount=3852.0, type='Income')
1199: Transaction(date='2024-01-04', description='Man care resource.', category=Investment, amount=2036.0, type='Income')
1200: Transaction(date='2024-01-07', description='Notice make.', category=Other, amount=2266.0, type='Income')
1201: Transaction(date='2024-01-09', description='Occur live take media hot.', category=Entertainment, amount=1436.41, type='Expense')
1202: Transaction(date='2024-01-09', description='Wife pay good within fly.', category=Other, amount=880.0, type='Income')
1203: Transaction(date='2024-01-10', description='And student give score agree.', category=Rent, amount=1546.57, type='Expense')
1204: Transaction(date='2024-01-11', description='Best generation group goal.', category=Utilities, amount=1928.07, type='Expense')
1205: Transaction(date='2024-01-11', description='Doctor fine be.', category=Investment, amount=4045.0, type='Income')
1206: Transaction(date='2024-01-11', description='Boy college mrs.', category=Salary, amount=1849.5, type='Expense')
1207: Transaction(date='2024-01-12', description='Price institution method inside.', category=Utilities, amount=1386.72, type='Expense')
1208: Transaction(date='2024-01-13', description='Onto worry help hope.', category=Investment, amount=1348.0, type='Income')
1209: Transaction(date='2024-01-14', description='Space beyond.', category=Health & Fitness, amount=1013.18, type='Expense')
1210: Transaction(date='2024-01-15', description='Market specific property coach.', category=Entertainment, amount=554.46, type='Expense')
1211: Transaction(date='2024-01-16', description='Wide cut player wind.', category=Travel, amount=911.16, type='Expense')
1212: Transaction(date='2024-01-16', description='Degree enter father of source.', category=Other, amount=4696.0, type='Income')
1213: Transaction(date='2024-01-16', description='Behind front attack song little.', category=Health & Fitness, amount=1426.63, type='Expense')
1214: Transaction(date='2024-01-17', description='Somebody prepare few sound.', category=Entertainment, amount=409.77, type='Expense')
1215: Transaction(date='2024-01-19', description='Finish increase economy middle public.', category=Travel, amount=1376.36, type='Expense')
1216: Transaction(date='2024-01-19', description='Thing physical might.', category=Utilities, amount=1962.86, type='Expense')
1217: Transaction(date='2024-01-19', description='Material produce station visit environmental.', category=Utilities, amount=1012.75, type='Expense')
1218: Transaction(date='2024-01-19', description='Table trade various risk art.', category=Health & Fitness, amount=1423.6, type='Expense')
1219: Transaction(date='2024-01-20', description='Leg drive second she.', category=Salary, amount=1169.78, type='Expense')
1220: Transaction(date='2024-01-22', description='Five store ask data.', category=Shopping, amount=1351.74, type='Expense')
1221: Transaction(date='2024-01-23', description='Statement position either.', category=Travel, amount=1756.18, type='Expense')
1222: Transaction(date='2024-01-24', description='Draw change agree someone.', category=Food & Drink, amount=1050.23, type='Expense')
1223: Transaction(date='2024-01-25', description='Magazine yard instead town.', category=Investment, amount=934.0, type='Income')
1224: Transaction(date='2024-01-26', description='Evening ahead smile.', category=Entertainment, amount=541.29, type='Expense')
1225: Transaction(date='2024-01-26', description='Compare system white magazine after.', category=Travel, amount=1369.95, type='Expense')
1226: Transaction(date='2024-01-26', description='Own test too imagine.', category=Investment, amount=2917.0, type='Income')
1227: Transaction(date='2024-01-27', description='However style.', category=Food & Drink, amount=723.78, type='Expense')
1228: Transaction(date='2024-01-28', description='Born same usually.', category=Entertainment, amount=1487.4, type='Expense')
1229: Transaction(date='2024-01-30', description='Wear use edge this draw.', category=Entertainment, amount=1026.74, type='Expense')
1230: Transaction(date='2024-02-01', description='Forget once hope security.', category=Travel, amount=1388.94, type='Expense')
1231: Transaction(date='2024-02-06', description='Design blood degree ever.', category=Other, amount=3674.0, type='Income')
1232: Transaction(date='2024-02-07', description='Television argue mouth.', category=Health & Fitness, amount=1932.01, type='Expense')
1233: Transaction(date='2024-02-07', description='Nation little east everyone.', category=Other, amount=2163.0, type='Income')
1234: Transaction(date='2024-02-10', description='Little happy final population.', category=Salary, amount=1661.67, type='Expense')
1235: Transaction(date='2024-02-10', description='Real small.', category=Health & Fitness, amount=1512.66, type='Expense')
1236: Transaction(date='2024-02-10', description='Need finally when.', category=Entertainment, amount=1554.53, type='Expense')
1237: Transaction(date='2024-02-14', description='Movement door kind.', category=Travel, amount=903.14, type='Expense')
1238: Transaction(date='2024-02-14', description='Until truth feeling view.', category=Rent, amount=382.08, type='Expense')
1239: Transaction(date='2024-02-15', description='Decision three trouble.', category=Investment, amount=3590.0, type='Income')
1240: Transaction(date='2024-02-16', description='Smile ever bag single.', category=Other, amount=2664.0, type='Income')
1241: Transaction(date='2024-02-17', description='Say unit four institution.', category=Other, amount=546.0, type='Income')
1242: Transaction(date='2024-02-19', description='Recognize media goal.', category=Utilities, amount=89.48, type='Expense')
1243: Transaction(date='2024-02-21', description='Law agree appear political.', category=Food & Drink, amount=1549.59, type='Expense')
1244: Transaction(date='2024-02-21', description='Able customer figure.', category=Travel, amount=986.2, type='Expense')
1245: Transaction(date='2024-02-21', description='Book economic.', category=Health & Fitness, amount=155.03, type='Expense')
1246: Transaction(date='2024-02-22', description='Rate society floor long same.', category=Entertainment, amount=1914.41, type='Expense')
1247: Transaction(date='2024-02-23', description='Reach step son.', category=Utilities, amount=837.7, type='Expense')
1248: Transaction(date='2024-02-23', description='Increase consider.', category=Rent, amount=705.31, type='Expense')
1249: Transaction(date='2024-02-24', description='Difference way type positive phone.', category=Health & Fitness, amount=668.82, type='Expense')
1250: Transaction(date='2024-02-24', description='Cup red collection husband.', category=Salary, amount=1993.3, type='Expense')
1251: Transaction(date='2024-02-25', description='Employee become happy water bring.', category=Food & Drink, amount=238.3, type='Expense')
1252: Transaction(date='2024-02-27', description='Significant up still management.', category=Entertainment, amount=1809.95, type='Expense')
1253: Transaction(date='2024-02-27', description='Course technology site.', category=Entertainment, amount=1918.2, type='Expense')
1254: Transaction(date='2024-02-27', description='High issue deal democratic.', category=Health & Fitness, amount=768.93, type='Expense')
1255: Transaction(date='2024-02-28', description='Industry impact certainly.', category=Rent, amount=777.89, type='Expense')
1256: Transaction(date='2024-02-29', description='Dog late eat.', category=Travel, amount=59.16, type='Expense')
1257: Transaction(date='2024-03-02', description='Beat light the window.', category=Health & Fitness, amount=1033.48, type='Expense')
1258: Transaction(date='2024-03-03', description='Require remain save.', category=Salary, amount=97.26, type='Expense')
1259: Transaction(date='2024-03-05', description='Believe arrive.', category=Entertainment, amount=647.42, type='Expense')
1260: Transaction(date='2024-03-05', description='Likely great paper.', category=Investment, amount=1871.0, type='Income')
1261: Transaction(date='2024-03-06', description='Effect people.', category=Food & Drink, amount=1257.6, type='Expense')
1262: Transaction(date='2024-03-06', description='Provide car throughout american.', category=Entertainment, amount=1731.25, type='Expense')
1263: Transaction(date='2024-03-06', description='College manage fund.', category=Rent, amount=419.1, type='Expense')
1264: Transaction(date='2024-03-07', description='Bed report item.', category=Shopping, amount=1296.66, type='Expense')
1265: Transaction(date='2024-03-07', description='Finish without.', category=Utilities, amount=57.27, type='Expense')
1266: Transaction(date='2024-03-08', description='Security unit executive.', category=Food & Drink, amount=184.17, type='Expense')
1267: Transaction(date='2024-03-11', description='Stop unit might job close.', category=Other, amount=1077.0, type='Income')
1268: Transaction(date='2024-03-13', description='New type pm.', category=Entertainment, amount=1822.86, type='Expense')
1269: Transaction(date='2024-03-15', description='Country approach.', category=Salary, amount=1264.13, type='Expense')
1270: Transaction(date='2024-03-16', description='Like clear.', category=Food & Drink, amount=785.11, type='Expense')
1271: Transaction(date='2024-03-18', description='Break time manager kid.', category=Investment, amount=1018.0, type='Income')
1272: Transaction(date='2024-03-26', description='Leader both rest challenge.', category=Food & Drink, amount=1404.76, type='Expense')
1273: Transaction(date='2024-03-27', description='Data exist.', category=Investment, amount=2874.0, type='Income')
1274: Transaction(date='2024-03-28', description='State none production defense your.', category=Travel, amount=1217.07, type='Expense')
1275: Transaction(date='2024-03-29', description='Short congress turn east marriage.', category=Salary, amount=769.01, type='Expense')
1276: Transaction(date='2024-03-30', description='Ago race quite hour.', category=Food & Drink, amount=953.24, type='Expense')
1277: Transaction(date='2024-03-30', description='Child ball store process.', category=Utilities, amount=1496.04, type='Expense')
1278: Transaction(date='2024-04-03', description='Western order accept wide prevent.', category=Travel, amount=1920.67, type='Expense')
1279: Transaction(date='2024-04-04', description='Water yourself trade someone.', category=Other, amount=591.0, type='Income')
1280: Transaction(date='2024-04-06', description='Tell soon executive.', category=Utilities, amount=255.24, type='Expense')
1281: Transaction(date='2024-04-07', description='Natural own.', category=Other, amount=4900.0, type='Income')
1282: Transaction(date='2024-04-08', description='Effort drive side human.', category=Rent, amount=616.34, type='Expense')
1283: Transaction(date='2024-04-09', description='Clear single expect.', category=Health & Fitness, amount=431.36, type='Expense')
1284: Transaction(date='2024-04-09', description='Fear song save.', category=Utilities, amount=330.3, type='Expense')
1285: Transaction(date='2024-04-10', description='Capital program story mention majority.', category=Travel, amount=396.71, type='Expense')
1286: Transaction(date='2024-04-10', description='Environmental outside bed institution.', category=Investment, amount=4996.0, type='Income')
1287: Transaction(date='2024-04-11', description='Control piece behavior message wind.', category=Rent, amount=588.4, type='Expense')
1288: Transaction(date='2024-04-13', description='Us money positive.', category=Investment, amount=2117.0, type='Income')
1289: Transaction(date='2024-04-16', description='Life bad soon.', category=Salary, amount=1903.56, type='Expense')
1290: Transaction(date='2024-04-17', description='Worker range feel.', category=Entertainment, amount=209.09, type='Expense')
1291: Transaction(date='2024-04-17', description='Into notice necessary true message.', category=Entertainment, amount=642.39, type='Expense')
1292: Transaction(date='2024-04-17', description='Including let result spend.', category=Travel, amount=1389.98, type='Expense')
1293: Transaction(date='2024-04-18', description='One shoulder administration.', category=Other, amount=3091.0, type='Income')
1294: Transaction(date='2024-04-20', description='Skill successful and.', category=Utilities, amount=19.62, type='Expense')
1295: Transaction(date='2024-04-21', description='Finish right serious.', category=Entertainment, amount=1427.28, type='Expense')
1296: Transaction(date='2024-04-22', description='Space game truth special decision.', category=Travel, amount=1681.82, type='Expense')
1297: Transaction(date='2024-04-23', description='Else pull its adult house.', category=Other, amount=4431.0, type='Income')
1298: Transaction(date='2024-04-25', description='Pretty least if.', category=Health & Fitness, amount=1551.79, type='Expense')
1299: Transaction(date='2024-04-26', description='Peace wind guess.', category=Rent, amount=359.78, type='Expense')
1300: Transaction(date='2024-04-26', description='Because career course.', category=Travel, amount=1718.84, type='Expense')
1301: Transaction(date='2024-04-30', description='Teacher decide possible.', category=Investment, amount=2656.0, type='Income')
1302: Transaction(date='2024-05-02', description='Success teach mrs beat.', category=Rent, amount=1457.58, type='Expense')
1303: Transaction(date='2024-05-03', description='Listen against coach.', category=Food & Drink, amount=1627.16, type='Expense')
1304: Transaction(date='2024-05-04', description='White likely teacher one.', category=Entertainment, amount=1813.54, type='Expense')
1305: Transaction(date='2024-05-04', description='Notice street anything.', category=Salary, amount=1311.28, type='Expense')
1306: Transaction(date='2024-05-05', description='Piece require.', category=Other, amount=2329.0, type='Income')
1307: Transaction(date='2024-05-05', description='Say other never fact.', category=Travel, amount=276.42, type='Expense')
1308: Transaction(date='2024-05-14', description='Main opportunity particularly question.', category=Rent, amount=799.6, type='Expense')
1309: Transaction(date='2024-05-15', description='Research month instead.', category=Entertainment, amount=1697.48, type='Expense')
1310: Transaction(date='2024-05-17', description='As throughout individual list manager.', category=Utilities, amount=1516.84, type='Expense')
1311: Transaction(date='2024-05-20', description='Recently again explain court.', category=Investment, amount=1967.0, type='Income')
1312: Transaction(date='2024-05-22', description='Discussion detail training fund.', category=Utilities, amount=1474.48, type='Expense')
1313: Transaction(date='2024-05-23', description='Third rich hospital.', category=Health & Fitness, amount=90.44, type='Expense')
1314: Transaction(date='2024-05-25', description='Somebody son suddenly perform.', category=Entertainment, amount=135.09, type='Expense')
1315: Transaction(date='2024-05-26', description='Project can.', category=Food & Drink, amount=1340.49, type='Expense')
1316: Transaction(date='2024-05-27', description='Paper beyond indeed.', category=Health & Fitness, amount=782.31, type='Expense')
1317: Transaction(date='2024-05-27', description='Doctor news yourself.', category=Food & Drink, amount=89.41, type='Expense')
1318: Transaction(date='2024-05-31', description='Ground city still.', category=Rent, amount=1410.44, type='Expense')
1319: Transaction(date='2024-06-01', description='Southern front.', category=Travel, amount=1671.4, type='Expense')
1320: Transaction(date='2024-06-02', description='Dinner no main year size.', category=Utilities, amount=759.55, type='Expense')
1321: Transaction(date='2024-06-02', description='Environment character real middle prove.', category=Rent, amount=1038.16, type='Expense')
1322: Transaction(date='2024-06-03', description='Sometimes while sense window.', category=Entertainment, amount=1037.58, type='Expense')
1323: Transaction(date='2024-06-04', description='Among black picture that treat.', category=Health & Fitness, amount=978.85, type='Expense')
1324: Transaction(date='2024-06-06', description='Remain seven marriage.', category=Food & Drink, amount=1008.66, type='Expense')
1325: Transaction(date='2024-06-07', description='Positive free case.', category=Investment, amount=2384.0, type='Income')
1326: Transaction(date='2024-06-09', description='Wide shoulder according strong.', category=Other, amount=2232.0, type='Income')
1327: Transaction(date='2024-06-10', description='Expert commercial entire resource.', category=Investment, amount=3467.0, type='Income')
1328: Transaction(date='2024-06-12', description='Picture early send morning.', category=Rent, amount=1325.61, type='Expense')
1329: Transaction(date='2024-06-13', description='Soon including important.', category=Health & Fitness, amount=1443.31, type='Expense')
1330: Transaction(date='2024-06-13', description='Some gun ahead certainly act.', category=Salary, amount=963.25, type='Expense')
1331: Transaction(date='2024-06-14', description='Be fire.', category=Shopping, amount=54.67, type='Expense')
1332: Transaction(date='2024-06-15', description='Article service produce political.', category=Investment, amount=1603.0, type='Income')
1333: Transaction(date='2024-06-16', description='Within woman.', category=Food & Drink, amount=1782.69, type='Expense')
1334: Transaction(date='2024-06-17', description='Information though red.', category=Investment, amount=2615.0, type='Income')
1335: Transaction(date='2024-06-21', description='Training account evidence month.', category=Shopping, amount=1202.72, type='Expense')
1336: Transaction(date='2024-06-21', description='Land day name small.', category=Food & Drink, amount=1758.23, type='Expense')
1337: Transaction(date='2024-06-22', description='Thought sit identify culture.', category=Travel, amount=1895.04, type='Expense')
1338: Transaction(date='2024-06-23', description='Already free.', category=Salary, amount=1735.93, type='Expense')
1339: Transaction(date='2024-06-23', description='Yeah exist behavior necessary miss.', category=Other, amount=1669.0, type='Income')
1340: Transaction(date='2024-06-23', description='Agreement someone.', category=Utilities, amount=1250.19, type='Expense')
1341: Transaction(date='2024-06-23', description='Strategy fight institution.', category=Utilities, amount=1451.13, type='Expense')
1342: Transaction(date='2024-06-23', description='Rule success.', category=Salary, amount=1197.1, type='Expense')
1343: Transaction(date='2024-07-01', description='Medical buy series walk large.', category=Other, amount=3657.0, type='Income')
1344: Transaction(date='2024-07-05', description='Law work.', category=Investment, amount=2157.0, type='Income')
1345: Transaction(date='2024-07-06', description='Research visit program suddenly doctor.', category=Shopping, amount=1654.76, type='Expense')
1346: Transaction(date='2024-07-07', description='Project spring of.', category=Utilities, amount=956.98, type='Expense')
1347: Transaction(date='2024-07-07', description='Mind easy foreign old.', category=Food & Drink, amount=1756.73, type='Expense')
1348: Transaction(date='2024-07-07', description='Better republican stock bank.', category=Rent, amount=195.45, type='Expense')
1349: Transaction(date='2024-07-07', description='Thousand live land hard.', category=Food & Drink, amount=1794.97, type='Expense')
1350: Transaction(date='2024-07-13', description='Open million house enough.', category=Shopping, amount=1048.87, type='Expense')
1351: Transaction(date='2024-07-13', description='You ready way rule.', category=Food & Drink, amount=1398.31, type='Expense')
1352: Transaction(date='2024-07-16', description='Certain camera represent.', category=Salary, amount=1581.22, type='Expense')
1353: Transaction(date='2024-07-20', description='Tell brother defense list second.', category=Rent, amount=856.72, type='Expense')
1354: Transaction(date='2024-07-21', description='Tv budget system book.', category=Entertainment, amount=1821.07, type='Expense')
1355: Transaction(date='2024-07-22', description='Face mission brother figure hear.', category=Other, amount=2611.0, type='Income')
1356: Transaction(date='2024-07-23', description='Agreement central six instead.', category=Investment, amount=4780.0, type='Income')
1357: Transaction(date='2024-07-23', description='Democratic standard major.', category=Investment, amount=3572.0, type='Income')
1358: Transaction(date='2024-07-25', description='Give that individual of not.', category=Health & Fitness, amount=823.6, type='Expense')
1359: Transaction(date='2024-07-26', description='Little result paper seven.', category=Other, amount=2577.0, type='Income')
1360: Transaction(date='2024-07-26', description='Seven project chance road.', category=Shopping, amount=1133.69, type='Expense')
1361: Transaction(date='2024-07-26', description='Professor important do house must.', category=Shopping, amount=1512.39, type='Expense')
1362: Transaction(date='2024-07-26', description='Position tend religious.', category=Salary, amount=365.65, type='Expense')
1363: Transaction(date='2024-07-27', description='Someone religious night buy nice.', category=Salary, amount=515.02, type='Expense')
1364: Transaction(date='2024-07-27', description='My serious whatever miss seek.', category=Health & Fitness, amount=1550.65, type='Expense')
1365: Transaction(date='2024-07-27', description='Build view.', category=Utilities, amount=1711.45, type='Expense')
1366: Transaction(date='2024-07-28', description='Ever day.', category=Entertainment, amount=898.72, type='Expense')
1367: Transaction(date='2024-07-28', description='Be move fly open.', category=Utilities, amount=209.55, type='Expense')
1368: Transaction(date='2024-07-29', description='Future congress.', category=Health & Fitness, amount=1603.7, type='Expense')
1369: Transaction(date='2024-07-29', description='Fish either boy between collection.', category=Food & Drink, amount=329.87, type='Expense')
1370: Transaction(date='2024-07-29', description='Eight bed fill.', category=Food & Drink, amount=1610.7, type='Expense')
1371: Transaction(date='2024-07-30', description='Arrive which consider.', category=Salary, amount=1852.46, type='Expense')
1372: Transaction(date='2024-07-30', description='Hope floor include.', category=Rent, amount=533.55, type='Expense')
1373: Transaction(date='2024-08-01', description='Heart still building.', category=Food & Drink, amount=1451.74, type='Expense')
1374: Transaction(date='2024-08-02', description='Sell only manage.', category=Shopping, amount=1231.79, type='Expense')
1375: Transaction(date='2024-08-02', description='We especially five network.', category=Food & Drink, amount=1891.97, type='Expense')
1376: Transaction(date='2024-08-03', description='Ahead plant trial affect.', category=Salary, amount=883.55, type='Expense')
1377: Transaction(date='2024-08-03', description='Push east never size fight.', category=Travel, amount=397.55, type='Expense')
1378: Transaction(date='2024-08-08', description='Base player data whose.', category=Entertainment, amount=1234.23, type='Expense')
1379: Transaction(date='2024-08-09', description='World site mr interest treatment.', category=Health & Fitness, amount=486.12, type='Expense')
1380: Transaction(date='2024-08-11', description='A let.', category=Investment, amount=1211.0, type='Income')
1381: Transaction(date='2024-08-14', description='A value lot.', category=Shopping, amount=712.72, type='Expense')
1382: Transaction(date='2024-08-14', description='Yourself successful defense subject.', category=Entertainment, amount=1040.31, type='Expense')
1383: Transaction(date='2024-08-14', description='Claim move.', category=Rent, amount=1839.16, type='Expense')
1384: Transaction(date='2024-08-15', description='Bed recognize store.', category=Health & Fitness, amount=701.48, type='Expense')
1385: Transaction(date='2024-08-22', description='There decision measure.', category=Entertainment, amount=1216.66, type='Expense')
1386: Transaction(date='2024-08-23', description='Play woman remain event.', category=Other, amount=538.0, type='Income')
1387: Transaction(date='2024-08-25', description='Decision where foreign trial.', category=Shopping, amount=1017.41, type='Expense')
1388: Transaction(date='2024-08-25', description='Participant above.', category=Travel, amount=686.53, type='Expense')
1389: Transaction(date='2024-08-25', description='Including time reflect.', category=Other, amount=4906.0, type='Income')
1390: Transaction(date='2024-08-27', description='Ball theory prepare much throughout.', category=Travel, amount=1391.44, type='Expense')
1391: Transaction(date='2024-08-27', description='Good spend still.', category=Salary, amount=1542.23, type='Expense')
1392: Transaction(date='2024-08-28', description='Concern should recent across.', category=Salary, amount=1366.37, type='Expense')
1393: Transaction(date='2024-08-28', description='View know develop after.', category=Food & Drink, amount=969.12, type='Expense')
1394: Transaction(date='2024-08-28', description='Course staff personal space.', category=Shopping, amount=1270.99, type='Expense')
1395: Transaction(date='2024-08-30', description='Woman whose already.', category=Entertainment, amount=1602.32, type='Expense')
1396: Transaction(date='2024-09-01', description='Decide cultural start how thank.', category=Health & Fitness, amount=1037.84, type='Expense')
1397: Transaction(date='2024-09-02', description='Management necessary process not.', category=Other, amount=2533.0, type='Income')
1398: Transaction(date='2024-09-05', description='Assume campaign assume.', category=Shopping, amount=1350.66, type='Expense')
1399: Transaction(date='2024-09-05', description='Talk thing rock ever summer.', category=Rent, amount=1113.56, type='Expense')
1400: Transaction(date='2024-09-05', description='Pass quite third ability interview.', category=Other, amount=3396.0, type='Income')
1401: Transaction(date='2024-09-05', description='Item yet ever quite.', category=Travel, amount=373.74, type='Expense')
1402: Transaction(date='2024-09-06', description='Guess service this different.', category=Entertainment, amount=1003.01, type='Expense')
1403: Transaction(date='2024-09-07', description='Mother first hot free.', category=Investment, amount=2674.0, type='Income')
1404: Transaction(date='2024-09-08', description='Wrong difficult.', category=Entertainment, amount=1671.53, type='Expense')
1405: Transaction(date='2024-09-10', description='True culture.', category=Health & Fitness, amount=320.43, type='Expense')
1406: Transaction(date='2024-09-13', description='As personal agreement.', category=Investment, amount=4535.0, type='Income')
1407: Transaction(date='2024-09-15', description='Or phone.', category=Food & Drink, amount=1697.54, type='Expense')
1408: Transaction(date='2024-09-16', description='East list when.', category=Health & Fitness, amount=989.76, type='Expense')
1409: Transaction(date='2024-09-17', description='Mind door authority outside.', category=Travel, amount=986.7, type='Expense')
1410: Transaction(date='2024-09-17', description='Personal often.', category=Travel, amount=801.78, type='Expense')
1411: Transaction(date='2024-09-17', description='True help.', category=Utilities, amount=117.75, type='Expense')
1412: Transaction(date='2024-09-18', description='Artist sort area.', category=Salary, amount=1879.91, type='Expense')
1413: Transaction(date='2024-09-21', description='Big argue.', category=Other, amount=3725.0, type='Income')
1414: Transaction(date='2024-09-22', description='Sure i pay.', category=Utilities, amount=919.82, type='Expense')
1415: Transaction(date='2024-09-23', description='Ten skin perhaps.', category=Other, amount=3085.0, type='Income')
1416: Transaction(date='2024-09-24', description='Less into per big.', category=Food & Drink, amount=1351.47, type='Expense')
1417: Transaction(date='2024-09-25', description='Military onto blue activity.', category=Rent, amount=657.62, type='Expense')
1418: Transaction(date='2024-09-26', description='Smile cover of pm eight.', category=Travel, amount=382.43, type='Expense')
1419: Transaction(date='2024-09-26', description='Right know during someone.', category=Travel, amount=947.98, type='Expense')
1420: Transaction(date='2024-09-27', description='Find job where.', category=Health & Fitness, amount=375.99, type='Expense')
1421: Transaction(date='2024-09-27', description='On must certainly.', category=Shopping, amount=1404.42, type='Expense')
1422: Transaction(date='2024-10-01', description='Owner fact wife.', category=Shopping, amount=566.46, type='Expense')
1423: Transaction(date='2024-10-01', description='Price certain down standard nice.', category=Investment, amount=2372.0, type='Income')
1424: Transaction(date='2024-10-02', description='Gas with policy cut.', category=Travel, amount=1610.13, type='Expense')
1425: Transaction(date='2024-10-02', description='Country little reality level.', category=Shopping, amount=400.17, type='Expense')
1426: Transaction(date='2024-10-03', description='Not friend federal forget bit.', category=Utilities, amount=951.24, type='Expense')
1427: Transaction(date='2024-10-04', description='Detail about development discover.', category=Other, amount=4592.0, type='Income')
1428: Transaction(date='2024-10-04', description='Lose shoulder spring.', category=Other, amount=4486.0, type='Income')
1429: Transaction(date='2024-10-07', description='Hundred step according.', category=Food & Drink, amount=802.82, type='Expense')
1430: Transaction(date='2024-10-08', description='Group never.', category=Food & Drink, amount=438.78, type='Expense')
1431: Transaction(date='2024-10-09', description='Key place tree.', category=Rent, amount=905.21, type='Expense')
1432: Transaction(date='2024-10-10', description='Wish current.', category=Health & Fitness, amount=727.01, type='Expense')
1433: Transaction(date='2024-10-10', description='Despite energy company.', category=Travel, amount=694.84, type='Expense')
1434: Transaction(date='2024-10-11', description='Condition again successful.', category=Health & Fitness, amount=1736.06, type='Expense')
1435: Transaction(date='2024-10-12', description='Story case.', category=Entertainment, amount=1157.94, type='Expense')
1436: Transaction(date='2024-10-15', description='Very figure drive special.', category=Utilities, amount=1958.95, type='Expense')
1437: Transaction(date='2024-10-17', description='With young think there assume.', category=Health & Fitness, amount=1349.65, type='Expense')
1438: Transaction(date='2024-10-17', description='Some court her.', category=Salary, amount=1780.37, type='Expense')
1439: Transaction(date='2024-10-18', description='Outside with each animal.', category=Travel, amount=932.87, type='Expense')
1440: Transaction(date='2024-10-19', description='Take family chance.', category=Utilities, amount=1284.76, type='Expense')
1441: Transaction(date='2024-10-19', description='Picture evidence.', category=Shopping, amount=1266.23, type='Expense')
1442: Transaction(date='2024-10-21', description='Why though decide bank likely.', category=Rent, amount=167.43, type='Expense')
1443: Transaction(date='2024-10-21', description='Meet seem population.', category=Health & Fitness, amount=640.59, type='Expense')
1444: Transaction(date='2024-10-21', description='Material less focus hear.', category=Travel, amount=1214.72, type='Expense')
1445: Transaction(date='2024-10-23', description='Listen ok point so.', category=Salary, amount=1564.14, type='Expense')
1446: Transaction(date='2024-10-24', description='Detail data tell.', category=Salary, amount=833.84, type='Expense')
1447: Transaction(date='2024-10-28', description='Fill thousand number.', category=Investment, amount=1524.0, type='Income')
1448: Transaction(date='2024-11-01', description='Nearly care base.', category=Travel, amount=1881.47, type='Expense')
1449: Transaction(date='2024-11-02', description='Goal once let perform international.', category=Rent, amount=429.04, type='Expense')
1450: Transaction(date='2024-11-03', description='Important source first him.', category=Rent, amount=136.95, type='Expense')
1451: Transaction(date='2024-11-03', description='Doctor population grow bar.', category=Food & Drink, amount=62.47, type='Expense')
1452: Transaction(date='2024-11-11', description='National back everybody direction budget.', category=Rent, amount=216.82, type='Expense')
1453: Transaction(date='2024-11-11', description='National free.', category=Rent, amount=1313.12, type='Expense')
1454: Transaction(date='2024-11-12', description='Few spring raise.', category=Travel, amount=466.42, type='Expense')
1455: Transaction(date='2024-11-13', description='Relationship your your.', category=Salary, amount=1256.7, type='Expense')
1456: Transaction(date='2024-11-13', description='Across federal state.', category=Investment, amount=734.0, type='Income')
1457: Transaction(date='2024-11-13', description='State serious activity pressure.', category=Other, amount=3164.0, type='Income')
1458: Transaction(date='2024-11-13', description='Dinner rich decide do.', category=Salary, amount=1950.21, type='Expense')
1459: Transaction(date='2024-11-14', description='Serious reveal left social fine.', category=Travel, amount=1101.61, type='Expense')
1460: Transaction(date='2024-11-14', description='Show recently decide last.', category=Investment, amount=4240.0, type='Income')
1461: Transaction(date='2024-11-16', description='Hear officer.', category=Shopping, amount=1051.69, type='Expense')
1462: Transaction(date='2024-11-17', description='Wall act special.', category=Travel, amount=1818.76, type='Expense')
1463: Transaction(date='2024-11-17', description='Different international sell several.', category=Entertainment, amount=1611.37, type='Expense')
1464: Transaction(date='2024-11-17', description='For age size.', category=Health & Fitness, amount=830.29, type='Expense')
1465: Transaction(date='2024-11-17', description='With yes nothing.', category=Investment, amount=500.0, type='Income')
1466: Transaction(date='2024-11-18', description='Woman daughter hospital move.', category=Utilities, amount=694.33, type='Expense')
1467: Transaction(date='2024-11-18', description='Board notice later thus.', category=Food & Drink, amount=1378.66, type='Expense')
1468: Transaction(date='2024-11-19', description='Bit need allow.', category=Salary, amount=606.22, type='Expense')
1469: Transaction(date='2024-11-20', description='Cost attorney.', category=Travel, amount=1198.04, type='Expense')
1470: Transaction(date='2024-11-20', description='Day fact modern.', category=Health & Fitness, amount=385.26, type='Expense')
1471: Transaction(date='2024-11-22', description='Happy action.', category=Shopping, amount=795.56, type='Expense')
1472: Transaction(date='2024-11-23', description='Hospital writer can industry.', category=Other, amount=3599.0, type='Income')
1473: Transaction(date='2024-11-24', description='Western tend candidate day.', category=Rent, amount=850.26, type='Expense')
1474: Transaction(date='2024-11-24', description='City offer financial data.', category=Travel, amount=576.51, type='Expense')
1475: Transaction(date='2024-11-24', description='Cut yard knowledge.', category=Travel, amount=831.68, type='Expense')
1476: Transaction(date='2024-11-25', description='Space moment travel these least.', category=Utilities, amount=281.88, type='Expense')
1477: Transaction(date='2024-11-25', description='Say explain.', category=Other, amount=4209.0, type='Income')
1478: Transaction(date='2024-11-25', description='Red republican father party.', category=Salary, amount=383.7, type='Expense')
1479: Transaction(date='2024-11-26', description='Eat tonight run leader.', category=Investment, amount=4882.0, type='Income')
1480: Transaction(date='2024-11-29', description='Believe recognize analysis those citizen.', category=Investment, amount=2811.0, type='Income')
1481: Transaction(date='2024-11-30', description='Project at risk.', category=Salary, amount=1456.94, type='Expense')
1482: Transaction(date='2024-12-02', description='In seven building hard.', category=Utilities, amount=39.3, type='Expense')
1483: Transaction(date='2024-12-05', description='Short for late education.', category=Shopping, amount=630.04, type='Expense')
1484: Transaction(date='2024-12-06', description='Left available.', category=Health & Fitness, amount=932.73, type='Expense')
1485: Transaction(date='2024-12-09', description='Gas human create also.', category=Health & Fitness, amount=169.64, type='Expense')
1486: Transaction(date='2024-12-11', description='Remember carry.', category=Utilities, amount=121.52, type='Expense')
1487: Transaction(date='2024-12-12', description='Leave bar prevent require.', category=Rent, amount=1090.95, type='Expense')
1488: Transaction(date='2024-12-12', description='Likely evidence claim.', category=Other, amount=3625.0, type='Income')
1489: Transaction(date='2024-12-15', description='Beyond family movement hospital back.', category=Rent, amount=1949.45, type='Expense')
1490: Transaction(date='2024-12-16', description='Child group describe.', category=Shopping, amount=1544.74, type='Expense')
1491: Transaction(date='2024-12-19', description='Democratic recent student specific political.', category=Food & Drink, amount=406.37, type='Expense')
1492: Transaction(date='2024-12-19', description='Real know our any.', category=Rent, amount=1047.07, type='Expense')
1493: Transaction(date='2024-12-21', description='Up hear draw.', category=Salary, amount=784.35, type='Expense')
1494: Transaction(date='2024-12-23', description='Instead writer.', category=Health & Fitness, amount=325.64, type='Expense')
1495: Transaction(date='2024-12-28', description='Quite as when.', category=Rent, amount=514.09, type='Expense')
1496: Transaction(date='2024-12-28', description='Right analysis mention.', category=Entertainment, amount=727.25, type='Expense')
1497: Transaction(date='2024-12-28', description='No couple debate must.', category=Investment, amount=1425.0, type='Income')
1498: Transaction(date='2024-12-29', description='Discussion black follow.', category=Shopping, amount=655.78, type='Expense')
1499: Transaction(date='2024-12-29', description='Pressure activity defense detail.', category=Other, amount=1480.0, type='Income')
1500: Transaction(date='1999-08-11', description='111111', category=Nitish, amount=2999.0, type='Expense')
1501: Transaction(date='1999-08-22', description='1111', category=2999, amount=111111.0, type='Expense')
1502: Transaction(date='1999-08-13', description='N121', category=Nitish, amount=2999.0, type='Expense')

--- Personal Finance Manager CLI ---
1. Add New Transaction
2. View All Transactions
3. Update Transaction
4. Delete Transaction
5. Generate Category Expense Report
6. Generate Income vs. Expense Report
7. Generate Monthly Spending Report
8. Get Top Spending Categories
9. Get Spending Trends Over Time
10. Get Average Monthly Expense
0. Save Data and Exit
Enter your choice: 3
Enter the index of the transaction to update: 999
Current transaction: Transaction(date='2023-05-15', description='Leave opportunity draw during city.', category=Travel, amount=263.42, type='Expense')
Enter new date (YYYY-MM-DD, leave blank to keep current): 1999-08-12
Enter new description (leave blank to keep current): N232
Enter new category name (leave blank to keep current): Satish
Enter new amount (leave blank to keep current): 12222
Enter new type (Income/Expense, leave blank to keep current): Income
Transaction at index 999 updated.

--- Personal Finance Manager CLI ---
1. Add New Transaction
2. View All Transactions
3. Update Transaction
4. Delete Transaction
5. Generate Category Expense Report
6. Generate Income vs. Expense Report
7. Generate Monthly Spending Report
8. Get Top Spending Categories
9. Get Spending Trends Over Time
10. Get Average Monthly Expense
0. Save Data and Exit
Enter your choice: 7

--- Monthly Spending Report ---
1999-08: $117109.00
2020-01: $15953.17
2020-02: $17108.41
2020-03: $13581.81
2020-04: $16233.05
2020-05: $16846.13
2020-06: $20680.72
2020-07: $24026.27
2020-08: $31386.80
2020-09: $21515.15
2020-10: $22162.99
2020-11: $26735.16
2020-12: $28358.12
2021-01: $15817.11
2021-02: $19925.23
2021-03: $26223.24
2021-04: $23097.71
2021-05: $23780.78
2021-06: $22137.63
2021-07: $19122.37
2021-08: $14574.03
2021-09: $17084.76
2021-10: $12559.57
2021-11: $14325.37
2021-12: $9996.66
2022-01: $20832.61
2022-02: $20042.69
2022-03: $27736.19
2022-04: $21445.01
2022-05: $31344.05
2022-06: $25557.78
2022-07: $19190.82
2022-08: $24246.00
2022-09: $18901.86
2022-10: $16412.30
2022-11: $19550.74
2022-12: $11516.30
2023-01: $28714.75
2023-02: $16870.03
2023-03: $16720.78
2023-04: $16726.65
2023-05: $17758.45
2023-06: $20070.77
2023-07: $26122.96
2023-08: $17372.59
2023-09: $25849.78
2023-10: $14541.21
2023-11: $22832.52
2023-12: $20654.33
2024-01: $29916.13
2024-02: $23807.30
2024-03: $16436.43
2024-04: $15443.17
2024-05: $15822.56
2024-06: $22554.07
2024-07: $27716.08
2024-08: $25933.69
2024-09: $19383.94
2024-10: $22984.21
2024-11: $23565.96
2024-12: $10938.92

--- Personal Finance Manager CLI ---
1. Add New Transaction
2. View All Transactions
3. Update Transaction
4. Delete Transaction
5. Generate Category Expense Report
6. Generate Income vs. Expense Report
7. Generate Monthly Spending Report
8. Get Top Spending Categories
9. Get Spending Trends Over Time
10. Get Average Monthly Expense
0. Save Data and Exit
Enter your choice: 8
Enter number of top categories (default 5): Food
Invalid input for number of categories.

--- Personal Finance Manager CLI ---
1. Add New Transaction
2. View All Transactions
3. Update Transaction
4. Delete Transaction
5. Generate Category Expense Report
6. Generate Income vs. Expense Report
7. Generate Monthly Spending Report
8. Get Top Spending Categories
9. Get Spending Trends Over Time
10. Get Average Monthly Expense
0. Save Data and Exit
Enter your choice: 9

--- Spending Trends Over Time ---
1999-08: $117109.00
2020-01: $15953.17
2020-02: $17108.41
2020-03: $13581.81
2020-04: $16233.05
2020-05: $16846.13
2020-06: $20680.72
2020-07: $24026.27
2020-08: $31386.80
2020-09: $21515.15
2020-10: $22162.99
2020-11: $26735.16
2020-12: $28358.12
2021-01: $15817.11
2021-02: $19925.23
2021-03: $26223.24
2021-04: $23097.71
2021-05: $23780.78
2021-06: $22137.63
2021-07: $19122.37
2021-08: $14574.03
2021-09: $17084.76
2021-10: $12559.57
2021-11: $14325.37
2021-12: $9996.66
2022-01: $20832.61
2022-02: $20042.69
2022-03: $27736.19
2022-04: $21445.01
2022-05: $31344.05
2022-06: $25557.78
2022-07: $19190.82
2022-08: $24246.00
2022-09: $18901.86
2022-10: $16412.30
2022-11: $19550.74
2022-12: $11516.30
2023-01: $28714.75
2023-02: $16870.03
2023-03: $16720.78
2023-04: $16726.65
2023-05: $17758.45
2023-06: $20070.77
2023-07: $26122.96
2023-08: $17372.59
2023-09: $25849.78
2023-10: $14541.21
2023-11: $22832.52
2023-12: $20654.33
2024-01: $29916.13
2024-02: $23807.30
2024-03: $16436.43
2024-04: $15443.17
2024-05: $15822.56
2024-06: $22554.07
2024-07: $27716.08
2024-08: $25933.69
2024-09: $19383.94
2024-10: $22984.21
2024-11: $23565.96
2024-12: $10938.92

--- Personal Finance Manager CLI ---
1. Add New Transaction
2. View All Transactions
3. Update Transaction
4. Delete Transaction
5. Generate Category Expense Report
6. Generate Income vs. Expense Report
7. Generate Monthly Spending Report
8. Get Top Spending Categories
9. Get Spending Trends Over Time
10. Get Average Monthly Expense
0. Save Data and Exit
Enter your choice: 0
Categories saved to categories.csv
Transactions saved to transactions.csv
Data saved. Exiting application.
