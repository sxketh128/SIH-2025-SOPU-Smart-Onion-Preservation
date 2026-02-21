<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Smart Onion Preservation Unit Simulator</title>
    <style>
        body { margin: 0; overflow: hidden; font-family: 'Inter', sans-serif; background-color: #1a1a1a; color: #e0e0e0; }
        canvas { display: block; }
        .label {
            position: absolute;
            background: rgba(0, 0, 0, 0.7);
            color: #fff;
            padding: 5px 10px;
            border-radius: 6px;
            font-size: 0.8em;
            pointer-events: none;
            transition: all 0.2s ease-out;
            border: 1px solid rgba(255, 255, 255, 0.2);
        }
        #info-panel {
            position: absolute;
            top: 20px;
            left: 20px;
            background: rgba(0, 0, 0, 0.6);
            padding: 15px;
            border-radius: 12px;
            max-width: 350px;
            pointer-events: none;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.3);
            transition: all 0.3s ease-in-out;
            border: 1px solid rgba(255, 255, 255, 0.1);
        }
        #info-panel h1 { font-size: 2rem; margin-top: 0; margin-bottom: 5px; color: #4CAF50; }
        #info-panel p { font-size: 0.9rem; margin: 0; line-height: 1.4; }
        #instructions {
            position: absolute;
            bottom: 20px;
            left: 50%;
            transform: translateX(-50%);
            background: rgba(0, 0, 0, 0.6);
            padding: 10px 20px;
            border-radius: 12px;
            font-size: 0.9rem;
            color: #b0b0b0;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.3);
        }
    </style>
</head>
<body>

<div id="info-panel">
    <h1>The Smart Onion Unit</h1>
    <p>A smart, solar-powered solution to reduce post-harvest onion loss. Rotate and zoom to explore the design.</p>
</div>
<div id="instructions">
    Use your mouse or finger to rotate and zoom.
</div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js"></script>

