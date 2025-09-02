# nexus
WhatsApp bot
// Nexus by DSM
// DSM Nexus v1.0.0

const { default: makeWASocket, useMultiFileAuthState } = require('@whiskeysockets/baileys')
const chalk = require('chalk')

async function startBot() {
    const { state, saveCreds } = await useMultiFileAuthState('session')
    const sock = makeWASocket({
        auth: state,
        printQRInTerminal: true
    })

    sock.ev.on('creds.update', saveCreds)

    // Event: New Message
    sock.ev.on('messages.upsert', async ({ messages }) => {
        const msg = messages[0]
        if (!msg.message) return
        const from = msg.key.remoteJid
        const text = msg.message.conversation || msg.message.extendedTextMessage?.text

        console.log(chalk.green(`💬 Message from ${from}: ${text}`))

        // === DSM Nexus Commands ===

        // .alive
        if (text === '.alive') {
            await sock.sendMessage(from, { text: "✅ *DSM Nexus is Online*\n🚀 Powered by DSM" })
        }

        // .ping
        if (text === '.ping') {
            await sock.sendMessage(from, { text: "🏓 Pong! DSM Nexus responding..." })
        }

        // .menu
        if (text === '.menu') {
            await sock.sendMessage(from, { text: `
╔═══════════════════╗
   *🤖 DSM Nexus*  
   Version: *1.0.0*
   By: DSM Tech
╚═══════════════════╝

*Available Commands:*
.alive - Check bot status
.ping - Test response speed
.menu - Show this menu
            `})
        }
    })
}

startBot()
