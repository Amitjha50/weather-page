# weather-page
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Modern Weather — Local + Forecast + AQI</title>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;700&display=swap" rel="stylesheet">
  <style>
    :root{
      --bg1:#fff7ed; --bg2:#eef2ff; --accent:#ff8a65; --muted:#64748b; --card:#ffffff;
      --shadow: 0 8px 30px rgba(2,6,23,0.08);
      --brand-grad: linear-gradient(135deg,#ffb085,#ffd89b);
    }
    *{box-sizing:border-box}
    body{font-family:Inter,system-ui,Arial;background:linear-gradient(180deg,var(--bg1),var(--bg2));min-height:100vh;margin:0;color:#102a43;display:flex;flex-direction:column}
    .wrap{max-width:1200px;margin:28px auto;padding:20px;flex:1}
    header.site{display:flex;align-items:center;justify-content:space-between;margin-bottom:18px}
    .brand{display:flex;align-items:center;gap:12px}
    .logo{width:44px;height:44px;border-radius:10px;background:var(--brand-grad);display:grid;place-items:center;font-weight:700;color:#fff;box-shadow:0 6px 18px rgba(255,138,101,0.20)}
    h1{font-size:1.05rem;margin:0}
    .search{display:flex;gap:8px;align-items:center}
    .search input{padding:10px 12px;border-radius:12px;border:1px solid rgba(16,42,67,0.06);width:260px}
    .search button{padding:10px 12px;border-radius:12px;border:none;background:var(--accent);color:#fff;cursor:pointer;box-shadow:0 6px 18px rgba(255,138,101,0.12)}
    .controls{display:flex;gap:8px;align-items:center}
    .unit-btn{padding:6px 10px;border-radius:10px;border:1px solid rgba(16,42,67,0.06);cursor:pointer;background:#fff}

    .grid{display:grid;grid-template-columns:1fr 360px;gap:18px}
    .main-card{background:var(--card);border-radius:16px;padding:18px;box-shadow:var(--shadow);border:1px solid rgba(16,42,67,0.03)}
    .top-row{display:flex;align-items:center;justify-content:space-between}
    .location{font-weight:700;font-size:1.05rem}
    .temp-large{display:flex;align-items:center;gap:16px}
    .temp-val{font-size:3.2rem;font-weight:700}
    .weather-desc{color:var(--muted)}
    .meta{margin-top:8px;color:var(--muted);font-size:0.95rem}
    .status{margin-top:10px;color:var(--muted);font-size:0.95rem}

    .hourly{margin-top:18px}
    .hourly-list{display:flex;gap:12px;overflow:auto;padding-bottom:6px}
    .hour-card{min-width:84px;background:linear-gradient(180deg,rgba(255,255,255,0.95),rgba(255,255,255,0.90));border-radius:12px;padding:10px;text-align:center;border:1px solid rgba(16,42,67,0.03)}
    .hour-card small{display:block;color:var(--muted);margin-bottom:6px}

    .side{position:sticky;top:28px}
    .panel{background:var(--card);border-radius:12px;padding:14px;box-shadow:var(--shadow);margin-bottom:14px;border:1px solid rgba(16,42,67,0.03)}
    .panel h3{margin:0 0 8px 0;font-size:0.98rem}
    .aqi-bubble{display:inline-block;padding:8px 12px;border-radius:999px;font-weight:600}

    footer.note{margin-top:12px;color:var(--muted);font-size:0.9rem}

    /* developer footer */
    .dev-footer{display:flex;align-items:center;justify-content:space-between;gap:16px;margin-top:20px;padding:14px;border-radius:12px;background:linear-gradient(180deg,rgba(255,255,255,0.85),rgba(255,255,255,0.8));box-shadow:0 8px 30px rgba(2,6,23,0.06);border:1px solid rgba(16,42,67,0.04)}
    .dev-left{display:flex;align-items:center;gap:12px}
    .dev-badge{width:56px;height:56px;border-radius:12px;background:var(--brand-grad);display:grid;place-items:center;color:#fff;font-weight:700;box-shadow:0 8px 24px rgba(255,138,101,0.14)}
    .dev-content{line-height:1}
    .dev-content .title{font-weight:700}
    .dev-content .tag{color:var(--muted);font-size:0.95rem}
    .dev-right{display:flex;align-items:center;gap:12px}
    .dev-link{padding:8px 12px;border-radius:10px;background:transparent;border:1px solid rgba(16,42,67,0.06);cursor:pointer}

    /* small pulse on badge */
    @keyframes pulse { 0% { transform:scale(1); } 60% { transform:scale(1.03); } 100% { transform:scale(1); } }
    .dev-badge:hover { animation: pulse 900ms ease-in-out; }

    /* responsive */
    @media (max-width:980px){.grid{grid-template-columns:1fr}.side{position:static}} 
    @media (max-width:520px){.search input{width:140px}.temp-val{font-size:2rem}.dev-footer{flex-direction:column;align-items:flex-start}}
  </style>
</head>
<body>
  <div class="wrap">
    <header class="site">
      <div class="brand">
        <div class="logo">W</div>
        <div>
          <h1>Modern Weather</h1>
          <div style="color:var(--muted);font-size:0.9rem">Auto location • Forecast • Air quality</div>
        </div>
      </div>

      <div style="display:flex;gap:12px;align-items:center">
        <div class="search">
          <input id="searchInput" placeholder="Search city (e.g. Delhi)" />
          <button id="searchBtn">Search</button>
        </div>
        <div class="controls">
          <button id="unitToggle" class="unit-btn">°C</button>
        </div>
      </div>
    </header>

    <div class="grid">
      <main>
        <section class="main-card" id="mainCard">
          <div class="top-row">
            <div>
              <div id="place" class="location">—</div>
              <div id="timeLocal" class="status">—</div>
            </div>
            <div class="temp-large">
              <div>
                <div class="temp-val" id="temp">—°</div>
                <div class="weather-desc" id="desc">—</div>
              </div>
              <div style="width:100px;text-align:center">
                <img id="icon" src="" alt="icon" width="80" height="80"/>
              </div>
            </div>
          </div>

          <div class="meta" id="meta">—</div>

          <div class="hourly" id="hourlySection">
            <h3 style="margin:10px 0">Hourly</h3>
            <div class="hourly-list" id="hourlyList"></div>
          </div>
        </section>

        <footer class="note">Tip: Serve this page via HTTPS or localhost for geolocation permission.</footer>
      </main>

      <aside class="side">
        <div class="panel">
          <h3>Current Conditions</h3>
          <div id="currentDetails">—</div>
          <div style="margin-top:10px;display:flex;gap:10px;align-items:center">
            <a id="mapLink" target="_blank" rel="noopener">Open in Google Maps</a>
            <button id="refreshBtn" style="margin-left:auto;padding:6px 10px;border-radius:8px;border:none;background:#f1f5f9;cursor:pointer">Refresh</button>
          </div>
        </div>

        <div class="panel">
          <h3>Air Quality (AQI)</h3>
          <div id="aqiBlock">—</div>
        </div>
      </aside>
    </div>

    <!-- Developer credit / branding -->
    <div class="dev-footer" role="contentinfo" aria-label="Developer credit">
      <div class="dev-left">
        <div class="dev-badge">NN</div>
        <div class="dev-content">
          <div class="title">Developed by NN Softwave Technology</div>
          <div class="tag">Empowering ideas into digital reality.</div>
        </div>
      </div>
      <div class="dev-right">
        <button class="dev-link" onclick="window.open('https://example.com','_blank')">Visit</button>
        <button class="dev-link" onclick="navigator.clipboard?.writeText('NN Softwave Technology')">Copy</button>
      </div>
    </div>
  </div>

  <script>
    // === CONFIG ===
    const OPENWEATHER_API_KEY = 'b0b613fb206490def6da40cf3421341c'; // <-- put your key here
    const HOURS_TO_SHOW = 24; // show next 24 hours when available
    // units state: 'metric' => Celsius; 'imperial' => Fahrenheit
    let units = 'metric';

    // DOM
    const placeEl = document.getElementById('place');
    const timeLocalEl = document.getElementById('timeLocal');
    const tempEl = document.getElementById('temp');
    const descEl = document.getElementById('desc');
    const iconEl = document.getElementById('icon');
    const metaEl = document.getElementById('meta');
    const hourlyList = document.getElementById('hourlyList');
    const currentDetails = document.getElementById('currentDetails');
    const aqiBlock = document.getElementById('aqiBlock');
    const mapLink = document.getElementById('mapLink');
    const refreshBtn = document.getElementById('refreshBtn');
    const searchBtn = document.getElementById('searchBtn');
    const searchInput = document.getElementById('searchInput');
    const unitToggle = document.getElementById('unitToggle');

    refreshBtn.addEventListener('click', () => detectAndShow(true));
    searchBtn.addEventListener('click', searchByName);
    searchInput.addEventListener('keydown', e => { if(e.key === 'Enter') searchByName(); });
    unitToggle.addEventListener('click', () => {
      units = (units === 'metric') ? 'imperial' : 'metric';
      unitToggle.textContent = units === 'metric' ? '°C' : '°F';
      if (lastCoords) safeFetchAll(lastCoords.lat, lastCoords.lon);
    });

    let lastCoords = null;

    // startup
    if (!OPENWEATHER_API_KEY || OPENWEATHER_API_KEY === 'REPLACE_WITH_YOUR_API_KEY') {
      alert('Please set your OpenWeatherMap API key in the code (OPENWEATHER_API_KEY).');
    } else {
      detectAndShow();
    }

    async function detectAndShow() {
      setStatus('Detecting location…');
      try {
        const pos = await getPositionWithTimeout(10000);
        lastCoords = { lat: pos.coords.latitude, lon: pos.coords.longitude };
        await safeFetchAll(lastCoords.lat, lastCoords.lon);
      } catch (err) {
        console.warn('Geolocation failed:', err);
        setStatus('Using IP-based location (approximate)…');
        try {
          const ipResp = await fetch('https://ipapi.co/json/');
          const ipData = await ipResp.json();
          const lat = parseFloat(ipData.latitude);
          const lon = parseFloat(ipData.longitude);
          if (!lat || !lon) throw new Error('No coords from IP');
          lastCoords = { lat, lon };
          await safeFetchAll(lat, lon);
        } catch (e) {
          console.error(e);
          setStatus('Could not determine location');
        }
      }
    }

    function getPositionWithTimeout(timeoutMs = 10000) {
      return new Promise((resolve, reject) => {
        if (!navigator.geolocation) return reject(new Error('Geolocation not supported'));
        const timer = setTimeout(() => reject(new Error('Timeout')), timeoutMs);
        navigator.geolocation.getCurrentPosition(pos => { clearTimeout(timer); resolve(pos); }, err => { clearTimeout(timer); reject(err); }, { enableHighAccuracy: true, maximumAge: 300000 });
      });
    }

    async function safeFetchAll(lat, lon) {
      try {
        await fetchAll(lat, lon);
      } catch (err) {
        console.error(err);
        setStatus('Error: ' + (err.message || err));
      }
    }

    // fetch and render everything
    async function fetchAll(lat, lon) {
      setStatus('Resolving place name…');

      // Reverse geocoding (OpenWeatherMap geocoding)
      const rgeo = await fetch(`https://api.openweathermap.org/geo/1.0/reverse?lat=${lat}&lon=${lon}&limit=1&appid=${OPENWEATHER_API_KEY}`);
      if (!rgeo.ok) throw new Error('Reverse geocoding failed');
      const rjson = await rgeo.json();
      const place = (rjson && rjson[0]) ? `${rjson[0].name || ''}${rjson[0].state?(', '+rjson[0].state):''}${rjson[0].country?(', '+rjson[0].country):''}` : 'Your location';
      placeEl.textContent = place;

      setStatus('Fetching current weather…');
      // current weather
      const curResp = await fetch(`https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&units=${units}&appid=${OPENWEATHER_API_KEY}`);
      if (!curResp.ok) {
        if (curResp.status === 401) throw new Error('Invalid API key (401)');
        throw new Error('Weather API failed: ' + curResp.status);
      }
      const cur = await curResp.json();

      tempEl.textContent = Math.round(cur.main.temp) + (units === 'metric' ? '°C' : '°F');
      descEl.textContent = capitalize(cur.weather[0].description);
      iconEl.src = `https://openweathermap.org/img/wn/${cur.weather[0].icon}@2x.png`;
      iconEl.alt = cur.weather[0].main;
      metaEl.textContent = `Feels like ${Math.round(cur.main.feels_like)}° · Humidity ${cur.main.humidity}% · Wind ${cur.wind.speed} ${units === 'metric' ? 'm/s' : 'mph'}`;
      currentDetails.innerHTML = `<div>Pressure: ${cur.main.pressure} hPa</div><div>Visibility: ${cur.visibility/1000} km</div>`;
      mapLink.href = `https://www.google.com/maps/search/?api=1&query=${lat},${lon}`;
      mapLink.textContent = `Open in Google Maps — ${lat.toFixed(4)}, ${lon.toFixed(4)}`;

      timeLocalEl.textContent = `Local time: ${new Date().toLocaleString()}`;

      // Forecast: prefer One Call (hourly) for hourly data (24 entries)
      setStatus('Fetching hourly forecast…');
      let hourlyData = null;
      try {
        const oneResp = await fetch(`https://api.openweathermap.org/data/2.5/onecall?lat=${lat}&lon=${lon}&exclude=minutely,daily,alerts&units=${units}&appid=${OPENWEATHER_API_KEY}`);
        if (oneResp.ok) {
          const oneJson = await oneResp.json();
          if (oneJson.hourly && oneJson.hourly.length) {
            hourlyData = oneJson.hourly.slice(0, HOURS_TO_SHOW); // next HOURS_TO_SHOW hours
          }
        }
      } catch (e) {
        console.warn('OneCall failed, will fallback to 3-hour forecast', e);
      }

      // Fallback: 3-hour forecast endpoint (list entries every 3 hours)
      if (!hourlyData) {
        const forResp = await fetch(`https://api.openweathermap.org/data/2.5/forecast?lat=${lat}&lon=${lon}&units=${units}&appid=${OPENWEATHER_API_KEY}`);
        if (!forResp.ok) throw new Error('Forecast API failed');
        const forJson = await forResp.json();
        const stepsNeeded = Math.max(1, Math.ceil(HOURS_TO_SHOW / 3));
        hourlyData = forJson.list.slice(0, stepsNeeded).map(item => {
          return {
            dt: item.dt,
            temp: item.main.temp,
            weather: item.weather,
            pop: item.pop
          };
        });
      }

      // Render hourlyData
      hourlyList.innerHTML = '';
      for (let entry of hourlyData) {
        const dt = new Date(entry.dt * 1000);
        const tempV = Math.round(entry.temp);
        const ico = entry.weather && entry.weather[0] ? entry.weather[0].icon : (entry.weather ? entry.weather.icon : '');
        const pop = entry.pop ? Math.round(entry.pop * 100) + '%' : '';
        hourlyList.insertAdjacentHTML('beforeend', `
          <div class="hour-card">
            <small>${dt.toLocaleString([], {hour: '2-digit', minute:'2-digit'})}</small>
            <img src="https://openweathermap.org/img/wn/${ico}@2x.png" width="48" height="48" alt=""/>
            <div style="font-weight:600">${tempV} ${units === 'metric' ? '°C' : '°F'}</div>
            <div style="color:var(--muted);font-size:0.85rem">${pop}</div>
          </div>
        `);
      }

      // Air quality (AQI)
      setStatus('Fetching air quality index…');
      try {
        const aqiResp = await fetch(`https://api.openweathermap.org/data/2.5/air_pollution?lat=${lat}&lon=${lon}&appid=${OPENWEATHER_API_KEY}`);
        if (aqiResp.ok) {
          const aqiJson = await aqiResp.json();
          const aqi = aqiJson.list && aqiJson.list[0] && aqiJson.list[0].main ? aqiJson.list[0].main.aqi : null;
          const comps = aqiJson.list && aqiJson.list[0] && aqiJson.list[0].components ? aqiJson.list[0].components : {};
          aqiBlock.innerHTML = renderAQI(aqi, comps);
        } else {
          aqiBlock.textContent = 'AQI unavailable';
        }
      } catch (e) {
        console.warn('AQI fetch failed', e);
        aqiBlock.textContent = 'AQI unavailable';
      }

      setStatus('Updated: ' + new Date().toLocaleString());
    }

    // search by city name -> use OpenWeatherMap geocoding -> fetchAll
    async function searchByName() {
      const q = searchInput.value.trim();
      if (!q) return;
      setStatus('Searching location...');
      const resp = await fetch(`https://api.openweathermap.org/geo/1.0/direct?q=${encodeURIComponent(q)}&limit=1&appid=${OPENWEATHER_API_KEY}`);
      if (!resp.ok) { setStatus('Search failed'); return; }
      const arr = await resp.json();
      if (!arr || !arr.length) { setStatus('Place not found'); return; }
      const loc = arr[0];
      lastCoords = { lat: loc.lat, lon: loc.lon };
      await safeFetchAll(loc.lat, loc.lon);
    }

    function renderAQI(aqi, comps) {
      if (!aqi) return 'No AQI data';
      const labels = ['Good','Fair','Moderate','Poor','Very Poor'];
      const colors = ['#16a34a','#f59e0b','#f97316','#ef4444','#7c2d12'];
      const idx = Math.max(0, Math.min(aqi - 1, labels.length - 1));
      return `<div style="display:flex;align-items:center;gap:12px">
        <div class="aqi-bubble" style="background:${colors[idx]};color:#fff">${aqi}</div>
        <div>
          <strong>${labels[idx]}</strong>
          <div style="color:var(--muted);font-size:0.9rem">PM₂.₅ ${comps.pm2_5 ? comps.pm2_5 + ' μg/m³' : '—'}</div>
        </div>
      </div>`;
    }

    function setStatus(txt) {
      timeLocalEl.textContent = txt || '';
      document.title = txt ? txt + ' — Modern Weather' : 'Modern Weather';
    }

    function capitalize(s) { return s && s[0] ? s[0].toUpperCase() + s.slice(1) : s }
  </script>
</body>
</html>

with the help of code, you can view the weather conditions and humidity of your area by allowing location access or by using the search bar.
