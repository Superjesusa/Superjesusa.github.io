<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Global Portfolio USD</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
</head>
<body class="bg-slate-50 p-4 font-sans text-slate-900">

    <div class="max-w-md mx-auto pb-24">
        <header class="mb-6 flex justify-between items-center">
            <div>
                <h1 class="text-3xl font-black tracking-tight">Portfolio <span class="text-blue-600">USD</span></h1>
                <p class="text-[10px] font-bold text-slate-400 uppercase tracking-widest" id="dateDisplay"></p>
            </div>
            <div class="text-right">
                <span class="text-lg font-black text-slate-900" id="timeDisplay">00:00</span>
            </div>
        </header>

        <div class="bg-white p-6 rounded-[2.5rem] shadow-xl mb-8 border border-slate-100">
            <div class="flex justify-between items-start mb-2">
                <div>
                    <p class="text-[10px] uppercase tracking-widest text-slate-400 font-black mb-1">Total Balance (USD)</p>
                    <h2 id="totalBalance" class="text-4xl font-black">$0.00</h2>
                </div>
                <div id="gainBadge" class="bg-slate-100 text-slate-400 px-3 py-1 rounded-full text-xs font-black">0.0%</div>
            </div>
            
            <div class="h-40 w-full mt-4">
                <canvas id="performanceChart"></canvas>
            </div>

            <div class="flex gap-2 mt-6">
                <select id="benchmarkSelect" onchange="toggleBenchmark()" class="bg-slate-100 border-none rounded-xl text-[10px] font-black px-3 py-2 text-slate-600 focus:ring-0">
                    <option value="none">COMPARE: NONE</option>
                    <option value="SPY">COMPARE: SPY</option>
                    <option value="QQQ">COMPARE: QQQ</option>
                </select>
                <div class="flex flex-1 bg-slate-100 p-1 rounded-xl border border-slate-100">
                    <button class="flex-1 py-1 text-[10px] font-black rounded-lg bg-white shadow-sm text-blue-600">1M</button>
                    <button class="flex-1 py-1 text-[10px] font-black rounded-lg text-slate-400">1Y</button>
                </div>
            </div>
        </div>

        <div id="stockList" class="space-y-4 mb-10"></div>

        <div class="bg-slate-900 text-white p-8 rounded-[2.5rem] shadow-2xl">
            <h3 class="font-black text-xl mb-6">Add Asset</h3>
            <div class="space-y-4">
                <input type="text" id="manualTicker" placeholder="TICKER (e.g. ASML, EQNR.OL)" class="w-full p-4 bg-slate-800 rounded-2xl text-sm font-bold uppercase border-none text-white">
                <div class="flex gap-4">
                    <input type="number" id="manualQty" placeholder="QTY" class="w-1/2 p-4 bg-slate-800 rounded-2xl text-sm font-bold border-none text-white">
                    <input type="number" id="manualCost" placeholder="AVG COST" class="w-1/2 p-4 bg-slate-800 rounded-2xl text-sm font-bold border-none text-white">
                </div>
                <select id="currencySelect" class="w-full p-4 bg-slate-800 rounded-2xl text-sm font-bold border-none text-white uppercase">
                    <option value="USD">Purchased in USD</option>
                    <option value="EUR">Purchased in EUR</option>
                    <option value="NOK">Purchased in NOK</option>
                </select>
                <button onclick="addStock()" class="w-full bg-blue-600 py-5 rounded-2xl font-black text-lg active:scale-95 transition-all">Save to Portfolio</button>
            </div>
        </div>
    </div>

    <script>
        let myChart;
        let rates = { NOK: 0.095, EUR: 1.09, USD: 1.0 }; // Fallback rates

        async function updateDisplay() {
            const now = new Date();
            document.getElementById('timeDisplay').innerText = now.getHours().toString().padStart(2, '0') + ":" + now.getMinutes().toString().padStart(2, '0');
            document.getElementById('dateDisplay').innerText = now.toLocaleDateString('en-US', { weekday: 'long', month: 'short', day: 'numeric' });
        }

        async function fetchRates() {
            try {
                const res = await fetch('https://open.er-api.com/v6/latest/USD');
                const data = await res.json();
                rates.NOK = 1 / data.rates.NOK;
                rates.EUR = 1 / data.rates.EUR;
            } catch(e) { console.error("Rate fetch failed, using fallback"); }
        }

        async function fetchPrice(ticker) {
            try {
                const res = await fetch(`https://api.marketdata.app/v1/stocks/quotes/${ticker}/`);
                const data = await res.json();
                if (data.last) return data.last[0];
                throw new Error();
            } catch (e) {
                const fallbacks = { 'ASML': 1220.00, 'EQNR.OL': 28.50, 'SPY': 590.20, 'QQQ': 510.40 };
                return fallbacks[ticker.toUpperCase()] || 150.00;
            }
        }

        function addStock() {
            const ticker = document.getElementById('manualTicker').value.toUpperCase().trim();
            const qty = parseFloat(document.getElementById('manualQty').value);
            const cost = parseFloat(document.getElementById('manualCost').value);
            const currency = document.getElementById('currencySelect').value;

            if (!ticker || isNaN(qty) || isNaN(cost)) return alert("Fill all fields");

            let portfolio = JSON.parse(localStorage.getItem('usdPortfolio') || '{}');
            portfolio[ticker] = { qty, cost, currency };
            localStorage.setItem('usdPortfolio', JSON.stringify(portfolio));
            location.reload();
        }

        async function renderPortfolio() {
            await fetchRates();
            const portfolio = JSON.parse(localStorage.getItem('usdPortfolio') || '{}');
            const listContainer = document.getElementById('stockList');
            listContainer.innerHTML = '';
            
            let totalValueUSD = 0;
            let totalCostUSD = 0;

            for (const ticker in portfolio) {
                const item = portfolio[ticker];
                const livePriceLocal = await fetchPrice(ticker);
                
                // Convert local price to USD if it's a known Euro/NOK stock
                let currentUSD = (ticker.includes('.OL') || item.currency === 'NOK') ? livePriceLocal * rates.NOK :
                                 (item.currency === 'EUR') ? livePriceLocal * rates.EUR : livePriceLocal;
                
                let costUSD = item.cost * (rates[item.currency] || 1);
                
                let valueUSD = currentUSD * item.qty;
                let gainUSD = (currentUSD - costUSD) * item.qty;
                
                totalValueUSD += valueUSD;
                totalCostUSD += (costUSD * item.qty);

                listContainer.innerHTML += `
                    <div class="bg-white p-5 rounded-3xl shadow-sm border border-slate-100 flex justify-between items-center">
                        <div>
                            <p class="font-black text-slate-900">${ticker}</p>
                            <p class="text-[9px] font-bold text-slate-400 uppercase">Cost: $${costUSD.toFixed(2)}</p>
                        </div>
                        <div class="text-right">
                            <p class="font-bold ${gainUSD >= 0 ? 'text-green-500' : 'text-red-500'}">$${valueUSD.toLocaleString(undefined, {maximumFractionDigits:0})}</p>
                            <p class="text-[9px] font-black ${gainUSD >= 0 ? 'text-green-400' : 'text-red-400'}">${gainUSD >= 0 ? '+' : ''}$${gainUSD.toFixed(0)}</p>
                        </div>
                    </div>`;
            }

            document.getElementById('totalBalance').innerText = "$" + totalValueUSD.toLocaleString(undefined, {minimumFractionDigits: 2});
            const pct = totalCostUSD > 0 ? ((totalValueUSD - totalCostUSD) / totalCostUSD) * 100 : 0;
            document.getElementById('gainBadge').innerText = (pct >= 0 ? '+' : '') + pct.toFixed(1) + "%";
            document.getElementById('gainBadge').className = `px-3 py-1 rounded-full text-xs font-black ${pct >= 0 ? 'bg-green-100 text-green-700' : 'bg-red-100 text-red-700'}`;
        }

        function initChart() {
            const ctx = document.getElementById('performanceChart').getContext('2d');
            myChart = new Chart(ctx, {
                type: 'line',
                data: {
                    labels: ['W1', 'W2', 'W3', 'W4', 'Now'],
                    datasets: [{
                        label: 'Portfolio',
                        data: [100, 105, 102, 110, 108],
                        borderColor: '#2563eb',
                        borderWidth: 3, pointRadius: 0, tension: 0.4, fill: true,
                        backgroundColor: 'rgba(37, 99, 235, 0.05)'
                    }]
                },
                options: {
                    responsive: true, maintainAspectRatio: false,
                    plugins: { legend: { display: false } },
                    scales: { x: { display: false }, y: { display: false } }
                }
            });
        }

        function toggleBenchmark() {
            const bench = document.getElementById('benchmarkSelect').value;
            if (myChart.data.datasets.length > 1) myChart.data.datasets.pop();
            
            if (bench !== 'none') {
                const mockBenchData = (bench === 'SPY') ? [100, 102, 103, 105, 106] : [100, 104, 101, 108, 112];
                myChart.data.datasets.push({
                    label: bench,
                    data: mockBenchData,
                    borderColor: '#cbd5e1',
                    borderDash: [5, 5],
                    borderWidth: 2, pointRadius: 0, tension: 0.4, fill: false
                });
            }
            myChart.update();
        }

        window.onload = () => { updateDisplay(); initChart(); renderPortfolio(); };
    </script>
</body>
</html>
