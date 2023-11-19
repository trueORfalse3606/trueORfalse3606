import telebot
import requests
import json
import time

# Récupérer le jeton du bot
bot_token = "votre_jeton_de_bot"

# Créer une instance du bot
bot = telebot.TeleBot(bot_token)

# Définir un gestionnaire de messages
@bot.message_handler(commands=["start"])
def start(message):
    # Envoyer un message à l'utilisateur
    bot.send_message(message.chat.id, """Que puis-je faire pour vous ?

* Recevoir un signal : envoyez le message "recevoir signal".

* Autres demandes : veuillez préciser.

* Pour quitter, envoyez le message "quitter".""")

@bot.message_handler(commands=["recevoir signal"])
def recevoir_signal(message):
    # URL du jeu
    url = 'https://api.1win.com/games/lucky-jet/get-state'

    # Obtention des données du jeu
    data = get_data(url)

    # Prédiction du résultat du jeu
    result = predict(data)

    # Obtention de l'heure actuelle
    local_time = get_local_time()

    # Affichage de la prédiction
    bot.send_message(message.chat.id, f'Prédiction pour {local_time.tm_hour}h-{local_time.tm_min}min-{local_time.tm_sec}s : {result}')

# Lancer le bot
bot.polling()

# Fonction pour obtenir les données du jeu
def get_data(url):
    response = requests.get(url)
    if response.status_code == 200:
        return json.loads(response.content)
    else:
        return None

# Fonction pour prédire le résultat du jeu
def predict(data):
    # Liste des résultats précédents
    previous_results = []
    for result in data['results']:
        previous_results.append(result['value'])

    # Moyenne des résultats précédents
    mean = sum(previous_results) / len(previous_results)

    # Prédiction
    if mean > 2.9:
        return 2.9
    return mean

# Fonction pour obtenir l'heure actuelle en UTC
def get_utc_time():
    return time.gmtime()

# Fonction pour obtenir l'heure actuelle en local
def get_local_time():
    utc_time = get_utc_time()
    local_time = time.localtime(time.mktime(utc_time) - timedelta(hours=3))
    return local_time
