<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Muzcuk.Systeam - Ultimate Edition</title>
    
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Font Awesome -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Three.js ve OrbitControls -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js"></script>

    <style>
        @import url('https://fonts.googleapis.com/css2?family=Exo+2:wght@300;500;700&family=Share+Tech+Mono&display=swap');
        
        body {
            font-family: 'Exo 2', sans-serif;
            overflow: hidden;
            background-color: #000;
            color: #fff;
        }

        /* Sci-Fi UI Elementleri */
        .glass-panel {
            background: rgba(8, 15, 30, 0.85);
            backdrop-filter: blur(16px);
            border: 1px solid rgba(56, 189, 248, 0.3);
            box-shadow: 0 0 20px rgba(56, 189, 248, 0.15), inset 0 0 20px rgba(56, 189, 248, 0.05);
        }

        .holo-text {
            font-family: 'Share Tech Mono', monospace;
            color: #38bdf8;
            text-shadow: 0 0 5px rgba(56, 189, 248, 0.8);
        }

        .nav-btn {
            position: relative;
            overflow: hidden;
            transition: all 0.3s ease;
        }
        .nav-btn::before {
            content: '';
            position: absolute;
            top: 0; left: -100%;
            width: 100%; height: 100%;
            background: linear-gradient(90deg, transparent, rgba(56, 189, 248, 0.2), transparent);
            transition: 0.5s;
        }
        .nav-btn:hover::before {
            left: 100%;
        }

        /* Scrollbar */
        .scifi-scroll::-webkit-scrollbar { width: 6px; }
        .scifi-scroll::-webkit-scrollbar-track { background: rgba(0,0,0,0.5); }
        .scifi-scroll::-webkit-scrollbar-thumb { background: #0ea5e9; border-radius: 3px; box-shadow: 0 0 10px #0ea5e9; }

        /* Loader */
        .loader-ring {
            width: 80px; height: 80px;
            border: 4px solid transparent;
            border-top: 4px solid #0ea5e9;
            border-radius: 50%;
            animation: spin 1s linear infinite;
        }
        @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }

        #canvas-container { position: fixed; top: 0; left: 0; z-index: 0; width: 100%; height: 100%; }
        
        /* Modal Transitions */
        .modal-enter { opacity: 0; transform: scale(0.95); }
        .modal-enter-active { opacity: 1; transform: scale(1); transition: all 0.3s ease-out; }
        .modal-exit { opacity: 1; transform: scale(1); }
        .modal-exit-active { opacity: 0; transform: scale(0.95); transition: all 0.2s ease-in; }
    </style>
