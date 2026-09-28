# Student-Registration-System
This project is a Student Result Management System developed using Python. It allows a user to enter student details, calculate results, save records in an Excel file, and search for a student’s result using their roll number. The program provides a graphical user interface, so the user can enter information through text boxes and perform actions 
import tkinter as tk
from tkinter import ttk, messagebox
import mysql.connector


# ---------------- DATABASE CONNECTION ----------------

def connect_database():
    try:
        conn = mysql.connector.connect(
            host="localhost",
            user="root",
            password="YOUR_PASSWORD",
            database="student_db"
        )
        return conn

    except mysql.connector.Error as e:
        messagebox.showerror("Database Error", str(e))
        return None


# ---------------- CLEAR FIELDS ----------------

def clear_fields():
    roll_entry.delete(0, tk.END)
    name_entry.delete(0, tk.END)
    email_entry.delete(0, tk.END)
    phone_entry.delete(0, tk.END)
    course_entry.delete(0, tk.END)
    year_entry.delete(0, tk.END)
    address_entry.delete(0, tk.END)


# ---------------- ADD STUDENT ----------------

def add_student():

    roll = roll_entry.get()
    name = name_entry.get()
    email = email_entry.get()
    phone = phone_entry.get()
    course = course_entry.get()
    year = year_entry.get()
    address = address_entry.get()

    if roll == "" or name == "":
        messagebox.showwarning(
            "Input Error",
            "Roll Number and Name are required."
        )
        return

    conn = connect_database()

    if conn:
        cursor = conn.cursor()

        try:
            query = """
            INSERT INTO students
            (roll_no, name, email, phone, course, year, address)
            VALUES (%s, %s, %s, %s, %s, %s, %s)
            """

            values = (
                roll,
                name,
                email,
                phone,
                course,
                year,
                address
            )

            cursor.execute(query, values)
            conn.commit()

            messagebox.showinfo(
                "Success",
                "Student registered successfully!"
            )

            clear_fields()
            show_students()

        except mysql.connector.Error as e:
            messagebox.showerror("Error", str(e))

        finally:
            cursor.close()
            conn.close()


# ---------------- SHOW STUDENTS ----------------

def show_students():

    for item in table.get_children():
        table.delete(item)

    conn = connect_database()

    if conn:
        cursor = conn.cursor()

        cursor.execute("SELECT * FROM students")

        records = cursor.fetchall()

        for record in records:
            table.insert("", tk.END, values=record)

        cursor.close()
        conn.close()


# ---------------- SEARCH STUDENT ----------------

def search_student():

    roll = roll_entry.get()

    if roll == "":
        messagebox.showwarning(
            "Input Error",
            "Enter Roll Number to search."
        )
        return

    conn = connect_database()

    if conn:
        cursor = conn.cursor()

        query = "SELECT * FROM students WHERE roll_no = %s"

        cursor.execute(query, (roll,))

        record = cursor.fetchone()

        if record:

            clear_fields()

            roll_entry.insert(0, record[1])
            name_entry.insert(0, record[2])
            email_entry.insert(0, record[3])
            phone_entry.insert(0, record[4])
            course_entry.insert(0, record[5])
            year_entry.insert(0, record[6])
            address_entry.insert(0, record[7])

        else:
            messagebox.showinfo(
                "Not Found",
                "Student not found."
            )

        cursor.close()
        conn.close()


# ---------------- UPDATE STUDENT ----------------

def update_student():

    roll = roll_entry.get()
    name = name_entry.get()
    email = email_entry.get()
    phone = phone_entry.get()
    course = course_entry.get()
    year = year_entry.get()
    address = address_entry.get()

    if roll == "":
        messagebox.showwarning(
            "Input Error",
            "Enter Roll Number."
        )
        return

    conn = connect_database()

    if conn:
        cursor = conn.cursor()

        try:

            query = """
            UPDATE students
            SET name=%s,
                email=%s,
                phone=%s,
                course=%s,
                year=%s,
                address=%s
            WHERE roll_no=%s
            """

            values = (
                name,
                email,
                phone,
                course,
                year,
                address,
                roll
            )

            cursor.execute(query, values)
            conn.commit()

            if cursor.rowcount > 0:
                messagebox.showinfo(
                    "Success",
                    "Student details updated."
                )
            else:
                messagebox.showinfo(
                    "Not Found",
                    "Student not found."
                )

            show_students()

        except mysql.connector.Error as e:
            messagebox.showerror("Error", str(e))

        finally:
            cursor.close()
            conn.close()


# ---------------- DELETE STUDENT ----------------

