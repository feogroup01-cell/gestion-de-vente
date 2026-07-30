<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <title>Gestion Commission - Propriétaire</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <style>
        * { margin:0; padding:0; box-sizing:border-box; font-family: Arial; }
        body { background:#f5f5f5; }
.header { background:#009d2a; color:white; padding:15px; text-align:center; position:sticky; top:0; display:flex; justify-content:space-between; align-items:center; }
.container { padding:15px; max-width:900px; margin:auto; }
.card { background:white; border-radius:12px; padding:15px; margin-bottom:15px; box-shadow:0 2px 5px rgba(0,0,0,0.1); }
.btn { background:#009d2a; color:white; border:none; padding:12px; border-radius:8px; width:100%; font-weight:bold; cursor:pointer; margin-top:10px; }
.btn-orange { background:#ff6600; }
.btn-rouge { background:#ff4444; padding:6px 10px; font-size:0.8em; width:auto; }
.btn-bleu { background:#007bff; padding:6px 10px; font-size:0.8em; width:auto; }
.btn-vert { background:#28a745; padding:6px 10px; font-size:0.8em; width:auto; }
.btn-violet { background:#6f42c1; }
        input, select, textarea { width:100%; padding:10px; margin:8px 0; border:1px solid #ddd; border-radius:8px; }
        table { width:100%; border-collapse:collapse; background:white; font-size:0.9em; }
        th { background:#009d2a; color:white; padding:10px; position:sticky; top:0; }
        td { padding:10px; border:1px solid #ddd; text-align:center; }
.page { display:none; }
.page.active { display:block; }
.montant-client { color:#009d2a; font-weight:bold; }
.montant-prop { color:#ff6600; font-weight:bold; }
.total-box { display:flex; gap:15px; flex-wrap:wrap; }
.total-box div { flex:1; min-width:150px; text-align:center; padding:15px; border-radius:8px; }
.box-client { background:#e8f5e9; }
.box-prop { background:#fff3e0; }
.box-total { background:#e3f2fd; }
.actions { display:flex; justify-content:center; gap:5px; flex-wrap:wrap; }
.ventes-jour { max-height:300px; overflow-y:auto; }
.filtre-mois { display:flex; gap:10px; }
    </style>
</head>
<body>

<div class="header">
    <h1>📊 GESTION COMMISSION</h1>
    <button onclick="showPage('commissionPage')" class="btn-violet" style="color:white; border:none; padding:8px; border-radius:5px;">Espace Commission</button>
</div>

<div id="propPage" class="page active">
    <div class="container">
        <div class="card" style="background:#fff3cd; border:2px solid #ff6600;">
            <h3>⚙️ Paramètres Propriétaire</h3>
            <label>Votre Numéro WhatsApp - Format: 225...</label>
            <input id="numProprietaire" type="tel" value="225501093051">
            <label>Votre % sur chaque vente</label>
            <input id="pctProprietaire" type="number" value="10">
            <button class="btn" onclick="sauverParametres()">Enregistrer Paramètres</button>
        </div>

        <div class="card">
            <h3>1. Ajouter un Client</h3>
            <input id="nomClient" placeholder="Nom du client">
            <input id="telClient" placeholder="Téléphone Client WhatsApp ex: 22507010203">
            <input id="pctClient" type="number" placeholder="Pourcentage Client % ex: 5">
            <button class="btn" onclick="ajouterClient()">Enregistrer Client</button>
        </div>

        <div class="card">
            <h3>2. Saisir une Vente</h3>
            <input id="searchClient" onkeyup="filtrerListeClient()" placeholder="🔍 Rechercher un client...">
            <select id="selectClient" size="5" style="height:120px;"></select>
            <input id="montantVente" type="number" placeholder="Montant de la vente FCFA">
            <input id="dateVente" type="date">
            <button class="btn btn-orange" onclick="ajouterVente()">+ Enregistrer la Vente</button>
        </div>

        <div class="card">
            <h3>📋 Ventes du Jour <span id="dateDuJour"></span></h3>
            <div class="ventes-jour">
                <table id="tableVentesJour"><tr><th>Heure</th><th>Client</th><th>Montant</th><th>Comm Client</th><th>Action</th></tr></table>
            </div>
            <div class="total-box" id="totalGlobal" style="margin-top:10px;"></div>
        </div>

        <div class="card">
            <h3>3. Résumé par Client du Jour</h3>
            <button class="btn" onclick="calculerJour()">Générer Résumé</button>
            <div id="resumeJour" style="overflow-x:auto; margin-top:10px;"></div>
        </div>

        <div class="card">
            <h3>Liste Clients Enregistrés <span id="nbClients"></span></h3>
            <table id="tableClients"><tr><th>Nom</th><th>% Client</th><th>Tél</th><th>Actions</th></tr></table>
        </div>

    </div>
</div>

<div id="commissionPage" class="page">
    <div class="container">
        <div class="card">
            <h3>📅 Historique des Commissions</h3>
            <div class="filtre-mois">
                <input id="moisDebut" type="month">
                <input id="moisFin" type="month">
            </div>
            <button class="btn btn-violet" onclick="voirHistoriqueCommission()">Voir Historique</button>
        </div>

        <div class="card">
            <h3>Commission des Clients</h3>
            <div id="tableCommissionClients"></div>
        </div>

        <div class="card">
            <h3>Commission du Propriétaire</h3>
            <div id="tableCommissionProp"></div>
        </div>

        <button class="btn" style="background:#666;" onclick="showPage('propPage')">Retour Accueil</button>
    </div>
</div>

<script>
let clients = JSON.parse(localStorage.getItem('clients')) || [];
let ventes = JSON.parse(localStorage.getItem('ventes')) || [];
let pctProprietaire = parseFloat(localStorage.getItem('pctProprietaire')) || 10;
let numProprietaire = localStorage.getItem('numProprietaire') || "225501093051";

function showPage(id){ document.querySelectorAll('.page').forEach(p=>p.classList.remove('active')); document.getElementById(id).classList.add('active'); chargerClients(); chargerVentesJour(); }

document.getElementById('pctProprietaire').value = pctProprietaire;
document.getElementById('numProprietaire').value = numProprietaire;
document.getElementById('moisDebut').value = new Date().toISOString().slice(0,7);
document.getElementById('moisFin').value = new Date().toISOString().slice(0,7);

function sauverParametres(){
    pctProprietaire = parseFloat(document.getElementById('pctProprietaire').value);
    numProprietaire = document.getElementById('numProprietaire').value;
    localStorage.setItem('pctProprietaire', pctProprietaire);
    localStorage.setItem('numProprietaire', numProprietaire);
    alert("Paramètres enregistrés");
}

function ajouterClient(){
    let nom = document.getElementById('nomClient').value;
    let tel = document.getElementById('telClient').value.replace(/[^0-9]/g, '');
    let pct = document.getElementById('pctClient').value;
    if(!nom ||!tel ||!pct) return alert("Remplir tout");
    if(!tel.startsWith('225')) return alert("Le numéro client doit commencer par 225");
    clients.push({id: Date.now(), nom, tel, pourcentage: parseFloat(pct)});
    localStorage.setItem('clients', JSON.stringify(clients));
    document.getElementById('nomClient').value = ""; document.getElementById('telClient').value = ""; document.getElementById('pctClient').value = "";
    alert("Client ajouté et sauvegardé");
    chargerClients();
}

function modifierClient(id){
    let nouveauPct = prompt("Nouveau pourcentage pour ce client:");
    if(nouveauPct!= null && nouveauPct!= ""){
        let client = clients.find(c => c.id == id);
        client.pourcentage = parseFloat(nouveauPct);
        localStorage.setItem('clients', JSON.stringify(clients));
        chargerClients();
        alert("Pourcentage mis à jour");
    }
}

function supprimerClient(id){
    if(confirm("Supprimer ce client et toutes ses ventes?")){
        clients = clients.filter(c => c.id!= id);
        ventes = ventes.filter(v => v.clientId!= id);
        localStorage.setItem('clients', JSON.stringify(clients));
        localStorage.setItem('ventes', JSON.stringify(ventes));
        chargerClients(); chargerVentesJour();
    }
}

function filtrerListeClient(){
    let filtre = document.getElementById('searchClient').value.toLowerCase();
    chargerClients(filtre);
}

function chargerClients(filtre=""){
    let select = document.getElementById('selectClient');
    let table = document.getElementById('tableClients');
    document.getElementById('nbClients').innerText = "("+clients.length+")";
    let clientsFiltres = clients.filter(c => c.nom.toLowerCase().includes(filtre));
    select.innerHTML = '';
    let rows = '<tr><th>Nom</th><th>% Client</th><th>Tél</th><th>Actions</th></tr>';
    clientsFiltres.forEach(c=>{
        select.innerHTML += `<option value="${c.id}">${c.nom} - ${c.pourcentage}%</option>`;
        rows += `<tr><td>${c.nom}</td><td>${c.pourcentage}%</td><td>${c.tel}</td><td class="actions"><button class="btn-vert" onclick="modifierClient(${c.id})">Modifier %</button><button class="btn-rouge" onclick="supprimerClient(${c.id})">Supprimer</button></td></tr>`;
    });
    table.innerHTML = rows;
}

function supprimerVente(id){
    if(confirm("Supprimer cette vente?")){
        ventes = ventes.filter(v => v.id!= id);
        localStorage.setItem('ventes', JSON.stringify(ventes));
        chargerVentesJour();
    }
}

function ajouterVente(){
    let clientId = parseInt(document.getElementById('selectClient').value);
    let montant = parseFloat(document.getElementById('montantVente').value);
    let date = document.getElementById('dateVente').value || new Date().toISOString().split('T')[0];
    if(!clientId ||!montant) return alert("Choisir client et montant");
    ventes.push({id: Date.now(), clientId, montant, date, heure: new Date().toLocaleTimeString('fr-FR', {hour:'2-digit', minute:'2-digit'})});
    localStorage.setItem('ventes', JSON.stringify(ventes));
    document.getElementById('montantVente').value = "";
    chargerVentesJour();
}

function chargerVentesJour(){
    let aujourdhui = document.getElementById('dateVente').value || new Date().toISOString().split('T')[0];
    document.getElementById('dateDuJour').innerText = aujourdhui;
    let ventesDuJour = ventes.filter(v => v.date == aujourdhui);
    let rows = '<tr><th>Heure</th><th>Client</th><th>Montant</th><th>Comm Client</th><th>Action</th></tr>';
    let totalVente = 0; let totalCommClient = 0; let totalCommProp = 0;
    ventesDuJour.forEach(v=>{
        let client = clients.find(c => c.id == v.clientId);
        if(client){
            let commClient = (v.montant * client.pourcentage) / 100;
            let commProp = (v.montant * pctProprietaire) / 100;
            totalVente += v.montant; totalCommClient += commClient; totalCommProp += commProp;
            rows += `<tr><td>${v.heure}</td><td>${client.nom}</td><td>${v.montant} FCFA</td><td class="montant-client">${commClient} FCFA</td><td><button class="btn-rouge" onclick="supprimerVente(${v.id})">X</button></td></tr>`;
        }
    });
    document.getElementById('tableVentesJour').innerHTML = rows;
    document.getElementById('totalGlobal').innerHTML = `<div class="box-client"><h4>Total Ventes</h4><p>${totalVente} FCFA</p></div><div class="box-client"><h4>Comm. Clients</h4><p class="montant-client">${totalCommClient} FCFA</p></div><div class="box-prop"><h4>Commission du Propriétaire</h4><p class="montant-prop">${totalCommProp} FCFA</p></div>`;
}

function calculerJour(){
    let aujourdhui = document.getElementById('dateVente').value || new Date().toISOString().split('T')[0];
    let ventesDuJour = ventes.filter(v => v.date == aujourdhui);
    if(ventesDuJour.length == 0) return document.getElementById('resumeJour').innerHTML = "<p>Aucune vente pour cette date</p>";
    let resume = '<table><tr><th>Client</th><th>Vente Totale</th><th>% Client</th><th>Commission Client</th><th>Commission du Propriétaire</th><th>Action</th></tr>';
    clients.forEach(client=>{
        let ventesClient = ventesDuJour.filter(v=>v.clientId==client.id);
        let totalClient = ventesClient.reduce((a,b)=>a+b.montant,0);
        if(totalClient > 0){
            let commissionClient = (totalClient * client.pourcentage) / 100;
            let commissionProp = (totalClient * pctProprietaire) / 100;
            resume += `<tr><td>${client.nom}</td><td>${totalClient} FCFA</td><td>${client.pourcentage}%</td><td class="montant-client">${commissionClient} FCFA</td><td class="montant-prop">${commissionProp} FCFA</td><td><button class="btn-bleu" onclick="envoyerClient(${client.id})">📲 Envoyer</button></td></tr>`;
        }
    });
    resume += '</table>';
    document.getElementById('resumeJour').innerHTML = resume;
}

function envoyerClient(clientId){
    if(!numProprietaire) return alert("Renseigner votre numéro WhatsApp en haut d'abord");
    let aujourdhui = document.getElementById('dateVente').value || new Date().toISOString().split('T')[0];
    let client = clients.find(c => c.id == clientId);
    let ventesClient = ventes.filter(v=>v.clientId==clientId && v.date==aujourdhui);
    let totalClient = ventesClient.reduce((a,b)=>a+b.montant,0);
    let commissionClient = (totalClient * client.pourcentage) / 100;
    let commissionProp = (totalClient * pctProprietaire) / 100;
    let detailVentes = "";
    ventesClient.forEach(v => detailVentes += `- ${v.heure} : ${v.montant} FCFA\n`);
    let message = `*RECAP DU ${aujourdhui}*\n\nBonjour ${client.nom}\n\nVos ventes du jour:\n${detailVentes}\n*Total Vente: ${totalClient} FCFA*\n*Votre %: ${client.pourcentage}%*\n*Votre Commission: ${commissionClient} FCFA*\n*Commission du Propriétaire: ${commissionProp} FCFA*\n\nEnvoyé par: ${numProprietaire}\nMerci`;
    let url = `https://wa.me/${client.tel}?text=${encodeURIComponent(message)}`;
    window.open(url, '_blank');
}

// NOUVEAU : ESPACE COMMISSION
function voirHistoriqueCommission(){
    let debut = document.getElementById('moisDebut').value + "-01";
    let fin = document.getElementById('moisFin').value + "-31";

    let ventesPeriode = ventes.filter(v => v.date >= debut && v.date <= fin);
    let ventesTotal = ventes; // depuis le début

    // TABLEAU CLIENTS
    let tableClients = '<table><tr><th>Client</th><th>%</th><th>Commission Période</th><th>Commission Totale</th></tr>';
    let totalCommClientsPeriode = 0;
    let totalCommClientsGlobal = 0;

    clients.forEach(client=>{
        let ventesClientPeriode = ventesPeriode.filter(v=>v.clientId==client.id).reduce((a,b)=>a+b.montant,0);
        let ventesClientTotal = ventesTotal.filter(v=>v.clientId==client.id).reduce((a,b)=>a+b.montant,0);

        let commPeriode = (ventesClientPeriode * client.pourcentage) / 100;
        let commTotal = (ventesClientTotal * client.pourcentage) / 100;

        totalCommClientsPeriode += commPeriode;
        totalCommClientsGlobal += commTotal;

        if(commPeriode > 0 || commTotal > 0){
            tableClients += `<tr><td>${client.nom}</td><td>${client.pourcentage}%</td><td class="montant-client">${commPeriode} FCFA</td><td class="montant-client">${commTotal} FCFA</td></tr>`;
        }
    });
    tableClients += `<tr style="background:#e8f5e9; font-weight:bold;"><td>TOTAL CLIENTS</td><td></td><td class="montant-client">${totalCommClientsPeriode} FCFA</td><td class="montant-client">${totalCommClientsGlobal} FCFA</td></tr></table>`;
    document.getElementById('tableCommissionClients').innerHTML = tableClients;

    // TABLEAU PROPRIETAIRE
    let ventesPropPeriode = ventesPeriode.reduce((a,b)=>a+b.montant,0);
    let ventesPropTotal = ventesTotal.reduce((a,b)=>a+b.montant,0);

    let commPropPeriode = (ventesPropPeriode * pctProprietaire) / 100;
    let commPropTotal = (ventesPropTotal * pctProprietaire) / 100;

    let tableProp = `<table>
        <tr><th>Période</th><th>Ventes Période</th><th>Commission du Propriétaire</th></tr>
        <tr><td>${debut} au ${fin}</td><td>${ventesPropPeriode} FCFA</td><td class="montant-prop">${commPropPeriode} FCFA</td></tr>
        <tr style="background:#fff3e0; font-weight:bold;"><td>DEPUIS LE DEBUT</td><td>${ventesPropTotal} FCFA</td><td class="montant-prop">${commPropTotal} FCFA</td></tr>
    </table>`;
    document.getElementById('tableCommissionProp').innerHTML = tableProp;
}

chargerClients();
chargerVentesJour();
document.getElementById('dateVente').value = new Date().toISOString().split('T')[0];
</script>
</body>
</html>
