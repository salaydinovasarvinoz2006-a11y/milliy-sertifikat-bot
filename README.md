import logging
from telegram import Update
from telegram.ext import ApplicationBuilder, CommandHandler, ContextTypes

logging.basicConfig(
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    level=logging.INFO
)

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_name = update.effective_user.first_name
    await update.message.reply_text(f"Assalomu alaykum, {user_name}! Botingizga xush kelibsiz.")

if __name__ == '__main__':
    TOKEN = "8884206258:AAH3pT3Vys4-R7_nkKTQXUKRSkqYcDOrVW8"
    
    app = ApplicationBuilder().token(TOKEN).build()
    app.add_handler(CommandHandler('start', start))
    app.run_polling()
    
