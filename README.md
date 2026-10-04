
import os
import google.generativeai as genai
from telegram import Update
from telegram.ext import Application, CommandHandler, MessageHandler, ContextTypes, filters

# API keys will be added through GitHub Secrets
TELEGRAM_TOKEN = os.environ["8688241097:AAEy0QzrViZ83z_8CufEEc9yYrW42DbgRj0"]
GEMINI_API_KEY = os.environ["AQ.Ab8RN6Ks7sxob-wUfLKnjQpFX-YJxVMPEBNW_rCr6DiRWUxt_Q"]

genai.configure(api_key=GEMINI_API_KEY)
model = genai.GenerativeModel("gemini-2.0-flash")


async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text(
        "👋 Welcome to CreatorGrow AI!\n\n"
        "Send me a YouTube, Facebook, Instagram, TikTok, or website link "
        "and I will help you analyze it."
    )


async def analyze(update: Update, context: ContextTypes.DEFAULT_TYPE):
    text = update.message.text.strip()

    if not text.startswith(("http://", "https://")):
        await update.message.reply_text(
            "🔗 Please send a valid link."
        )
        return

    await update.message.reply_text("⏳ Analyzing your link...")

    prompt = f"""
You are CreatorGrow AI.

Analyze this public social-media/web link:
{text}

Give a useful creator-growth report:
1. Content/topic summary
2. Strengths
3. Weaknesses
4. 5 content ideas
5. 3 title ideas
6. 5 relevant hashtags
7. Practical growth suggestions

Be concise and helpful.
"""

    try:
        response = model.generate_content(prompt)
        result = response.text

        await update.message.reply_text(
            "📊 CreatorGrow AI Report\n\n" + result
        )

    except Exception as e:
        await update.message.reply_text(
            "❌ Sorry, something went wrong. Please try again later."
        )


def main():
    app = Application.builder().token(TELEGRAM_TOKEN).build()

    app.add_handler(CommandHandler("start", start))
    app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, analyze))

    print("CreatorGrow AI Bot is running...")
    app.run_polling()


if __name__ == "__main__":
    main()
