import telebot
import requests
import random
from datetime import datetime
import schedule
import time
import threading

# === Configuration ===
BOT_TOKEN = "7989292150:AAH20ZGzPCapo-DLiqFOgX2HtzrRwJPXOBI"
CHANNEL_USERNAME = "@bestwinner224"
PROMO_CODE = "MDC24"
API_FOOTBALL_KEY = "681966372002f8f2d9cc615819019db5"
ADMIN_ID = 5426221156
VIP_USERS = [5426221156]  # Liste des ID Telegram VIP

bot = telebot.TeleBot(BOT_TOKEN)

# === Types de pronostics possibles ===
TYPES_PARIS = [
    "Victoire de l'équipe 1",
    "Victoire de l'équipe 2",
    "Match nul ou équipe 1 gagne",
    "Les deux équipes marquent",
    "+2.5 buts dans le match",
    "-2.5 buts dans le match"
]

# === Fonction pour générer un pari aléatoire ===
def generer_pronostic():
    pari = random.choice(TYPES_PARIS)
    cote = round(random.uniform(1.50, 2.80), 2)
    return pari, cote

# === Récupérer les matchs depuis l'API FOOTBALL ===
def get_matchs_du_jour_avec_pronostics(nb_matchs=2):
    url = "https://v3.football.api-sports.io/fixtures"
    headers = {"x-apisports-key": API_FOOTBALL_KEY}
    params = {
        "date": datetime.now().strftime("%Y-%m-%d"),
        "timezone": "Africa/Conakry"
    }
    response = requests.get(url, headers=headers, params=params)
    if response.status_code == 200:
        data = response.json()
        matchs = data.get("response", [])
        if not matchs:
            return "Désolé, aucun match disponible aujourd'hui."
        message = "📊 *Pronostics du jour :*\n"
        for match in matchs[:nb_matchs]:
            equipe1 = match["teams"]["home"]["name"]
            equipe2 = match["teams"]["away"]["name"]
            heure = match["fixture"]["date"][11:16]
            pari, cote = generer_pronostic()
            message += (
                f"\n• {equipe1} vs {equipe2} à {heure}\n"
                f"  ➤ Pronostic : {pari}\n"
                f"  ➤ Cote : {cote}\n"
            )
        return message
    else:
        return "Erreur de connexion à l'API."

# === Fonction message info canal matin et soir ===
def envoyer_infos_canal():
    moment = "Bonjour" if datetime.now().hour < 12 else "Bonsoir"
    texte = (
        "Bienvenue sur le canal où les pronostics riment avec résultats ! "
        "Ici, on ne plaisante pas avec les chiffres : analyses pointues, cotes sélectionnées et gains réguliers sont au rendez-vous. "
        f"Rejoignez-nous pour transformer vos paris en profits !\n\n{moment} à tous !\n\n"
        "Restez connectés, les pronostics arrivent à 10h !"
    )
    bouton = telebot.types.InlineKeyboardMarkup()
    bouton.add(telebot.types.InlineKeyboardButton("Voir les bonus", callback_data="bonus"))
    bot.send_message(CHANNEL_USERNAME, texte, reply_markup=bouton)

# === Envoyer les pronostics publics ===
def envoyer_pronostics():
    message = get_matchs_du_jour_avec_pronostics()
    bot.send_message(CHANNEL_USERNAME, message, parse_mode="Markdown")
    vip_msg = (
        "🔥 Pour recevoir *les pronostics VIP 100% sûrs*, avec des types spéciaux (score exact, les deux marquent, double chance...), "
        f"clique ici : @{bot.get_me().username}"
    )
    bouton = telebot.types.InlineKeyboardMarkup()
    bouton.add(telebot.types.InlineKeyboardButton("Voir les bonus", callback_data="bonus"))
    bot.send_message(CHANNEL_USERNAME, vip_msg, reply_markup=bouton, parse_mode="Markdown")

    # Envoyer les pronostics VIP à ceux qui ont payé
    for user_id in VIP_USERS:
        message_vip = get_matchs_du_jour_avec_pronostics(2)
        bot.send_message(user_id, "🎯 *Pronostics VIP du jour :*\n" + message_vip, parse_mode="Markdown")
        bot.send_message(ADMIN_ID, f"Le VIP {user_id} a reçu ses pronostics.")

# === Gérer les bonus ===
@bot.callback_query_handler(func=lambda call: call.data == "bonus")
def envoyer_bonus(call):
    texte = (
        "Voici les 2 manières de gagner des bonus :\n\n"
        "1️⃣ *Partage du lien du canal* : tu gagnes 1.000 GNF à chaque partage de ce lien :\n"
        f"{CHANNEL_USERNAME}\n\n"
        "2️⃣ *Partage du code promo* : tu gagnes 10.000 GNF si une personne recharge 20.000 GNF avec ce code :\n"
        f"`{PROMO_CODE}`"
    )
    bot.send_message(call.from_user.id, texte, parse_mode="Markdown")

# === Commande /start ===
@bot.message_handler(commands=["start"])
def start(msg):
    bot.reply_to(msg, "Bienvenue sur le bot officiel de BestWinner224 !")

# === Programmation automatique ===
def scheduler():
    schedule.every().day.at("07:00").do(envoyer_infos_canal)
    schedule.every().day.at("19:00").do(envoyer_infos_canal)
    schedule.every().day.at("10:00").do(envoyer_pronostics)
    while True:
        schedule.run_pending()
        time.sleep(1)

# === Lancer dans un thread séparé ===
threading.Thread(target=scheduler).start()

# === Démarrer le bot ===
bot.infinity_polling()
