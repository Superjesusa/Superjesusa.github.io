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
            <h1 class="text-3xl font-black">My Portfolio</h1>
            <p class="text-slate-500 font-medium">Updated: <span id="time">00:00</span></p>
        </header>

        <div class="bg-slate-900 text-white p-8 rounded-3xl mb-8 shadow-2xl">
            <p class="text-xs uppercase tracking-widest opacity-60 font-bold mb-1">Total Value</p>
            <h2 id="totalValue" class="text-4xl font-bold tracking-tight">$0.00</h2>
        </div>

        <div id="stockList" class="space-y-4 mb-10">
            </div>

        <div class="bg-white p-6 rounded-3xl border border-slate-200 shadow-lg mb-40">
            <h3 class="font-bold mb-3">Add / Update Stock</h3>
            <div class="flex gap-2 mb-3">
                <input type="text" id="manualTicker" placeholder="Ticker (e.g. AAPL)" class="w-1/2 p-3 bg-slate-100 rounded-xl text-sm font-bold uppercase">
                <input type="number" id="manualQty" placeholder="Qty" class="w-1/2 p-3 bg-slate-100 rounded-xl text-sm font-bold">
            </div>
            <button onclick="addStock()" class="w-full bg-blue-600 text-white py-3 rounded-xl font-bold active:scale-95 transition-all">
                Save Stock
            </button>
            <button onclick="clearAll()" class="w-full mt-2 text-slate-400 text-xs py-2">Clear All Data</button>
        </div>
    </div>

    <script>
        // Use military time
        function updateTime() {
            const now = new Date();
            const timeStr = now.getHours().toString().padStart(2, '0') + ":" + 
                            now.getMinutes().toString().padStart(2, '0');
            document.getElementById('time').innerText = timeStr;
        }

        // Load data on start
        window.onload = function() {
            updateTime();
            renderPortfolio();
        };

        function addStock() {
            const ticker = document.getElementById('manualTicker').value.toUpperCase().trim();
            const qty = parseFloat(document.getElementById('manualQty').value);

            if (!ticker || isNaN(qty)) {
                alert("Please enter a valid Ticker and Quantity.");
                return;
            }

            let portfolio = JSON.parse(localStorage.getItem('myPortfolio') || '{}');
            portfolio[ticker] = qty; // Add or update
            
            localStorage.setItem('myPortfolio', JSON.stringify(portfolio));
            
            document.getElementById('manualTicker').value = '';
            document.getElementById('manualQty').value = '';
            renderPortfolio();
        }

        function renderPortfolio() {
            const portfolio = JSON.parse(localStorage.getItem('myPortfolio') || '{}');
            const listContainer = document.getElementById('stockList');
            listContainer.innerHTML = '';
            let totalValue = 0;

            for (const ticker in portfolio) {
                const qty = portfolio[ticker];
                const mockPrice = 150; 
                const value = qty * mockPrice;
                totalValue += value;

                listContainer.innerHTML += `
                    <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-100 flex justify-between items-center">
                        <div>
                            <p class="font-black text-slate-800 text-lg">${ticker}</p>
                            <p class="text-xs font-bold text-slate-400 uppercase">${qty.toLocaleString()} Shares</p>
                        </div>
                        <p class="font-black text-slate-900">$${value.toLocaleString()}</p>
                    </div>`;
            }

            document.getElementById('totalValue').innerText = "$" + totalValue.toLocaleString();
            updateTime();
        }

        function clearAll() {
            if (confirm("Delete all saved data?")) {
                localStorage.removeItem('myPortfolio');
                renderPortfolio();
            }
        }
    </script>
</body>
</html>
