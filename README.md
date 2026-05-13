# bilet-bot
import telebot
from datetime import datetime, timedelta
import random
import re

TOKEN = "8307596159:AAES-a6TjEaAaP_j6LPogq2Eb9vsoBqtL4o"
bot = telebot.TeleBot(TOKEN)

PRET = 6
VALABILITATE_ORE = 1
bilete_active = {}

def genereaza_nr_bilet():
    return random.randint(10000000, 99999999)

@bot.message_handler(commands=['start', 'menu'])
def start(message):
    markup = telebot.types.ReplyKeyboardMarkup(row_width=2, resize_keyboard=True)
    markup.add("🎫 Cumpara bilet", "🔍 Verifica bilet", "ℹ️ Informatii")
    bot.send_message(message.chat.id, "🚌 *Bun venit la Sistemul de Bilete Electronice!*\n\nAlege o optiune:", parse_mode='Markdown', reply_markup=markup)

@bot.message_handler(func=lambda message: True)
def handle_messages(message):
    text = message.text
    if text == "🎫 Cumpara bilet":
        msg = bot.send_message(message.chat.id, "🚌 *Introdu numarul transportului (2000-2099):*", parse_mode='Markdown')
        bot.register_next_step_handler(msg, cumpara_bilet)
    elif text == "🔍 Verifica bilet":
        msg = bot.send_message(message.chat.id, "🔍 *Introdu numarul biletului (8 cifre):*", parse_mode='Markdown')
        bot.register_next_step_handler(msg, verifica_bilet)
    elif text == "ℹ️ Informatii":
        bot.send_message(message.chat.id, "ℹ️ *Sistem Bilete Electronice Moldova*\n\n💰 Pret: 6 lei\n⏰ Valabil: 1 ora\n🚌 Transport: 2000-2099", parse_mode='Markdown')
    else:
        start(message)

def cumpara_bilet(message):
    text = message.text.strip()
    if not re.fullmatch(r'\d{4}', text):
        bot.send_message(message.chat.id, "❌ Numar invalid. Permis doar 2000-2099.")
        return
    cod = int(text)
    if cod < 2000 or cod > 2099:
        bot.send_message(message.chat.id, "❌ Numar invalid. Permis doar 2000-2099.")
        return
    moment = datetime.now()
    expirare = moment + timedelta(hours=1)
    nr_bilet = genereaza_nr_bilet()
    bilete_active[nr_bilet] = {'cod': cod, 'expira': expirare}
    bot.send_message(message.chat.id, f"🎫 *Bilet achizitionat!*\n\n🧾 Numar: `{nr_bilet}`\n🚌 Transport: `{cod}`\n⏰ {moment.strftime('%H:%M')} - {expirare.strftime('%H:%M')}\n💰 Pret: 6 lei", parse_mode='Markdown')

def verifica_bilet(message):
    text = message.text.strip()
    if not text.isdigit() or len(text) != 8:
        bot.send_message(message.chat.id, "❌ Numar invalid. Trebuie 8 cifre.")
        return
    nr = int(text)
    if nr not in bilete_active:
        bot.send_message(message.chat.id, f"❌ Biletul {nr} nu a fost gasit.")
    elif datetime.now() < bilete_active[nr]['expira']:
        bot.send_message(message.chat.id, f"✅ Biletul {nr} este VALID.")
    else:
        bot.send_message(message.chat.id, f"❌ Biletul {nr} a EXPIRAT.")

print("✅ Botul a pornit!")
bot.infinity_polling()
