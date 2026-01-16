<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Stock Tracker Pro</title>
    <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-slate-50 p-4 font-sans text-slate-900">

    <div class="max-w-md mx-auto">
        <header class="mb-6">
            <h1 class="text-3xl font-black">Portfolio</h1>
            <p class="text-slate-500 font-medium">Sync Time: <span id="time">00:00</span></p>
        </header>

        <div class="bg-slate-900 text-white p-8 rounded-3xl mb-8 shadow-2xl">
            <p class="text-xs uppercase tracking-widest opacity-60 font-bold mb-1">Total Value</p>
            <h2 id="totalValue" class="text-5xl font-bold">$0.00</h2>
        </div>

        <div id="stockList" class="space-y-4 mb-40">
            <div class="bg-white p-8 rounded-2xl text-center border-2 border-dashed border-slate-200">
                <p class="text-slate-400">Waiting for CSV...</p>
            </div>
        </div>

        <div class="fixed bottom-6 left-1/2 -translate-x-1/2 w-[calc(100%-2rem)] max-w-md bg-white p-4 rounded-3xl border border-slate-200 shadow-2xl">
            <input type="file" id="csvFile" accept=".csv" class="mb-3 block w-full text-xs text-slate-500"/>
            <button onclick="processTransactions()" class="w-full bg-blue-600 text-white py-4 rounded-2xl font-black text-lg">
                Calculate Portfolio
            </button>
        </div>
    </div>

    <script>
        function updateTime() {
            const now = new Date();
            const timeStr = now.getHours().toString().padStart(2, '0') + ":" + now.getMinutes().toString().padStart(2, '0');
            document.getElementById('time').innerText = timeStr;
        }

        function processTransactions() {
            try {
                const fileInput = document.getElementById('csvFile');
                const file = fileInput.files[0];
                
                if (!file) {
                    alert("STEP 1: No file selected. Please click 'Choose File' first.");
                    return;
                }

                const reader = new FileReader();
                
                reader.onerror = function() {
                    alert("ERROR: Could not read the file from your computer.");
                };

                reader.onload = function(e) {
                    const text = e.target.result;
                    if (!text) { alert("ERROR: The file appears to be empty."); return; }
                    
                    const rows = text.split(/\r?\n/).filter(row => row.trim() !== "");
                    const firstRow = rows[0].split(',');
                    
                    alert("STEP 2: File loaded. Columns found: " + firstRow.join(" | "));

                    const headers = firstRow.map(h => h.replace(/["]/g, '').trim().toLowerCase());
                    const symbolIdx = headers.findIndex(h => h.includes('symbol') || h.includes('ticker'));
                    const qtyIdx = headers.findIndex(h => h.includes('shares') || h.includes('quantity') || h.includes('amount'));
                    
                    if (symbolIdx === -1 || qtyIdx === -1) {
                        alert("STEP 3 ERROR: Could not find 'Symbol' or 'Shares' columns. Check your CSV header names.");
                        return;
                    }

                    const portfolio = {};
                    for (let i = 1; i < rows.length; i++) {
                        const cols = rows[i].split(',');
                        const ticker = cols[symbolIdx]?.replace(/["]/g, '').trim();
                        const qty = parseFloat(cols[qtyIdx]?.replace(/["]/g, '').trim()) || 0;
