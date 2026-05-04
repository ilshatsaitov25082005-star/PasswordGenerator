# PasswordGenerator
Автор: Мавлютов Ильназ
Курс: 9ИСИП-11П-25(29)*Язык: Python 
Python: Язык программирования.
Tkinter: Библиотека для создания графического интерфейса (*GUI*).
random: Стандартная библиотека для генерации случайных чисел/строк.
json: Стандартная библиотека для работы с файлом истории.
Git: Система контроля версий.



    import tkinter as tk
    from tkinter import ttk, messagebox, font as tkfont
    import random
    import json
    import os

    MIN_LENGTH = 4
    MAX_LENGTH = 32

    DIGITS = "0123456789"
    LETTERS = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"
    SPECIAL = "!@#$%^&*()_+-=[]{}|;:,.<>?/"

    class PasswordGeneratorApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Генератор случайных паролей")
        self.root.geometry("600x450")
        self.root.resizable(False, False)
        
        self.history = []
        self.load_history()
        
        self.create_widgets()
        self.update_history_table()
        
    def create_widgets(self):
        settings_frame = tk.LabelFrame(self.root, text="Настройки пароля", padx=10, pady=10)
        settings_frame.pack(pady=10, fill="x", padx=20)
        
        tk.Label(settings_frame, text="Длина:").grid(row=0, column=0, sticky="e")
        self.length_var = tk.IntVar(value=12)
        self.slider = tk.Scale(settings_frame, from_=MIN_LENGTH, to=MAX_LENGTH, orient="horizontal",
                              variable=self.length_var, length=250)
        self.slider.grid(row=0, column=1, columnspan=2, sticky="w")
        
        self.use_digits = tk.BooleanVar(value=True)
        self.use_letters = tk.BooleanVar(value=True)
        self.use_special = tk.BooleanVar(value=True)
        
        tk.Checkbutton(settings_frame, text="Цифры", variable=self.use_digits).grid(row=1, column=1, sticky="w")
        tk.Checkbutton(settings_frame, text="Буквы", variable=self.use_letters).grid(row=2, column=1, sticky="w")
        tk.Checkbutton(settings_frame, text="Спецсимволы", variable=self.use_special).grid(row=3, column=1, sticky="w")
        
        gen_btn = tk.Button(settings_frame, text="Сгенерировать", command=self.generate_password,
                           font=tkfont.Font(size=10))
        gen_btn.grid(row=4, column=1, pady=15)
        
        self.password_entry = tk.Entry(self.root, font=tkfont.Font(size=14), width=40)
        self.password_entry.pack(pady=5)
        
        copy_btn = tk.Button(self.root, text="Копировать", command=self.copy_to_clipboard)
        copy_btn.pack(pady=5)
        
        history_frame = tk.LabelFrame(self.root, text="История", padx=10, pady=10)
        history_frame.pack(fill="both", expand=True, padx=20, pady=(0, 10))
        
        self.tree = ttk.Treeview(history_frame, columns=("password",), show="headings", height=7)
        self.tree.heading("password", text="Пароль")
        self.tree.column("password", width=350)
        
        vsb = ttk.Scrollbar(history_frame, orient="vertical", command=self.tree.yview)
        self.tree.configure(yscrollcommand=vsb.set)
        
        self.tree.pack(side="left", fill="both", expand=True)
        vsb.pack(side="right", fill="y")
    
    def generate_password(self):
        length = self.length_var.get()
        
        if not (self.use_digits.get() or self.use_letters.get() or self.use_special.get()):
            messagebox.showerror("Ошибка", "Выберите хотя бы один тип символов!")
            return

        pool = ""
        if self.use_digits.get():
            pool += DIGITS
        if self.use_letters.get():
            pool += LETTERS
        if self.use_special.get():
            pool += SPECIAL

        password = ''.join(random.choices(pool, k=length))
        
        self.password_entry.delete(0, tk.END)
        self.password_entry.insert(0, password)
        
        self.history.append(password)
        if len(self.history) > 20:
            self.history.pop(0)
            
        self.save_history()
        self.update_history_table()
    
    def copy_to_clipboard(self):
        password = self.password_entry.get()
        if password:
            self.root.clipboard_clear()
            self.root.clipboard_append(password)
            messagebox.showinfo("Успех", "Пароль скопирован в буфер обмена!")
    
    def save_history(self):
        with open("history.json", "w", encoding="utf-8") as f:
            json.dump(self.history, f, ensure_ascii=False, indent=2)
    
    def load_history(self):
        if os.path.exists("history.json"):
            try:
                with open("history.json", "r", encoding="utf-8") as f:
                    data = json.load(f)
                    if isinstance(data, list):
                        self.history = data[:20]
            except (json.JSONDecodeError, FileNotFoundError):
                pass

    def update_history_table(self):
        for i in self.tree.get_children():
            self.tree.delete(i)
            
        for pwd in reversed(self.history):
            self.tree.insert("", "end", values=(pwd,))
            
    if __name__ == "__main__":
    root = tk.Tk()
    app = PasswordGeneratorApp(root)
    root.mainloop()
