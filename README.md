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
