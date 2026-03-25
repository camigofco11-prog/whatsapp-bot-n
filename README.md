# whatsapp-bot-n
Nano, diversión, entretenimiento 
{
  "name": "whatsapp-bot",
  "version": "1.0.0",
  "description": "Bot de WhatsApp llamado Nano con comandos interactivos",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js"
  },
  "keywords": ["whatsapp", "bot", "nano"],
  "author": "camigofco11-prog",
  "license": "MIT",
  "dependencies": {
    "whatsapp-web.js": "^1.25.0",
    "qrcode-terminal": "^0.12.0",
    "dotenv": "^16.0.3",
    "axios": "^1.6.0"
  },
  "devDependencies": {
    "nodemon": "^3.0.1"
  }
}
const { Client, LocalAuth, MessageMedia } = require('whatsapp-web.js');
const qrcode = require('qrcode-terminal');
const dotenv = require('dotenv');
const fs = require('fs');
const path = require('path');

dotenv.config();

const client = new Client({
  authStrategy: new LocalAuth(),
});

// Almacenar datos de usuarios
const usersFile = path.join(__dirname, 'data', 'users.json');
let users = {};

// Cargar datos de usuarios
function loadUsers() {
  try {
    if (fs.existsSync(usersFile)) {
      users = JSON.parse(fs.readFileSync(usersFile, 'utf8'));
    }
  } catch (err) {
    console.error('Error cargando usuarios:', err);
    users = {};
  }
}

// Guardar datos de usuarios
function saveUsers() {
  try {
    if (!fs.existsSync(path.join(__dirname, 'data'))) {
      fs.mkdirSync(path.join(__dirname, 'data'), { recursive: true });
    }
    fs.writeFileSync(usersFile, JSON.stringify(users, null, 2));
  } catch (err) {
    console.error('Error guardando usuarios:', err);
  }
}

// Obtener o crear usuario
function getUser(userId) {
  if (!users[userId]) {
    users[userId] = {
      id: userId,
      points: 0,
      level: 1,
      gamesPlayed: 0,
      wins: 0
    };
    saveUsers();
  }
  return users[userId];
}

