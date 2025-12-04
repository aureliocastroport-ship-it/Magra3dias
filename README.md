from telegram import Update
from telegram.ext import ApplicationBuilder, MessageHandler, CommandHandler, ContextTypes, filters

TOKEN = "8495402163:AAE1v6lkRchYIDZ4xpctN0gpXmDMwc9-QVw"

participants = {}

TIPS = [
    "💧 Beba 2L de água hoje.",
    "🥗 Priorize vegetais no almoço e jantar.",
    "🍎 Evite açúcar e processados."
]

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user = update.message.from_user
    participants[user.id] = {"name": user.first_name, "day1": None, "day3": None}
    await update.message.reply_text("Bem-vindo ao Desafio de 3 dias! Envie seu *peso do Dia 1*.")

async def collect(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.message.from_user.id

    try:
        weight = float(update.message.text.replace(",", "."))
    except:
        await update.message.reply_text("Envie apenas o número do peso.")
        return

    data = participants[user_id]

    if data["day1"] is None:
        data["day1"] = weight
        await update.message.reply_text("Peso do Dia 1 registrado! No Dia 3 envie seu peso final.")
    else:
        data["day3"] = weight
        diff = data["day3"] - data["day1"]
        await update.message.reply_text(
            f"📊 Resultado final:\n\n"
            f"Dia 1: {data['day1']} kg\n"
            f"Dia 3: {data['day3']} kg\n"
            f"Variação: {diff:+.2f} kg"
        )

async def help(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text("Comandos:\n/start\n/help")

def main():
    app = ApplicationBuilder().token(TOKEN).build()

    app.add_handler(CommandHandler("start", start))
    app.add_handler(CommandHandler("help", help))
    app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, collect))

    app.run_polling()

if __name__ == "__main__":
    main()
