import telebot
from telebot.types import InlineKeyboardMarkup, InlineKeyboardButton

TOKEN = "7986330881:AAEC64tVKdci1gmDDJ8qV41UBOHfFzW2hl8"
ADMIN_ID = 6330863360
PAYMENT_LINK = "https://google.com"

bot = telebot.TeleBot(TOKEN)

user_state = {}

DOGS_PRICES = {
    "1000": 5200,
    "2000": 10400,
    "5000": 26000,
    "10000": 52000,
    "20000": 104000,
    "50000": 260000
}

STAR_PRICES = {
    "1": 2700,
    "5": 13500,
    "10": 27000,
    "50": 135000,
    "100": 270000,
    "500": 1350000
}

# ---------------- START ----------------
@bot.message_handler(commands=['start'])
def start(message):
    markup = InlineKeyboardMarkup(row_width=1)
    markup.add(
        InlineKeyboardButton("💰 قیمت خرید داگز 🐶", callback_data="dogs"),
        InlineKeyboardButton("💰 قیمت خرید استارز 🌟", callback_data="stars"),
        InlineKeyboardButton("💳 موجودی حساب", callback_data="balance")
    )
    bot.send_message(message.chat.id, "به فروشگاه داگاستارز 🐶🌟 خوش‌آمدید!", reply_markup=markup)


# ---------------- CALLBACK ----------------
@bot.callback_query_handler(func=lambda call: True)
def callback(call):
    bot.answer_callback_query(call.id)

    if call.data == "dogs":
        markup = InlineKeyboardMarkup(row_width=2)
        for qty in DOGS_PRICES:
            markup.add(InlineKeyboardButton(f"{qty} 🐶", callback_data=f"d_{qty}"))
        bot.send_message(call.message.chat.id, "تعداد داگز را انتخاب کنید:", reply_markup=markup)

    elif call.data.startswith("d_"):
        user_state[call.from_user.id] = "buying_dogs"
        qty = call.data.replace("d_", "")
        price = DOGS_PRICES[qty]

        markup = InlineKeyboardMarkup()
        markup.add(InlineKeyboardButton("خرید 💳", url=PAYMENT_LINK))

        bot.send_message(call.message.chat.id, f"💰 قیمت {qty} 🐶: {price} تومان", reply_markup=markup)
        bot.send_message(call.message.chat.id, "بعد از پرداخت، عکس رسید🧾 را ارسال کنید تا خرید شما تکمیل شود ✅")

    elif call.data == "stars":
        markup = InlineKeyboardMarkup(row_width=2)
        for qty in STAR_PRICES:
            markup.add(InlineKeyboardButton(f"{qty} 🌟", callback_data=f"s_{qty}"))
        bot.send_message(call.message.chat.id, "تعداد استارز را انتخاب کنید:", reply_markup=markup)

    elif call.data.startswith("s_"):
        user_state[call.from_user.id] = "buying_stars"
        qty = call.data.replace("s_", "")
        price = STAR_PRICES[qty]

        markup = InlineKeyboardMarkup()
        markup.add(InlineKeyboardButton("خرید 💳", url=PAYMENT_LINK))

        bot.send_message(call.message.chat.id, f"💰 قیمت {qty} 🌟: {price} تومان", reply_markup=markup)
        bot.send_message(call.message.chat.id, "بعد از پرداخت، عکس رسید🧾 را ارسال کنید تا خرید شما تکمیل شود ✅")


# ---------------- عکس رسید ----------------
@bot.message_handler(content_types=['photo'])
def photo_handler(message):
    bot.forward_message(ADMIN_ID, message.chat.id, message.message_id)
    user_id = message.from_user.id

    if user_id in user_state:
        if user_state[user_id] == "buying_stars":
            bot.send_message(message.chat.id, "رسید شما دریافت شد ✅")
            bot.send_message(message.chat.id, "با تشکر از خرید شما آدرس پست خود را ارسال کنید🌟")
            user_state[user_id] = "waiting_address"
        elif user_state[user_id] == "buying_dogs":
            bot.send_message(message.chat.id, "خرید شما ثبت شد و در صورت انجام واریز سفارش شما در کمتر از ۳۰ دقیقه به @UltraWallet_bot واریز می‌گردد🎉✅")
            del user_state[user_id]


# ---------------- آدرس استارز ----------------
@bot.message_handler(content_types=['text'])
def text_handler(message):
    user_id = message.from_user.id
    if user_id in user_state and user_state[user_id] == "waiting_address":
        bot.send_message(message.chat.id, "خرید شما ثبت شد و در صورت انجام واریز سفارش شما در کمتر از ۳۰ دقیقه انجام می‌گردد 🎉✅")
        bot.send_message(ADMIN_ID, f"📦 آدرس جدید استارز:\n{message.text}\nاز کاربر: {user_id}")
        del user_state[user_id]


print("Bot is running...")
bot.infinity_polling()
