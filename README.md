<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>My Stock Tracker</title>
    <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-gray-100 p-4 font-sans text-gray-900">

    <header class="mb-6">
        <h1 class="text-2xl font-bold">Portfolio</h1>
        <p class="text-gray-500 text-sm">Last Sync: <span id="time">00:00</span></p>
    </header>

    <div id="summary" class="bg-slate-900 text-white p-6 rounded-xl mb-6 shadow-xl">
        <p class="text-sm opacity-70">Total Portfolio Value</p>
        <h2 id="totalValue" class="text-4xl font-bold">$0.00</h2>
    </div>

    <div id="stockList" class="space-y-3 mb-10">
        <div id="debug" class="text-center py-10 border-2 border-dashed border-gray-300 rounded-xl text-gray-400">
            No data loaded yet.
        </div>
    </div>

    <div class="fixed bottom-0 left-0 right-0 bg-white p-6 border-t border-gray-200 shadow-2xl">
        <input type="file" id="csvFile" accept=".csv" class="mb-3 block w-full text-sm text-gray-500 file:mr-4 file:py-2 file:px-4 file:rounded-full file:border-0 file:text-sm file:font-semibold file:bg-blue-50 file:text-blue-700"/>
        <button onclick="processTransactions()" class="w-full bg-blue-600 text-white py-4 rounded-xl font-bold active:scale-95 transition-transform">Calculate Portfolio</button>
    </div>

    <script>
        function updateTime() {
            const now = new Date();
            const timeStr = now.getHours().toString().padStart(2, '0') + ":" + now.getMinutes().toString().padStart(2, '0');
            document.getElementById('time').innerText = timeStr;
        }

        function processTransactions() {
            const fileInput = document.getElementById('csvFile');
            const file = fileInput.files[0];
            if (!file) { alert("Select a file first!"); return; }

            const reader = new FileReader();
            reader.onload = function(e) {
                const text = e.target.result;
                const rows = text.split('\n').map(row => row.split(','));
                const headers = rows[0].map(h => h.replace(/"/g, '').trim().toLowerCase());
                
                // Find column indexes automatically
                const symbolIdx = headers.findIndex(h => h.includes('symbol') || h.includes('ticker'));
                const qtyIdx = headers.findIndex(h => h.includes('shares') || h.includes('quantity') || h.includes('amount'));
                const typeIdx = headers.findIndex(h => h.includes('type') || h.includes('transaction'));

                if (symbolIdx === -1 || qtyIdx === -1) {
                    alert("Error: Could not find 'Symbol' or 'Quantity' columns in your CSV.");
                    return;
                }

                const portfolio = {};
                for (let i = 1; i < rows.length; i++) {
                    const cols = rows[i];
                    if (cols.length < 2) continue;

                    const ticker = cols[symbolIdx].replace(/"/g, '').trim();
                    const qty = parseFloat(cols[qtyIdx]) || 0;
                    const type = typeIdx !== -1 ? cols[typeIdx].toLowerCase() : 'buy';

                    if (!ticker) continue;
                    if (!portfolio[ticker]) portfolio[ticker] = 0;

                    if (type.includes('sell')) {
                        portfolio[ticker] -= qty;
                    } else {
                        portfolio[ticker] += qty;
                    }
                }

                const listContainer = document.getElementById('stockList');
                listContainer.innerHTML = '';
                let totalValue = 0;

                for (const ticker in portfolio) {
                    const finalQty = portfolio[ticker];
                    if (finalQty <= 0) continue;

                    const mockPrice = 150;
                    const val = finalQty * mockPrice;
                    totalValue += val;

                    listContainer.innerHTML += `
                        <div class="bg-white p-4 rounded-xl shadow-sm flex justify-between items-center border border-gray-100">
                            <div><p class="font-black">${ticker}</p><p class="text-xs text-gray-500">${finalQty.toLocaleString()} Shares</p></div>
                            <p class="font-bold">$${val.toLocaleString()}</p>
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
