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
            <p class="text-xs uppercase tracking-widest opacity-60 font-bold mb-1">Total Portfolio Value</p>
            <h2 id="totalValue" class="text-5xl font-bold">$0.00</h2>
        </div>

        <div id="stockList" class="space-y-4 mb-40">
            <div id="statusMessage" class="bg-white p-8 rounded-2xl text-center border-2 border-dashed border-slate-200">
                <p class="text-slate-400">Upload your Stock Events CSV to begin...</p>
            </div>
        </div>

        <div class="fixed bottom-6 left-1/2 -translate-x-1/2 w-[calc(100%-2rem)] max-w-md bg-white p-4 rounded-3xl border border-slate-200 shadow-2xl">
            <input type="file" id="csvFile" accept=".csv" class="mb-3 block w-full text-xs text-slate-500 file:mr-4 file:py-2 file:px-4 file:rounded-full file:border-0 file:text-xs file:font-bold file:bg-slate-100 file:text-slate-700"/>
            <button onclick="processTransactions()" class="w-full bg-blue-600 text-white py-4 rounded-2xl font-black text-lg active:scale-95 transition-all">
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
            const fileInput = document.getElementById('csvFile');
            const file = fileInput.files[0];

            // 1. Check if file is selected
            if (!file) {
                alert("Standard Warning: No file selected. Please choose a CSV file.");
                return;
            }

            // 2. Check for correct file type
            if (!file.name.toLowerCase().endsWith('.csv')) {
                alert("Invalid File Type: Please upload a file ending in .csv");
                return;
            }

            const reader = new FileReader();
            reader.onload = function(e) {
                const content = e.target.result;
                const rows = content.split(/\r?\n/).filter(row => row.trim() !== "");
                
                if (rows.length < 2) {
                    alert("Invalid Data: The file appears to be empty or missing data rows.");
                    return;
                }

                const headers = rows[0].split(',').map(h => h.replace(/["]/g, '').trim());

                // 3. Check for specific columns based on your image
                const symbolIdx = headers.indexOf("Symbol");
                const qtyIdx = headers.indexOf("Quantity");

                if (symbolIdx === -1 || qtyIdx === -1) {
                    alert("Column Error: Could not find required 'Symbol' or 'Quantity' columns. Please ensure you are using the correct Stock Events export.");
                    console.error("Found headers:", headers);
                    return;
                }

                const portfolio = {};
                let validRows = 0;

                for (let i = 1; i < rows.length; i++) {
                    const cols = rows[i].split(',');
                    if (cols.length <= Math.max(symbolIdx, qtyIdx)) continue;

                    const ticker = cols[symbolIdx].replace(/["]/g, '').trim();
                    const qty = parseFloat(cols[qtyIdx].replace(/["]/g, '').trim()) || 0;

                    if (ticker) {
                        portfolio[ticker] = (portfolio[ticker] || 0) + qty;
                        validRows++;
                    }
                }

                if (validRows === 0) {
                    alert("Process Error: We found the columns, but no valid stock data was processed.");
                    return;
                }

                renderPortfolio(portfolio);
            };

            reader.onerror = function() {
                alert("System Error: Failed to read the file. Try saving the file again or using a different browser.");
            };

            reader.readAsText(file);
        }

        function renderPortfolio(portfolio) {
            const listContainer = document.getElementById('stockList');
            listContainer.innerHTML = '';
            let totalValue = 0;

            for (const ticker in portfolio) {
                const qty = portfolio[ticker];
                if (qty <= 0) continue;

                const mockPrice = 150; 
                const value = qty * mockPrice;
                totalValue += value;

                listContainer.innerHTML += `
                    <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-100 flex justify-between items-center">
                        <div>
                            <p class="font-black text-slate-800 text-lg">${ticker}</p>
                            <p class="text-xs font-bold text-slate-400 uppercase">${qty.toLocaleString()} Shares</p>
                        </div>
                        <div class="text-right">
                            <p class="font-black text-slate-900">$${value.toLocaleString()}</p>
                            <p class="text-[10px] text-blue-500 font-bold uppercase tracking-widest">Est. Market Value</p>
                        </div>
                    </div>`;
            }

            document.getElementById('totalValue').innerText = "$" + totalValue.toLocaleString();
            updateTime();
        }
    </script>
</body>
</html>
