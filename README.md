import logging
from datetime import datetime
from telegram import Update
from telegram.ext import ApplicationBuilder, CommandHandler, MessageHandler, filters, ContextTypes

logging.basicConfig(
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    level=logging.INFO
)

# Admin ID raqami (boshida 0 turibdi, /id deb yozib o'zgartirishingiz mumkin)
ADMIN_ID = 8880664748

# Testlar bazasi: { test_kodi: {"answers": "abcd...", "end_time": "23:00", "active": True, "users": {}} }
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

# 1. ADMIN UCHUN: Vaqtli test qo'shish (/add_test 13028*abcd...*23:00)
async def add_test(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id
    
    if ADMIN_ID != 0 and user_id != ADMIN_ID:
        await update.message.reply_text("⛔️ Bu buyruq faqat bot egasi uchun!")
        return

    try:
        args = " ".join(context.args)
        # Format: /add_test 13028*abcdabcd*23:00
        parts = args.split("*")
        test_code = parts[0].strip()
        correct_answers = parts[1].strip().lower()
        end_time_str = parts[2].strip() if len(parts) > 2 else "23:59"

        TESTS[test_code] = {
            "answers": correct_answers,
            "end_time": end_time_str,
            "active": True,
            "users": {} # Javob bergan o'quvchilar ro'yxati
        }
        
        await update.message.reply_text(
            f"✅ **Yangi test muvaffaqiyatli saqlandi!**\n\n"
            f"🔑 **Test kodi:** `{test_code}`\n"
            f"📊 **Savollar soni:** {len(correct_answers)} ta\n"
            f"⏰ **Tugash vaqti:** {end_time_str}",
            parse_mode="Markdown"
        )
    except Exception:
        await update.message.reply_text(
            "❌ Xato format!\n\nTest qo'shish uchun shunday yozing:\n"
            "`/add_test TestKodi*javoblar*TugashVaqti`\n\n"
            "Misol: `/add_test 13028*abcdabcdabcd*23:00`",
            parse_mode="Markdown"
        )

# 2. ADMIN UCHUN: Testni vaqtidan oldin yoki vaqtida to'xtatish (/stop_test 13028)
async def stop_test(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id
    if ADMIN_ID != 0 and user_id != ADMIN_ID:
        return

    try:
        test_code = context.args[0].strip()
        if test_code in TESTS:
            TESTS[test_code]["active"] = False
            
            # Statistika hisoblash
            results = TESTS[test_code]["users"]
            total_students = len(results)
            
            grades = {"A+": 0, "A": 0, "B+": 0, "B": 0, "C+": 0, "C": 0, "Sertifikat ololmadi": 0}
            for res in results.values():
                grades[res['grade']] += 1

            stat_text = (
                f"🔴 **Test yakunlandi!**\n\n"
                f"📌 **Test kodi:** {test_code}\n"
                f"👥 **Jami qatnashchilar:** {total_students} ta\n\n"
                f"📊 **Statistika:**\n"
                f"🥇 A+: {grades['A+']} ta\n"
                f"🥈 A: {grades['A']} ta\n"
                f"🥉 B+: {grades['B+']} ta\n"
                f"🔹 B: {grades['B']} ta\n"
                f"🔹 C+: {grades['C+']} ta\n"
                f"🔹 C: {grades['C']} ta\n"
                f"❌ Sertifikat ololmaganlar: {grades['Sertifikat ololmadi']} ta"
            )
            await update.message.reply_text(stat_text, parse_mode="Markdown")
        else:
            await update.message.reply_text("Bunday test kodi topilmadi.")
    except Exception:
        await update.message.reply_text("Misol: `/stop_test 13028`", parse_mode="Markdown")

# 3. O'QUVCHILAR UCHUN: Javoblarni tekshirish
async def handle_answers(update: Update, context: ContextTypes.DEFAULT_TYPE):
    msg = update.message.text.strip().lower()

    if "*" not in msg:
        await update.message.reply_text("⚠️ Javobni `TestKodi*javoblar` shaklida yuboring (Masalan: `13028*abcd...`)", parse_mode="Markdown")
        return

    test_code, user_answers = msg.split("*", 1)
    test_code = test_code.strip()
    user_answers = user_answers.strip()

    if test_code not in TESTS:
        await update.message.reply_text("❌ Bunday test kodi topilmadi.")
        return

    test_data = TESTS[test_code]

    # Vaqt va holatni tekshirish
    now_str = datetime.now().strftime("%H:%M")
    if not test_data["active"] or now_str > test_data["end_time"]:
        test_data["active"] = False
        await update.message.reply_text("🔴 **Afsuski, ushbu test uchun vaqt tugagan va qabul yopilgan!**", parse_mode="Markdown")
        return

    # Bitta o'quvchi qayta topshirishini oldini olish
    user_id = update.effective_user.id
    if user_id in test_data["users"]:
        await update.message.reply_text("⚠️ Siz ushbu testga allaqachon javob topshirgansiz! Qayta topshirish taqiqlanadi.")
        return

    correct_answers = test_data["answers"]
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
        grade = "Sertifikat ololmadi"

    user_name = update.effective_user.full_name
    
    # Natijani xotiraga saqlash
    test_data["users"][user_id] = {"name": user_name, "score": correct_count, "grade": grade}

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
    app.add_handler(CommandHandler('stop_test', stop_test))
    app.add_handler(MessageHandler(filters.TEXT & (~filters.COMMAND), handle_answers))

    app.run_polling()
    
