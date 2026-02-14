@bot.callback_query_handler(func=lambda call: True)
def callback_query(call):
    try:

        if call.data == "price_dogs":
            markup = InlineKeyboardMarkup(row_width=2)
            for qty in DOGS_PRICES:
                markup.add(InlineKeyboardButton(f"{qty} 🐶", callback_data=f"dogs_{qty}"))
            bot.send_message(call.message.chat.id,
                             "تعداد داگز را انتخاب کنید:",
                             reply_markup=markup)

        elif call.data.startswith("dogs_"):
            qty = call.data.split("_")[1]
            price = DOGS_PRICES[qty]

            bot.send_message(call.message.chat.id,
                             f"💰 قیمت {qty} 🐶: {price} تومان")

            bot.send_message(call.message.chat.id,
                             "لطفاً بعد از انجام تراکنش عکس رسید🧾تراکنش را برای ربات ارسال کنید تا خرید شما تکمیل شود✅")

        elif call.data == "price_stars":
            markup = InlineKeyboardMarkup(row_width=2)
            for qty in STAR_PRICES:
                markup.add(InlineKeyboardButton(f"{qty} 🌟", callback_data=f"stars_{qty}"))
            bot.send_message(call.message.chat.id,
                             "تعداد استارز را انتخاب کنید:",
                             reply_markup=markup)

        elif call.data.startswith("stars_"):
            qty = call.data.split("_")[1]
            price = STAR_PRICES[qty]

            bot.send_message(call.message.chat.id,
                             f"💰 قیمت {qty} 🌟: {price} تومان")

            bot.send_message(call.message.chat.id,
                             "لطفاً بعد از انجام تراکنش عکس رسید🧾تراکنش را برای ربات ارسال کنید تا خرید شما تکمیل شود✅")

        elif call.data == "balance":
            bot.send_message(call.message.chat.id,
                             "💳 موجودی شما: 0 واحد")

    except Exception as e:
        print("ERROR:", e)

    # 🔥 این خط حتماً باید آخر باشد
    bot.answer_callback_query(call.id)
