<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>South Park Drone Delivery</title>
    <style>
        :root {
            --neon-blue: #00f2ff;
            --neon-green: #39ff14;
            --neon-red: #ff3131;
            --ui-bg: rgba(10, 10, 15, 0.98);
        }

        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            -webkit-touch-callout: none;
            -webkit-user-select: none;
            user-select: none;
        }

        html, body {
            width: 100%;
            height: 100%;
            overflow: hidden;
            background: #6fcf6a;
            font-family: 'Comic Sans MS', 'Chalkboard SE', 'Arial Black', sans-serif;
            position: fixed;
            touch-action: none;
        }

        canvas {
            display: block;
            width: 100%;
            height: 100%;
            image-rendering: crisp-edges;
        }

        #hud {
            position: fixed;
            top: 15px;
            left: 15px;
            right: 15px;
            display: flex;
            justify-content: space-between;
            pointer-events: none;
            z-index: 10;
        }

        .stat-box {
            background: var(--ui-bg);
            padding: 10px 18px;
            border-radius: 12px;
            border: 3px solid #000;
            box-shadow: 3px 3px 0 #000;
        }

        .stat-label {
            font-size: 11px;
            text-transform: uppercase;
            color: #fff;
            font-weight: 900;
            text-shadow: 2px 2px 0 #000;
        }

        .stat-value {
            font-size: 24px;
            font-weight: 900;
            color: var(--neon-green);
            text-shadow: 2px 2px 0 #000;
        }

        #menu-btn {
            position: fixed;
            top: 15px;
            right: 15px;
            width: 110px;
            height: 50px;
            background: #f1c40f;
            color: #000;
            border: 3px solid #000;
            border-radius: 10px;
            font-weight: 900;
            font-size: 14px;
            cursor: pointer;
            z-index: 50;
            box-shadow: 3px 3px 0 #000;
            pointer-events: auto;
        }

        #menu-btn:active {
            transform: translate(2px, 2px);
            box-shadow: 1px 1px 0 #000;
        }

        #meters {
            position: fixed;
            top: 90px;
            left: 15px;
            width: 160px;
            pointer-events: none;
            z-index: 10;
        }

        .m-bar {
            background: rgba(0,0,0,0.6);
            height: 12px;
            border-radius: 6px;
            overflow: hidden;
            border: 2px solid #000;
            margin: 4px 0 10px 0;
        }

        .m-fill {
            height: 100%;
            width: 100%;
            transition: width 0.2s;
        }

        #battery-fill {
            background: var(--neon-green);
        }

        #battery-fill.overcharged {
            background: linear-gradient(90deg, #ffff00, #ff00ff, #00ffff);
            animation: pulse 0.5s infinite;
        }

        @keyframes pulse {
            0%, 100% { filter: brightness(1); }
            50% { filter: brightness(1.5); }
        }

        #heat-fill {
            background: var(--neon-red);
            width: 0%;
        }

        #shop-overlay {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0,0,0,0.95);
            display: none;
            flex-direction: column;
            padding: 20px;
            padding-bottom: 100px;
            box-sizing: border-box;
            overflow-y: scroll;
            overflow-x: hidden;
            -webkit-overflow-scrolling: touch;
            overscroll-behavior: contain;
            z-index: 100;
            pointer-events: auto;
            touch-action: pan-y;
        }

        #shop-overlay.active {
            display: flex;
        }
        
        #shop-overlay * {
            touch-action: auto;
        }

        .close-btn {
            background: var(--neon-red);
            color: white;
            border: 3px solid #000;
            padding: 15px;
            border-radius: 10px;
            font-weight: 900;
            margin-bottom: 20px;
            cursor: pointer;
            box-shadow: 3px 3px 0 #000;
            pointer-events: auto;
            position: sticky;
            top: 0;
            z-index: 101;
            flex-shrink: 0;
        }

        .close-btn:active {
            transform: translate(2px, 2px);
            box-shadow: 1px 1px 0 #000;
        }

        .card {
            background: rgba(255,255,255,0.1);
            padding: 12px;
            margin-bottom: 15px;
            border-radius: 12px;
            border: 3px solid #000;
            color: white;
            box-shadow: 3px 3px 0 #000;
            flex-shrink: 0;
        }

        .card h2 {
            color: var(--neon-blue);
            margin-bottom: 12px;
            font-size: 18px;
            text-shadow: 2px 2px 0 #000;
        }

        .buy-btn {
            width: 100%;
            padding: 12px;
            margin-top: 10px;
            background: var(--neon-blue);
            color: #000;
            border: 3px solid #000;
            border-radius: 8px;
            font-weight: 900;
            cursor: pointer;
            box-shadow: 2px 2px 0 #000;
            pointer-events: auto;
        }

        .buy-btn:active {
            transform: translate(1px, 1px);
            box-shadow: 1px 1px 0 #000;
        }

        .buy-btn:disabled {
            background: #333;
            color: #666;
            cursor: not-allowed;
            pointer-events: auto;
        }

        #joy-base {
            position: fixed;
            bottom: 40px;
            right: 40px;
            width: 140px;
            height: 140px;
            background: rgba(0,0,0,0.3);
            border: 4px solid #000;
            border-radius: 50%;
            z-index: 10;
            touch-action: none;
            box-shadow: 3px 3px 0 #000;
        }

        #joy-stick {
            position: absolute;
            top: 35px;
            left: 35px;
            width: 70px;
            height: 70px;
            background: white;
            border-radius: 50%;
            border: 3px solid #000;
            box-shadow: 2px 2px 0 #000;
            pointer-events: none;
        }

        #notify {
            position: fixed;
            bottom: 25%;
            width: 100%;
            text-align: center;
            font-size: 32px;
            font-weight: 900;
            text-shadow: 3px 3px 0 #000;
            pointer-events: none;
            opacity: 0;
            transition: 0.3s;
            z-index: 20;
        }

        .stat-label-meter {
            text-shadow: 2px 2px 0 #000;
            font-size: 11px;
            text-transform: uppercase;
            color: #fff;
            font-weight: 900;
        }
    </style>
