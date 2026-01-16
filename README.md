# Superjesusa.github.io
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
<body class="bg-gray-100 p-4 font-sans">

    <header class="mb-6">
        <h1 class="text-2xl font-bold">Portfolio Performance</h1>
        <p class="text-gray-500 text-sm">Last updated: <span id="time">14:30</span></p>
    </header>

    <div class="card p-6 mb-6 bg-blue-600 text-white">
        <p class="text-sm opacity-80">Total Value</p>
        <h2 class="text-3xl font-bold">$12,450.00</h2>
        <p class="text-sm mt-2">+2.4% Today</p>
    </div>

    <div class="space-y-4">
        <div class="card p-4 flex justify-between items-center">
            <div>
                <h3 class="font-bold">AAPL</h3>
                <p class="text-xs text-gray-400">10 Shares</p>
            </div>
            <div class="text-right">
                <p class="font-bold">$190.25</p>
                <p class="text-xs text-green-500">+1.20%</p>
            </div>
        </div>
        
        <div class="card p-4 flex justify-between items-center">
            <div>
                <h3 class="font-bold">TSLA</h3>
                <p class="text-xs text-gray-400">5 Shares</p>
            </div>
            <div class="text-right">
                <p class="font-bold">$240.50</p>
                <p class="text-xs text-red-500">-0.45%</p>
            </div>
        </div>
    </div>

    <div class="mt-10 border-t pt-6">
        <h3 class="text-lg font-bold mb-2">Sync Data</h3>
        <p class="text-xs text-gray-500 mb-4">Paste your stock list or CSV export below:</p>
        <textarea id="importBox" class="w-full h-32 p-3 rounded-lg border focus:ring-2 focus:ring-blue-500" placeholder="Ticker, Quantity, Avg Cost..."></textarea>
        <button onclick="importData()" class="mt-3 w-full bg-black text-white py-3 rounded-lg font-bold">Update Portfolio</button>
    </div>

    <script>
        // Set military time as per your preference
        function updateTime() {
            const now = new Date();
            const timeStr = now.getHours().toString().padStart(2, '0') + ":" + 
                            now.getMinutes().toString().padStart(2, '0');
            document.getElementById('time').innerText = timeStr;
        }
        updateTime();

        function importData() {
            const data = document.getElementById('importBox').value;
            alert("Data received! (In a real app, this would parse your CSV and update the list above).");
        }
    </script>
</body>
</html>
