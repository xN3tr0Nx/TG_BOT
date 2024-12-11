# TG_bot

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
