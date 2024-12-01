# Самый простой разговорный Telegram бот
## *Это краткий топик посвященный простому созданию разговорного Telegram бота, в котором вы найдете открытый код и краткое опсание всех этапов.*
- Для начала мы создаем бота через **[BotFather](t.me/BotFather)** и сохраняем HTTP API токен
- Устанавливем на компьютер, **Visual Studio Code** и открыв вкладку *"Extensions"* уже в самом приложении скачиваем **Python**
- Следующим шагом мы устанавливем **[pyTelegramBotAPI](pyTelegramBotAPI)** (всё делаем по инструкции, прописываем через консоль или терминал в зависимости от ОС)

## И наконец мы готовы просто закопипастить код и лицезреть бота.
```Python
import logging
from telegram import Update, ReplyKeyboardRemove
from telegram.ext import Application, CommandHandler, MessageHandler, filters, ConversationHandler, ContextTypes

# Настройки логирования
logging.basicConfig(format='%(asctime)s - %(name)s - %(levelname)s - %(message)s', level=logging.INFO)
logger = logging.getLogger(__name__)

# Состояния беседы
START, QUESTION = range(2)

# Список базовых вопросов и ответов
qa_pairs = {
    "Как дела?": "У меня всё хорошо, спасибо!",
    "Что ты можешь?": "Я могу отвечать на ваши вопросы и вести с вами беседу.",
    "Как тебя зовут?": "Я бот, созданный для общения с вами.",
    "Где ты живешь?": "Я живу в облаке, так что я всегда рядом!",
    "Какой твой любимый цвет?": "Я люблю все цвета, но особенно люблю синий.",
    "Ты умеешь шутить?": "Конечно! Почему программисты не могут раздуть шарик? Потому что они не могут найти в нем дырку!",
    "Что ты ешь?": "Я не ем, но если бы мог, я бы выбрал облачные конфеты!",
    "Какой у тебя хобби?": "Мое хобби - помогать людям и общаться с ними.",
    "Что ты думаешь о людях?": "Люди — удивительные существа с огромным потенциалом!",
    "Какой у тебя день рождения?": "У меня нет дня рождения, но я рад, что могу общаться с вами!",
    # Добавьте еще вопросы и ответы...
    "Что такое Python?": "Python — это простой и популярный язык программирования.",
    "Как мы можем помочь окружающей среде?": "Можно уменьшить использование пластика и сортировать мусор!",
    "Как часто нужно заниматься спортом?": "По рекомендации, 150 минут в неделю для поддержания здоровья.",
    "Ты любишь музыку?": "Да, я люблю музыку! Особенно песни о технологиях.",
    "Есть ли у тебя любимый фильм?": "Я не смотрю фильмы, но многие говорят, что 'Матрица' интересный.",
    "Чем ты можешь мне помочь?": "Я могу ответить на ваши вопросы и поддержать беседу.",
    "Где находится центр Земли?": "На глубине около 6,371 километров.",
    "Каков цвет неба?": "Это обычно голубой, но может меняться в зависимости от времени суток.",
    "Сколько времени сейчас?": "Я не могу сказать точное время, но всегда готов помочь!",
    "Почему мы спим?": "Сон помогает нашему мозгу восстанавливаться и улучшает здоровье.",
    "Какой ваш любимый способ путешествовать?": "Я предпочитаю путешествовать через интернет!",
    "Какой самый высокий гора в мире?": "Эверест — самая высокая гора на нашей планете.",
    "Что такое AI?": "AI — это искусственный интеллект, который имитирует человеческие способности.",
    "Кто написал 'Войну и мир'?": "Эту книгу написал Лев Толстой.",
    "Что такое 'интернет вещей'?": "Это концепция, когда устройства подключены к интернету и могут обмениваться данными.",
    "Как вы видите будущее?": "Я оптимистично смотрю на будущее с передовыми технологиями.",
    "Что такое экология?": "Экология изучает отношения между организмами и их окружением.",
    "Каковы преимущества здорового питания?": "Это поддерживает здоровье и обеспечивает энергией.",
    "Что такое программирование?": "Это процесс написания инструкций для компьютера.",
    "Какие языки, кроме Python, вы знаете?": "Я знаком с Java, C++, JavaScript и другими.",
    "Что такое 'блокчейн'?": "Это распределенная база данных для безопасной обработки транзакций.",
    "Каковы преимущества изучения языков?": "Улучшает общение, расширяет горизонты и карьерные возможности.",
    "Идём на ночовку?": "ИДЕМ НА НОЧОВКУ!",
    "Какое сейчас время?": "Сейчас московское время: {}.",
    "Сколько времени?": "Сейчас московское время: {}.",
    "Сколько сейчас времени?": "Сейчас московское время: {}.",
    "Время":"Сейчас {}",
     "Что такое климатические изменения?": "Это долгосрочные изменения в температуре и погоде на планете.",
    "Как можно улучшить память?": "Регулярные упражнения, здоровое питание и достаточный сон могут помочь улучшить память.",
    "Почему необходимо пить воду?": "Вода важна для поддержания жизнедеятельности организма и его нормального функционирования.",
    "Что такое философия?": "Это наука о самых общих принципах бытия и мышления.",
    "Какой язык программирования самый популярный?": "На данный момент одним из самых популярных является Python.",
    "Что такое модель поведения?": "Это система реакций и действий, характерная для конкретного субъекта.",
    "Каковы основные источники энергии?": "Солнечная, ветровая, гидроэлектрическая и ядерная энергия.",
    "Что такое виртуальная реальность?": "Это технологии, создающие имитацию реального мира с помощью компьютеров.",
    "Каковы признаки здорового образа жизни?": "Правильное питание, регулярные физические нагрузки и отсутствие вредных привычек.",
    "Что такое социология?": "Это наука, изучающая общество и социальные процессы.",
    "Кто такие специалисты по данным?": "Это профессионалы, занимающиеся анализом и интерпретацией данных.",
    "Что такое маркетинг?": "Это процесс изучения рынка и продвижения товаров или услуг.",
    "Какой самый распространенный элемент на Земле?": "Это кислород, составляющий примерно 46% земной коры.",
    "Каковы преимущества физической активности?": "Физическая активность укрепляет здоровье и улучшает общее самочувствие.",
    "Что ты думаешь о дружбе?": "Дружба — это важная часть жизни, она поддерживает и вдохновляет.",
    "Какова роль искусства в обществе?": "Искусство обогащает культуру и выражает человечность.",
    "Что такое экономика?": "Экономика изучает производство, распределение и потребление благ.",
    "Как мы можем бороться с изменением климата?": "Необходимо уменьшить загрязнение и переходить на возобновляемые источники энергии.",
    "На чем основывается научный метод?": "Научный метод основывается на наблюдении, гипотезировании и экспериментировании.",
    "Каковы этапы развития жизни на Земле?": "Этапы: простейшие организмы, многоклеточные организмы, животные и растения.",
    "Что такое искусственный интеллект?": "Это область компьютерных наук, посвященная созданию умных машин.",
    "Каковы основные принципы демократии?": "Принципы: свобода слова, право голоса и наличие справедливых выборов.",
    "Что такое виртуальная реальность?": "Это технология, создающая искусственный мир для взаимодействия пользователя.",
    "Каковы преимущества медитации?": "Медитация снижает стресс, улучшает концентрацию и общее психическое состояние.",
    "Что такое человек и общество?": "Это изучение взаимодействия человека с обществом и его культурой.",
    # Добавьте еще вопросы и ответы...
}

# Функция старта
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE) -> int:

    await update.message.reply_text("Привет! Я бот, и я могу отвечать на вопросы. Задайте мне любой вопрос!")
    return QUESTION
# Функция для получения текущего времени в Москве
def get_moscow_time():
    moscow_tz = pytz.timezone('Europe/Moscow')
    moscow_time = datetime.now(moscow_tz)
    return moscow_time.strftime("%H:%M")

# Функция обработки сообщений
async def handle_message(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_message = update.message.text
    for question, answer in qa_pairs.items():
        if question in user_message:
            if "{}" in answer:  # Если в ответе нужно вставить текущее время
                current_time = get_moscow_time()
                answer = answer.format(current_time)
            await update.message.reply_text(answer)
            return
    await update.message.reply_text("Извините, я не знаю ответа на этот вопрос.")


# Обработчик вопросов
async def handle_question(update: Update, context: ContextTypes.DEFAULT_TYPE) -> int:
    question = update.message.text
    response = qa_pairs.get(question, "Извините, я не понимаю этого вопроса. Можете задать что-то другое?")
    await update.message.reply_text(response)
    return QUESTION

# Функция отмены
async def cancel(update: Update, context: ContextTypes.DEFAULT_TYPE) -> int:
    await update.message.reply_text("Хорошо, до свидания!")
    return ConversationHandler.END

# Основная функция
def main() -> None:
    """Запускает бота."""
    application = Application.builder().token("ВАШ АПИ ТОКЕН, ОБЯЗАТЕЛЬНО В КОВЫЧКАХ").build()

    # Обработчик разговорного потока
    conv_handler = ConversationHandler(
        entry_points=[CommandHandler("start", start)],
        states={
            QUESTION: [MessageHandler(filters.TEXT & ~filters.COMMAND, handle_question)],
        },
        fallbacks=[CommandHandler("cancel", cancel)],
    )

    application.add_handler(conv_handler)

    # Запуск бота
    application.run_polling()

if __name__ == "__main__":
    main()
```
### Вот и всё, ваш бот готов. Если вы всё правильно делали, то справа в верхнем углу будет треугольник запуска, тыкаем в него. <span style="color:red"> *Не забудьте указать API вашего бота в 127 строчке!!!*</span>
Дальше его можно захостить, но это уже по желанию, как вы хотите. А так это полность рабочий бот, в котором вы можете сами добавлять вопросы и ответы. Код практически полностью сделан на нейронке и под давлением дедлайна, но а так, всё работает ;)
# Tg [papakarle](t.me/papakarle) Inst [papakarle](https://www.instagram.com/papakarle)
<a href="https://iimg.su/i/LxVLk"><img src="https://s.iimg.su/s/01/PcUpHizMko1aevlEtI5GJMrUIDWLAdRUuCJ5kXDP.jpg"></a>
