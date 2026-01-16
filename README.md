<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>My Stock Tracker</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body { -webkit-tap-highlight-color: transparent; }
        .card { background: white; border-radius: 12px; box-shadow: 0 4px 6px rgba(0,0,0,0.1); }
    </style>
</head>
<body class="bg-gray-100 p-4 font-sans text-gray-900">

    <header class="mb-6">
        <h1 class="text-2xl font-bold">Portfolio Performance</h1>
        <p class="text-gray-500 text-sm">Last updated: <span id="time">00:00</span></p>
    </header>

    <div id="summary" class="card p-6 mb-6 bg-blue-600 text-white">
        <p class="text-sm opacity-80">Total Value</p>
        <h2 id="totalValue" class="text-3xl font-bold">$0.00</h2>
        <p class="text-sm mt-2">Upload CSV to see stats</p>
    </div>

    <div id="stockList" class="space-y-4">
        <p class="text-center text-gray-400 py-10">No stocks loaded yet.</p>
    </div>

    <div class="mt-10 border-t border-gray-300 pt-6">
        <h3 class="text-lg font-bold mb-2">Sync from Stock Events</h3>
        <p class="text-xs text-gray-500 mb-4">Select the .csv file you exported from your app:</p>
        
        <label class="block">
            <span class="sr-only">Choose file</span>
            <input type="file" id="csvFile" accept=".csv" onchange="handleFileSelect(event)" 
                class="block w-full text-sm text-gray-500
                file:mr-4 file:py-3 file:px-4
                file:rounded-full file:border-0
                file:text-sm file:font-semibold
                file:bg-blue-50 file:text-blue-700
                hover:file:bg-blue-100 cursor-pointer"/>
        </label>
    </div>

    <script>
        // Set military time (e.g., 11:55)
        function updateTime() {
            const now = new Date();
            const timeStr = now.getHours().toString().padStart(2, '0') + ":" + 
                            now.getMinutes().toString().padStart(2, '0');
            document.getElementById('time').innerText = timeStr;
        }
        updateTime();

        function handleFileSelect(event) {
            const file = event.target.files[0];
            if (!file) return;

            const reader = new FileReader();
            reader.onload = function(e) {
                parseCSV(e.target.result);
            };
            reader.readAsText(file);
        }

        function parseCSV(data) {
            const rows = data.split('\n');
            const listContainer = document.getElementById('stockList');
            listContainer.innerHTML = ''; 
            
            let totalPortfolioValue = 0;

            // Loop through rows (skip header)
            for (let i = 1; i < rows.length; i++) {
                const cols = rows[i].split(',');
                if (cols.length < 2) continue;

                // Typical Stock Events CSV structure: Ticker is usually col 0, Qty is col 2
                const ticker = cols[0].replace(/"/g, '').trim();
                const qty = parseFloat(cols[2]) || 0;
                
                // For now, we simulate a price of $150.00 until we add the real API
                const mockPrice = 150.00; 
                const stockValue = qty * mockPrice;
                totalPortfolioValue
