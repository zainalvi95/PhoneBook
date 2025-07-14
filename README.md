

from tkinter import *
from tkinter import messagebox
from tkinter import ttk
import csv
import re

# Dictionary to store contacts (Name -> Phone, Email, Blocked, Favorite)
contacts = {}

def load_contacts():
    try:
        with open("contacts.csv", mode="r", newline="", encoding="utf-8") as file:
            reader = csv.DictReader(file)
            for row in reader:
                contacts[row["Name"]] = {
                    "Phone": row["Phone"],
                    "Email": row["Email"],
                    "Blocked": row["Blocked"] == "True",
                    "Favorite": row["Favorite"] == "True"
                }
    except FileNotFoundError:
        pass

# Save contacts to CSV
def save_contacts():
    with open("contacts.csv", mode="w", newline="", encoding="utf-8") as file:
        fieldnames = ["Name", "Phone", "Email", "Blocked", "Favorite"]
        writer = csv.DictWriter(file, fieldnames=fieldnames)
        writer.writeheader()
        for name, details in contacts.items():
            writer.writerow({
                "Name": name,
                "Phone": details["Phone"],
                "Email": details["Email"],
                "Blocked": details["Blocked"],
                "Favorite": details["Favorite"]
            })

# Function to add a contact


def add_contact():
    def save_contact():
        name = entry_name.get().strip()
        phone = entry_phone.get().strip()
        email = entry_email.get().strip()

        # Validation regex patterns
        name_pattern = r"^[a-zA-Z\s]+$"  # (only letters allowed)
        phone_pattern = r"^\d{11}$"  # Only numbers, exactly 11 digits
        email_pattern = r"^[\w\.-]+@[\w\.-]+\.\w+$"  # Valid email format

        # Validate name (only alphabetic characters allowed, no numbers or special characters)
        if not re.match(name_pattern, name):
            messagebox.showerror(
                "Error",
                "Invalid name! Name should consist of only letters."
            )
            return

        # Validate phone number (only digits allowed, exactly 11 digits)
        if not re.match(phone_pattern, phone):
            messagebox.showerror(
                "Error", "Invalid phone number! Phone number should be exactly 11 digits (numbers only)."
            )
            return

        # Validate email only if provided
        if email and not re.match(email_pattern, email):
            messagebox.showerror(
                "Error", "Invalid email! Please enter a valid email address."
            )
            return

        # Check for duplicate phone number
        for contact_name, details in contacts.items():
            if details["Phone"] == phone:
                messagebox.showerror(
                    "Error", f"Phone number '{phone}' is already associated with '{contact_name}'."
                )
                return

        # Save contact
        if name and phone:
            if name in contacts:
                messagebox.showerror("Error", f"Contact '{name}' already exists!")
                return
            contacts[name] = {
                "Phone": phone,
                "Email": email if email else "N/A",  # Save "N/A" if no email is provided
                "Blocked": False,
                "Favorite": False,
            }
            messagebox.showinfo("Success", f"Contact '{name}' saved!")
            add_window.destroy()
            save_contacts()
        else:
            messagebox.showerror("Error", "Name and Phone fields are required!")

    # Create add contact window
    add_window = Toplevel(root)
    add_window.geometry("350x300")
    add_window.title("Add Contact")

    # Name field
    Label(add_window, text="Name:", font="Arial 12").pack(pady=10)
    entry_name = Entry(add_window, font="Arial 12")
    entry_name.pack(pady=5)

    # Phone field
    Label(add_window, text="Phone:", font="Arial 12").pack(pady=10)
    entry_phone = Entry(add_window, font="Arial 12")
    entry_phone.pack(pady=5)

    # Email field (Optional)
    Label(add_window, text="Email (Optional):", font="Arial 12").pack(pady=10)
    entry_email = Entry(add_window, font="Arial 12")
    entry_email.pack(pady=5)

    # Save button
    Button(add_window, text="Save", bg="#2ECC71", fg="white", font="Arial 12 bold", command=save_contact).pack(
        pady=20
    )