</head>
<body>
    <canvas id="gameCanvas"></canvas>

    <div id="hud">
        <div class="stat-box">
            <div class="stat-label">CASH</div>
            <div class="stat-value">$<span id="cash">0</span></div>
        </div>
    </div>

    <button id="menu-btn">GARAGE</button>

    <div id="meters">
        <div class="stat-label-meter">BATTERY</div>
        <div class="m-bar"><div id="battery-fill" class="m-fill"></div></div>
        <div class="stat-label-meter">HEAT</div>
        <div class="m-bar"><div id="heat-fill" class="m-fill"></div></div>
    </div>

    <div id="shop-overlay">
        <button class="close-btn" id="close-btn">CLOSE GARAGE</button>
        <div id="drone-shop"></div>
        <div id="upgrade-shop"></div>
    </div>

    <div id="joy-base">
        <div id="joy-stick"></div>
    </div>

    <div id="notify"></div>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;

        // ============================================================================
        // GAME STATE
        // ============================================================================
        let money = 0;
        let droneIdx = 0;
        const joy = { active: false, x: 0, y: 0 };
        const particles = [];
        const confetti = []; // For balloon pop effects
        const clouds = [];
        const trees = [];
        const houses = [];
        const people = [];
        const balloons = []; // TASK 5: Balloon power-ups
        const roads = []; // TASK 4: Grid-based roads
        let screenShake = 0; // TASK 5: Screen shake effect
        // Smooth tilt values (lerped toward actual velocity for natural feel)
        let tiltX = 0; // left/right bank angle
        let tiltY = 0; // forward/back pitch angle

        // === WIND SYSTEM ===
        const wind = { vx: 0, vy: 0, targetVx: 0, targetVy: 0, timer: 0, label: 'CALM' };

        // === HAZARD SYSTEM ===
        const grumpyNeighbors = [];
        const projectiles = [];

        // === FRAGILE CARGO ===
        // stored on p.fragile, payout penalty tracked in p.fragileHits

        // === NEON TRAIL ===
        const droneTrail = []; // { x, y, age } entries

        // TASK 1: Enhanced drone definitions with visual complexity tiers
        const DRONES = [
            { 
                name: '📦 Cardboard Classic', 
                color: '#d4a373', 
                speed: 4, 
                acc: 0.5, 
                cost: 0,
                desc: 'Built from a pizza box. Flies like one too.',
                tier: 0, // Basic frame
                bodySize: 30,
                armLength: 35,
                armWidth: 6,
                propSize: 15,
                hasGlow: false
            },
            { 
                name: '🛴 Scooter Ripper', 
                color: '#e74c3c', 
                speed: 6, 
                acc: 0.65, 
                cost: 150,
                desc: 'Stolen from some kid. Pretty fast!',
                tier: 1, // Reinforced frame
                bodySize: 32,
                armLength: 38,
                armWidth: 7,
                propSize: 17,
                hasGlow: false
            },
            { 
                name: '🚁 Speed Demon', 
                color: '#ff3131', 
                speed: 8, 
                acc: 0.8, 
                cost: 350,
                desc: 'Red means faster, everyone knows that.',
                tier: 2, // Sport frame
                bodySize: 34,
                armLength: 40,
                armWidth: 8,
                propSize: 18,
                hasGlow: false
            },
            { 
                name: '✈️ Turbo Express', 
                color: '#00d4ff', 
                speed: 10, 
                acc: 1.0, 
                cost: 750,
                desc: 'Aerodynamic AF. Handles like a dream.',
                tier: 3, // Pro frame with streamlining
                bodySize: 36,
                armLength: 42,
                armWidth: 9,
                propSize: 20,
                hasGlow: true
            },
            { 
                name: '🎮 Gaming Rig 9000', 
                color: '#9b59b6', 
                speed: 12, 
                acc: 1.2, 
                cost: 1500,
                desc: 'RGB lighting makes it go faster. Science.',
                tier: 4, // RGB frame
                bodySize: 38,
                armLength: 45,
                armWidth: 10,
                propSize: 22,
                hasGlow: true,
                rgb: true
            },
            { 
                name: '🚀 Mega Blaster', 
                color: '#ff00ff', 
                speed: 15, 
                acc: 1.4, 
                cost: 3000,
                desc: 'Might be illegal in 3 states. Worth it.',
                tier: 5, // Racing frame
                bodySize: 40,
                armLength: 48,
                armWidth: 11,
                propSize: 24,
                hasGlow: true,
                rgb: true
            },
            { 
                name: '👽 Alien Tech X', 
                color: '#39ff14', 
                speed: 20, 
                acc: 1.8, 
                cost: 6000,
                desc: 'Found it at Area 51. Don\'t ask questions.',
                tier: 6, // Alien tech
                bodySize: 42,
                armLength: 50,
                armWidth: 12,
                propSize: 26,
                hasGlow: true,
                rgb: true,
                alien: true
            }
        ];

        const upgrades = {
            battery:   { lvl: 0, max: 5, cost: 50  },
            cooling:   { lvl: 0, max: 5, cost: 75  },
            winch:     { lvl: 0, max: 3, cost: 100 },
            gyro:      { lvl: 0, max: 3, cost: 120 },
            nav:       { lvl: 0, max: 1, cost: 200 },
            aero:      { lvl: 0, max: 3, cost: 180 },   // Aero-Stabilizers: wind resistance
            ecm:       { lvl: 0, max: 3, cost: 220 },   // ECM: deflects neighbor projectiles
            shockHook: { lvl: 0, max: 3, cost: 150 }    // Shock-Absorbing Hook: reduces fragile penalty
        };

        const p = {
            x: 0, y: 0, vx: 0, vy: 0, bat: 100, batMax: 100,
            heat: 0, cable: 0, cargo: null, target: null, mode: 'idle',
            overcharged: false, overchargeAmount: 0,
            fragile: false, fragileHits: 0,   // fragile cargo state
            stunTimer: 0                        // stun from projectile hits
        };
        
        let initialized = false;
        const restaurants = [];
        const houses_dest = [];

        // ============================================================================
        // TASK 4: GRID-BASED NEIGHBORHOOD ROADS
        // ============================================================================
        function generateRoadGrid() {
            // Create horizontal roads
            for (let y = -1500; y <= 1500; y += 300) {
                roads.push({
                    type: 'horizontal',
                    x: -2000,
                    y: y,
                    width: 4000,
                    height: 40
                });
            }
            
            // Create vertical roads
            for (let x = -2000; x <= 2000; x += 300) {
                roads.push({
                    type: 'vertical',
                    x: x,
                    y: -1500,
                    width: 40,
                    height: 3000
                });
            }
        }

        function drawRoads() {
            roads.forEach(road => {
                // Road surface
                ctx.fillStyle = '#333';
                ctx.strokeStyle = '#000';
                ctx.lineWidth = 3;
                
                if (road.type === 'horizontal') {
                    ctx.fillRect(road.x, road.y - road.height/2, road.width, road.height);
                    ctx.strokeRect(road.x, road.y - road.height/2, road.width, road.height);
                    
                    // Center line
                    ctx.strokeStyle = '#ffff00';
                    ctx.lineWidth = 2;
                    ctx.setLineDash([20, 15]);
                    ctx.beginPath();
                    ctx.moveTo(road.x, road.y);
                    ctx.lineTo(road.x + road.width, road.y);
                    ctx.stroke();
                    ctx.setLineDash([]);
                } else {
                    ctx.fillRect(road.x - road.width/2, road.y, road.width, road.height);
                    ctx.strokeRect(road.x - road.width/2, road.y, road.width, road.height);
                    
                    // Center line
                    ctx.strokeStyle = '#ffff00';
                    ctx.lineWidth = 2;
                    ctx.setLineDash([20, 15]);
                    ctx.beginPath();
                    ctx.moveTo(road.x, road.y);
                    ctx.lineTo(road.x, road.y + road.height);
                    ctx.stroke();
                    ctx.setLineDash([]);
                }
            });
        }

        // ============================================================================
        // WORLD GENERATION
        // ============================================================================
        function init() {
            console.log('🎮 INIT STARTED');
            
            // TASK 4: Generate road grid first
            generateRoadGrid();
            console.log('✅ Roads generated:', roads.length);

            // Generate clouds
            for (let i = 0; i < 8; i++) {
                clouds.push({
                    x: Math.random() * 4000 - 2000,
                    y: Math.random() * 1500 - 1500,
                    size: 40 + Math.random() * 40,
                    speed: 0.1 + Math.random() * 0.3
                });
            }

            // Generate trees (avoid roads)
            for (let i = 0; i < 40; i++) {
                let x, y;
                do {
                    x = Math.random() * 4000 - 2000;
                    y = Math.random() * 3000 - 1500;
                } while (isOnRoad(x, y, 50));
                
                trees.push({ x, y, size: 30 + Math.random() * 30 });
            }

            // Generate houses aligned to street grid (avoid roads)
            for (let i = 0; i < 25; i++) {
                let x, y;
                do {
                    // Snap to grid positions between roads
                    x = Math.floor(Math.random() * 13) * 300 - 1850;
                    y = Math.floor(Math.random() * 10) * 300 - 1350;
                } while (isOnRoad(x, y, 50));
                
                houses.push({
                    x, y,
                    w: 80 + Math.random() * 60,
                    h: 70 + Math.random() * 40,
                    color: ['#f1c40f', '#e74c3c', '#3498db', '#9b59b6', '#1abc9c'][Math.floor(Math.random() * 5)],
                    roofColor: ['#c0392b', '#8e44ad', '#2c3e50', '#16a085'][Math.floor(Math.random() * 4)]
                });
            }

            // Generate restaurants (aligned to grid)
            const restaurantNames = ['PIZZA SHACK', 'BURGER BARN', 'TACO TOWN', 'NOODLE HUT', 'WING PALACE'];
            for (let i = 0; i < 5; i++) {
                let x, y;
                do {
                    x = Math.floor(Math.random() * 13) * 300 - 1850;
                    y = Math.floor(Math.random() * 10) * 300 - 1350;
                } while (isOnRoad(x, y, 50));
                
                restaurants.push({ x, y, name: restaurantNames[i] });
            }

            // Generate delivery destinations
            for (let i = 0; i < 8; i++) {
                let x, y;
                do {
                    x = Math.floor(Math.random() * 13) * 300 - 1850;
                    y = Math.floor(Math.random() * 10) * 300 - 1350;
                } while (isOnRoad(x, y, 50));
                
                houses_dest.push({ x, y });
            }

            // TASK 3: Generate diverse South Park people (KEEP EXISTING + ADD NEW)
            const peopleColors = [
                { body: '#e74c3c', skin: '#f4c2a3', name: 'Red' },
                { body: '#3498db', skin: '#f4c2a3', name: 'Blue' },
                { body: '#f1c40f', skin: '#f4c2a3', name: 'Yellow' },
                { body: '#9b59b6', skin: '#f4c2a3', name: 'Purple' },
                { body: '#e67e22', skin: '#f4c2a3', name: 'Orange' },
                { body: '#1abc9c', skin: '#f4c2a3', name: 'Teal' },
                { body: '#2ecc71', skin: '#f4c2a3', name: 'Green' },
                { body: '#34495e', skin: '#f4c2a3', name: 'Gray' },
                // TASK 3: ADD people with darker skin tones (respectfully)
                { body: '#c0392b', skin: '#8b5a3c', name: 'RedDark' },
                { body: '#2980b9', skin: '#6b4423', name: 'BlueDark' },
                { body: '#f39c12', skin: '#4a2c1a', name: 'YellowDark' },
                { body: '#8e44ad', skin: '#3d2817', name: 'PurpleDark' },
                { body: '#d35400', skin: '#5c3317', name: 'OrangeDark' },
                { body: '#16a085', skin: '#704214', name: 'TealDark' },
                { body: '#27ae60', skin: '#593f1a', name: 'GreenDark' },
                { body: '#2c3e50', skin: '#3e2723', name: 'GrayDark' }
            ];

            for (let i = 0; i < 50; i++) { // Increased from 30 to 50 for more life
                const colorScheme = peopleColors[Math.floor(Math.random() * peopleColors.length)];
                let x, y;
                do {
                    x = (Math.random() - 0.5) * 3500;
                    y = (Math.random() - 0.5) * 2800;
                } while (isOnRoad(x, y, 30));
                
                people.push({
                    x, y,
                    vx: (Math.random() - 0.5) * 0.5,
                    vy: (Math.random() - 0.5) * 0.5,
                    color: colorScheme.body,
                    skin: colorScheme.skin,
                    facingRight: Math.random() > 0.5,
                    walkCycle: Math.random() * 100
                });
            }

            // TASK 5: Generate balloons scattered around neighborhood
            for (let i = 0; i < 15; i++) {
                balloons.push({
                    x: (Math.random() - 0.5) * 3500,
                    y: (Math.random() - 0.5) * 2500,
                    color: ['#ff3131', '#3498db', '#f1c40f', '#9b59b6', '#39ff14'][Math.floor(Math.random() * 5)],
                    bobOffset: Math.random() * Math.PI * 2,
                    active: true
                });
            }

            // Spawn grumpy neighbors near houses
            const grumpyNames = ['😡 KAREN', '👴 GRANDPA', '🧹 HAROLD', '📞 BRENDA', '🔧 HANK'];
            houses.forEach((h, i) => {
                if (i % 3 === 0) { // ~1 in 3 houses has a grumpy neighbor
                    grumpyNeighbors.push({
                        x: h.x + (Math.random() - 0.5) * 60,
                        y: h.y + h.h / 2 + 20,
                        name: grumpyNames[Math.floor(Math.random() * grumpyNames.length)],
                        throwCooldown: 0,
                        angry: false,
                        walkCycle: Math.random() * 100,
                        facingRight: Math.random() > 0.5
                    });
                }
            });

            updateShop();

            // STEP 3: Explicitly spawn the drone
            p.x = 0;
            p.y = 0;
            p.vx = 0;
            p.vy = 0;
            p.bat = p.batMax;
            p.heat = 0;

            // newJob() MUST be called after p is reset, and mode must NOT be overwritten after this
            newJob();
            
            // Initialize joystick controls for mobile
            const jBase = document.getElementById('joy-base');
            const jStick = document.getElementById('joy-stick');

            jBase.addEventListener('touchstart', (e) => {
                e.preventDefault();
                joy.active = true;
            }, { passive: false });

            window.addEventListener('touchmove', (e) => {
                // Don't interfere with shop scrolling
                if (document.getElementById('shop-overlay').classList.contains('active')) {
                    return;
                }
                
                e.preventDefault();
                if (!joy.active) return;
                const t = e.touches[0];
                const r = jBase.getBoundingClientRect();
                let dx = t.clientX - (r.left + 70);
                let dy = t.clientY - (r.top + 70);
                const dist = Math.hypot(dx, dy);
                if (dist > 50) {
                    dx *= 50 / dist;
                    dy *= 50 / dist;
                }
                joy.x = dx;
                joy.y = dy;
                jStick.style.transform = `translate(${dx}px, ${dy}px)`;
            }, { passive: false });

            window.addEventListener('touchend', (e) => {
                // Don't interfere with shop button taps
                if (document.getElementById('shop-overlay').classList.contains('active')) {
                    joy.active = false;
                    joy.x = 0;
                    joy.y = 0;
                    jStick.style.transform = 'translate(0,0)';
                    return;
                }
                e.preventDefault();
                joy.active = false;
                joy.x = 0;
                joy.y = 0;
                jStick.style.transform = 'translate(0,0)';
            }, { passive: false });
            
            // Garage button event listener
            const garageBtn = document.getElementById('menu-btn');
            garageBtn.addEventListener('touchstart', (e) => {
                e.preventDefault();
                e.stopPropagation();
                toggleShop();
            }, { passive: false });
            
            garageBtn.addEventListener('click', (e) => {
                e.preventDefault();
                e.stopPropagation();
                toggleShop();
            });
            
            // Close button event listener
            const closeBtn = document.getElementById('close-btn');
            closeBtn.addEventListener('touchstart', (e) => {
                e.preventDefault();
                e.stopPropagation();
                toggleShop();
            }, { passive: false });
            
            closeBtn.addEventListener('click', (e) => {
                e.preventDefault();
                e.stopPropagation();
                toggleShop();
            });
            
            initialized = true;
            console.log('✅ INITIALIZED = TRUE');
            console.log('🎮 Player position:', p.x, p.y);
            console.log('🚁 Drone index:', droneIdx, DRONES[droneIdx].name);
            
            // STEP 7: Signal that game is alive
            setTimeout(() => {
                msg('🚁 DRONE ONLINE', '#00ff00');
            }, 500);
            
            console.log('🔄 Starting game loop...');
            gameLoop();
            console.log('✅ INIT COMPLETE');
        }

        // Helper function to check if position is on a road
        function isOnRoad(x, y, margin = 30) {
            return roads.some(road => {
                if (road.type === 'horizontal') {
                    return Math.abs(y - road.y) < (road.height/2 + margin);
                } else {
                    return Math.abs(x - road.x) < (road.width/2 + margin);
                }
            });
        }

        // ============================================================================
        // TASK 2: ENHANCED DRONE DRAWING WITH TILT, WIND, NEON SPEED OVERLAY
        // ============================================================================
        function drawDrone(x, y, droneData, heat, speed, tx, ty) {
            const { color, tier, bodySize, armLength, armWidth, propSize, hasGlow, rgb, alien } = droneData;
            const halfBody = bodySize / 2;
            const propDist = armLength * 0.7;
            const now = Date.now();

            // --- SPEED-REACTIVE NEON COLOR ---
            // idle=cyan, mid=yellow-orange, fast=red, overheated=white-red pulse
            const maxSpeed = droneData.speed;
            const speedRatio = Math.min(speed / maxSpeed, 1);
            let neonR, neonG, neonB;
            if (speedRatio < 0.5) {
                // cyan -> yellow
                const t = speedRatio / 0.5;
                neonR = Math.floor(0   + 255 * t);
                neonG = Math.floor(210 + 45  * t);
                neonB = Math.floor(255 - 255 * t);
            } else {
                // yellow -> red
                const t = (speedRatio - 0.5) / 0.5;
                neonR = 255;
                neonG = Math.floor(255 - 255 * t);
                neonB = 0;
            }
            if (heat > 70) {
                // Overheat pulse flicker
                const flicker = 0.5 + 0.5 * Math.sin(now * 0.03);
                neonR = 255;
                neonG = Math.floor(neonG * (1 - flicker * 0.8));
                neonB = Math.floor(150 * flicker);
            }
            const neonColor = `rgb(${neonR},${neonG},${neonB})`;
            const glowIntensity = 10 + speedRatio * 30 + (heat > 70 ? 15 * Math.sin(now * 0.02) : 0);

            // --- TILT TRANSFORM (perspective skew) ---
            // tx = left/right bank, ty = forward/back pitch
            // We simulate 3D tilt by scaling y-axis for pitch and x-axis for bank
            const bankScale  = Math.cos(tx * 0.8);   // x squish from side banking
            const pitchScale = Math.cos(ty * 0.7);   // y squish from forward pitch
            // Shadow on ground shows real lean direction
            ctx.save();
            ctx.translate(x, y + 18);
            ctx.scale(1 + Math.abs(tx) * 0.3, 0.25 * pitchScale);
            ctx.globalAlpha = 0.18;
            ctx.fillStyle = '#000';
            ctx.beginPath();
            ctx.ellipse(0, 0, halfBody + armLength * 0.9, halfBody + armLength * 0.4, 0, 0, Math.PI * 2);
            ctx.fill();
            ctx.globalAlpha = 1;
            ctx.restore();

            ctx.save();
            ctx.translate(x, y);
            // Apply tilt as a perspective-style transform
            ctx.transform(
                bankScale,       // scaleX
                ty * 0.15,       // skewX (pitch tips front/back)
                tx * 0.15,       // skewY (bank tips left/right)
                pitchScale,      // scaleY
                0, 0
            );

            // --- OUTER GLOW RING (speed indicator) ---
            if (hasGlow || rgb) {
                let glowColorOuter;
                if (rgb) {
                    const gt = now * 0.003;
                    glowColorOuter = `hsl(${(gt * 60) % 360},100%,60%)`;
                } else {
                    glowColorOuter = neonColor;
                }
                ctx.shadowBlur = glowIntensity;
                ctx.shadowColor = glowColorOuter;
                // Draw invisible circle just to cast shadow ring
                ctx.strokeStyle = glowColorOuter;
                ctx.lineWidth = 2;
                ctx.globalAlpha = 0.4 + speedRatio * 0.4;
                ctx.beginPath();
                ctx.arc(0, 0, halfBody + armLength * 0.85, 0, Math.PI * 2);
                ctx.stroke();
                ctx.globalAlpha = 1;
                ctx.shadowBlur = 0;
            }

            // Center body
            ctx.shadowBlur = glowIntensity * 0.5;
            ctx.shadowColor = neonColor;
            ctx.fillStyle = color;
            ctx.strokeStyle = '#000';
            ctx.lineWidth = 3;
            if (alien) {
                ctx.beginPath();
                for (let i = 0; i < 6; i++) {
                    const ang = (Math.PI / 3) * i - Math.PI / 6;
                    const px = Math.cos(ang) * halfBody;
                    const py = Math.sin(ang) * halfBody;
                    if (i === 0) ctx.moveTo(px, py); else ctx.lineTo(px, py);
                }
                ctx.closePath();
                ctx.fill();
                ctx.stroke();
            } else {
                ctx.fillRect(-halfBody, -halfBody, bodySize, bodySize);
                ctx.strokeRect(-halfBody, -halfBody, bodySize, bodySize);
            }
            ctx.shadowBlur = 0;

            // Arms
            ctx.fillStyle = color;
            ctx.strokeStyle = '#000';
            ctx.lineWidth = 3;
            ctx.save();
            ctx.rotate(Math.PI / 4);
            ctx.fillRect(-armLength, -armWidth/2, armLength * 2, armWidth);
            ctx.strokeRect(-armLength, -armWidth/2, armLength * 2, armWidth);
            ctx.restore();
            ctx.save();
            ctx.rotate(-Math.PI / 4);
            ctx.fillRect(-armLength, -armWidth/2, armLength * 2, armWidth);
            ctx.strokeRect(-armLength, -armWidth/2, armLength * 2, armWidth);
            ctx.restore();

            // Propellers + rotor discs
            const rotTime = (now * (0.012 + speedRatio * 0.025)) % (Math.PI * 2);
            const propPositions = [[-propDist, -propDist], [propDist, -propDist], [-propDist, propDist], [propDist, propDist]];

            propPositions.forEach((pos, idx) => {
                const dir = idx % 2 === 0 ? rotTime : -rotTime;

                // Rotor disc — translucent blur disc that gets more opaque with speed
                const discAlpha = 0.08 + speedRatio * 0.22;
                const discGrad = ctx.createRadialGradient(pos[0], pos[1], 0, pos[0], pos[1], propSize);
                discGrad.addColorStop(0,   `rgba(${neonR},${neonG},${neonB},${discAlpha * 2})`);
                discGrad.addColorStop(0.6, `rgba(${neonR},${neonG},${neonB},${discAlpha})`);
                discGrad.addColorStop(1,   `rgba(${neonR},${neonG},${neonB},0)`);
                ctx.fillStyle = discGrad;
                ctx.beginPath();
                ctx.arc(pos[0], pos[1], propSize + 4, 0, Math.PI * 2);
                ctx.fill();

                // Motor hub
                ctx.fillStyle = '#222';
                ctx.strokeStyle = '#000';
                ctx.lineWidth = 2;
                ctx.beginPath();
                ctx.arc(pos[0], pos[1], 5 + tier * 0.5, 0, Math.PI * 2);
                ctx.fill();
                ctx.stroke();

                // Blade arcs — neon colored, speed-reactive
                ctx.shadowBlur = 6 + speedRatio * 10;
                ctx.shadowColor = neonColor;
                ctx.strokeStyle = neonColor;
                ctx.lineWidth = 3 + tier * 0.3;
                ctx.globalAlpha = 0.6 + speedRatio * 0.35;
                ctx.beginPath();
                ctx.arc(pos[0], pos[1], propSize, dir, dir + 1.5);
                ctx.stroke();
                ctx.beginPath();
                ctx.arc(pos[0], pos[1], propSize, dir + Math.PI, dir + Math.PI + 1.5);
                ctx.stroke();
                if (tier >= 4) {
                    ctx.globalAlpha = 0.4 + speedRatio * 0.2;
                    ctx.beginPath();
                    ctx.arc(pos[0], pos[1], propSize * 0.7, dir + 0.75, dir + 0.75 + 1.5);
                    ctx.stroke();
                    ctx.beginPath();
                    ctx.arc(pos[0], pos[1], propSize * 0.7, dir + 0.75 + Math.PI, dir + 0.75 + Math.PI + 1.5);
                    ctx.stroke();
                }
                ctx.globalAlpha = 1;
                ctx.shadowBlur = 0;
            });

            // Camera
            ctx.fillStyle = tier >= 5 ? '#ff0000' : '#111';
            ctx.strokeStyle = '#000';
            ctx.lineWidth = 1;
            ctx.beginPath();
            ctx.arc(0, halfBody * 0.5, 3 + tier * 0.3, 0, Math.PI * 2);
            ctx.fill();
            ctx.stroke();

            // Alien pulse ring
            if (alien) {
                ctx.strokeStyle = '#39ff14';
                ctx.lineWidth = 2;
                ctx.shadowBlur = 10;
                ctx.shadowColor = '#39ff14';
                ctx.globalAlpha = 0.5 + 0.5 * Math.sin(now * 0.008);
                ctx.beginPath();
                ctx.arc(0, 0, halfBody + 8, 0, Math.PI * 2);
                ctx.stroke();
                ctx.globalAlpha = 1;
                ctx.shadowBlur = 0;
            }

            // Speed streak lines shooting back from body when fast
            if (speedRatio > 0.35) {
                const streakCount = Math.floor(speedRatio * 6);
                const streakLen = speedRatio * 28;
                // Direction opposite to movement (tx/ty give us lean direction)
                const streakAngle = Math.atan2(-ty, -tx) + Math.PI;
                ctx.globalAlpha = (speedRatio - 0.35) * 0.9;
                ctx.strokeStyle = neonColor;
                ctx.lineWidth = 1.5;
                ctx.shadowBlur = 4;
                ctx.shadowColor = neonColor;
                for (let s = 0; s < streakCount; s++) {
                    const spread = (s - streakCount / 2) * 8;
                    const sx = Math.cos(streakAngle + Math.PI/2) * spread;
                    const sy = Math.sin(streakAngle + Math.PI/2) * spread;
                    ctx.beginPath();
                    ctx.moveTo(sx, sy);
                    ctx.lineTo(sx + Math.cos(streakAngle) * streakLen * (0.6 + Math.random() * 0.4), 
                               sy + Math.sin(streakAngle) * streakLen * (0.6 + Math.random() * 0.4));
                    ctx.stroke();
                }
                ctx.globalAlpha = 1;
                ctx.shadowBlur = 0;
            }

            ctx.restore();
        }

        // Spawn wind particles from all 4 rotors (called from game loop)
        function spawnRotorWind(px, py, droneData, speed) {
            if (speed < 0.3) return;
            const { armLength, propSize } = droneData;
            const propDist = armLength * 0.7;
            const propPositions = [[-propDist, -propDist], [propDist, -propDist], [-propDist, propDist], [propDist, propDist]];
            // Only spawn occasionally; more at higher speed
            if (Math.random() > 0.25 + speed * 0.04) return;
            const pick = propPositions[Math.floor(Math.random() * 4)];
            const wx = px + pick[0];
            const wy = py + pick[1];
            // Ring of tiny wind puffs outward from rotor
            const count = 2 + Math.floor(speed * 0.3);
            for (let i = 0; i < count; i++) {
                const ang = Math.random() * Math.PI * 2;
                const spd = (propSize * 0.08) + Math.random() * speed * 0.12;
                particles.push({
                    x: wx + Math.cos(ang) * (propSize * 0.4),
                    y: wy + Math.sin(ang) * (propSize * 0.4),
                    vx: Math.cos(ang) * spd,
                    vy: Math.sin(ang) * spd + 0.4, // slight downwash
                    size: 3 + Math.random() * 3,
                    maxLife: 1, life: 0.7 + Math.random() * 0.3,
                    type: 'wind',
                    color: speed > 8 ? '#ffaa00' : '#aaf0ff'
                });
            }
        }

        function drawCloud(x, y, size) {
            ctx.fillStyle = 'rgba(255, 255, 255, 0.8)';
            ctx.strokeStyle = '#000';
            ctx.lineWidth = 2;
            
            ctx.beginPath();
            ctx.arc(x - size/3, y, size/2, 0, Math.PI * 2);
            ctx.arc(x + size/3, y, size/2, 0, Math.PI * 2);
            ctx.arc(x, y - size/4, size/2, 0, Math.PI * 2);
            ctx.fill();
            ctx.stroke();
        }

        function drawTree(x, y, size) {
            ctx.fillStyle = '#5d4e37';
            ctx.strokeStyle = '#000';
            ctx.lineWidth = 2;
            ctx.fillRect(x - 5, y, 10, size/2);
            ctx.strokeRect(x - 5, y, 10, size/2);

            ctx.fillStyle = '#228b22';
            ctx.beginPath();
            ctx.moveTo(x, y - size/2);
            ctx.lineTo(x - size/2, y + size/4);
            ctx.lineTo(x + size/2, y + size/4);
            ctx.closePath();
            ctx.fill();
            ctx.stroke();
        }

        function drawHouse(x, y, w, h, color, roofColor) {
            ctx.fillStyle = color;
            ctx.strokeStyle = '#000';
            ctx.lineWidth = 3;
            ctx.fillRect(x - w/2, y - h/2, w, h);
            ctx.strokeRect(x - w/2, y - h/2, w, h);

            ctx.fillStyle = roofColor;
            ctx.beginPath();
            ctx.moveTo(x, y - h/2 - 25);
            ctx.lineTo(x - w/2 - 10, y - h/2);
            ctx.lineTo(x + w/2 + 10, y - h/2);
            ctx.closePath();
            ctx.fill();
            ctx.stroke();

            ctx.fillStyle = '#8b4513';
            ctx.fillRect(x - 10, y + h/2 - 30, 20, 30);
            ctx.strokeRect(x - 10, y + h/2 - 30, 20, 30);

            ctx.fillStyle = '#87ceeb';
            ctx.fillRect(x - w/2 + 10, y - 10, 15, 15);
            ctx.strokeRect(x - w/2 + 10, y - 10, 15, 15);
            ctx.fillRect(x + w/2 - 25, y - 10, 15, 15);
            ctx.strokeRect(x + w/2 - 25, y - 10, 15, 15);
        }

        function drawRestaurant(x, y, name) {
            const w = 120;
            const h = 90;

            ctx.fillStyle = '#ff6b35';
            ctx.strokeStyle = '#000';
            ctx.lineWidth = 3;
            ctx.fillRect(x - w/2, y - h/2, w, h);
            ctx.strokeRect(x - w/2, y - h/2, w, h);

            ctx.fillStyle = '#d62828';
            ctx.fillRect(x - w/2 - 5, y - h/2 - 10, w + 10, 10);
            ctx.strokeRect(x - w/2 - 5, y - h/2 - 10, w + 10, 10);

            ctx.fillStyle = '#f1c40f';
            ctx.fillRect(x - 50, y - h/2 - 40, 100, 25);
            ctx.strokeRect(x - 50, y - h/2 - 40, 100, 25);

            ctx.fillStyle = '#000';
            ctx.font = 'bold 10px Comic Sans MS';
            ctx.textAlign = 'center';
            ctx.fillText(name, x, y - h/2 - 22);

            ctx.fillStyle = '#8b4513';
            ctx.fillRect(x - 15, y + h/2 - 40, 30, 40);
            ctx.strokeRect(x - 15, y + h/2 - 40, 30, 40);

            ctx.fillStyle = '#87ceeb';
            ctx.fillRect(x - w/2 + 10, y - 15, 20, 20);
            ctx.strokeRect(x - w/2 + 10, y - 15, 20, 20);
            ctx.fillRect(x + w/2 - 30, y - 15, 20, 20);
            ctx.strokeRect(x + w/2 - 30, y - 15, 20, 20);
        }

        function drawDestinationHouse(x, y) {
            const w = 100;
            const h = 80;

            ctx.fillStyle = '#9b59b6';
            ctx.strokeStyle = '#000';
            ctx.lineWidth = 3;
            ctx.fillRect(x - w/2, y - h/2, w, h);
            ctx.strokeRect(x - w/2, y - h/2, w, h);

            ctx.fillStyle = '#2c3e50';
            ctx.beginPath();
            ctx.moveTo(x, y - h/2 - 25);
            ctx.lineTo(x - w/2 - 10, y - h/2);
            ctx.lineTo(x + w/2 + 10, y - h/2);
            ctx.closePath();
            ctx.fill();
            ctx.stroke();

            ctx.fillStyle = '#e74c3c';
            ctx.fillRect(x - w/2 - 25, y + h/2 - 15, 12, 15);
            ctx.strokeRect(x - w/2 - 25, y + h/2 - 15, 12, 15);
            ctx.fillRect(x - w/2 - 25, y + h/2 - 25, 12, 10);
            ctx.strokeRect(x - w/2 - 25, y + h/2 - 25, 12, 10);

            ctx.fillStyle = '#34495e';
            ctx.fillRect(x - 12, y + h/2 - 35, 24, 35);
            ctx.strokeRect(x - 12, y + h/2 - 35, 24, 35);
        }

        // TASK 3: Updated drawPerson to use diverse skin tones
        function drawPerson(x, y, color, skin, facingRight, walkCycle) {
            ctx.save();
            ctx.translate(x, y);
            
            if (!facingRight) {
                ctx.scale(-1, 1);
            }

            // Head (circle) - uses skin tone
            ctx.fillStyle = skin;
            ctx.strokeStyle = '#000';
            ctx.lineWidth = 2;
            ctx.beginPath();
            ctx.arc(0, -25, 12, 0, Math.PI * 2);
            ctx.fill();
            ctx.stroke();

            // Eyes (simple dots)
            ctx.fillStyle = '#000';
            ctx.beginPath();
            ctx.arc(-4, -26, 1.5, 0, Math.PI * 2);
            ctx.arc(4, -26, 1.5, 0, Math.PI * 2);
            ctx.fill();

            // Mouth (simple line)
            ctx.beginPath();
            ctx.moveTo(-3, -22);
            ctx.lineTo(3, -22);
            ctx.stroke();

            // Body (rectangle)
            ctx.fillStyle = color;
            ctx.fillRect(-8, -13, 16, 18);
            ctx.strokeRect(-8, -13, 16, 18);

            // Arms (simple lines)
            const armSwing = Math.sin(walkCycle * 0.15) * 5;
            ctx.strokeStyle = color;
            ctx.lineWidth = 3;
            ctx.beginPath();
            ctx.moveTo(-8, -8);
            ctx.lineTo(-12, -3 + armSwing);
            ctx.stroke();
            
            ctx.beginPath();
            ctx.moveTo(8, -8);
            ctx.lineTo(12, -3 - armSwing);
            ctx.stroke();

            // Legs (simple rectangles with walk cycle)
            const legSwing = Math.sin(walkCycle * 0.15) * 3;
            ctx.fillStyle = '#2c3e50';
            ctx.strokeStyle = '#000';
            ctx.lineWidth = 2;
            
            ctx.fillRect(-6, 5, 5, 10 + legSwing);
            ctx.strokeRect(-6, 5, 5, 10 + legSwing);
            
            ctx.fillRect(1, 5, 5, 10 - legSwing);
            ctx.strokeRect(1, 5, 5, 10 - legSwing);

            // Shoes
            ctx.fillStyle = '#000';
            ctx.fillRect(-8, 14 + legSwing, 7, 3);
            ctx.fillRect(1, 14 - legSwing, 7, 3);

            ctx.restore();
        }

        // TASK 5: Draw balloon function
        function drawBalloon(x, y, color, bobOffset) {
            const bob = Math.sin(Date.now() * 0.002 + bobOffset) * 5;
            const balloonY = y + bob;

            // String
            ctx.strokeStyle = '#333';
            ctx.lineWidth = 1;
            ctx.beginPath();
            ctx.moveTo(x, balloonY + 15);
            ctx.lineTo(x, balloonY + 40);
            ctx.stroke();

            // Balloon
            ctx.fillStyle = color;
            ctx.strokeStyle = '#000';
            ctx.lineWidth = 2;
            ctx.beginPath();
            ctx.arc(x, balloonY, 15, 0, Math.PI * 2);
            ctx.fill();
            ctx.stroke();

            // Highlight
            ctx.fillStyle = 'rgba(255,255,255,0.6)';
            ctx.beginPath();
            ctx.arc(x - 5, balloonY - 5, 4, 0, Math.PI * 2);
            ctx.fill();

            // Knot
            ctx.fillStyle = color;
            ctx.fillRect(x - 2, balloonY + 14, 4, 3);
            ctx.strokeRect(x - 2, balloonY + 14, 4, 3);
        }

        // TASK 5: Confetti particle function
        function spawnConfetti(x, y) {
            for (let i = 0; i < 30; i++) {
                confetti.push({
                    x, y,
                    vx: (Math.random() - 0.5) * 8,
                    vy: (Math.random() - 0.5) * 8 - 2,
                    color: ['#ff3131', '#3498db', '#f1c40f', '#9b59b6', '#39ff14'][Math.floor(Math.random() * 5)],
                    size: 3 + Math.random() * 3,
                    life: 1,
                    maxLife: 1,
                    rotation: Math.random() * Math.PI * 2,
                    rotSpeed: (Math.random() - 0.5) * 0.3
                });
            }
        }

        function spawnWildParticle(x, y, type) {
            particles.push({
                x, y,
                vx: (Math.random() - 0.5) * 2,
                vy: (Math.random() - 0.5) * 2,
                size: type === 'smoke' ? 3 : type === 'spark' ? 2 : 4,
                maxLife: 1,
                life: 1,
                type,
                color: type === 'smoke' ? '#888' : type === 'spark' ? '#ffff00' : '#d4a373'
            });
        }

        // ============================================================================
        // GAME LOOP
        // ============================================================================
        let frameCount = 0;
        function gameLoop() {
            requestAnimationFrame(gameLoop);
            
            if (!initialized) return;
            
            frameCount++;
            if (frameCount === 1) {
                console.log('🎬 FIRST FRAME RENDERED');
                console.log('Player:', p.x, p.y, 'Velocity:', p.vx, p.vy);
            }

            // Sky gradient
            const grad = ctx.createLinearGradient(0, 0, 0, canvas.height);
            grad.addColorStop(0, '#87ceeb');
            grad.addColorStop(1, '#6fcf6a');
            ctx.fillStyle = grad;
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            // TASK 5: Screen shake effect
            let shakeX = 0, shakeY = 0;
            if (screenShake > 0) {
                shakeX = (Math.random() - 0.5) * screenShake;
                shakeY = (Math.random() - 0.5) * screenShake;
                screenShake *= 0.9;
                if (screenShake < 0.5) screenShake = 0;
            }

            // Camera offset
            const camX = canvas.width / 2 - p.x + shakeX;
            const camY = canvas.height / 2 - p.y + shakeY;

            ctx.save();
            ctx.translate(camX, camY);

            // Draw clouds (parallax background)
            clouds.forEach(c => {
                c.x += c.speed;
                if (c.x > 2000) c.x = -2000;
                drawCloud(c.x + p.x * 0.1, c.y + p.y * 0.1, c.size);
            });

            // TASK 4: Draw road grid
            drawRoads();

            // Draw trees
            trees.forEach(t => drawTree(t.x, t.y, t.size));

            // Draw houses
            houses.forEach(h => drawHouse(h.x, h.y, h.w, h.h, h.color, h.roofColor));

            // Draw restaurants
            restaurants.forEach(r => drawRestaurant(r.x, r.y, r.name));

            // Draw destination houses
            houses_dest.forEach(h => drawDestinationHouse(h.x, h.y));

            // TASK 5: Draw and update balloons
            balloons.forEach((balloon, idx) => {
                if (!balloon.active) return;

                drawBalloon(balloon.x, balloon.y, balloon.color, balloon.bobOffset);

                // Check collision with drone
                const dist = Math.hypot(balloon.x - p.x, balloon.y - p.y);
                if (dist < 30) {
                    // POP! DRAMATIC MOMENT
                    balloon.active = false;
                    spawnConfetti(balloon.x, balloon.y);
                    screenShake = 15; // Big shake!
                    
                    // OVERCHARGE battery
                    p.overcharged = true;
                    p.overchargeAmount = 25;
                    p.bat = Math.min(p.batMax + 25, p.batMax + 25); // Set to 125%
                    
                    msg('BALLOON POP! ⚡OVERCHARGED⚡', '#ffff00');
                    
                    // Respawn balloon elsewhere after 10 seconds
                    setTimeout(() => {
                        balloon.x = (Math.random() - 0.5) * 3500;
                        balloon.y = (Math.random() - 0.5) * 2500;
                        balloon.active = true;
                    }, 10000);
                }
            });

            // Update and draw people
            people.forEach(person => {
                person.x += person.vx;
                person.y += person.vy;
                person.walkCycle += 1;

                if (person.x > 2000) person.x = -2000;
                if (person.x < -2000) person.x = 2000;
                if (person.y > 1500) person.y = -1500;
                if (person.y < -1500) person.y = 1500;

                if (person.vx > 0) person.facingRight = true;
                if (person.vx < 0) person.facingRight = false;

                if (Math.random() < 0.01) {
                    person.vx = (Math.random() - 0.5) * 0.5;
                    person.vy = (Math.random() - 0.5) * 0.5;
                }

                drawPerson(person.x, person.y, person.color, person.skin, person.facingRight, person.walkCycle);
            });

            // ============================================================================
            // PLAYER PHYSICS WITH UPGRADES
            // ============================================================================
            const drone = DRONES[droneIdx];
            const maxSpeed = drone.speed;

            // === WIND SYSTEM ===
            wind.timer++;
            if (wind.timer > 1800) { // shift every 30 sec at 60fps
                wind.timer = 0;
                const strength = Math.random() * 1.8;
                const angle = Math.random() * Math.PI * 2;
                wind.targetVx = Math.cos(angle) * strength;
                wind.targetVy = Math.sin(angle) * strength;
                const dirs = ['N','NE','E','SE','S','SW','W','NW'];
                wind.label = strength < 0.3 ? 'CALM' : dirs[Math.round(angle / (Math.PI/4)) % 8] + ' WIND';
            }
            // Lerp toward target wind
            wind.vx += (wind.targetVx - wind.vx) * 0.002;
            wind.vy += (wind.targetVy - wind.vy) * 0.002;
            // Aero upgrade reduces wind effect (30% per level)
            const aeroFactor = Math.max(0, 1 - upgrades.aero.lvl * 0.3);

            // GYRO UPGRADE: Better acceleration and tighter handling
            const accel = drone.acc * (1 + upgrades.gyro.lvl * 0.15);
            const friction = 0.95 - (upgrades.gyro.lvl * 0.01);

            // Joystick input (blocked by stun)
            if (p.stunTimer > 0) {
                p.stunTimer--;
                // Spin out while stunned
                p.vx *= 0.88;
                p.vy *= 0.88;
            } else {
                if (joy.active) {
                    p.vx += (joy.x / 50) * accel;
                    p.vy += (joy.y / 50) * accel;
                }
                // Apply wind drift
                p.vx += wind.vx * aeroFactor * 0.05;
                p.vy += wind.vy * aeroFactor * 0.05;
            }

            // Friction & speed limiting
            p.vx *= friction;
            p.vy *= friction;
            const speed = Math.hypot(p.vx, p.vy);
            if (speed > maxSpeed) {
                p.vx *= maxSpeed / speed;
                p.vy *= maxSpeed / speed;
            }

            // Fragile cargo: sudden jerk penalty
            if (p.fragile && p.cargo && speed > 7) {
                const penalty = Math.max(0, 1 - upgrades.shockHook.lvl * 0.33);
                if (Math.random() < 0.02 * penalty) {
                    p.fragileHits++;
                    screenShake = 3;
                    msg('💥 FRAGILE CARGO JOSTLED! -$5', '#ff9800');
                }
            }

            p.x += p.vx;
            p.y += p.vy;

            // === UPDATE PROJECTILES ===
            for (let i = projectiles.length - 1; i >= 0; i--) {
                const proj = projectiles[i];
                proj.x += proj.vx;
                proj.y += proj.vy;
                proj.vy += 0.15; // gravity
                proj.vx *= 0.99;
                proj.life--;
                if (proj.life <= 0) { projectiles.splice(i, 1); continue; }

                // Hit detection
                const hitDist = Math.hypot(proj.x - p.x, proj.y - p.y);
                // ECM upgrade has a chance to deflect
                const deflectChance = upgrades.ecm.lvl * 0.33;
                if (hitDist < 30) {
                    if (Math.random() < deflectChance) {
                        // ECM deflect — flash and destroy projectile
                        screenShake = 2;
                        spawnConfetti(proj.x, proj.y);
                        projectiles.splice(i, 1);
                        msg('⚡ ECM DEFLECTED!', '#00f2ff');
                    } else {
                        // Hit! stun + heat spike
                        p.stunTimer = 90;
                        p.heat = Math.min(100, p.heat + 20);
                        screenShake = 12;
                        projectiles.splice(i, 1);
                        msg('💥 HIT BY PROJECTILE! STUNNED!', 'var(--neon-red)');
                    }
                    continue;
                }

                // Draw projectile (shoe emoji-style brown blob)
                ctx.save();
                ctx.translate(proj.x, proj.y);
                ctx.rotate(Math.atan2(proj.vy, proj.vx));
                ctx.fillStyle = '#8b5e3c';
                ctx.strokeStyle = '#000';
                ctx.lineWidth = 2;
                ctx.beginPath();
                ctx.ellipse(0, 0, 10, 6, 0, 0, Math.PI * 2);
                ctx.fill();
                ctx.stroke();
                // Lace
                ctx.strokeStyle = '#fff';
                ctx.lineWidth = 1;
                ctx.beginPath();
                ctx.moveTo(-4, -2); ctx.lineTo(4, -2);
                ctx.moveTo(-4,  2); ctx.lineTo(4,  2);
                ctx.stroke();
                ctx.restore();
            }

            // === UPDATE GRUMPY NEIGHBORS ===
            grumpyNeighbors.forEach(n => {
                if (n.throwCooldown > 0) n.throwCooldown--;
                const distToDrone = Math.hypot(n.x - p.x, n.y - p.y);
                n.angry = distToDrone < 200;
                n.facingRight = p.x > n.x;

                // Throw if drone is close + slow + cooldown ready
                if (n.angry && distToDrone < 160 && speed < 4 && n.throwCooldown === 0) {
                    n.throwCooldown = 300 + Math.floor(Math.random() * 180); // 5-8s cooldown
                    const angle = Math.atan2(p.y - n.y, p.x - n.x);
                    const throwSpeed = 4 + Math.random() * 2;
                    projectiles.push({
                        x: n.x, y: n.y - 20,
                        vx: Math.cos(angle) * throwSpeed,
                        vy: Math.sin(angle) * throwSpeed - 2,
                        life: 120
                    });
                    msg(`${n.name} THREW A SHOE! MOVE!`, '#ff9800');
                }

                // Draw neighbor
                n.walkCycle += n.angry ? 0.5 : 0.2;
                drawPerson(n.x, n.y, n.angry ? '#ff3131' : '#e67e22', '#f4c2a3', n.facingRight, n.walkCycle);

                // Angry exclamation
                if (n.angry) {
                    ctx.fillStyle = '#ff3131';
                    ctx.font = 'bold 14px Comic Sans MS';
                    ctx.textAlign = 'center';
                    ctx.fillText('!!', n.x, n.y - 55);
                }
            });

            // COOLING UPGRADE: Reduces heat accumulation and increases cooling rate
            const heatRate = 0.3 - (upgrades.cooling.lvl * 0.04);
            const coolRate = 0.5 + (upgrades.cooling.lvl * 0.25);
            p.heat = Math.min(100, p.heat + speed * heatRate);
            p.heat = Math.max(0, p.heat - coolRate);
            
            // BATTERY UPGRADE: Already handled in battery max, but also reduce drain
            const batteryDrain = 0.02 + speed * (0.005 - upgrades.battery.lvl * 0.0008);
            
            // TASK 5: Handle overcharge decay
            if (p.overcharged) {
                if (p.bat > p.batMax) {
                    p.bat -= 0.05; // Slowly decay overcharge
                    if (p.bat <= p.batMax) {
                        p.overcharged = false;
                        p.overchargeAmount = 0;
                    }
                } else {
                    p.overcharged = false;
                    p.overchargeAmount = 0;
                }
            } else {
                p.bat = Math.max(0, p.bat - batteryDrain);
            }

            // Dead battery effect
            if (p.bat === 0) {
                p.vx *= 0.9;
                p.vy *= 0.9;
                msg('BATTERY DEAD! Find a balloon!', 'var(--neon-red)');
            }

            // Update UI - TASK 5: Show overcharge visually
            const batteryFill = document.getElementById('battery-fill');
            const batteryPercent = p.overcharged ? 
                Math.min(125, (p.bat / p.batMax) * 100) : 
                (p.bat / p.batMax * 100);
            batteryFill.style.width = batteryPercent + '%';
            
            if (p.overcharged) {
                batteryFill.classList.add('overcharged');
            } else {
                batteryFill.classList.remove('overcharged');
            }
            
            document.getElementById('heat-fill').style.width = p.heat + '%';

            // ============================================================================
            // TARGET INTERACTION
            // ============================================================================
            if (p.target) {
                const dist = Math.hypot(p.target.x - p.x, p.target.y - p.y);

                // Draw target marker
                ctx.strokeStyle = p.target.color;
                ctx.lineWidth = 4;
                ctx.globalAlpha = 0.4;
                ctx.setLineDash([10, 10]);
                ctx.beginPath();
                ctx.moveTo(p.x, p.y);
                ctx.lineTo(p.target.x, p.target.y);
                ctx.stroke();
                ctx.setLineDash([]);
                ctx.globalAlpha = 1.0;

                // Draw Landing Zone Indicator
                ctx.beginPath();
                ctx.arc(p.target.x, p.target.y, 140, 0, Math.PI * 2);
                ctx.strokeStyle = p.target.color;
                ctx.lineWidth = 2;
                ctx.setLineDash([5, 5]);
                ctx.stroke();
                ctx.setLineDash([]);
                
                // Pulsing fill if close enough but too fast
                if (dist < 140 && speed >= 3.0) {
                    ctx.globalAlpha = 0.2 + Math.sin(Date.now() * 0.01) * 0.1;
                    ctx.fillStyle = "#ffffff";
                    ctx.fill();
                    ctx.globalAlpha = 1.0;
                    
                    // Show "SLOW DOWN" hint
                    ctx.fillStyle = "#fff";
                    ctx.font = "bold 16px Arial";
                    ctx.textAlign = "center";
                    ctx.fillText("SLOW DOWN TO LAND", p.target.x, p.target.y - 150);
                }

                // Draw arrow at target
                const angle = Math.atan2(p.target.y - p.y, p.target.x - p.x);
                const arrowX = p.target.x - Math.cos(angle) * 40;
                const arrowY = p.target.y - Math.sin(angle) * 40;

                ctx.fillStyle = p.target.color;
                ctx.strokeStyle = '#000';
                ctx.lineWidth = 3;
                ctx.beginPath();
                ctx.moveTo(p.target.x, p.target.y);
                ctx.lineTo(arrowX - Math.sin(angle) * 20, arrowY + Math.cos(angle) * 20);
                ctx.lineTo(arrowX + Math.sin(angle) * 20, arrowY - Math.cos(angle) * 20);
                ctx.closePath();
                ctx.fill();
                ctx.stroke();

                // Pickup/delivery logic
                // Speed threshold scales with winch upgrade level (base 3.0, +0.5 per level)
                const pickupSpeedThreshold = 3.0 + upgrades.winch.lvl * 0.5;
                if (dist < 140 && speed < pickupSpeedThreshold) {
                    if (p.mode === 'pickup') {
                        p.cargo = '📦';
                        p.mode = 'deliver';
                        const dest = houses_dest[Math.floor(Math.random() * houses_dest.length)];
                        p.target = { x: dest.x, y: dest.y, color: '#39ff14' };
                        screenShake = 5;
                        msg('📦 CARGO SECURED! DELIVER IT!', 'var(--neon-green)');
                    } else if (p.mode === 'deliver') {
                        let payout = 50 + Math.floor(Math.random() * 50);
                        const fragilePenalty = p.fragile ? p.fragileHits * 5 : 0;
                        payout = Math.max(10, payout - fragilePenalty);
                        money += payout;
                        p.cargo = null;
                        p.target = null;
                        p.mode = 'idle';
                        p.fragile = false;
                        p.fragileHits = 0;
                        screenShake = 8;
                        spawnConfetti(p.x, p.y);
                        const bonusNote = fragilePenalty > 0 ? ` (-$${fragilePenalty} fragile)` : '';
                        msg(`+$${payout}! DELIVERY COMPLETE${bonusNote}`, 'var(--neon-green)');
                        updateShop();
                        setTimeout(() => newJob(), 2000);
                    }
                }
            }

            // Draw drone with cargo - TASK 1: WINCH UPGRADE affects cable stability
            // Also animate cable lowering when approaching a pickup target
            const approachingTarget = p.target && Math.hypot(p.target.x - p.x, p.target.y - p.y) < 200;
            const cableExtend = approachingTarget && p.mode === 'pickup' 
                ? Math.max(0, Math.min(1, (200 - Math.hypot(p.target.x - p.x, p.target.y - p.y)) / 200))
                : 0;

            if (p.cargo || cableExtend > 0) {
                const cableSwing = p.cargo 
                    ? Math.sin(Date.now() * 0.005) * (5 - upgrades.winch.lvl * 1.5)
                    : 0;
                const cableLength = p.cargo ? 40 : cableExtend * 40;

                // Draw cable
                ctx.strokeStyle = '#000';
                ctx.lineWidth = 2;
                ctx.beginPath();
                ctx.moveTo(p.x, p.y);
                ctx.lineTo(p.x + cableSwing, p.y + cableLength);
                ctx.stroke();

                // Draw hook at bottom of cable when lowering (no cargo yet)
                if (!p.cargo && cableExtend > 0) {
                    ctx.strokeStyle = '#aaa';
                    ctx.lineWidth = 3;
                    ctx.beginPath();
                    ctx.arc(p.x + cableSwing, p.y + cableLength, 5, 0, Math.PI * 1.5);
                    ctx.stroke();
                }

                if (p.cargo) {
                    ctx.fillStyle = '#8b4513';
                    ctx.strokeStyle = '#000';
                    ctx.lineWidth = 3;
                    ctx.fillRect(p.x - 15 + cableSwing, p.y + 40, 30, 25);
                    ctx.strokeRect(p.x - 15 + cableSwing, p.y + 40, 30, 25);
                    
                    ctx.strokeStyle = '#f1c40f';
                    ctx.lineWidth = 4;
                    ctx.beginPath();
                    ctx.moveTo(p.x - 15 + cableSwing, p.y + 52);
                    ctx.lineTo(p.x + 15 + cableSwing, p.y + 52);
                    ctx.stroke();
                }
            }

            // === NEON TRAIL ===
            const trailMaxLen = 12 + DRONES[droneIdx].tier * 8; // tier 0=12, tier 6=60
            droneTrail.push({ x: p.x, y: p.y });
            if (droneTrail.length > trailMaxLen) droneTrail.shift();
            if (droneTrail.length > 2 && speed > 0.5) {
                const trailColor = DRONES[droneIdx].color;
                for (let i = 1; i < droneTrail.length; i++) {
                    const t = i / droneTrail.length;
                    ctx.globalAlpha = t * 0.6 * (speed / DRONES[droneIdx].speed);
                    ctx.strokeStyle = trailColor;
                    ctx.lineWidth = t * 5;
                    ctx.shadowBlur = t * 10;
                    ctx.shadowColor = trailColor;
                    ctx.lineCap = 'round';
                    ctx.beginPath();
                    ctx.moveTo(droneTrail[i-1].x, droneTrail[i-1].y);
                    ctx.lineTo(droneTrail[i].x, droneTrail[i].y);
                    ctx.stroke();
                }
                ctx.globalAlpha = 1;
                ctx.shadowBlur = 0;
            }

            // Draw drone - tilt lerp: smoothly track velocity for natural banking
            const tiltLerp = 0.08;
            tiltX += (p.vx * 0.045 - tiltX) * tiltLerp;
            tiltY += (p.vy * 0.045 - tiltY) * tiltLerp;
            drawDrone(p.x, p.y, DRONES[droneIdx], p.heat, speed, tiltX, tiltY);

            // Rotor wind particles
            spawnRotorWind(p.x, p.y, DRONES[droneIdx], speed);

            // Smoke/spark trail particles
            if (speed > 1) {
                spawnWildParticle(p.x, p.y, 'smoke');
                if (speed > 10) spawnWildParticle(p.x, p.y, 'spark');
            }

            // Update regular particles
            for (let i = particles.length - 1; i >= 0; i--) {
                let part = particles[i];
                part.x += part.vx;
                part.y += part.vy;
                part.life -= 0.015;

                if (part.type === 'smoke') {
                    part.size += 0.2;
                } else if (part.type === 'wind') {
                    part.size += 0.35;
                    part.vx *= 0.93;
                    part.vy *= 0.93;
                } else {
                    part.size *= 0.95;
                }

                if (part.life <= 0) {
                    particles.splice(i, 1);
                    continue;
                }

                ctx.globalAlpha = part.life / part.maxLife;
                ctx.fillStyle = part.color;

                if (part.type === 'spark') {
                    ctx.shadowBlur = 10;
                    ctx.shadowColor = '#fff000';
                    ctx.fillRect(part.x, part.y, part.size * 2, part.size * 2);
                    ctx.shadowBlur = 0;
                } else if (part.type === 'wind') {
                    ctx.strokeStyle = part.color;
                    ctx.lineWidth = 1.2;
                    ctx.shadowBlur = 4;
                    ctx.shadowColor = part.color;
                    ctx.beginPath();
                    ctx.arc(part.x, part.y, part.size, 0, Math.PI * 2);
                    ctx.stroke();
                    ctx.shadowBlur = 0;
                } else {
                    ctx.beginPath();
                    ctx.arc(part.x, part.y, part.size, 0, Math.PI * 2);
                    ctx.fill();
                }
            }

            // TASK 5: Update confetti
            for (let i = confetti.length - 1; i >= 0; i--) {
                let conf = confetti[i];
                conf.x += conf.vx;
                conf.y += conf.vy;
                conf.vy += 0.2; // Gravity
                conf.vx *= 0.98;
                conf.rotation += conf.rotSpeed;
                conf.life -= 0.01;

                if (conf.life <= 0) {
                    confetti.splice(i, 1);
                    continue;
                }

                ctx.globalAlpha = conf.life;
                ctx.save();
                ctx.translate(conf.x, conf.y);
                ctx.rotate(conf.rotation);
                ctx.fillStyle = conf.color;
                ctx.fillRect(-conf.size/2, -conf.size/2, conf.size, conf.size);
                ctx.restore();
            }

            ctx.globalAlpha = 1.0;
            ctx.restore(); // end world transform

            // === WIND HUD (screen-space, bottom left) ===
            const windSpeed = Math.hypot(wind.vx, wind.vy);
            if (windSpeed > 0.1) {
                const wx = 90, wy = canvas.height - 60;
                ctx.save();
                ctx.translate(wx, wy);
                // Wind arrow
                const wAng = Math.atan2(wind.vy, wind.vx);
                ctx.strokeStyle = windSpeed > 1.2 ? '#ff9800' : '#00f2ff';
                ctx.lineWidth = 3;
                ctx.shadowBlur = 8;
                ctx.shadowColor = ctx.strokeStyle;
                ctx.globalAlpha = 0.85;
                ctx.beginPath();
                ctx.moveTo(-20, 0);
                ctx.lineTo(20, 0);
                ctx.stroke();
                ctx.rotate(wAng);
                ctx.beginPath();
                ctx.moveTo(14, 0); ctx.lineTo(6, -7); ctx.lineTo(6, 7); ctx.closePath();
                ctx.fillStyle = ctx.strokeStyle;
                ctx.fill();
                ctx.shadowBlur = 0;
                ctx.restore();
                ctx.fillStyle = windSpeed > 1.2 ? '#ff9800' : '#aef';
                ctx.font = 'bold 11px Comic Sans MS';
                ctx.textAlign = 'center';
                ctx.globalAlpha = 0.85;
                ctx.fillText(wind.label, wx, wy + 18);
                ctx.globalAlpha = 1;
            }

            // === STUN FLASH OVERLAY ===
            if (p.stunTimer > 0) {
                ctx.globalAlpha = (p.stunTimer / 90) * 0.35 * (Math.sin(Date.now() * 0.05) * 0.5 + 0.5);
                ctx.fillStyle = '#ff3131';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                ctx.globalAlpha = 1;
                ctx.fillStyle = '#fff';
                ctx.font = 'bold 28px Comic Sans MS';
                ctx.textAlign = 'center';
                ctx.fillText('💫 STUNNED!', canvas.width / 2, canvas.height / 2 - 40);
            }

            // === FRAGILE CARGO LABEL ===
            if (p.fragile && p.cargo) {
                const penalty = p.fragileHits * 5;
                ctx.fillStyle = penalty > 20 ? '#ff3131' : '#ff9800';
                ctx.font = 'bold 14px Comic Sans MS';
                ctx.textAlign = 'center';
                ctx.globalAlpha = 0.9;
                ctx.fillText(`🥚 FRAGILE  -$${penalty} SO FAR`, canvas.width / 2, canvas.height - 180);
                ctx.globalAlpha = 1;
            }

            // NAV UPGRADE - Navigation compass
            if (upgrades.nav.lvl > 0 && p.target) {
                const ang = Math.atan2(p.target.y - p.y, p.target.x - p.x);
                ctx.save();
                ctx.translate(canvas.width / 2, canvas.height / 2);
                ctx.rotate(ang);
                ctx.fillStyle = '#00f2ff';
                ctx.strokeStyle = '#000';
                ctx.lineWidth = 3;
                ctx.globalAlpha = 0.6 + Math.sin(Date.now() * 0.005) * 0.4;
                ctx.beginPath();
                ctx.moveTo(50, 0);
                ctx.lineTo(30, -15);
                ctx.lineTo(30, 15);
                ctx.closePath();
                ctx.fill();
                ctx.stroke();
                ctx.restore();
            }
        }

        // ============================================================================
        // UI FUNCTIONS
        // ============================================================================
        function newJob() {
            const restaurant = restaurants[Math.floor(Math.random() * restaurants.length)];
            p.target = { x: restaurant.x, y: restaurant.y, color: '#ff9800', isRestaurant: true };
            p.mode = 'pickup';
            p.fragile = Math.random() < 0.25; // 25% chance of fragile job
            p.fragileHits = 0;
            if (p.fragile) {
                msg(`🥚 FRAGILE JOB AT ${restaurant.name}! BE CAREFUL!`, '#ff9800');
            } else {
                msg(`PICKUP AT ${restaurant.name}!`, "var(--neon-blue)");
            }
        }

        function msg(t, c) {
            const n = document.getElementById('notify');
            n.innerText = t;
            n.style.color = c;
            n.style.opacity = 1;
            setTimeout(() => (n.style.opacity = 0), 2000);
        }

        function toggleShop() {
            const overlay = document.getElementById('shop-overlay');
            overlay.classList.toggle('active');
            if (overlay.classList.contains('active')) {
                updateShop();
            }
        }

        // Event delegation — handle ALL button taps in the shop via touchstart
        // This bypasses the window touchend e.preventDefault() that kills synthetic clicks
        document.getElementById('shop-overlay').addEventListener('touchstart', (e) => {
            const btn = e.target.closest('button.buy-btn');
            if (!btn || btn.disabled) return;
            e.preventDefault();
            e.stopPropagation();
            // Read the action encoded in data attributes
            const action = btn.dataset.action;
            const val    = btn.dataset.val;
            if (action === 'buyD')       { buyD(parseInt(val)); }
            if (action === 'buyUpgrade') { buyUpgrade(val); }
        }, { passive: false });

        function updateShop() {
            document.getElementById('cash').innerText = money;

            let droneHTML = '<div class="card"><h2>🚁 HANGAR - SELECT YOUR RIDE</h2>';
            DRONES.forEach((d, i) => {
                const isOwned = i <= droneIdx;
                const canBuy = money >= d.cost;
                const isActive = droneIdx === i;
                
                droneHTML += `
                <div style="margin-bottom: 10px; padding: 10px; background: ${isActive ? 'rgba(57,255,20,0.2)' : 'rgba(0,0,0,0.4)'}; border-radius: 8px; border: 3px solid ${isActive ? '#39ff14' : '#000'}; box-shadow: 2px 2px 0 #000;">
                    <div style="display: flex; align-items: center; margin-bottom: 6px;">
                        <div style="width: 35px; height: 35px; background: ${d.color}; border: 3px solid #000; border-radius: 5px; margin-right: 10px; box-shadow: 2px 2px 0 #000; flex-shrink: 0;"></div>
                        <div style="flex: 1; min-width: 0;">
                            <strong style="color: ${d.color}; font-size: 14px; text-shadow: 2px 2px 0 #000;">${d.name}</strong><br>
                            <span style="color: #ccc; font-size: 10px;">${d.desc}</span>
                        </div>
                    </div>
                    <div style="color: #fff; font-size: 12px; margin: 6px 0; text-shadow: 1px 1px 0 #000;">
                        ⚡ Speed: <span style="color: #00d4ff;">${d.speed}</span> | 
                        🚀 Accel: <span style="color: #ff9800;">${d.acc}</span>
                    </div>
                    <button class="buy-btn"
                        data-action="buyD" data-val="${i}"
                        style="background: ${isActive ? '#39ff14' : isOwned ? '#00d4ff' : (canBuy ? '#f1c40f' : '#333')}; padding: 10px; margin-top: 6px;"
                        ${!isOwned && !canBuy ? 'disabled' : ''}>
                        ${isActive ? '✓ EQUIPPED' : (isOwned ? 'EQUIP' : (canBuy ? `BUY - $${d.cost}` : `🔒 $${d.cost}`))}
                    </button>
                </div>`;
            });
            droneHTML += '</div>';

            let upHTML = '<div class="card"><h2>⚡ UPGRADES</h2>';
            for (let k in upgrades) {
                const u = upgrades[k];
                const names = {
                    battery:   '🔋 Battery Cells',
                    cooling:   '❄️ Cooling System',
                    winch:     '⚙️ Cargo Winch',
                    gyro:      '🎯 Stability Gyro',
                    nav:       '🧭 Navigation AI',
                    aero:      '🌪️ Aero-Stabilizers',
                    ecm:       '⚡ ECM Shield',
                    shockHook: '🪝 Shock-Absorbing Hook'
                };
                const descs = {
                    battery:   'Increases max battery capacity',
                    cooling:   'Reduces heat buildup from speed',
                    winch:     'Stabilizes cargo swing AND widens pickup speed window',
                    gyro:      'Improves acceleration and handling',
                    nav:       'Shows compass pointing to destination',
                    aero:      'Cuts wind drift impact by 30% per level',
                    ecm:       'Each level adds 33% chance to deflect thrown shoes',
                    shockHook: 'Reduces fragile cargo damage penalty per hit'
                };
                upHTML += `
                <div style="margin-bottom: 10px; padding: 10px; background: rgba(0,0,0,0.4); border-radius: 8px; border: 3px solid #000; box-shadow: 2px 2px 0 #000;">
                    <strong style="color: #00d4ff; text-shadow: 1px 1px 0 #000; font-size: 13px;">${names[k]}</strong><br>
                    <span style="color: #aaa; font-size: 10px;">${descs[k]}</span><br>
                    <span style="color: #fff; text-shadow: 1px 1px 0 #000; font-size: 12px;">Level ${u.lvl}/${u.max}</span>
                    <button class="buy-btn"
                        data-action="buyUpgrade" data-val="${k}"
                        style="background: ${u.lvl >= u.max ? '#39ff14' : (money >= u.cost ? '#f1c40f' : '#333')}; padding: 10px; margin-top: 6px;"
                        ${u.lvl >= u.max || money < u.cost ? 'disabled' : ''}>
                        ${u.lvl >= u.max ? '✓ MAXED OUT' : `Upgrade - $${u.cost}`}
                    </button>
                </div>`;
            }
            upHTML += '</div>';

            document.getElementById('drone-shop').innerHTML = droneHTML;
            document.getElementById('upgrade-shop').innerHTML = upHTML;
        }

        window.buyD = (i) => {
            const drone = DRONES[i];
            const isOwned = i <= droneIdx;
            
            if (isOwned) {
                droneIdx = i;
                msg(`${drone.name} EQUIPPED!`, "var(--neon-green)");
                updateShop();
            } 
            else if (money >= drone.cost) {
                money -= drone.cost;
                droneIdx = i;
                msg(`${drone.name} PURCHASED!`, "var(--neon-green)");
                updateShop();
            }
        };

        // TASK 1: COMPLETE UPGRADE WIRING
        window.buyUpgrade = (key) => {
            const u = upgrades[key];
            if (u.lvl < u.max && money >= u.cost) {
                money -= u.cost;
                u.lvl++;
                u.cost = Math.floor(u.cost * 1.6);

                // Battery upgrade increases max capacity
                if (key === 'battery') {
                    p.batMax += 25;
                    p.bat = p.batMax; // Refill on upgrade
                }
                // Cooling upgrade - effects applied in game loop
                // Winch upgrade - effects applied in cargo rendering
                // Gyro upgrade - effects applied in physics
                // Nav upgrade - effects applied in compass rendering

                updateShop();
                msg(`${key.toUpperCase()} UPGRADED!`, "var(--neon-green)");
            }
        };

        // STEP 1: Guarantee game starts reliably
        window.addEventListener('load', init);
        
        // Fallback for older browsers
        if (document.readyState === 'complete' || document.readyState === 'interactive') {
            setTimeout(init, 1);
        }
    </script>
</body>
</html>