// Comandos disponibles
const commands = {
  help: async (msg, args) => {
    const helpText = `
╔═══════════════════════════════════════╗
║         🤖 COMANDOS DE NANO 🤖        ║
╚═══════════════════════════════════════╝

📋 *INFORMACIÓN*
  !help - Muestra este menú de ayuda
  !info - Información del bot Nano
  !puntos - Ve tus puntos actuales

👋 *INTERACCIÓN*
  !hola - Saludo personalizado
  !beso - Envía GIF de beso anime (etiqueta aleatoria)
  !kaisen - Bienvenida al tema Kaisen

🎮 *JUEGOS Y APUESTAS*
  !juego <tipo> - Juega (dados, moneda, cartas)
  !apuesta <cantidad> - Haz una apuesta
  !ranking - Ver ranking de jugadores

💡 *EJEMPLO*
  !juego dados
  !apuesta 10
    `;
    await msg.reply(helpText);
  },

  info: async (msg, args) => {
    const infoText = `
╔═══════════════════════════════════════╗
║         ℹ️ INFORMACIÓN DE NANO ℹ️      ║
╚═══════════════════════════════════════╝

🤖 *Nombre:* Nano
📱 *Plataforma:* WhatsApp
⚙️ *Estado:* Activo
📊 *Versión:* 1.0.0

*Funcionalidades:*
✅ Comandos interactivos
✅ Sistema de puntos
✅ Juegos y apuestas
✅ Respuestas automáticas
✅ Tema Kaisen exclusivo

👨‍💻 *Desarrollador:* camigofco11-prog
    `;
    await msg.reply(infoText);
  },

  hola: async (msg, args) => {
    const greetings = [
      '¡Hola! 👋 ¿Qué tal? Yo soy Nano 🤖',
      '¡Saludos! 😊 Encantado de verte',
      '¡Hey! 🎉 ¿Cómo estás hoy?',
      '¡Ey! ¿Qué onda? Aquí Nano a tu servicio 🚀'
    ];
    const randomGreeting = greetings[Math.floor(Math.random() * greetings.length)];
    await msg.reply(randomGreeting);
  },

  kaisen: async (msg, args) => {
    const kaisenText = `
╔═══════════════════════════════════════╗
║      🔥 BIENVENIDO AL TEMA KAISEN 🔥 ║
╚═══════════════════════════════════════╝

¡Bienvenido a la comunidad de Jujutsu Kaisen! 🌟

*Información del tema:*
📺 Jujutsu Kaisen es un manga y anime increíble
🎬 Sigue a Yuji Itadori en su aventura
⚡ Lleno de acción, misterio y emoción
👹 Enfrenta cursed spirits y descubre secretos

*Comandos del tema:*
!kaisen personajes - Ver personajes
!kaisen curiosidades - Datos interesantes
!kaisen ep - Episodios recomendados

¡Que disfrutes! 🎌
    `;
    await msg.reply(kaisenText);
  },

  beso: async (msg, args) => {
    const chat = await msg.getChat();
    const participants = chat.participants || [];

    if (participants.length === 0) {
      await msg.reply('❌ Este comando solo funciona en grupos');
      return;
    }

    // Seleccionar miembro al azar
    const randomMember = participants[Math.floor(Math.random() * participants.length)];
    const mention = `@${randomMember.id.user}`;

    const kissText = `
💋 *¡Beso Anime para ti!* 💋

${mention}

Aquí va este adorable beso de anime~ ✨😘
    `;

    await msg.reply(kissText);
    
    // GIF de beso anime (enlace de ejemplo)
    try {
      const media = await MessageMedia.fromUrl(
        'https://media.tenor.com/images/6f74eb1b615b9e1a53147a0a3c62a626/tenor.gif'
      );
      await msg.reply(media);
    } catch (err) {
      console.error('Error enviando GIF:', err);
    }
  },

  puntos: async (msg, args) => {
    const userId = msg.from;
    const user = getUser(userId);
    
    const pointsText = `
╔═══════════════════════════════════════╗
║         💰 TUS PUNTOS 💰              ║
╚═══════════════════════════════════════╝

⭐ *Puntos:* ${user.points}
🏆 *Nivel:* ${user.level}
🎮 *Juegos Jugados:* ${user.gamesPlayed}
✅ *Victorias:* ${user.wins}
📊 *Porcentaje:* ${user.gamesPlayed > 0 ? ((user.wins / user.gamesPlayed) * 100).toFixed(2) : 0}%
    `;
    
    await msg.reply(pointsText);
  },

  juego: async (msg, args) => {
    if (args.length === 0) {
      await msg.reply('❌ Uso: !juego <dados|moneda|cartas>');
      return;
    }

    const userId = msg.from;
    const user = getUser(userId);
    const gameType = args[0].toLowerCase();

    let result = '';
    let won = false;

    if (gameType === 'dados') {
      const dice1 = Math.floor(Math.random() * 6) + 1;
      const dice2 = Math.floor(Math.random() * 6) + 1;
      const total = dice1 + dice2;
      won = total >= 10;
      result = `
🎲 *DADOS* 🎲
Resultado: ${dice1} + ${dice2} = ${total}
${won ? '✅ ¡GANASTE! +50 puntos' : '❌ Perdiste -10 puntos'}
      `;
      user.points += won ? 50 : -10;
    } else if (gameType === 'moneda') {
      const flip = Math.random() < 0.5 ? 'CARA' : 'CRUZ';
      won = Math.random() < 0.5;
      result = `
🪙 *MONEDA* 🪙
Resultado: ${flip}
${won ? '✅ ¡GANASTE! +30 puntos' : '❌ Perdiste -10 puntos'}
      `;
      user.points += won ? 30 : -10;
    } else if (gameType === 'cartas') {
      const card = Math.floor(Math.random() * 13) + 1;
      won = card >= 10;
      result = `
🃏 *CARTAS* 🃏
Tu carta: ${card}
${won ? '✅ ¡GANASTE! +40 puntos' : '❌ Perdiste -10 puntos'}
      `;
      user.points += won ? 40 : -10;
    } else {
      await msg.reply('❌ Tipo de juego no válido. Usa: dados, moneda o cartas');
      return;
    }

    user.gamesPlayed++;
    if (won) user.wins++;
    user.points = Math.max(0, user.points); // No permitir puntos negativos
    
    saveUsers();
    await msg.reply(result);
  },

  apuesta: async (msg, args) => {
    if (args.length === 0) {
      await msg.reply('❌ Uso: !apuesta <cantidad>');
      return;
    }

    const userId = msg.from;
    const user = getUser(userId);
    const apuestaAmount = parseInt(args[0]);

    if (isNaN(apuestaAmount) || apuestaAmount <= 0) {
      await msg.reply('❌ Debes apostar un número válido de puntos');
      return;
    }

    if (user.points < apuestaAmount) {
      await msg.reply(`❌ No tienes suficientes puntos. Tienes: ${user.points}`);
      return;
    }

    const won = Math.random() < 0.5;
    const resultado = won ? apuestaAmount * 2 : -apuestaAmount;
    user.points += resultado;

    saveUsers();

    const apuestaText = `
🎰 *APUESTA REALIZADA* 🎰
Apostaste: ${apuestaAmount} puntos
${won ? `✅ ¡GANASTE! +${apuestaAmount} puntos` : `❌ Perdiste ${apuestaAmount} puntos`}
💰 *Puntos actuales:* ${user.points}
    `;

    await msg.reply(apuestaText);
  },

  ranking: async (msg, args) => {
    const sortedUsers = Object.values(users)
      .sort((a, b) => b.points - a.points)
      .slice(0, 10);

    let rankingText = `
╔═══════════════════════════════════════╗
║         🏆 TOP 10 RANKING 🏆          ║
╚═══════════════════════════════════════╝
`;

    sortedUsers.forEach((user, index) => {
      rankingText += `\n${index + 1}. ${user.id} - ${user.points} puntos`;
    });

    await msg.reply(rankingText);
  }
};

