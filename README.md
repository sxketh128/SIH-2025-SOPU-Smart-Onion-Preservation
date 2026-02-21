<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Smart Onion Preservation Unit Simulator (Fixed)</title>
    <style>
        body { margin: 0; overflow: hidden; font-family: 'Inter', sans-serif; color: #e0e0e0; background-color: #111; }
        canvas { display: block; }
        .label {
            position: absolute;
            background: rgba(0, 0, 0, 0.8);
            color: #fff;
            padding: 4px 8px;
            border-radius: 4px;
            font-size: 11px;
            pointer-events: none;
            border: 1px solid #4CAF50;
            white-space: nowrap;
            z-index: 100;
            transform: translate(-50%, -50%);
            text-shadow: 0 0 4px black;
        }
        #info-panel {
            position: absolute;
            top: 20px;
            left: 20px;
            background: rgba(0, 0, 0, 0.7);
            padding: 15px 20px;
            border-radius: 12px;
            max-width: 300px;
            pointer-events: none;
            border: 1px solid #4CAF50;
            backdrop-filter: blur(5px);
            z-index: 200;
        }
        #info-panel h1 { font-size: 1.5rem; margin: 0 0 5px 0; color: #4CAF50; font-weight: 500; }
        #info-panel p { font-size: 0.85rem; margin: 0; line-height: 1.5; color: #ddd; }
        #instructions {
            position: absolute;
            bottom: 20px;
            left: 50%;
            transform: translateX(-50%);
            background: rgba(0, 0, 0, 0.6);
            padding: 8px 20px;
            border-radius: 30px;
            font-size: 0.9rem;
            color: #ccc;
            border: 1px solid #4CAF50;
            z-index: 200;
            letter-spacing: 0.5px;
        }
    </style>
</head>
<body>
<div id="info-panel">
    <h1>SOPU Simulator</h1>
    <p>Smart Onion Preservation Unit. Solar-powered IoT monitoring and bottom-up ventilation system. All components labeled.</p>
</div>
<div id="instructions">🖱️ Drag to Rotate • Scroll to Zoom • Hover labels indicate components</div>

<!-- Use specific version of Three.js and OrbitControls from CDN (fixed path) -->
<script type="importmap">
    {
        "imports": {
            "three": "https://unpkg.com/three@0.128.0/build/three.module.js",
            "three/addons/": "https://unpkg.com/three@0.128.0/examples/jsm/"
        }
    }
</script>

