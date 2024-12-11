# TG_bot
import telebot

# Ваш токен Telegram бота
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

@bot.message_handler(func=lambda message: current_action == 'update')
def get_new_state(message):
    product_id = message.text
    if product_id in products:
        bot.reply_to(message, "Введите новое состояние:")
        current_action = 'update_state'
        bot.register_next_step_handler(message, lambda m: update_state(m, product_id))
    else:
        bot.reply_to(message, "Товар с таким ID не найден.")

def update_state(message, product_id):
    new_state = message.text
    products[product_id]['status'] = new_state
    bot.reply_to(message, "Данные обновлены.")

@bot.message_handler(commands=['add'])
def add_product(message):
    global current_action
    current_action = 'add'
    bot.reply_to(message, "Введите название нового товара:")

@bot.message_handler(func=lambda message: current_action == 'add')
def enter_product_details(message):
    product_name = message.text
    bot.reply_to(message, "Введите ID товара:")
    current_action = 'add_id'
    bot.register_next_step_handler(message, lambda m: add_id(m, product_name))

def add_id(message, product_name):
    product_id = message.text
    bot.reply_to(message, "Введите модель товара:")
    current_action = 'add_model'
    bot.register_next_step_handler(message, lambda m: add_model(m, product_id, product_name))

def add_model(message, product_id, product_name):
    product_model = message.text
    bot.reply_to(message, "Введите состояние товара:")
    current_action = 'add_status'
    bot.register_next_step_handler(message, lambda m: finalize_addition(m, product_id, product_name, product_model))

def finalize_addition(message, product_id, product_name, product_model):
    product_status = message.text
    products[product_id] = {'name': product_name, 'model': product_model, 'status': product_status}
    bot.reply_to(message, f"Товар '{product_name}' успешно добавлен.\nСписок товаров: {products}")

# Запуск бота
bot.polling()
# Дашборд
# dashbord
#https://docs.google.com/spreadsheets/d/1NMYaZkFNUUkofuRxJonx5sMa78Dt5HrWhq3LHPOazVE/export?format=csv
import dash
from dash import dcc, html, dash_table
import pandas as pd

# Загрузка данных
url = 'https://docs.google.com/spreadsheets/d/1NMYaZkFNUUkofuRxJonx5sMa78Dt5HrWhq3LHPOazVE/export?format=csv'
df = pd.read_csv(url)

# Создание приложения Dash
app = dash.Dash(__name__)

app.layout = html.Div(children=[
    html.H1(children='Проверка качества товаров', style={'color': 'red'}),
    
    dcc.Dropdown(
        id='status-filter',
        options=[
            {'label': 'Все', 'value': 'ALL'},
            {'label': 'Брак', 'value': 'Брак'},
            {'label': 'Удовлетворительно', 'value': 'Удовлетворительно'}
        ],
        value='ALL',
        clearable=False
    ),

    dash_table.DataTable(
        id='table',
        columns=[{"name": i, "id": i} for i in df.columns],
        data=df.to_dict('records'),
        filter_action='native',
        page_action='native',
        page_size=10
    )
])

@app.callback(
    dash.dependencies.Output('table', 'data'),
    [dash.dependencies.Input('status-filter', 'value')]
)
def update_table(selected_status):
    if selected_status == 'ALL':
        filtered_df = df
    else:
        filtered_df = df[df['Состояние'] == selected_status]
    return filtered_df.to_dict('records')

if __name__ == '__main__':
    app.run_server(debug=True)
