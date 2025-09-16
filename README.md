import tkinter as tk
from tkinter import ttk, messagebox, simpledialog, filedialog
from PIL import Image, ImageTk
import os

class VKMessenger:
    def __init__(self, root):
        self.root = root
        self.root.title("VK Мессенджер")
        self.root.geometry("1000x700")
        self.root.configure(bg='#EDEDED')
        
        # Настройки темы
        self.dark_mode = False
        self.current_user = {
            'name': 'Иван Иванов',
            'status': 'В сети',
            'age': '25',
            'city': 'Москва',
            'avatar_path': None
        }
        
        self.setup_ui()
        
    def setup_ui(self):
        # Главный фрейм
        self.main_frame = tk.Frame(self.root, bg='#EDEDED')
        self.main_frame.pack(fill='both', expand=True)
        
        # Боковая панель (как в VK)
        self.sidebar = tk.Frame(self.main_frame, bg='#4680C2', width=200)
        self.sidebar.pack(side='left', fill='y')
        self.sidebar.pack_propagate(False)
        
        # Создаем элементы боковой панели
        self.create_sidebar()
        
        # Основная область
        self.main_area = tk.Frame(self.main_frame, bg='#EDEDED')
        self.main_area.pack(side='right', fill='both', expand=True)
        
        # Заголовок
        self.header = tk.Frame(self.main_area, bg='#FFFFFF', height=60)
        self.header.pack(fill='x')
        self.header.pack_propagate(False)
        
        tk.Label(self.header, text="Мои сообщения", font=("Arial", 16, "bold"), 
                bg='#FFFFFF', fg='#000000').pack(pady=15)
        
        # Область чата
        self.chat_frame = tk.Frame(self.main_area, bg='#FFFFFF')
        self.chat_frame.pack(fill='both', expand=True, padx=10, pady=10)
        
        # Поле ввода сообщения
        self.input_frame = tk.Frame(self.main_area, bg='#FFFFFF', height=60)
        self.input_frame.pack(fill='x', padx=10, pady=5)
        self.input_frame.pack_propagate(False)
        
        self.message_entry = tk.Entry(self.input_frame, font=("Arial", 12), 
                                     bg='#F5F5F5', relief='flat')
        self.message_entry.pack(side='left', fill='x', expand=True, padx=5, pady=10)
        self.message_entry.bind('<Return>', self.send_message)
        
        self.send_btn = tk.Button(self.input_frame, text="➤", font=("Arial", 14),
                                 command=self.send_message, bg='#4680C2', fg='white',
                                 relief='flat', width=3)
        self.send_btn.pack(side='right', padx=5, pady=10)
        
        # Область сообщений
        self.messages_frame = tk.Frame(self.chat_frame, bg='#E5E5E5')
        self.messages_frame.pack(fill='both', expand=True)
        
        self.canvas = tk.Canvas(self.messages_frame, bg='#E5E5E5')
        self.scrollbar = ttk.Scrollbar(self.messages_frame, orient="vertical", command=self.canvas.yview)
        self.scrollable_frame = tk.Frame(self.canvas, bg='#E5E5E5')
        
        self.scrollable_frame.bind(
            "<Configure>",
            lambda e: self.canvas.configure(scrollregion=self.canvas.bbox("all"))
        )
        
        self.canvas.create_window((0, 0), window=self.scrollable_frame, anchor="nw")
        self.canvas.configure(yscrollcommand=self.scrollbar.set)
        
        self.canvas.pack(side="left", fill="both", expand=True)
        self.scrollbar.pack(side="right", fill="y")
        
        # Добавляем тестовые сообщения
        self.add_message("Привет! Как дела?", "Друг Петя", False)
        self.add_message("Привет! Все отлично, а у тебя?", "Я", True)
        
    def create_sidebar(self):
        # Аватар и информация о пользователе
        avatar_frame = tk.Frame(self.sidebar, bg='#4680C2')
        avatar_frame.pack(pady=20)
        
        self.avatar_label = tk.Label(avatar_frame, text="👤", font=("Arial", 24), 
                                    bg='#4680C2', fg='white', cursor="hand2")
        self.avatar_label.pack()
        self.avatar_label.bind("<Button-1>", lambda e: self.open_profile_settings())
        
        tk.Label(avatar_frame, text=self.current_user['name'], 
                font=("Arial", 12, "bold"), bg='#4680C2', fg='white').pack(pady=5)
        tk.Label(avatar_frame, text=self.current_user['status'], 
                font=("Arial", 10), bg='#4680C2', fg='#DDDDDD').pack()
        
        # Кнопки меню
        menu_items = [
            ("💬 Мои сообщения", None),
            ("👥 Друзья", None),
            ("📷 Фотографии", None),
            ("⚙️ Настройки", self.open_settings),
            ("🌙 Тёмная тема", self.toggle_theme),
            ("🚪 Выйти", self.exit_app)
        ]
        
        for text, command in menu_items:
            btn = tk.Button(self.sidebar, text=text, font=("Arial", 11),
                           bg='#4680C2', fg='white', relief='flat', anchor='w',
                           command=command if command else None)
            btn.pack(fill='x', padx=10, pady=5)
            btn.bind("<Enter>", lambda e, b=btn: b.config(bg='#5A8DC8'))
            btn.bind("<Leave>", lambda e, b=btn: b.config(bg='#4680C2'))
    
    def add_message(self, text, sender, is_me):
        message_frame = tk.Frame(self.scrollable_frame, bg='#E5E5E5')
        message_frame.pack(fill='x', pady=2)
        
        if is_me:
            bg_color = '#CCE4FF'
            align = 'right'
        else:
            bg_color = '#FFFFFF'
            align = 'left'
        
        msg = tk.Label(message_frame, text=text, font=("Arial", 11),
                      bg=bg_color, fg='#000000', relief='raised',
                      wraplength=300, justify='left')
        msg.pack(side=align, padx=10, pady=5)
        
        sender_label = tk.Label(message_frame, text=sender, font=("Arial", 8),
                               bg='#E5E5E5', fg='#666666')
        sender_label.pack(side=align, padx=10)
        
    def send_message(self, event=None):
        message = self.message_entry.get().strip()
        if message:
            self.add_message(message, "Я", True)
            self.message_entry.delete(0, tk.END)
            # Автоответ
            self.root.after(1000, lambda: self.add_message("Получил твое сообщение!", "Друг Петя", False))
            
    def open_settings(self):
        settings_window = tk.Toplevel(self.root)
        settings_window.title("Настройки VK")
        settings_window.geometry("500x400")
        settings_window.configure(bg='#FFFFFF')
        settings_window.resizable(False, False)
        
        notebook = ttk.Notebook(settings_window)
        notebook.pack(fill='both', expand=True, padx=10, pady=10)
        
        # Вкладка общего профиля
        general_frame = ttk.Frame(notebook, padding=10)
        notebook.add(general_frame, text="Основное")
        
        ttk.Label(general_frame, text="Настройки профиля", font=("Arial", 14, "bold")).grid(row=0, column=0, columnspan=2, pady=10)
        
        fields = [
            ("Имя:", "name"),
            ("Статус:", "status"),
            ("Возраст:", "age"),
            ("Город:", "city")
        ]
        
        self.settings_vars = {}
        for i, (label, key) in enumerate(fields, 1):
            ttk.Label(general_frame, text=label).grid(row=i, column=0, sticky='w', pady=5)
            var = tk.StringVar(value=self.current_user[key])
            entry = ttk.Entry(general_frame, textvariable=var, width=30)
            entry.grid(row=i, column=1, pady=5, padx=10)
            self.settings_vars[key] = var
        
        ttk.Button(general_frame, text="Сменить аватар", 
                  command=self.change_avatar).grid(row=5, column=0, columnspan=2, pady=10)
        
        ttk.Button(general_frame, text="Сохранить", 
                  command=lambda: self.save_settings(settings_window)).grid(row=6, column=0, columnspan=2, pady=10)
        
        # Вкладка внешнего вида
        appearance_frame = ttk.Frame(notebook, padding=10)
        notebook.add(appearance_frame, text="Внешний вид")
        
        ttk.Label(appearance_frame, text="Тема оформления", font=("Arial", 14, "bold")).grid(row=0, column=0, columnspan=2, pady=10)
        
        self.theme_var = tk.StringVar(value="light")
        ttk.Radiobutton(appearance_frame, text="Светлая тема", variable=self.theme_var, 
                       value="light").grid(row=1, column=0, sticky='w', pady=5)
        ttk.Radiobutton(appearance_frame, text="Тёмная тема", variable=self.theme_var, 
                       value="dark").grid(row=2, column=0, sticky='w', pady=5)
        
        ttk.Button(appearance_frame, text="Применить тему", 
                  command=self.apply_theme).grid(row=3, column=0, pady=20)
    
    def change_avatar(self):
        file_path = filedialog.askopenfilename(
            filetypes=[("Image files", "*.jpg *.jpeg *.png *.gif")]
        )
        if file_path:
            try:
                image = Image.open(file_path)
                image = image.resize((100, 100), Image.Resampling.LANCZOS)
                photo = ImageTk.PhotoImage(image)
                
                self.current_user['avatar_path'] = file_path
                self.avatar_label.config(image=photo, text="")
                self.avatar_label.image = photo
                
                messagebox.showinfo("Успех", "Аватар успешно изменен!")
            except Exception as e:
                messagebox.showerror("Ошибка", f"Не удалось загрузить изображение: {e}")
    
    def save_settings(self, window):
        try:
            for key, var in self.settings_vars.items():
                self.current_user[key] = var.get()
            
            # Обновляем информацию в боковой панели
            for widget in self.sidebar.winfo_children():
                if isinstance(widget, tk.Frame):
                    for label in widget.winfo_children():
                        if isinstance(label, tk.Label) and "Иван" in label.cget("text"):
                            label.config(text=self.current_user['name'])
            
            messagebox.showinfo("Успех", "Настройки сохранены!")
            window.destroy()
            
        except Exception as e:
            messagebox.showerror("Ошибка", f"Ошибка при сохранении: {e}")
    
    def apply_theme(self):
        if self.theme_var.get() == "dark":
            self.apply_dark_theme()
        else:
            self.apply_light_theme()
    
    def apply_dark_theme(self):
        colors = {
            'bg': '#2C2C2C',
            'fg': '#FFFFFF',
            'sidebar_bg': '#1E1E1E',
            'header_bg': '#333333',
            'chat_bg': '#2C2C2C',
            'input_bg': '#3C3C3C'
        }
        self.update_theme(colors)
        self.dark_mode = True
    
    def apply_light_theme(self):
        colors = {
            'bg': '#EDEDED',
            'fg': '#000000',
            'sidebar_bg': '#4680C2',
            'header_bg': '#FFFFFF',
            'chat_bg': '#E5E5E5',
            'input_bg': '#F5F5F5'
        }
        self.update_theme(colors)
        self.dark_mode = False
    
    def update_theme(self, colors):
        self.root.configure(bg=colors['bg'])
        self.main_frame.configure(bg=colors['bg'])
        self.sidebar.configure(bg=colors['sidebar_bg'])
        self.main_area.configure(bg=colors['bg'])
        self.header.configure(bg=colors['header_bg'])
        self.chat_frame.configure(bg=colors['bg'])
        self.input_frame.configure(bg=colors['header_bg'])
        self.messages_frame.configure(bg=colors['chat_bg'])
        self.canvas.configure(bg=colors['chat_bg'])
        self.scrollable_frame.configure(bg=colors['chat_bg'])
        self.message_entry.configure(bg=colors['input_bg'], fg=colors['fg'])
        
        # Обновляем все метки
        for widget in self.root.winfo_children():
            self.update_widget_colors(widget, colors)
    
    def update_widget_colors(self, widget, colors):
        if isinstance(widget, (tk.Label, tk.Button)):
            if widget not in [self.avatar_label]:
                widget.configure(bg=colors.get('bg', widget.cget('bg')),
                               fg=colors.get('fg', widget.cget('fg')))
        
        for child in widget.winfo_children():
            self.update_widget_colors(child, colors)
    
    def toggle_theme(self):
        if self.dark_mode:
            self.apply_light_theme()
        else:
            self.apply_dark_theme()
    
    def exit_app(self):
        if messagebox.askyesno("Выход", "Вы уверены, что хотите выйти?"):
            self.root.quit()

def main():
    root = tk.Tk()
    app = VKMessenger(root)
    root.mainloop()

if __name__ == "__main__":
    main()
