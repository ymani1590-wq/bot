git clone https://git.msk0.amvera.ru/pablick/viktor
import logging
from telegram import Update, ReplyKeyboardMarkup, KeyboardButton
from telegram.ext import Application, CommandHandler, MessageHandler, filters, ContextTypes, ConversationHandler

# Настройка логирования
logging.basicConfig(
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    level=logging.INFO
)
logger = logging.getLogger(__name__)

# Состояния разговора
CHOOSING, CHOOSE_STARS, CHOOSE_GIFT_15, CHOOSE_GIFT_25 = range(4)

# Цены и варианты
STARS_15_PRICE = "15 звезд - 30 руб"
STARS_25_PRICE = "25 звезд - 45 руб"
GIFTS_15 = ["💖 Сердечко", "🧸 Мишка"]
GIFTS_25 = ["🌹 Роза", "🎁 Подарок"]

# Ссылки (ЗАМЕНИ НА СВОИ)
GROUP_LINK = "https://t.me/zvezditelegram_bots"  # ЗАМЕНИ НА ССЫЛКУ СВОЕЙ ГРУППЫ
CHANNEL_LINK = "https://t.me/zvezditelegram_bots"  # ЗАМЕНИ НА ССЫЛКУ СВОЕГО КАНАЛА

# Клавиатуры
def get_main_keyboard():
    keyboard = [
        [KeyboardButton("⭐ Купить звезды")],
        [KeyboardButton("💬 Наш чат"), KeyboardButton("📢 Наш канал")],
        [KeyboardButton("⭐ Оставить отзыв")]
    ]
    return ReplyKeyboardMarkup(keyboard, resize_keyboard=True)

def get_stars_keyboard():
    keyboard = [
        [KeyboardButton(STARS_15_PRICE)],
        [KeyboardButton(STARS_25_PRICE)],
        [KeyboardButton("Назад")]
    ]
    return ReplyKeyboardMarkup(keyboard, resize_keyboard=True)

def get_gift_15_keyboard():
    keyboard = [
        [KeyboardButton(GIFTS_15[0]), KeyboardButton(GIFTS_15[1])],
        [KeyboardButton("Назад")]
    ]
    return ReplyKeyboardMarkup(keyboard, resize_keyboard=True)

def get_gift_25_keyboard():
    keyboard = [
        [KeyboardButton(GIFTS_25[0]), KeyboardButton(GIFTS_25[1])],
        [KeyboardButton("Назад")]
    ]
    return ReplyKeyboardMarkup(keyboard, resize_keyboard=True)

# Команда start
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user = update.message.from_user
    welcome_text = f"""
Привет, {user.first_name}! 👋

Добро пожаловать в магазин звезд! ✨

Здесь ты можешь приобрести красивые звезды для своих близких с приятными подарками!

Нажми '⭐ Купить звезды' чтобы начать покупку!

Также присоединяйся к нашему сообществу! 🌟
    """
    await update.message.reply_text(welcome_text, reply_markup=get_main_keyboard())
    return CHOOSING

# Обработка выбора "Купить звезды"
async def buy_stars(update: Update, context: ContextTypes.DEFAULT_TYPE):
    text = """
Отлично! Выбери количество звезд:

⭐ 15 звезд - 30 руб
✨ 25 звезд - 45 руб

Выбери подходящий вариант:
    """
    await update.message.reply_text(text, reply_markup=get_stars_keyboard())
    return CHOOSE_STARS

