<!DOCTYPE html>
<html lang="cs">
<head>
  <meta charset="UTF-8">
  <title>Airplane Game</title>
  <style>
    body { margin: 0; overflow: hidden; background: #87CEEB; }
    canvas { display: block; }
    #hud {
      position: absolute;
      top: 10px; left: 10px;
      padding: 10px 15px;
      background: rgba(0,0,0,0.4);
      color: #fff;
      font-family: monospace;
      font-size: 14px;
      border-radius: 8px;
    }
  </style>
</head>
<body>
  <div id="hud">Rychlost: 0<br>Výška: 0</div>
  <script src="https://cdn.jsdelivr.net/npm/three@0.158.0/build/three.min.js"></script>
  <script>
    // Scéna a kamera
    const scene = new THREE.Scene();
    const camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
    const renderer = new THREE.WebGLRenderer({antialias: true});
    renderer.setSize(window.innerWidth, window.innerHeight);
    document.body.appendChild(renderer.domElement);

    // Osvětlení
    const light = new THREE.DirectionalLight(0xffffff, 1);
    light.position.set(10,10,10);
    scene.add(light);

    // Letadlo (jednoduchý model)
    const planeGeometry = new THREE.BoxGeometry(2, 0.5, 6);
    const planeMaterial = new THREE.MeshStandardMaterial({color: 0xff0000});
    const airplane = new THREE.Mesh(planeGeometry, planeMaterial);

    const wingGeometry = new THREE.BoxGeometry(8, 0.2, 1);
    const wing = new THREE.Mesh(wingGeometry, planeMaterial);
    wing.position.y = 0; wing.position.z = 0;
    airplane.add(wing);

    scene.add(airplane);

    // Kamera za letadlem
    camera.position.set(0, 2, -10);
    camera.lookAt(airplane.position);

    // Ovládání
    let speed = 0.1;
    let acceleration = 0;
    const keys = {};
    window.addEventListener("keydown", e => keys[e.key.toLowerCase()] = true);
    window.addEventListener("keyup", e => keys[e.key.toLowerCase()] = false);

    // HUD
    const hud = document.getElementById("hud");

    function animate() {
      requestAnimationFrame(animate);

      // Ovládání rychlosti
      if (keys["w"]) acceleration += 0.001;
      if (keys["s"]) acceleration -= 0.001;
      acceleration = Math.max(Math.min(acceleration, 0.2), -0.05);
      speed += acceleration;
      if (speed < 0) speed = 0;

      // Ovládání směru
      if (keys["a"]) airplane.rotation.y += 0.02;
      if (keys["d"]) airplane.rotation.y -= 0.02;
      if (keys["arrowup"]) airplane.rotation.x += 0.01;
      if (keys["arrowdown"]) airplane.rotation.x -= 0.01;

      // Pohyb vpřed podle směru
      const forward = new THREE.Vector3(0,0,1);
      forward.applyQuaternion(airplane.quaternion);
      airplane.position.addScaledVector(forward, speed);

      // Kamera sleduje letadlo
      const camOffset = new THREE.Vector3(0, 3, -12).applyQuaternion(airplane.quaternion);
      camera.position.copy(airplane.position.clone().add(camOffset));
      camera.lookAt(airplane.position);

      // HUD update
      hud.innerHTML = `Rychlost: ${(speed*100).toFixed(0)}<br>Výška: ${airplane.position.y.toFixed(1)}`;

      renderer.render(scene, camera);
    }
    animate();

    // Resize
    window.addEventListener("resize", () => {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    });
  </script>
</body>
</html>