def delete_student():

    roll = roll_entry.get()

    if roll == "":
        messagebox.showwarning(
            "Input Error",
            "Enter Roll Number."
        )
        return

    answer = messagebox.askyesno(
        "Confirm Delete",
        "Do you want to delete this student?"
    )

    if answer:

        conn = connect_database()

        if conn:
            cursor = conn.cursor()

            try:

                query = "DELETE FROM students WHERE roll_no=%s"

                cursor.execute(query, (roll,))
                conn.commit()

                if cursor.rowcount > 0:
                    messagebox.showinfo(
                        "Success",
                        "Student deleted successfully."
                    )
                    clear_fields()
                    show_students()
                else:
                    messagebox.showinfo(
                        "Not Found",
                        "Student not found."
                    )

            except mysql.connector.Error as e:
                messagebox.showerror("Error", str(e))

            finally:
                cursor.close()
                conn.close()


# ---------------- MAIN WINDOW ----------------

root = tk.Tk()

root.title("Student Registration System")

root.geometry("1100x650")

root.resizable(False, False)


# ---------------- TITLE ----------------

title = tk.Label(
    root,
    text="Student Registration System",
    font=("Arial", 24, "bold")
)

title.pack(pady=15)


# ---------------- FORM FRAME ----------------

form_frame = tk.Frame(root)

form_frame.pack(pady=10)


# Roll Number

tk.Label(
    form_frame,
    text="Roll Number",
    font=("Arial", 12)
).grid(row=0, column=0, padx=10, pady=8)

roll_entry = tk.Entry(
    form_frame,
    width=30,
    font=("Arial", 12)
)

roll_entry.grid(row=0, column=1, padx=10)


# Name

tk.Label(
    form_frame,
    text="Name",
    font=("Arial", 12)
).grid(row=1, column=0, padx=10, pady=8)

name_entry = tk.Entry(
    form_frame,
    width=30,
    font=("Arial", 12)
)

name_entry.grid(row=1, column=1, padx=10)


# Email

tk.Label(
    form_frame,
    text="Email",
    font=("Arial", 12)
).grid(row=2, column=0, padx=10, pady=8)

email_entry = tk.Entry(
    form_frame,
    width=30,
    font=("Arial", 12)
)

email_entry.grid(row=2, column=1, padx=10)


# Phone

tk.Label(
    form_frame,
    text="Phone",
    font=("Arial", 12)
).grid(row=3, column=0, padx=10, pady=8)

phone_entry = tk.Entry(
    form_frame,
    width=30,
    font=("Arial", 12)
)

phone_entry.grid(row=3, column=1, padx=10)


# Course

tk.Label(
    form_frame,
    text="Course",
    font=("Arial", 12)
).grid(row=0, column=2, padx=10, pady=8)

course_entry = tk.Entry(
    form_frame,
    width=30,
    font=("Arial", 12)
)

course_entry.grid(row=0, column=3, padx=10)


# Year

tk.Label(
    form_frame,
    text="Year",
    font=("Arial", 12)
).grid(row=1, column=2, padx=10, pady=8)

year_entry = tk.Entry(
    form_frame,
    width=30,
    font=("Arial", 12)
)

year_entry.grid(row=1, column=3, padx=10)


# Address

tk.Label(
    form_frame,
    text="Address",
    font=("Arial", 12)
).grid(row=2, column=2, padx=10, pady=8)

address_entry = tk.Entry(
    form_frame,
    width=30,
    font=("Arial", 12)
)

address_entry.grid(row=2, column=3, padx=10)


# ---------------- BUTTONS ----------------

button_frame = tk.Frame(root)

button_frame.pack(pady=15)


tk.Button(
    button_frame,
    text="Add Student",
    width=15,
    command=add_student
).grid(row=0, column=0, padx=5)


tk.Button(
    button_frame,
    text="Search",
    width=15,
    command=search_student
).grid(row=0, column=1, padx=5)


tk.Button(
    button_frame,
    text="Update",
    width=15,
    command=update_student
).grid(row=0, column=2, padx=5)


tk.Button(
    button_frame,
    text="Delete",
    width=15,
    command=delete_student
).grid(row=0, column=3, padx=5)


tk.Button(
    button_frame,
    text="Clear",
    width=15,
    command=clear_fields
).grid(row=0, column=4, padx=5)


tk.Button(
    button_frame,
    text="Refresh",
    width=15,
    command=show_students
).grid(row=0, column=5, padx=5)


# ---------------- TABLE ----------------

table_frame = tk.Frame(root)

table_frame.pack(pady=10)


columns = (
    "ID",
    "Roll No",
    "Name",
    "Email",
    "Phone",
    "Course",
    "Year",
    "Address"
)


table = ttk.Treeview(
    table_frame,
    columns=columns,
    show="headings",
    height=12
)


for column in columns:

    table.heading(
        column,
        text=column
    )

    table.column(
        column,
        width=120
    )


table.pack(side=tk.LEFT)


# Scrollbar

scrollbar = ttk.Scrollbar(
    table_frame,
    orient=tk.VERTICAL,
    command=table.yview
)

table.configure(
    yscrollcommand=scrollbar.set
)

scrollbar.pack(side=tk.RIGHT, fill=tk.Y)


# ---------------- RUN PROGRAM ----------------

show_students()

root.mainloop()
