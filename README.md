"scripts": {
  "start": "node index.js"
}
DSM Nexus Auth Code: NEXUS-482193
Use this code on dashboard.dsm.tech to link WhatsApp.
const crypto = require('crypto')
function generateCode() {
    return "NEXUS-" + crypto.randomBytes(3).toString("hex").toUpperCase()
}
console.log(generateCode())
dsm-nexus/
 ├── bot/
 │   └── index.js       # WhatsApp bot (Baileys)
 ├── server/
 │   └── server.js      # Express backend
 ├── web/
 │   ├── pages/         # Next.js frontend
 │   └── package.json
 ├── package.json
 └── README.md
// DSM Nexus Bot
// Version 1.0.0

const { default: makeWASocket, useMultiFileAuthState } = require('@whiskeysockets/baileys')
const crypto = require('crypto')
const path = require('path')

async function startBot(sessionPath = 'session') {
    const { state, saveCreds } = await useMultiFileAuthState(sessionPath)
    const sock = makeWASocket({
        auth: state,
        printQRInTerminal: false // We'll send QR to web
    })

    sock.ev.on('creds.update', saveCreds)

    // Generate NEXUS code
    function generateCode() {
        return "NEXUS-" + crypto.randomBytes(3).toString("hex").toUpperCase()
    }

    // Basic commands
    sock.ev.on('messages.upsert', async ({ messages }) => {
        const msg = messages[0]
        if (!msg.message) return
        const from = msg.key.remoteJid
        const text = msg.message.conversation || msg.message.extendedTextMessage?.text

        if (text === '.alive') {
            await sock.sendMessage(from, { text: "✅ DSM Nexus Online!" })
        }
        if (text === '.ping') {
            await sock.sendMessage(from, { text: "🏓 Pong from DSM Nexus!" })
        }
        if (text === '.menu') {
            await sock.sendMessage(from, { text: `
🤖 *DSM Nexus v1.0.0*
Commands:
.alive - Bot status
.ping - Response check
.menu - Show menu
            `})
        }
    })

    return { sock, generateCode }
}

module.exports = { startBot }
const express = require('express')
const qrcode = require('qrcode')
const { startBot } = require('../bot/index')

const app = express()
const PORT = process.env.PORT || 3001

let botInstance = null
let qrCodeData = ""

app.get('/start', async (req, res) => {
    if (!botInstance) {
        const { sock } = await startBot('session')
        botInstance = sock

        sock.ev.on('connection.update', async (update) => {
            const { qr, connection } = update
            if (qr) {
                qrCodeData = await qrcode.toDataURL(qr)
            }
            if (connection === 'open') {
                console.log("✅ DSM Nexus Connected!")
            }
        })
    }
    res.json({ message: "DSM Nexus started" })
})

app.get('/qr', (req, res) => {
    if (qrCodeData) {
        res.send(`<img src="${qrCodeData}" />`)
    } else {
        res.send("No QR generated yet. Start the bot first.")
    }
})

app.listen(PORT, () => {
    console.log(`🚀 DSM Nexus API running on port ${PORT}`)
})
import { useEffect, useState } from 'react'

export default function Home() {
  const [qr, setQr] = useState(null)

  const startBot = async () => {
    await fetch('/api/start')
    loadQr()
  }

  const loadQr = async () => {
    const res = await fetch('/api/qr')
    const text = await res.text()
    setQr(text)
  }

  return (
    <div className="p-6 text-center">
      <h1 className="text-3xl font-bold">🤖 DSM Nexus Dashboard</h1>
      <button 
        className="bg-blue-600 text-white px-4 py-2 rounded mt-4"
        onClick={startBot}>
        Start Bot
      </button>
      <div className="mt-6" dangerouslySetInnerHTML={{ __html: qr }} />
    </div>
  )
}
npm install express qrcode @whiskeysockets/baileys
cd web && npm install next react react-dom
node server/server.js
cd web && npm run dev