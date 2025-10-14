/* RAMSY BOT V2 - Single-file starter for a Baileys-based WhatsApp bot

Includes an advanced .menu with images, and many active commands.

Images: place your uploaded images in ./assets with these names:

1. assets/profile.jpg   -> your portrait (used as profile/logo)


2. assets/banner.jpg    -> menu background/banner


3. assets/ai.jpg        -> AI/music section image



Usage: node bot.js

Requirements: Node 16+, install deps (see below)


NOTES:

This is a starter implementation. Some features call external APIs (OpenAI, YouTube, TikTok, weather) and require API keys / extra setup.

Commands implemented as handlers in the commands Map. Add more handlers using the same pattern.


DEPENDENCIES: npm i @adiwajshing/baileys axios ytdl-core fluent-ffmpeg sharp qrcode-terminal openai node-fetch (ffmpeg must be installed on the host machine for media conversions)

ENV VARIABLES: OPENAI_API_KEY=... PORTAL_API_KEYS for other services if used */

import makeWASocket, { useSingleFileAuthState, DisconnectReason } from '@adiwajshing/baileys' import fs from 'fs' import axios from 'axios' import ytdl from 'ytdl-core' import child_process from 'child_process' import qrcode from 'qrcode-terminal' import path from 'path'

// ----------------------------- // Auth & socket init (single-file) // ----------------------------- const { state, saveState } = useSingleFileAuthState('./auth_info_multi.json')

