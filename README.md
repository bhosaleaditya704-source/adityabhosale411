# adityabhosale411import tkinter as tk
from tkinter import messagebox
from pymongo import MongoClient

# MongoDB Connection
client = MongoClient("mongodb://localhost:27017/")
db = client["person_db"]
collection = db["people"]

# GUI App
app = tk.Tk()
app.title("Personal Information Management System")
app.geometry("500x500")

# Fields for Personal Info
fields = ["Person ID", "Name", "Age", "Email", "Phone"]
entries = {}

# Create Labels and Entry Widgets
for idx, field in enumerate(fields):
    label = tk.Label(app, text=field)
    label.grid(row=idx, column=0, padx=10, pady=5, sticky=tk.W)

    entry = tk.Entry(app, width=30)
    entry.grid(row=idx, column=1, padx=10, pady=5)
    entries[field] = entry


def clear_fields():
    for entry in entries.values():
        entry.delete(0, tk.END)


def create_person():
    data = {
        "person_id": entries["Person ID"].get(),
        "name": entries["Name"].get(),
        "age": entries["Age"].get(),
        "email": entries["Email"].get(),
        "phone": entries["Phone"].get(),
    }
    if not data["person_id"]:
        messagebox.showerror("Error", "Person ID is required.")
        return

    if collection.find_one({"person_id": data["person_id"]}):
        messagebox.showerror("Error", "Person with this ID already exists.")
        return

    collection.insert_one(data)
    messagebox.showinfo("Success", "Person added successfully.")
    clear_fields()


def read_person():
    person_id = entries["Person ID"].get()
    if not person_id:
        messagebox.showerror("Error", "Person ID is required to read.")
        return

    person = collection.find_one({"person_id": person_id})
    if person:
        entries["Name"].delete(0, tk.END)
        entries["Name"].insert(0, person.get("name", ""))

        entries["Age"].delete(0, tk.END)
        entries["Age"].insert(0, person.get("age", ""))

        entries["Email"].delete(0, tk.END)
        entries["Email"].insert(0, person.get("email", ""))

        entries["Phone"].delete(0, tk.END)
        entries["Phone"].insert(0, person.get("phone", ""))
    else:
        messagebox.showerror("Error", "Person not found.")


def update_person():
    person_id = entries["Person ID"].get()
    if not person_id:
        messagebox.showerror("Error", "Person ID is required to update.")
        return

    new_data = {
        "name": entries["Name"].get(),
        "age": entries["Age"].get(),
        "email": entries["Email"].get(),
        "phone": entries["Phone"].get(),
    }

    result = collection.update_one({"person_id": person_id}, {"$set": new_data})
    if result.matched_count:
        messagebox.showinfo("Success", "Person updated successfully.")
    else:
        messagebox.showerror("Error", "Person not found.")
    clear_fields()


def delete_person():
    person_id = entries["Person ID"].get()
    if not person_id:
        messagebox.showerror("Error", "Person ID is required to delete.")
        return

    result = collection.delete_one({"person_id": person_id})
    if result.deleted_count:
        messagebox.showinfo("Success", "Person deleted successfully.")
    else:
        messagebox.showerror("Error", "Person not found.")
    clear_fields()


# Buttons
button_frame = tk.Frame(app)
button_frame.grid(row=6, column=0, columnspan=2, pady=20)

tk.Button(button_frame, text="Create", width=10, command=create_person).grid(row=0, column=0, padx=5)
tk.Button(button_frame, text="Read", width=10, command=read_person).grid(row=0, column=1, padx=5)
tk.Button(button_frame, text="Update", width=10, command=update_person).grid(row=0, column=2, padx=5)
tk.Button(button_frame, text="Delete", width=10, command=delete_person).grid(row=0, column=3, padx=5)

# Run the GUI loop
app.mainloop()
