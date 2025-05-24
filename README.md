"# Conference Management System" 
import tkinter as tk
from tkinter import ttk, messagebox
from tkcalendar import DateEntry
import mysql.connector
from datetime import datetime

# Database Connection
def connect_db():
    return mysql.connector.connect(
        host="localhost",
        user="root",
        password="aparnar8843@*",
        database="conproj3"
    )

# ---------------------- LOGIN PAGE ----------------------
def login_page():
    root.withdraw()
    login_win = tk.Toplevel(root)
    login_win.title("Login")
    login_win.geometry("400x300")
    login_win.configure(bg="#AED6F1")

    tk.Label(login_win, text="Username:", bg="#AED6F1").pack(pady=5)
    username_entry = tk.Entry(login_win)
    username_entry.pack(pady=5)

    tk.Label(login_win, text="Password:", bg="#AED6F1").pack(pady=5)
    password_entry = tk.Entry(login_win, show="*")
    password_entry.pack(pady=5)

    def check_login():
        username = username_entry.get()
        password = password_entry.get()
        conn = connect_db()
        cursor = conn.cursor()
        cursor.execute("SELECT * FROM users WHERE username=%s AND password=%s", (username, password))
        user = cursor.fetchone()
        conn.close()

        if user:
            messagebox.showinfo("Login Successful", "Welcome!")
            login_win.destroy()
            conference_page()
        else:
            messagebox.showerror("Error", "Invalid Credentials")

    tk.Button(login_win, text="Login", command=check_login, bg="#2E86C1", fg="white").pack(pady=10)
    tk.Button(login_win, text="Register", command=register_page, bg="#48C9B0", fg="white").pack(pady=5)

# ---------------------- REGISTRATION PAGE ----------------------
def register_page():
    reg_win = tk.Toplevel(root)
    reg_win.title("Register")
    reg_win.geometry("400x400")
    reg_win.configure(bg="#A9DFBF")

    tk.Label(reg_win, text="Username:", bg="#A9DFBF").pack(pady=5)
    username_entry = tk.Entry(reg_win)
    username_entry.pack(pady=5)

    tk.Label(reg_win, text="Email:", bg="#A9DFBF").pack(pady=5)
    email_entry = tk.Entry(reg_win)
    email_entry.pack(pady=5)

    tk.Label(reg_win, text="Password:", bg="#A9DFBF").pack(pady=5)
    password_entry = tk.Entry(reg_win, show="*")
    password_entry.pack(pady=5)

    def register():
        username = username_entry.get()
        email = email_entry.get()
        password = password_entry.get()

        if not username or not email or not password:
            messagebox.showerror("Error", "All fields are required")
            return

        conn = connect_db()
        cursor = conn.cursor()
        cursor.execute("INSERT INTO users (username, email, password) VALUES (%s, %s, %s)", (username, email, password))
        conn.commit()
        conn.close()
        messagebox.showinfo("Success", "Registration Successful")
        reg_win.destroy()

    tk.Button(reg_win, text="Register", command=register, bg="#27AE60", fg="white").pack(pady=10)

# ---------------------- CONFERENCE MANAGEMENT ----------------------
def conference_page():
    conf_win = tk.Toplevel(root)
    conf_win.title("Conference Management")
    conf_win.geometry("700x400")
    conf_win.configure(bg="#E0F7FA")

    frame = tk.Frame(conf_win, bg="#B2EBF2")
    frame.pack(pady=10, padx=10, fill=tk.X)

    tk.Label(frame, text="Title:", bg="#B2EBF2").grid(row=0, column=0, padx=5, pady=5, sticky=tk.W)
    title_entry = tk.Entry(frame)
    title_entry.grid(row=0, column=1, padx=5, pady=5)

    tk.Label(frame, text="Description:", bg="#B2EBF2").grid(row=1, column=0, padx=5, pady=5, sticky=tk.W)
    desc_entry = tk.Entry(frame)
    desc_entry.grid(row=1, column=1, padx=5, pady=5)

    tk.Label(frame, text="Date:", bg="#B2EBF2").grid(row=2, column=0, padx=5, pady=5, sticky=tk.W)
    date_entry = DateEntry(frame, width=12, background='darkblue', foreground='white', borderwidth=2, date_pattern='yyyy-MM-dd')
    date_entry.grid(row=2, column=1, padx=5, pady=5)

    tk.Label(frame, text="Venue:", bg="#B2EBF2").grid(row=3, column=0, padx=5, pady=5, sticky=tk.W)
    venue_entry = tk.Entry(frame)
    venue_entry.grid(row=3, column=1, padx=5, pady=5)

    tree = ttk.Treeview(conf_win, columns=("ID", "Title", "Description", "Date", "Venue"), show="headings")
    tree.heading("ID", text="ID")
    tree.heading("Title", text="Title")
    tree.heading("Description", text="Description")
    tree.heading("Date", text="Date")
    tree.heading("Venue", text="Venue")
    tree.pack(fill=tk.BOTH, expand=True)

    def load_conferences():
        for i in tree.get_children():
            tree.delete(i)

        conn = connect_db()
        cursor = conn.cursor()
        cursor.execute("SELECT * FROM conferences")
        for row in cursor.fetchall():
            tree.insert("", "end", values=row)
        conn.close()

    def add_conference():
        title = title_entry.get()
        description = desc_entry.get()
        date = date_entry.get_date().strftime('%Y-%m-%d')
        venue = venue_entry.get()

        conn = connect_db()
        cursor = conn.cursor()
        cursor.execute("INSERT INTO conferences (title, description, date, venue) VALUES (%s, %s, %s, %s)", 
                       (title, description, date, venue))
        conn.commit()
        conn.close()
        load_conferences()
        messagebox.showinfo("Success", "Conference Added")

    def update_conference():
        selected_item = tree.selection()
        if not selected_item:
            messagebox.showerror("Error", "Select a conference to update")
            return

        conf_id = tree.item(selected_item)['values'][0]
        title = title_entry.get()
        description = desc_entry.get()
        date = date_entry.get_date().strftime('%Y-%m-%d')
        venue = venue_entry.get()

        conn = connect_db()
        cursor = conn.cursor()
        cursor.execute("UPDATE conferences SET title=%s, description=%s, date=%s, venue=%s WHERE id=%s",
                       (title, description, date, venue, conf_id))
        conn.commit()
        conn.close()
        load_conferences()
        messagebox.showinfo("Success", "Conference Updated")

    def delete_conference():
        selected_item = tree.selection()
        if not selected_item:
            messagebox.showerror("Error", "Select a conference to delete")
            return

        conf_id = tree.item(selected_item)['values'][0]
        conn = connect_db()
        cursor = conn.cursor()
        cursor.execute("DELETE FROM conferences WHERE id=%s", (conf_id,))
        conn.commit()
        conn.close()
        load_conferences()
        messagebox.showinfo("Success", "Conference Deleted")

    tk.Button(frame, text="Add", command=add_conference, bg="green", fg="white").grid(row=4, column=0, pady=10)
    tk.Button(frame, text="Update", command=update_conference, bg="blue", fg="white").grid(row=4, column=1, pady=10)
    tk.Button(frame, text="Delete", command=delete_conference, bg="red", fg="white").grid(row=4, column=2, pady=10)

    load_conferences()

# ---------------------- MAIN WINDOW ----------------------
root = tk.Tk()
root.title("Conference System")
root.geometry("400x200")
tk.Button(root, text="Login", command=login_page).pack(pady=50)
root.mainloop()
