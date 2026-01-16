<div class="mt-10 border-t border-gray-300 pt-6">
        <h3 class="text-lg font-bold mb-2">Sync from Stock Events</h3>
        <p class="text-xs text-gray-500 mb-4">1. Select file. 2. Tap Process.</p>
        
        <input type="file" id="csvFile" accept=".csv" 
               class="block w-full text-sm text-gray-500 mb-4
               file:mr-4 file:py-3 file:px-4
               file:rounded-full file:border-0
               file:text-sm file:font-semibold
               file:bg-blue-50 file:text-blue-700"/>

        <button onclick="processUploadedFile()" 
                class="w-full bg-blue-600 text-white py-4 rounded-xl font-bold shadow-lg active:bg-blue-700">
            Process Portfolio Data
        </button>
    </div>

    <script>
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
                listContainer.innerHTML = ''; 
                
                let totalPortfolioValue = 0;

                for (let i = 1; i < rows.length; i++) {
                    const cols = rows[i].split(',');
                    if (cols.length < 2) continue;

                    const ticker = cols[0].replace(/"/g, '').trim();
                    const qty = parseFloat(cols[2]) || 0;
                    const mockPrice = 150.00; 
                    const stockValue = qty * mockPrice;
                    totalPortfolioValue += stockValue;

                    listContainer.innerHTML += `
                        <div class="card p-4 flex justify-between items-center">
                            <div>
                                <h3 class="font-bold text-lg">${ticker}</h3>
                                <p class="text-xs text-gray-400">${qty} Shares</p>
                            </div>
                            <div class="text-right">
                                <p class="font-bold">$${mockPrice.toFixed(2)}</p>
                                <p class="text-xs text-blue-500">Live API Pending</p>
                            </div>
                        </div>`;
                }

                document.getElementById('totalValue').innerText = "$" + totalPortfolioValue.toLocaleString();
                updateTime();
                alert("Portfolio Updated at " + document.getElementById('time').innerText);
            };
            reader.readAsText(file);
        }
    </script>
