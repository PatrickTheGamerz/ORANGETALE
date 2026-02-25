<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Web 3D Tycoon Prototype</title>
    <style>
        /* CSS to make the game take up the whole screen */
        body { margin: 0; overflow: hidden; background-color: #87CEEB; font-family: Arial, sans-serif; }
        
        /* The User Interface on top of the 3D game */
        #ui-container {
            position: absolute;
            top: 20px;
            left: 20px;
            background: rgba(0, 0, 0, 0.7);
            color: white;
            padding: 15px;
            border-radius: 8px;
            z-index: 10;
        }
        
        button {
            background-color: #28a745;
            color: white;
            border: none;
            padding: 10px 15px;
            font-size: 16px;
            cursor: pointer;
            border-radius: 5px;
            font-weight: bold;
        }
        
        button:hover { background-color: #218838; }
    </style>
    
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
</head>
<body>

    <div id="ui-container">
        <h2>Mini 3D Tycoon</h2>
        <p>🎮 Use <b>W A S D</b> to move</p>
        <button id="buyDropper">Spawn Tycoon Dropper ($0)</button>
        <p>Blocks Dropped: <span id="score">0</span></p>
    </div>

    <script>
        // 1. SET UP THE 3D SCENE, CAMERA, AND RENDERER
        const scene = new THREE.Scene();
        scene.background = new THREE.Color(0x87CEEB); // Sky blue background
        
        const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        document.body.appendChild(renderer.domElement);

        // 2. ADD LIGHTING SO WE CAN SEE 3D SHAPES
        const ambientLight = new THREE.AmbientLight(0xffffff, 0.6);
        scene.add(ambientLight);
        const directionalLight = new THREE.DirectionalLight(0xffffff, 0.8);
        directionalLight.position.set(10, 20, 10);
        scene.add(directionalLight);

        // 3. CREATE THE GREEN BASEPLATE (The Floor)
        const floorGeometry = new THREE.PlaneGeometry(100, 100);
        const floorMaterial = new THREE.MeshStandardMaterial({ color: 0x228B22 });
        const floor = new THREE.Mesh(floorGeometry, floorMaterial);
        floor.rotation.x = -Math.PI / 2; // Lay it flat
        scene.add(floor);

        // 4. CREATE THE PLAYER (Blue Block)
        const playerGeometry = new THREE.BoxGeometry(1, 1, 1);
        const playerMaterial = new THREE.MeshStandardMaterial({ color: 0x0000ff });
        const player = new THREE.Mesh(playerGeometry, playerMaterial);
        player.position.y = 0.5; // Rest on top of the floor
        scene.add(player);

        // Position Camera slightly behind and above the player
        camera.position.set(0, 4, 6);

        // 5. PLAYER MOVEMENT CONTROLS (W, A, S, D)
        const keys = { w: false, a: false, s: false, d: false };
        const speed = 0.15;

        window.addEventListener('keydown', (event) => {
            if (keys.hasOwnProperty(event.key.toLowerCase())) keys[event.key.toLowerCase()] = true;
        });
        window.addEventListener('keyup', (event) => {
            if (keys.hasOwnProperty(event.key.toLowerCase())) keys[event.key.toLowerCase()] = false;
        });

        // 6. TYCOON DROPPER MECHANIC
        const droppedBlocks = [];
        let score = 0;

        document.getElementById('buyDropper').addEventListener('click', () => {
            // Create a small red block
            const dropGeometry = new THREE.BoxGeometry(0.5, 0.5, 0.5);
            const dropMaterial = new THREE.MeshStandardMaterial({ color: 0xff0000 });
            const drop = new THREE.Mesh(dropGeometry, dropMaterial);
            
            // Spawn it in the sky near the player
            drop.position.set(player.position.x + (Math.random() * 4 - 2), 10, player.position.z - 2);
            scene.add(drop);
            droppedBlocks.push(drop);

            // Update UI Score
            score++;
            document.getElementById('score').innerText = score;
        });

        // 7. THE GAME LOOP (Runs 60 times a second to animate everything)
        function animate() {
            requestAnimationFrame(animate);

            // Move Player
            if (keys.w) player.position.z -= speed;
            if (keys.s) player.position.z += speed;
            if (keys.a) player.position.x -= speed;
            if (keys.d) player.position.x += speed;

            // Make Camera follow the player
            camera.position.x = player.position.x;
            camera.position.z = player.position.z + 6;
            camera.lookAt(player.position);

            // Animate Tycoon Blocks falling (Basic Gravity)
            droppedBlocks.forEach(block => {
                if (block.position.y > 0.25) {
                    block.position.y -= 0.1; // Fall down
                }
            });

            // Render the scene
            renderer.render(scene, camera);
        }

        // Handle window resizing
        window.addEventListener('resize', () => {
            renderer.setSize(window.innerWidth, window.innerHeight);
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
        });

        // Start the game loop
        animate();
    </script>
</body>
</html>
