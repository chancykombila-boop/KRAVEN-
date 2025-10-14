# commands.py

def menu_principal():
    return (
        "🤖 *RAMSY BOT - MENU PRINCIPAL* 🤖\n\n"
        "1️⃣ Infos générales\n"
        "2️⃣ Traduction\n"
        "3️⃣ Deviner un chiffre\n"
        "4️⃣ Autres commandes fun\n"
        "Envoyez le numéro de votre choix."
    )

def repondre_commande(message):
    message = message.strip()
    
    if message == "1":
        return "Voici quelques infos intéressantes sur le monde et la tech 🌍💻"
    elif message == "2":
        return "Envoyez le texte que vous voulez traduire, je m’occupe du reste 📝"
    elif message == "3":
        import random
        return f"Devinez un chiffre entre 1 et 10 ! J’ai choisi : {random.randint(1,10)} 🤔"
    elif message == "4":
        return "Commandes fun : envoyez 'blague', 'citation', ou 'jeu' 🎉"
    else:
        return "Commande inconnue 😅, envoyez 'menu' pour voir le menu principal."from flask import Flask, request
from twilio.twiml.messaging_response import MessagingResponse
from openai import OpenAI
import os
from dotenv import load_dotenv
from commands import menu_principal, repondre_commande
from history import ajouter_message, recuperer_historique

load_dotenv()

client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))
app = Flask(__name__)

@app.route("/whatsapp", methods=["POST"])
def whatsapp_reply():
    incoming_msg = request.values.get("Body", "").strip()
    user = request.values.get("From")  # numéro WhatsApp de l'utilisateur
    resp = MessagingResponse()
    
    # Vérifier si c'est une commande personnalisée
    reply = repondre_commande(incoming_msg)
    
    if reply:
        resp.message(reply)
    else:
        # Ajouter le message utilisateur à l'historique
        ajouter_message(user, "user", incoming_msg)
        # Envoyer l'historique à ChatGPT
        history = recuperer_historique(user)
        
        response = client.chat.completions.create(
            model="gpt-5",
            messages=history
        )
        reply = response.choices[0].message.content
        # Ajouter la réponse à l'historique
        ajouter_message(user, "assistant", reply)
        resp.message(reply)
    
    return str(resp)

if __name__ == "__main__":
    app.run(port=5000)return "🎉 Surprise ! Voici ton image : https://tonsite.com/images/monimage.jpg"from flask import Flask, request
from twilio.twiml.messaging_response import MessagingResponse
from openai import OpenAI
import os
from dotenv import load_dotenv

# Charger les variables d'environnement
load_dotenv()

# Initialiser OpenAI
client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))

app = Flask(__name__)

@app.route("/whatsapp", methods=["POST"])
def whatsapp_reply():
    # Récupérer le message de l'utilisateur
    incoming_msg = request.values.get("Body", "")
    
    # Envoyer à ChatGPT
    response = client.chat.completions.create(
        model="gpt-5",
        messages=[
            {"role": "user", "content": incoming_msg}
        ]
    )
    
    # Récupérer la réponse de ChatGPT
    reply = response.choices[0].message.content
    
    # Envoyer la réponse à WhatsApp
    resp = MessagingResponse()
    resp.message(reply)
    
    return str(resp)

if __name__ == "__main__":
    app.run(port=5000)
