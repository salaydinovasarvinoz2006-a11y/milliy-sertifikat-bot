import logging
from telegram import Update
from telegram.ext import ApplicationBuilder, CommandHandler, MessageHandler, filters, ContextTypes

logging.basicConfig(
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    level=logging.INFO
)

# Admin (Sizning) Telegram ID raqamingiz (Administrator huquqi uchun)
ADMIN_ID = 0  # Botingizga /id deb yozsangiz, ID raqamingizni chiqaradi

# Testlar bazasi (Xotirada saqlanadi)
TESTS = {}

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id
    user_name = update.effective_user.first_name
    
    msg = (
        f"Assalomu alaykum, {user_name}!\n\n"
        "📝 **Ona tili va adabiyot | Milliy Sertifikat Test Bot**\n\n"
        "Javoblaringizni tekshirish uchun quyidagi formatda yuboring:\n"
        "`TestKodi*javoblar`\n\n"
        "📌 *Misol:* `13028*abcdabcd...`\n\n"
        "💡 *Sizning ID raqamingiz:* `" + str(user_id) + "`"
    )
    await update.message.reply_text(msg, parse_mode="Markdown")

# ADMIN UCHUN: Yangi test qo'shish buyrug'i (/add_test 13028*abcd...)
async def add_test(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id
    
    # Agar admin ID o'rnatilmagan bo'lsa yoki boshqa odam yozsa:
    if ADMIN_ID != 0 and user_id != ADMIN_ID:
        await update.message.reply_text("⛔️ Bu buyruq faqat bot egasi uchun!")
        return

    try:
        args = " ".join(context.args)
        test_code, correct_answers = args.split("*")
        test_code = test_code.strip()
        correct_answers = correct_answers.strip().lower()

        TESTS[test_code] = correct_answers
        await update.message.reply_text(
            f"✅ **Yangi test muvaffaqiyatli saqlandi!**\n\n"
            f"🔑 **Test kodi:** `{test_code}`\n"
            f"📊 **Savollar soni:** {len(correct_answers)} ta",
            parse_mode="Markdown"
        )
    except Exception:
        await update.message.reply_text(
            "❌ Xato format!\n\nTest qo'shish uchun shunday yozing:\n"
            "`/add_test TestKodi*to'g'ri_javoblar`\n\n"
            "Misol: `/add_test 13028*abcdabcdabcd`",
            parse_mode="Markdown"
        )

# O'QUVCHILAR UCHUN: Javoblarni tekshirish
async def handle_answers(update: Update, context: ContextTypes.DEFAULT_TYPE):
    msg = update.message.text.strip().lower()

    if "*" not in msg:
        await update.message.reply_text("⚠️ Javobni `TestKodi*javoblar` shaklida yuboring (Masalan: `13028*abcd...`)", parse_mode="Markdown")
        return

    test_code, user_answers = msg.split("*", 1)
    test_code = test_code.strip()
    user_answers = user_answers.strip()

    if test_code not in TESTS:
        await update.message.reply_text("❌ Bunday test kodi topilmadi yoki test yakunlangan.")
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
        f"✅ **To'g'ri javoblar:** {correct_count} / {total_questions} ta\n"
        f"📈 **Natija:** {percentage}%\n"
        f"🏆 **Daraja:** {grade}"
    )

    await update.message.reply_text(result_text, parse_mode="Markdown")

if __name__ == '__main__':
    TOKEN = "8884206258:AAH3pT3Vys4-R7_nkKTQXUKRSkqYcDOrVW8"

    app = ApplicationBuilder().token(TOKEN).build()

    app.add_handler(CommandHandler('start', start))
    app.add_handler(CommandHandler('add_test', add_test))
    app.add_handler(MessageHandler(filters.TEXT & (~filters.COMMAND), handle_answers))

    app.run_polling()
    
