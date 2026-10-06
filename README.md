import logging
from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import ApplicationBuilder, CommandHandler, CallbackQueryHandler, ContextTypes

# Logging (xatoliklarni kuzatish uchun)
logging.basicConfig(
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    level=logging.INFO
)

# Start komandasi
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_first_name = update.effective_user.first_name
    
    # Asosiy menyu tugmalari
    keyboard = [
        [InlineKeyboardButton("📚 Ona tili va adabiyot", callback_data='ona_tili')],
        [InlineKeyboardButton("🇬🇧 Ingliz tili (IELTS / CEFR)", callback_data='ingliz_tili')],
        [InlineKeyboardButton("✍️ Testlar va topshiriqlar", callback_data='testlar')],
        [InlineKeyboardButton("ℹ️ Yordam", callback_data='help')]
    ]
    reply_markup = InlineKeyboardMarkup(keyboard)

    text = (
        f"Assalomu alaykum, {user_first_name}!\n\n"
        f"<b>Milliy Sertifikat</b> botiga xush kelibsiz! 📚\n\n"
        f"Qaysi boʻlim boʻyicha tayyorgarlik koʻrmoqchisiz? Quyi menyudan tanlang:"
    )
    
    if update.message:
        await update.message.reply_text(text, parse_mode='HTML', reply_markup=reply_markup)
    elif update.callback_query:
        await update.callback_query.message.edit_text(text, parse_mode='HTML', reply_markup=reply_markup)

# Tugmalar bosilganda ishlaydigan funksiya
async def button_handler(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = update.callback_query
    await query.answer()

    back_button = [[InlineKeyboardButton("⬅️ Orqaga", callback_data='main_menu')]]
    reply_markup = InlineKeyboardMarkup(back_button)

    if query.data == 'ona_tili':
        await query.message.edit_text(
            "📚 <b>Ona tili va adabiyot boʻlimi</b>\n\n"
            "Bu yerda siz gramatika, fonetika, leksikologiya hamda adabiyotdan metodik qoʻllanmalarni topishingiz mumkin.",
            parse_mode='HTML',
            reply_markup=reply_markup
        )
    elif query.data == 'ingliz_tili':
        await query.message.edit_text(
            "🇬🇧 <b>Ingliz tili boʻlimi</b>\n\n"
            "Grammar, Reading va Listening topshiriqlari hamda sertifikat namunalari bu yerga joylanadi.",
            parse_mode='HTML',
            reply_markup=reply_markup
        )
    elif query.data == 'testlar':
        await query.message.edit_text(
            "✍️ <b>Testlar va Milliy sertifikat namunalari</b>\n\n"
            "Yaqin orada bu boʻlimga onlayn testlar qoʻshiladi.",
            parse_mode='HTML',
            reply_markup=reply_markup
        )
    elif query.data == 'help':
        await query.message.edit_text(
            "ℹ️ <b>Yordam boʻlimi</b>\n\n"
            "Bot boʻyicha taklif va savollaringiz boʻlsa, administrator bilan bogʻlaning.",
            parse_mode='HTML',
            reply_markup=reply_markup
        )
    elif query.data == 'main_menu':
        await start(update, context)

​8884206258:AAH3pT3Vys4-R7_nkKTQXUKRSkqYcD0rVW8
    # BotFather'dan olingan TOKEN'ni shu yerga qo'yasiz:
    TOKEN = "YOUR_BOT_TOKEN_HERE"
    
    app = ApplicationBuilder().token(TOKEN).build()
    
    app.add_handler(CommandHandler("start", start))
    app.add_handler(CallbackQueryHandler(button_handler))
    
    print("Bot ishga tushdi...")
    app.run_polling()
    # milliy-sertifikat-bot
