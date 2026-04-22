# tfftfgguyuhhjdrimport tkinter as tk
from tkinter import ttk, messagebox
import requests
import json
import os

class CurrencyConverter:
    def __init__(self, root):
        self.root = root
        self.root.title("Currency Converter")
        self.history_file = "conversion_history.json"
        self.load_history()

        # API ключ (замените на ваш)
        self.api_key = "YOUR_API_KEY"

        self.setup_ui()

    def setup_ui(self):
        # Выбор валюты FROM
        ttk.Label(self.root, text="Из:").grid(row=0, column=0, padx=5, pady=5)
        self.from_currency = ttk.Combobox(self.root, values=self.get_currencies())
        self.from_currency.grid(row=0, column=1, padx=5, pady=5)

        # Выбор валюты TO
        ttk.Label(self.root, text="В:").grid(row=1, column=0, padx=5, pady=5)
        self.to_currency = ttk.Combobox(self.root, values=self.get_currencies())
        self.to_currency.grid(row=1, column=1, padx=5, pady=5)

        # Поле ввода суммы
        ttk.Label(self.root, text="Сумма:").grid(row=2, column=0, padx=5, pady=5)
        self.amount_entry = ttk.Entry(self.root)
        self.amount_entry.grid(row=2, column=1, padx=5, pady=5)

        # Кнопка конвертации
        self.convert_btn = ttk.Button(self.root, text="Конвертировать", command=self.convert)
        self.convert_btn.grid(row=3, column=0, columnspan=2, pady=10)

        # Таблица истории
        self.history_tree = ttk.Treeview(self.root, columns=("From", "To", "Amount", "Result"), show="headings")
        self.history_tree.heading("From", text="Из валюты")
        self.history_tree.heading("To", text="В валюту")
        self.history_tree.heading("Amount", text="Сумма")
        self.history_tree.heading("Result", text="Результат")
        self.history_tree.grid(row=4, column=0, columnspan=2, padx=5, pady=5)

        self.refresh_history()

    def get_currencies(self):
        # Список популярных валют
        return ["USD", "EUR", "RUB", "GBP", "JPY", "CNY"]

    def load_history(self):
        if os.path.exists(self.history_file):
            with open(self.history_file, 'r', encoding='utf-8') as f:
                self.history = json.load(f)
        else:
            self.history = []

    def save_history(self):
        with open(self.history_file, 'w', encoding='utf-8') as f:
            json.dump(self.history, f, ensure_ascii=False, indent=4)

    def refresh_history(self):
        for item in self.history_tree.get_children():
            self.history_tree.delete(item)
        for record in self.history:
            self.history_tree.insert("", "end", values=(
                record["from"],
                record["to"],
                record["amount"],
                record["result"]
            ))

    def convert(self):
        try:
            amount = float(self.amount_entry.get())
            if amount <= 0:
                messagebox.showerror("Ошибка", "Сумма должна быть положительным числом")
                return

            from_curr = self.from_currency.get()
            to_curr = self.to_currency.get()

            if not from_curr or not to_curr:
                messagebox.showerror("Ошибка", "Выберите валюты")
                return

            # Запрос к API
            url = f"https://api.exchangerate-api.com/v4/latest/{from_curr}"
            response = requests.get(url)
            data = response.json()

            rate = data["rates"].get(to_curr)
            if rate is None:
                messagebox.showerror("Ошибка", "Не удалось получить курс для выбранной валюты")
                return

            result = amount * rate

            # Сохранение в историю
            record = {
                "from": from_curr,
                "to": to_curr,
                "amount": amount,
                "result": round(result, 2)
            }
            self.history.append(record)
            self.save_history()
            self.refresh_history()

            messagebox.showinfo("Результат", f"{amount} {from_curr} = {result:.2f} {to_curr}")

        except ValueError:
            messagebox.showerror("Ошибка", "Введите корректное число")
        except Exception as e:
            messagebox.showerror("Ошибка", f"Произошла ошибка: {e}")

if __name__ == "__main__":
    root = tk.Tk()
    app = CurrencyConverter(root)
    root.mainloop()