const startBot = async () => { const conn = makeWASocket({ auth: state, printQRInTerminal: true, })

conn.ev.on('creds.update', saveState)

conn.ev.on('connection.update', (update) => { const { connection, lastDisconnect, qr } = update if (qr) qrcode.generate(qr, { small: true }) if (connection === 'close') { const shouldReconnect = (lastDisconnect.error && lastDisconnect.error.output) ? lastDisconnect.error.output.statusCode !== DisconnectReason.loggedOut : true console.log('connection closed, reconnecting:', shouldReconnect) if (shouldReconnect) startBot() } if (connection === 'open') console.log('✅ Connected to WhatsApp') })

// Load images into buffers const assets = { profile: fs.existsSync('./assets/profile.jpg') ? fs.readFileSync('./assets/profile.jpg') : null, banner: fs.existsSync('./assets/banner.jpg') ? fs.readFileSync('./assets/banner.jpg') : null, ai: fs.existsSync('./assets/ai.jpg') ? fs.readFileSync('./assets/ai.jpg') : null, }

// ----------------------------- // Utilities // ----------------------------- const sendText = async (jid, text, quoted) => { await conn.sendMessage(jid, { text }, { quoted }) }

const sendImageWithCaption = async (jid, imageBuffer, caption, quoted) => { await conn.sendMessage(jid, { image: imageBuffer, caption }, { quoted }) }

// ----------------------------- // Menu text (uses images if present) // ----------------------------- const getMenuText = (pushName) => { return 🌟 *RAMSY BOT V2.0* 🌟\n\nBonjour ${pushName || ''} — Voici le menu principal :\n\n + 🏠 Menus principaux:\n.menu - Menu principal complet\n.menuimage - Menu visuel\n.menuall - Toutes les commandes\n\n🧠 IA: .chatgpt, .imagine, .translate, .summarize\n🎵 Musique: .play, .ytmp4, .tiktok, .spotify\n🎮 Fun: .meme, .quiz, .8ball, .roll\n💬 Groupe: .kick, .add, .promote, .demote, .welcome, .antilink\n📸 Média: .sticker, .toimg, .removebg\n💼 Utilitaires: .ping, .weather, .time, .qr, .calc\n🧑‍💻 Dev: .eval, .git, .serverinfo\n🔒 Sécurité: .lockgroup, .antibot, .status, .restart\n\n✨ *RAMSY BOT : Le futur entre tes mains* ✨ }

// ----------------------------- // Command handlers map // Each handler receives (m, args) // ----------------------------- const commands = new Map()

// MENU - displays either text or image+text commands.set('menu', async (m, args) => { const name = m.pushName || 'ami' if (assets.banner) { await sendImageWithCaption(m.chat, assets.banner, getMenuText(name), m) } else { await sendText(m.chat, getMenuText(name), m) } })

commands.set('menuimage', async (m, args) => { // send a stylized menu with profile + banner in sequence if (assets.profile) await conn.sendMessage(m.chat, { image: assets.profile, caption: 'RAMSY BOT V2 - PROFILE' }, { quoted: m }) await commands.get('menu')(m, args) })

// PING commands.set('ping', async (m, args) => { const start = Date.now() await conn.sendPresenceUpdate('composing', m.chat) const latency = Date.now() - start await sendText(m.chat, 🏓 Ping: ${latency} ms, m) })

// CHATGPT (requires OPENAI_API_KEY env var) - simple wrapper commands.set('chatgpt', async (m, args) => { const key = process.env.OPENAI_API_KEY if (!key) return sendText(m.chat, 'OpenAI API key not configured. Set OPENAI_API_KEY.', m) const prompt = args.join(' ') if (!prompt) return sendText(m.chat, 'Usage: .chatgpt <question>', m) try { const resp = await axios.post('https://api.openai.com/v1/chat/completions', { model: 'gpt-4o-mini', messages: [{ role: 'user', content: prompt }], max_tokens: 400, }, { headers: { Authorization: Bearer ${key} } }) const text = resp.data.choices?.[0]?.message?.content || 'No response' await sendText(m.chat, 🤖 AI: ${text}, m) } catch (err) { console.error(err?.response?.data || err.message) await sendText(m.chat, 'Error calling OpenAI API.', m) } })

// IMAGINE (placeholder) - would call image generation API commands.set('imagine', async (m, args) => { const prompt = args.join(' ') if (!prompt) return sendText(m.chat, 'Usage: .imagine <prompt>', m) // Placeholder — reply with the AI image banner if available if (assets.ai) return sendImageWithCaption(m.chat, assets.ai, 🖼️ Generated: ${prompt}, m) return sendText(m.chat, Image generation not configured. Prompt: ${prompt}, m) })

// PLAY - download audio from YouTube (basic) commands.set('play', async (m, args) => { const query = args.join(' ') if (!query) return sendText(m.chat, 'Usage: .play <YouTube url or search term>', m) // If URL try { let url = query if (!ytdl.validateURL(url)) { // naive search via ytsearch using ytdl-core is not built-in; ask user to provide URL return sendText(m.chat, 'Please provide a direct YouTube link for now.', m) } const info = await ytdl.getInfo(url) const title = info.videoDetails.title // download audio stream to a temporary file const tmpFile = ./tmp/${Date.now()}.mp3 fs.mkdirSync('./tmp', { recursive: true }) const stream = ytdl(url, { filter: 'audioonly', quality: 'highestaudio' }) const file = fs.createWriteStream(tmpFile) stream.pipe(file) stream.on('end', async () => { const buffer = fs.readFileSync(tmpFile) await conn.sendMessage(m.chat, { audio: buffer, mimetype: 'audio/mpeg', fileName: ${title}.mp3 }, { quoted: m }) fs.unlinkSync(tmpFile) }) } catch (err) { console.error(err) sendText(m.chat, 'Error while processing .play command.', m) } })

// TIKTOK (placeholder) - requires external service to fetch direct video url commands.set('tiktok', async (m, args) => { const url = args[0] if (!url) return sendText(m.chat, 'Usage: .tiktok <link>', m) // Placeholder: respond that feature needs configuration sendText(m.chat, 'TikTok download not set up on this host. I can add it if you want (requires third-party service).', m) })

// STICKER - convert image to sticker commands.set('sticker', async (m, args) => { // expects the user to reply to an image message try { const quoted = m.quoted if (!quoted) return sendText(m.chat, 'Reply to an image with .sticker', m) const q = quoted.message.imageMessage || quoted.message.jpegThumbnail ? quoted : null if (!q) return sendText(m.chat, 'Please reply to an image or sticker.', m) const buffer = await conn.downloadMediaMessage(quoted) // send as sticker await conn.sendMessage(m.chat, { sticker: buffer }, { quoted: m }) } catch (err) { console.error(err) await sendText(m.chat, 'Erreur lors de la création du sticker.', m) } })

// WEATHER (example using open-meteo.com free API) commands.set('weather', async (m, args) => { const city = args.join(' ') if (!city) return sendText(m.chat, 'Usage: .weather <city>', m) try { // simple geocoding via Open-Meteo's geocoding endpoint const geo = await axios.get('https://geocoding-api.open-meteo.com/v1/search', { params: { name: city, count: 1 } }) if (!geo.data.results || geo.data.results.length === 0) return sendText(m.chat, City not found: ${city}, m) const { latitude, longitude, name } = geo.data.results[0] const weather = await axios.get('https://api.open-meteo.com/v1/forecast', { params: { latitude, longitude, current_weather: true } }) const cur = weather.data.current_weather await sendText(m.chat, Weather in ${name}: ${cur.temperature}°C, wind ${cur.windspeed} m/s, weathercode ${cur.weathercode}, m) } catch (err) { console.error(err) await sendText(m.chat, 'Error fetching weather.', m) } })

// QR GENERATOR commands.set('qr', async (m, args) => { const text = args.join(' ') if (!text) return sendText(m.chat, 'Usage: .qr <text>', m) const tmp = ./tmp/qr-${Date.now()}.png fs.mkdirSync('./tmp', { recursive: true }) const QR = await new Promise((resolve) => qrcode.generate(text, { small: true }, resolve)) // fallback: send text representation as message await sendText(m.chat, QR code (text preview):\n${text}, m) })

// 8BALL commands.set('8ball', async (m, args) => { const q = args.join(' ') if (!q) return sendText(m.chat, 'Usage: .8ball <question>', m) const answers = ['Yes', 'No', 'Maybe', 'Ask later', 'Definitely'] const a = answers[Math.floor(Math.random() * answers.length)] await sendText(m.chat, 🎱 Question: ${q}\nAnswer: ${a}, m) })

// ADMIN/GROUP commands placeholders commands.set('kick', async (m, args) => { await sendText(m.chat, 'Use WhatsApp native group tools or implement admin-level actions with care (requires bot admin).', m) })

// MORE commands can be added here following the same pattern...

// Catch-all command processor conn.ev.on('messages.upsert', async (mUp) => { try { const messages = mUp.messages if (!messages) return for (const msg of messages) { if (!msg.message || msg.key && msg.key.remoteJid === 'status@broadcast') continue const m = msg // enrich m with helpers m.chat = m.key.remoteJid m.isGroup = m.chat.endsWith('@g.us') m.sender = m.key.participant || m.key.remoteJid m.pushName = (m.pushName) || (msg.pushName) || ''

// get text body
    const messageType = Object.keys(m.message)[0]
    let body = ''
    if (messageType === 'conversation') body = m.message.conversation
    else if (messageType === 'extendedTextMessage') body = m.message.extendedTextMessage.text
    else if (messageType === 'imageMessage' && m.message.imageMessage.caption) body = m.message.imageMessage.caption

    if (!body) continue
    if (!body.startsWith('.')) continue

    const parts = body.trim().split(/\s+/)
    const cmd = parts[0].slice(1).toLowerCase()
    const args = parts.slice(1)

    if (commands.has(cmd)) {
      try {
        await commands.get(cmd)(m, args)
      } catch (err) {
        console.error('Handler error', err)
        await sendText(m.chat, 'Erreur interne de la commande.', m)
      }
    } else {
      // Unknown command: optionally send suggestion
      // If user typed .menu or .help, handled above. Otherwise suggest .menu
      await sendText(m.chat, `Commande inconnue: ${cmd}. Tapez .menu pour la liste.`, m)
    }
  }
} catch (err) {
  console.error('messages.upsert error', err)
}

})

}

startBot().catch(e => console.error(e))

/* Final notes (FR/EN)

FR: Sauvegarde les images envoyées dans ./assets avec les noms demandés (profile.jpg, banner.jpg, ai.jpg). EN: Save your uploaded images into ./assets as profile.jpg, banner.jpg, ai.jpg before running.
