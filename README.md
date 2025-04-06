import telebot
from pymongo import MongoClient

API_TOKEN = '7086533105:AAFbRdYjsOF0XlQzTy9R0TQLdAX6zNmYHHA'

MONGO_URI = 'mongodb+srv://rabbit:rabbitt@rabbit.dxuillb.mongodb.net/?retryWrites=true&w=majority&appName=rabbit'

bot = telebot.TeleBot(API_TOKEN)
client = MongoClient(MONGO_URI)
db = client['rabbit']            # Database name
collection = db['movies']        # Collection name (make sure you create this)

@bot.message_handler(commands=['start'])
def send_welcome(message):
    bot.reply_to(message, "Welcome to MovieBot!\nSend me a hindi dubbed movie name and I'll send you the download link. But you must install the DISKWALA app.")

@bot.message_handler(func=lambda message: True)
def search_movie(message):
    query = message.text.strip().lower()
    result = collection.find_one({"title": {"$regex": query, "$options": "i"}})

    if result:
        title = result.get('title', 'Unknown Title')
        year = result.get('year', 'N/A')
        gplink = result.get('gplink', 'No link found')
        reply = f"**{title} ({year})**\nDownload: {gplink}"
    else:
        reply = "Sorry, I couldn't find that movie. Try a different name."

    bot.send_message(message.chat.id, reply, parse_mode='Markdown')

bot.polling()
