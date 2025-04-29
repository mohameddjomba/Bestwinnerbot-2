import telebot

TOKEN = '7561839376:AAFzMptESqnXuB7C8Y4QdLv6kqwhQ-VkleI'
bot = telebot.TeleBot(TOKEN)

@bot.message_handler(commands=['start'])
def start(message):
    markup = telebot.types.ReplyKeyboardMarkup(resize_keyboard=True)
    markup.add('1 - Obtenir le code promo', '2 - Récupérer mon cadeau', '3 - Avoir des informations')
    bot.send_message(message.chat.id, "Bienvenue sur Best Winner Guide !\nQue souhaites-tu faire ?", reply_markup=markup)

@bot.message_handler(func=lambda message: True)
def reply(message):
    if message.text.startswith('1'):
        bot.send_message(message.chat.id, "Ton code promo est : MDC24")
    elif message.text.startswith('2'):
        ask_use_code(message)
    elif message.text.startswith('3'):
        bot.send_message(message.chat.id, "Bienvenue chez Best Winner Guide !\nIci, nous te proposons les meilleurs pronostics pour maximiser tes chances de succès dans les paris sportifs.\n\nRejoins notre canal officiel : https://t.me/bestwinner224\n\nSi tu es nouveau, crée ton compte avec le code promo MDC24 pour avoir 200% de bonus sur ton premier dépôt et un remboursement de 50% sur tes pertes.")
    else:
        bot.send_message(message.chat.id, "Merci pour ton message !")

def ask_use_code(message):
    markup = telebot.types.ReplyKeyboardMarkup(resize_keyboard=True)
    markup.add('Oui', 'Non')
    bot.send_message(message.chat.id, "As-tu utilisé mon code promo ?", reply_markup=markup)
    bot.register_next_step_handler(message, check_code_use)

def check_code_use(message):
    if message.text.lower() == 'oui':
        bot.send_message(message.chat.id, "Super !\n\nEnvoie-moi maintenant une capture d’écran avec ton numéro de compte.")
        bot.register_next_step_handler(message, ask_share_link)
    else:
        bot.send_message(message.chat.id, "Voici ton code promo : MDC24\nUtilise-le pour continuer.")

def ask_share_link(message):
    bot.send_message(message.chat.id, "Maintenant, envoie-moi la capture d’écran de ton lien partagé.")
    bot.register_next_step_handler(message, final_step)

def final_step(message):
    bot.send_message(message.chat.id, "Parfait ! Contacte @djomba2024 pour récupérer ton cadeau.")

bot.infinity_polling()
