**🗺️ Guide : Connecter Claude Desktop à QGIS sur Ubuntu Linux**  
**Comment piloter QGIS avec l'intelligence artificielle Claude - pour les grands débutants**  
*✍️ * *Rédigé par Jean-Marie Prévost - Géomaticien, Seine-et-Marne*  
 *  
 📅 * *Configuration validée en mai 2026 sur Ubuntu 22.04 / QGIS 3.34.4 "Prizren"*  
![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnEAAAACCAYAAAA3pIp+AAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAANUlEQVR4nO3OMQ2AABAAsSPBCUZfEnoYmFDBhAU2QtIq6DIzW7UHAMBfnGt1V8fXEwAAXrse/wcF74lXkIsAAAAASUVORK5CYII=)  
**🧭 Avant de commencer - Comprendre ce qu'on va faire**  
Imaginez que QGIS est un atelier de cartographie, et que Claude est un assistant très compétent.  
   
 Normalement, Claude et QGIS ne se parlent pas - ils sont dans deux mondes séparés.  
Ce guide explique comment construire un **pont de communication** entre les deux, appelé  **MCP** (*Model Context Protocol*). Une fois ce pont en place, je peux dire à Claude, en français naturel :  
*"Crée une couche de points en Lambert 93 et applique-lui une symbologie rouge"*  
Et Claude le fait **directement dans mon QGIS ouvert**, en temps réel. ✨  
![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnEAAAACCAYAAAA3pIp+AAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAAM0lEQVR4nO3OUQmAQBBAwSdcjsu6HYxoDsEK/okwk2COmdnVGQAAf3GtalX76wkAAK/dDxFWBDkFf6+SAAAAAElFTkSuQmCC)  
**📋 Ce dont j'ai besoin**  
Avant de commencer, je vérifie que j'ai bien :  
| | |  
|-|-|  
| **Élément** | **Comment vérifier** |   
| Ubuntu Linux (20.04 ou plus récent) | lsb_release -a dans le terminal |   
| QGIS 3.x installé | qgis --version dans le terminal |   
| Python 3.10 ou plus | python3 --version dans le terminal |   
| Git installé | git --version dans le terminal |   
| Connexion internet | Pour télécharger les outils |   
| Un compte Anthropic (claude.ai) | Pour se connecter à Claude Desktop |   
   
![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnEAAAACCAYAAAA3pIp+AAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAANklEQVR4nO3OMQ2AABAAsSNBACPiUML0NpGACyywEZJWQZeZ2aszAAD+4l6rrTq+ngAA8Nr1AL/SBEZwuCSwAAAAAElFTkSuQmCC)  
**🏗️ Architecture - Comment ça marche**  
┌─────────────────────────────────────────────────────────────┐  
 │                                                             │  
 │   Claude Desktop  ←→  Serveur MCP  ←→  Plugin QGIS MCP    │  
 │   (interface chat)     (pont local)     (inside QGIS)       │  
 │                                                             │  
 └─────────────────────────────────────────────────────────────┘  
   
