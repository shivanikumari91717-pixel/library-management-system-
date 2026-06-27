# library-management-system-
A python -based library management system to manage books and student records.
import json
import os
from pathlib import Path

# File path for storing books data
BOOKS_FILE = "data/books.json"

def ensure_data_directory():
    """Create data directory if it doesn't exist"""
    Path("data").mkdir(exist_ok=True)

def load_books():
    """Load books from JSON file"""
    ensure_data_directory()
    
    if not os.path.exists(BOOKS_FILE):
        return []
    
    try:
        with open(BOOKS_FILE, 'r', encoding='utf-8') as file:
            return json.load(file)
    except json.JSONDecodeError:
        print("Error: Corrupted books file. Starting fresh.")
        return []

def save_books(books):
    """Save books to JSON file"""
    ensure_data_directory()
    
    try:
        with open(BOOKS_FILE, 'w', encoding='utf-8') as file:
            json.dump(books, file, indent=4, ensure_ascii=False)
        return True
    except IOError as e:
        print(f"Error saving books: {e}")
        return False

def add_book(title, author, isbn, quantity=1):
    """
    Add a new book to the library
    
    Args:
        title (str): Book title
        author (str): Author name
        isbn (str): ISBN number
        quantity (int): Number of copies (default: 1)
    
    Returns:
        bool: True if successful, False otherwise
    """
    if not title or not author or not isbn:
        print("Error: Title, author, and ISBN are required.")
        return False
    
    books = load_books()
    
    # Check if book with same ISBN already exists
    for book in books:
        if book['isbn'] == isbn:
            print(f"Book with ISBN {isbn} already exists.")
            return False
    
    new_book = {
        'id': len(books) + 1,
        'title': title,
        'author': author,
        'isbn': isbn,
        'quantity': quantity
    }
    
    books.append(new_book)
    
    if save_books(books):
        print(f"✓ Book '{title}' added successfully!")
        return True
    return False

def delete_book(isbn):
    """
    Delete a book from the library by ISBN
    
    Args:
        isbn (str): ISBN of the book to delete
    
    Returns:
        bool: True if successful, False otherwise
    """
    books = load_books()
    
    for i, book in enumerate(books):
        if book['isbn'] == isbn:
            removed_book = books.pop(i)
            
            if save_books(books):
                print(f"✓ Book '{removed_book['title']}' deleted successfully!")
                return True
            return False
    
    print(f"Error: Book with ISBN {isbn} not found.")
    return False

def search_book(query):
    """
    Search for books by title, author, or ISBN
    
    Args:
        query (str): Search query
    
    Returns:
        list: List of matching books
    """
    books = load_books()
    query_lower = query.lower()
    
    results = [
        book for book in books
        if query_lower in book['title'].lower()
        or query_lower in book['author'].lower()
        or query_lower in book['isbn'].lower()
    ]
    
    return results

def display_all_books():
    """
    Display all books in the library
    
    Returns:
        list: List of all books
    """
    books = load_books()
    return books

def get_book_by_isbn(isbn):
    """
    Get book details by ISBN
    
    Args:
        isbn (str): ISBN of the book
    
    Returns:
        dict: Book details or None if not found
    """
    books = load_books()
    for book in books:
        if book['isbn'] == isbn:
            return book
    return None

def update_book_quantity(isbn, quantity):
    """
    Update the quantity of a book
    
    Args:
        isbn (str): ISBN of the book
        quantity (int): New quantity
    
    Returns:
        bool: True if successful, False otherwise
    """
    books = load_books()
    
    for book in books:
        if book['isbn'] == isbn:
            if quantity < 0:
                print("Error: Quantity cannot be negative.")
                return False
            book['quantity'] = quantity
            
            if save_books(books):
                print(f"✓ Quantity updated for '{book['title']}'")
                return True
            return False
    
    print(f"Error: Book with ISBN {isbn} not found.")
    return False