<script>
    // --- Scene Setup ---
    let scene, camera, renderer, controls;
    let airParticles;
    const initialCameraPosition = new THREE.Vector3(8, 4, 10);
    const labels = [];

    function init() {
        scene = new THREE.Scene();
        scene.background = new THREE.Color(0x1a1a1a);

        camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
        camera.position.copy(initialCameraPosition);

        renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.setPixelRatio(window.devicePixelRatio);
        document.body.appendChild(renderer.domElement);

        // --- Lighting ---
        const ambientLight = new THREE.AmbientLight(0x404040, 2);
        scene.add(ambientLight);

        const directionalLight = new THREE.DirectionalLight(0xffffff, 1.5);
        directionalLight.position.set(5, 10, 7);
        scene.add(directionalLight);

        // --- Orbit Controls ---
        controls = new THREE.OrbitControls(camera, renderer.domElement);
        controls.enableDamping = true;
        controls.dampingFactor = 0.05;
        controls.screenSpacePanning = false;
        controls.maxPolarAngle = Math.PI / 2;
        controls.minDistance = 5;
        controls.maxDistance = 20;

        createModel();
        
        window.addEventListener('resize', onWindowResize, false);
        animate();
    }

    function createModel() {
        // --- Materials ---
        const shedMaterial = new THREE.MeshStandardMaterial({ color: 0x8B4513, roughness: 0.8, metalness: 0.1 });
        const onionMaterial = new THREE.MeshPhongMaterial({ color: 0xffa500, shininess: 30 });
        const solarPanelMaterial = new THREE.MeshBasicMaterial({ color: 0x222222, side: THREE.DoubleSide });
        const chamberMaterial = new THREE.MeshStandardMaterial({ color: 0x5a5a5a, metalness: 0.2, roughness: 0.6 });
        const sensorMaterial = new THREE.MeshPhongMaterial({ color: 0x00aaff, shininess: 80 });
        const fanMaterial = new THREE.MeshStandardMaterial({ color: 0xcccccc, metalness: 0.8, roughness: 0.3 });
        const coolingPanelMaterial = new THREE.MeshBasicMaterial({ color: 0x00ffff, transparent: true, opacity: 0.3 });
        const silicaGelMaterial = new THREE.MeshPhongMaterial({ color: 0xadd8e6, shininess: 50 });
        const technicalChamberMaterial = new THREE.MeshStandardMaterial({ color: 0x2e8b57, metalness: 0.2, roughness: 0.6, transparent: true, opacity: 0.7 });
        const secondChamberMaterial = new THREE.MeshStandardMaterial({ color: 0x800080, metalness: 0.2, roughness: 0.6, transparent: true, opacity: 0.7 });
        const tDoorMaterial = new THREE.MeshStandardMaterial({ color: 0x6a0dad, metalness: 0.8, roughness: 0.4 });
        const groundMaterial = new THREE.MeshBasicMaterial({ color: 0x5c4a31 });
        const wireMaterial = new THREE.LineBasicMaterial({ color: 0xffff00 });

        // --- Ground Plane ---
        const groundGeometry = new THREE.BoxGeometry(20, 0.1, 20);
        const groundMesh = new THREE.Mesh(groundGeometry, groundMaterial);
        groundMesh.position.y = -3;
        scene.add(groundMesh);

        // --- Shed and Walls ---
        const shedGeometry = new THREE.BoxGeometry(10, 5, 8);
        const shedMesh = new THREE.Mesh(shedGeometry, new THREE.MeshBasicMaterial({ color: 0x8B4513, transparent: true, opacity: 0.1 }));
        shedMesh.position.y = 2.5;
        scene.add(shedMesh);
        
        const roofGeometry = new THREE.BoxGeometry(10.2, 0.2, 8.2);
        const roofMesh = new THREE.Mesh(roofGeometry, shedMaterial);
        roofMesh.position.y = 5.1;
        scene.add(roofMesh);

        const wallGeometry = new THREE.BoxGeometry(0.1, 5, 8);
        const wallMesh1 = new THREE.Mesh(wallGeometry, shedMaterial);
        wallMesh1.position.set(-5, 2.5, 0);
        scene.add(wallMesh1);
        
        const wallMesh2 = new THREE.Mesh(wallGeometry, shedMaterial);
        wallMesh2.position.set(5, 2.5, 0);
        scene.add(wallMesh2);
        
        const wall3Geometry = new THREE.BoxGeometry(10, 5, 0.1);
        const wallMesh3 = new THREE.Mesh(wall3Geometry, shedMaterial);
        wallMesh3.position.set(0, 2.5, -4);
        scene.add(wallMesh3);

        const sideVentGeometry = new THREE.BoxGeometry(10, 5, 0.1);
        const sideVentMesh = new THREE.Mesh(sideVentGeometry, new THREE.MeshStandardMaterial({ color: 0x333333, transparent: true, opacity: 0.8 }));
        sideVentMesh.position.set(0, 2.5, 4);
        scene.add(sideVentMesh);

        // Exhaust fans on side walls
        const exhaustFanGeometry = new THREE.CylinderGeometry(0.5, 0.5, 0.2, 32);
        const exhaustFanMesh1 = new THREE.Mesh(exhaustFanGeometry, fanMaterial);
        exhaustFanMesh1.position.set(-5, 2.5, 2);
        exhaustFanMesh1.rotation.z = Math.PI / 2;
        scene.add(exhaustFanMesh1);
        addLabel(exhaustFanMesh1, "Exhaust Fan");

        const exhaustFanMesh2 = new THREE.Mesh(exhaustFanGeometry, fanMaterial);
        exhaustFanMesh2.position.set(5, 2.5, -2);
        exhaustFanMesh2.rotation.z = Math.PI / 2;
        scene.add(exhaustFanMesh2);
        addLabel(exhaustFanMesh2, "Exhaust Fan");
        
        // --- Raised Floor with Gaps ---
        const floorGeometry = new THREE.BoxGeometry(9, 0.1, 7);
        const floorMesh = new THREE.Mesh(floorGeometry, new THREE.MeshStandardMaterial({ color: 0x7a7a7a, roughness: 0.8 }));
        floorMesh.position.y = 0;
        scene.add(floorMesh);

        const gapsGeometry = new THREE.BoxGeometry(0.1, 0.1, 7); // Made gaps very small
        const gapsMaterial = new THREE.MeshBasicMaterial({ color: 0x1a1a1a });
        const gapCount = 40; // Increased gap count
        for (let i = 0; i < gapCount; i++) {
            const gap = new THREE.Mesh(gapsGeometry, gapsMaterial);
            gap.position.set(4.5 - i * 0.22, 0.05, 0); // Adjusted spacing
            scene.add(gap);
        }

        // --- Onions on the Floor ---
        const onionsGroup = new THREE.Group();
        const onionGeometry = new THREE.SphereGeometry(0.12, 16, 16);
        const onionSpacing = 0.1; // Reduced spacing to make it denser
        const gridCount = 40; // Increased grid count

        for (let i = 0; i < gridCount; i++) {
            for (let j = 0; j < gridCount; j++) {
                const onion = new THREE.Mesh(onionGeometry, onionMaterial);
                onion.position.x = (i - gridCount / 2) * onionSpacing;
                onion.position.z = (j - gridCount / 2) * onionSpacing;
                onion.position.y = 0.2 + Math.random() * 0.1;
                onionsGroup.add(onion);
            }
        }
        scene.add(onionsGroup);
        
        // --- Underground Air Chamber ---
        const airChamberGeometry = new THREE.BoxGeometry(9, 2, 7);
        const airChamberMesh = new THREE.Mesh(airChamberGeometry, chamberMaterial);
        airChamberMesh.position.set(0, -1, 0);
        scene.add(airChamberMesh);
        addLabel(airChamberMesh, "Underground Air Chamber");

        // --- Technical Chamber ---
        const techChamberGeometry = new THREE.BoxGeometry(4, 2, 4);
        const techChamberMesh = new THREE.Mesh(techChamberGeometry, technicalChamberMaterial);
        techChamberMesh.position.set(7, -1, 0);
        scene.add(techChamberMesh);
        addLabel(techChamberMesh, "Technical Chamber");

        // --- Additional Chamber on opposite side ---
        const secondChamberGeometry = new THREE.BoxGeometry(4, 2, 4);
        const secondChamberMesh = new THREE.Mesh(secondChamberGeometry, secondChamberMaterial);
        secondChamberMesh.position.set(-7, -1, 0);
        scene.add(secondChamberMesh);
        addLabel(secondChamberMesh, "Sorting Chamber");


        // --- T-shaped Door ---
        const tDoorGeometry = new THREE.BoxGeometry(1, 2, 0.5);
        const tDoorMesh = new THREE.Mesh(tDoorGeometry, tDoorMaterial);
        tDoorMesh.position.set(5, -1, -2);
        scene.add(tDoorMesh);
        addLabel(tDoorMesh, "Air Intake");

        // --- Connectors from T-Door to Chambers ---
        const tToAirPipeGeo = new THREE.BoxGeometry(1, 0.5, 4);
        const tToAirPipe = new THREE.Mesh(tToAirPipeGeo, tDoorMaterial);
        tToAirPipe.position.set(5.5, -1, 0);
        scene.add(tToAirPipe);
        
        const tToTechPipeGeo = new THREE.BoxGeometry(1, 0.5, 4);
        const tToTechPipe = new THREE.Mesh(tToTechPipeGeo, tDoorMaterial);
        tToTechPipe.position.set(6.5, -1, 2);
        scene.add(tToTechPipe);
        
        // --- Raspberry Pi and Sensors ---
        const rpGeometry = new THREE.BoxGeometry(0.5, 0.1, 0.3);
        const rpMaterial = new THREE.MeshBasicMaterial({ color: 0xcc0000 });
        const raspberryPi = new THREE.Mesh(rpGeometry, rpMaterial);
        raspberryPi.position.set(7, -1, 0.5);
        scene.add(raspberryPi);
        addLabel(raspberryPi, "Raspberry Pi");

        const sensorGeometry = new THREE.BoxGeometry(0.1, 0.1, 0.1);
        const dhtSensor = new THREE.Mesh(sensorGeometry, sensorMaterial);
        dhtSensor.position.set(7.5, -1, 0.2);
        scene.add(dhtSensor);
        addLabel(dhtSensor, "DHT11 Sensor");

        // New sensors inside the main chamber
        const sensor2 = new THREE.Mesh(sensorGeometry, sensorMaterial);
        sensor2.position.set(-4.5, 2.5, 0);
        scene.add(sensor2);
        addLabel(sensor2, "Internal Sensor");

        const sensor3 = new THREE.Mesh(sensorGeometry, sensorMaterial);
        sensor3.position.set(4.5, 2.5, 0);
        scene.add(sensor3);
        addLabel(sensor3, "Internal Sensor");

        const sensor4 = new THREE.Mesh(sensorGeometry, sensorMaterial);
        sensor4.position.set(0, 2.5, -3.5);
        scene.add(sensor4);
        addLabel(sensor4, "Internal Sensor");

        // --- Cooling Panels and Fans ---
        const panelGeometry = new THREE.BoxGeometry(1.5, 0.1, 0.5);
        const panelMesh = new THREE.Mesh(panelGeometry, coolingPanelMaterial);
        panelMesh.position.set(7, -0.5, -1.5);
        scene.add(panelMesh);
        addLabel(panelMesh, "Cooling Panels");
        
        const fanGeometry = new THREE.CylinderGeometry(0.5, 0.1, 0.1, 32);
        const fanMesh = new THREE.Mesh(fanGeometry, fanMaterial);
        fanMesh.position.set(7, -0.5, 1.5);
        scene.add(fanMesh);
        addLabel(fanMesh, "Air-Pushing Fans");

        // --- Silica Gel ---
        const gelGeometry = new THREE.SphereGeometry(0.05, 8, 8);
        const gelGroup = new THREE.Group();
        for (let i = 0; i < 50; i++) {
            const gel = new THREE.Mesh(gelGeometry, silicaGelMaterial);
            gel.position.x = (Math.random() - 0.5) * 8;
            gel.position.z = (Math.random() - 0.5) * 6;
            gel.position.y = 0.5 + Math.random() * 0.1;
            gelGroup.add(gel);
        }
        scene.add(gelGroup);
        addLabel(gelGroup, "Dehumidifying Agents");

        // --- Solar Panels and Wires ---
        const solarPanelGeo = new THREE.BoxGeometry(5, 0.1, 3);
        const solarPanel = new THREE.Mesh(solarPanelGeo, solarPanelMaterial);
        solarPanel.position.set(0, 5.5, -2);
        solarPanel.rotation.x = -Math.PI / 6;
        scene.add(solarPanel);
        addLabel(solarPanel, "Solar Panel");

        const wirePoints = [];
        wirePoints.push(new THREE.Vector3(0, 5.5, -2)); // From solar panel on roof
        wirePoints.push(new THREE.Vector3(5, 5.1, -2)); // To the edge of the roof
        wirePoints.push(new THREE.Vector3(5.1, -1, -2)); // Down the side of the shed
        wirePoints.push(new THREE.Vector3(6.5, -1, 1)); // To the technical chamber
        const wireGeometry = new THREE.BufferGeometry().setFromPoints(wirePoints);
        const wire = new THREE.Line(wireGeometry, wireMaterial);
        scene.add(wire);

        // Animate the air particles
        airParticles = new THREE.Group();
        const particleGeometry = new THREE.SphereGeometry(0.02, 8, 8);
        const particleMaterial = new THREE.MeshBasicMaterial({ color: 0x4ddb4d });

        for (let i = 0; i < 300; i++) {
            const particle = new THREE.Mesh(particleGeometry, particleMaterial);
            particle.position.x = (Math.random() - 0.5) * 8;
            particle.position.z = (Math.random() - 0.5) * 6;
            particle.position.y = -2; // Start in the chamber
            particle.userData.speed = 0.005 + Math.random() * 0.01;
            airParticles.add(particle);
        }
        scene.add(airParticles);

        function addLabel(object, text) {
            const div = document.createElement('div');
            div.className = 'label';
            div.textContent = text;
            document.body.appendChild(div);
            labels.push({ element: div, object: object });
        }
    }

    // --- Animation Loop ---
    function animate() {
        requestAnimationFrame(animate);

        // Animate air particles from chamber up through gaps
        airParticles.children.forEach(particle => {
            if (particle.position.y < -0.9) {
                // Inside chamber, move up
                particle.position.y += particle.userData.speed;
            } else if (particle.position.y < 0.2) {
                // Passing through gaps, a bit faster
                particle.position.y += particle.userData.speed * 2;
            } else {
                // Onions level, move slower and dissipate
                particle.position.y += particle.userData.speed * 0.5;
            }

            if (particle.position.y > 1) {
                particle.position.y = -2; // Reset to start
            }
        });

        // Update labels
        labels.forEach(label => {
            const vector = new THREE.Vector3();
            label.object.getWorldPosition(vector);
            const position = vector.project(camera);

            const x = (position.x * .5 + .5) * window.innerWidth;
            const y = (position.y * -.5 + .5) * window.innerHeight;

            label.element.style.transform = `translate(-50%, -50%) translate(${x}px,${y}px)`;
        });

        controls.update();
        renderer.render(scene, camera);
    }

    // --- Handle Window Resize ---
    function onWindowResize() {
        camera.aspect = window.innerWidth / window.innerHeight;
        camera.updateProjectionMatrix();
        renderer.setSize(window.innerWidth, window.innerHeight);
    }

    window.onload = init;
</script>
</body>
</html>
