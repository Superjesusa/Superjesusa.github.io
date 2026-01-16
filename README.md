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

    <header class="mb-6 flex justify-between items-end">
        <div>
            <h1 class="text-2xl font-bold text-gray-800">Portfolio</h1>
            <p class="text-gray-500 text-sm">Last Sync: <span id="time">00:00</span></p>
        </div>
    </header>

    <div id="summary" class="card p-6 mb-6 bg-slate-900 text-white shadow-xl">
        <p class="text-sm opacity-70">Total Portfolio Value</p>
        <h2 id="totalValue" class="text-4xl font-bold tracking-tight">$0.00</h2>
    </div>

    <div id="stockList" class="space-y-3">
        <div class="text-center py-10 border-2 border-dashed border-gray-300 rounded-xl">
            <p class="text-gray-400 italic">Upload Transactions CSV below.</p>
        </div>
    </div>

    <div class="mt-10 border-t border-gray-300 pt-6 pb-10">
        <h3 class="text-lg font-bold mb-1">Import Transactions</h3>
        <p class="text-xs text-gray-500 mb-4">Exported from Stock Events</p>
        
        <div class="space-y-3">
            <input type="file" id="csvFile" accept=".csv" 
                class="block w-full text-sm text-gray-500
                file:mr-4 file:py-3 file:px-4
                file:rounded-lg file:border-0
                file:text-sm file:font-semibold
                file:bg-blue-600 file:text-white"/>

            <button onclick="processTransactions()" 
                class="w-full bg-black text-white py-4 rounded-xl font-bold shadow-lg active:scale-95 transition-transform">
                Calculate Portfolio
            </button>
        </div>
    </div>

    <script>
        function updateTime() {
            const now = new Date();
            const timeStr = now.getHours().toString().padStart(2, '0') + ":" + 
                            now.getMinutes().toString().padStart(2, '0');
            document.getElementById('time').innerText = timeStr;
        }
        updateTime();

        function processTransactions() {
            const fileInput = document.getElementById('csvFile');
            const file = fileInput.files[0];
            if (!file) return;

            const reader = new FileReader();
            reader.onload = function(e) {
                const text = e.target.result;
                const rows = text.split('\n');
                const portfolio = {}; // Use an object to group stocks

                // 1. Group transactions by Ticker
                for (let i = 1; i < rows.length; i++) {
                    const cols = rows[i].split(',');
                    if (cols.length < 5) continue;

                    // Standard Stock Events Transaction CSV:
                    // Symbol: Col 0 | Type: Col 1 | Quantity: Col 2
                    const ticker = cols[0].replace(/"/g, '').trim();
                    const type = cols[1].replace(/"/g, '').trim().toLowerCase();
                    const qty = parseFloat(cols[2]) || 0;

                    if (!portfolio[ticker]) portfolio[ticker] = 0;

                    // If it's a "Buy", add. If it's a "Sell", subtract.
                    if (type.includes('buy')) {
                        portfolio[ticker] += qty;
                    } else if (type.includes('sell')) {
                        portfolio[ticker] -= qty;
                    }
                }

                // 2. Render grouped results
                const listContainer = document.getElementById('stockList');
                listContainer.innerHTML = '';
                let totalValue = 0;

                for (const ticker in portfolio) {
                    const finalQty = portfolio[ticker];
                    if (finalQty <= 0) continue; // Skip stocks you no longer own

                    const mockPrice = 150.00;
                    const stockValue = finalQty * mockPrice;
                    totalValue += stockValue;

                    listContainer.innerHTML += `
                        <div class="card p-4 flex justify-between items-center">
                            <div>
                                <h3 class="font-black text-gray-800">${ticker}</h3>
                                <p class="text-xs text-gray-500">${finalQty.toLocaleString()} Shares</p>
                            </div>
                            <div class="text-right">
                                <p class="font-bold text-gray-900">$${stockValue.toLocaleString()}</p>
                            </div>
                        </div>`;
                }

                document.getElementById('totalValue').innerText = "$" + totalValue.toLocaleString();
                updateTime();
            };
            reader.readAsText(file);
        }
    </script>
</body>
</html>
