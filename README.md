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
        .animate-pop { animation: pop 0.3s ease-out; }
        @keyframes pop {
            0% { transform: scale(0.95); opacity: 0; }
            100% { transform: scale(1); opacity: 1; }
        }
    </style>
</head>
<body class="bg-gray-100 p-4 font-sans text-gray-900">

    <header class="mb-6 flex justify-between items-end">
        <div>
            <h1 class="text-2xl font-bold text-gray-800">Portfolio</h1>
            <p class="text-gray-500 text-sm">Last Sync: <span id="time">00:00</span></p>
        </div>
        <div class="text-right">
            <span class="bg-green-100 text-green-700 text-xs font-bold px-2 py-1 rounded">LIVE</span>
        </div>
    </header>

    <div id="summary" class="card p-6 mb-6 bg-slate-900 text-white shadow-xl">
        <p class="text-sm opacity-70">Total Portfolio Value</p>
        <h2 id="totalValue" class="text-4xl font-bold tracking-tight">$0.00</h2>
        <div class="mt-4 flex items-center text-sm">
            <span class="bg-white/20 px-2 py-1 rounded mr-2">Est. Daily Move</span>
            <span class="text-green-400 font-bold">+0.00%</span>
        </div>
    </div>

    <div id="stockList" class="space-y-3">
        <div class="text-center py-10 border-2 border-dashed border-gray-300 rounded-xl">
            <p class="text-gray-400 italic">No data loaded. Upload your Stock Events CSV below.</p>
        </div>
    </div>

    <div class="mt-10 border-t border-gray-300 pt-6 pb-10">
        <h3 class="text-lg font-bold mb-1">Import Data</h3>
        <p class="text-xs text-gray-500 mb-4">Export your CSV from Stock Events and select it here.</p>
        
        <div class="space-y-3">
            <input type="file" id="csvFile" accept=".csv" 
                class="block w-full text-sm text-gray-500
                file:mr-4 file:py-3 file:px-4
                file:rounded-lg file:border-0
                file:text-sm file:font-semibold
                file:bg-blue-600 file:text-white
                hover:file:bg-blue-700 cursor-pointer"/>

            <button onclick="processUploadedFile()" 
                class="w-full bg-black text-white py-4 rounded-xl font-bold shadow-lg active:scale-95 transition-transform">
                Process Portfolio Data
            </button>
        </div>
    </div>

    <script>
        // Use military time as per user preference
        function updateTime() {
            const now = new Date();
            const timeStr = now.getHours().toString().padStart(2, '0') + ":" + 
                            now.getMinutes().toString().padStart(2, '0');
            document.getElementById('time').innerText = timeStr;
        }
        updateTime();

        function processUploadedFile() {
            const fileInput = document.getElementById('csvFile');
            const file = fileInput.files[0];
            
            if (!file) {
                alert("Please select a CSV file first!");
                return;
            }

            const reader = new FileReader();
            reader.onload = function(e) {
                const text = e.target.result;
                const rows = text.split('\n');
                const listContainer = document.getElementById('stockList');
                
                // Clear existing UI
                listContainer.innerHTML = ''; 
                
                let totalPortfolioValue = 0;
                let count = 0;

                // Loop through rows (starting at index 1 to skip header)
                for (let i = 1; i < rows.length; i++) {
                    const cols = rows[i].split(',');
                    
                    // Basic validation to ensure row isn't empty
                    if (cols.length < 2) continue;

                    // Parse data based on typical Stock Events CSV order:
                    // Symbol/Ticker (index 0), Shares (index 2)
                    const ticker = cols[0].replace(/"/g, '').trim();
                    const qty = parseFloat(cols[2]) || 0;
                    
                    if (ticker && qty > 0) {
                        const mockPrice = 150.00; // placeholder price
                        const stockValue = qty * mockPrice;
                        totalPortfolioValue += stockValue;
                        count++;

                        listContainer.innerHTML += `
                            <div class="card p-4 flex justify-between items-center animate-pop">
                                <div>
                                    <h3 class="font-black text-gray-800">${ticker}</h3>
                                    <p class="text-xs text-gray-500">${qty.toLocaleString()} Shares</p>
                                </div>
                                <div class="text-right">
                                    <p class="font-bold text-gray-900">$${(qty * mockPrice).toLocaleString()}</p>
                                    <p class="text-[10px] text-blue-600 font-bold uppercase tracking-widest">Mock Price</p>
                                </div>
                            </div>`;
                    }
                }

                if (count === 0) {
                    listContainer.innerHTML = '<p class="text-red-500 text-center">Could not read any stocks. Check your CSV format.</p>';
                } else {
                    document.getElementById('totalValue').innerText = "$" + totalPortfolioValue.toLocaleString();
                    updateTime();
                }
            };
            reader.readAsText(file);
        }
    </script>
</body>
</html>