// Respuestas automáticas
const autoResponses = {
  'buenos dias': '¡Buenos días! 🌅 Espero que tengas un excelente día',
  'buenas noches': '¡Buenas noches! 🌙 Que descanses bien',
  'hola nano': '¡Hola! 👋 ¿En qué puedo ayudarte?',
  'gracias nano': '¡De nada! 😊 Siempre es un placer ayudarte',
  'kaisen': '¿Interesado en Kaisen? Usa !kaisen para más info 🔥'
};

client.on('qr', (qr) => {
  qrcode.generate(qr, { small: true });
  console.log('Escanea el código QR con WhatsApp');
});

client.on('ready', () => {
  console.log('✅ Bot Nano está listo!');
  loadUsers();
});

client.on('message', async (msg) => {
  try {
    // Ignorar mensajes del bot
    if (msg.from === 'status@broadcast') return;

    const content = msg.body.toLowerCase().trim();

    // Procesar comandos
    if (content.startsWith('!')) {
      const args = content.slice(1).split(' ');
      const command = args[0];
      args.shift();

      if (commands[command]) {
        await commands[command](msg, args);
      } else {
        await msg.reply('❌ Comando no encontrado. Usa !help para ver los comandos disponibles');
      }
    }

    // Respuestas automáticas
    for (const [key, response] of Object.entries(autoResponses)) {
      if (content.includes(key)) {
        await msg.reply(response);
        break;
      }
    }

  } catch (error) {
    console.error('Error procesando mensaje:', error);
  }
});

client.initialize();
# Variables de entorno para Nano Bot
NODE_ENV=production
BOT_NAME=Nano
BOT_PREFIX=!
# 🤖 Nano Bot - WhatsApp Bot

Bot interactivo para WhatsApp con sistema de comandos, juegos, apuestas y puntos.

## 🚀 Características

- ✅ Sistema de comandos con prefijo `!`
- ✅ Juegos interactivos (dados, moneda, cartas)
- ✅ Sistema de apuestas con puntos
- ✅ Tema exclusivo Kaisen
- ✅ Respuestas automáticas
- ✅ Ranking de jugadores
- ✅ Sistema de puntos y niveles

## 📋 Comandos

### Información
- `!help` - Muestra todos los comandos
- `!info` - Información del bot
- `!puntos` - Ve tus puntos actuales

### Interacción
- `!hola` - Saludo personalizado
- `!beso` - Envía GIF de beso anime (solo en grupos)
- `!kaisen` - Bienvenida al tema Kaisen

### Juegos
- `!juego dados` - Juega a los dados
- `!juego moneda` - Juega a la moneda
- `!juego cartas` - Juega con cartas
- `!apuesta <cantidad>` - Haz una apuesta
- `!ranking` - Ver top 10 jugadores

## ⚙️ Instalación

```bash
# 1. Clonar repositorio
git clone https://github.com/camigofco11-prog/whatsapp-bot.git
cd whatsapp-bot

# 2. Instalar dependencias
npm install

# 3. Configurar variables de entorno
cp .env.example .env

# 4. Iniciar el bot
npm start
```

## 🔐 Primera vez

1. Ejecuta `npm start`
2. Escanea el código QR con WhatsApp
3. ¡El bot estará listo!

## 📊 Sistema de Puntos

- **Juego de Dados**: +50 (gana) / -10 (pierde)
- **Juego de Moneda**: +30 (gana) / -10 (pierde)
- **Juego de Cartas**: +40 (gana) / -10 (pierde)
- **Apuesta**: Ganas el doble o pierdes tu apuesta

## 🎮 Respuestas Automáticas

El bot responde automáticamente a:
- "buenos dias" → ¡Buenos días! 🌅
- "buenas noches" → ¡Buenas noches! 🌙
- "gracias nano" → ¡De nada! 😊
- Y más...

## 📝 Estructura del Proyecto

```
whatsapp-bot/
├── index.js              # Archivo principal
├── package.json          # Dependencias
├── .env.example          # Variables de entorno
├── README.md             # Este archivo
└── data/
    └── users.json        # Datos de usuarios (auto-generado)
```

## 🛠️ Desarrollo

Para modo desarrollo con nodemon:
```bash
npm run dev
```

## 📄 Licencia

MIT

## 👨‍💻 Autor

**camigofco11-prog**

---

¡Disfruta jugando con Nano! 🎮✨
# Dependencias
node_modules/
npm-debug.log
yarn-error.log

# Variables de entorno
.env
.env.local

# Datos de usuarios
data/
.wwebjs_auth/

# Editor
.vscode/
.idea/
*.swp
*.swo
*~

# Sistema
.DS_Store
Thumbs.db