# Обработка выбора количества звезд
async def choose_stars(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_choice = update.message.text
    
    if user_choice == STARS_15_PRICE:
        context.user_data['stars'] = 15
        context.user_data['price'] = 30
        text = "Отличный выбор! 15 звезд 🎉\n\nТеперь выбери подарок:"
        await update.message.reply_text(text, reply_markup=get_gift_15_keyboard())
        return CHOOSE_GIFT_15
        
    elif user_choice == STARS_25_PRICE:
        context.user_data['stars'] = 25
        context.user_data['price'] = 45
        text = "Великолепно! 25 звезд ✨\n\nТеперь выбери подарок:"
        await update.message.reply_text(text, reply_markup=get_gift_25_keyboard())
        return CHOOSE_GIFT_25
        
    elif user_choice == "Назад":
        await update.message.reply_text("Главное меню:", reply_markup=get_main_keyboard())
        return CHOOSING

# Обработка выбора подарка для 15 звезд
async def choose_gift_15(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_choice = update.message.text
    
    if user_choice in GIFTS_15:
        context.user_data['gift'] = user_choice
        return await process_order(update, context)
    elif user_choice == "Назад":
        await update.message.reply_text("Выбери количество звезд:", reply_markup=get_stars_keyboard())
        return CHOOSE_STARS
    else:
        await update.message.reply_text("Пожалуйста, выбери подарок из предложенных вариантов:")
        return CHOOSE_GIFT_15

# Обработка выбора подарка для 25 звезд
async def choose_gift_25(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_choice = update.message.text
    
    if user_choice in GIFTS_25:
        context.user_data['gift'] = user_choice
        return await process_order(update, context)
    elif user_choice == "Назад":
        await update.message.reply_text("Выбери количество звезд:", reply_markup=get_stars_keyboard())
        return CHOOSE_STARS
    else:
        await update.message.reply_text("Пожалуйста, выбери подарок из предложенных вариантов:")
        return CHOOSE_GIFT_25

# Обработка заказа
async def process_order(update: Update, context: ContextTypes.DEFAULT_TYPE):
    stars = context.user_data.get('stars')
    price = context.user_data.get('price')
    gift = context.user_data.get('gift')
    
    order_text = f"""
🎉 Твой заказ:

⭐ Количество звезд: {stars}
🎁 Выбранный подарок: {gift}
💰 Сумма к оплате: {price} руб

Для завершения покупки свяжись с администратором: @ryzen5600

Или отправь платеж на карту:
2200 7017 0000 1579

Не забудь указать в комментарии:
"Звезды {stars} + {gift}"

📝 После оплаты пришли скриншот чека администратору для подтверждения заказа.

🌟 Если вам понравилось обслуживание, можете оставить отзыв в нашем чате!
    """
    
    await update.message.reply_text(order_text, reply_markup=get_main_keyboard())
    
    # Очищаем данные пользователя
    context.user_data.clear()
    
    return CHOOSING

# Обработка кнопки "Наш чат"
async def show_group(update: Update, context: ContextTypes.DEFAULT_TYPE):
    group_text = f"""
💬 Наш дружный чат

Присоединяйся к нашему сообществу! Здесь ты можешь:
• Общаться с другими покупателями
• Задавать вопросы
• Делиться впечатлениями
• Оставлять отзывы о покупках

Присоединиться: {https://t.me/zvezditelegram_bots}
    """
    await update.message.reply_text(group_text, reply_markup=get_main_keyboard())
    return CHOOSING

# Обработка кнопки "Наш канал"
async def show_channel(update: Update, context: ContextTypes.DEFAULT_TYPE):
    channel_text = f"""
📢 Наш канал

Подписывайся на наш канал чтобы быть в курсе:
• Новых акций и скидок
• Специальных предложений
• Обновлений ассортимента
• Полезной информации

Подписаться: {https://t.me/zvezditelegram_bots}
    """
    await update.message.reply_text(channel_text, reply_markup=get_main_keyboard())
    return CHOOSING

# Обработка кнопки "Оставить отзыв"
async def leave_review(update: Update, context: ContextTypes.DEFAULT_TYPE):
    review_text = f"""
⭐ Оставить отзыв

Спасибо, что хотите поделиться впечатлениями! 😊

Вы можете оставить отзыв в нашем чате:
{https://t.me/zvezditelegram_bots}

Или написать напрямую администратору: @ryzen5600

Ваши отзывы помогают нам становиться лучше! 🌟
    """
    await update.message.reply_text(review_text, reply_markup=get_main_keyboard())
    return CHOOSING

# Обработка неизвестных сообщений
async def handle_unknown(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text(
        "Пожалуйста, используй кнопки для навигации 🎯",
        reply_markup=get_main_keyboard()
    )
    return CHOOSING

# Основная функция
def main():
    # Твой токен
    TOKEN = "8575492289:AAFpaX_3HGOLYjPDPNMkZQ6vqPJxvDS7C84"
    
    application = Application.builder().token(TOKEN).build()
    
    conv_handler = ConversationHandler(
        entry_points=[CommandHandler('start', start)],
        states={
            CHOOSING: [
                MessageHandler(filters.Text("⭐ Купить звезды"), buy_stars),
                MessageHandler(filters.Text("💬 Наш чат"), show_group),
                MessageHandler(filters.Text("📢 Наш канал"), show_channel),
                MessageHandler(filters.Text("⭐ Оставить отзыв"), leave_review)
            ],
            CHOOSE_STARS: [
                MessageHandler(filters.Text([STARS_15_PRICE, STARS_25_PRICE, "Назад"]), choose_stars)
            ],
            CHOOSE_GIFT_15: [
                MessageHandler(filters.Text(GIFTS_15 + ["Назад"]), choose_gift_15)
            ],
            CHOOSE_GIFT_25: [
                MessageHandler(filters.Text(GIFTS_25 + ["Назад"]), choose_gift_25)
            ],
        },
        fallbacks=[MessageHandler(filters.TEXT & ~filters.COMMAND, handle_unknown)],
    )
    
    application.add_handler(conv_handler)
    
    # Запуск бота
    print("Бот запущен...")
    application.run_polling()

if __name__ == '__main__':
    main()