# Function to display all contacts
def display_all_contacts():
    if not contacts:
        messagebox.showinfo("No Contacts", "No contacts to display.")
        return

    display_window = Toplevel(root)
    display_window.title("Contacts")
    display_window.geometry("800x400")

    frame=Frame(display_window)
    frame.pack(pady=10, padx=10)

    class BSTNode:
        def __init__(self, key, contact):
            self.key = key
            self.contact = contact
            self.left = None
            self.right = None

    class BST:
        def __init__(self):
            self.root = None

        def insert(self, key, contact):
            def _insert(node, key, contact):
                if not node:
                    return BSTNode(key, contact)
                if key < node.key:
                    node.left = _insert(node.left, key, contact)
                else:
                    node.right = _insert(node.right, key, contact)
                return node

            self.root = _insert(self.root, key, contact)

        def in_order_traversal(self):
            result = []

            def _in_order_traversal(node):
                if node:
                    _in_order_traversal(node.left)
                    result.append((node.key, node.contact))
                    _in_order_traversal(node.right)

            _in_order_traversal(self.root)
            return result

    # Construct BST and populate it with contacts
    bst = BST()
    for name, details in contacts.items():
        bst.insert(name, (name, details))

    # Perform in-order traversal to get sorted contacts by name
    sorted_contacts = bst.in_order_traversal()

    # Create Treeview to display contacts
    treeview = ttk.Treeview(frame)
    treeview["columns"] = ["Name", "Phone", "Email", "Blocked", "Favorite"]

    treeview.column("#0", width=0, stretch=False)
    treeview.column("Name", width=150, anchor="w")
    treeview.column("Phone", width=150, anchor="w")
    treeview.column("Email", width=200, anchor="w")
    treeview.column("Blocked", width=100, anchor="w")
    treeview.column("Favorite", width=100, anchor="w")

    treeview.heading("#0", text="", anchor="w")
    treeview.heading("Name", text="Name", anchor="w")
    treeview.heading("Phone", text="Phone", anchor="w")
    treeview.heading("Email", text="Email", anchor="w")
    treeview.heading("Blocked", text="Blocked", anchor="w")
    treeview.heading("Favorite", text="Favorite", anchor="w")

    # Insert sorted contacts into the Treeview
    for _, (name, details) in sorted_contacts:
        treeview.insert("", "end", values=(
            name,
            details["Phone"],
            details["Email"],
            "Yes" if details["Blocked"] else "No",
            "Yes" if details["Favorite"] else "No"
        ))

    style = ttk.Style()
    style.configure("Treeview.Heading", font=("Arial", 14, "bold"))
    style.configure("Treeview", font=("Arial", 12), rowheight=25)

    treeview.pack(expand=True, fill="both")

def search_contact():
    def perform_search():
        search_query = entry_search.get().strip()
        search_type = search_by.get()
        found = False

        # Validation for "Name" option
        if search_type == "Name":
            if any(char.isdigit() for char in search_query) or "@" in search_query:
                messagebox.showerror("Error", "Please enter a valid name (no numbers or email)!")
                return

        if not search_query:
            messagebox.showerror("Error", "Please enter a search query!")
            return

        for name, details in contacts.items():
            if (
                (search_type == "Name" and search_query.lower() in name.lower()) or
                (search_type == "Phone" and search_query in details['Phone']) or
                (search_type == "Email" and search_query.lower() in details['Email'].lower())
            ):
                messagebox.showinfo(
                    "Search Result",
                    f"Contact Found:\n\nName: {name}\nPhone: {details['Phone']}\nEmail: {details['Email']}"
                )
                found = True
                break

        if not found:
            messagebox.showerror("Error", f"No contact found with {search_type}: {search_query}")

    # Create search window
    search_window = Toplevel(root)
    search_window.geometry("400x250")
    search_window.title("Search Contact")

    # Search type dropdown
    Label(search_window, text="Search By:", font="Arial 12").pack(pady=10)
    search_by = StringVar(search_window)
    search_by.set("Name")  # Default option
    dropdown = ttk.Combobox(search_window, textvariable=search_by, font="Arial 12", state="readonly")
    dropdown["values"] = ("Name", "Phone", "Email")
    dropdown.pack(pady=5)

    # Search query input
    Label(search_window, text="Enter Search Query:", font="Arial 12").pack(pady=10)
    entry_search = Entry(search_window, font="Arial 12", width=30)
    entry_search.pack(pady=5)

    # Search button
    Button(
        search_window, text="Search", bg="#3498DB", fg="white",
        font="Arial 12 bold", command=perform_search
    ).pack(pady=20)


