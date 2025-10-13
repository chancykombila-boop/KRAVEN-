export default {
  command: ['menu'],
  description: 'Affiche le menu principal',
  async execute({ conn, msg }) {
    const menu = `
🤖 *RAMSY_KING - MENU PRINCIPAL* 🤖

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
    await conn.sendMessage(msg.key.remoteJid, { text: menu }, { quoted: msg });
  }
};
