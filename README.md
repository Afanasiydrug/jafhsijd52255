from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup, InputFile
from telegram.ext import Application, CommandHandler, CallbackQueryHandler, ContextTypes, MessageHandler, filters
import os
from datetime import datetime

TOKEN = "7573111771:AAGb_p-3rGlkB8pS59_DGr1lbizYS7X6XKo"  
CHANNEL_ID = "-1002292177682"  
CHANNEL_LINK = "https://t.me/+sbXu2o_NRlIxNjBi"
YOUR_USER_ID = 1555314491  # Ваш Telegram ID

# Словари для хранения данных
verified_users = {}  # Хранит информацию о проверенных пользователях
user_states = {}     # Хранит состояния пользователей
user_reports = {}    # Хранит количество репортов

async def log_new_subscription(context: ContextTypes.DEFAULT_TYPE, user_id: int, username: str, first_name: str):
    """Отправляет уведомление о новой подписке с количеством пользователей"""
    try:
        total_users = len(verified_users)  # Получаем общее количество пользователей
        message = (
            "🆕 НОВЫЙ ПОЛЬЗОВАТЕЛЬ ПОДПИСАЛСЯ НА КАНАЛ!\n\n"
            f"👤 Имя: {first_name}\n"
            f"🆔 ID: {user_id}\n"
            f"🔖 Юзернейм: @{username if username else 'нет'}\n"
            f"👥 Всего пользователей: {total_users}\n"
            f"⏰ Время: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}"
        )
        await context.bot.send_message(chat_id=YOUR_USER_ID, text=message)
    except Exception as e:
        print(f"Ошибка отправки лога: {e}")

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id
    
    if user_id in verified_users:
        await send_main_menu(update, context)
    else:
        keyboard = [
            [InlineKeyboardButton("🔗 Подписаться на канал", url=CHANNEL_LINK)],
            [InlineKeyboardButton("✅ Проверить подписку", callback_data="check_sub")]
        ]
        
        await update.message.reply_text(
            "📢 Для доступа к боту необходимо подписаться на наш канал:",
            reply_markup=InlineKeyboardMarkup(keyboard)
        )

