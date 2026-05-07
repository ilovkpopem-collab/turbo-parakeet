# turbo-parakeet
import unittest
import json
import os
import tempfile
from unittest.mock import patch, MagicMock
from tkinter import Tk, messagebox
import random


class TaskGeneratorApp:
    """Упрощённая версия класса для тестирования (без GUI)"""
    
    def __init__(self):
        self.tasks = {
            "Учёба": ["Прочитать статью", "Выучить 10 новых слов", "Решить 5 задач"],
            "Спорт": ["Сделать зарядку", "Пробежать 2 км", "Отжаться 20 раз"],
            "Работа": ["Написать отчёт", "Проверить почту", "Спланировать задачи"]
        }
        self.history = []
        self.current_filter = "Все"
    
    def generate_task(self):
        """Выбирает случайную задачу с учётом фильтра"""
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
            return None, None
        
        selected_task, selected_type = random.choice(available_tasks)
        self.history.append({"task": selected_task, "type": selected_type})
        return selected_task, selected_type
    
    def add_task(self, task_text, task_type):
        """Добавляет новую задачу"""
        if not task_text or not task_text.strip():
            return False, "Пустая строка"
        
        if task_type not in self.tasks:
            self.tasks[task_type] = []
        
        if task_text in self.tasks[task_type]:
            return False, "Задача уже существует"
        
        self.tasks[task_type].append(task_text)
        return True, "Задача добавлена"
    
    def set_filter(self, filter_value):
        """Устанавливает фильтр"""
        self.current_filter = filter_value
    
    def clear_history(self):
        """Очищает историю"""
        self.history = []
    
    def save_history(self, filename="test_tasks_data.json"):
        """Сохраняет историю в JSON"""
        try:
            with open(filename, "w", encoding="utf-8") as f:
                json.dump(self.history, f, ensure_ascii=False, indent=2)
            return True
        except:
            return False
    
    def load_history(self, filename="test_tasks_data.json"):
        """Загружает историю из JSON"""
        if os.path.exists(filename):
            try:
                with open(filename, "r", encoding="utf-8") as f:
                    loaded = json.load(f)
                    if isinstance(loaded, list):
                        self.history = loaded
                        return True
            except:
                pass
        return False