# Function to update a contact (modify the contact details)
# Function to save contacts to a CSV file
def save_contacts_to_csv():
    """Save updated contacts to a CSV file."""
    with open("contacts.csv", "w", newline="") as file:
        writer = csv.writer(file)
        writer.writerow(["Name", "Phone", "Email", "Blocked", "Favorite"])
        for name, details in contacts.items():
            writer.writerow([name, details["Phone"], details["Email"], details["Blocked"], details["Favorite"]])

# Function to load contacts from a CSV file
def load_contacts_from_csv():
    """Load contacts from a CSV file into the contacts dictionary."""
    try:
        with open("contacts.csv", "r") as file:
            reader = csv.DictReader(file)
            for row in reader:
                contacts[row["Name"]] = {
                    "Phone": row["Phone"],
                    "Email": row["Email"],
                    "Blocked": row["Blocked"] == "True",
                    "Favorite": row["Favorite"] == "True",
                }
    except FileNotFoundError:
        pass  # File doesn't exist yet

# Function to update a contact
def update_contact():
    def update_existing_contact():
        old_name = entry_name.get().strip()
        current_phone = entry_current_phone.get().strip()
        new_name = entry_new_name.get().strip()
        new_phone = entry_new_phone.get().strip()
        new_email = entry_new_email.get().strip()

        # Validation regex patterns
        name_pattern = r"^[a-zA-Z\s]+$"
        phone_pattern = r"^\d{11}$"
        email_pattern = r"^[\w\.-]+@[\w\.-]+\.\w+$"

        # Validate old name
        if not re.match(name_pattern, old_name):
            messagebox.showerror("Error", "Invalid name! Name should only contain letters and spaces.")
            return

        # Validate current phone number
        if not current_phone or not re.match(phone_pattern, current_phone):
            messagebox.showerror("Error", "Invalid current phone number! Phone number should be exactly 11 digits.")
            return

        # Validate new name (if provided)
        if new_name and not re.match(name_pattern, new_name):
            messagebox.showerror("Error", "Invalid new name! Name should only contain letters and spaces.")
            return

        # Validate new phone number (if provided)
        if new_phone and not re.match(phone_pattern, new_phone):
            messagebox.showerror("Error", "Invalid new phone number! Phone number should be exactly 11 digits.")
            return

        # Validate email only if provided
        if new_email and not re.match(email_pattern, new_email):
            messagebox.showerror("Error", "Invalid email! Please enter a valid email address.")
            return

        # Check if the old name exists in contacts
        if old_name not in contacts:
            messagebox.showerror("Error", f"Contact '{old_name}' not found!")
            return

        # Check if the current phone matches the stored phone for the contact
        if contacts[old_name]["Phone"] != current_phone:
            messagebox.showerror("Error", "Current phone number does not match the stored phone number for this contact.")
            return

        # Update the contact
        contact = contacts.pop(old_name)  # Remove old entry and update details
        updated_name = new_name if new_name else old_name
        updated_phone = new_phone if new_phone else contact["Phone"]
        updated_email = new_email if new_email else contact["Email"]

        contacts[updated_name] = {
            "Phone": updated_phone,
            "Email": updated_email,
            "Blocked": contact["Blocked"],
            "Favorite": contact["Favorite"],
        }

        # Save updates to the CSV file
        save_contacts_to_csv()

        messagebox.showinfo("Success", f"Contact '{updated_name}' updated!")
        update_window.destroy()

    update_window = Toplevel(root)
    update_window.geometry("450x500")
    update_window.title("Update Contact")

    Label(update_window, text="Current Name:", font="Arial 12").pack(pady=10)
    entry_name = Entry(update_window, font="Arial 12")
    entry_name.pack(pady=5)

    Label(update_window, text="Current Phone:", font="Arial 12").pack(pady=10)  # Mandatory field
    entry_current_phone = Entry(update_window, font="Arial 12")
    entry_current_phone.pack(pady=5)

    Label(update_window, text="New Name (Optional):", font="Arial 12").pack(pady=10)
    entry_new_name = Entry(update_window, font="Arial 12")
    entry_new_name.pack(pady=5)

    Label(update_window, text="New Phone (Optional):", font="Arial 12").pack(pady=10)  # Optional
    entry_new_phone = Entry(update_window, font="Arial 12")
    entry_new_phone.pack(pady=5)

    Label(update_window, text="New Email (Optional):", font="Arial 12").pack(pady=10)  # Optional
    entry_new_email = Entry(update_window, font="Arial 12")
    entry_new_email.pack(pady=5)

    Button(update_window, text="Update", bg="#F39C12", fg="white", font="Arial 12 bold", command=update_existing_contact).pack(pady=20)