async def button_handler(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = update.callback_query
    await query.answer()
    
    if query.data == "check_sub":
        user_id = query.from_user.id
        try:
            member = await context.bot.get_chat_member(CHANNEL_ID, user_id)
            if member.status in ["member", "administrator", "creator"]:
                verified_users[user_id] = datetime.now()
                
                # Отправляем лог о новой подписке
                await log_new_subscription(
                    context,
                    user_id,
                    query.from_user.username,
                    query.from_user.first_name
                )
                
                await query.edit_message_text("✅ Вы подписаны! Доступ разрешён.")
                await send_main_menu(update, context)
            else:
                await query.edit_message_text(
                    "❌ Вы ещё не подписаны!",
                    reply_markup=InlineKeyboardMarkup([
                        [InlineKeyboardButton("🔗 Подписаться", url=CHANNEL_LINK)],
                        [InlineKeyboardButton("🔄 Проверить снова", callback_data="check_sub")]
                    ]))
        except Exception as e:
            await query.edit_message_text(f"⚠ Ошибка: {e}")

async def send_main_menu(update: Update, context: ContextTypes.DEFAULT_TYPE):
    photo_path = "C:\\Users\\timur\Desktop\\AI IMAGE\\durov.jpg"
    
    if not os.path.exists(photo_path):
        keyboard = [
            [InlineKeyboardButton("🌐 Отправить репорты", callback_data="button1")],
            [InlineKeyboardButton("👤 Мой профиль", callback_data="button2")],
            [InlineKeyboardButton("🆘 Помощь", callback_data="button3")]
        ]
        
        await context.bot.send_message(
            chat_id=update.effective_chat.id,
            text="🏠 Добро пожаловать в главное меню!",
            reply_markup=InlineKeyboardMarkup(keyboard))
    else:
        keyboard = [
            [InlineKeyboardButton("🌐 Отправить репорты", callback_data="button1")],
            [InlineKeyboardButton("👤 Мой профиль", callback_data="button2")],
            [InlineKeyboardButton("🆘 Помощь", callback_data="button3")]
        ]
        
        with open(photo_path, 'rb') as photo:
            await context.bot.send_photo(
                chat_id=update.effective_chat.id,
                photo=InputFile(photo),
                caption="🏠 Добро пожаловать в главное меню!",
                reply_markup=InlineKeyboardMarkup(keyboard))

async def handle_main_menu_buttons(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = update.callback_query
    await query.answer()
    
    if query.data == "button1":
        photo_path_button1 = "C:\\Users\\timur\\Desktop\\AI IMAGE\\durov.jpg"
        
        keyboard = [
            [InlineKeyboardButton("✉️ MAIL SNOS", callback_data="mail_snos")],
            [InlineKeyboardButton("📟 SESSION SNOS", callback_data="session_snos")],
            [InlineKeyboardButton("🔙 Назад", callback_data="back_to_main")]
        ]
        
        if os.path.exists(photo_path_button1):
            with open(photo_path_button1, 'rb') as photo:
                await context.bot.send_photo(
                    chat_id=update.effective_chat.id,
                    photo=InputFile(photo),
                    caption="⚡️ Выберите тип репорта",
                    reply_markup=InlineKeyboardMarkup(keyboard))
        else:
            await query.edit_message_text(
                text="⚡️ Выберите тип репорта",
                reply_markup=InlineKeyboardMarkup(keyboard))
    
    elif query.data == "button2":
        user = query.from_user
        user_id = user.id
        first_name = user.first_name or ""
        last_name = user.last_name or ""
        username = user.username or "нет"
        
        report_count = user_reports.get(user_id, 0)
        
        photo_path_button2 = "C:\\Users\\timur\\Desktop\\AI IMAGE\\durov.jpg"
        
        keyboard = [
            [InlineKeyboardButton("🔙 Назад", callback_data="back_to_main")]
        ]
        
        message_text = (
            f"👤 Ваш профиль:\n\n"
            f"🆔 ID: {user_id}\n"
            f"📛 Имя: {first_name} {last_name}\n"
            f"🔖 Юзернейм: @{username}\n"
            f"📊 Отправлено репортов: {report_count}"
        )
        
        if os.path.exists(photo_path_button2):
            with open(photo_path_button2, 'rb') as photo:
                await context.bot.send_photo(
                    chat_id=update.effective_chat.id,
                    photo=InputFile(photo),
                    caption=message_text,
                    reply_markup=InlineKeyboardMarkup(keyboard))
        else:
            await query.edit_message_text(
                text=message_text,
                reply_markup=InlineKeyboardMarkup(keyboard))
    
    elif query.data == "button3":
        photo_path_button3 = "C:\\Users\\timur\\Desktop\\AI IMAGE\\durov.jpg"
        
        keyboard = [
            [InlineKeyboardButton("🔙 Назад", callback_data="back_to_main")]
        ]
        
        if os.path.exists(photo_path_button3):
            with open(photo_path_button3, 'rb') as photo:
                await context.bot.send_photo(
                    chat_id=update.effective_chat.id,
                    photo=InputFile(photo),
                    caption="⚙️ Инструкция по работе с ботом\n\n🆘 Помощь:\n\n1. Выберите метод (MAIL SNOS или SESSION SNOS)\n\n2. Для канала: Укажите ссылку на канал и нарушение через запятую\n\n3. Для аккаунта: Укажите ссылку на сообщение\n\n4. Выберите причину репорта\n\n5. Подтвердите или отмените репорт\n\n💨Скорость сноса зависит ТОЛЬКО от поддержки телеграмма\n\nСпасибо за использование нашего бота!🎉",
                    reply_markup=InlineKeyboardMarkup(keyboard))
        else:
            await query.edit_message_text(
                text="⚙️ Инструкция по работе с ботом\n\n🆘 Помощь:\n\n1. Выберите метод (MAIL SNOS или SESSION SNOS)\n\n2. Для канала: Укажите ссылку на канал и нарушение через запятую\n\n3. Для аккаунта: Укажите ссылку на сообщение\n\n4. Выберите причину репорта\n\n5. Подтвердите или отмените репорт\n\n💨Скорость сноса зависит ТОЛЬКО от поддержки телеграмма\n\nСпасибо за использование нашего бота!🎉",
                reply_markup=InlineKeyboardMarkup(keyboard))
    
    elif query.data == "back_to_main":
        await send_main_menu(update, context)
    elif query.data == "mail_snos":
        photo_path_mail_snos = "C:\\Users\\timur\\Desktop\\AI IMAGE\\durov.jpg"
        
        keyboard = [
            [InlineKeyboardButton("📢 Репорт на канал", callback_data="report_channel")],
            [InlineKeyboardButton("👤 Репорт на аккаунт", callback_data="report_account")],
            [InlineKeyboardButton("🔙 Назад", callback_data="back_to_main")]
        ]
        
        if os.path.exists(photo_path_mail_snos):
            with open(photo_path_mail_snos, 'rb') as photo:
                await context.bot.send_photo(
                    chat_id=update.effective_chat.id,
                    photo=InputFile(photo),
                    caption="📧 MAIL SNOS - Выберите действие",
                    reply_markup=InlineKeyboardMarkup(keyboard))
        else:
            await query.edit_message_text(
                text="📧 MAIL SNOS - Выберите действие",
                reply_markup=InlineKeyboardMarkup(keyboard))
    elif query.data == "session_snos":
        photo_path_session_snos = "C:\\Users\\timur\\Desktop\\AI IMAGE\\durov.jpg"
        
        keyboard = [
            [InlineKeyboardButton("👤 Репорт на аккаунт", callback_data="report_account")],
            [InlineKeyboardButton("🔙 Назад", callback_data="back_to_main")]
        ]
        
        if os.path.exists(photo_path_session_snos):
            with open(photo_path_session_snos, 'rb') as photo:
                await context.bot.send_photo(
                    chat_id=update.effective_chat.id,
                    photo=InputFile(photo),
                    caption="📟 SESSION SNOS - Выберите действие",
                    reply_markup=InlineKeyboardMarkup(keyboard))
        else:
            await query.edit_message_text(
                text="📟 SESSION SNOS - Выберите действие",
                reply_markup=InlineKeyboardMarkup(keyboard))
    elif query.data == "report_channel":
        user_states[query.from_user.id] = "waiting_for_channel_report"
        
        photo_path_report_channel = "C:\\Users\\timur\\Desktop\\AI IMAGE\\durov.jpg"
        
        keyboard = [
            [InlineKeyboardButton("❌ Отменить", callback_data="back_to_main")]
        ]
        
        if os.path.exists(photo_path_report_channel):
            with open(photo_path_report_channel, 'rb') as photo:
                await context.bot.send_photo(
                    chat_id=update.effective_chat.id,
                    photo=InputFile(photo),
                    caption="📢 Вы выбрали репорт на канал\n\n💎 Укажите ссылку на канал и нарушение через запятую\n\nПример: https://t.me/channel, https://t.me/channel/123\n\n✨ Укажите уникальные и публичные ссылки на канал и сообщение!",
                    reply_markup=InlineKeyboardMarkup(keyboard))
        else:
            await query.edit_message_text(
                text="📢 Вы выбрали репорт на канал\n\n💎 Укажите ссылку на канал и нарушение через запятую\n\nПример: https://t.me/channel, https://t.me/channel/123\n\n✨ Укажите уникальные и публичные ссылки на канал и сообщение!",
                reply_markup=InlineKeyboardMarkup(keyboard))
    elif query.data == "report_account":
        user_states[query.from_user.id] = "waiting_for_account_report"
        
        photo_path_report_account = "C:\\Users\\timur\\Desktop\\AI IMAGE\\durov.jpg"
        
        keyboard = [
            [InlineKeyboardButton("❌ Отменить", callback_data="back_to_main")]
        ]
        
        if os.path.exists(photo_path_report_account):
            with open(photo_path_report_account, 'rb') as photo:
                await context.bot.send_photo(
                    chat_id=update.effective_chat.id,
                    photo=InputFile(photo),
                    caption="👤 Вы выбрали репорт на аккаунт\n\n💎 Укажите уникальную ссылку на публичное сообщение для репорта (приватные ссылки не принимаются)\n\nПример: https://t.me/chat/123\n\n🌈 Убедитесь, что ссылка активна, публична и уникальна!",
                    reply_markup=InlineKeyboardMarkup(keyboard))
        else:
            await query.edit_message_text(
                text="👤 Вы выбрали репорт на аккаунт\n\n💎 Укажите уникальную ссылку на публичное сообщение для репорта (приватные ссылки не принимаются)\n\nПример: https://t.me/chat/123\n\n🌈 Убедитесь, что ссылка активна, публична и уникальна!",
                reply_markup=InlineKeyboardMarkup(keyboard))
    elif query.data in ["spam_report", "personal_data_report", "violence_report", "other_report"]:
        user_id = query.from_user.id
        user_reports[user_id] = user_reports.get(user_id, 0) + 1
        
        photo_path_confirmation = "C:\\Users\\timur\\Desktop\\AI IMAGE\\durov.jpg"
        
        keyboard = [
            [InlineKeyboardButton("🔙 В МЕНЮ", callback_data="back_to_main")]
        ]
        
        if os.path.exists(photo_path_confirmation):
            with open(photo_path_confirmation, 'rb') as photo:
                await context.bot.send_photo(
                    chat_id=update.effective_chat.id,
                    photo=InputFile(photo),
                    caption="💎 Жалобы отправлены!\n\n👥 Репорты успешно завершены! Спасибо за использование нашего бота! 🎉",
                    reply_markup=InlineKeyboardMarkup(keyboard))
        else:
            await query.edit_message_text(
                text="💎 Жалобы отправлены!\n\n👥 Репорты успешно завершены! Спасибо за использование нашего бота! 🎉",
                reply_markup=InlineKeyboardMarkup(keyboard))

async def handle_user_message(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.message.from_user.id
    
    if user_id in user_states:
        if user_states[user_id] == "waiting_for_channel_report":
            photo_path_violation = "C:\\Users\\timur\\Desktop\\AI IMAGE\\durov.jpg"
            
            keyboard = [
                [InlineKeyboardButton("🚫 СПАМ", callback_data="spam_report")],
                [InlineKeyboardButton("🔒 Личные данные", callback_data="personal_data_report")],
                [InlineKeyboardButton("⚔️ Насилие", callback_data="violence_report")],
                [InlineKeyboardButton("❓ Другое", callback_data="other_report")],
                [InlineKeyboardButton("❌ Отменить", callback_data="back_to_main")]
            ]
            
            if os.path.exists(photo_path_violation):
                with open(photo_path_violation, 'rb') as photo:
                    await context.bot.send_photo(
                        chat_id=update.effective_chat.id,
                        photo=InputFile(photo),
                        caption=f"📌 Вы отправили: {update.message.text}\n\nВыберите тип нарушения:",
                        reply_markup=InlineKeyboardMarkup(keyboard))
            else:
                await context.bot.send_message(
                    chat_id=update.effective_chat.id,
                    text=f"📌 Вы отправили: {update.message.text}\n\nВыберите тип нарушения:",
                    reply_markup=InlineKeyboardMarkup(keyboard))
            
            del user_states[user_id]
        
        elif user_states[user_id] == "waiting_for_account_report":
            photo_path_account_violation = "C:\\Users\\timur\\Desktop\\AI IMAGE\\durov.jpg"
            
            keyboard = [
                [InlineKeyboardButton("🚫 СПАМ", callback_data="spam_report")],
                [InlineKeyboardButton("🔒 Личные данные", callback_data="personal_data_report")],
                [InlineKeyboardButton("⚔️ Насилие", callback_data="violence_report")],
                [InlineKeyboardButton("❓ Другое", callback_data="other_report")],
                [InlineKeyboardButton("❌ Отменить", callback_data="back_to_main")]
            ]
            
            if os.path.exists(photo_path_account_violation):
                with open(photo_path_account_violation, 'rb') as photo:
                    await context.bot.send_photo(
                        chat_id=update.effective_chat.id,
                        photo=InputFile(photo),
                        caption=f"👤 Вы отправили: {update.message.text}\n\nВыберите тип нарушения для аккаунта:",
                        reply_markup=InlineKeyboardMarkup(keyboard))
            else:
                await context.bot.send_message(
                    chat_id=update.effective_chat.id,
                    text=f"👤 Вы отправили: {update.message.text}\n\nВыберите тип нарушения для аккаунта:",
                    reply_markup=InlineKeyboardMarkup(keyboard))
            
            del user_states[user_id]

def main():
    app = Application.builder().token(TOKEN).build()
    
    app.add_handler(CommandHandler("start", start))
    app.add_handler(CallbackQueryHandler(button_handler, pattern="^check_sub$"))
    app.add_handler(CallbackQueryHandler(handle_main_menu_buttons, pattern="^(button|back_to_main|mail_snos|session_snos|report_channel|report_account|spam_report|personal_data_report|violence_report|other_report)"))
    app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, handle_user_message))
    
    print("Бот запущен! Используйте /start")
    app.run_polling()

if __name__ == "__main__":
    main()
    import time
    while True:
        try:
            main()
        except Exception as e:
            print(f"Ошибка: {e}. Перезапуск через 5 секунд...")
            time.sleep(5)
