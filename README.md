<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>METRO LTA - Chef Thierry</title>
    <style>
        :root { --main-bg: #f4f7f6; }
        body { font-family: sans-serif; background: var(--main-bg); margin: 0; padding: 10px; padding-bottom: 120px; }
        
        /* Liste vide */
        .empty-state { text-align: center; color: #95a5a6; margin-top: 40px; border: 2px dashed #bdc3c7; border-radius: 15px; padding: 30px; }
        
        /* Catégories */
        .category-block { background: white; border-radius: 10px; margin-bottom: 15px; box-shadow: 0 4px 6px rgba(0,0,0,0.1); overflow: hidden; border-left: 10px solid var(--c); }
        .category-header { background: var(--c); color: white; padding: 12px; font-weight: bold; font-size: 0.9em; text-transform: uppercase; }
        
        /* Articles */
        .item { display: flex; align-items: center; padding: 15px; border-bottom: 1px solid #eee; background: white; }
        .item:active { background: #f9f9f9; }
        input[type="checkbox"] { width: 30px; height: 30px; margin-right: 15px; }
        .qty { font-weight: bold; color: #2c3e50; background: #ecf0f1; padding: 3px 8px; border-radius: 5px; margin-right: 10px; }
        .item-text { flex-grow: 1; font-size: 1.2em; color: #34495e; }

        /* Barre de commandes fixe en bas */
        .footer-controls { position: fixed; bottom: 0; left: 0; width: 100%; background: white; padding: 20px 0; display: flex; justify-content: center; gap: 30px; box-shadow: 0 -5px 15px rgba(0,0,0,0.1); z-index: 1000; }
        .btn { border: none; border-radius: 50%; width: 70px; height: 70px; color: white; cursor: pointer; display: flex; align-items: center; justify-content: center; font-size: 30px; box-shadow: 0 4px 10px rgba(0,0,0,0.2); }
        .btn-kb { background: #3498db; }
        .btn-mic { background: #e74c3c; }
        .btn-mic.active { background: #27ae60; animation: blink 1s infinite; }

        @keyframes blink { 0% { opacity: 1; } 50% { opacity: 0.7; } 100% { opacity: 1; } }

        /* Saisie Manuelle */
        #input-overlay { display:none; position:fixed; top:0; left:0; width:100%; height:100%; background:rgba(0,0,0,0.8); z-index:2000; padding:20px; box-sizing:border-box; }
        .input-card { background: white; padding: 20px; border-radius: 15px; margin-top: 50px; }
        .input-card input { width: 100%; padding: 15px; font-size: 1.2em; border: 2px solid #ddd; border-radius: 8px; box-sizing: border-box; }
        .input-card button { width: 100%; padding: 15px; margin-top: 10px; background: #3498db; color: white; border: none; border-radius: 8px; font-weight: bold; font-size: 1.1em; }
    </style>
</head>
<body>

<div id="app">
    <div class="empty-state">
        <h2>👨‍🍳 Bonjour Chef !</h2>
        <p>Votre liste Metro est vide.<br>Utilisez les boutons ci-dessous pour ajouter vos produits.</p>
    </div>
</div>

<div id="input-overlay">
    <div class="input-card">
        <h3>Nouveau produit</h3>
        <input type="text" id="manualInput" placeholder="Ex: 5kg farine">
        <button onclick="processInput(document.getElementById('manualInput').value)">AJOUTER</button>
        <button onclick="closeInput()" style="background:#bdc3c7;">ANNULER</button>
    </div>
</div>

<div class="footer-controls">
    <button class="btn btn-kb" onclick="openInput()">⌨️</button>
    <button class="btn btn-mic" id="micBtn">🎤</button>
</div>

<script>
    const COLORS = {
        "HERBES": "#27ae60", "LÉGUMES": "#2ecc71", "BOF": "#f1c40f", "BOUCHERIE": "#c0392b",
        "SURGELÉ": "#3498db", "ALCOOLS": "#8e44ad", "ÉPICERIE_1": "#e67e22", "ASIATIQUE": "#d35400",
        "SOFTS": "#1abc9c", "ÉPICERIE_2": "#d35400", "ÉPICERIE_3": "#7f8c8d", "EAUX": "#2980b9",
        "PÂTISSERIE": "#fd79a8", "ÉQUIPEMENT": "#2c3e50", "À CLASSER": "#95a5a6"
    };
    const ORDER = Object.keys(COLORS);

    let memoire = JSON.parse(localStorage.getItem('metro_memoire')) || { "farine": "ÉPICERIE_1", "veau": "ÉPICERIE_2" };
    let list = JSON.parse(localStorage.getItem('metro_list')) || [];

    function openInput() { document.getElementById('input-overlay').style.display = 'block'; document.getElementById('manualInput').focus(); }
    function closeInput() { document.getElementById('input-overlay').style.display = 'none'; }

    function processInput(txt) {
        if (!txt) return;
        const regex = /^(\d+\s?(kg|g|l|cl|ml|bottes|colis|caisses|pièces|unités)?)\s?de?\s?(.*)/i;
        const match = txt.match(regex);
        const nom = match ? match[3] : txt;
        const qty = match ? match[1] : "";
        const cat = memoire[nom.toLowerCase()] || "À CLASSER";
        
        list.push({ id: Date.now(), nom, qty, cat, checked: false });
        localStorage.setItem('metro_list', JSON.stringify(list));
        closeInput();
        document.getElementById('manualInput').value = '';
        render();
    }

    function render() {
        const app = document.getElementById('app');
        const activeItems = list.filter(i => !i.checked);
        if (activeItems.length === 0) {
            app.innerHTML = '<div class="empty-state"><h2>👨‍🍳 Bonjour Chef !</h2><p>Votre liste est vide.</p></div>';
            return;
        }
        app.innerHTML = '';
        ORDER.forEach(cat => {
            const items = activeItems.filter(i => i.cat === cat);
            if (items.length > 0) {
                const block = document.createElement('div');
                block.className = 'category-block';
                block.style.setProperty('--c', COLORS[cat]);
                let html = `<div class="category-header">${cat}</div>`;
                items.forEach(item => {
                    html += `<div class="item">
                        <input type="checkbox" onchange="checkItem(${item.id})">
                        <span class="item-text">${item.qty ? `<span class="qty">${item.qty}</span>` : ''}${item.nom}</span>
                    </div>`;
                });
                block.innerHTML = html;
                app.appendChild(block);
            }
        });
    }

    function checkItem(id) {
        list.find(i => i.id === id).checked = true;
        localStorage.setItem('metro_list', JSON.stringify(list));
        setTimeout(render, 300);
    }

    // MICROPHONE
    const micBtn = document.getElementById('micBtn');
    const Speech = window.webkitSpeechRecognition || window.SpeechRecognition;
    if (Speech) {
        const rec = new Speech();
        rec.lang = 'fr-FR';
        micBtn.onclick = () => { rec.start(); micBtn.classList.add('active'); };
        rec.onresult = (e) => { processInput(e.results[0][0].transcript); micBtn.classList.remove('active'); };
        rec.onend = () => micBtn.classList.remove('active');
    }

    render();
</script>
</body>
</html>
