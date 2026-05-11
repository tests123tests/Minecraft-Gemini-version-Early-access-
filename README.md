```html
<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Minecraft Clone JS - 60 FPS Edition</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/simplex-noise@2.4.0/simplex-noise.min.js"></script>
    <style>
        body { margin: 0; overflow: hidden; font-family: 'Segoe UI', sans-serif; touch-action: none; background: #000; }
        canvas { display: block; }
        
        #ui-container {
            position: absolute;
            top: 0; left: 0; width: 100%; height: 100%;
            pointer-events: none;
            display: block;
        }

        #crosshair {
            position: absolute;
            top: 50%; left: 50%; width: 16px; height: 16px;
            border: 2px solid white; transform: translate(-50%, -50%);
            mix-blend-mode: difference;
        }

        #main-menu {
            position: absolute;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.85);
            display: flex; flex-direction: column; align-items: center; justify-content: center;
            z-index: 100; pointer-events: auto;
        }
        .menu-btn {
            width: 200px; padding: 15px; margin: 5px;
            background: #555; color: white; border: 2px solid #fff;
            text-align: center; cursor: pointer; font-weight: bold;
        }
        .menu-btn:active { background: #888; }

        #joystick-container { position: absolute; bottom: 160px; left: 30px; width: 90px; height: 90px; background: rgba(255,255,255,0.1); border-radius: 50%; pointer-events: auto; border: 1px solid rgba(255,255,255,0.3); }
        #joystick-knob { position: absolute; top: 25px; left: 25px; width: 40px; height: 40px; background: rgba(255, 255, 255, 0.4); border-radius: 50%; }

        #action-buttons { position: absolute; bottom: 160px; right: 30px; display: flex; flex-direction: column; gap: 10px; pointer-events: auto; }
        .btn-action { width: 60px; height: 60px; background: rgba(0, 0, 0, 0.5); border: 1px solid white; color: white; border-radius: 10px; display: flex; align-items: center; justify-content: center; font-size: 10px; font-weight: bold; }

        #btn-pause { position: absolute; top: 20px; left: 50%; transform: translateX(-50%); pointer-events: auto; background: rgba(255, 0, 0, 0.7); color: white; padding: 10px 20px; border: 2px solid white; font-weight: bold; cursor: pointer; z-index: 120; }
        #btn-menu-open { position: absolute; top: 20px; left: 20px; pointer-events: auto; background: rgba(0,0,0,0.5); color: white; padding: 10px; border: 1px solid white; }

        #inventory-btn { position: absolute; top: 20px; right: 80px; pointer-events: auto; background: rgba(0,0,0,0.7); padding: 8px 15px; color: white; border: 1px solid white; cursor: pointer; }
        #inventory-panel {
            position: absolute;
            top: 50%; left: 50%; transform: translate(-50%, -50%);
            width: 300px; background: rgba(0,0,0,0.95); border: 2px solid #555;
            display: none; flex-direction: column; padding: 15px; gap: 10px; pointer-events: auto; z-index: 110;
        }
        #inventory-grid { display: grid; grid-template-columns: repeat(4, 1fr); gap: 8px; }
        .inv-slot { height: 60px; border: 1px solid #777; display: flex; flex-direction: column; align-items: center; justify-content: center; color: white; font-size: 10px; cursor: pointer; }
        .inv-slot.selected { border: 2px solid yellow; background: rgba(255, 255, 0, 0.1); }
        
        #craft-table-btn { width: 100%; padding: 10px; background: #795548; color: white; border: 1px solid white; font-weight: bold; display: none; margin-top: 10px; }

        #crafting-menu {
            position: absolute;
            top: 50%; left: 50%; transform: translate(-50%, -50%);
            width: 250px; background: rgba(40,40,40,0.98); border: 3px solid #5d4037;
            display: none; flex-direction: column; padding: 20px; gap: 15px; pointer-events: auto; z-index: 150;
        }
        .craft-item-btn { padding: 12px; background: #555; color: white; border: 1px solid #fff; text-align: center; cursor: pointer; font-size: 12px; }

        #hotbar { position: absolute; bottom: 20px; left: 50%; transform: translateX(-50%); display: flex; gap: 5px; background: rgba(0,0,0,0.6); padding: 5px; border: 2px solid #333; pointer-events: auto; }
        .hotbar-slot { width: 45px; height: 45px; border: 2px solid #555; background: rgba(255,255,255,0.1); display: flex; align-items: center; justify-content: center; position: relative; cursor: pointer; }
        .hotbar-slot.active { border-color: #fff; background: rgba(255,255,255,0.2); }
        .hotbar-icon { width: 25px; height: 25px; }
        .hotbar-count { position: absolute; bottom: 1px; right: 3px; color: white; font-size: 10px; text-shadow: 1px 1px black; }

        #btn-open-crafting { position: absolute; bottom: 80px; left: 50%; transform: translateX(-50%); padding: 10px 20px; background: #4caf50; color: white; border: 2px solid white; display: none; pointer-events: auto; }
    </style>
</head>
<body>

    <div id="main-menu">
        <h1 style="color: white; margin-bottom: 20px;">MINECRAFT DROPS</h1>
        <div class="menu-btn" id="resume-btn">JOGAR</div>
        <div class="menu-btn" id="respawn-btn">RESPAWN</div>
        <div class="menu-btn" id="texture-btn">ADICIONAR TEXTURAS</div>
    </div>

    <div id="inventory-panel">
        <div id="inventory-grid"></div>
        <button id="craft-table-btn">FAZER CRAFTING TABLE</button>
    </div>

    <div id="crafting-menu">
        <h3 style="color: white; margin: 0 0 10px 0; text-align: center;">CRAFTING</h3>
        <div class="craft-item-btn" id="craft-pickaxe">PICARETA DE MADEIRA (5 Mad.)</div>
        <div class="craft-item-btn" id="craft-axe">MACHADO DE MADEIRA (5 Mad.)</div>
        <button onclick="document.getElementById('crafting-menu').style.display='none'" style="margin-top: 10px;">FECHAR</button>
    </div>

    <div id="ui-container">
        <div id="btn-pause">PAUSAR</div>
        <div id="btn-menu-open">MENU</div>
        <div id="inventory-btn">INV</div>
        <div id="crosshair"></div>
        <div id="joystick-container"><div id="joystick-knob"></div></div>
        <div id="action-buttons">
            <div class="btn-action" id="btn-jump">PULAR</div>
            <div class="btn-action" id="btn-break">QUEBRAR</div>
            <div class="btn-action" id="btn-place">COLOCAR</div>
        </div>
        <button id="btn-open-crafting">ABRIR</button>
        <div id="hotbar"></div>
    </div>

    <script>
        // Configurações básicas
        const blockSize = 1;
        const worldSize = 40; 
        const worldDepth = 12; 
        let selectedSlot = 0;
        let selectedInvIndex = -1;
        let useTextures = false;
        let gameActive = false;

        // Dados dos blocos
        const blockData = {
            grass: { color: 0x4caf50, label: 'Grama' },
            dirt: { color: 0x8b4513, label: 'Terra' },
            stone: { color: 0x808080, label: 'Pedra' },
            wood: { color: 0x5d4037, label: 'Madeira' },
            leaves: { color: 0x2e7d32, label: 'Folhas' },
            sand: { color: 0xf4a460, label: 'Areia' },
            water: { color: 0x2196f3, label: 'Água' },
            crafting_table: { color: 0xbc8f8f, label: 'Bancada' },
            pickaxe: { color: 0xdddddd, label: 'Picareta' },
            axe: { color: 0xcccccc, label: 'Machado' }
        };

        let inventory = [
            { type: 'grass', count: 10 },
            { type: 'wood', count: 0 },
            { type: 'stone', count: 0 },
            { type: 'dirt', count: 0 },
            { type: 'sand', count: 0 }
        ];

        // Inicialização Three.js otimizada para 60 FPS
        const scene = new THREE.Scene();
        scene.background = new THREE.Color(0x87ceeb);
        scene.fog = new THREE.Fog(0x87ceeb, 20, 60);

        const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 100);
        const renderer = new THREE.WebGLRenderer({ antialias: false, powerPreference: "high-performance" });
        renderer.setPixelRatio(window.devicePixelRatio);
        renderer.setSize(window.innerWidth, window.innerHeight);
        document.body.appendChild(renderer.domElement);

        scene.add(new THREE.AmbientLight(0xffffff, 0.9));
        const sun = new THREE.DirectionalLight(0xffffff, 0.3);
        sun.position.set(5, 15, 5);
        scene.add(sun);

        const player = {
            height: 2.5,
            pos: new THREE.Vector3(0, 20, 0),
            vel: new THREE.Vector3(),
            onGround: false,
            speed: 0.14,
            lookSensitivity: 0.005
        };

        const blocks = [];
        const drops = [];
        const simplex = new SimplexNoise();
        const cubeGeo = new THREE.BoxGeometry(blockSize, blockSize, blockSize);

        function generateTexture(color) {
            const canvas = document.createElement('canvas');
            canvas.width = 16; canvas.height = 16;
            const ctx = canvas.getContext('2d');
            ctx.fillStyle = `#${color.toString(16).padStart(6, '0')}`;
            ctx.fillRect(0, 0, 16, 16);
            ctx.fillStyle = 'rgba(0,0,0,0.1)';
            for(let i=0; i<8; i++) ctx.fillRect(Math.random()*16, Math.random()*16, 2, 2);
            const tex = new THREE.CanvasTexture(canvas);
            tex.magFilter = THREE.NearestFilter;
            return tex;
        }

        function createBlock(x, y, z, type) {
            const matConfig = { color: blockData[type].color };
            if(useTextures) matConfig.map = generateTexture(blockData[type].color);
            if(type === 'water') { matConfig.transparent = true; matConfig.opacity = 0.5; }
            
            const material = new THREE.MeshLambertMaterial(matConfig);
            const mesh = new THREE.Mesh(cubeGeo, material);
            mesh.position.set(x, y, z);
            mesh.userData = { type, isWater: type === 'water' };
            scene.add(mesh);
            blocks.push(mesh);
        }

        function createDrop(x, y, z, type) {
            if (type === 'water') return;
            const dropGeo = new THREE.BoxGeometry(0.3, 0.3, 0.3);
            const material = new THREE.MeshLambertMaterial({ color: blockData[type].color });
            const drop = new THREE.Mesh(dropGeo, material);
            drop.position.set(x, y, z);
            drop.userData = { type };
            scene.add(drop);
            drops.push(drop);
        }

        function generateWorld() {
            blocks.forEach(b => scene.remove(b));
            blocks.length = 0;
            for (let x = -worldSize/2; x < worldSize/2; x++) {
                for (let z = -worldSize/2; z < worldSize/2; z++) {
                    const noiseH = simplex.noise2D(x/20, z/20);
                    const riverNoise = Math.abs(simplex.noise2D(x/30, z/30));
                    let h = Math.floor(noiseH * 5 + 8);
                    for (let y = -worldDepth; y <= Math.max(h, 4); y++) {
                        if (riverNoise < 0.08 && y <= 4 && y > 0) {
                            if (y === 4) createBlock(x, y, z, 'water');
                            else createBlock(x, y, z, 'sand');
                        } else if (y <= h) {
                            let type = y === h ? (riverNoise < 0.15 ? 'sand' : 'grass') : 'stone';
                            if (y < h && y > h-3 && type === 'grass') type = 'dirt';
                            if (y < -2) type = 'stone'; 
                            createBlock(x, y, z, type);
                        }
                    }
                    if (x % 10 === 0 && z % 10 === 0 && Math.random() > 0.5 && riverNoise > 0.2) {
                        for(let ty=1; ty<5; ty++) createBlock(x, h+ty, z, 'wood');
                        for(let lx=-1; lx<=1; lx++) {
                            for(let lz=-1; lz<=1; lz++) {
                                createBlock(x+lx, h+5, z+lz, 'leaves');
                            }
                        }
                    }
                }
            }
        }

        function addOrUpdateInventory(type, amount) {
            let item = inventory.find(i => i.type === type);
            if(item) item.count += amount;
            else inventory.push({ type, count: amount });
        }

        function renderInventory() {
            const grid = document.getElementById('inventory-grid');
            grid.innerHTML = '';
            inventory.forEach((item, i) => {
                const slot = document.createElement('div');
                slot.className = `inv-slot ${i === selectedInvIndex ? 'selected' : ''}`;
                slot.innerHTML = `<div style="width:20px; height:20px; background:#${blockData[item.type].color.toString(16).padStart(6,'0')}"></div>
                                  <span>${blockData[item.type].label}</span>
                                  <b>${item.count}</b>`;
                slot.onclick = (e) => { e.stopPropagation(); selectedInvIndex = i; renderInventory(); };
                grid.appendChild(slot);
            });
            const woodItem = inventory.find(i => i.type === 'wood');
            const craftBtn = document.getElementById('craft-table-btn');
            if(woodItem && woodItem.count >= 5) {
                craftBtn.style.display = 'block';
                craftBtn.onclick = () => { woodItem.count -= 5; addOrUpdateInventory('crafting_table', 1); renderInventory(); renderHotbar(); };
            } else craftBtn.style.display = 'none';
        }

        function renderHotbar() {
            const h = document.getElementById('hotbar');
            h.innerHTML = '';
            inventory.slice(0, 5).forEach((item, i) => {
                const s = document.createElement('div');
                s.className = `hotbar-slot ${i === selectedSlot ? 'active' : ''}`;
                if(item && item.count > 0) {
                    const icon = document.createElement('div');
                    icon.className = 'hotbar-icon';
                    icon.style.background = `#${blockData[item.type].color.toString(16).padStart(6, '0')}`;
                    s.appendChild(icon);
                    const count = document.createElement('div');
                    count.className = 'hotbar-count';
                    count.innerText = item.count;
                    s.appendChild(count);
                }
                s.onclick = (e) => { 
                    if(!gameActive) return;
                    e.stopPropagation();
                    if (selectedInvIndex !== -1) {
                        const temp = inventory[i];
                        inventory[i] = inventory[selectedInvIndex];
                        inventory[selectedInvIndex] = temp;
                        selectedInvIndex = -1;
                        renderInventory();
                    }
                    selectedSlot = i; renderHotbar(); checkCraftingContext(); 
                };
                h.appendChild(s);
            });
            checkCraftingContext();
        }

        function checkCraftingContext() {
            const currentItem = inventory[selectedSlot];
            const btn = document.getElementById('btn-open-crafting');
            if(currentItem && currentItem.type === 'crafting_table' && currentItem.count > 0) btn.style.display = 'block';
            else btn.style.display = 'none';
        }

        document.getElementById('btn-open-crafting').onclick = () => { document.getElementById('crafting-menu').style.display = 'flex'; };

        const craftItem = (resultType) => {
            const woodItem = inventory.find(i => i.type === 'wood');
            if(woodItem && woodItem.count >= 5) {
                woodItem.count -= 5;
                addOrUpdateInventory(resultType, 1);
                renderInventory(); renderHotbar();
                document.getElementById('crafting-menu').style.display = 'none';
            }
        };

        document.getElementById('craft-pickaxe').onclick = () => craftItem('pickaxe');
        document.getElementById('craft-axe').onclick = () => craftItem('axe');

        const menu = document.getElementById('main-menu');
        const ui = document.getElementById('ui-container');
        const btnPause = document.getElementById('btn-pause');
        
        function setGameActive(active) {
            gameActive = active;
            if(active) { btnPause.innerText = "PAUSAR"; btnPause.style.background = "rgba(255, 0, 0, 0.7)"; }
            else { btnPause.innerText = "RETOMAR"; btnPause.style.background = "rgba(0, 200, 0, 0.7)"; }
        }

        btnPause.onclick = () => setGameActive(!gameActive);
        document.getElementById('resume-btn').onclick = () => { menu.style.display = 'none'; ui.style.display = 'block'; setGameActive(true); };
        document.getElementById('respawn-btn').onclick = () => { player.pos.set(0, 20, 0); player.vel.set(0, 0, 0); menu.style.display = 'none'; ui.style.display = 'block'; setGameActive(true); };
        document.getElementById('texture-btn').onclick = () => { useTextures = !useTextures; generateWorld(); };
        document.getElementById('btn-menu-open').onclick = () => { menu.style.display = 'flex'; ui.style.display = 'none'; setGameActive(false); };
        document.getElementById('inventory-btn').onclick = () => {
            if(!gameActive) return;
            const p = document.getElementById('inventory-panel');
            const isOpen = p.style.display === 'flex';
            p.style.display = isOpen ? 'none' : 'flex';
            if(!isOpen) { selectedInvIndex = -1; renderInventory(); }
        };

        let touchX, touchY, yaw = 0, pitch = 0;
        window.addEventListener('touchstart', e => {
            if(!gameActive) return;
            const isUI = e.target.closest('#joystick-container') || e.target.closest('#action-buttons') || e.target.closest('#hotbar') || e.target.closest('#inventory-panel') || e.target.closest('#crafting-menu') || e.target.id === 'btn-menu-open' || e.target.id === 'inventory-btn' || e.target.id === 'btn-pause' || e.target.id === 'btn-open-crafting';
            if (!isUI) { touchX = e.touches[0].pageX; touchY = e.touches[0].pageY; }
        });
        window.addEventListener('touchmove', e => {
            if (!gameActive || touchX === undefined) return;
            const isUI = e.target.closest('#joystick-container') || e.target.closest('#action-buttons') || e.target.closest('#hotbar') || e.target.closest('#inventory-panel') || e.target.closest('#crafting-menu') || e.target.id === 'btn-menu-open' || e.target.id === 'inventory-btn' || e.target.id === 'btn-pause' || e.target.id === 'btn-open-crafting';
            if (isUI) return;
            yaw -= (e.touches[0].pageX - touchX) * player.lookSensitivity;
            pitch -= (e.touches[0].pageY - touchY) * player.lookSensitivity;
            pitch = Math.max(-1.4, Math.min(1.4, pitch));
            camera.rotation.set(pitch, yaw, 0, 'YXZ');
            touchX = e.touches[0].pageX; touchY = e.touches[0].pageY;
        });
        window.addEventListener('touchend', () => { touchX = undefined; });

        const knob = document.getElementById('joystick-knob');
        let moveF = 0, moveS = 0;
        document.getElementById('joystick-container').addEventListener('touchmove', e => {
            if(!gameActive) return;
            e.preventDefault(); e.stopPropagation();
            const r = e.currentTarget.getBoundingClientRect();
            const dx = e.touches[0].pageX - (r.left + r.width/2), dy = e.touches[0].pageY - (r.top + r.height/2);
            const d = Math.min(45, Math.sqrt(dx*dx+dy*dy)), ang = Math.atan
