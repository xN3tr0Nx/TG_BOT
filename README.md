# TG_bot
## токен Telegram бота
API_TOKEN = '7729422654:AAFJb5d7m94NpiOfSux7hEjv1R1o04GDecE'
bot = telebot.TeleBot(API_TOKEN)
# Словарь для хранения товаров
products = {}
current_action = None

@bot.message_handler(commands=['start'])
def start(message):
    bot.reply_to(message, "Здравствуйте, я ваш помощник, выберете действие:\n"
                          "1) /spisok\n"
                          "2) /delete\n"
                          "3) /new_data\n"
                          "4) /add")
@bot.message_handler(commands=['spisok'])
def list_products(message):
    if products:
        response = "Список товаров:\n"
        for id, item in products.items():
            response += f"ID: {id}, Название: {item['name']}, Модель: {item['model']}, Состояние: {item['status']}\n"
    else:
        response = "Товаров нет."
    bot.reply_to(message, response)

@bot.message_handler(commands=['delete'])
def delete_product(message):
    global current_action
    current_action = 'delete'
    bot.reply_to(message, "Введите название товара для удаления:")

@bot.message_handler(func=lambda message: current_action == 'delete')
def confirm_delete(message):
    name_to_delete = message.text
    for id, item in list(products.items()):
        if item['name'] == name_to_delete:
            del products[id]
            bot.reply_to(message, f"Товар '{name_to_delete}' успешно удален.")
            return
    bot.reply_to(message, f"Товар '{name_to_delete}' не найден.")

@bot.message_handler(commands=['new_data'])
def update_product(message):
    global current_action
    current_action = 'update'
    bot.reply_to(message, "Введите ID товара для обновления состояния:")
