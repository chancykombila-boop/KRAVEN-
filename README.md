const menu = `
🤖 *🤖 RAMSY BOT - LISTE DES MENUS* 🤖

╭──────────────────╮
│   *CATÉGORIES*   │
╰──────────────────╯

┌─ *🏠 MENUS PRINCIPAUX*
│ • .menu - Menu principal complet
│ • .menu2 - Menu alternatif
│ • .menuimage - Menu visuel avec image
│ • .menuall - Toutes les commandes

┌─ *🎯 MENUS SPÉCIALISÉS*
│ • .menubasic - Commandes essentielles
│ • .menumedia - Outils médias & création
│ • .menugroup - Gestion de groupe
│ • .menuai - Intelligence artificielle
│ • .menusettings - Paramètres bot

┌─ *🔧 COMMANDES RAPIDES*
│ • .ping - Test de latence
│ • .info - Informations bot
│ • .alive - Statut du bot
│ • .owner - Contact propriétaire
│ • .sticker - Créer sticker

💡 *Explorez chaque catégorie pour découvrir toutes les fonctionnalités !*
`;

const handler = async (m, { conn }) => {
  await conn.sendMessage(m.chat, { text: menu }, { quoted: m });
};

handler.command = ['menu', 'menu1'];
handler.help = ['menu'];
handler.tags = ['main'];

export default handler;const handler = async (m, { conn }) => {
  const start = new Date();
  const end = new Date();
  await conn.sendMessage(m.chat, { text: `🏓 Ping: ${end - start} ms` }, { quoted: m });
};

handler.command = ['ping'];
handler.help = ['ping'];
handler.tags = ['main'];

export default handler;
