import tkinter as tk
from tkinter import messagebox
from tkinter import ttk
import matplotlib.pyplot as plt

data = []

def add_entry():
    amount = entry_amount.get()
    category = entry_category.get()
    description = entry_description.get()
    if not amount or not category:
        messagebox.showwarning("Input Error", "Amount and Category are required!")
        return
    try:
        amount = float(amount)
        data.append({"amount": amount, "category": category, "description": description})
        update_summary()
        entry_amount.delete(0, tk.END)
        entry_category.delete(0, tk.END)
        entry_description.delete(0, tk.END)
    except ValueError:
        messagebox.showerror("Input Error", "Please enter a valid number for the amount.")

def update_summary():
    total_income = sum(item["amount"] for item in data if item["amount"] > 0)
    total_expense = sum(-item["amount"] for item in data if item["amount"] < 0)
    balance = total_income - total_expense
    label_income.config(text=f"Total Income: ₹{total_income:.2f}")
    label_expense.config(text=f"Total Expense: ₹{total_expense:.2f}")
    label_balance.config(text=f"Balance: ₹{balance:.2f}")

def show_chart():
    if not data:
        messagebox.showinfo("No Data", "No data to display!")
        return
    categories = {}
    for item in data:
        if item["amount"] < 0:
            cat = item["category"]
            categories[cat] = categories.get(cat, 0) - item["amount"]
    if not categories:
        messagebox.showinfo("No Data", "No expenses to display!")
        return
    labels = categories.keys()
    sizes = categories.values()
    plt.pie(sizes, labels=labels, autopct="%1.1f%%", startangle=140)
    plt.title("Expense Distribution")
    plt.show()

def save_data():
    with open("finance_data.txt", "w") as file:
        for item in data:
            file.write(f'{item["amount"]},{item["category"]},{item["description"]}\n')
    messagebox.showinfo("Success", "Data saved to finance_data.txt")

def view_saved_data():
    try:
        with open("finance_data.txt", "r") as file:
            saved_data = file.readlines()
        if not saved_data:
            messagebox.showinfo("No Data", "No saved data available!")
            return
        view_window = tk.Toplevel(root)
        view_window.title("Saved Data")
        view_window.geometry("400x400")
        tk.Label(view_window, text="Saved Entries:", font=("Arial", 14)).pack(pady=10)
        text_area = tk.Text(view_window, wrap=tk.WORD, font=("Arial", 10))
        text_area.pack(expand=True, fill=tk.BOTH, padx=10, pady=10)
        for line in saved_data:
            text_area.insert(tk.END, line)
        text_area.config(state=tk.DISABLED)
    except FileNotFoundError:
        messagebox.showerror("Error", "No saved data file found!")

root = tk.Tk()
root.title("Personal Finance Tracker")
root.geometry("500x500")

tk.Label(root, text="Amount (use negative for expense):").pack(pady=5)
entry_amount = tk.Entry(root)
entry_amount.pack(pady=5)

tk.Label(root, text="Category:").pack(pady=5)
entry_category = tk.Entry(root)
entry_category.pack(pady=5)

tk.Label(root, text="Description (optional):").pack(pady=5)
entry_description = tk.Entry(root)
entry_description.pack(pady=5)

tk.Button(root, text="Add Entry", command=add_entry).pack(pady=10)

label_income = tk.Label(root, text="Total Income: ₹0.00", font=("Arial", 12))
label_income.pack()

label_expense = tk.Label(root, text="Total Expense: ₹0.00", font=("Arial", 12))
label_expense.pack()

label_balance = tk.Label(root, text="Balance: ₹0.00", font=("Arial", 12))
label_balance.pack()

tk.Button(root, text="Show Expense Chart", command=show_chart).pack(pady=10)
tk.Button(root, text="Save Data", command=save_data).pack(pady=10)
tk.Button(root, text="View Saved Data", command=view_saved_data).pack(pady=10)

root.mainloop()
