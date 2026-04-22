import tkinter as tk
from tkinter import ttk, messagebox, simpledialog
import json
import os
import random

class RandomTaskGenerator:
    def __init__(self, root):
        self.root = root
        self.root.title("Random Task Generator")
        self.history_file = "task_history.json"
        self.tasks_file = "tasks.json"

        # Предопределённые задачи по категориям
        self.default_tasks = {
            "Учёба": ["Прочитать статью", "Выучить 10 слов", "Решить 5 задач", "Поработать над проектом"],
            "Спорт": ["Сделать зарядку", "Пробежать 3 км", "100 приседаний", "Йога 30 минут"],
            "Работа": ["Ответить на письма", "Составить отчёт", "Провести встречу", "Проверить документы"]
        }

        self.load_data()
        self.setup_ui()

    def load_data(self):
        # Загрузка задач
        if os.path.exists(self.tasks_file):
            with open(self.tasks_file, 'r', encoding='utf-8') as f:
                self.tasks = json.load(f)
        else:
            self.tasks = self.default_tasks
            self.save_tasks()

        # Загрузка истории
        if os.path.exists(self.history_file):