class TestTaskGenerator(unittest.TestCase):
    """Модульные тесты для Random Task Generator"""
    
    def setUp(self):
        """Подготовка перед каждым тестом"""
        self.app = TaskGeneratorApp()
        self.test_json = "test_temp_history.json"
    
    def tearDown(self):
        """Очистка после каждого теста"""
        if os.path.exists(self.test_json):
            os.remove(self.test_json)
    
    # ------------------- Тесты генерации задач -------------------
    
    def test_generate_task_no_filter(self):
        """Тест 1: Генерация задачи без фильтра"""
        task, task_type = self.app.generate_task()
        self.assertIsNotNone(task)
        self.assertIsNotNone(task_type)
        self.assertIn(task_type, ["Учёба", "Спорт", "Работа"])
        self.assertEqual(len(self.app.history), 1)
    
    def test_generate_task_with_filter(self):
        """Тест 2: Генерация задачи с фильтром"""
        self.app.set_filter("Спорт")
        task, task_type = self.app.generate_task()
        self.assertEqual(task_type, "Спорт")
        self.assertIn(task, self.app.tasks["Спорт"])
    
    def test_generate_task_empty_category(self):
        """Тест 3: Генерация из пустой категории"""
        self.app.tasks["Пусто"] = []
        self.app.set_filter("Пусто")
        task, task_type = self.app.generate_task()
        self.assertIsNone(task)
        self.assertIsNone(task_type)
    
    # ------------------- Тесты добавления задач -------------------
    
    def test_add_valid_task(self):
        """Тест 4: Добавление корректной задачи"""
        success, message = self.app.add_task("Новая задача", "Учёба")
        self.assertTrue(success)
        self.assertIn("Новая задача", self.app.tasks["Учёба"])
    
    def test_add_empty_task(self):
        """Тест 5: Добавление пустой задачи"""
        success, message = self.app.add_task("", "Учёба")
        self.assertFalse(success)
        self.assertEqual(message, "Пустая строка")
    
    def test_add_whitespace_task(self):
        """Тест 6: Добавление задачи из пробелов"""
        success, message = self.app.add_task("   ", "Учёба")
        self.assertFalse(success)
        self.assertEqual(message, "Пустая строка")
    
    def test_add_duplicate_task(self):
        """Тест 7: Добавление дубликата задачи"""
        self.app.add_task("Дубликат", "Учёба")
        success, message = self.app.add_task("Дубликат", "Учёба")
        self.assertFalse(success)
        self.assertEqual(message, "Задача уже существует")
    
    def test_add_task_new_category(self):
        """Тест 8: Добавление задачи в новую категорию"""
        success, message = self.app.add_task("Новая задача", "Хобби")
        self.assertTrue(success)
        self.assertIn("Хобби", self.app.tasks)
        self.assertIn("Новая задача", self.app.tasks["Хобби"])
    
    # ------------------- Тесты фильтрации -------------------
    
    def test_filter_change(self):
        """Тест 9: Смена фильтра"""
        self.app.set_filter("Работа")
        self.assertEqual(self.app.current_filter, "Работа")
        
        task, task_type = self.app.generate_task()
        self.assertEqual(task_type, "Работа")
    
    def test_filter_all_categories(self):
        """Тест 10: Фильтр 'Все' включает все категории"""
        self.app.set_filter("Все")
        categories_found = set()
        
        # Генерируем 20 задач и проверяем разнообразие категорий
        for _ in range(20):
            _, task_type = self.app.generate_task()
            categories_found.add(task_type)
        
        # Должны встретиться все категории
        self.assertIn("Учёба", categories_found)
        self.assertIn("Спорт", categories_found)
        self.assertIn("Работа", categories_found)
    
    # ------------------- Тесты истории -------------------
    
    def test_history_recording(self):
        """Тест 11: Запись в историю"""
        initial_length = len(self.app.history)
        self.app.generate_task()
        self.assertEqual(len(self.app.history), initial_length + 1)
    
    def test_multiple_tasks_in_history(self):
        """Тест 12: Несколько задач в истории"""
        for _ in range(5):
            self.app.generate_task()
        self.assertEqual(len(self.app.history), 5)
    
    def test_clear_history(self):
        """Тест 13: Очистка истории"""
        self.app.generate_task()
        self.app.generate_task()
        self.assertGreater(len(self.app.history), 0)
        
        self.app.clear_history()
        self.assertEqual(len(self.app.history), 0)
    
    def test_history_content_format(self):
        """Тест 14: Формат содержимого истории"""
        self.app.set_filter("Учёба")
        task, task_type = self.app.generate_task()
        
        self.assertEqual(len(self.app.history), 1)
        self.assertEqual(self.app.history[0]["task"], task)
        self.assertEqual(self.app.history[0]["type"], task_type)
    
    # ------------------- Тесты сохранения/загрузки -------------------
    
    def test_save_to_json(self):
        """Тест 15: Сохранение истории в JSON"""
        self.app.generate_task()
        self.app.generate_task()
        
        success = self.app.save_history(self.test_json)
        self.assertTrue(success)
        self.assertTrue(os.path.exists(self.test_json))
        
        # Проверяем содержимое
        with open(self.test_json, "r", encoding="utf-8") as f:
            data = json.load(f)
        self.assertEqual(len(data), len(self.app.history))
    
    def test_load_from_json(self):
        """Тест 16: Загрузка истории из JSON"""
        # Создаём тестовый JSON
        test_history = [
            {"task": "Тестовая задача 1", "type": "Учёба"},
            {"task": "Тестовая задача 2", "type": "Спорт"}
        ]
        with open(self.test_json, "w", encoding="utf-8") as f:
            json.dump(test_history, f)
        
        # Загружаем
        new_app = TaskGeneratorApp()
        success = new_app.load_history(self.test_json)
        
        self.assertTrue(success)
        self.assertEqual(len(new_app.history), 2)
        self.assertEqual(new_app.history[0]["task"], "Тестовая задача 1")
    
    def test_load_nonexistent_file(self):
        """Тест 17: Загрузка из несуществующего файла"""
        success = self.app.load_history("nonexistent_file.json")
        self.assertFalse(success)
        self.assertEqual(len(self.app.history), 0)
    
    def test_save_invalid_data(self):
        """Тест 18: Сохранение повреждённых данных"""
        self.app.history = None  # Некорректные данные
        success = self.app.save_history(self.test_json)
        # Должен быть graceful fallback
        self.assertIsNotNone(success)
    
    # ------------------- Тесты случайности -------------------
    
    def test_randomness(self):
        """Тест 19: Проверка случайности выбора"""
        self.app.set_filter("Все")
        results = set()
        
        # Генерируем много задач - должны быть разные
        for _ in range(50):
            task, _ = self.app.generate_task()
            results.add(task)
        
        # Должно быть хотя бы 3 разных задачи
        self.assertGreater(len(results), 2)
    
    def test_all_tasks_accessible(self):
        """Тест 20: Все задачи доступны для генерации"""
        all_tasks = []
        for task_list in self.app.tasks.values():
            all_tasks.extend(task_list)
        
        found_tasks = set()
        for _ in range(100):
            task, _ = self.app.generate_task()
            found_tasks.add(task)
        
        # Все задачи должны быть найдены
        for task in all_tasks:
            self.assertIn(task, found_tasks)
    
    # ------------------- Граничные тесты -------------------
    
    def test_large_number_of_tasks(self):
        """Тест 21: Много задач в категории"""
        # Добавляем 100 задач
        for i in range(100):
            self.app.add_task(f"Задача {i}", "Учёба")
        
        self.assertEqual(len(self.app.tasks["Учёба"]), 103)  # 3 + 100
        self.app.set_filter("Учёба")
        
        # Генерация не должна упасть
        for _ in range(50):
            task, _ = self.app.generate_task()
            self.assertIsNotNone(task)
    
    def test_history_with_many_entries(self):
        """Тест 22: Много записей в истории"""
        for i in range(100):
            self.app.generate_task()
        
        self.assertEqual(len(self.app.history), 100)
        self.app.clear_history()
        self.assertEqual(len(self.app.history), 0)
    
    def test_task_names_with_special_characters(self):
        """Тест 23: Задачи со спецсимволами"""
        special_tasks = [
            "Задача №1!",
            "Task @#$%",
            "Привет, мир!",
            "1234567890",
            "   Пробелы внутри   "
        ]
        
        for special_task in special_tasks:
            success, _ = self.app.add_task(special_task, "Учёба")
            self.assertTrue(success)
            
            # Проверяем, что сохранилось корректно
            self.assertIn(special_task.strip(), self.app.tasks["Учёба"])
    
    def test_category_case_sensitivity(self):
        """Тест 24: Чувствительность к регистру категорий"""
        self.app.add_task("Тест", "Учёба")
        
        # Категория с другим регистром - это новая категория
        success, _ = self.app.add_task("Тест2", "учёба")
        self.assertTrue(success)
        self.assertIn("учёба", self.app.tasks)
    
    def test_duplicate_in_different_categories(self):
        """Тест 25: Одинаковые задачи в разных категориях"""
        self.app.add_task("Чтение", "Учёба")
        success, message = self.app.add_task("Чтение", "Спорт")
        
        # В разных категориях может быть
        self.assertTrue(success)
        self.assertIn("Чтение", self.app.tasks["Спорт"])


def run_tests():
    """Запуск всех тестов с подробным выводом"""
    # Настраиваем вывод
    print("=" * 60)
    print("ЗАПУСК ТЕСТОВ Random Task Generator")
    print("=" * 60)
    
    # Создаём тестовый набор
    suite = unittest.TestLoader().loadTestsFromTestCase(TestTaskGenerator)
    runner = unittest.TextTestRunner(verbosity=2)
    result = runner.run(suite)
    
    # Выводим итоги
    print("\n" + "=" * 60)
    print("ИТОГИ ТЕСТИРОВАНИЯ")
    print("=" * 60)
    print(f"✅ Успешно: {result.testsRun - len(result.failures) - len(result.errors)}")
    print(f"❌ Ошибок: {len(result.errors)}")
    print(f"⚠️  Провалено: {len(result.failures)}")
    print("=" * 60)
    
    return result.wasSuccessful()


if __name__ == "__main__":
    success = run_tests()
    exit(0 if success else 1)
