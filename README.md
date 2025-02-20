from telegram import Update, Bot
from telegram.ext import ApplicationBuilder, ChatJoinRequestHandler, CommandHandler, ContextTypes
import asyncio
import json
import os

# ✅ Bot ki details
BOT_TOKEN = '7938303675:AAFoBrpjtx-Nhm-9ShgRjK8Evrm0ifK94jg'
ADMIN_ID = 5973616762
DATA_FILE = 'users.json'

# ✅ User data save karna
def load_users():
    if os.path.exists(DATA_FILE):
        with open(DATA_FILE, 'r') as f:
            return json.load(f)
    return []

def save_users(users):
    with open(DATA_FILE, 'w') as f:
        json.dump(users, f)

# ✅ Jab user join kare, data save karo aur notify karo
async def handle_join_request(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user = update.chat_join_request.from_user
    username = user.username if user.username else user.first_name
    group_name = update.chat_join_request.chat.title

    # ⏳ 30 seconds ka wait
    await asyncio.sleep(30)
    await update.chat_join_request.approve()

    # 📩 Admin ko notify karo
    admin_msg = f"👤 *New User Joined!*\n🔗 Username: @{username}\n📛 Group: {group_name}"
    await context.bot.send_message(chat_id=ADMIN_ID, text=admin_msg, parse_mode="Markdown")

    # 📨 User ko welcome message bhejo
    try:
        await context.bot.send_message(chat_id=user.id, text=f"🎉 Congratulations @{username}! Aap '{group_name}' group me successfully add ho gaye ho via @Automaticacceptrequest0056bot.")
    except:
        pass  # Ignore agar user ne /start nahi kiya ho

    # ✅ User ko save karo
    users = load_users()
    if user.id not in users:
        users.append(user.id)
        save_users(users)

# 🎯 /start command (user registration ke liye)
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id
    users = load_users()
    if user_id not in users:
        users.append(user_id)
        save_users(users)
    await update.message.reply_text("🤖 Bot activated! Aapko updates milte rahenge.")

# 🎯 Broadcast command (Admin only)
async def broadcast(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if update.effective_user.id != ADMIN_ID:
        await update.message.reply_text("🚫 Aapko ye access nahi hai.")
        return

    msg_text = ' '.join(context.args)
    if not msg_text:
        await update.message.reply_text("⚠️ Usage: /broadcast [message]")
        return

    users = load_users()
    success_count = 0
    for user_id in users:
        try:
            await context.bot.send_message(chat_id=user_id, text=f"📢 *Broadcast Message:*\n\n{msg_text}", parse_mode="Markdown")
            success_count += 1
        except:
            pass
    await update.message.reply_text(f"✅ Broadcast complete! {success_count}/{len(users)} users ne message receive kiya.")

# 🎯 /usercount command (Admin ko total users dikhaye)
async def user_count(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if update.effective_user.id != ADMIN_ID:
        await update.message.reply_text("🚫 Aapko ye access nahi hai.")
        return
    users = load_users()
    await update.message.reply_text(f"👥 Total Users: {len(users)}")

# 🚀 Main function to run the bot
def main():
    app = ApplicationBuilder().token(BOT_TOKEN).build()
    app.add_handler(ChatJoinRequestHandler(handle_join_request))
    app.add_handler(CommandHandler('start', start))
    app.add_handler(CommandHandler('broadcast', broadcast))
    app.add_handler(CommandHandler('usercount', user_count))

    print("🤖 Bot chal raha hai... (Auto-accept, Notifications, Broadcast, User Count ready)")
    app.run_polling()

if __name__ == '__main__':
    main()
