try:
    from fpdf import FPDF
except ImportError:
    import subprocess
    import sys
    subprocess.check_call([sys.executable, "-m", "pip", "install", "fpdf"])
    from fpdf import FPDF
import tkinter as tk
from tkinter import filedialog, messagebox, ttk
import os

# Global variable to store the current file path
current_file = None

# Function to create a new file
def new_file():
    global current_file
    current_file = None
    text_box.delete("1.0", tk.END)

# Function to open an existing file
def open_file():
    global current_file
    file_path = filedialog.askopenfilename(
        filetypes=[("Text Files", "*.txt"), ("All Files", "*.*")]
    )
    if file_path:
        current_file = file_path
        with open(file_path, "r", encoding="utf-8") as file:
            text_box.delete("1.0", tk.END)
            text_box.insert(tk.END, file.read())

# Function to save the current file
def save_file():
    global current_file
    if current_file:
        with open(current_file, "w", encoding="utf-8") as file:
            file.write(text_box.get("1.0", tk.END).strip())
        messagebox.showinfo("Success", "File saved successfully!")
    else:
        save_as_file()

# Function to save the file as a new file
def save_as_file():
    global current_file
    file_path = filedialog.asksaveasfilename(
        defaultextension=".txt",
        filetypes=[("Text Files", "*.txt"), ("All Files", "*.*")]
    )
    if file_path:
        current_file = file_path
        with open(file_path, "w", encoding="utf-8") as file:
            file.write(text_box.get("1.0", tk.END).strip())
        messagebox.showinfo("Success", "File saved successfully!")

# Function to generate the PDF
def generate_pdf():
    content = text_box.get("1.0", tk.END).strip()
    if not content:
        messagebox.showerror("Error", "Text box is empty. Please enter some text.")
        return

    file_path = filedialog.asksaveasfilename(
        defaultextension=".pdf",
        filetypes=[("PDF files", "*.pdf")],
        title="Save PDF As"
    )
    if not file_path:
        return

    try:
        pdf = FPDF()
        pdf.add_page()
        pdf.set_auto_page_break(auto=True, margin=15)
        pdf.set_font("Arial", size=12)

        for line in content.split('\n'):
            pdf.cell(0, 10, txt=line.strip().encode('latin-1', 'replace').decode('latin-1'), ln=True)

        pdf.output(file_path)
        messagebox.showinfo("Success", f"PDF saved successfully at {file_path}")
    except Exception as e:
        messagebox.showerror("Error", f"An error occurred while saving the PDF: {e}")

# Function to bold selected text
def bold_text():
    try:
        text_box.tag_add("bold", tk.SEL_FIRST, tk.SEL_LAST)
        text_box.tag_configure("bold", font=("Arial", 12, "bold"))
    except tk.TclError:
        messagebox.showerror("Error", "No text selected to bold.")

# Function to italicize selected text
def italic_text():
    try:
        text_box.tag_add("italic", tk.SEL_FIRST, tk.SEL_LAST)
        text_box.tag_configure("italic", font=("Arial", 12, "italic"))
    except tk.TclError:
        messagebox.showerror("Error", "No text selected to italicize.")

# Function to underline selected text
def underline_text():
    try:
        text_box.tag_add("underline", tk.SEL_FIRST, tk.SEL_LAST)
        text_box.tag_configure("underline", font=("Arial", 12, "underline"))
    except tk.TclError:
        messagebox.showerror("Error", "No text selected to underline.")

# Function to change the language of the text
def change_language():
    language = language_combobox.get()
    if language == "English":
        text_box.config(font=("Arial", 12))
    elif language == "Spanish":
        text_box.config(font=("Arial", 12))  # Add language-specific fonts if needed
    elif language == "French":
        text_box.config(font=("Arial", 12))  # Add language-specific fonts if needed
    else:
        messagebox.showerror("Error", "Unsupported language selected.")

# Create the main GUI window
root = tk.Tk()
root.title("Professional PDF Maker")
root.geometry("900x600")
root.minsize(700, 500)

# Apply a modern theme
style = ttk.Style()
style.theme_use("clam")

# Create a menu bar
menu_bar = tk.Menu(root)
file_menu = tk.Menu(menu_bar, tearoff=0)
file_menu.add_command(label="New", command=new_file)
file_menu.add_command(label="Open", command=open_file)
file_menu.add_command(label="Save", command=save_file)
file_menu.add_command(label="Save As", command=save_as_file)
file_menu.add_separator()
file_menu.add_command(label="Exit", command=root.quit)
menu_bar.add_cascade(label="File", menu=file_menu)

edit_menu = tk.Menu(menu_bar, tearoff=0)
edit_menu.add_command(label="Bold", command=bold_text)
edit_menu.add_command(label="Italic", command=italic_text)
edit_menu.add_command(label="Underline", command=underline_text)
menu_bar.add_cascade(label="Edit", menu=edit_menu)

root.config(menu=menu_bar)

# Create a frame for the text box and scrollbar
frame = ttk.Frame(root)
frame.pack(fill=tk.BOTH, expand=True, padx=10, pady=10)

# Add a text box with a vertical scrollbar
text_box = tk.Text(frame, wrap=tk.WORD, font=("Arial", 12))
scrollbar = ttk.Scrollbar(frame, orient=tk.VERTICAL, command=text_box.yview)
text_box.config(yscrollcommand=scrollbar.set)
scrollbar.pack(side=tk.RIGHT, fill=tk.Y)
text_box.pack(side=tk.LEFT, fill=tk.BOTH, expand=True)

# Create a frame for buttons
button_frame = ttk.Frame(root)
button_frame.pack(fill=tk.X, padx=10, pady=10)

# Add buttons for PDF generation and text formatting
generate_button = ttk.Button(button_frame, text="Generate PDF", command=generate_pdf)
generate_button.pack(side=tk.LEFT, padx=5)

bold_button = ttk.Button(button_frame, text="Bold", command=bold_text)
bold_button.pack(side=tk.LEFT, padx=5)

italic_button = ttk.Button(button_frame, text="Italic", command=italic_text)
italic_button.pack(side=tk.LEFT, padx=5)

underline_button = ttk.Button(button_frame, text="Underline", command=underline_text)
underline_button.pack(side=tk.LEFT, padx=5)

# Add a dropdown to select the language
language_label = ttk.Label(button_frame, text="Language:")
language_label.pack(side=tk.LEFT, padx=5)
language_combobox = ttk.Combobox(button_frame, values=["English", "Spanish", "French"], state="readonly", width=10)
language_combobox.set("English")
language_combobox.pack(side=tk.LEFT, padx=5)
language_combobox.bind("<<ComboboxSelected>>", lambda e: change_language())

# Run the GUI event loop
root.mainloop()
