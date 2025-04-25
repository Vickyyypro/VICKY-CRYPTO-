# VICKY-CRYPTO-
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>GMMA Crypto Scanner with Guppy Indicator</title>
  <style>
    body {
      background-color: #121212;
      color: #ffffff;
      font-family: Arial, sans-serif;
      margin: 20px;
    }
    h2 {
      color: #00ffcc;
    }
    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 10px;
    }
    th, td {
      border: 1px solid #444;
      padding: 10px;
      text-align: left;
      cursor: pointer;
    }
    th {
      background-color: #1e1e1e;
    }
    tr:hover {
      background-color: #1e1e1e;
    }
    .status {
      margin-bottom: 20px;
      background-color: #222;
      padding: 10px;
      border-left: 4px solid #00ffcc;
    }
    .run-btn {
      background-color: #00ffcc;
      color: #000;
      padding: 10px 20px;
      border: none;
      cursor: pointer;
      font-weight: bold;
      border-radius: 4px;
      margin-top: 10px;
    }
    .tabs {
      margin-bottom: 20px;
    }
    .tab-btn {
      background-color: #1e1e1e;
      color: #ccc;
      padding: 10px 15px;
      margin-right: 10px;
      border: none;
      border-radius: 5px;
      cursor: pointer;
    }
    .tab-btn.active {
      background-color: #00ffcc;
      color: #000;
      font-weight: bold;
    }
    .section {
      display: none;
    }
    .section.active {
      display: block;
    }
    .timeframe-tabs {
      display: flex;
      margin-bottom: 10px;
      flex-wrap: wrap;
    }
    .timeframe-btn {
      background-color: #333;
      color: #ccc;
      padding: 5px 10px;
      margin-right: 5px;
      margin-bottom: 5px;
      border: none;
      border-radius: 3px;
      cursor: pointer;
      font-size: 12px;
    }
    .timeframe-btn.active {
      background-color: #00ffcc;
      color: #000;
    }
    .signal-strong {
      color: #00ffcc;
      font-weight: bold;
    }
    .signal-watch {
      color: #ffff00;
    }
    #tv_chart_container {
      height: 500px;
    }
    .target-progress {
      width: 100%;
      background-color: #333;
      height: 10px;
      border-radius: 5px;
      margin-top: 5px;
    }
    .target-progress-bar {
      height: 100%;
      background-color: #00ffcc;
      border-radius: 5px;
      width: 0%;
    }
    .target-price {
      color: #00ffcc;
      font-weight: bold;
    }
    .stop-loss {
      color: #ff6666;
      font-weight: bold;
    }
  </style>
