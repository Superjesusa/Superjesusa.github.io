<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Stock Tracker Pro</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
</head>
<body class="bg-slate-50 p-4 font-sans text-slate-900">

    <div class="max-w-md mx-auto pb-20">
        <header class="mb-6 flex justify-between items-center">
            <h1 class="text-3xl font-black">Performance</h1>
            <span class="text-slate-500 font-bold" id="time">00:00</span>
        </header>

        <div class="bg-white p-6 rounded-3xl shadow-xl mb-6 border border-slate-100">
            <div class="flex justify-between items-start mb-4">
                <div>
                    <p class="text-xs uppercase tracking-widest text-slate-400 font-bold">Total Gain/Loss</p>
                    <h2 id="totalGain" class="text-3xl font-black text-green-500">+$0.00</h2>
                </div>
                <div class="bg-green-100 text-green-700 px-2 py-1 rounded-lg text-xs font-bold">+0.0%</div>
            </div>
            
            <div class="h-48 w-full">
                <canvas id="performanceChart"></canvas>
            </div>

            <div class="flex justify-between mt-6 bg-slate-100 p-1 rounded-2xl">
                <button onclick="updateChart('1M')" class="flex-1 py-2 text-xs font-bold rounded-xl bg-white shadow-sm">1M</button>
                <button onclick="updateChart('3M')" class="flex-1 py-2 text-xs font-bold rounded-xl">3M</button>
                <button onclick="updateChart('1Y')" class="flex-1 py-2 text-xs font-bold rounded-xl">1Y</button>
                <button onclick="updateChart('YTD')" class="flex-1 py-2 text-xs font-bold rounded-xl">YTD</button>
            </div>
        </div>

        <div id="stockList" class="space-y-4 mb-10"></div>

        <div class="bg-white p-6 rounded-3xl border border-slate-200 shadow-lg">
            <h3 class="font-bold mb-4 flex items-center gap-2">
                <span class="bg-blue-600 w-2 h-6 rounded-full"></span>
                Add Investment
            </h3>
            <div class="space-y-3">
                <input type="text" id="manualTicker" placeholder="Ticker (e.g. TSLA)" class="w-full p-4 bg-slate-50 rounded-2xl text-sm font-bold uppercase border-none focus:ring-2 focus:ring-blue-500">
                <div class="flex gap-2">
                    <input type="number" id="manualQty" placeholder="Shares" class="w-1/2 p-4 bg-slate-50 rounded-2xl text-sm font-bold border-none">
                    <input type="number" id="manualCost" placeholder="Avg Cost" class="w-1/2 p-4 bg-slate-50 rounded-2xl text-sm font-bold border-none">
                </div>
                <button onclick="addStock()" class="w-full bg-slate-900 text-white py-4 rounded-2xl font-black text-lg active:scale-95 transition-all">
                    Update Portfolio
                </button>
            </div>
            <button onclick="clearAll()" class="w-full mt-4 text-slate-300 text-[10px] uppercase font-bold tracking-widest">Wipe Data</button>
        </div>
    </div>

    <script>
        let myChart;

        function updateTime() {
            const now = new Date();
            document.getElementById('time').innerText = now.getHours().toString().padStart(2, '0') + ":" + now.getMinutes().toString().padStart(2, '0');
        }

        window.onload = function() {
            updateTime();
            initChart();
            renderPortfolio();
        };

        function addStock() {
            const ticker = document.getElementById('manualTicker').value.toUpperCase().trim();
            const qty = parseFloat(document.getElementById('manualQty').value);
            const cost = parseFloat(document.getElementById('manualCost').value);

            if (!ticker || isNaN(qty) || isNaN(cost)) {
                alert("Please fill in Ticker, Quantity, and Average Cost.");
                return;
            }

            let portfolio = JSON.parse(localStorage.getItem('visualPortfolio') || '{}');
            portfolio[ticker] = { qty, cost };
            localStorage.setItem('visualPortfolio', JSON.stringify(portfolio));
            
            document.getElementById('manualTicker').value = '';
            document.getElementById('manualQty').value = '';
            document.getElementById('manualCost').value = '';
            renderPortfolio();
        }

        function renderPortfolio() {
            const portfolio = JSON.parse(localStorage.getItem('visualPortfolio') || '{}');
            const listContainer = document.getElementById('stockList');
            listContainer.innerHTML = '';
            let totalGain = 0;

            for (const ticker in portfolio) {
                const data = portfolio[ticker];
                const currentPrice = 180; // Placeholder for Live API
                const gain = (currentPrice - data.cost) * data.qty;
                totalGain += gain;

                listContainer.innerHTML += `
                    <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-100 flex justify-between items-center">
                        <div>
                            <p class="font-black text-slate-800">${ticker}</p>
                            <p class="text-[10px] font-bold text-slate-400 uppercase">Avg Cost: $${data.cost}</p>
                        </div>
                        <div class="text-right">
                            <p class="font-bold ${gain >= 0 ? 'text-green-500' : 'text-red-500'}">
                                ${gain >= 0 ? '+' : ''}$${gain.toLocaleString()}
                            </p>
                            <p class="text-[10px] font-bold text-slate-400">${data.qty} SHARES</p>
                        </div>
                    </div>`;
            }

            document.getElementById('totalGain').innerText = (totalGain >= 0 ? '+$' : '-$') + Math.abs(totalGain).toLocaleString();
            document.getElementById('totalGain').className = `text-3xl font-black ${totalGain >= 0 ? 'text-green-500' : 'text-red-500'}`;
        }

        function initChart() {
            const ctx = document.getElementById('performanceChart').getContext('2d');
            myChart = new Chart(ctx, {
                type: 'line',
                data: {
                    labels: ['Week 1', 'Week 2', 'Week 3', 'Week 4'],
                    datasets: [{
                        label: 'Portfolio Value',
                        data: [1200, 1350, 1280, 1450], // Mock history
                        borderColor: '#2563eb',
                        borderWidth: 3,
                        tension: 0.4,
                        pointRadius: 0,
                        fill: true,
                        backgroundColor: (context) => {
                            const gradient = ctx.createLinearGradient(0, 0, 0, 200);
                            gradient.addColorStop(0, 'rgba(37, 99, 235, 0.2)');
                            gradient.addColorStop(1, 'rgba(37, 99, 235, 0)');
                            return gradient;
                        }
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: { legend: { display: false } },
                    scales: {
                        x: { display: false },
                        y: { display: false }
                    }
                }
            });
        }

        function updateChart(timeframe) {
            // This is where we will hook up the API for historical data
            alert("Switching to " + timeframe + " view. Hook up an API key to see real historical data!");
        }

        function clearAll() {
            if (confirm("Reset everything?")) {
                localStorage.removeItem('visualPortfolio');
                location.reload();
            }
        }
    </script>
</body>
</html>