# Function to delete a contact
def delete_contact():
    def perform_delete():
        name = entry_delete.get().strip()  # Trim whitespace from the input
        if name in contacts:
            # Show confirmation dialog
            confirm = messagebox.askyesno(
                "Confirm Delete", f"Are you sure you want to delete the contact '{name}'?"
            )
            if confirm:  # If the user confirms
                del contacts[name]  # Remove the contact from the dictionary
                save_contacts()  # Save the updated contacts to the CSV file
                messagebox.showinfo("Success", f"Contact '{name}' deleted!")
                delete_window.destroy()
            else:  # If the user cancels
                messagebox.showinfo("Cancelled", "Delete action cancelled.")
        else:
            messagebox.showerror("Error", "Contact not found.")

    delete_window = Toplevel(root)
    delete_window.geometry("300x150")
    delete_window.title("Delete Contact")

    Label(delete_window, text="Enter Name:", font="Arial 12").pack(pady=10)
    entry_delete = Entry(delete_window, font="Arial 12")
    entry_delete.pack(pady=5)

    Button(delete_window, text="Delete", bg="#E74C3C", fg="white", font="Arial 12 bold", command=perform_delete).pack(
        pady=20
    )


# Function to block a contact
def block_contact():
    def block_existing_contact():
        name = entry_block.get()
        if name in contacts:
            contacts[name]['Blocked'] = True
            messagebox.showinfo("Blocked", f"Contact {name} is now blocked!")
            block_window.destroy()
        else:
            messagebox.showerror("Error", "Contact not found.")

    block_window = Toplevel(root)
    block_window.geometry("300x150")
    block_window.title("Block Contact")

    Label(block_window, text="Enter Name to Block:", font="Arial 12").pack(pady=10)
    entry_block = Entry(block_window, font="Arial 12")
    entry_block.pack(pady=5)

    Button(block_window, text="Block", bg="#9B59B6", fg="white", font="Arial 12 bold", command=block_existing_contact).pack(pady=20)

# Function to unblock a contact
def unblock_contact():
    def unblock_existing_contact():
        name = entry_unblock.get()
        if name in contacts and contacts[name]['Blocked']:
            contacts[name]['Blocked'] = False
            messagebox.showinfo("Unblocked", f"Contact {name} is now unblocked!")
            unblock_window.destroy()
        else:
            messagebox.showerror("Error", "Contact not found or not blocked.")

    unblock_window = Toplevel(root)
    unblock_window.geometry("300x150")
    unblock_window.title("Unblock Contact")

    Label(unblock_window, text="Enter Name to Unblock:", font="Arial 12").pack(pady=10)
    entry_unblock = Entry(unblock_window, font="Arial 12")
    entry_unblock.pack(pady=5)

    Button(unblock_window, text="Unblock", bg="#2ECC71", fg="white", font="Arial 12 bold", command=unblock_existing_contact).pack(pady=20)

# Function to mark a contact as favorite
def favorite_contact():
    def favorite_existing_contact():
        name = entry_favorite.get()
        if name in contacts:
            contacts[name]['Favorite'] = True
            messagebox.showinfo("Favorite", f"Contact {name} is now marked as favorite!")
            favorite_window.destroy()
        else:
            messagebox.showerror("Error", "Contact not found.")

    favorite_window = Toplevel(root)
    favorite_window.geometry("300x150")
    favorite_window.title("Favorite Contact")

    Label(favorite_window, text="Enter Name to Favorite:", font="Arial 12").pack(pady=10)
    entry_favorite = Entry(favorite_window, font="Arial 12")
    entry_favorite.pack(pady=5)

    Button(favorite_window, text="Favorite", bg="#F39C12", fg="white", font="Arial 12 bold", command=favorite_existing_contact).pack(pady=20)

