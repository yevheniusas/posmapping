<!DOCTYPE html>
<html lang="uk">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Мартелоскоп Pro: Картографування</title>
    
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    
    <!-- Leaflet CSS & JS (Map) -->
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" integrity="sha256-p4NxAoJBhIIN+hmNHrzRCf9tD/miZyoHS5obTRR9BMY=" crossorigin=""/>
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js" integrity="sha256-20nQCchB9co0qIjJZRGuk2/Z9VM+kNiyxNV1lvTlZBo=" crossorigin=""></script>

    <!-- Lucide Icons -->
    <script src="https://unpkg.com/lucide@latest"></script>

    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;700&display=swap');
        body { font-family: 'Inter', sans-serif; }
        #map { height: 100%; width: 100%; border-radius: 0.5rem; z-index: 0; }
        .glass-panel {
            background: rgba(255, 255, 255, 0.98);
            border: 1px solid rgba(229, 231, 235, 0.8);
        }
        /* Custom scrollbar */
        ::-webkit-scrollbar { width: 6px; }
        ::-webkit-scrollbar-track { background: #f1f1f1; }
        ::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 3px; }
        ::-webkit-scrollbar-thumb:hover { background: #94a3b8; }
        
        .tree-marker { transition: all 0.3s ease; }
        .tree-marker:hover { transform: scale(1.2); z-index: 1000 !important; }
        
        #xyCanvas { background-color: #f8fafc; border: 1px solid #e2e8f0; border-radius: 4px; cursor: crosshair; }
    </style>
</head>
<body class="bg-slate-100 text-slate-800 h-screen flex flex-col overflow-hidden">

    <!-- Header -->
    <header class="bg-emerald-900 text-white p-3 shadow-lg shrink-0 z-20 flex justify-between items-center">
        <div class="flex items-center gap-2 px-2">
            <i data-lucide="tree-deciduous" class="w-6 h-6 text-emerald-400"></i>
            <div>
                <h1 class="text-lg font-bold leading-tight">Marteloscope Pro</h1>
                <p class="text-[10px] text-emerald-300 opacity-80">Forestry Mapping Tool</p>
            </div>
        </div>
        <button onclick="clearStorage()" class="text-xs bg-emerald-800 hover:bg-red-700 px-2 py-1 rounded transition" title="Скинути всі дані">
            Скинути проект
        </button>
    </header>

    <!-- Main Layout -->
    <div class="flex flex-grow overflow-hidden">
        
        <!-- Sidebar (Controls) -->
        <aside class="w-full md:w-[480px] bg-white shadow-xl z-10 flex flex-col overflow-hidden border-r border-slate-200">
            
            <!-- Tabs Navigation -->
            <div class="flex border-b border-slate-200 shrink-0 bg-slate-50">
                <button onclick="switchTab('centers')" id="tab-centers" class="flex-1 py-3 text-[10px] sm:text-xs font-bold uppercase tracking-wide text-emerald-700 border-b-2 border-emerald-600 bg-white">
                    1. Центри
                </button>
                <button onclick="switchTab('trees')" id="tab-trees" class="flex-1 py-3 text-[10px] sm:text-xs font-bold uppercase tracking-wide text-slate-500 hover:text-emerald-600">
                    2. Ввід
                </button>
                <button onclick="switchTab('list')" id="tab-list" class="flex-1 py-3 text-[10px] sm:text-xs font-bold uppercase tracking-wide text-slate-500 hover:text-emerald-600">
                    3. Таблиця
                </button>
                <button onclick="switchTab('xy')" id="tab-xy" class="flex-1 py-3 text-[10px] sm:text-xs font-bold uppercase tracking-wide text-slate-500 hover:text-emerald-600">
                    4. Декартова (X/Y)
                </button>
            </div>

            <!-- Tab Content: Centers -->
            <div id="content-centers" class="flex-grow flex flex-col overflow-y-auto p-4 space-y-4">
                <div class="bg-slate-50 p-4 rounded-lg border border-slate-200 shadow-sm">
                    <h3 class="text-sm font-bold text-slate-700 mb-3 flex items-center gap-2">
                        <i data-lucide="map-pin" class="w-4 h-4 text-emerald-600"></i> Додати центр ділянки
                    </h3>
                    
                    <!-- Toggle Mode -->
                    <div class="flex mb-3 bg-slate-200 p-1 rounded text-[10px]">
                        <button onclick="setCenterMode('absolute')" id="mode-absolute" class="flex-1 py-1.5 rounded shadow-sm bg-white font-bold text-slate-800 transition">Координати / GPS</button>
                        <button onclick="setCenterMode('relative')" id="mode-relative" class="flex-1 py-1.5 rounded text-slate-500 font-medium hover:text-slate-800 transition">Азимут від іншого</button>
                    </div>

                    <div class="space-y-3">
                        <input type="text" id="centerName" placeholder="Назва (напр. Сектор А)" class="w-full p-2 text-sm border rounded focus:ring-2 focus:ring-emerald-500 outline-none">
                        
                        <!-- Absolute Mode Inputs -->
                        <div id="inputs-absolute" class="space-y-2">
                            <div class="grid grid-cols-2 gap-2">
                                <input type="number" id="centerLat" step="any" placeholder="Широта (Lat)" class="p-2 text-sm border rounded">
                                <input type="number" id="centerLon" step="any" placeholder="Довгота (Lon)" class="p-2 text-sm border rounded">
                            </div>
                            <button onclick="getCurrentLocation()" class="w-full py-2 bg-slate-200 hover:bg-slate-300 text-slate-700 text-xs rounded transition flex items-center justify-center gap-1">
                                <i data-lucide="crosshair" class="w-3 h-3"></i> Взяти GPS з пристрою
                            </button>
                        </div>

                        <!-- Relative Mode Inputs -->
                        <div id="inputs-relative" class="hidden space-y-2">
                            <div>
                                <label class="block text-[10px] font-bold text-slate-400 uppercase mb-1">Базова точка</label>
                                <select id="refCenterSelect" class="w-full p-2 border border-slate-300 rounded text-sm bg-white"></select>
                            </div>
                            <div class="grid grid-cols-2 gap-2">
                                <div>
                                    <label class="block text-[10px] font-bold text-slate-400 uppercase mb-1">Азимут (°)</label>
                                    <input type="number" id="centerAz" step="any" placeholder="0-360" class="w-full p-2 text-sm border rounded">
                                </div>
                                <div>
                                    <label class="block text-[10px] font-bold text-slate-400 uppercase mb-1">Відстань (м)</label>
                                    <input type="number" id="centerDist" step="any" placeholder="Метри" class="w-full p-2 text-sm border rounded">
                                </div>
                            </div>
                        </div>

                        <button onclick="addCenter()" class="w-full py-2 bg-emerald-600 hover:bg-emerald-700 text-white text-xs font-bold rounded transition shadow-sm flex items-center justify-center gap-2 mt-2">
                            <i data-lucide="plus" class="w-4 h-4"></i> Додати центр
                        </button>
                    </div>
                </div>

                <div class="flex-grow">
                    <h3 class="text-[10px] font-bold text-slate-400 uppercase mb-2 tracking-wider">Активні центри</h3>
                    <ul id="centersList" class="space-y-2"></ul>
                </div>
            </div>

            <!-- Tab Content: Trees Input -->
            <div id="content-trees" class="hidden flex-grow flex flex-col overflow-y-auto p-4 space-y-4">
                
                <div class="bg-blue-50 p-4 rounded-lg border border-blue-100">
                    <label class="block text-[10px] font-bold text-blue-800 uppercase mb-1">Робочий центр</label>
                    <select id="activeCenterSelect" class="w-full p-2 border border-blue-300 rounded text-sm bg-white font-semibold text-blue-900">
                        <option value="">Спочатку додайте центри...</option>
                    </select>
                </div>

                <div class="bg-white p-4 rounded-lg border border-slate-200 shadow-sm flex-grow flex flex-col">
                    <div class="flex justify-between items-center mb-2">
                        <label class="block text-xs font-bold text-slate-700 uppercase">Дані дерев (CSV)</label>
                        <button onclick="insertSampleData()" class="text-[10px] text-emerald-600 hover:underline">Вставити приклад</button>
                    </div>
                    
                    <div class="mb-3 flex-grow">
                        <div class="bg-slate-100 text-[10px] p-2 rounded mb-2 text-slate-600 border border-slate-200 font-mono">
                            Формат: Номер, Азимут, Відстань, Порода, Діаметр (см), Висота (м)
                        </div>
                        <textarea id="csvInput" class="w-full h-40 p-2 text-xs font-mono border border-slate-300 rounded focus:ring-2 focus:ring-emerald-500 outline-none resize-none" placeholder="1, 45, 12.5, Сосна, 32, 24&#10;2, 120, 8.3, Дуб, 44, 21&#10;3, 270, 15.0, Береза, 28, 19"></textarea>
                    </div>

                    <div class="flex gap-2">
                        <input type="file" id="fileUpload" accept=".csv,.txt" class="hidden" onchange="handleFileUpload(this)">
                        <button onclick="document.getElementById('fileUpload').click()" class="flex-1 py-2 border border-slate-300 hover:bg-slate-50 text-slate-600 text-xs rounded flex items-center justify-center gap-1 font-medium">
                            <i data-lucide="upload" class="w-3 h-3"></i> Імпорт файлу
                        </button>
                        <button onclick="processTrees()" class="flex-1 py-2 bg-emerald-600 hover:bg-emerald-700 text-white text-xs font-bold rounded shadow-sm flex items-center justify-center gap-1">
                            <i data-lucide="calculator" class="w-3 h-3"></i> Розрахувати
                        </button>
                    </div>
                </div>
            </div>

            <!-- Tab Content: Tree List -->
            <div id="content-list" class="hidden flex-grow flex flex-col overflow-hidden">
                 <div class="p-3 bg-slate-50 border-b flex justify-between items-center">
                    <h3 class="text-xs font-bold text-slate-700 uppercase">Список дерев</h3>
                    <div class="text-xs text-emerald-700 font-bold">Всього: <span id="totalTreesCount">0</span></div>
                 </div>
                 <div class="flex-grow overflow-y-auto p-0">
                     <table class="w-full text-left text-xs">
                         <thead class="bg-slate-100 text-slate-500 sticky top-0 z-10 shadow-sm">
                             <tr>
                                 <th class="p-2 font-medium">№</th>
                                 <th class="p-2 font-medium">Lat / Lon</th>
                                 <th class="p-2 font-medium">Порода</th>
                                 <th class="p-2 font-medium">D (см)</th>
                                 <th class="p-2 font-medium">H (м)</th>
                                 <th class="p-2 font-medium text-right">Дія</th>
                             </tr>
                         </thead>
                         <tbody id="treesTableBody" class="divide-y divide-slate-100">
                             <!-- Rows -->
                         </tbody>
                     </table>
                 </div>
                 <div class="p-3 border-t bg-slate-50">
                    <button onclick="downloadAllData()" class="w-full py-2 bg-blue-600 hover:bg-blue-700 text-white text-sm font-bold rounded shadow-sm flex items-center justify-center gap-2">
                        <i data-lucide="download" class="w-4 h-4"></i> Експорт Excel (CSV)
                    </button>
                 </div>
            </div>

            <!-- Tab Content: Cartesian (X/Y) -->
            <div id="content-xy" class="hidden flex-grow flex flex-col overflow-y-auto p-4 space-y-4">
                 <div class="bg-amber-50 p-4 rounded-lg border border-amber-200">
                    <h3 class="text-sm font-bold text-amber-800 mb-2 flex items-center gap-2">
                        <i data-lucide="axis-3d" class="w-4 h-4"></i> Трансформація координат
                    </h3>
                    <p class="text-[10px] text-amber-700 mb-3">
                        Перетворення GPS (Lat/Lon) у локальну плоску систему (X/Y в метрах) з можливістю обертання для вирівнювання ділянки.
                    </p>
                    
                    <div class="space-y-4">
                        <div>
                            <div class="flex justify-between items-center mb-1">
                                <label class="text-xs font-bold text-slate-700">Кут повороту (Rotation): <span id="rotVal" class="text-emerald-600">0</span>°</label>
                            </div>
                            <input type="range" id="rotationSlider" min="-180" max="180" value="0" step="1" class="w-full h-2 bg-slate-200 rounded-lg appearance-none cursor-pointer accent-emerald-600" oninput="updateXYPreview()">
                            <div class="flex justify-between text-[10px] text-slate-400 mt-1">
                                <span>-180°</span>
                                <span>0°</span>
                                <span>+180°</span>
                            </div>
                        </div>

                        <!-- Preview Canvas -->
                        <div class="relative w-full aspect-square bg-white border rounded shadow-inner flex items-center justify-center overflow-hidden">
                             <canvas id="xyCanvas" class="w-full h-full"></canvas>
                             <div class="absolute top-2 left-2 text-[10px] text-slate-400 font-mono">Y axis ↑</div>
                             <div class="absolute bottom-2 right-2 text-[10px] text-slate-400 font-mono">X axis →</div>
                        </div>

                        <button onclick="downloadXYData()" class="w-full py-2 bg-emerald-700 hover:bg-emerald-800 text-white text-sm font-bold rounded shadow-sm flex items-center justify-center gap-2">
                            <i data-lucide="file-down" class="w-4 h-4"></i> Завантажити X/Y (.csv)
                        </button>
                    </div>
                 </div>
            </div>

            <!-- Footer -->
            <div class="p-2 bg-slate-800 text-[10px] text-center text-slate-400">
                <span id="storageStatus" class="text-emerald-400">Автозбереження активне</span> | WGS84
            </div>
        </aside>

        <!-- Map Area -->
        <main class="flex-grow relative bg-slate-200">
            <div id="map" class="h-full w-full"></div>
            
            <!-- Dynamic Legend -->
            <div id="mapLegend" class="absolute top-4 right-4 bg-white/95 backdrop-blur p-3 rounded-lg shadow-lg z-[400] min-w-[140px] border border-slate-200">
                <h4 class="text-xs font-bold mb-2 text-slate-700 border-b pb-1">Легенда</h4>
                <div class="flex items-center gap-2 mb-1.5">
                    <div class="w-3 h-3 rounded-full bg-red-500 border border-white shadow-sm"></div>
                    <span class="text-xs text-slate-600">Центр ділянки</span>
                </div>
                <!-- Species items will be injected here -->
                <div id="speciesLegend"></div>
            </div>
        </main>

    </div>

    <!-- Notification Toast -->
    <div id="toast" class="fixed bottom-5 right-5 bg-slate-800 text-white px-4 py-2 rounded shadow-lg transform translate-y-20 transition-transform duration-300 z-[1000] text-sm flex items-center gap-2">
        <i data-lucide="info" class="w-4 h-4 text-blue-400"></i> <span id="toastMsg">Notification</span>
    </div>

    <script>
        // --- Configuration ---
        const speciesColors = {
            'сосна': '#166534', // green-800
            'ялина': '#064e3b', // emerald-900
            'дуб': '#854d0e',   // yellow-800 (brownish)
            'береза': '#94a3b8', // slate-400
            'вільха': '#78350f', // amber-900
            'осика': '#a3e635',  // lime-400
            'default': '#000000' // black
        };

        const R_earth = 6371e3; 

        // --- State ---
        let map;
        let centers = []; 
        let trees = [];   
        let centerMode = 'absolute';
        let uniqueSpecies = new Set();
        let xyCtx; // Canvas Context

        // --- Init ---
        window.onload = function() {
            if (window.lucide) lucide.createIcons();
            initMap();
            loadFromStorage();
            
            // Init Canvas
            const canvas = document.getElementById('xyCanvas');
            // Fix blurriness on high DPI
            const dpr = window.devicePixelRatio || 1;
            const rect = canvas.getBoundingClientRect();
            canvas.width = rect.width * dpr;
            canvas.height = rect.height * dpr;
            xyCtx = canvas.getContext('2d');
            xyCtx.scale(dpr, dpr);

            // Switch logic
            if (centers.length > 0) {
                renderCentersList();
                updateDropdowns();
                renderTreeTable();
                updateLegend();
            }
        };

        function initMap() {
            map = L.map('map').setView([50.4501, 30.5234], 6);
            L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
                maxZoom: 22,
                attribution: '© OpenStreetMap'
            }).addTo(map);
        }

        // --- Storage Logic ---
        function saveToStorage() {
            const data = {
                centers: centers.map(c => ({...c, marker: null})), 
                trees: trees.map(t => ({...t, marker: null, line: null}))
            };
            localStorage.setItem('marteloscope_v2', JSON.stringify(data));
            
            const status = document.getElementById('storageStatus');
            status.innerText = "Збережено";
            setTimeout(() => status.innerText = "Автозбереження активне", 2000);
        }

        function loadFromStorage() {
            const raw = localStorage.getItem('marteloscope_v2');
            if (!raw) return;

            try {
                const data = JSON.parse(raw);
                data.centers.forEach(c => {
                    const centerObj = { ...c, marker: null };
                    centers.push(centerObj);
                    createCenterMarker(centerObj);
                });
                data.trees.forEach(t => {
                    const treeObj = { ...t, marker: null, line: null };
                    trees.push(treeObj);
                    createTreeMarker(treeObj);
                });
                if (centers.length > 0) {
                    const last = centers[centers.length - 1];
                    map.setView([last.lat, last.lon], 18);
                }
                updateTreeCount();
            } catch (e) {
                console.error("Save file corrupted", e);
                showToast("Помилка завантаження збереження");
            }
        }

        window.clearStorage = function() {
            if(confirm("Видалити всі дані і почати заново?")) {
                localStorage.removeItem('marteloscope_v2');
                location.reload();
            }
        }

        // --- Visual Logic ---
        function getSpeciesColor(sp) {
            const normalized = sp.toLowerCase().trim();
            for (const [key, color] of Object.entries(speciesColors)) {
                if (normalized.includes(key)) return color;
            }
            return speciesColors['default'];
        }

        function getMarkerRadius(dbh) {
            if (!dbh) return 4;
            return Math.max(3, Math.min(15, dbh / 8)); 
        }

        function updateLegend() {
            const legendDiv = document.getElementById('speciesLegend');
            legendDiv.innerHTML = '';
            
            uniqueSpecies.clear();
            trees.forEach(t => { if(t.species) uniqueSpecies.add(t.species.toLowerCase().trim()); });

            if (uniqueSpecies.size === 0) {
                legendDiv.innerHTML = `<div class="text-[10px] text-slate-400 italic">Дерева відсутні</div>`;
                return;
            }

            uniqueSpecies.forEach(sp => {
                const color = getSpeciesColor(sp);
                const display = sp.charAt(0).toUpperCase() + sp.slice(1);
                
                const item = document.createElement('div');
                item.className = 'flex items-center gap-2 mb-1';
                item.innerHTML = `
                    <div class="w-2.5 h-2.5 rounded-full shadow-sm" style="background-color: ${color}"></div>
                    <span class="text-xs text-slate-600">${display}</span>
                `;
                legendDiv.appendChild(item);
            });
        }

        // --- Center Management ---
        window.setCenterMode = function(mode) {
            centerMode = mode;
            document.getElementById('mode-absolute').className = mode === 'absolute' ? 
                "flex-1 py-1.5 rounded shadow-sm bg-white font-bold text-slate-800 transition" : 
                "flex-1 py-1.5 rounded text-slate-500 font-medium hover:text-slate-800 transition";
            
            document.getElementById('mode-relative').className = mode === 'relative' ? 
                "flex-1 py-1.5 rounded shadow-sm bg-white font-bold text-slate-800 transition" : 
                "flex-1 py-1.5 rounded text-slate-500 font-medium hover:text-slate-800 transition";

            document.getElementById('inputs-absolute').className = mode === 'absolute' ? "space-y-2" : "hidden space-y-2";
            document.getElementById('inputs-relative').className = mode === 'relative' ? "space-y-2" : "hidden space-y-2";
        }

        window.addCenter = function() {
            const nameInput = document.getElementById('centerName');
            const name = nameInput.value || `Сектор ${centers.length + 1}`;
            let lat, lon;

            if (centerMode === 'absolute') {
                lat = parseFloat(document.getElementById('centerLat').value);
                lon = parseFloat(document.getElementById('centerLon').value);
            } else {
                const refId = document.getElementById('refCenterSelect').value;
                const az = parseFloat(document.getElementById('centerAz').value);
                const dist = parseFloat(document.getElementById('centerDist').value);
                if (!refId || isNaN(az) || isNaN(dist)) { showToast("Перевірте дані"); return; }
                const ref = centers.find(c => c.id === refId);
                const coords = calculateDestination(ref.lat, ref.lon, az, dist);
                lat = coords.lat; 
                lon = coords.lon;
            }

            if (isNaN(lat) || isNaN(lon)) { showToast("Помилка координат"); return; }

            const centerObj = {
                id: Date.now().toString(),
                name, lat, lon,
                color: '#ef4444'
            };

            centers.push(centerObj);
            createCenterMarker(centerObj);
            
            nameInput.value = '';
            renderCentersList();
            updateDropdowns();
            map.setView([lat, lon], 19);
            saveToStorage();
            showToast("Центр додано");
        }

        function createCenterMarker(c) {
            const icon = L.divIcon({
                className: 'custom-center-icon',
                html: `<div style="background-color:#ef4444; width:14px; height:14px; border-radius:2px; border:2px solid white; box-shadow: 0 1px 3px rgba(0,0,0,0.4); transform: rotate(45deg);"></div>`,
                iconSize: [14, 14], iconAnchor: [7, 7]
            });
            c.marker = L.marker([c.lat, c.lon], {icon}).addTo(map)
                .bindPopup(`<b>${c.name}</b><br>${c.lat.toFixed(6)}, ${c.lon.toFixed(6)}`);
        }

        // --- Tree Management ---
        window.processTrees = function() {
            const centerId = document.getElementById('activeCenterSelect').value;
            const text = document.getElementById('csvInput').value;
            if (!centerId || !text.trim()) { showToast("Оберіть центр та введіть дані"); return; }

            const center = centers.find(c => c.id === centerId);
            const lines = text.split(/\r?\n/);
            let count = 0;

            lines.forEach(line => {
                if (!line.trim()) return;
                const p = line.split(/[,;\t]+/).map(s => s.trim());
                if (p.length < 3) return;

                const num = p[0];
                const az = parseFloat(p[1]);
                const dist = parseFloat(p[2]);
                const species = p[3] || "Невідомо";
                const dbh = parseFloat(p[4]) || 0;
                const height = parseFloat(p[5]) || 0;

                if (isNaN(az) || isNaN(dist)) return;

                const coords = calculateDestination(center.lat, center.lon, az, dist);
                
                const treeObj = {
                    id: Date.now() + Math.random(),
                    centerId, num, az, dist, species, dbh, height,
                    lat: coords.lat, lon: coords.lon
                };

                trees.push(treeObj);
                createTreeMarker(treeObj);
                count++;
            });

            if (count > 0) {
                showToast(`Додано ${count} дерев`);
                document.getElementById('csvInput').value = "";
                updateTreeCount();
                renderTreeTable();
                updateLegend();
                saveToStorage();
                
                const bounds = L.latLngBounds([center.marker.getLatLng()]);
                trees.filter(t => t.centerId === centerId).forEach(t => bounds.extend(t.marker.getLatLng()));
                map.fitBounds(bounds, {padding: [50, 50]});
            }
        }

        function createTreeMarker(t) {
            const color = getSpeciesColor(t.species);
            const radius = getMarkerRadius(t.dbh);
            
            const icon = L.divIcon({
                className: 'tree-marker',
                html: `<div style="background-color:${color}; width:${radius*2}px; height:${radius*2}px; border-radius:50%; border:1px solid white; box-shadow: 0 1px 2px rgba(0,0,0,0.3);"></div>`,
                iconSize: [radius*2, radius*2],
                iconAnchor: [radius, radius]
            });

            t.marker = L.marker([t.lat, t.lon], {icon}).addTo(map)
                .bindPopup(`
                    <div class="text-xs">
                        <b>Дерево №${t.num}</b> (${t.species})<br>
                        D: ${t.dbh} см, H: ${t.height} м<br>
                        Az: ${t.az}°, Dist: ${t.dist}м
                    </div>
                `);

            const center = centers.find(c => c.id === t.centerId);
            if (center) {
                t.line = L.polyline([[center.lat, center.lon], [t.lat, t.lon]], {
                    color: color, weight: 1, opacity: 0.2, dashArray: '3, 3'
                }).addTo(map);
            }
        }

        window.deleteTree = function(treeId) {
            const idx = trees.findIndex(t => t.id == treeId);
            if (idx > -1) {
                const t = trees[idx];
                if(t.marker) map.removeLayer(t.marker);
                if(t.line) map.removeLayer(t.line);
                trees.splice(idx, 1);
                
                renderTreeTable();
                updateTreeCount();
                updateLegend();
                saveToStorage();
                showToast("Дерево видалено");
            }
        }

        // --- Cartesian Transformation Logic ---
        
        // Helper: Lat/Lon to Meters relative to origin
        function latLonToMeters(lat, lon, originLat, originLon) {
            const x = (lon - originLon) * (Math.PI/180) * R_earth * Math.cos(originLat * Math.PI/180);
            const y = (lat - originLat) * (Math.PI/180) * R_earth;
            return {x, y};
        }

        // Helper: Rotate Point around (0,0)
        function rotatePoint(x, y, angleDeg) {
            const rad = angleDeg * Math.PI / 180;
            const cos = Math.cos(rad);
            const sin = Math.sin(rad);
            return {
                x: x * cos - y * sin,
                y: x * sin + y * cos
            };
        }

        window.updateXYPreview = function() {
            if (centers.length === 0) return;
            
            const canvas = document.getElementById('xyCanvas');
            const rot = parseInt(document.getElementById('rotationSlider').value);
            document.getElementById('rotVal').innerText = rot;
            
            const origin = centers[0]; // First center is origin (0,0)
            const pts = [];

            // Transform all points
            trees.forEach(t => {
                const m = latLonToMeters(t.lat, t.lon, origin.lat, origin.lon);
                const r = rotatePoint(m.x, m.y, rot);
                pts.push({ x: r.x, y: r.y, color: getSpeciesColor(t.species) });
            });

            // Draw
            const w = canvas.clientWidth; // Use client dimensions for logic
            const h = canvas.clientHeight;
            
            // Clear
            xyCtx.clearRect(0, 0, canvas.width, canvas.height); // Use actual buffer dimensions
            
            // Need buffer dimensions for drawing operations since we scaled context
            // But logic coordinates should be 0..w
            
            if(pts.length === 0) {
                xyCtx.font = "12px sans-serif";
                xyCtx.fillStyle = "#94a3b8";
                xyCtx.textAlign = "center";
                xyCtx.fillText("Немає дерев для відображення", w/2, h/2);
                return;
            }

            // Find bounds to scale fit
            let minX = Infinity, maxX = -Infinity, minY = Infinity, maxY = -Infinity;
            pts.forEach(p => {
                minX = Math.min(minX, p.x); maxX = Math.max(maxX, p.x);
                minY = Math.min(minY, p.y); maxY = Math.max(maxY, p.y);
            });

            // Add padding
            const pad = 5;
            minX -= pad; maxX += pad; minY -= pad; maxY += pad;
            
            const rangeX = maxX - minX || 10;
            const rangeY = maxY - minY || 10;
            const scale = Math.min(w / rangeX, h / rangeY) * 0.9; // 90% fill
            
            const offsetX = (w - rangeX * scale) / 2;
            const offsetY = (h - rangeY * scale) / 2;

            // Draw axis
            xyCtx.strokeStyle = "#e2e8f0";
            xyCtx.beginPath();
            // Need to map (0,0) to canvas
            const zeroX = offsetX + (0 - minX) * scale;
            const zeroY = h - (offsetY + (0 - minY) * scale); // Invert Y for canvas
            
            xyCtx.moveTo(zeroX, 0); xyCtx.lineTo(zeroX, h);
            xyCtx.moveTo(0, zeroY); xyCtx.lineTo(w, zeroY);
            xyCtx.stroke();

            // Draw Points
            pts.forEach(p => {
                const cx = offsetX + (p.x - minX) * scale;
                const cy = h - (offsetY + (p.y - minY) * scale);
                
                xyCtx.fillStyle = p.color;
                xyCtx.beginPath();
                xyCtx.arc(cx, cy, 3, 0, Math.PI * 2);
                xyCtx.fill();
            });
        }

        window.downloadXYData = function() {
            if (trees.length === 0) { showToast("Немає даних"); return; }
            
            const rot = parseInt(document.getElementById('rotationSlider').value);
            const origin = centers[0];
            
            let csv = "ID,Species,DBH,Height,Original_Lat,Original_Lon,X_Local,Y_Local,Rotated_X,Rotated_Y\n";
            
            trees.forEach(t => {
                const m = latLonToMeters(t.lat, t.lon, origin.lat, origin.lon);
                const r = rotatePoint(m.x, m.y, rot);
                
                csv += `"${t.num}","${t.species}",${t.dbh},${t.height},${t.lat},${t.lon},${m.x.toFixed(3)},${m.y.toFixed(3)},${r.x.toFixed(3)},${r.y.toFixed(3)}\n`;
            });

            const blob = new Blob(["\uFEFF" + csv], { type: 'text/csv;charset=utf-8;' });
            const link = document.createElement("a");
            link.href = URL.createObjectURL(blob);
            link.download = `cartesian_trees_rot${rot}_${new Date().toISOString().slice(0,10)}.csv`;
            link.click();
        }


        // --- Rendering Lists & Helpers ---
        function renderCentersList() {
            const list = document.getElementById('centersList');
            list.innerHTML = '';
            centers.forEach(c => {
                const li = document.createElement('li');
                li.className = "bg-white p-2 rounded border border-slate-200 shadow-sm flex justify-between items-center";
                li.innerHTML = `
                    <div class="flex items-center gap-2 cursor-pointer" onclick="panToCenter('${c.id}')">
                        <div class="w-2 h-2 bg-red-500 rotate-45"></div>
                        <div>
                            <div class="text-xs font-bold text-slate-700">${c.name}</div>
                            <div class="text-[10px] text-slate-400 font-mono">${c.lat.toFixed(5)}, ${c.lon.toFixed(5)}</div>
                        </div>
                    </div>
                    <button onclick="removeCenter('${c.id}')" class="text-slate-300 hover:text-red-500"><i data-lucide="trash-2" class="w-3 h-3"></i></button>
                `;
                list.appendChild(li);
            });
            if(window.lucide) lucide.createIcons();
        }

        function renderTreeTable() {
            const tbody = document.getElementById('treesTableBody');
            tbody.innerHTML = '';
            const sorted = [...trees].sort((a,b) => {
                if(a.centerId !== b.centerId) return a.centerId.localeCompare(b.centerId);
                return parseInt(a.num) - parseInt(b.num);
            });
            sorted.forEach(t => {
                const cName = centers.find(c => c.id === t.centerId)?.name || '???';
                const tr = document.createElement('tr');
                tr.className = "hover:bg-slate-50 transition group";
                const coordStr = `${t.lat.toFixed(6)}, ${t.lon.toFixed(6)}`;
                tr.innerHTML = `
                    <td class="p-2 border-b font-mono text-slate-600">${t.num}</td>
                    <td class="p-2 border-b font-mono text-[10px] text-slate-500">${coordStr}</td>
                    <td class="p-2 border-b font-medium text-slate-800">${t.species || '-'}</td>
                    <td class="p-2 border-b text-slate-600">${t.dbh || '-'}</td>
                    <td class="p-2 border-b text-slate-600">${t.height || '-'}</td>
                    <td class="p-2 border-b text-right">
                        <button onclick="zoomToTree('${t.id}')" class="text-blue-400 hover:text-blue-600 mr-2"><i data-lucide="search" class="w-3 h-3"></i></button>
                        <button onclick="deleteTree('${t.id}')" class="text-slate-300 hover:text-red-500"><i data-lucide="trash" class="w-3 h-3"></i></button>
                    </td>
                `;
                tbody.appendChild(tr);
            });
            if(window.lucide) lucide.createIcons();
        }

        window.switchTab = function(tab) {
            ['centers', 'trees', 'list', 'xy'].forEach(t => {
                document.getElementById(`content-${t}`).classList.add('hidden');
                const btn = document.getElementById(`tab-${t}`);
                btn.className = "flex-1 py-3 text-[10px] sm:text-xs font-bold uppercase tracking-wide text-slate-500 hover:text-emerald-600";
            });
            document.getElementById(`content-${tab}`).classList.remove('hidden');
            document.getElementById(`tab-${tab}`).className = "flex-1 py-3 text-[10px] sm:text-xs font-bold uppercase tracking-wide text-emerald-700 border-b-2 border-emerald-600 bg-white";
            
            // Trigger redraw on tab switch if needed
            if(tab === 'xy') updateXYPreview();
        }

        function updateDropdowns() {
            const opts = centers.map(c => `<option value="${c.id}">${c.name}</option>`).join('');
            document.getElementById('activeCenterSelect').innerHTML = opts || '<option>Додайте центр</option>';
            document.getElementById('refCenterSelect').innerHTML = opts || '<option>Додайте центр</option>';
        }

        function calculateDestination(lat1, lon1, az, dist) {
            const radLat = lat1 * Math.PI / 180;
            const radLon = lon1 * Math.PI / 180;
            const radAz = az * Math.PI / 180;
            const lat2 = Math.asin(Math.sin(radLat)*Math.cos(dist/R_earth) + Math.cos(radLat)*Math.sin(dist/R_earth)*Math.cos(radAz));
            const lon2 = radLon + Math.atan2(Math.sin(radAz)*Math.sin(dist/R_earth)*Math.cos(radLat), Math.cos(dist/R_earth)-Math.sin(radLat)*Math.sin(lat2));
            return { lat: lat2 * 180 / Math.PI, lon: (lon2 * 180 / Math.PI + 540) % 360 - 180 };
        }

        window.downloadAllData = function() {
            if (trees.length === 0) { showToast("Немає даних"); return; }
            let csv = "Center,Lat,Lon,TreeNum,Species,DBH,Height,Azimuth,Dist,TreeLat,TreeLon\n";
            trees.forEach(t => {
                const c = centers.find(x => x.id === t.centerId);
                if(c) csv += `"${c.name}",${c.lat},${c.lon},"${t.num}","${t.species}",${t.dbh},${t.height},${t.az},${t.dist},${t.lat},${t.lon}\n`;
            });
            const blob = new Blob(["\uFEFF" + csv], { type: 'text/csv;charset=utf-8;' });
            const link = document.createElement("a");
            link.href = URL.createObjectURL(blob);
            link.download = `marteloscope_${new Date().toISOString().slice(0,10)}.csv`;
            link.click();
        }

        window.insertSampleData = function() {
             document.getElementById('csvInput').value = "1, 45, 12.5, Сосна, 32, 24\n2, 120, 8.3, Дуб, 44, 21\n3, 270, 15.0, Береза, 28, 19\n4, 310, 22.1, Ялина, 50, 28";
        }
        
        window.removeCenter = function(id) {
            if(!confirm("Видалити центр і всі його дерева?")) return;
            centers = centers.filter(c => {
                if(c.id === id) { map.removeLayer(c.marker); return false; }
                return true;
            });
            for (let i = trees.length - 1; i >= 0; i--) {
                if (trees[i].centerId === id) {
                    if(trees[i].marker) map.removeLayer(trees[i].marker);
                    if(trees[i].line) map.removeLayer(trees[i].line);
                    trees.splice(i, 1);
                }
            }
            renderCentersList(); updateDropdowns(); renderTreeTable(); updateLegend(); saveToStorage();
        }
        
        window.panToCenter = function(id) {
            const c = centers.find(x => x.id === id);
            if(c) map.setView([c.lat, c.lon], 19);
        }

        window.zoomToTree = function(id) {
             const t = trees.find(x => x.id == id);
             if(t) {
                 map.setView([t.lat, t.lon], 21);
                 t.marker.openPopup();
             }
        }

        window.getCurrentLocation = function() {
            if("geolocation" in navigator) {
                navigator.geolocation.getCurrentPosition(p => {
                    document.getElementById('centerLat').value = p.coords.latitude.toFixed(6);
                    document.getElementById('centerLon').value = p.coords.longitude.toFixed(6);
                });
            } else showToast("Геолокація недоступна");
        }

        window.updateTreeCount = function() {
            document.getElementById('totalTreesCount').innerText = trees.length;
        }

        window.showToast = function(msg) {
            const t = document.getElementById('toast');
            document.getElementById('toastMsg').innerText = msg;
            t.classList.remove('translate-y-20');
            setTimeout(() => t.classList.add('translate-y-20'), 3000);
        }
    </script>
</body>
</html>
