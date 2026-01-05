# Personal-finance-tracker1
Personal finance tracker
import json
import os
import shutil
import csv
from datetime import datetime
from collections import defaultdict

DATA_FILE = "expenses.json"
BACKUP_FOLDER = "backups"

# ---------------- VALIDATION ---------------- #
def validate_amount(amount):
    try:
        amount = float(amount)
        if amount <= 0:
            raise ValueError
        return amount
    except ValueError:
        raise ValueError("Amount must be a positive number.")

def validate_date(date_str):
    try:
        datetime.strptime(date_str, "%Y-%m-%d")
        return date_str
    except ValueError:
        raise ValueError("Date must be YYYY-MM-DD format.")

def validate_category(category):
    if not category.strip():
        raise ValueError("Category cannot be empty.")
    return category.strip()

# ---------------- FILE HANDLING ---------------- #
def load_expenses():
    if not os.path.exists(DATA_FILE):
        return []
    try:
        with open(DATA_FILE, "r") as file:
            return json.load(file)
    except Exception:
        return []

def save_expenses(expenses):
    with open(DATA_FILE, "w") as file:
        json.dump(expenses, file, indent=4)

# ---------------- EXPENSE OPERATIONS ---------------- #
def add_expense(expenses):
    date = validate_date(input("Enter date (YYYY-MM-DD): "))
    category = validate_category(input("Enter category: "))
    amount = validate_amount(input("Enter amount: "))
    description = input("Description: ")

    expenses.append({
        "date": date,
        "category": category,
        "amount": amount,
        "description": description
    })
    save_expenses(expenses)
    print("✅ Expense added successfully")

def search_by_category(expenses):
    category = input("Enter category to search: ").lower()
    results = [e for e in expenses if e["category"].lower() == category]

    if not results:
        print("No records found.")
    else:
        for e in results:
            print(e)

def filter_by_month(expenses):
    month = input("Enter month (MM): ")
    filtered = [e for e in expenses if e["date"][5:7] == month]

    if not filtered:
        print("No expenses for this month.")
    else:
        for e in filtered:
            print(e)

# ---------------- REPORTS ---------------- #
def monthly_report(expenses):
    month = input("Enter month (MM): ")
    filtered = [e for e in expenses if e["date"][5:7] == month]

    total = sum(e["amount"] for e in filtered)
    category_summary = defaultdict(float)

    for e in filtered:
        category_summary[e["category"]] += e["amount"]

    print("\n📊 Monthly Report")
    print("Total Expense:", total)
    for cat, amt in category_summary.items():
        print(f"{cat}: {amt}")

# ---------------- CSV EXPORT ---------------- #
def export_to_csv(expenses):
    if not expenses:
        print("No expenses to export.")
        return

    filename = input("Enter CSV filename (without .csv): ") + ".csv"

    try:
        with open(filename, "w", newline="") as csvfile:
            writer = csv.DictWriter(
                csvfile,
                fieldnames=["date", "category", "amount", "description"]
            )
            writer.writeheader()
            writer.writerows(expenses)

        print(f"📁 Expenses exported successfully to {filename}")

    except Exception as e:
        print("CSV Export failed:", e)

# ---------------- BACKUP ---------------- #
def create_backup():
    os.makedirs(BACKUP_FOLDER, exist_ok=True)
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    backup_file = f"{BACKUP_FOLDER}/expenses_{timestamp}.json"

    if os.path.exists(DATA_FILE):
        shutil.copy(DATA_FILE, backup_file)
        print("📁 Backup created successfully")
    else:
        print("No data file to backup.")

# ---------------- MENU ---------------- #
def menu():
    print("\n==== Personal Finance Tracker ====")
    print("1. Add Expense")
    print("2. Search by Category")
    print("3. Filter by Month")
    print("4. Monthly Report")
    print("5. Backup Data")
    print("6. Export to CSV")
    print("7. Exit")

# ---------------- MAIN ---------------- #
def main():
    expenses = load_expenses()

    while True:
        try:
            menu()
            choice = input("Choose option: ")

            if choice == "1":
                add_expense(expenses)
            elif choice == "2":
                search_by_category(expenses)
            elif choice == "3":
                filter_by_month(expenses)
            elif choice == "4":
                monthly_report(expenses)
            elif choice == "5":
                create_backup()
            elif choice == "6":
                export_to_csv(expenses)
            elif choice == "7":
                print("👋 Exiting... Thank you!")
                break
            else:
                print("❌ Invalid choice")

        except Exception as e:
            print("⚠ Error:", e)

if __name__ == "__main__":
    main()
