import telebot
from telebot import types
import requests

TELEGRAM_TOKEN = '8058995226:AAE8vFCF4JBxOQyoBz1VkOnE4FTsp5MPey0'
API_KEY = 'c9b6ade6ab2a471fba5df4cdbd06fbb2'

bot = telebot.TeleBot(TELEGRAM_TOKEN)
user_data = {}
CURRENCIES = ["USD", "KZT", "RUB", "TRY", "EUR"]


def get_conversion_rate(from_cur, to_cur):
    url = f"https://api.twelvedata.com/exchange_rate?symbol={from_cur}/{to_cur}&apikey={API_KEY}"
    res = requests.get(url).json()
    return float(res['rate']) if 'rate' in res else None


def show_main_menu(chat_id):
    markup = types.ReplyKeyboardMarkup(resize_keyboard=True, row_width=2)
    btn1 = types.KeyboardButton("📈 Показать курс")
    btn2 = types.KeyboardButton("💱 Обмен")
    btn3 = types.KeyboardButton("⚙️ Настройки")
    markup.add(btn1, btn2, btn3)
    bot.send_message(chat_id, "Главное меню:", reply_markup=markup)


@bot.message_handler(commands=['start'])
def start(message):
    user_data[message.chat.id] = {}
    show_main_menu(message.chat.id)
    bot.send_message(message.chat.id, "🔔 Реклама: Лучший обмен по курсу — только у нас!")


@bot.message_handler(func=lambda m: m.text == "📈 Показать курс")
def show_rates(message):
    bot.send_message(message.chat.id, "⏳ В ожидании...")
    
    base = "USD"
    result = "🌍 Валюты СНГ:\n"
    for cur in ["UZS", "KZT", "RUB", "TRY", "EUR"]:
        rate = get_conversion_rate(base, cur)
        if rate:
            result += f"{base} → {cur} = {round(rate, 4)}\n"
    
    bot.send_message(message.chat.id, result)
    show_main_menu(message.chat.id)


@bot.message_handler(func=lambda m: m.text == "💱 Обмен")
def exchange_step1(message):
    user_data[message.chat.id] = {}
    bot.send_message(message.chat.id, "⏳ В ожидании...")
    
    markup = types.ReplyKeyboardMarkup(resize_keyboard=True, row_width=2)
    buttons = [types.KeyboardButton(cur) for cur in CURRENCIES]
    markup.add(*buttons)
    markup.add(types.KeyboardButton("↩️ Назад"))
    
    bot.send_message(message.chat.id, "Выберите валюту, которую хотите конвертировать:", reply_markup=markup)
    bot.register_next_step_handler(message, exchange_step2)


def exchange_step2(message):
    if message.text == "↩️ Назад":
        return show_main_menu(message.chat.id)
    
    chat_id = message.chat.id
    if message.text not in CURRENCIES:
        bot.send_message(chat_id, "❌ Пожалуйста, выберите валюту из списка.")
        return exchange_step1(message)
    
    user_data[chat_id]['from'] = message.text
    bot.send_message(chat_id, "⏳ В ожидании...")
    
    markup = types.ReplyKeyboardMarkup(resize_keyboard=True, row_width=2)
    buttons = [types.KeyboardButton(cur) for cur in CURRENCIES if cur != user_data[chat_id]['from']]
    markup.add(*buttons)
    markup.add(types.KeyboardButton("↩️ Назад"))
    
    bot.send_message(chat_id, "Выберите валюту, в которую хотите конвертировать:", reply_markup=markup)
    bot.register_next_step_handler(message, exchange_amount)


def exchange_amount(message):
    if message.text == "↩️ Назад":
        return show_main_menu(message.chat.id)
    
    chat_id = message.chat.id
    if message.text not in CURRENCIES:
        bot.send_message(chat_id, "❌ Пожалуйста, выберите валюту из списка.")
        return exchange_step2(message)
    
    user_data[chat_id]['to'] = message.text
    bot.send_message(chat_id, "⏳ В ожидании...")
    bot.send_message(chat_id, f"Введите сумму в {user_data[chat_id]['from']} (например: 100):")
    bot.register_next_step_handler(message, calculate_exchange)


def calculate_exchange(message):
    chat_id = message.chat.id
    
    if message.text == "↩️ Назад":
        return show_main_menu(chat_id)
    
    try:
        amount = float(message.text)
        from_cur = user_data[chat_id]['from']
        to_cur = user_data[chat_id]['to']
        rate = get_conversion_rate(from_cur, to_cur)
        
        if rate is None:
            bot.send_message(chat_id, "❌ Не удалось получить курс. Попробуйте позже.")
            return show_main_menu(chat_id)
        
        result = round(amount * rate, 2)
        bot.send_message(chat_id, f"💱 {amount} {from_cur} = {result} {to_cur} (курс: {rate:.4f})")
        
        # Предлагаем сделать еще один обмен
        markup = types.ReplyKeyboardMarkup(resize_keyboard=True, row_width=2)
        markup.add(types.KeyboardButton("💱 Сделать еще один обмен"), types.KeyboardButton("↩️ Назад"))
        bot.send_message(chat_id, "Хотите сделать еще один обмен?", reply_markup=markup)
        bot.register_next_step_handler(message, handle_another_exchange)
        
    except ValueError:
        bot.send_message(chat_id, "❌ Ошибка. Введите число (например: 100):")
        bot.register_next_step_handler(message, calculate_exchange)


def handle_another_exchange(message):
    if message.text == "💱 Сделать еще один обмен":
        exchange_step1(message)
    else:
        show_main_menu(message.chat.id)


@bot.message_handler(func=lambda m: m.text == "⚙️ Настройки")
def settings(message):
    markup = types.ReplyKeyboardMarkup(resize_keyboard=True, row_width=2)
    btn1 = types.KeyboardButton("🌐 Выбрать язык")
    btn2 = types.KeyboardButton("📞 Связь с разработчиком")
    btn3 = types.KeyboardButton("↩️ Назад")
    markup.add(btn1, btn2, btn3)
    bot.send_message(message.chat.id, "Настройки:", reply_markup=markup)


@bot.message_handler(func=lambda m: m.text == "🌐 Выбрать язык")
def language_settings(message):
    markup = types.ReplyKeyboardMarkup(resize_keyboard=True, row_width=2)
    btn1 = types.KeyboardButton("Русский")
    btn2 = types.KeyboardButton("O'zbekcha")
    btn3 = types.KeyboardButton("Қазақша")
    btn4 = types.KeyboardButton("English")
    btn5 = types.KeyboardButton("↩️ Назад")
    markup.add(btn1, btn2, btn3, btn4, btn5)
    bot.send_message(message.chat.id, "Выберите язык:", reply_markup=markup)


@bot.message_handler(func=lambda m: m.text == "📞 Связь с разработчиком")
def contact_support(message):
    bot.send_message(
        message.chat.id,
        "Для связи с разработчиком:\nТелефон: +998887177071\nTelegram: @expressproway"
    )
    show_main_menu(message.chat.id)


@bot.message_handler(func=lambda m: m.text == "↩️ Назад")
def back_to_menu(message):
    show_main_menu(message.chat.id)


bot.polling(none_stop=True)
