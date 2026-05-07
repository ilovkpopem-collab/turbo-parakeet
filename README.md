# turbo-parakeet
import json
import random
import os
from tkinter import *
from tkinter import messagebox, ttk

# ------------------- Класс приложения -------------------
class TaskGeneratorApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Random Task Generator")
        self.root.geometry("600x500")
        self.root.resizable(False, False)

        # Данные
        self.tasks = {
            "Учёба": ["Прочитать статью", "Выучить 10 новых слов", "Решить 5 задач", "Посмотреть лекцию"],
            "Спорт": ["Сделать зарядку", "Пробежать 2 км", "Отжаться 20 раз", "Поплавать в бассейне"],
            "Работа": ["Написать отчёт", "Проверить почту", "Спланировать задачи", "Позвонить клиенту"]
        }
        self.history = []       # список словарей {"task": "...", "type": "..."}
        self.current_filter = "Все"

        # Загрузка истории
        self.load_history()

        # Интерфейс
        self.create_widgets()
        self.update_history_display()

    # ------------------- Создание GUI -------------------
    def create_widgets(self):
        # Верхняя панель: генерация задачи
        frame_gen = LabelFrame(self.root, text="Генерация задачи", padx=10, pady=10)
        frame_gen.pack(fill="x", padx=10, pady=5)

        self.btn_generate = Button(frame_gen, text="Сгенерировать задачу", command=self.generate_task,
                                   bg="#4CAF50", fg="white", font=("Arial", 12))
        self.btn_generate.pack(pady=5)

        self.lbl_task = Label(frame_gen, text="", font=("Arial", 14, "bold"), fg="#2196F3", wraplength=500)
        self.lbl_task.pack(pady=10)

        # Панель фильтрации
        frame_filter = LabelFrame(self.root, text="Фильтр по типу", padx=10, pady=5)
        frame_filter.pack(fill="x", padx=10, pady=5)

        self.filter_var = StringVar(value="Все")
        filter_options = ["Все"] + list(self.tasks.keys())
        filter_menu = ttk.Combobox(frame_filter, textvariable=self.filter_var, values=filter_options, state="readonly")
        filter_menu.pack(side="left", padx=5)
        filter_menu.bind("<<ComboboxSelected>>", self.on_filter_change)

        self.btn_apply_filter = Button(frame_filter, text="Применить фильтр", command=self.on_filter_change)
        self.btn_apply_filter.pack(side="left", padx=5)

        # Панель добавления новой задачи
        frame_add = LabelFrame(self.root, text="Добавить новую задачу", padx=10, pady=5)
        frame_add.pack(fill="x", padx=10, pady=5)

        Label(frame_add, text="Тип:").grid(row=0, column=0, padx=5, pady=5, sticky="e")
        self.type_combobox = ttk.Combobox(frame_add, values=list(self.tasks.keys()), state="readonly")
        self.type_combobox.grid(row=0, column=1, padx=5, pady=5)
        self.type_combobox.current(0)

        Label(frame_add, text="Задача:").grid(row=1, column=0, padx=5, pady=5, sticky="e")
        self.entry_task = Entry(frame_add, width=40)
        self.entry_task.grid(row=1, column=1, padx=5, pady=5)

        self.btn_add = Button(frame_add, text="Добавить задачу", command=self.add_task, bg="#FFC107")
        self.btn_add.grid(row=1, column=2, padx=5, pady=5)

        # История задач
        frame_history = LabelFrame(self.root, text="История задач", padx=10, pady=5)
        frame_history.pack(fill="both", expand=True, padx=10, pady=5)

        scrollbar = Scrollbar(frame_history)
        scrollbar.pack(side="right", fill="y")

        self.history_listbox = Listbox(frame_history, yscrollcommand=scrollbar.set, font=("Arial", 10))
        self.history_listbox.pack(fill="both", expand=True)
        scrollbar.config(command=self.history_listbox.yview)

        # Кнопка очистки истории
        self.btn_clear_history = Button(self.root, text="Очистить историю", command=self.clear_history, bg="#f44336", fg="white")
        self.btn_clear_history.pack(pady=5)

    # ------------------- Логика работы -------------------
    def generate_task(self):
        """Выбирает случайную задачу с учётом фильтра и добавляет в историю"""
        available_tasks = []
        available_types = []

        if self.current_filter == "Все":
            for task_type, task_list in self.tasks.items():
                for task in task_list:
                    available_tasks.append((task, task_type))
        else:
            if self.current_filter in self.tasks:
                for task in self.tasks[self.current_filter]:
                    available_tasks.append((task, self.current_filter))

        if not available_tasks:
            messagebox.showwarning("Нет задач", f"Нет задач в категории '{self.current_filter}'. Добавьте новую задачу.")
            return

        selected_task, selected_type = random.choice(available_tasks)
        self.lbl_task.config(text=f"🎯 {selected_task}")

        # Сохраняем в историю
        self.history.append({"task": selected_task, "type": selected_type})
        self.save_history()
        self.update_history_display()

    def add_task(self):
        """Добавляет новую задачу в список (с проверкой на пустую строку)"""
        task_text = self.entry_task.get().strip()
        task_type = self.type_combobox.get()

        if not task_text:
            messagebox.showerror("Ошибка", "Название задачи не может быть пустым!")
            return

        if task_type not in self.tasks:
            self.tasks[task_type] = []

        if task_text in self.tasks[task_type]:
            messagebox.showinfo("Инфо", "Такая задача уже существует в этом типе.")
            return

        self.tasks[task_type].append(task_text)
        self.entry_task.delete(0, END)
        messagebox.showinfo("Успех", f"Задача '{task_text}' добавлена в категорию '{task_type}'.")

        # Обновляем фильтр комбобокса при необходимости
        self.filter_var.set("Все")
        self.current_filter = "Все"
        self.update_filter_combobox()

    def update_filter_combobox(self):
        """Обновляет список типов в выпадающем списке фильтра"""
        filter_options = ["Все"] + list(self.tasks.keys())
        self.filter_menu['values'] = filter_options

    def on_filter_change(self, event=None):
        """Обрабатывает смену фильтра"""
        self.current_filter = self.filter_var.get()
        # Не обновляем список задач, просто фильтруем при генерации

    def update_history_display(self):
        """Отображает историю в Listbox"""
        self.history_listbox.delete(0, END)
        for entry in self.history:
            self.history_listbox.insert(END, f"[{entry['type']}] {entry['task']}")

    def clear_history(self):
        """Очищает историю задач"""
        self.history.clear()
        self.save_history()
        self.update_history_display()
        messagebox.showinfo("История очищена", "Вся история удалена.")

    # ------------------- Работа с JSON -------------------
    def save_history(self):
        """Сохраняет историю в JSON-файл"""
        try:
            with open("tasks_data.json", "w", encoding="utf-8") as f:
                json.dump(self.history, f, ensure_ascii=False, indent=2)
        except Exception as e:
            print(f"Ошибка сохранения: {e}")

    def load_history(self):
        """Загружает историю из JSON-файла"""
        if os.path.exists("tasks_data.json"):
            try:
                with open("tasks_data.json", "r", encoding="utf-8") as f:
                    loaded = json.load(f)
                    if isinstance(loaded, list):
                        self.history = loaded
            except Exception as e:
                print(f"Ошибка загрузки: {e}")
                self.history = []

# ------------------- Запуск приложения -------------------
if __name__ == "__main__":
    root = Tk()
    app = TaskGeneratorApp(root)
    root.mainloop()
