# turbo-parakeet
# Random Task Generator

**Автор:** Виолетта Накохова

## Описание

Random Task Generator — это приложение с графическим интерфейсом для генерации случайных задач по категориям. Помогает разнообразить дела и не забывать о важных активностях.

## Функции

- Генерация случайной задачи
- Фильтрация по типу (Учёба, Спорт, Работа)
- Добавление своих задач
- Сохранение истории в JSON
- Загрузка истории при запуске
- Проверка на пустые строки при добавлении

## Установка и запуск

1. Установите Python 3.6 или выше
2. Скачайте файл `task_generator.py`
3. Запустите командой:
```bash
python task_generator.py
import json
import random
import os
from tkinter import *
from tkinter import messagebox, ttk

class TaskGeneratorApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Random Task Generator")
        self.root.geometry("650x550")
        self.root.resizable(False, False)

        # Предопределённые задачи по типам
        self.tasks = {
            "Учёба": ["Прочитать статью", "Выучить 10 новых слов", "Решить 5 задач", "Посмотреть лекцию", "Написать конспект"],
            "Спорт": ["Сделать зарядку", "Пробежать 2 км", "Отжаться 20 раз", "Поплавать в бассейне", "Пойти на тренировку"],
            "Работа": ["Написать отчёт", "Проверить почту", "Спланировать задачи", "Позвонить клиенту", "Сделать презентацию"]
        }
        self.history = []
        self.current_filter = "Все"

        # Загрузка истории из JSON
        self.load_history()

        # Создание интерфейса
        self.create_widgets()
        self.update_history_display()

    def create_widgets(self):
        # Рамка генерации задачи
        frame_gen = LabelFrame(self.root, text="🎲 Генератор задач", padx=10, pady=10, font=("Arial", 10, "bold"))
        frame_gen.pack(fill="x", padx=10, pady=5)

        self.btn_generate = Button(frame_gen, text="🔀 Сгенерировать задачу", command=self.generate_task,
                                   bg="#4CAF50", fg="white", font=("Arial", 12, "bold"), height=2)
        self.btn_generate.pack(pady=5, fill="x")

        self.lbl_task = Label(frame_gen, text="", font=("Arial", 14, "bold"), fg="#2196F3", wraplength=600, pady=10)
        self.lbl_task.pack()

        # Рамка фильтрации
        frame_filter = LabelFrame(self.root, text="🔍 Фильтр по типу", padx=10, pady=5, font=("Arial", 10, "bold"))
        frame_filter.pack(fill="x", padx=10, pady=5)

        self.filter_var = StringVar(value="Все")
        filter_options = ["Все"] + list(self.tasks.keys())
        self.filter_menu = ttk.Combobox(frame_filter, textvariable=self.filter_var, values=filter_options, state="readonly", width=20)
        self.filter_menu.pack(side="left", padx=5)
        self.filter_menu.bind("<<ComboboxSelected>>", self.on_filter_change)

        self.btn_apply_filter = Button(frame_filter, text="Применить фильтр", command=self.on_filter_change, bg="#2196F3", fg="white")
        self.btn_apply_filter.pack(side="left", padx=5)

        # Рамка добавления задачи
        frame_add = LabelFrame(self.root, text="➕ Добавить новую задачу", padx=10, pady=5, font=("Arial", 10, "bold"))
        frame_add.pack(fill="x", padx=10, pady=5)

        Label(frame_add, text="Тип:").grid(row=0, column=0, padx=5, pady=5, sticky="e")
        self.type_combobox = ttk.Combobox(frame_add, values=list(self.tasks.keys()), state="readonly", width=15)
        self.type_combobox.grid(row=0, column=1, padx=5, pady=5)
        self.type_combobox.current(0)

        Label(frame_add, text="Задача:").grid(row=1, column=0, padx=5, pady=5, sticky="e")
        self.entry_task = Entry(frame_add, width=40)
        self.entry_task.grid(row=1, column=1, padx=5, pady=5)

        self.btn_add = Button(frame_add, text="Добавить", command=self.add_task, bg="#FF9800", fg="white")
        self.btn_add.grid(row=1, column=2, padx=5, pady=5)

        # Рамка истории
        frame_history = LabelFrame(self.root, text="📜 История задач", padx=10, pady=5, font=("Arial", 10, "bold"))
        frame_history.pack(fill="both", expand=True, padx=10, pady=5)

        scrollbar = Scrollbar(frame_history)
        scrollbar.pack(side="right", fill="y")

        self.history_listbox = Listbox(frame_history, yscrollcommand=scrollbar.set, font=("Arial", 10), height=12)
        self.history_listbox.pack(fill="both", expand=True)
        scrollbar.config(command=self.history_listbox.yview)

        # Кнопка очистки
        self.btn_clear = Button(self.root, text="🗑 Очистить историю", command=self.clear_history, bg="#f44336", fg="white", font=("Arial", 10))
        self.btn_clear.pack(pady=5)

    def generate_task(self):
        available_tasks = []

        if self.current_filter == "Все":
            for task_type, task_list in self.tasks.items():
                for task in task_list:
                    available_tasks.append((task, task_type))
        else:
            if self.current_filter in self.tasks:
                for task in self.tasks[self.current_filter]:
                    available_tasks.append((task, self.current_filter))

        if not available_tasks:
            messagebox.showwarning("Нет задач", f"В категории '{self.current_filter}' нет задач. Добавьте новую задачу.")
            return

        selected_task, selected_type = random.choice(available_tasks)
        self.lbl_task.config(text=f"✨ {selected_task} ✨")

        self.history.append({"task": selected_task, "type": selected_type})
        self.save_history()
        self.update_history_display()

    def add_task(self):
        task_text = self.entry_task.get().strip()
        task_type = self.type_combobox.get()

        if not task_text:
            messagebox.showerror("Ошибка", "Название задачи не может быть пустым!")
            return

        if task_type not in self.tasks:
            self.tasks[task_type] = []

        if task_text in self.tasks[task_type]:
            messagebox.showinfo("Информация", "Такая задача уже существует в этой категории.")
            return

        self.tasks[task_type].append(task_text)
        self.entry_task.delete(0, END)
        messagebox.showinfo("Успех", f"Задача '{task_text}' добавлена в категорию '{task_type}'.")

        # Обновляем фильтр
        self.filter_var.set("Все")
        self.current_filter = "Все"
        self.update_filter_menu()

    def update_filter_menu(self):
        filter_options = ["Все"] + list(self.tasks.keys())
        self.filter_menu['values'] = filter_options

    def on_filter_change(self, event=None):
        self.current_filter = self.filter_var.get()

    def update_history_display(self):
        self.history_listbox.delete(0, END)
        for entry in self.history:
            self.history_listbox.insert(END, f"[{entry['type']}] {entry['task']}")

    def clear_history(self):
        if messagebox.askyesno("Подтверждение", "Вы уверены, что хотите очистить всю историю?"):
            self.history.clear()
            self.save_history()
            self.update_history_display()
            messagebox.showinfo("История очищена", "Все задачи удалены из истории.")

    def save_history(self):
        try:
            with open("tasks_data.json", "w", encoding="utf-8") as f:
                json.dump(self.history, f, ensure_ascii=False, indent=2)
        except Exception as e:
            print(f"Ошибка сохранения: {e}")

    def load_history(self):
        if os.path.exists("tasks_data.json"):
            try:
                with open("tasks_data.json", "r", encoding="utf-8") as f:
                    loaded = json.load(f)
                    if isinstance(loaded, list):
                        self.history = loaded
            except Exception as e:
                print(f"Ошибка загрузки: {e}")
                self.history = []

if __name__ == "__main__":
    root = Tk()
    app = TaskGeneratorApp(root)
    root.mainloop()