</head>
<body>

  <h2>GMMA Crypto Scanner with Guppy Indicator</h2>

  <!-- TradingView Widget -->
  <div class="tradingview-widget-container" style="margin-bottom: 20px;">
    <div class="timeframe-tabs">
      <button class="timeframe-btn" onclick="changeTimeframe('1')">1m</button>
      <button class="timeframe-btn" onclick="changeTimeframe('5')">5m</button>
      <button class="timeframe-btn active" onclick="changeTimeframe('15')">15m</button>
      <button class="timeframe-btn" onclick="changeTimeframe('60')">1h</button>
      <button class="timeframe-btn" onclick="changeTimeframe('240')">4h</button>
      <button class="timeframe-btn" onclick="changeTimeframe('1D')">1D</button>
    </div>
    <div id="tv_chart_container"></div>
  </div>

  <div class="status">
    <strong>Status:</strong> <span id="status">Offline</span><br>
    <strong>API Requests Sent:</strong> <span id="requestCount">0</span><br>
    <strong>Last Scan:</strong> <span id="lastScanTime">Never</span><br><br>
    <button class="run-btn" onclick="runScanner()">Run Scanner Now</button>
  </div>

  <div class="tabs">
    <button class="tab-btn active" onclick="showTab('scanned')">Scanned Pairs</button>
    <button class="tab-btn" onclick="showTab('triggered')">Triggered Setups</button>
    <button class="tab-btn" onclick="showTab('waiting')">Waiting Pairs</button>
  </div>

  <div id="scanned" class="section active">
    <h2>Scanned Pairs (Top 100 by Volume)</h2>
    <table>
      <thead>
        <tr>
          <th>Symbol</th>
          <th>Price (USD)</th>
          <th>24h Change</th>
          <th>Volume</th>
        </tr>
      </thead>
      <tbody id="scannedTable"></tbody>
    </table>
  </div>

  <div id="triggered" class="section">
    <h2>Triggered Setups</h2>
    <table>
      <thead>
        <tr>
          <th>Symbol</th>
          <th>Price (USD)</th>
          <th>Entry</th>
          <th>Stop Loss</th>
          <th>Target (1:3)</th>
          <th>Progress</th>
          <th>Timeframe</th>
        </tr>
      </thead>
      <tbody id="triggeredTable"></tbody>
    </table>
  </div>

  <div id="waiting" class="section">
    <h2>Waiting Pairs</h2>
    <table>
      <thead>
        <tr>
          <th>Symbol</th>
          <th>Price (USD)</th>
          <th>Signal Status</th>
        </tr>
      </thead>
      <tbody id="waitingTable"></tbody>
    </table>
  </div>

  <script src="https://s3.tradingview.com/tv.js"></script>
  <script>
    let requestCount = 0;
    let tvWidget = null;
    let currentTriggeredPairs = [];
    let currentWaitingPairs = [];
    let currentSymbol = "BTCUSDT";
    let currentTimeframe = "15";
    let allPairsData = [];

    // Initialize TradingView widget
    function initChart() {
      if (tvWidget !== null) {
        tvWidget.remove();
      }
      
      tvWidget = new TradingView.widget({
        "autosize": true,
        "symbol": `BINANCE:${currentSymbol}`,
        "interval": currentTimeframe,
        "timezone": "Etc/UTC",
        "theme": "dark",
        "style": "1",
        "locale": "en",
        "toolbar_bg": "#121212",
        "enable_publishing": false,
        "hide_top_toolbar": false,
        "hide_side_toolbar": false,
        "allow_symbol_change": true,
        "container_id": "tv_chart_container",
        "studies": getGuppyStudies(),
        "overrides": {
          "paneProperties.background": "#121212",
          "paneProperties.vertGridProperties.color": "#333",
          "paneProperties.horzGridProperties.color": "#333",
          "symbolWatermarkProperties.transparency": 90,
          "scalesProperties.textColor": "#AAA",
          "mainSeriesProperties.candleStyle.wickUpColor": "#00ffcc",
          "mainSeriesProperties.candleStyle.wickDownColor": "#ff6666"
        }
      });
    }

    // Get Guppy MMA studies configuration
    function getGuppyStudies() {
      return [
        // Short-term EMAs (3,5,8,10,12,15)
        { id: "MASimple@tv-basicstudies", inputs: { length: 3, color: "#2962FF" } },
        { id: "MASimple@tv-basicstudies", inputs: { length: 5, color: "#2962FF" } },
        { id: "MASimple@tv-basicstudies", inputs: { length: 8, color: "#2962FF" } },
        { id: "MASimple@tv-basicstudies", inputs: { length: 10, color: "#2962FF" } },
        { id: "MASimple@tv-basicstudies", inputs: { length: 12, color: "#2962FF" } },
        { id: "MASimple@tv-basicstudies", inputs: { length: 15, color: "#2962FF" } },
        // Long-term EMAs (30,35,40,45,50,60)
        { id: "MASimple@tv-basicstudies", inputs: { length: 30, color: "#FF6D00" } },
        { id: "MASimple@tv-basicstudies", inputs: { length: 35, color: "#FF6D00" } },
        { id: "MASimple@tv-basicstudies", inputs: { length: 40, color: "#FF6D00" } },
        { id: "MASimple@tv-basicstudies", inputs: { length: 45, color: "#FF6D00" } },
        { id: "MASimple@tv-basicstudies", inputs: { length: 50, color: "#FF6D00" } },
        { id: "MASimple@tv-basicstudies", inputs: { length: 60, color: "#FF6D00" } }
      ];
    }

    function changeTimeframe(timeframe) {
      currentTimeframe = timeframe;
      updateActiveTimeframeButton();
      initChart();
    }

    function updateActiveTimeframeButton() {
      document.querySelectorAll('.timeframe-btn').forEach(btn => {
        btn.classList.remove('active');
      });
      const activeBtn = document.querySelector(`.timeframe-btn[onclick="changeTimeframe('${currentTimeframe}')"]`);
      if (activeBtn) activeBtn.classList.add('active');
    }

    async function fetchUSDTData() {
      try {
        document.getElementById('status').textContent = 'Fetching data...';
        const response = await fetch('https://api.binance.com/api/v3/ticker/24hr');
        const data = await response.json();
        return data.filter(item => item.symbol.endsWith('USDT'));
      } catch (error) {
        console.error("Error fetching data:", error);
        document.getElementById('status').textContent = 'Fetch error';
        return [];
      }
    }

    function getTopUSDTByVolume(pairs, limit = 100) {
      return pairs.sort((a, b) => parseFloat(b.quoteVolume) - parseFloat(a.quoteVolume)).slice(0, limit);
    }

    function evaluateGMMA(pair) {
      // Simulate GMMA evaluation
      const priceChange = parseFloat(pair.priceChangePercent);
      const volume = parseFloat(pair.quoteVolume);
      const currentPrice = parseFloat(pair.lastPrice);
      
      if (priceChange > 5 && volume > 1000000) {
        // Calculate 1:3 risk-reward targets
        const entryPrice = currentPrice;
        const stopLoss = entryPrice * 0.97; // 3% stop loss
        const targetPrice = entryPrice + (3 * (entryPrice - stopLoss)); // 1:3 reward
        
        return {
          status: 'triggered',
          timeframe: ['15','60','240'][Math.floor(Math.random()*3)],
          entryPrice: entryPrice.toFixed(4),
          stopLoss: stopLoss.toFixed(4),
          targetPrice: targetPrice.toFixed(4),
          currentPrice: currentPrice.toFixed(4)
        };
      } else if (priceChange > 2 && volume > 500000) {
        return {
          status: 'waiting'
        };
      }
      return null;
    }

    function createClickableRow(symbol, price, extra = '') {
      const tr = document.createElement('tr');
      tr.innerHTML = `<td>${symbol}</td><td>$${price}</td>${extra}`;
      tr.onclick = () => {
        currentSymbol = symbol.replace('/USDT', 'USDT');
        initChart();
      };
      return tr;
    }

    function formatNumber(num, decimals = 2) {
      return parseFloat(num).toFixed(decimals).replace(/\B(?=(\d{3})+(?!\d))/g, ",");
    }

    function calculateProgress(currentPrice, entryPrice, targetPrice) {
      if (currentPrice <= entryPrice) return 0;
      if (currentPrice >= targetPrice) return 100;
      return ((currentPrice - entryPrice) / (targetPrice - entryPrice)) * 100;
    }

    function updateTriggeredTable() {
      const triggeredTable = document.getElementById('triggeredTable');
      triggeredTable.innerHTML = '';
      
      currentTriggeredPairs.forEach(pair => {
        const currentPrice = parseFloat(pair.currentPrice);
        const entryPrice = parseFloat(pair.entryPrice);
        const targetPrice = parseFloat(pair.targetPrice);
        const progress = calculateProgress(currentPrice, entryPrice, targetPrice);
        
        triggeredTable.appendChild(createClickableRow(
          pair.symbol, 
          pair.currentPrice, 
          `<td>$${pair.entryPrice}</td>
           <td class="stop-loss">$${pair.stopLoss}</td>
           <td class="target-price">$${pair.targetPrice}</td>
           <td>
             <div class="target-progress">
               <div class="target-progress-bar" style="width: ${progress}%"></div>
             </div>
             ${progress.toFixed(1)}%
           </td>
           <td>${pair.timeframe}m</td>`
        ));
      });
    }

    async function updateScanner(pairs) {
      allPairsData = pairs;
      const topPairs = getTopUSDTByVolume(pairs, 100);
      const scannedTable = document.getElementById('scannedTable');
      const waitingTable = document.getElementById('waitingTable');

      scannedTable.innerHTML = '';
      waitingTable.innerHTML = '';
      
      currentTriggeredPairs = [];
      currentWaitingPairs = [];

      topPairs.forEach(pair => {
        const symbol = pair.symbol.replace('USDT', '/USDT');
        const price = parseFloat(pair.lastPrice).toFixed(4);
        const priceChange = parseFloat(pair.priceChangePercent).toFixed(2);
        const volume = formatNumber(pair.quoteVolume);
        const priceChangeColor = priceChange >= 0 ? '#00ffcc' : '#ff6666';
        const priceChangeDisplay = `${priceChange >= 0 ? '+' : ''}${priceChange}%`;

        // Scanned pairs table
        scannedTable.appendChild(createClickableRow(
          symbol, 
          price, 
          `<td style="color:${priceChangeColor}">${priceChangeDisplay}</td><td>$${volume}</td>`
        ));

        const gmmaSignal = evaluateGMMA(pair);
        if (gmmaSignal?.status === 'triggered') {
          currentTriggeredPairs.push({
            symbol, 
            tvSymbol: pair.symbol, 
            timeframe: gmmaSignal.timeframe,
            entryPrice: gmmaSignal.entryPrice,
            stopLoss: gmmaSignal.stopLoss,
            targetPrice: gmmaSignal.targetPrice,
            currentPrice: gmmaSignal.currentPrice
          });
        } else if (gmmaSignal?.status === 'waiting') {
          currentWaitingPairs.push({symbol, tvSymbol: pair.symbol});
          waitingTable.appendChild(createClickableRow(
            symbol, 
            price, 
            `<td><span class="signal-watch">Watching</span></td>`
          ));
        }
      });

      // Update the triggered table with progress bars
      updateTriggeredTable();

      document.getElementById('lastScanTime').textContent = new Date().toLocaleString();
      document.getElementById('status').textContent = 'Online';
    }

    async function runScanner() {
      document.getElementById('status').textContent = 'Scanning...';
      incrementRequest();
      try {
        const usdtPairs = await fetchUSDTData();
        await updateScanner(usdtPairs);
        
        // Update prices every 5 seconds for the triggered pairs
        setInterval(async () => {
          if (currentTriggeredPairs.length > 0) {
            const response = await fetch('https://api.binance.com/api/v3/ticker/24hr');
            const data = await response.json();
            
            currentTriggeredPairs.forEach(pair => {
              const updatedPair = data.find(item => item.symbol === pair.tvSymbol);
              if (updatedPair) {
                pair.currentPrice = parseFloat(updatedPair.lastPrice).toFixed(4);
              }
            });
            
            updateTriggeredTable();
          }
        }, 5000);
      } catch (error) {
        document.getElementById('status').textContent = 'Error';
        console.error("Scanner error:", error);
      }
    }

    function incrementRequest() {
      requestCount++;
      document.getElementById('requestCount').textContent = requestCount;
    }

    function showTab(tabId) {
      document.querySelectorAll('.tab-btn').forEach(btn => btn.classList.remove('active'));
      document.querySelectorAll('.section').forEach(sec => sec.classList.remove('active'));
      document.querySelector(`[onclick="showTab('${tabId}')"]`).classList.add('active');
      document.getElementById(tabId).classList.add('active');
      
      // Load appropriate chart when switching tabs
      if (tabId === 'triggered' && currentTriggeredPairs.length > 0) {
        const firstPair = currentTriggeredPairs[0];
        currentSymbol = firstPair.tvSymbol;
        currentTimeframe = firstPair.timeframe || '15';
        updateActiveTimeframeButton();
        initChart();
      } else if (tabId === 'waiting' && currentWaitingPairs.length > 0) {
        currentSymbol = currentWaitingPairs[0].tvSymbol;
        initChart();
      } else if (tabId === 'scanned' && allPairsData.length > 0) {
        currentSymbol = allPairsData[0].symbol;
        initChart();
      }
    }

    // Initialize the page
    window.onload = function() {
      initChart();
      runScanner();
      setInterval(runScanner, 60000);
    };
  </script>
</body>
</html>
