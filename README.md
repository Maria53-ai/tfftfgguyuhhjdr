import tkinter as tk
from tkinter import ttk, messagebox, simpledialog
import json
import os
from datetime import datetime

class WeatherLogApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Weather Log")
        self.data_file = "weather_data.json"
        self.load_data()
        self.setup_ui()

    def load_data(self):
        if os.path.exists(self.data_file):
            with open(self.data_file, 'r', encoding='utf-8') as f:
                self.records = json.load(f)
        else:
            self.records = []

    def save_data(self):
        with open(self.data_file, 'w', encoding='utf-8') as f:
            json.dump(self.records, f, ensure_ascii=False, indent=4)

    def setup_ui(self):
        # Поля ввода
        ttk.Label(self.root, text="Дата (YYYY-MM-DD):").grid(row=0, column=0, padx=5, pady=5, sticky='w')
        self.date_entry = ttk.Entry(self.root)
        self.date_entry.grid(row=0, column=1, padx=5, pady=5)

        ttk.Label(self.root, text="Температура (°C):").grid(row=1, column=0, padx=5, pady=5, sticky='w')
        self.temp_entry = ttk.Entry(self.root)
        self.temp_entry.grid(row=1, column=1, padx=5, pady=5)

        ttk.Label(self.root, text="Описание погоды:").grid(row=2, column=0, padx=5, pady=5, sticky='w')
        self.desc_entry = ttk.Entry(self.root)
        self.desc_entry.grid(row=2, column=1, padx=5, pady=5)

        ttk.Label(self.root, text="Осадки:").grid(row=3, column=0, padx=5, pady=5, sticky='w')
        self.precip_var = tk.BooleanVar()
        ttk.Checkbutton(self.root, variable=self.precip_var, text="Да").grid(row=3, column=1, sticky='w', padx=5)

        # Кнопка добавления записи
        self.add_btn = ttk.Button(self.root, text="Добавить запись", command=self.add_record)
        self.add_btn.grid(row=4, column=0, columnspan=2, pady=10)

        # Фильтры
        ttk.Label(self.root, text="Фильтр по дате:").grid(row=5, column=0, padx=5, pady=5, sticky='w')
        self.filter_date_entry = ttk.Entry(self.root)
        self.filter_date_entry.grid(row=5, column=1, padx=5, pady=5)

        ttk.Label(self.root, text="Фильтр по температуре (>):").grid(row=6, column=0, padx=5, pady=5, sticky='w')
        self.filter_temp_entry = ttk.Entry(self.root)
        self.filter_temp_entry.grid(row=6, column=1, padx=5, pady=5)

        self.filter_btn = ttk.Button(self.root, text="Применить фильтры", command=self.apply_filters)
        self.filter_btn.grid(row=7, column=0, columnspan=2, pady=10)

        # Таблица записей
        columns = ("Date", "Temperature", "Description", "Precipitation")
        self.tree = ttk.Treeview(self.root, columns=columns, show="headings")
        for col in columns:
            self.tree.heading(col, text=col)
            self.tree.column(col,
