# nevidimka-bot
api/bot.js
// ======== ВСТАВ СВІЙ ТОКЕН НИЖЧЕ ========
const TOKEN = "Here is the token for bot nevidimka @nevidimkaWeb_bot:

7912546142:AAGr8KTgHZ3-cS7JdIldJaMesJHcX8EnkIE";
// =========================================

const API = `https://api.telegram.org/bot${TOKEN}`;

async function sendMessage(chat_id, text, extra = {}) {
  return fetch(`${API}/sendMessage`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      chat_id,
      text,
      ...extra
    })
  });
}

function mainMenu() {
  return {
    reply_markup: {
      inline_keyboard: [
        [
          { text: "🏠 Головна", callback_data: "home" },
          { text: "👤 Профіль", callback_data: "profile" }
        ],
        [
          { text: "❓ Допомога", callback_data: "help" }
        ]
      ]
    }
  };
}

export default async function handler(req, res) {

  if (req.method === "GET") {
    return res.status(200).send("Bot is running");
  }

  const update = req.body;

  if (update.message) {
    const chat_id = update.message.chat.id;
    const text = update.message.text || "";

    if (text === "/start") {
      await sendMessage(chat_id,
        "Привіт! Я бот nevidimka 🤖",
        mainMenu()
      );
    } else {
      await sendMessage(chat_id,
        "Ти написав: " + text,
        mainMenu()
      );
    }
  }

  if (update.callback_query) {
    const chat_id = update.callback_query.message.chat.id;
    const data = update.callback_query.data;

    await fetch(`${API}/answerCallbackQuery`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        callback_query_id: update.callback_query.id
      })
    });

    if (data === "profile") {
      const user = update.callback_query.from;
      await sendMessage(chat_id,
        `👤 Профіль:\nІм'я: ${user.first_name}\nID: ${user.id}`,
        mainMenu()
      );
    }

    if (data === "home") {
      await sendMessage(chat_id,
        "🏠 Головна сторінка",
        mainMenu()
      );
    }

    if (data === "help") {
      await sendMessage(chat_id,
        "Команди:\n/start",
        mainMenu()
      );
    }
  }

  return res.status(200).json({ ok: true });
}
