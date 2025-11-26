<!DOCTYPE html>
<html lang="uk">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Мартелоскоп: Картографування Дерев</title>
    
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
        ::-webkit-scrollbar { width: 8px; }
        ::-webkit-scrollbar-track { background: #f1f1f1; }
        ::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 4px; }
        ::-webkit-scrollbar-thumb:hover { background: #94a3b8; }
    </style>
</head>
<body class="bg-slate-100 text-slate-800 h-screen flex flex-col overflow-hidden">

    <!-- Header -->
    <header class="bg-emerald-800 text-white p-3 shadow-lg shrink-0 z-20">
        <div class="flex items-center justify-between px-4">
            <div class="flex items-center gap-2">
                <i data-lucide="tree-pine" class="w-6 h-6"></i>
                <h1 class="text-lg font-bold">Мартелоскоп Mapper</h1>
            </div>
            <div class="text-xs opacity-80 hidden sm:block">Інструмент таксації лісу</div>
        </div>
    </header>

    <!-- Main Layout -->
    <div class="flex flex-grow overflow-hidden">
        
        <!-- Sidebar (Controls) -->
        <aside class="w-full md:w-[400px] bg-white shadow-xl z-10 flex flex-col overflow-hidden border-r border-slate-200">
            
            <!-- Tabs Navigation -->
            <div class="flex border-b border-slate-200 shrink-0">
                <button onclick="switchTab('centers')" id="tab-centers" class="flex-1 py-3 text-sm font-semibold text-emerald-700 border-b-2 border-emerald-600 bg-emerald-50">
                    1. Центри ділянок
                </button>
                <button onclick="switchTab('trees')" id="tab-trees" class="flex-1 py-3 text-sm font-semibold text-slate-500 hover:text-emerald-600">
                    2. Завантаження дерев
                </button>
            </div>

            <!-- Tab Content: Centers -->
            <div id="content-centers" class="flex-grow flex flex-col overflow-y-auto p-4 space-y-4">
                <div class="bg-slate-50 p-4 rounded-lg border border-slate-200">
                    <h3 class="text-sm font-bold text-slate-700 mb-3 flex items-center gap-2">
                        <i data-lucide="map-pin" class="w-4 h-4"></i> Додати центр ділянки
                    </h3>
                    
                    <!-- Toggle Mode -->
                    <div class="flex mb-3 bg-slate-200 p-1 rounded text-xs">
                        <button onclick="setCenterMode('absolute')" id="mode-absolute" class="flex-1 py-1 rounded shadow-sm bg-white font-bold text-slate-700 transition">Координати / GPS</button>
                        <button onclick="setCenterMode('relative')" id="mode-relative" class="flex-1 py-1 rounded text-slate-500 font-medium hover:text-slate-700 transition">Від іншої точки</button>
                    </div>

                    <div class="space-y-3">
                        <input type="text" id="centerName" placeholder="Назва (напр. Центр 1)" class="w-full p-2 text-sm border rounded focus:ring-2 focus:ring-emerald-500 outline-none">
                        
                        <!-- Absolute Mode Inputs -->
                        <div id="inputs-absolute" class="space-y-2">
                            <div class="grid grid-cols-2 gap-2">
                                <input type="number" id="centerLat" step="any" placeholder="Широта" class="p-2 text-sm border rounded">
                                <input type="number" id="centerLon" step="any" placeholder="Довгота" class="p-2 text-sm border rounded">
                            </div>
                            <button onclick="getCurrentLocation()" class="w-full py-2 bg-slate-200 hover:bg-slate-300 text-slate-700 text-xs rounded transition flex items-center justify-center gap-1">
                                <i data-lucide="crosshair" class="w-3 h-3"></i> Отримати GPS
                            </button>
                        </div>

                        <!-- Relative Mode Inputs -->
                        <div id="inputs-relative" class="hidden space-y-2">
                            <div>
                                <label class="block text-[10px] font-bold text-slate-400 uppercase mb-1">Базова точка (від якої міряємо)</label>
                                <select id="refCenterSelect" class="w-full p-2 border border-slate-300 rounded text-sm bg-white">
                                    <option value="">Спочатку додайте хоча б один центр...</option>
                                </select>
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
                            <i data-lucide="plus-circle" class="w-4 h-4"></i> Додати центр
                        </button>
                    </div>
                </div>

                <div class="flex-grow">
                    <h3 class="text-xs font-bold text-slate-400 uppercase mb-2">Список центрів</h3>
                    <ul id="centersList" class="space-y-2">
                        <!-- Centers will be added here -->
                        <li class="text-sm text-slate-400 italic text-center py-4">Центри ще не додані</li>
                    </ul>
                </div>
            </div>

            <!-- Tab Content: Trees -->
            <div id="content-trees" class="hidden flex-grow flex flex-col overflow-y-auto p-4 space-y-4">
                
                <!-- Step 1: Select Center -->
                <div class="bg-blue-50 p-4 rounded-lg border border-blue-100">
                    <label class="block text-xs font-bold text-blue-800 uppercase mb-2">Крок А: Оберіть активний центр</label>
                    <select id="activeCenterSelect" class="w-full p-2 border border-blue-300 rounded text-sm bg-white">
                        <option value="">Спочатку додайте центри...</option>
                    </select>
                </div>

                <!-- Step 2: Input Data -->
                <div class="bg-white p-4 rounded-lg border border-slate-200 shadow-sm">
                    <label class="block text-xs font-bold text-slate-700 uppercase mb-2">Крок Б: Дані про дерева</label>
                    
                    <div class="mb-3">
                        <p class="text-xs text-slate-500 mb-2">Формат: <span class="font-mono bg-slate-100 px-1">Номер, Азимут(°), Відстань(м)</span></p>
                        <textarea id="csvInput" rows="6" class="w-full p-2 text-xs font-mono border border-slate-300 rounded focus:ring-2 focus:ring-emerald-500 outline-none" placeholder="1, 45, 12.5&#10;2, 120, 8.3&#10;3, 270, 15.0"></textarea>
                    </div>

                    <div class="flex gap-2 mb-3">
                        <input type="file" id="fileUpload" accept=".csv,.txt" class="hidden" onchange="handleFileUpload(this)">
                        <button onclick="document.getElementById('fileUpload').click()" class="flex-1 py-2 border border-slate-300 hover:bg-slate-50 text-slate-600 text-xs rounded flex items-center justify-center gap-1">
                            <i data-lucide="upload" class="w-3 h-3"></i> З файлу
                        </button>
                        <button onclick="processTrees()" class="flex-1 py-2 bg-emerald-600 hover:bg-emerald-700 text-white text-xs font-bold rounded shadow-sm flex items-center justify-center gap-1">
                            <i data-lucide="play" class="w-3 h-3"></i> Розрахувати
                        </button>
                    </div>
                </div>

                <!-- Stats & Export -->
                <div class="border-t pt-4 space-y-3">
                    <div class="flex justify-between items-center">
                        <h3 class="text-xs font-bold text-slate-400 uppercase">Всього дерев</h3>
                        <span id="totalTreesCount" class="text-sm font-bold text-emerald-700">0</span>
                    </div>
                    
                    <button onclick="downloadAllData()" class="w-full py-2 bg-blue-600 hover:bg-blue-700 text-white text-sm font-bold rounded shadow-sm flex items-center justify-center gap-2">
                        <i data-lucide="download" class="w-4 h-4"></i> Завантажити звіт (.csv)
                    </button>

                    <button onclick="clearAllTrees()" class="text-xs text-red-500 hover:text-red-700 underline w-full text-center">Очистити всі дерева</button>
                </div>
            </div>

            <!-- Footer -->
            <div class="p-3 bg-slate-50 border-t border-slate-200 text-xs text-center text-slate-500">
                Використовуйте WGS84 координати
            </div>
        </aside>

        <!-- Map Area -->
        <main class="flex-grow relative bg-slate-200">
            <div id="map" class="h-full w-full"></div>
            
            <!-- Legend / Info overlay -->
            <div class="absolute top-4 right-4 bg-white/90 backdrop-blur p-3 rounded shadow-md z-[400] max-w-xs">
                <h4 class="text-xs font-bold mb-2">Легенда</h4>
                <div class="flex items-center gap-2 mb-1">
                    <div class="w-3 h-3 rounded-full bg-red-500 border-2 border-white shadow-sm"></div>
                    <span class="text-xs">Центр ділянки</span>
                </div>
                <div class="flex items-center gap-2">
                    <div class="w-2 h-2 rounded-full bg-emerald-600"></div>
                    <span class="text-xs">Дерево</span>
                </div>
            </div>
        </main>

    </div>

    <!-- Notification Toast -->
    <div id="toast" class="fixed bottom-5 right-5 bg-slate-800 text-white px-4 py-2 rounded shadow-lg transform translate-y-20 transition-transform duration-300 z-50 text-sm">
        Notification
    </div>

    <script>
        // --- Icons ---
        if (window.lucide) lucide.createIcons();

        // --- State Management ---
        var map;
        var centers = []; // Array of {id, name, lat, lon, color, marker}
        var trees = [];   // Array of {id, centerId, treeNum, lat, lon, marker, line, distance, azimuth}
        var centerMode = 'absolute'; // 'absolute' or 'relative'
        var R_earth = 6371e3; 
        
        // Colors for different centers to distinguish them
        var centerColors = ['#ef4444', '#f59e0b', '#3b82f6', '#8b5cf6', '#ec4899'];

        // --- Initialization ---
        window.onload = function() {
            initMap();
        };

        function initMap() {
            map = L.map('map').setView([50.4501, 30.5234], 6); // Zoomed out initially

            L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
                maxZoom: 21,
                attribution: '© OpenStreetMap'
            }).addTo(map);
        }

        // --- Tab Logic ---
        window.switchTab = function(tabName) {
            document.getElementById('content-centers').classList.add('hidden');
            document.getElementById('content-trees').classList.add('hidden');
            document.getElementById('tab-centers').classList.remove('border-b-2', 'border-emerald-600', 'bg-emerald-50', 'text-emerald-700');
            document.getElementById('tab-trees').classList.remove('border-b-2', 'border-emerald-600', 'bg-emerald-50', 'text-emerald-700');
            
            document.getElementById('tab-centers').classList.add('text-slate-500');
            document.getElementById('tab-trees').classList.add('text-slate-500');

            document.getElementById(`content-${tabName}`).classList.remove('hidden');
            const activeTab = document.getElementById(`tab-${tabName}`);
            activeTab.classList.remove('text-slate-500');
            activeTab.classList.add('border-b-2', 'border-emerald-600', 'bg-emerald-50', 'text-emerald-700');
        }

        // --- Center Logic ---
        
        window.setCenterMode = function(mode) {
            centerMode = mode;
            
            const btnAbs = document.getElementById('mode-absolute');
            const btnRel = document.getElementById('mode-relative');
            const inpAbs = document.getElementById('inputs-absolute');
            const inpRel = document.getElementById('inputs-relative');

            if (mode === 'absolute') {
                btnAbs.classList.add('bg-white', 'shadow-sm', 'text-slate-700');
                btnAbs.classList.remove('text-slate-500');
                btnRel.classList.remove('bg-white', 'shadow-sm', 'text-slate-700');
                btnRel.classList.add('text-slate-500');
                
                inpAbs.classList.remove('hidden');
                inpRel.classList.add('hidden');
            } else {
                btnRel.classList.add('bg-white', 'shadow-sm', 'text-slate-700');
                btnRel.classList.remove('text-slate-500');
                btnAbs.classList.remove('bg-white', 'shadow-sm', 'text-slate-700');
                btnAbs.classList.add('text-slate-500');
                
                inpRel.classList.remove('hidden');
                inpAbs.classList.add('hidden');
            }
        }

        window.addCenter = function() {
            const name = document.getElementById('centerName').value || `Центр ${centers.length + 1}`;
            
            let lat, lon;

            if (centerMode === 'absolute') {
                // Get from inputs
                lat = parseFloat(document.getElementById('centerLat').value);
                lon = parseFloat(document.getElementById('centerLon').value);
            } else {
                // Calculate relative
                const refId = document.getElementById('refCenterSelect').value;
                const az = parseFloat(document.getElementById('centerAz').value);
                const dist = parseFloat(document.getElementById('centerDist').value);

                if (!refId) { showToast("Оберіть базову точку!"); return; }
                if (isNaN(az) || isNaN(dist)) { showToast("Введіть азимут та відстань!"); return; }

                const refCenter = centers.find(c => c.id === refId);
                if (!refCenter) { showToast("Базова точка не знайдена"); return; }

                const coords = calculateDestination(refCenter.lat, refCenter.lon, az, dist);
                lat = coords.lat;
                lon = coords.lon;
            }

            if (isNaN(lat) || isNaN(lon)) {
                showToast("Помилка координат");
                return;
            }

            const id = Date.now().toString();
            const color = centerColors[centers.length % centerColors.length];

            // Create Marker
            const icon = L.divIcon({
                className: 'custom-center-icon',
                html: `<div style="background-color:${color}; width:16px; height:16px; border-radius:50%; border:2px solid white; box-shadow: 0 0 5px rgba(0,0,0,0.4);"></div>`,
                iconSize: [16, 16],
                iconAnchor: [8, 8]
            });

            const marker = L.marker([lat, lon], {icon: icon})
                .addTo(map)
                .bindPopup(`<b>${name}</b><br>${lat.toFixed(5)}, ${lon.toFixed(5)}`);

            const centerObj = { id, name, lat, lon, color, marker };
            centers.push(centerObj);

            // Update UI
            renderCentersList();
            updateSelectDropdown();
            updateRefCenterDropdown(); // Update the relative mode dropdown too
            
            // Clear inputs based on mode
            document.getElementById('centerName').value = '';
            if(centerMode === 'absolute') {
                document.getElementById('centerLat').value = '';
                document.getElementById('centerLon').value = '';
            } else {
                document.getElementById('centerAz').value = '';
                document.getElementById('centerDist').value = '';
            }

            // Focus map
            map.setView([lat, lon], 18);
            showToast(`Центр "${name}" додано`);
        }

        function renderCentersList() {
            const list = document.getElementById('centersList');
            list.innerHTML = '';

            if (centers.length === 0) {
                list.innerHTML = '<li class="text-sm text-slate-400 italic text-center py-4">Центри ще не додані</li>';
                return;
            }

            centers.forEach((c, index) => {
                const li = document.createElement('li');
                li.className = "bg-white p-3 rounded border border-slate-200 shadow-sm flex justify-between items-center";
                li.innerHTML = `
                    <div class="flex items-center gap-2 cursor-pointer" onclick="panToCenter('${c.id}')">
                        <div class="w-3 h-3 rounded-full" style="background-color: ${c.color}"></div>
                        <div>
                            <div class="text-sm font-bold text-slate-700">${c.name}</div>
                            <div class="text-xs text-slate-400 font-mono">${c.lat.toFixed(5)}, ${c.lon.toFixed(5)}</div>
                        </div>
                    </div>
                    <button onclick="removeCenter('${c.id}')" class="text-slate-400 hover:text-red-500">
                        <i data-lucide="trash-2" class="w-4 h-4"></i>
                    </button>
                `;
                list.appendChild(li);
            });
            lucide.createIcons();
        }

        window.removeCenter = function(id) {
            const index = centers.findIndex(c => c.id === id);
            if (index > -1) {
                // Remove marker
                map.removeLayer(centers[index].marker);
                
                // Remove associated trees
                // Iterate backwards to avoid index issues
                for (let i = trees.length - 1; i >= 0; i--) {
                    if (trees[i].centerId === id) {
                        map.removeLayer(trees[i].marker);
                        if(trees[i].line) map.removeLayer(trees[i].line);
                        trees.splice(i, 1);
                    }
                }

                centers.splice(index, 1);
                renderCentersList();
                updateSelectDropdown();
                updateRefCenterDropdown();
                updateTreeCount();
                showToast("Центр та пов'язані дерева видалено");
            }
        }

        window.panToCenter = function(id) {
            const c = centers.find(c => c.id === id);
            if (c) map.setView([c.lat, c.lon], 19);
        }

        function updateSelectDropdown() {
            const select = document.getElementById('activeCenterSelect');
            select.innerHTML = '';
            
            if (centers.length === 0) {
                const opt = document.createElement('option');
                opt.text = "Спочатку додайте центри...";
                select.add(opt);
                return;
            }

            centers.forEach(c => {
                const opt = document.createElement('option');
                opt.value = c.id;
                opt.text = c.name;
                select.add(opt);
            });
        }

        function updateRefCenterDropdown() {
            const select = document.getElementById('refCenterSelect');
            const currentVal = select.value;
            select.innerHTML = '';

            if (centers.length === 0) {
                const opt = document.createElement('option');
                opt.text = "Спочатку додайте хоча б один центр...";
                select.add(opt);
                return;
            }

            centers.forEach(c => {
                const opt = document.createElement('option');
                opt.value = c.id;
                opt.text = c.name;
                select.add(opt);
            });

            if(currentVal) select.value = currentVal;
        }

        // --- Tree Processing ---

        window.handleFileUpload = function(input) {
            const file = input.files[0];
            if (!file) return;

            const reader = new FileReader();
            reader.onload = function(e) {
                document.getElementById('csvInput').value = e.target.result;
                showToast("Файл завантажено в текстове поле");
            };
            reader.readAsText(file);
        }

        window.processTrees = function() {
            const centerId = document.getElementById('activeCenterSelect').value;
            if (!centerId || centers.length === 0) {
                showToast("Оберіть центр ділянки!");
                return;
            }

            const rawData = document.getElementById('csvInput').value;
            if (!rawData.trim()) {
                showToast("Введіть дані про дерева");
                return;
            }

            const center = centers.find(c => c.id === centerId);
            const lines = rawData.split(/\r?\n/);
            let countAdded = 0;

            lines.forEach(line => {
                if (!line.trim()) return;
                
                // Flexible splitting: comma, semicolon, or tab
                const parts = line.split(/[,;\t]+/);
                if (parts.length < 3) return; // Need at least Num, Az, Dist

                const treeNum = parts[0].trim();
                const azimuth = parseFloat(parts[1].replace(',', '.'));
                const distance = parseFloat(parts[2].replace(',', '.'));

                if (isNaN(azimuth) || isNaN(distance)) return;

                // Calculate coordinates
                const coords = calculateDestination(center.lat, center.lon, azimuth, distance);
                
                addTreeToMap(center, treeNum, coords.lat, coords.lon, distance, azimuth);
                countAdded++;
            });

            if (countAdded > 0) {
                showToast(`Додано ${countAdded} дерев`);
                document.getElementById('csvInput').value = ""; // Clear input
                updateTreeCount();
                
                // Fit bounds to see new trees
                const group = new L.featureGroup([
                    center.marker, 
                    ...trees.filter(t => t.centerId === center.id).map(t => t.marker)
                ]);
                map.fitBounds(group.getBounds(), {padding: [50, 50]});
            } else {
                showToast("Не вдалося розпізнати дані. Перевірте формат.");
            }
        }

        function calculateDestination(lat1Deg, lon1Deg, azDeg, distMeters) {
            const lat1 = toRad(lat1Deg);
            const lon1 = toRad(lon1Deg);
            const brng = toRad(azDeg);
            const d = distMeters; // Distance in meters

            const lat2 = Math.asin( Math.sin(lat1)*Math.cos(d/R_earth) +
                        Math.cos(lat1)*Math.sin(d/R_earth)*Math.cos(brng) );
            
            const lon2 = lon1 + Math.atan2(Math.sin(brng)*Math.sin(d/R_earth)*Math.cos(lat1),
                                Math.cos(d/R_earth)-Math.sin(lat1)*Math.sin(lat2));

            return {
                lat: toDeg(lat2),
                lon: (toDeg(lon2) + 540) % 360 - 180
            };
        }

        function addTreeToMap(center, num, lat, lon, dist, az) {
            // Marker Style
            const icon = L.divIcon({
                className: 'tree-icon',
                html: `<div style="background-color:#10b981; width:8px; height:8px; border-radius:50%; border:1px solid white;"></div>`,
                iconSize: [8, 8],
                iconAnchor: [4, 4]
            });

            const marker = L.marker([lat, lon], {icon: icon})
                .addTo(map)
                .bindPopup(`
                    <b>Дерево №${num}</b><br>
                    Центр: ${center.name}<br>
                    Відстань: ${dist} м<br>
                    Coords: ${lat.toFixed(6)}, ${lon.toFixed(6)}
                `);

            // Optional: Draw line from center to tree
            const line = L.polyline([[center.lat, center.lon], [lat, lon]], {
                color: center.color,
                weight: 1,
                opacity: 0.4
            }).addTo(map);

            trees.push({
                id: Date.now() + Math.random(),
                centerId: center.id,
                treeNum: num,
                lat: lat,
                lon: lon,
                marker: marker,
                line: line,
                distance: dist,
                azimuth: az
            });
        }

        // --- Export Functionality ---
        window.downloadAllData = function() {
            if (trees.length === 0) {
                showToast("Немає даних для експорту");
                return;
            }

            // Headers
            let csvContent = "Center Name,Center Lat,Center Lon,Tree Number,Tree Lat,Tree Lon,Distance (m),Azimuth (deg)\n";

            // Rows
            trees.forEach(function(t) {
                const center = centers.find(c => c.id === t.centerId);
                if (!center) return;

                // Wrap strings in quotes to handle potential commas in names
                const row = [
                    `"${center.name}"`,
                    center.lat,
                    center.lon,
                    `"${t.treeNum}"`,
                    t.lat.toFixed(8),
                    t.lon.toFixed(8),
                    t.distance,
                    t.azimuth
                ].join(",");
                csvContent += row + "\r\n";
            });

            // Create a Blob with UTF-8 BOM for Excel compatibility with Cyrillic
            const blob = new Blob(["\uFEFF" + csvContent], { type: 'text/csv;charset=utf-8;' });
            const url = URL.createObjectURL(blob);
            
            const link = document.createElement("a");
            link.setAttribute("href", url);
            const date = new Date().toISOString().slice(0,10);
            link.setAttribute("download", `marteloscope_data_${date}.csv`);
            document.body.appendChild(link);
            
            link.click();
            
            document.body.removeChild(link);
            URL.revokeObjectURL(url);
            
            showToast("Файл завантажено!");
        }

        window.clearAllTrees = function() {
            if(!confirm("Ви впевнені? Це видалить всі дерева, але залишить центри.")) return;
            
            trees.forEach(t => {
                map.removeLayer(t.marker);
                if(t.line) map.removeLayer(t.line);
            });
            trees = [];
            updateTreeCount();
            showToast("Всі дерева очищено");
        }

        function updateTreeCount() {
            document.getElementById('totalTreesCount').innerText = trees.length;
        }

        // --- Utility ---
        window.getCurrentLocation = function() {
            if ("geolocation" in navigator) {
                navigator.geolocation.getCurrentPosition(p => {
                    document.getElementById('centerLat').value = p.coords.latitude.toFixed(6);
                    document.getElementById('centerLon').value = p.coords.longitude.toFixed(6);
                });
            } else {
                showToast("Геолокація недоступна");
            }
        }

        window.toRad = function(deg) { return deg * Math.PI / 180; }
        window.toDeg = function(rad) { return rad * 180 / Math.PI; }

        window.showToast = function(msg) {
            const t = document.getElementById('toast');
            t.innerText = msg;
            t.classList.remove('translate-y-20');
            setTimeout(() => t.classList.add('translate-y-20'), 3000);
        }

    </script>
</body>
</html>
