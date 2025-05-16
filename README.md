import tkinter as tk
from tkinter import messagebox

# ------------------ Library Classes ------------------

class Book:
    def __init__(self, title, author, isbn):
        self.title = title
        self.author = author
        self.isbn = isbn
        self.is_lent = False

    def __str__(self):
        return f"{self.title} by {self.author} (ISBN: {self.isbn})"

class BookNotAvailableError(Exception):
    pass

class Library:
    def __init__(self):
        self.books = []

    def add_book(self, book):
        self.books.append(book)

    def lend_book(self, isbn):
        for book in self.books:
            if book.isbn == isbn and not book.is_lent:
                book.is_lent = True
                return book
        raise BookNotAvailableError("Book is not available.")

    def return_book(self, isbn):
        for book in self.books:
            if book.isbn == isbn:
                book.is_lent = False

    def __iter__(self):
        return (book for book in self.books if not book.is_lent)

class Ebook(Book):
    def __init__(self, title, author, isbn, download_size):
        super().__init__(title, author, isbn)
        self.download_size = download_size

    def __str__(self):
        return f"{super().__str__()} - {self.download_size}MB"

class DigitalLibrary(Library):
    def __init__(self):
        super().__init__()
        self.ebooks = []

    def add_ebook(self, title, author, isbn, download_size):
        ebook = Ebook(title, author, isbn, download_size)
        self.ebooks.append(ebook)
        self.books.append(ebook)

# ------------------ GUI Code ------------------

lib = DigitalLibrary()

# Create window
root = tk.Tk()
root.title("Library Management System")
root.geometry("500x500")

# Title
tk.Label(root, text="Title:").grid(row=0, column=0, padx=10, pady=5, sticky='w')
title_entry = tk.Entry(root)
title_entry.grid(row=0, column=1, padx=10, pady=5)

# Author
tk.Label(root, text="Author:").grid(row=1, column=0, padx=10, pady=5, sticky='w')
author_entry = tk.Entry(root)
author_entry.grid(row=1, column=1, padx=10, pady=5)

# ISBN
tk.Label(root, text="ISBN:").grid(row=2, column=0, padx=10, pady=5, sticky='w')
isbn_entry = tk.Entry(root)
isbn_entry.grid(row=2, column=1, padx=10, pady=5)

# Ebook Checkbox
is_ebook = tk.BooleanVar()
ebook_checkbox = tk.Checkbutton(root, text="Is this an eBook?", variable=is_ebook)
ebook_checkbox.grid(row=3, column=0, padx=10, pady=5, sticky='w')

# Download Size
tk.Label(root, text="Download Size (MB):").grid(row=4, column=0, padx=10, pady=5, sticky='w')
download_entry = tk.Entry(root)
download_entry.grid(row=4, column=1, padx=10, pady=5)

# Enable/disable download size field
def toggle_download_entry():
    if is_ebook.get():
        download_entry.config(state='normal')
    else:
        download_entry.delete(0, tk.END)
        download_entry.config(state='disabled')

is_ebook.trace("w", lambda *args: toggle_download_entry())
toggle_download_entry()  # Initial state

# Add Book Button
def add_book():
    title = title_entry.get()
    author = author_entry.get()
    isbn = isbn_entry.get()

    if not (title and author and isbn):
        messagebox.showerror("Error", "Please fill in all required fields.")
        return

    if is_ebook.get():
        try:
            download_size = float(download_entry.get())
            lib.add_ebook(title, author, isbn, download_size)
        except ValueError:
            messagebox.showerror("Error", "Download size must be a number.")
            return
    else:
        lib.add_book(Book(title, author, isbn))

    messagebox.showinfo("Success", f"Book '{title}' added!")
    update_books_list()
    title_entry.delete(0, tk.END)
    author_entry.delete(0, tk.END)
    isbn_entry.delete(0, tk.END)
    download_entry.delete(0, tk.END)

tk.Button(root, text="Add Book", command=add_book).grid(row=5, column=0, columnspan=2, pady=10)

# Book List Display
tk.Label(root, text="Available Books:").grid(row=6, column=0, columnspan=2, pady=5)
books_text = tk.Text(root, height=10, width=50)
books_text.grid(row=7, column=0, columnspan=2, padx=10)

def update_books_list():
    books_text.delete("1.0", tk.END)
    for book in lib:
        books_text.insert(tk.END, str(book) + "\n")

update_books_list()

# Run the GUI loop
root.mainloop()
# LibraryGUI_Tkinter
