const { default: makeWASocket, useMultiFileAuthState } = require('@whiskeysockets/baileys')
const chalk = require('chalk')

// Bot config
const BOT_NAME = 'Nexus'
const OWNER_NAME = 'DSM'
const VERSION = '2.0.0'

// Set your WhatsApp JID (phone number with @s.whatsapp.net)
const OWNER_JID = 'put-owner-whatsapp-jid-here' // Example: '1234567890@s.whatsapp.net'

let startTime = Date.now()
let totalMessages = 0

// Utilities
function generateNexusLinkCode() {
    return `Nexus${Math.floor(10000 + Math.random() * 90000)}`
}
function getUptime() {
    let ms = Date.now() - startTime
    let sec = Math.floor(ms / 1000)
    let min = Math.floor(sec / 60)
    let hr = Math.floor(min / 60)
    return `${hr}h ${min % 60}m ${sec % 60}s`
}
function isOwner(jid) {
    return jid === OWNER_JID
}

async function startBot() {
    const { state, saveCreds } = await useMultiFileAuthState('session')
    const sock = makeWASocket({
        auth: state,
        printQRInTerminal: true
    })

    sock.ev.on('creds.update', saveCreds)

    sock.ev.on('messages.upsert', async ({ messages }) => {
        const msg = messages[0]
        if (!msg.message) return
        const from = msg.key.remoteJid
        const text = msg.message.conversation || msg.message.extendedTextMessage?.text
        totalMessages++

        console.log(chalk.green(`💬 Message from ${from}: ${text}`))

        // === Nexus Commands ===

        // .alive
        if (text === '.alive') {
            await sock.sendMessage(from, { text: `✅ *${BOT_NAME}* is Online\n🚀 Owner: ${OWNER_NAME}\nVersion: ${VERSION}` })
        }

        // .ping
        else if (text === '.ping') {
            await sock.sendMessage(from, { text: "🏓 Pong! Nexus responding..." })
        }

        // .menu
        else if (text === '.menu') {
            await sock.sendMessage(from, { text: `
╔═══════════════════╗\n   *🤖 ${BOT_NAME}*  \n   Version: *${VERSION}*\n   Owner: *${OWNER_NAME}*\n╚═══════════════════╝\n\n*Available Commands:*\n.alive - Check bot status\n.ping - Test response speed\n.menu - Show this menu\n.link - Generate a WhatsApp link code\n.help - Usage guide\n.about - Info about bot and DSM\n.uptime - Show bot uptime\n.stats - Bot stats\n.random - Fun fact or meme\n.weather <city> - Weather info\n.news - Latest news headlines\n.max - Show max features\n\n*Owner Only:*\n.broadcast <msg>\n.setname <name>\n.setstatus <status>`})
        }

        // .link
        else if (text === '.link') {
            const linkCode = generateNexusLinkCode()
            await sock.sendMessage(from, {
                text: `🔗 *${BOT_NAME} Link Code*\n\nYour unique WhatsApp link code is:\n*${linkCode}*\n\nShare this code to link with *${BOT_NAME}* (Owner: ${OWNER_NAME})!`
            })
        }

        // .help
        else if (text === '.help') {
            await sock.sendMessage(from, {
                text: `*${BOT_NAME} Help*\n\nUse .menu to see all commands. Ask ${OWNER_NAME} for support.`
            })
        }

        // .about
        else if (text === '.about') {
            await sock.sendMessage(from, {
                text: `*${BOT_NAME}*\nWhatsApp Bot by ${OWNER_NAME}.\nVersion: ${VERSION}\n\nContact owner for support!`
            })
        }

        // .uptime
        else if (text === '.uptime') {
            await sock.sendMessage(from, { text: `⏱️ Uptime: ${getUptime()}` })
        }

        // .stats
        else if (text === '.stats') {
            await sock.sendMessage(from, { text: `📊 Bot Stats:\nMessages handled: ${totalMessages}\nUptime: ${getUptime()}` })
        }

        // .random
        else if (text === '.random') {
            const facts = [
                "Did you know? Honey never spoils.",
                "Random meme: https://i.imgur.com/4M7IWwP.jpg",
                "Fun fact: Octopuses have three hearts.",
                "Joke: Why did the bot cross the road? To automate the chicken!"
            ]
            await sock.sendMessage(from, { text: facts[Math.floor(Math.random() * facts.length)] })
        }

        // .weather <city>
        else if (text.startsWith('.weather ')) {
            const city = text.slice(9).trim()
            // TODO: Integrate weather API here (stub)
            await sock.sendMessage(from, { text: `🌦️ Weather for ${city}: [API integration needed]` })
        }

        // .news
        else if (text === '.news') {
            // TODO: Integrate news API here (stub)
            await sock.sendMessage(from, { text: `📰 Latest news: [API integration needed]` })
        }

        // .max
        else if (text === '.max') {
            await sock.sendMessage(from, { text: `\n*MAX Mode: All Features Unlocked*\n- Owner: ${OWNER_NAME}\n- Bot: ${BOT_NAME} v${VERSION}\n- Uptime: ${getUptime()}\n- Commands: .alive .ping .menu .link .help .about .uptime .stats .random .weather .news\n- Owner Only: .broadcast .setname .setstatus\n- [API features need setup by owner]`})
        }

        // Owner-only: .broadcast <msg>
        else if (text.startsWith('.broadcast ') && isOwner(from)) {
            const broadcastMsg = text.slice(11).trim()
            // NOTE: You need to load all chats and send message to each
            await sock.sendMessage(from, { text: "Broadcast feature: [Owner implementation needed]" })
        }

        // Owner-only: .setname <name>
        else if (text.startsWith('.setname ') && isOwner(from)) {
            // NOTE: Changing bot name at runtime is not supported in this skeleton
            await sock.sendMessage(from, { text: "Set name feature: [Owner implementation needed]" })
        }

        // Owner-only: .setstatus <status>
        else if (text.startsWith('.setstatus ') && isOwner(from)) {
            // NOTE: Changing bot status at runtime is not supported in this skeleton
            await sock.sendMessage(from, { text: "Set status feature: [Owner implementation needed]" })
        }
    })
}

startBot()