# Function to mark a contact as unfavorite
def unfavorite_contact():
    def unfavorite_existing_contact():
        name = entry_unfavorite.get()
        if name in contacts and contacts[name]['Favorite']:
            contacts[name]['Favorite'] = False
            messagebox.showinfo("Unfavorite", f"Contact {name} is now unfavorite!")
            unfavorite_window.destroy()
        else:
            messagebox.showerror("Error", "Contact not found or not marked as favorite.")

    unfavorite_window = Toplevel(root)
    unfavorite_window.geometry("300x150")
    unfavorite_window.title("Unfavorite Contact")

    Label(unfavorite_window, text="Enter Name to Unfavorite:", font="Arial 12").pack(pady=10)
    entry_unfavorite = Entry(unfavorite_window, font="Arial 12")
    entry_unfavorite.pack(pady=5)

    Button(unfavorite_window, text="Unfavorite", bg="#E74C3C", fg="white", font="Arial 12 bold", command=unfavorite_existing_contact).pack(pady=20)



# Function to display emergency numbers
def display_emergency_numbers():
    emergency_contacts = {
        "Ambulance": "115",
        "Police Emergency": "15",
        "Child Protection Welfare Bureau": "1121",
        "Rescue 1122": "042-99231701-2",
        "Firefighting": "16",
        "Punjab Women Helpline": "1043",
        "Provincial Disaster Management Authority (PDMA)": "1129",
        "Ambulance Edhi": "042-37847050, 042-37847060"
    }

    # Create a new window to display emergency numbers
    emergency_window = Toplevel(root)
    emergency_window.title("Emergency Numbers")
    emergency_window.geometry("500x400")

    # Header Label
    header_label = Label(
        emergency_window, text="Emergency Contacts", 
        font=("Arial", 16, "bold"), fg="#34495E", pady=10
    )
    header_label.pack()

    # Frame to hold the emergency numbers
    frame = Frame(emergency_window, padx=10, pady=10)
    frame.pack(expand=True, fill="both")

    # Create labels for each emergency service and number
    for service, number in emergency_contacts.items():
        service_label = Label(
            frame, text=f"{service}: ", 
            font=("Arial", 12, "bold"), anchor="w", justify="left"
        )
        service_label.pack(anchor="w", pady=5)

        number_label = Label(
            frame, text=number, 
            font=("Arial", 12), fg="blue", anchor="w", justify="left"
        )
        number_label.pack(anchor="w", pady=0)




# Tkinter root setup
root = Tk()
root.geometry("800x600")
root.title("Phonebook ")
root.config(bg="#ECF0F1")  # Light gray background

# Header
header = Label(root, text="Phonebook ", bg="#34495E", fg="white", font="Arial 24 bold", padx=10, pady=20)
header.pack(fill=X)

# Main buttons frame
frame = Frame(root, bg="#ECF0F1")
frame.pack(pady=10)

# Buttons for various functionalities
button_style = {"bg": "Orange", "fg": "BLack", "font": "Arial 12 bold", "height": 2, "width": 18}

Button(frame, text="Add Contact", **button_style, command=add_contact).grid(row=0, column=0, padx=20, pady=10)
Button(frame, text="Search Contact", **button_style, command=search_contact).grid(row=0, column=1, padx=20, pady=10)
Button(frame, text="Update Contact", **button_style, command=update_contact).grid(row=1, column=0, padx=20, pady=10)
Button(frame, text="Delete Contact", **button_style, command=delete_contact).grid(row=1, column=1, padx=20, pady=10)
Button(frame, text="Display All Contacts", **button_style, command=display_all_contacts).grid(row=4, column=0, padx=20, pady=10)
Button(frame, text="Block Contact", **button_style, command=block_contact).grid(row=2, column=0, padx=20, pady=10)
Button(frame, text="Unblock Contact", **button_style, command=unblock_contact).grid(row=2, column=1, padx=20, pady=10)
Button(frame, text="Favorite Contact", **button_style, command=favorite_contact).grid(row=3, column=0, padx=20, pady=10)
Button(frame, text="Unfavorite Contact", **button_style, command=unfavorite_contact).grid(row=3, column=1, padx=20, pady=10)
Button(frame, text="Emergency Numbers", **button_style, command=display_emergency_numbers).grid(row=4, column=1, padx=20,pady=10)
# Exit Button
exit_button = Button(root, text="Exit", bg="Blue", fg="white", font="Arial 12 bold", height=2, width=15, command=root.quit)
exit_button.pack(pady=20)


load_contacts()
# Mainloop
root.mainloop()







