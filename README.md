import logging
from datetime import datetime
from telegram import Update
from telegram.ext import ApplicationBuilder, CommandHandler, MessageHandler, filters, ContextTypes

logging.basicConfig(format='%(asctime)s - %(name)s - %(levelname)s - %(message)s', level=logging.INFO)

ADMIN_ID = 8880664748
CHANNEL_USERNAME = "@Salaydinova_Ona_tili"

TESTS = {}

async def check_subscription(user_id, context):
    try:
        member = await context.bot.get_chat_member(chat_id=CHANNEL_USERNAME, user_id=user_id)
        if member.status in ['left', 'kicked']:
            return False
        return True
    except Exception:
        return True

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text("Assalomu alaykum! Test javoblarini yuborish uchun quyidagi formatda yozing:\n\nTestKodi*abcd...")

async def add_test(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id
    if ADMIN_ID != 0 and user_id != ADMIN_ID:
        await update.message.reply_text("Siz admin emassiz!")
        return

    try:
        args = " ".join(context.args)
        parts = args.split("*")
        test_code = parts[0].strip()
        correct_answers = parts[1].strip().lower()
        end_time_str = parts[2].strip()

        TESTS[test_code] = {
            "answers": correct_answers,
            "end_time": end_time_str,
            "active": True,
            "users": {}
        }

        await update.message.reply_text(f"✅ Test qo'shildi!\nKod: {test_code}\nTugash vaqti: {end_time_str}")
    except Exception as e:
        await update.message.reply_text("Xatolik! Format: /add_test 13028*abcd...*23:00")

async def handle_message(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id

    if not await check_subscription(user_id, context):
        await update.message.reply_text(
            f"⚠️ **Test javobini yuborish uchun avval rasmiy kanalimizga obuna boʻling!**\n\n"
            f"👉 Kanalimiz: {CHANNEL_USERNAME}\n\n"
            "Obuna boʻlgach, javoblaringizni qayta yuboring!"
        )
        return

    text = update.message.text.strip()
    if "*" not in text:
        await update.message.reply_text("Javob yuborish formati noto'g'ri. Masalan: 13028*abcd...")
        return

    parts = text.split("*")
    test_code = parts[0].strip()
    user_answers = parts[1].strip().lower()

    if test_code not in TESTS:
        await update.message.reply_text("Bunday test kodi topilmadi!")
        return

    test = TESTS[test_code]
    correct_answers = test["answers"]

    score = 0
    for u, c in zip(user_answers, correct_answers):
        if u == c:
            score += 1

    total = len(correct_answers)
    await update.message.reply_text(f"👤 Natijangiz:\n\nTo'g'ri javoblar: {score}/{total}")

if __name__ == '__main__':
    app = ApplicationBuilder().token("8884206258:AAH3pT3Vys4-R7_nkKTQXUKRSkqYcDOrVW8").build()

    app.add_handler(CommandHandler("start", start))
    app.add_handler(CommandHandler("add_test", add_test))
    app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, handle_message))

    app.run_polling()
    