Il y a **4 composants** à installer :  
1. **Claude Desktop** - l'application de chat (version Linux non officielle)  
2. **uv** - un gestionnaire de paquets Python ultra-rapide (requis par le plugin)  
3. **Le plugin QGIS MCP** - qui ouvre un serveur dans QGIS  
4. **Le fichier de configuration** - qui dit à Claude Desktop comment trouver QGIS  
![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnEAAAACCAYAAAA3pIp+AAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAANUlEQVR4nO3OMQ2AABAAsSNBCUrfDqrYGVDAgAU2QtIq6DIzW7UHAMBfHGt1V+fXEwAAXrseHCQGBEuErVgAAAAASUVORK5CYII=)  
**🔧 ÉTAPE 1 - Installer Claude Desktop**  
*⚠️ * ***Important :*** * Anthropic ne fournit pas officiellement Claude Desktop pour Linux.*  
 *  
 J'utilise le projet communautaire * *claude-desktop-debian* * qui repackage l'application Windows pour Linux.*  
 *  
 Source : *[ *https://github.com/aaddrick/claude-desktop-debian*](https://github.com/aaddrick/claude-desktop-debian "https://github.com/aaddrick/claude-desktop-debian")  
**1.1 - Ajouter le dépôt et installer**  
J'ouvre un **terminal** (Ctrl+Alt+T) et je colle ces commandes  **une par une** :  
# Ajouter la clé de sécurité du dépôt  
 curl -fsSL https://pkg.claude-desktop-debian.dev/KEY.gpg | \  
   sudo gpg --dearmor -o /usr/share/keyrings/claude-desktop.gpg  
   
# Ajouter le dépôt aux sources  
 echo "deb [signed-by=/usr/share/keyrings/claude-desktop.gpg arch=amd64,arm64] \  
   https://pkg.claude-desktop-debian.dev stable main" | \  
   sudo tee /etc/apt/sources.list.d/claude-desktop.list  
   
# Mettre à jour et installer  
 sudo apt update && sudo apt install claude-desktop  
   
**1.2 - Vérifier l'installation**  
claude-desktop --doctor  
   
✅ Si la commande répond sans erreur, Claude Desktop est installé.  
**1.3 - Lancer et se connecter**  
claude-desktop  
   
Je me connecte avec mon compte Anthropic (celui de claude.ai).  
![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnEAAAACCAYAAAA3pIp+AAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAANUlEQVR4nO3OMQ2AABAAsSNBCUrfD6LYGNDAgAU2QtIq6DIzW7UHAMBfHGt1V+fXEwAAXrseHDAF/orRG+cAAAAASUVORK5CYII=)  
**🔧 ÉTAPE 2 - Installer **uv  
uv est un outil Python nécessaire pour que le plugin QGIS MCP fonctionne.  
   
 Source : [https://astral.sh/uv](https://astral.sh/uv "https://astral.sh/uv")  
**2.1 - Installer uv**  
curl -LsSf https://astral.sh/uv/install.sh | sh  
   
# Recharger le terminal pour que la commande soit reconnue  
 source ~/.bashrc  
   
**2.2 - Vérifier que uv est bien installé**  
uv --version  
   
✅ Je dois voir quelque chose comme : uv 0.11.16 (x86_64-unknown-linux-gnu)  
**2.3 - S'assurer que uv est dans le PATH (important !)**  
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc  
 source ~/.bashrc  
   
*💡 * ***Pourquoi ?*** * QGIS ne charge pas automatiquement le PATH du terminal. Cette ligne garantit que QGIS trouve * *uv* * à chaque démarrage.*  
![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnEAAAACCAYAAAA3pIp+AAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAANklEQVR4nO3OQQmAABRAsSfYxZo/kSGMYQLPJrCCNxG2BFtmZquOAAD4i3Ot7mr/egIAwGvXA4qrBdGuSdJuAAAAAElFTkSuQmCC)  
**🔧 ÉTAPE 3 - Installer le plugin QGIS MCP**  
Le plugin QGIS MCP crée un serveur à l'intérieur de QGIS qui écoute les commandes de Claude.  
   
 Source : [https://github.com/nkarasiak/qgis-mcp](https://github.com/nkarasiak/qgis-mcp "https://github.com/nkarasiak/qgis-mcp")  
**3.1 - Télécharger le plugin**  
git clone https://github.com/nkarasiak/qgis-mcp ~/qgis-mcp  
   
*Ce téléchargement crée le dossier * *~/qgis-mcp* * dans mon répertoire personnel.*  
**3.2 - Créer le dossier des plugins QGIS (s'il n'existe pas)**  
mkdir -p ~/.local/share/QGIS/QGIS3/profiles/default/python/plugins/  
   
**3.3 - Créer un lien symbolique**  
ln -s ~/qgis-mcp/qgis_mcp_plugin \  
   ~/.local/share/QGIS/QGIS3/profiles/default/python/plugins/qgis_mcp_plugin  
   
*💡 * ***C'est quoi un lien symbolique ?*** * C'est comme un raccourci. QGIS pense que le plugin est dans son dossier, mais en réalité il lit depuis * *~/qgis-mcp* *. Très pratique pour les mises à jour !*  
**3.4 - Vérifier que le lien est bien créé**  
ls ~/.local/share/QGIS/QGIS3/profiles/default/python/plugins/  
   
✅ Je dois voir qgis_mcp_plugin dans la liste.  
![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnEAAAACCAYAAAA3pIp+AAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAANUlEQVR4nO3OMQ2AABAAsSNhZscZXlheJwqQgQU2QtIq6DIze3UGAMBf3Gu1VcfXEwAAXrseop8EQrmJduIAAAAASUVORK5CYII=)  
**🔧 ÉTAPE 4 - Activer le plugin dans QGIS**  
**4.1 - Lancer QGIS depuis le terminal (obligatoire !)**  
*⚠️ * ***Point critique :*** * Je dois toujours lancer QGIS depuis le terminal avec cette commande, pas depuis l'icône du bureau. Cela garantit que QGIS voit * *uv* * dans son environnement.*  
export PATH="$HOME/.local/bin:$PATH" && qgis &  
   
**4.2 - Activer le plugin**  
Dans QGIS :  
1. Menu **Extensions** →  **Installer/Gérer les extensions**  
2. Onglet **Installé**  
3. Chercher **"QGIS MCP"**  
4. ✅ Cocher la case  
Un bouton **MCP** apparaît dans la barre d'outils de QGIS.  
**4.3 - Vérifier le Health Checklist**  
1. Cliquer sur le bouton **MCP** dans la barre d'outils  
2. Onglet **Guide** → section  **Health Checklist**  
Je dois voir les 4 cases en vert :  
✅ Plugin Link Status  : linked  
 ✅ uv Installation     : found  
 ✅ Python Venv Ready   : ready  
 ✅ MCP Server Entry Point : exists  
   
*🔴 * ***Si *** ***uv Installation*** *** est en rouge :*** * Cliquer sur * ***"Setup Environment"*** * puis * ***"Refresh Checklist"*** *.*  
 *  
 Si toujours rouge, voir la section Dépannage en fin de guide.*  
![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnEAAAACCAYAAAA3pIp+AAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAANUlEQVR4nO3OQQmAABRAsSd4NIGBzPXBmAawhhW8ibAl2DIze3UGAMBf3Gu1VcfXEwAAXrsehaQEN+8fLHEAAAAASUVORK5CYII=)  
**🔧 ÉTAPE 5 - Configurer la connexion Claude ↔ QGIS**  
**5.1 - Écrire le fichier de configuration**  
Je colle ce bloc complet dans le terminal :  
python3 << 'PYEOF'  
 import json  
   
 config = {  
   "preferences": {},  
   "mcpServers": {  
     "qgis": {  
       "command": "uv",  
       "args": ["run", "--no-sync", "src/qgis_mcp/server.py"],  
       "cwd": "/home/TON_NOM_UTILISATEUR/qgis-mcp"  
     }  
   }  
 }  
   
 import os  
 path = os.path.expanduser('~/.config/Claude/claude_desktop_config.json')  
 os.makedirs(os.path.dirname(path), exist_ok=True)  
   
 with open(path, 'w') as f:  
     json.dump(config, f, indent=2)  
   
 print("Fichier écrit avec succès !")  
 print("Chemin :", path)  
 PYEOF  
   
*⚠️ * ***Remplacer *** ***TON_NOM_UTILISATEUR*** * par ton nom d'utilisateur Linux.*  
 *  
 Pour connaître ton nom : * *echo $USER*  
**5.2 - Vérifier le fichier**  
cat ~/.config/Claude/claude_desktop_config.json  
   
✅ Je dois voir la section mcpServers avec qgis à l'intérieur.  
*💡 * ***Si j'ai déjà des préférences dans ce fichier*** *, je dois les conserver et ajouter uniquement la section * *mcpServers* * à la fin, avant le dernier * *}* *.*  
![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnEAAAACCAYAAAA3pIp+AAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAANUlEQVR4nO3OQQmAABRAsSd4NIGRTPXNaQBrWMGbCFuCLTOzV2cAAPzFvVZbdXw9AQDgtesBhZQEOYZGgUEAAAAASUVORK5CYII=)  
**🔧 ÉTAPE 6 - Démarrer le serveur QGIS**  
**6.1 - Démarrer via le bouton MCP (méthode normale)**  
Dans QGIS, cliquer sur le bouton **MCP** →  **"Start Server"**  
   
 ✅ Je dois voir : *"Server running on port 9876"*  
**6.2 - Si le bouton est grisé (méthode alternative)**  
Ouvrir la **Console Python de QGIS** (Ctrl+Alt+P) et coller :  
from qgis.utils import plugins  
 p = plugins.get('qgis_mcp_plugin')  
 p.start_server()  
 print("Serveur démarré !")  
   
✅ La console affiche : Serveur démarré !  
![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnEAAAACCAYAAAA3pIp+AAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAANUlEQVR4nO3OQQmAABRAsSfYxKK/kYXEkyk8WcGbCFuCLTOzVXsAAPzFuVZ3dXw9AQDgtesB/v8F8JQadPwAAAAASUVORK5CYII=)  
**🔧 ÉTAPE 7 - Connecter Claude Desktop**  
**7.1 - Relancer Claude Desktop**  
# Fermer Claude Desktop  
 pkill -f claude-desktop  
   
 # Attendre 2 secondes  
 sleep 2  
   
 # Relancer  
 claude-desktop  
   
**7.2 - Vérifier que les outils QGIS sont disponibles**  
En bas de la fenêtre de chat de Claude Desktop, je dois voir une icône **🔨 outils** avec un nombre (ex: *51 tools*).  
*🔴 * ***Si l'icône n'apparaît pas :*** * Vérifier que le serveur QGIS est démarré (étape 6), puis relancer Claude Desktop.*  
![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnEAAAACCAYAAAA3pIp+AAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAANklEQVR4nO3OMQ2AABAAsSNhRAF6EPYDLhGADSywEZJWQZeZ2aszAAD+4l6rrTq+ngAA8Nr1AIWsBDYDm5cLAAAAAElFTkSuQmCC)  
**✅ TEST FINAL - La connexion fonctionne !**  
Dans **Claude Desktop**, je tape ce message :  
Ping QGIS et dis-moi la version installée et le système d'exploitation.  
   
✅ Claude doit répondre quelque chose comme :  
*"QGIS est bien installé et répond. La version détectée est QGIS 3.34.4 "Prizren"."*  
**🎉 Félicitations ! Claude pilote QGIS en direct.**  
![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnEAAAACCAYAAAA3pIp+AAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAANklEQVR4nO3OMQ2AABAAsSPBCj5fFgpQwYwEZiywEZJWQZeZ2ao9AAD+4lyruzq+ngAA8Nr1AMTRBeEgNK9YAAAAAElFTkSuQmCC)  
**🚀 Premiers prompts à essayer**  
Une fois la connexion établie, voici des exemples de ce que je peux dire à Claude Desktop :  
Dans QGIS, crée une couche point temporaire nommée "test"   
 en EPSG:2154 avec un champ texte "nom".  
   
Dans QGIS, ajoute un point aux coordonnées   
 X=657000, Y=6858000 nommé "Mon site".  
   
Dans QGIS, lance un tampon de 500 mètres   
 sur la couche "test" et nomme le résultat "tampon_500m".  
   
Dans QGIS, applique une symbologie rouge semi-transparente   
 à la couche "tampon_500m".  
   
Dans QGIS, sauvegarde le projet sous ~/Documents/mon_projet.qgz  
   
![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnEAAAACCAYAAAA3pIp+AAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAANUlEQVR4nO3OQQmAABRAsSd40A5GMORPYEt7WMGbCFuCLTNzVFcAAPzFvVZbdX49AQDgtf0BSrIDUgOg4eAAAAAASUVORK5CYII=)  
**🔄 Procédure de démarrage quotidien**  
À chaque session, je suis cet ordre :  
1️⃣  Terminal → export PATH="$HOME/.local/bin:$PATH" && qgis &  
 2️⃣  QGIS → bouton MCP → "Start Server"  
 3️⃣  Claude Desktop → vérifier l'icône 🔨  
 4️⃣  C'est parti !  
   
![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnEAAAACCAYAAAA3pIp+AAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAANklEQVR4nO3OMQ2AABAAsSNBACPykMH4NpGACyywEZJWQZeZ2aszAAD+4l6rrTo+jgAA8N71AL/CBEiG5xPoAAAAAElFTkSuQmCC)  
**🛠️ Dépannage - Les problèmes courants**  
**❌ **uv ** introuvable dans QGIS**  
# Vérifier que uv est installé  
 uv --version  
   
 # Vérifier son emplacement  
 which uv  
   
 # Ajouter au PATH de façon permanente  
 echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc  
 source ~/.bashrc  
   
Puis toujours lancer QGIS avec :  
export PATH="$HOME/.local/bin:$PATH" && qgis  
   
**❌ Le plugin QGIS MCP n'apparaît pas**  
# Vérifier que le lien symbolique existe  
 ls ~/.local/share/QGIS/QGIS3/profiles/default/python/plugins/  
   
 # Si qgis_mcp_plugin est absent, le recréer  
 ln -s ~/qgis-mcp/qgis_mcp_plugin \  
   ~/.local/share/QGIS/QGIS3/profiles/default/python/plugins/qgis_mcp_plugin  
   
Redémarrer QGIS et réactiver le plugin dans le gestionnaire d'extensions.  
**❌ Claude Desktop ne voit pas les outils QGIS (pas d'icône 🔨)**  
Vérifier le fichier de config :  
cat ~/.config/Claude/claude_desktop_config.json  
   
La section mcpServers doit être présente. Puis vérifier que le serveur QGIS est bien démarré, et relancer Claude Desktop.  
**❌ Le bouton "Start Server" est grisé**  
Utiliser la console Python de QGIS (voir Étape 6.2).  
**❌ Port 9876 déjà occupé**  
Dans la console Python de QGIS :  
import socket  
 s = socket.socket()  
 try:  
     s.bind(('localhost', 9876))  
     print("Port libre")  
 except OSError as e:  
     print("Port occupé :", e)  
 finally:  
     s.close()  
   
Si le port est occupé, redémarrer QGIS.  
![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnEAAAACCAYAAAA3pIp+AAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAANUlEQVR4nO3OMQ2AABAAsSPBCUZfE2IYmVDBhAU2QtIq6DIzW7UHAMBfnGt1V8fXEwAAXrse/xcF7U7sx4wAAAAASUVORK5CYII=)  
**📚 Sources et références**  
| | |  
|-|-|  
| **Ressource** | **URL** |   
| Plugin QGIS MCP (nkarasiak) | [https://github.com/nkarasiak/qgis-mcp](https://github.com/nkarasiak/qgis-mcp "https://github.com/nkarasiak/qgis-mcp") |   
| Claude Desktop pour Debian/Ubuntu | [https://github.com/aaddrick/claude-desktop-debian](https://github.com/aaddrick/claude-desktop-debian "https://github.com/aaddrick/claude-desktop-debian") |   
| Gestionnaire de paquets uv | [https://astral.sh/uv](https://astral.sh/uv "https://astral.sh/uv") |   
| Documentation MCP Anthropic | [https://docs.anthropic.com/mcp](https://docs.anthropic.com/mcp "https://docs.anthropic.com/mcp") |   
| Documentation PyQGIS | [https://docs.qgis.org/latest/fr/docs/pyqgis_developer_cookbook](https://docs.qgis.org/latest/fr/docs/pyqgis_developer_cookbook "https://docs.qgis.org/latest/fr/docs/pyqgis_developer_cookbook") |   
   
![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnEAAAACCAYAAAA3pIp+AAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAANUlEQVR4nO3OYQ1AABSAwY9JoICqL4Z8Ikiggn9mu0twy8wc1RkAAH9xbdVa7V9PAAB47X4A9CgEJQFjJ/EAAAAASUVORK5CYII=)  
**ℹ️ Notes importantes**  
*🔔 * ***Claude Desktop sur Linux n'est pas officiel.*** * Anthropic supporte officiellement Claude Desktop uniquement sur macOS et Windows. Le paquet * *.deb* * utilisé ici est maintenu par la communauté.*  
*🔔 * ***Sécurité.*** * Le plugin MCP permet à Claude d'exécuter du code Python directement dans QGIS. Ne l'utiliser que sur des projets de confiance.*  
*🔔 * ***Mises à jour du plugin.*** * Pour mettre à jour le plugin QGIS MCP :*  
*cd ~/qgis-mcp && git pull  
 *  
*Puis redémarrer QGIS.*  
![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnEAAAACCAYAAAA3pIp+AAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAANklEQVR4nO3OQQmAABRAsScYxpg/jFnsYARvRrCCNxG2BFtmZquOAAD4i3Ot7mr/egIAwGvXA22QBcposvV4AAAAAElFTkSuQmCC)  
*© Jean-Marie Prévost *  
   
 *Configuration testée et validée : Ubuntu Linux, QGIS 3.34.4 "Prizren", uv 0.11.16, mai 2026*  
