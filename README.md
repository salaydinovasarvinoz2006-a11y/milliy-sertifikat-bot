import logging
from telegram import Update
from telegram.ext import ApplicationBuilder, CommandHandler, MessageHandler, filters, ContextTypes

logging.basicConfig(
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    level=logging.INFO
)

TESTS = {
    "13028": "abcdabcdabcdabcdabcdabcdabcdabcdabcdabcdabcdabcd"
}

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_name = update.effective_user.first_name
    text = (
        f"Assalomu alaykum, {user_name}!\n\n"
        "Javoblaringizni yuborish uchun quyidagi formatda yozing:\n"
        "`Test_kodi*javoblar`\n\n"
        "Misol uchun: `13028*abcdaba...`"
    )
    await update.message.reply_text(text, parse_mode="Markdown")

async def handle_answers(update: Update, context: ContextTypes.DEFAULT_TYPE):
    msg = update.message.text.strip().lower()
    
    if "*" not in msg:
        await update.message.reply_text("Xato format! Javobni `TestKodi*javoblar` shaklida yuboring (Masalan: `13028*abcd...`)")
        return

    test_code, user_answers = msg.split("*", 1)
    test_code = test_code.strip()
    user_answers = user_answers.strip()

    if test_code not in TESTS:
        await update.message.reply_text("Bunday test kodi topilmadi yoki test yakunlangan.")
        return

    correct_answers = TESTS[test_code]
    total_questions = len(correct_answers)
    
    correct_count = 0
    for i in range(min(len(user_answers), total_questions)):
        if user_answers[i] == correct_answers[i]:
            correct_count += 1

    percentage = round((correct_count / total_questions) * 100, 1)
    
    if percentage >= 86:
        grade = "A+"
    elif percentage >= 80:
        grade = "A"
    elif percentage >= 70:
        grade = "B+"
    elif percentage >= 60:
        grade = "B"
    elif percentage >= 55:
        grade = "C+"
    elif percentage >= 50:
        grade = "C"
    else:
        grade = "Sertifikat ololmadingiz"

    user_name = update.effective_user.full_name
    
    result_text = (
        f"📊 **TEST NATIJANGIZ**\n\n"
        f"👤 **Ism:** {user_name}\n"
        f"📌 **Test kodi:** {test_code}\n"
        f"✅ **Tog'ri javoblar:** {correct_count} / {total_questions} ta\n"
        f"📈 **Natija:** {percentage}%\n"
        f"🏆 **Daraja:** {grade}"
    )

    await update.message.reply_text(result_text, parse_mode="Markdown")

if __name__ == '__main__':
    TOKEN = "8884206258:AAH3pT3Vys4-R7_nkKTQXUKRSkqYcDOrVW8"
    
    app = ApplicationBuilder().token(TOKEN).build()
    
    app.add_handler(CommandHandler('start', start))
    app.add_handler(MessageHandler(filters.TEXT & (~filters.COMMAND), handle_answers))
    
    app.run_polling()
    