<script type="module">
    import * as THREE from 'three';
    import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

    // --- setup scene, camera, renderer ---
    const scene = new THREE.Scene();
    scene.background = new THREE.Color(0x111a22); // darker background to make labels pop
    
    const camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000);
    camera.position.set(14, 7, 16);
    camera.lookAt(0, 2, 0);

    const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: false });
    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.setPixelRatio(window.devicePixelRatio);
    renderer.shadowMap.enabled = true; // optional but adds depth
    document.body.appendChild(renderer.domElement);

    // --- lighting ---
    const ambientLight = new THREE.AmbientLight(0xffffff, 0.6);
    scene.add(ambientLight);
    
    const sunLight = new THREE.DirectionalLight(0xfff5e6, 1);
    sunLight.position.set(10, 25, 10);
    sunLight.castShadow = true;
    sunLight.shadow.mapSize.width = 1024;
    sunLight.shadow.mapSize.height = 1024;
    scene.add(sunLight);
    
    const fillLight = new THREE.PointLight(0x446688, 0.5);
    fillLight.position.set(-5, 5, 10);
    scene.add(fillLight);

    // --- controls ---
    const controls = new OrbitControls(camera, renderer.domElement);
    controls.enableDamping = true;
    controls.dampingFactor = 0.06;
    controls.maxPolarAngle = Math.PI / 2;
    controls.target.set(0, 2, 0);
    controls.update();

    // --- data arrays for animations and labels ---
    const fanBlades = [];
    const labels = []; // { element, object }
    let airParticles = null;

    // --- helper to create labels ---
    function addLabel(parentObject, text, offsetY = 0.8) {
        const div = document.createElement('div');
        div.className = 'label';
        div.textContent = text;
        document.body.appendChild(div);
        labels.push({ 
            el: div, 
            obj: parentObject,
            offset: new THREE.Vector3(0, offsetY, 0) // slight offset above object
        });
    }

    // --- build the entire model ---
    function createModel() {
        // materials
        const matShed = new THREE.MeshStandardMaterial({ color: 0x8B5A2B, roughness: 0.7 }); // wood
        const matOnion = new THREE.MeshPhongMaterial({ color: 0xffaa33, emissive: 0x331100 });
        const matSolar = new THREE.MeshStandardMaterial({ color: 0x1e2b3a, emissive: 0x102030 });
        const matChamber = new THREE.MeshStandardMaterial({ color: 0x556677, roughness: 0.6 });
        const matTech = new THREE.MeshStandardMaterial({ color: 0x2a5a4a, transparent: true, opacity: 0.5 });
        const matSort = new THREE.MeshStandardMaterial({ color: 0x6a3e8a, transparent: true, opacity: 0.5 });
        const matGround = new THREE.MeshStandardMaterial({ color: 0x3a4a3a, roughness: 0.9 });
        const matWire = new THREE.LineBasicMaterial({ color: 0xffaa00 });
        const matBat1 = new THREE.MeshStandardMaterial({ color: 0xc0392b }); // red
        const matBat2 = new THREE.MeshStandardMaterial({ color: 0x27ae60 }); // green
        const matCool = new THREE.MeshStandardMaterial({ color: 0x2c82b0, transparent: true, opacity: 0.7 }); // blue
        const matSilica = new THREE.MeshPhongMaterial({ color: 0xb0c4de, emissive: 0x112233 }); // light blue

        // Ground
        const ground = new THREE.Mesh(new THREE.BoxGeometry(40, 0.5, 40), matGround);
        ground.position.y = -3.0;
        ground.receiveShadow = true;
        scene.add(ground);

        // Main shed structure (walls and roof)
        // Roof (visible top)
        const roof = new THREE.Mesh(new THREE.BoxGeometry(11.2, 0.4, 9.2), matShed);
        roof.position.y = 5.2;
        roof.castShadow = true;
        scene.add(roof);
        addLabel(roof, "Insulated Roof");

        // Walls
        const wallBack = new THREE.Mesh(new THREE.BoxGeometry(11, 5, 0.3), matShed);
        wallBack.position.set(0, 2.5, -4.5);
        wallBack.castShadow = true;
        scene.add(wallBack);

        const wallLeft = new THREE.Mesh(new THREE.BoxGeometry(0.3, 5, 9), matShed);
        wallLeft.position.set(-5.5, 2.5, 0);
        wallLeft.castShadow = true;
        scene.add(wallLeft);

        const wallRight = new THREE.Mesh(new THREE.BoxGeometry(0.3, 5, 9), matShed);
        wallRight.position.set(5.5, 2.5, 0);
        wallRight.castShadow = true;
        scene.add(wallRight);

        // Front wall with ventilation (semi-transparent)
        const ventWall = new THREE.Mesh(new THREE.BoxGeometry(11, 5, 0.2), new THREE.MeshStandardMaterial({ color: 0x888888, transparent: true, opacity: 0.3 }));
        ventWall.position.set(0, 2.5, 4.5);
        ventWall.receiveShadow = true;
        scene.add(ventWall);
        addLabel(ventWall, "Ventilation Wall (Open)");

        // Fans (3 holes with rotating blades)
        const holeGeo = new THREE.TorusGeometry(0.5, 0.08, 16, 32);
        const bladeGeo = new THREE.BoxGeometry(0.9, 0.1, 0.1);
        const bladeMat = new THREE.MeshStandardMaterial({ color: 0xcccccc, emissive: 0x222222 });

        for (let i = 0; i < 3; i++) {
            const xPos = -3 + i * 3;
            // Hole ring
            const hole = new THREE.Mesh(holeGeo, new THREE.MeshStandardMaterial({ color: 0x333333, emissive: 0x111111 }));
            hole.rotation.x = Math.PI / 2; // lie flat on Z axis? Actually we want ring facing outward
            hole.rotation.y = 0;
            hole.rotation.z = 0;
            hole.position.set(xPos, 2.5, 4.45);
            scene.add(hole);

            // Fan blades group
            const bladeGroup = new THREE.Group();
            for (let j = 0; j < 3; j++) {
                const blade = new THREE.Mesh(bladeGeo, bladeMat);
                blade.rotation.z = (j * Math.PI * 2) / 3;
                blade.position.set(0, 0, 0);
                blade.castShadow = true;
                bladeGroup.add(blade);
            }
            bladeGroup.position.set(xPos, 2.5, 4.5); // slightly in front
            scene.add(bladeGroup);
            fanBlades.push(bladeGroup);
            
            if (i === 1) addLabel(bladeGroup, "Exhaust Fans (Active)");
        }

        // Floor inside shed
        const floor = new THREE.Mesh(new THREE.BoxGeometry(10.4, 0.2, 8.4), new THREE.MeshStandardMaterial({ color: 0x5a6a7a }));
        floor.position.y = 0.0;
        floor.receiveShadow = true;
        scene.add(floor);
        addLabel(floor, "Perforated Floor");

        // Onions bulk (many spheres)
        const onionGroup = new THREE.Group();
        const onionGeo = new THREE.SphereGeometry(0.22, 8);
        for (let i = 0; i < 350; i++) {
            const o = new THREE.Mesh(onionGeo, matOnion);
            o.position.set(
                (Math.random() - 0.5) * 8,
                0.2 + Math.random() * 1.0,
                (Math.random() - 0.5) * 6
            );
            o.castShadow = true;
            o.receiveShadow = true;
            onionGroup.add(o);
        }
        scene.add(onionGroup);
        addLabel(onionGroup, "Onions Storage (~350 units)", 1.2);

        // Underground air chamber (visible through floor)
        const airChamber = new THREE.Mesh(new THREE.BoxGeometry(10, 2.2, 8), new THREE.MeshStandardMaterial({ color: 0x445566, emissive: 0x112233, transparent: true, opacity: 0.5 }));
        airChamber.position.y = -1.2;
        airChamber.receiveShadow = true;
        scene.add(airChamber);
        addLabel(airChamber, "Underground Air Plenum", -0.5);

        // Technical chamber (right)
        const techChamber = new THREE.Mesh(new THREE.BoxGeometry(4.5, 3.5, 4.5), matTech);
        techChamber.position.set(8.0, -1.2, 0);
        techChamber.castShadow = true;
        scene.add(techChamber);
        addLabel(techChamber, "Technical Chamber (IoT)", 1.0);

        // Sorting chamber (left)
        const sortChamber = new THREE.Mesh(new THREE.BoxGeometry(4.5, 3.5, 4.5), matSort);
        sortChamber.position.set(-8.0, -1.2, 0);
        sortChamber.castShadow = true;
        scene.add(sortChamber);
        addLabel(sortChamber, "Sorting & Pre-treatment", 1.0);

        // Batteries inside tech chamber
        const bat1 = new THREE.Mesh(new THREE.BoxGeometry(0.9, 1.2, 0.8), matBat1);
        bat1.position.set(7.2, -1.4, -0.8);
        bat1.castShadow = true;
        scene.add(bat1);
        addLabel(bat1, "Battery Bank A");

        const bat2 = new THREE.Mesh(new THREE.BoxGeometry(0.9, 1.2, 0.8), matBat2);
        bat2.position.set(7.2, -1.4, 0.8);
        bat2.castShadow = true;
        scene.add(bat2);
        addLabel(bat2, "Battery Bank B");

        // Cooling panel (radiator)
        const coolPanel = new THREE.Mesh(new THREE.BoxGeometry(0.2, 2.0, 2.5), matCool);
        coolPanel.position.set(9.8, -1.0, 0);
        coolPanel.castShadow = true;
        scene.add(coolPanel);
        addLabel(coolPanel, "Cooling Heat Exchanger");

        // Raspberry Pi / controller (small green board)
        const piGeo = new THREE.BoxGeometry(0.8, 0.15, 1.2);
        const piMat = new THREE.MeshStandardMaterial({ color: 0x2ecc71 });
        const pi = new THREE.Mesh(piGeo, piMat);
        pi.position.set(8.5, 0.2, 1.2);
        pi.castShadow = true;
        scene.add(pi);
        addLabel(pi, "Raspberry Pi & Sensors");

        // Silica gel packets (small spheres)
        const silicaGroup = new THREE.Group();
        const sGeo = new THREE.SphereGeometry(0.1, 5);
        for (let i = 0; i < 40; i++) {
            const s = new THREE.Mesh(sGeo, matSilica);
            s.position.set(
                (Math.random() - 0.5) * 7,
                0.4 + Math.random() * 1.0,
                (Math.random() - 0.5) * 5
            );
            silicaGroup.add(s);
        }
        scene.add(silicaGroup);
        addLabel(silicaGroup, "Silica Gel Sachets", 1.3);

        // Solar panels on roof
        const solarGroup = new THREE.Group();
        const panelMat = new THREE.MeshStandardMaterial({ color: 0x1a2b3c, emissive: 0x102030 });
        for (let k = 0; k < 4; k++) {
            const panel = new THREE.Mesh(new THREE.BoxGeometry(1.8, 0.1, 1.2), panelMat);
            panel.position.set(-3 + k * 2.2, 5.45, -1 + (k%2)*2);
            panel.rotation.y = 0.1;
            panel.castShadow = true;
            solarGroup.add(panel);
        }
        scene.add(solarGroup);
        addLabel(solarGroup, "Solar Array (400W)", 0.6);

        // Wiring from solar to tech chamber (yellow line)
        const points = [
            new THREE.Vector3(-1, 5.5, 0),
            new THREE.Vector3(3, 5.5, 0),
            new THREE.Vector3(5, 4.0, 0),
            new THREE.Vector3(7, 0.5, 0),
            new THREE.Vector3(7.5, -1.0, 0)
        ];
        const wireGeo = new THREE.BufferGeometry().setFromPoints(points);
        const wireLine = new THREE.Line(wireGeo, matWire);
        scene.add(wireLine);
        addLabel(wireLine, "Power & Data Cable");

        // Airflow particles (rising from air chamber)
        airParticles = new THREE.Group();
        const pGeo = new THREE.SphereGeometry(0.05, 4);
        const pMat = new THREE.MeshStandardMaterial({ color: 0xaaccff, emissive: 0x224466 });
        for (let i = 0; i < 120; i++) {
            const p = new THREE.Mesh(pGeo, pMat);
            p.position.set(
                (Math.random() - 0.5) * 8,
                -1.5 + Math.random() * 2.5,
                (Math.random() - 0.5) * 6
            );
            p.userData = { speed: 0.01 + Math.random() * 0.02, startY: p.position.y };
            airParticles.add(p);
        }
        scene.add(airParticles);
        addLabel(airParticles, "Airflow (bottom-up ventilation)", 0.5);
    }

    // --- build the scene ---
    createModel();

    // --- window resize handler ---
    window.addEventListener('resize', () => {
        camera.aspect = window.innerWidth / window.innerHeight;
        camera.updateProjectionMatrix();
        renderer.setSize(window.innerWidth, window.innerHeight);
    });

    // --- animation loop ---
    function animate() {
        requestAnimationFrame(animate);

        // rotate fan blades
        fanBlades.forEach(blade => {
            blade.rotation.z += 0.2;
        });

        // animate air particles upward
        if (airParticles) {
            airParticles.children.forEach(p => {
                p.position.y += p.userData.speed;
                if (p.position.y > 2.0) {
                    p.position.y = -1.8;
                }
            });
        }

        // update label positions (project 3D to 2D)
        labels.forEach(item => {
            const obj = item.obj;
            // compute world position of object center plus offset
            const worldPos = obj.position.clone();
            if (obj.parent) {
                obj.parent.localToWorld(worldPos);
            }
            // add manual offset (upwards)
            worldPos.y += 0.8; 

            // project to screen
            worldPos.project(camera);

            const x = (worldPos.x * 0.5 + 0.5) * window.innerWidth;
            const y = (-worldPos.y * 0.5 + 0.5) * window.innerHeight;

            // check if behind camera or far outside
            if (worldPos.z < 1 && x > 0 && x < window.innerWidth && y > 0 && y < window.innerHeight) {
                item.el.style.display = 'block';
                item.el.style.left = x + 'px';
                item.el.style.top = y + 'px';
            } else {
                item.el.style.display = 'none';
            }
        });

        controls.update();
        renderer.render(scene, camera);
    }

    animate();

    // small fix: initial label display might be improved by calling update once after a delay
    setTimeout(() => {
        // force a quick camera move to recalc labels
        camera.position.set(14, 7, 16);
        controls.update();
    }, 100);
</script>
</body>
</html>
