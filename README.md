<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Web Obby & Tycoon Engine</title>
    <style>
        body { margin: 0; overflow: hidden; background-color: #87CEEB; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; }
        
        /* The Game Joining Menu Screen */
        #join-menu {
            position: absolute;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.8);
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            color: white;
            z-index: 100;
        }

        #joinBtn {
            background-color: #00b06f;
            color: white;
            border: none;
            padding: 20px 40px;
            font-size: 24px;
            font-weight: bold;
            border-radius: 8px;
            cursor: pointer;
            text-transform: uppercase;
            box-shadow: 0 4px 6px rgba(0,0,0,0.3);
            transition: 0.2s;
        }

        #joinBtn:hover { background-color: #008f5a; transform: scale(1.05); }

        /* In-Game UI */
        #crosshair {
            position: absolute;
            top: 50%; left: 50%;
            width: 10px; height: 10px;
            background-color: white;
            border-radius: 50%;
            transform: translate(-50%, -50%);
            pointer-events: none;
            z-index: 10;
        }

        #controls-help {
            position: absolute;
            top: 10px; left: 10px;
            color: white;
            text-shadow: 1px 1px 2px black;
            font-size: 14px;
        }
    </style>
    
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/PointerLockControls.js"></script>
</head>
<body>

    <div id="join-menu">
        <h1 style="font-size: 48px; margin-bottom: 10px;">Tower of Obby Prototype</h1>
        <p style="margin-bottom: 30px; font-size: 18px;">Click below to join the server.</p>
        <button id="joinBtn">Play Game</button>
    </div>

    <div id="crosshair" style="display: none;"></div>
    <div id="controls-help" style="display: none;">
        <b>W A S D</b> - Move | <b>SPACE</b> - Jump | <b>ESC</b> - Leave Game
    </div>

    <script>
        // 1. SCENE SETUP
        const scene = new THREE.Scene();
        scene.background = new THREE.Color(0x87CEEB); // Sky
        scene.fog = new THREE.Fog(0x87CEEB, 10, 100); // Add fog for distance
        
        const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
        
        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        document.body.appendChild(renderer.domElement);

        // Lights
        scene.add(new THREE.AmbientLight(0xffffff, 0.6));
        const dirLight = new THREE.DirectionalLight(0xffffff, 0.8);
        dirLight.position.set(20, 50, 20);
        scene.add(dirLight);

        // 2. BUILD THE WORLD
        // Baseplate
        const floorGeo = new THREE.PlaneGeometry(200, 200);
        const floorMat = new THREE.MeshStandardMaterial({ color: 0x3cb371 }); // Medium Sea Green
        const floor = new THREE.Mesh(floorGeo, floorMat);
        floor.rotation.x = -Math.PI / 2;
        scene.add(floor);

        // A mini "Tower of Hell" Obby structure
        const platformGeo = new THREE.BoxGeometry(4, 1, 4);
        const colors = [0xff0000, 0xffa500, 0xffff00, 0x008000, 0x0000ff];
        
        for (let i = 0; i < 5; i++) {
            const platMat = new THREE.MeshStandardMaterial({ color: colors[i] });
            const platform = new THREE.Mesh(platformGeo, platMat);
            // Stagger them like stairs going up
            platform.position.set(i * 5, (i * 3) + 1, -15);
            scene.add(platform);
        }

        // 3. FIRST-PERSON CONTROLS (Pointer Lock)
        const controls = new THREE.PointerLockControls(camera, document.body);
        const joinMenu = document.getElementById('join-menu');
        const crosshair = document.getElementById('crosshair');
        const helpText = document.getElementById('controls-help');

        // When "Play Game" is clicked, lock the mouse and hide the menu
        document.getElementById('joinBtn').addEventListener('click', () => {
            controls.lock();
        });

        controls.addEventListener('lock', () => {
            joinMenu.style.display = 'none';
            crosshair.style.display = 'block';
            helpText.style.display = 'block';
        });

        // If the user hits ESC, show the menu again
        controls.addEventListener('unlock', () => {
            joinMenu.style.display = 'flex';
            crosshair.style.display = 'none';
            helpText.style.display = 'none';
        });

        // 4. MOVEMENT & PHYSICS LOGIC
        const velocity = new THREE.Vector3();
        const direction = new THREE.Vector3();
        let moveForward = false;
        let moveBackward = false;
        let moveLeft = false;
        let moveRight = false;
        let canJump = false;

        const onKeyDown = (event) => {
            switch (event.code) {
                case 'KeyW': moveForward = true; break;
                case 'KeyA': moveLeft = true; break;
                case 'KeyS': moveBackward = true; break;
                case 'KeyD': moveRight = true; break;
                case 'Space': 
                    if (canJump === true) velocity.y += 25; // Jump power
                    canJump = false;
                    break;
            }
        };

        const onKeyUp = (event) => {
            switch (event.code) {
                case 'KeyW': moveForward = false; break;
                case 'KeyA': moveLeft = false; break;
                case 'KeyS': moveBackward = false; break;
                case 'KeyD': moveRight = false; break;
            }
        };

        document.addEventListener('keydown', onKeyDown);
        document.addEventListener('keyup', onKeyUp);

        camera.position.y = 2; // Starting player height

        // 5. GAME LOOP
        let prevTime = performance.now();

        function animate() {
            requestAnimationFrame(animate);

            const time = performance.now();

            // Only calculate movement if the player is actually in-game (mouse is locked)
            if (controls.isLocked === true) {
                const delta = (time - prevTime) / 1000;

                // Apply Friction/Drag
                velocity.x -= velocity.x * 10.0 * delta;
                velocity.z -= velocity.z * 10.0 * delta;
                
                // Apply Gravity (pulling Y down)
                velocity.y -= 9.8 * 10.0 * delta;

                // Determine direction based on keys pressed
                direction.z = Number(moveForward) - Number(moveBackward);
                direction.x = Number(moveRight) - Number(moveLeft);
                direction.normalize(); // Ensure consistent speed in all directions

                // Apply Speed
                const speed = 40.0;
                if (moveForward || moveBackward) velocity.z -= direction.z * speed * delta;
                if (moveLeft || moveRight) velocity.x -= direction.x * speed * delta;

                // Move the camera based on velocity
                controls.moveRight(-velocity.x * delta);
                controls.moveForward(-velocity.z * delta);
                
                // Apply vertical velocity (jumping/falling)
                controls.getObject().position.y += (velocity.y * delta);

                // Basic Floor Collision Check
                if (controls.getObject().position.y < 2) {
                    velocity.y = 0;
                    controls.getObject().position.y = 2; // Snap back to ground level
                    canJump = true; // Reset jump when hitting the ground
                }
            }

            prevTime = time;
            renderer.render(scene, camera);
        }

        // Handle window resizing
        window.addEventListener('resize', () => {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        });

        animate();
    </script>
</body>
</html>