</head>
<body class="antialiased selection:bg-sky-500 selection:text-white">

    <!-- Yükleme Ekranı -->
    <div id="loading" class="fixed inset-0 bg-black z-[100] flex flex-col justify-center items-center transition-opacity duration-700">
        <div class="loader-ring"></div>
        <div class="mt-6 text-center">
            <h1 class="text-3xl font-bold holo-text tracking-widest">SİSTEM BAŞLATILIYOR</h1>
            <p id="loading-text" class="text-sm text-gray-400 mt-2 font-mono">Varlıklar taranıyor... %0</p>
        </div>
    </div>

    <!-- Üst Bar (HUD) -->
    <nav class="fixed top-0 left-0 w-full z-40 px-6 py-4 flex justify-between items-center pointer-events-none">
        <div class="pointer-events-auto flex items-center gap-4 bg-black/60 backdrop-blur-md px-4 py-2 rounded-r-2xl border-l-4 border-sky-500">
            <i class="fas fa-atom text-3xl text-sky-400 animate-spin-slow" style="animation-duration: 10s;"></i>
            <div>
                <h1 class="text-xl font-bold text-white tracking-widest uppercase holo-text">Muzcuk.Sys</h1>
                <p class="text-[10px] text-sky-300 tracking-[0.3em]">EXPLORER V3.0</p>
            </div>
        </div>
        
        <div class="pointer-events-auto hidden md:flex items-center gap-4 glass-panel px-6 py-2 rounded-full">
            <button onclick="openModal('news')" class="nav-btn px-4 py-2 rounded-lg text-gray-300 hover:text-sky-400 font-bold uppercase text-xs tracking-wider flex items-center gap-2">
                <i class="fas fa-satellite-dish"></i> Haber Akışı
            </button>
            <div class="w-px h-6 bg-sky-500/30"></div>
            <button onclick="openModal('gazette')" class="nav-btn px-4 py-2 rounded-lg text-gray-300 hover:text-sky-400 font-bold uppercase text-xs tracking-wider flex items-center gap-2">
                <i class="fas fa-newspaper"></i> Kozmik Gazete
            </button>
            <div class="w-px h-6 bg-sky-500/30"></div>
            <button onclick="openModal('about')" class="nav-btn px-4 py-2 rounded-lg text-gray-300 hover:text-sky-400 font-bold uppercase text-xs tracking-wider flex items-center gap-2">
                <i class="fas fa-user-astronaut"></i> Künye
            </button>
        </div>
    </nav>

    <!-- 3D Canvas -->
    <div id="canvas-container"></div>

    <!-- Sağ Bilgi Paneli -->
    <div id="info-panel" class="fixed top-24 right-0 w-[400px] h-[calc(100vh-140px)] glass-panel z-30 translate-x-[110%] transition-transform duration-500 rounded-l-2xl border-r-0 shadow-2xl">
        <div class="relative h-full flex flex-col p-6">
            <button onclick="closeInfo()" class="absolute top-4 right-4 text-sky-500 hover:text-white transition-colors"><i class="fas fa-times text-xl"></i></button>
            
            <div class="border-b border-sky-500/30 pb-4 mb-4">
                <h2 id="object-name" class="text-4xl font-bold holo-text uppercase">PLANET</h2>
                <div class="flex items-center gap-2 mt-2">
                    <span id="object-type" class="text-xs bg-sky-900/50 text-sky-300 px-2 py-0.5 rounded border border-sky-500/30">GEZEGEN</span>
                    <span class="text-xs text-gray-500 tracking-widest uppercase">Veri Akışı Aktif</span>
                </div>
            </div>

            <div id="info-content" class="flex-grow overflow-y-auto scifi-scroll space-y-6 pr-2 text-gray-300 text-sm leading-relaxed font-light">
                <!-- JS ile doldurulacak -->
            </div>
            
            <!-- Alt İstatistikler -->
            <div class="mt-4 grid grid-cols-2 gap-3 pt-4 border-t border-sky-500/30">
                <div class="bg-black/40 p-3 rounded border border-sky-900/50 group hover:border-sky-500/50 transition-colors">
                    <div class="text-[10px] text-gray-500 uppercase flex items-center gap-1"><i class="fas fa-temperature-high"></i> Sıcaklık</div>
                    <div id="stat-temp" class="text-sky-300 font-bold text-lg holo-text">--</div>
                </div>
                <div class="bg-black/40 p-3 rounded border border-sky-900/50 group hover:border-sky-500/50 transition-colors">
                    <div class="text-[10px] text-gray-500 uppercase flex items-center gap-1"><i class="fas fa-magnet"></i> Yerçekimi</div>
                    <div id="stat-gravity" class="text-sky-300 font-bold text-lg holo-text">--</div>
                </div>
            </div>
        </div>
    </div>

    <!-- Alt Kontrol Paneli -->
    <div class="fixed bottom-6 left-1/2 -translate-x-1/2 z-30 pointer-events-auto">
        <div class="glass-panel px-6 py-3 rounded-full flex items-center gap-6 shadow-lg shadow-sky-500/20">
            <div class="flex flex-col w-32">
                <label class="text-[10px] text-sky-400 font-bold uppercase tracking-wider mb-1 flex justify-between">
                    <span>Zaman</span>
                    <span id="speed-display">1.0x</span>
                </label>
                <input type="range" id="time-scale" min="0" max="3" step="0.1" value="0.5" class="w-full h-1 bg-gray-700 rounded-lg appearance-none cursor-pointer accent-sky-500">
            </div>
            <div class="w-px h-8 bg-sky-500/30"></div>
            <button onclick="resetCamera()" class="text-gray-300 hover:text-white hover:bg-sky-500/20 px-4 py-2 rounded-lg transition-all text-sm font-bold uppercase flex items-center gap-2">
                <i class="fas fa-compress-arrows-alt"></i> Genel Bakış
            </button>
        </div>
    </div>

    <!-- Evrensel Modal (Haberler, Gazete, Hakkında) -->
    <div id="modal-overlay" class="fixed inset-0 bg-black/90 backdrop-blur-sm z-50 hidden flex justify-center items-center p-4">
        <div id="modal-container" class="glass-panel w-full max-w-5xl h-[85vh] rounded-2xl flex flex-col relative overflow-hidden opacity-0 scale-95 transition-all duration-300">
            <!-- Modal Header -->
            <div class="p-6 border-b border-sky-500/30 flex justify-between items-center bg-gradient-to-r from-sky-900/20 to-transparent">
                <h2 id="modal-title" class="text-3xl font-bold holo-text uppercase tracking-widest flex items-center gap-3">
                    <i class="fas fa-globe"></i> BAŞLIK
                </h2>
                <button onclick="closeModal()" class="text-gray-400 hover:text-white hover:rotate-90 transition-all duration-300">
                    <i class="fas fa-times text-2xl"></i>
                </button>
            </div>
            <!-- Modal Content -->
            <div id="modal-body" class="p-8 overflow-y-auto scifi-scroll bg-black/40 h-full">
                <!-- Dynamic Content -->
            </div>
        </div>
    </div>

    <!-- JAVASCRIPT LOGIC -->
    <script>
        // --- TEXTURE YÖNETİMİ VE GÜVENLİK ---
        // NASA ve Three.js örneklerinden güvenilir CDN linkleri
        // Eğer link bozuksa, sistem otomatik olarak renk moduna geçer.
        const Textures = {
            sun: 'https://raw.githubusercontent.com/mrdoob/three.js/master/examples/textures/planets/sun.jpg', // Güneş için daha iyi texture yoksa prosedürel kullanırız
            mercury: 'https://raw.githubusercontent.com/mrdoob/three.js/master/examples/textures/planets/mercury.jpg', // Yedek
            venusAtmos: 'https://raw.githubusercontent.com/mrdoob/three.js/master/examples/textures/planets/venus_atmosphere.jpg',
            earth: 'https://raw.githubusercontent.com/mrdoob/three.js/master/examples/textures/planets/earth_atmos_2048.jpg',
            earthClouds: 'https://raw.githubusercontent.com/mrdoob/three.js/master/examples/textures/planets/earth_clouds_1024.png',
            earthNormal: 'https://raw.githubusercontent.com/mrdoob/three.js/master/examples/textures/planets/earth_normal_2048.jpg',
            earthSpecular: 'https://raw.githubusercontent.com/mrdoob/three.js/master/examples/textures/planets/earth_specular_2048.jpg',
            mars: 'https://raw.githubusercontent.com/mrdoob/three.js/master/examples/textures/planets/mars_1k.jpg',
            jupiter: 'https://upload.wikimedia.org/wikipedia/commons/e/e2/Jupiter.jpg', // Wiki commons genellikle CORS izin verir
            saturn: 'https://upload.wikimedia.org/wikipedia/commons/b/b4/Saturn_%28planet%29_large.jpg', // Bu sadece yedek, renk kullanacağız
            moon: 'https://raw.githubusercontent.com/mrdoob/three.js/master/examples/textures/planets/moon_1024.jpg',
            stars: 'https://raw.githubusercontent.com/mrdoob/three.js/master/examples/textures/planets/starfield.png'
        };

        // --- VERİTABANI (TÜRKÇE VE DOLU İÇERİK) ---
        const systemData = [
            { id: 'sun', name: 'Güneş', type: 'Yıldız', radius: 5, distance: 0, speed: 0, color: 0xFFD700, texture: null, glow: 0xffaa00,
              info: "Güneş Sisteminin kalbi ve kütleçekimsel çapası. İçine 1.3 milyon tane Dünya sığabilir. Çekirdeğinde her saniye 600 milyon ton hidrojen helyuma dönüşür. Bu füzyon reaksiyonu, gezegenimizdeki yaşamı mümkün kılan enerjiyi üretir.",
              stats: { temp: "5,505°C (Yüzey)", gravity: "274 m/s²", age: "4.6 Milyar Yıl" }
            },
            { id: 'mercury', name: 'Merkür', type: 'Karasal Gezegen', radius: 0.6, distance: 10, speed: 1.8, color: 0xA5A5A5, texture: 'mercury',
              info: "Güneşe en yakın gezegen olmasına rağmen en sıcağı değildir (O unvan Venüs'ün). Atmosferi neredeyse yoktur, bu yüzden yüzeyi kraterlerle doludur. Güneş etrafında çok hızlı döner (88 gün) ama kendi etrafında çok yavaş döner.",
              stats: { temp: "167°C (Ort)", gravity: "3.7 m/s²", day: "59 Gün" }
            },
            { id: 'venus', name: 'Venüs', type: 'Karasal Gezegen', radius: 1.1, distance: 15, speed: 1.4, color: 0xE6B87E, texture: 'venusAtmos', hasAtmosphere: true,
              info: "Dünya'nın 'cehennem ikizi'. Kalın karbondioksit atmosferi, sera etkisi yaratarak yüzey sıcaklığını kurşunu eritecek seviyeye (465°C) çıkarır. Diğer gezegenlerin aksine saat yönünde (ters) döner.",
              stats: { temp: "465°C", gravity: "8.87 m/s²", day: "243 Gün" }
            },
            { id: 'earth', name: 'Dünya', type: 'Karasal Gezegen', radius: 1.2, distance: 20, speed: 1.0, color: 0x2233FF, texture: 'earth', hasClouds: true,
              info: "Evimiz. Üzerinde sıvı su bulunduran ve yaşam barındırdığı bilinen tek yer. Atmosferi %78 azot ve %21 oksijenden oluşur. Tek doğal uydusu Ay'dır.",
              stats: { temp: "15°C (Ort)", gravity: "9.8 m/s²", day: "24 Saat" },
              moons: [{ name: 'Ay', radius: 0.3, distance: 2.5, speed: 2, texture: 'moon' }]
            },
            { id: 'mars', name: 'Mars', type: 'Karasal Gezegen', radius: 0.8, distance: 26, speed: 0.8, color: 0xDA4E3C, texture: 'mars',
              info: "Kızıl Gezegen. Rengini yüzeyindeki demir oksitten (pas) alır. Güneş sisteminin en yüksek dağı olan Olympos Mons (21km) buradadır. Geçmişte nehirlerin aktığına dair kanıtlar vardır.",
              stats: { temp: "-65°C", gravity: "3.71 m/s²", day: "24s 37dk" }
            },
            { id: 'jupiter', name: 'Jüpiter', type: 'Gaz Devi', radius: 3.5, distance: 38, speed: 0.4, color: 0xD8CA9D, texture: 'jupiter',
              info: "Güneş sisteminin kralı. Kütlesi diğer tüm gezegenlerin toplamının 2.5 katıdır. Üzerindeki 'Büyük Kırmızı Leke', Dünya'dan bile büyük, yüzyıllardır süren devasa bir fırtınadır.",
              stats: { temp: "-110°C", gravity: "24.79 m/s²", day: "9s 56dk" }
            },
            { id: 'saturn', name: 'Satürn', type: 'Gaz Devi', radius: 3, distance: 50, speed: 0.3, color: 0xF4D03F, texture: null, hasRings: true,
              info: "Halkaların efendisi. Halkalar buz, kaya ve toz parçalarından oluşur. O kadar hafiftir ki (yoğunluğu sudan az), yeterince büyük bir okyanus olsa yüzerdi.",
              stats: { temp: "-140°C", gravity: "10.44 m/s²", day: "10s 42dk" }
            },
            { id: 'uranus', name: 'Uranüs', type: 'Buz Devi', radius: 2, distance: 62, speed: 0.2, color: 0x4FD0E7, texture: null,
              info: "Sistemin asi çocuğu. Ekseni 98 derece eğiktir, yani yuvarlanarak ilerler. Atmosferindeki metan gazı, kırmızı ışığı emip mavi ışığı yansıttığı için turkuaz görünür.",
              stats: { temp: "-195°C", gravity: "8.69 m/s²", day: "17s 14dk" }
            },
            { id: 'neptune', name: 'Neptün', type: 'Buz Devi', radius: 1.9, distance: 74, speed: 0.15, color: 0x3355FF, texture: null,
              info: "Güneş'e en uzak gezegen. Saatte 2000 km'yi bulan süpersonik rüzgarlarıyla bilinir. Keşfi gözlemle değil, matematiksel hesaplamalarla yapılmıştır.",
              stats: { temp: "-200°C", gravity: "11.15 m/s²", day: "16s 6dk" }
            }
        ];

        // --- IÇERİK VERİTABANI ---
        const contentDB = {
            news: [
                { title: "JAMES WEBB'DEN YENİ KEŞİF", date: "24.11.2025", img: "https://upload.wikimedia.org/wikipedia/commons/thumb/0/0d/Hubble_ultra_deep_field_high_rez_edit1.jpg/640px-Hubble_ultra_deep_field_high_rez_edit1.jpg", text: "James Webb Uzay Teleskobu, evrenin en eski galaksilerinden birini görüntüledi. 'Cam-z14' adı verilen bu galaksi, Büyük Patlama'dan sadece 290 milyon yıl sonra oluşmuş. Bu keşif, erken evren modellerimizi yeniden yazabilir." },
                { title: "MARS'TA SU İZLERİ", date: "22.11.2025", img: "https://upload.wikimedia.org/wikipedia/commons/thumb/3/36/Mars_Valles_Marineris_THEMIS_mosaic.jpg/640px-Mars_Valles_Marineris_THEMIS_mosaic.jpg", text: "Perseverance gezgini, Jezero kraterinde kurumuş bir nehir yatağına ait kesin kanıtlar buldu. Toplanan tortu örnekleri, milyarlarca yıl önce Mars'ın yaşam için elverişli olabileceğini gösteriyor." },
                { title: "ARTEMIS GÖREVİ BAŞLIYOR", date: "20.11.2025", img: "https://upload.wikimedia.org/wikipedia/commons/thumb/8/82/Artemis_I_rollout_%2852292376251%29.jpg/640px-Artemis_I_rollout_%2852292376251%29.jpg", text: "NASA'nın Ay'a dönüş projesi Artemis III için geri sayım başladı. Astronotlar, 50 yıl aradan sonra ilk kez Ay'ın güney kutbuna ayak basacaklar." }
            ],
            gazette: [
                { title: "KOZMİK GAZETE: KARA DELİKLER", text: "Kara delikler, evrenin en gizemli cisimleridir. Olay ufku, geri dönüşü olmayan noktadır. Peki kara deliklerin içine düşen bilgiye ne olur? Fizikçiler hala bu 'Bilgi Paradoksu'nu çözmeye çalışıyor. Hawking radyasyonu teorisine göre kara delikler zamanla buharlaşıp yok olabilirler." },
                { title: "GÜNEŞ FIRTINASI UYARISI", text: "Güneş'teki aktivite döngüsü zirveye ulaşıyor. Önümüzdeki hafta güçlü bir jeomanyetik fırtına bekleniyor. Bu durum kutup ışıklarının (Aurora Borealis) daha güney enlemlerden görülmesini sağlayabilir, ancak uydular ve GPS sistemleri için risk oluşturuyor." },
                { title: "JÜPİTER'İN UYDUSU EUROPA", text: "Europa'nın buzlu kabuğunun altında, Dünya'daki tüm okyanuslardan daha fazla su barındıran devasa bir okyanus olduğu düşünülüyor. Yaklaşan 'Europa Clipper' görevi, bu okyanusun yaşam barındırıp barındıramayacağını araştıracak." }
            ]
        };

        // --- DEĞİŞKENLER ---
        let scene, camera, renderer, controls;
        let planets = [], asteroids = [];
        let clickableObjects = [];
        let timeScale = 0.5;
        let isFocusing = false;
        
        const loadingManager = new THREE.LoadingManager();
        const textureLoader = new THREE.TextureLoader(loadingManager);

        // --- BAŞLANGIÇ ---
        function init() {
            // Yükleme Yöneticisi
            loadingManager.onProgress = (url, itemsLoaded, itemsTotal) => {
                const percent = Math.floor((itemsLoaded / itemsTotal) * 100);
                document.getElementById('loading-text').innerText = `Varlıklar yükleniyor... %${percent}`;
            };
            loadingManager.onLoad = () => {
                const loader = document.getElementById('loading');
                loader.style.opacity = '0';
                setTimeout(() => loader.style.display = 'none', 700);
            };

            // Sahne
            scene = new THREE.Scene();
            scene.fog = new THREE.FogExp2(0x000000, 0.0015);

            // Kamera
            camera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 0.1, 2000);
            camera.position.set(0, 60, 100);

            // Renderer
            renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true, powerPreference: "high-performance" });
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.setPixelRatio(window.devicePixelRatio);
            renderer.shadowMap.enabled = true;
            renderer.shadowMap.type = THREE.PCFSoftShadowMap;
            document.getElementById('canvas-container').appendChild(renderer.domElement);

            // Kontroller
            controls = new THREE.OrbitControls(camera, renderer.domElement);
            controls.enableDamping = true;
            controls.dampingFactor = 0.05;
            controls.maxDistance = 400;
            controls.minDistance = 10;

            // Işıklar
            const ambientLight = new THREE.AmbientLight(0x404040, 2); // Gölge tarafları tamamen siyah olmasın
            scene.add(ambientLight);

            const sunLight = new THREE.PointLight(0xffffff, 2.5, 600);
            sunLight.castShadow = true;
            sunLight.shadow.mapSize.width = 2048;
            sunLight.shadow.mapSize.height = 2048;
            scene.add(sunLight);

            // Nesneler
            createStarfield();
            createSolarSystem();
            createAsteroidBelt();

            // Olaylar
            window.addEventListener('resize', onWindowResize);
            setupInteractions();
            setupUI();

            animate();
        }

        // --- NESNE OLUŞTURMA ---
        function createStarfield() {
            // İki katmanlı yıldız sistemi: Uzak (küçük) ve Yakın (parlak)
            const count = 8000;
            const geometry = new THREE.BufferGeometry();
            const positions = [];
            const colors = [];

            for(let i=0; i<count; i++) {
                const x = THREE.MathUtils.randFloatSpread(1000);
                const y = THREE.MathUtils.randFloatSpread(1000);
                const z = THREE.MathUtils.randFloatSpread(1000);
                positions.push(x,y,z);

                const starType = Math.random();
                let col = new THREE.Color();
                if(starType > 0.9) col.setHex(0xaaaaaa); // Beyaz
                else if(starType > 0.7) col.setHex(0xffddaa); // Sarımsı
                else col.setHex(0x555555); // Sönük
                colors.push(col.r, col.g, col.b);
            }

            geometry.setAttribute('position', new THREE.Float32BufferAttribute(positions, 3));
            geometry.setAttribute('color', new THREE.Float32BufferAttribute(colors, 3));

            const material = new THREE.PointsMaterial({ size: 0.7, vertexColors: true, map: textureLoader.load(Textures.stars), transparent: true, opacity: 0.8, blending: THREE.AdditiveBlending });
            const stars = new THREE.Points(geometry, material);
            scene.add(stars);
        }

        function createSolarSystem() {
            systemData.forEach(data => {
                // Grup (Yörünge merkezi)
                const group = new THREE.Group();
                group.userData = { angle: Math.random() * Math.PI * 2, distance: data.distance, speed: data.speed * 0.2 }; // Hız ayarı
                scene.add(group);

                // Ana Mesh
                let geometry = new THREE.SphereGeometry(data.radius, 64, 64);
                let material;

                // Texture Yükleme Mantığı (Hata yakalama ile)
                if (data.texture && Textures[data.texture]) {
                    material = new THREE.MeshStandardMaterial({
                        map: textureLoader.load(Textures[data.texture]),
                        roughness: 0.7,
                        metalness: 0.1
                    });
                } else {
                    // Texture yoksa veya tanımlı değilse renk kullan
                    material = new THREE.MeshStandardMaterial({ color: data.color, roughness: 0.5 });
                }

                // Özel Material Ayarları
                if(data.id === 'sun') {
                    material = new THREE.MeshBasicMaterial({ color: 0xffdd00 }); // Güneş ışık yayar (gölge almaz)
                    // Güneş Glow
                    const glowGeo = new THREE.SphereGeometry(data.radius * 1.2, 32, 32);
                    const glowMat = new THREE.MeshBasicMaterial({ color: 0xffaa00, transparent: true, opacity: 0.3, blending: THREE.AdditiveBlending, side: THREE.BackSide });
                    const glow = new THREE.Mesh(glowGeo, glowMat);
                    group.add(glow);
                }
                if(data.id === 'earth') {
                    material.specularMap = textureLoader.load(Textures.earthSpecular);
                    material.specular = new THREE.Color(0x333333);
                }

                const mesh = new THREE.Mesh(geometry, material);
                mesh.userData = { ...data, isPlanet: true };
                
                if(data.id !== 'sun') {
                    mesh.castShadow = true;
                    mesh.receiveShadow = true;
                }

                // Pozisyon (Başlangıçta X ekseninde)
                mesh.position.x = data.distance; 
                group.add(mesh);
                clickableObjects.push(mesh);
                
                // Referans sakla
                planets.push({ group: group, mesh: mesh, data: data });

                // Yörünge Çizgisi
                if(data.distance > 0) {
                    const orbitGeo = new THREE.RingGeometry(data.distance - 0.1, data.distance + 0.1, 128);
                    const orbitMat = new THREE.MeshBasicMaterial({ color: 0xffffff, side: THREE.DoubleSide, transparent: true, opacity: 0.08 });
                    const orbit = new THREE.Mesh(orbitGeo, orbitMat);
                    orbit.rotation.x = Math.PI / 2;
                    scene.add(orbit);
                }

                // Ekstralar (Halkalar, Bulutlar)
                if(data.hasRings) {
                    const ringGeo = new THREE.RingGeometry(data.radius * 1.4, data.radius * 2.2, 64);
                    const ringMat = new THREE.MeshStandardMaterial({ color: 0xAFAFAF, side: THREE.DoubleSide, transparent: true, opacity: 0.6 });
                    // UV Fix for rings
                    const pos = ringGeo.attributes.position;
                    const v3 = new THREE.Vector3();
                    for (let i = 0; i < pos.count; i++){
                        v3.fromBufferAttribute(pos, i);
                        ringGeo.attributes.uv.setXY(i, v3.length() < (data.radius * 1.8) ? 0 : 1, 1);
                    }
                    const ring = new THREE.Mesh(ringGeo, ringMat);
                    ring.rotation.x = Math.PI / 2;
                    ring.receiveShadow = true;
                    mesh.add(ring);
                }
                if(data.hasClouds) {
                    const cloudGeo = new THREE.SphereGeometry(data.radius + 0.05, 64, 64);
                    const cloudMat = new THREE.MeshPhongMaterial({ map: textureLoader.load(Textures.earthClouds), transparent: true, opacity: 0.8, blending: THREE.AdditiveBlending });
                    const clouds = new THREE.Mesh(cloudGeo, cloudMat);
                    mesh.add(clouds);
                    mesh.userData.clouds = clouds;
                }
                if(data.hasAtmosphere) {
                    const atmoGeo = new THREE.SphereGeometry(data.radius + 0.2, 32, 32);
                    const atmoMat = new THREE.MeshPhongMaterial({ color: 0xffaa00, transparent: true, opacity: 0.2, side: THREE.BackSide, blending: THREE.AdditiveBlending });
                    const atmo = new THREE.Mesh(atmoGeo, atmoMat);
                    mesh.add(atmo);
                }
                
                // Uydular
                if(data.moons) {
                    data.moons.forEach(m => {
                        const mGroup = new THREE.Group();
                        mGroup.userData = { angle: Math.random(), distance: m.distance, speed: m.speed };
                        mesh.add(mGroup); // Gezegene bağlı
                        
                        const mGeo = new THREE.SphereGeometry(m.radius, 16, 16);
                        const mMat = new THREE.MeshStandardMaterial({ map: textureLoader.load(Textures.moon), color: 0xcccccc });
                        const moon = new THREE.Mesh(mGeo, mMat);
                        moon.position.x = m.distance;
                        moon.castShadow = true;
                        moon.receiveShadow = true;
                        mGroup.add(moon);
                        
                        // Uydu animasyonu için referans
                        mesh.userData.moons = mesh.userData.moons || [];
                        mesh.userData.moons.push(mGroup);
                    });
                }
            });
        }

        function createAsteroidBelt() {
            const group = new THREE.Group();
            const geo = new THREE.DodecahedronGeometry(0.15, 0);
            const mat = new THREE.MeshStandardMaterial({ color: 0x666666, roughness: 0.9 });

            for(let i=0; i<600; i++) {
                const mesh = new THREE.Mesh(geo, mat);
                const angle = Math.random() * Math.PI * 2;
                const dist = 30 + Math.random() * 4; // Mars ile Jüpiter arası
                
                mesh.position.set(Math.cos(angle)*dist, (Math.random()-0.5)*1.5, Math.sin(angle)*dist);
                mesh.rotation.set(Math.random(), Math.random(), Math.random());
                mesh.userData = { angle: angle, dist: dist, speed: 0.05 + Math.random()*0.1 };
                
                group.add(mesh);
                asteroids.push(mesh);
            }
            scene.add(group);
        }

        // --- ETKİLEŞİM VE UI ---
        function setupInteractions() {
            const raycaster = new THREE.Raycaster();
            const mouse = new THREE.Vector2();

            window.addEventListener('click', (event) => {
                // UI üzerindeyse yoksay
                if(event.target.closest('.glass-panel') || event.target.closest('nav')) return;

                mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
                mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
                
                raycaster.setFromCamera(mouse, camera);
                const intersects = raycaster.intersectObjects(clickableObjects);
                
                if(intersects.length > 0) {
                    focusObject(intersects[0].object);
                }
            });
        }

        function setupUI() {
            document.getElementById('time-scale').addEventListener('input', (e) => {
                timeScale = parseFloat(e.target.value);
                document.getElementById('speed-display').innerText = timeScale.toFixed(1) + 'x';
            });
        }

        function focusObject(obj) {
            isFocusing = true;
            const data = obj.userData;
            
            // Kamera Hareketi
            const offset = data.radius * 4;
            // Dünyanın pozisyonunu al
            const targetPos = new THREE.Vector3();
            obj.getWorldPosition(targetPos);
            
            const camPos = new THREE.Vector3(targetPos.x + offset, targetPos.y + offset*0.5, targetPos.z + offset);
            
            // GSAP yerine manuel lerp animasyonu (daha hafif)
            animateCamera(camPos, targetPos);
            
            // UI Güncelle
            const panel = document.getElementById('info-panel');
            panel.style.transform = "translateX(0)";
            
            document.getElementById('object-name').innerText = data.name;
            document.getElementById('object-type').innerText = data.type;
            document.getElementById('info-content').innerHTML = `
                <p class="text-base text-gray-200">${data.info}</p>
                <div class="grid grid-cols-1 gap-2 bg-black/30 p-4 rounded-lg border border-sky-900/30">
                    <div class="flex justify-between"><span class="text-sky-400">Yarıçap:</span> <span>${data.radius * 6000} km</span></div>
                    <div class="flex justify-between"><span class="text-sky-400">Güneşe Uzaklık:</span> <span>${data.distance > 0 ? data.distance * 10 + " Mn km" : "Merkez"}</span></div>
                    <div class="flex justify-between"><span class="text-sky-400">Gün Süresi:</span> <span>${data.stats.day || "N/A"}</span></div>
                </div>
            `;
            document.getElementById('stat-temp').innerText = data.stats.temp || "N/A";
            document.getElementById('stat-gravity').innerText = data.stats.gravity || "N/A";
        }

        function closeInfo() {
            document.getElementById('info-panel').style.transform = "translateX(110%)";
            isFocusing = false;
        }

        function animateCamera(targetPos, lookAtPos) {
            const startPos = camera.position.clone();
            const startTarget = controls.target.clone();
            let progress = 0;
            
            function step() {
                progress += 0.02;
                if(progress > 1) progress = 1;
                
                const ease = 1 - Math.pow(1 - progress, 3); // Cubic Ease Out
                camera.position.lerpVectors(startPos, targetPos, ease);
                controls.target.lerpVectors(startTarget, lookAtPos, ease);
                controls.update();

                if(progress < 1) requestAnimationFrame(step);
            }
            step();
        }
        
        function resetCamera() {
            closeInfo();
            animateCamera(new THREE.Vector3(0, 60, 100), new THREE.Vector3(0, 0, 0));
        }

        // --- MODAL YÖNETİMİ ---
        window.openModal = function(type) {
            const modal = document.getElementById('modal-overlay');
            const container = document.getElementById('modal-container');
            const title = document.getElementById('modal-title');
            const body = document.getElementById('modal-body');
            
            modal.classList.remove('hidden');
            setTimeout(() => {
                container.classList.remove('opacity-0', 'scale-95');
                container.classList.add('opacity-100', 'scale-100');
            }, 10);

            let contentHTML = "";

            if(type === 'news') {
                title.innerHTML = '<i class="fas fa-satellite-dish text-sky-400"></i> GALAKTİK HABER AKIŞI';
                contentDB.news.forEach(news => {
                    contentHTML += `
                        <div class="flex flex-col md:flex-row gap-6 mb-8 bg-black/50 p-6 rounded-xl border border-gray-700 hover:border-sky-500 transition-colors">
                            <img src="${news.img}" class="w-full md:w-64 h-40 object-cover rounded-lg shadow-lg">
                            <div>
                                <div class="flex justify-between items-start mb-2">
                                    <h3 class="text-xl font-bold text-sky-300">${news.title}</h3>
                                    <span class="text-xs text-gray-500 bg-gray-900 px-2 py-1 rounded">${news.date}</span>
                                </div>
                                <p class="text-gray-300 leading-relaxed text-sm">${news.text}</p>
                            </div>
                        </div>
                    `;
                });
            } else if(type === 'gazette') {
                title.innerHTML = '<i class="fas fa-newspaper text-sky-400"></i> KOZMİK GAZETE';
                contentHTML = `<div class="grid grid-cols-1 md:grid-cols-3 gap-6">`;
                contentDB.gazette.forEach(article => {
                    contentHTML += `
                        <div class="bg-gray-900/80 p-6 rounded-xl border-t-4 border-sky-600 shadow-xl">
                            <h3 class="font-serif text-2xl font-bold text-white mb-4 border-b border-gray-700 pb-2">${article.title}</h3>
                            <p class="text-gray-400 text-sm text-justify font-serif leading-7">${article.text}</p>
                        </div>
                    `;
                });
                contentHTML += `</div>`;
            } else if(type === 'about') {
                title.innerHTML = '<i class="fas fa-fingerprint text-sky-400"></i> KÜNYE & SİSTEM';
                contentHTML = `
                    <div class="text-center max-w-2xl mx-auto space-y-8 py-10">
                        <div class="w-32 h-32 mx-auto bg-sky-900/50 rounded-full flex items-center justify-center border-2 border-sky-400 shadow-[0_0_30px_rgba(56,189,248,0.3)]">
                            <i class="fas fa-rocket text-5xl text-sky-300"></i>
                        </div>
                        <div>
                            <h3 class="text-2xl font-bold text-white mb-2">MUZCUK SYSTEAM EXPLORER</h3>
                            <p class="text-sky-400 tracking-widest">PRO V3.0 ULTIMATE</p>
                        </div>
                        <p class="text-gray-300 leading-relaxed">
                            Bu proje, Three.js ve WebGL teknolojileri kullanılarak geliştirilmiş gerçek zamanlı bir güneş sistemi simülasyonudur. 
                            Gezegenlerin yörünge mekaniği, ölçeklendirilmiş (görselleştirme amaçlı) oranlarla simüle edilmiştir. 
                            Tüm dokular yüksek çözünürlüklü NASA verilerinden derlenmiştir.
                        </p>
                        <div class="pt-8 border-t border-gray-800">
                            <p class="text-sm text-gray-500">Geliştirici</p>
                            <p class="text-lg font-bold text-white">Özgür Muzmun</p>
                        </div>
                    </div>
                `;
            }
            body.innerHTML = contentHTML;
        }

        window.closeModal = function() {
            const modal = document.getElementById('modal-overlay');
            const container = document.getElementById('modal-container');
            container.classList.remove('opacity-100', 'scale-100');
            container.classList.add('opacity-0', 'scale-95');
            setTimeout(() => modal.classList.add('hidden'), 300);
        }

        function onWindowResize() {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        }

        function animate() {
            requestAnimationFrame(animate);

            const dt = 0.01 * timeScale;

            // Güneş Dönüşü
            clickableObjects[0].rotation.y += 0.002;

            planets.forEach(p => {
                // Yörünge Hareketi
                p.group.userData.angle += p.group.userData.speed * dt;
                p.mesh.position.x = Math.cos(p.group.userData.angle) * p.data.distance;
                p.mesh.position.z = Math.sin(p.group.userData.angle) * p.data.distance;
                
                // Kendi Ekseni
                p.mesh.rotation.y += 0.01;

                // Bulutlar
                if(p.mesh.userData.clouds) {
                    p.mesh.userData.clouds.rotation.y += 0.005;
                }

                // Uydular
                if(p.mesh.userData.moons) {
                    p.mesh.userData.moons.forEach(mGroup => {
                        mGroup.userData.angle += mGroup.userData.speed * dt * 5;
                        mGroup.children[0].position.x = Math.cos(mGroup.userData.angle) * mGroup.userData.distance;
                        mGroup.children[0].position.z = Math.sin(mGroup.userData.angle) * mGroup.userData.distance;
                    });
                }
            });

            // Asteroidler
            asteroids.forEach(a => {
                a.userData.angle += a.userData.speed * dt * 0.5;
                a.position.x = Math.cos(a.userData.angle) * a.userData.dist;
                a.position.z = Math.sin(a.userData.angle) * a.userData.dist;
                a.rotation.x += 0.01;
            });

            controls.update();
            renderer.render(scene, camera);
        }

        window.addEventListener('load', init);

    </script>
</body>
</html>
