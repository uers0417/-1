<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>相互垂直同频率简谐振动合成 - 3D李萨如轨迹 | Three.js</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            font-family: 'Microsoft YaHei', 'Segoe UI', 'PingFang SC', Roboto, 'Helvetica Neue', sans-serif;
        }
        #info {
            position: absolute;
            top: 20px;
            left: 20px;
            background: rgba(0,0,0,0.7);
            backdrop-filter: blur(8px);
            color: white;
            padding: 12px 20px;
            border-radius: 12px;
            pointer-events: none;
            z-index: 10;
            border-left: 4px solid #ff3366;
            box-shadow: 0 4px 15px rgba(0,0,0,0.3);
            font-size: 14px;
        }
        #info h1 {
            margin: 0 0 5px;
            font-size: 1.2rem;
            letter-spacing: 1px;
        }
        #info p {
            margin: 0;
            font-size: 0.8rem;
            opacity: 0.9;
        }
        .controls-panel {
            position: absolute;
            bottom: 20px;
            right: 20px;
            background: rgba(30,30,40,0.85);
            backdrop-filter: blur(10px);
            border-radius: 16px;
            padding: 15px 20px;
            color: white;
            z-index: 20;
            font-size: 14px;
            min-width: 220px;
            border: 1px solid rgba(255,255,255,0.2);
            box-shadow: 0 8px 20px rgba(0,0,0,0.3);
            pointer-events: auto;
            font-weight: 500;
        }
        .controls-panel h3 {
            margin: 0 0 10px 0;
            font-size: 1rem;
            text-align: center;
            color: #ffaa66;
        }
        .control-group {
            margin-bottom: 12px;
        }
        .control-group label {
            display: block;
            font-size: 0.8rem;
            margin-bottom: 5px;
            opacity: 0.9;
        }
        input {
            width: 100%;
            cursor: pointer;
            background: rgba(20,20,30,0.9);
            border: none;
            height: 4px;
            border-radius: 5px;
            accent-color: #ff884d;
        }
        .value-display {
            display: inline-block;
            background: #000000aa;
            padding: 2px 6px;
            border-radius: 20px;
            font-size: 0.7rem;
            margin-left: 8px;
        }
        .param-row {
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        button {
            background: #ff884d;
            border: none;
            color: #1e1e2a;
            padding: 6px 14px;
            border-radius: 40px;
            cursor: pointer;
            font-weight: bold;
            margin-top: 8px;
            width: 100%;
            transition: 0.2s;
            font-size: 0.8rem;
            letter-spacing: 1px;
        }
        button:hover {
            background: #ffaa66;
            transform: scale(1.02);
            box-shadow: 0 2px 8px rgba(0,0,0,0.3);
        }
        .url-area {
            margin-top: 12px;
            font-size: 0.7rem;
            text-align: center;
            border-top: 1px solid rgba(255,255,255,0.2);
            padding-top: 8px;
            word-break: break-all;
        }
        a {
            color: #ffaa66;
            text-decoration: none;
        }
        .note {
            font-size: 0.7rem;
            text-align: center;
            margin-top: 8px;
            opacity: 0.7;
        }
        @media (max-width: 600px) {
            .controls-panel { transform: scale(0.9); right: -10px; bottom: 0; }
            #info { font-size: 10px; top: 10px; left: 10px; }
        }
    </style>
</head>
<body>
    <div id="info">
        <h1>⚡ 相互垂直 · 同频率简谐振动合成</h1>
        <p>X = A₁·sin(ωt) &nbsp;&nbsp; Y = A₂·sin(ωt + φ) &nbsp;&nbsp; | 轨迹: 椭圆 / 直线 (李萨如)</p>
    </div>
    <div class="controls-panel">
        <h3>🎛️ 振动参数调节</h3>
        <div class="control-group">
            <label>振幅 A₁ (X方向) <span id="axVal" class="value-display">2.00</span></label>
            <input type="range" id="ampX" min="0.5" max="3.0" step="0.01" value="2.0">
        </div>
        <div class="control-group">
            <label>振幅 A₂ (Y方向) <span id="ayVal" class="value-display">2.00</span></label>
            <input type="range" id="ampY" min="0.5" max="3.0" step="0.01" value="2.0">
        </div>
        <div class="control-group">
            <label>相位差 φ (degree) <span id="phiVal" class="value-display">90°</span></label>
            <input type="range" id="phase" min="0" max="360" step="1" value="90">
        </div>
        <div class="control-group">
            <label>频率系数 ω <span id="freqVal" class="value-display">1.00</span></label>
            <input type="range" id="freqScale" min="0.5" max="2.0" step="0.01" value="1.0">
        </div>
        <button id="downloadBtn">💾 下载本模型 (HTML)</button>
        <div class="url-area">
            🌐 页面地址: <span id="pageUrl">加载中...</span><br>
            <span class="note">※ 三维合成 | 拖动视角观察轨迹 | 参数实时更新轨迹与运动</span>
        </div>
    </div>

    <!-- 引入Three.js核心库及扩展 -->
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
        import { CSS2DRenderer, CSS2DObject } from 'three/addons/renderers/CSS2DRenderer.js';

        // --- 初始化场景、相机、渲染器 ---
        const scene = new THREE.Scene();
        scene.background = new THREE.Color(0x050b1a);
        scene.fog = new THREE.FogExp2(0x050b1a, 0.008); // 轻微雾效增强景深

        // 透视相机 (视角,宽高比,近平面,远平面)
        const camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000);
        camera.position.set(5, 4, 6);
        camera.lookAt(0, 0, 0);
        
        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.shadowMap.enabled = true; // 开启阴影映射增强质感
        renderer.setPixelRatio(window.devicePixelRatio);
        document.body.appendChild(renderer.domElement);
        
        // CSS2渲染文字标签
        const labelRenderer = new CSS2DRenderer();
        labelRenderer.setSize(window.innerWidth, window.innerHeight);
        labelRenderer.domElement.style.position = 'absolute';
        labelRenderer.domElement.style.top = '0px';
        labelRenderer.domElement.style.left = '0px';
        labelRenderer.domElement.style.pointerEvents = 'none';
        document.body.appendChild(labelRenderer.domElement);
        
        // --- 轨道控制 (允许用户交互) ---
        const controls = new OrbitControls(camera, renderer.domElement);
        controls.enableDamping = true;      // 惯性
        controls.dampingFactor = 0.05;
        controls.autoRotate = false;
        controls.enableZoom = true;
        controls.zoomSpeed = 1.2;
        controls.rotateSpeed = 1.0;
        controls.target.set(0, 0, 0);
        
        // --- 辅助元素: 网格地板 + 坐标轴 (强化三维感) ---
        const gridHelper = new THREE.GridHelper(12, 20, 0x88aaff, 0x335588);
        gridHelper.position.y = -0.2;
        gridHelper.material.transparent = true;
        gridHelper.material.opacity = 0.4;
        scene.add(gridHelper);
        
        // 自定义坐标轴: 红X, 绿Y, 蓝Z (带箭头更加直观)
        const axesHelper = new THREE.AxesHelper(4);
        scene.add(axesHelper);
        
        // 添加半透明辅助平面 (在z=0处显示一个淡淡平面指示振动平面)
        const planeMat = new THREE.MeshStandardMaterial({ color: 0x2266aa, side: THREE.DoubleSide, transparent: true, opacity: 0.08 });
        const guidePlane = new THREE.Mesh(new THREE.PlaneGeometry(6.5, 6.5), planeMat);
        guidePlane.rotation.x = -Math.PI / 2;
        guidePlane.position.z = 0;
        scene.add(guidePlane);
        
        // 添加环境光+点光源+定向光，让球体材质更好看
        const ambientLight = new THREE.AmbientLight(0x404060);
        scene.add(ambientLight);
        const mainLight = new THREE.DirectionalLight(0xffffff, 1);
        mainLight.position.set(2, 5, 3);
        mainLight.castShadow = true;
        mainLight.receiveShadow = false;
        scene.add(mainLight);
        const fillLight = new THREE.PointLight(0x4466cc, 0.5);
        fillLight.position.set(-2, 1, 2);
        scene.add(fillLight);
        const backLight = new THREE.PointLight(0xffaa66, 0.3);
        backLight.position.set(1, 2, -3);
        scene.add(backLight);
        
        // 装饰: 围绕的粒子星点 (视觉增强)
        const starGeometry = new THREE.BufferGeometry();
        const starCount = 800;
        const starPositions = new Float32Array(starCount * 3);
        for (let i = 0; i < starCount; i++) {
            starPositions[i*3] = (Math.random() - 0.5) * 200;
            starPositions[i*3+1] = (Math.random() - 0.5) * 100;
            starPositions[i*3+2] = (Math.random() - 0.5) * 100 - 50;
        }
        starGeometry.setAttribute('position', new THREE.BufferAttribute(starPositions, 3));
        const starMaterial = new THREE.PointsMaterial({ color: 0xaaccff, size: 0.08, transparent: true, opacity: 0.5 });
        const stars = new THREE.Points(starGeometry, starMaterial);
        scene.add(stars);
        
        // --- 核心模型: 运动质点(小球) + 轨迹线 ---
        // 小球材质 (金属质感)
        const sphereMat = new THREE.MeshStandardMaterial({ color: 0xff6666, emissive: 0x441122, roughness: 0.3, metalness: 0.7 });
        const sphere = new THREE.Mesh(new THREE.SphereGeometry(0.18, 32, 32), sphereMat);
        sphere.castShadow = true;
        scene.add(sphere);
        
        // 添加小球的光晕效果: 一个小光环
        const glowMat = new THREE.MeshBasicMaterial({ color: 0xff8866, transparent: true, opacity: 0.4 });
        const glowRing = new THREE.Mesh(new THREE.SphereGeometry(0.26, 16, 16), glowMat);
        sphere.add(glowRing);
        
        // 轨迹线对象 (动态更新)
        let currentTrajectory = null;
        
        // 投影辅助线: 从小球分别到X轴和Y轴的连线 (增强垂直振动投影理解)
        const xAxisLineMaterial = new THREE.LineBasicMaterial({ color: 0xff3366 });
        const yAxisLineMaterial = new THREE.LineBasicMaterial({ color: 0x33ff66 });
        let xProjLine = new THREE.Line(new THREE.BufferGeometry(), xAxisLineMaterial);
        let yProjLine = new THREE.Line(new THREE.BufferGeometry(), yAxisLineMaterial);
        scene.add(xProjLine);
        scene.add(yProjLine);
        
        // 在X轴和Y轴分别添加两个小辅助球体表示投影点 (更生动)
        const dotXMat = new THREE.MeshStandardMaterial({ color: 0xff8888, emissive: 0x331100 });
        const dotYMat = new THREE.MeshStandardMaterial({ color: 0x88ff88, emissive: 0x113311 });
        const projXSphere = new THREE.Mesh(new THREE.SphereGeometry(0.09, 16, 16), dotXMat);
        const projYSphere = new THREE.Mesh(new THREE.SphereGeometry(0.09, 16, 16), dotYMat);
        scene.add(projXSphere);
        scene.add(projYSphere);
        
        // 添加文字标签 (CSS2D)
        function makeLabel(text, color, position) {
            const div = document.createElement('div');
            div.textContent = text;
            div.style.color = color;
            div.style.fontSize = '18px';
            div.style.fontWeight = 'bold';
            div.style.textShadow = '1px 1px 0px black';
            div.style.background = 'rgba(0,0,0,0.5)';
            div.style.padding = '2px 6px';
            div.style.borderRadius = '20px';
            div.style.borderLeft = `3px solid ${color}`;
            const label = new CSS2DObject(div);
            label.position.copy(position);
            scene.add(label);
            return label;
        }
        makeLabel('X 方向', '#ff6666', new THREE.Vector3(4.2, -0.2, 0));
        makeLabel('Y 方向', '#66ff66', new THREE.Vector3(0, 4.2, 0));
        makeLabel('合成轨迹平面 (Z=0)', '#aaaaff', new THREE.Vector3(3, 2.5, 0.8));
        
        // 在X轴和Y轴末端添加装饰小球标记方向
        const arrowX = new THREE.Mesh(new THREE.SphereGeometry(0.08, 8, 8), new THREE.MeshStandardMaterial({ color: 0xff6666 }));
        arrowX.position.set(4.2, 0, 0);
        const arrowY = new THREE.Mesh(new THREE.SphereGeometry(0.08, 8, 8), new THREE.MeshStandardMaterial({ color: 0x66ff66 }));
        arrowY.position.set(0, 4.2, 0);
        scene.add(arrowX);
        scene.add(arrowY);
        
        // --- 参数定义 ---
        // 频率基值 ω = 2π * f, 为了动画直观，使用时间累加角度 = omega * time
        let params = {
            Ax: 2.0,
            Ay: 2.0,
            phiDeg: 90,    // 相位差(度)
            omegaScale: 1.0,
        };
        
        // DOM 元素绑定
        const axSlider = document.getElementById('ampX');
        const aySlider = document.getElementById('ampY');
        const phaseSlider = document.getElementById('phase');
        const freqSlider = document.getElementById('freqScale');
        const axVal = document.getElementById('axVal');
        const ayVal = document.getElementById('ayVal');
        const phiVal = document.getElementById('phiVal');
        const freqVal = document.getElementById('freqVal');
        
        function updateParamsFromUI() {
            params.Ax = parseFloat(axSlider.value);
            params.Ay = parseFloat(aySlider.value);
            params.phiDeg = parseFloat(phaseSlider.value);
            params.omegaScale = parseFloat(freqSlider.value);
            axVal.innerText = params.Ax.toFixed(2);
            ayVal.innerText = params.Ay.toFixed(2);
            phiVal.innerText = params.phiDeg + '°';
            freqVal.innerText = params.omegaScale.toFixed(2);
            // 参数改变后，重新生成轨迹线
            regenerateTrajectory();
        }
        
        // 根据当前参数生成轨迹曲线 (Line)
        function regenerateTrajectory() {
            if (currentTrajectory) scene.remove(currentTrajectory);
            const Ax = params.Ax;
            const Ay = params.Ay;
            const phiRad = params.phiDeg * Math.PI / 180;
            // 采样点数
            const segments = 200;
            const points = [];
            // 一个完整周期 0 到 2PI
            for (let i = 0; i <= segments; i++) {
                const t = (i / segments) * Math.PI * 2;
                const x = Ax * Math.sin(t);
                const y = Ay * Math.sin(t + phiRad);
                points.push(new THREE.Vector3(x, y, 0));
            }
            const geometry = new THREE.BufferGeometry().setFromPoints(points);
            const material = new THREE.LineBasicMaterial({ color: 0x3ac0ff, linewidth: 2, linecap: 'round', linejoin: 'round' });
            // 由于WebGL渲染器linewidth限制，但效果依然明显，使用发光质感
            const trailLine = new THREE.Line(geometry, material);
            scene.add(trailLine);
            currentTrajectory = trailLine;
            
            // 还可以添加轨迹光晕点 (额外装饰: 一些沿轨迹的粒子)
            // 简单增加几个光点装饰，可选
        }
        
        // 主动画循环需要的变量
        let clock = new THREE.Clock();
        
        // 更新质点位置和投影线
        function updateMotion(deltaTime) {
            // 累积时间角度，保证频率可调且连续
            // 使用全局累计时间而不是delta累加，避免暂停影响 (使用从开始计时)
            const now = performance.now() / 1000; // seconds
            const angularSpeed = 2.0 * Math.PI * params.omegaScale; // ω = 2πf, 这里基准频率为1Hz
            const angle = angularSpeed * now;
            const phiRad = params.phiDeg * Math.PI / 180;
            const x = params.Ax * Math.sin(angle);
            const y = params.Ay * Math.sin(angle + phiRad);
            const pos = new THREE.Vector3(x, y, 0);
            sphere.position.copy(pos);
            
            // 更新X轴投影连线: 从 (x,0,0) 到 (x,y,0)
            const xAxisPoint = new THREE.Vector3(x, 0, 0);
            const yAxisPoint = new THREE.Vector3(0, y, 0);
            // 更新投影线的几何体
            const xLinePoints = [xAxisPoint, pos];
            const yLinePoints = [yAxisPoint, pos];
            const xGeom = new THREE.BufferGeometry().setFromPoints(xLinePoints);
            const yGeom = new THREE.BufferGeometry().setFromPoints(yLinePoints);
            xProjLine.geometry.dispose(); // 避免内存累积
            yProjLine.geometry.dispose();
            xProjLine.geometry = xGeom;
            yProjLine.geometry = yGeom;
            
            // 更新投影小球位置
            projXSphere.position.copy(xAxisPoint);
            projYSphere.position.copy(yAxisPoint);
            
            // 动态改变小球周围光晕强度 (呼吸效果)
            const intensity = 0.5 + Math.sin(angle * 8) * 0.2;
            glowRing.material.opacity = 0.3 + Math.sin(angle * 6) * 0.2;
        }
        
        // 监听UI变化
        axSlider.addEventListener('input', () => { updateParamsFromUI(); });
        aySlider.addEventListener('input', () => { updateParamsFromUI(); });
        phaseSlider.addEventListener('input', () => { updateParamsFromUI(); });
        freqSlider.addEventListener('input', () => { updateParamsFromUI(); });
        
        // 显示页面URL (获取当前地址)
        const urlSpan = document.getElementById('pageUrl');
        if (urlSpan) {
            urlSpan.innerText = window.location.href;
            // 增加点击复制功能 (友好)
            urlSpan.style.cursor = 'pointer';
            urlSpan.title = '点击复制链接';
            urlSpan.addEventListener('click', () => {
                navigator.clipboard.writeText(window.location.href).then(() => {
                    alert('网址已复制到剪贴板！');
                }).catch(() => { alert('手动复制吧: ' + window.location.href); });
            });
        }
        
        // 下载HTML文件按钮功能 (序列化当前页面完整源码，实现保存本模型)
        const downloadBtn = document.getElementById('downloadBtn');
        downloadBtn.addEventListener('click', () => {
            // 获取当前完整HTML内容，包括最新的DOM状态（滑块值等）
            const doctype = '<!DOCTYPE html>\n';
            const html = document.documentElement.outerHTML;
            // 确保正确编码和下载
            const blob = new Blob([doctype + html], { type: 'text/html' });
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = 'lissajous_3d_vibration.html';
            document.body.appendChild(a);
            a.click();
            document.body.removeChild(a);
            URL.revokeObjectURL(url);
        });
        
        // 初始化轨迹
        updateParamsFromUI();
        
        // 为了让相机初始视角更合适，稍微调整一下控制目标
        controls.target.set(0, 0, 0);
        controls.update();
        
        // 简单的动画辅助：添加浮动粒子环绕轨迹 (装饰)
        const particleCount = 400;
        const particleGeo = new THREE.BufferGeometry();
        const particlePositions = new Float32Array(particleCount * 3);
        for (let i = 0; i < particleCount; i++) {
            particlePositions[i*3] = (Math.random() - 0.5) * 8;
            particlePositions[i*3+1] = (Math.random() - 0.5) * 8;
            particlePositions[i*3+2] = (Math.random() - 0.5) * 3 - 1;
        }
        particleGeo.setAttribute('position', new THREE.BufferAttribute(particlePositions, 3));
        const particleMat = new THREE.PointsMaterial({ color: 0x77aaff, size: 0.03, transparent: true });
        const particlesField = new THREE.Points(particleGeo, particleMat);
        scene.add(particlesField);
        
        // 额外增加两个半透明星环提升视觉效果
        const ringGeometry = new THREE.TorusGeometry(2.5, 0.03, 64, 200);
        const ringMat = new THREE.MeshStandardMaterial({ color: 0x44aaff, emissive: 0x114466, transparent: true, opacity: 0.25 });
        const ringX = new THREE.Mesh(ringGeometry, ringMat);
        ringX.rotation.x = Math.PI / 2;
        ringX.position.z = -0.2;
        scene.add(ringX);
        
        const ringY = new THREE.Mesh(ringGeometry, ringMat);
        ringY.rotation.z = Math.PI / 2;
        ringY.position.z = -0.2;
        scene.add(ringY);
        
        // 柔和动画更新
        function animate() {
            const delta = clock.getDelta();
            // 限制delta避免突变
            const safeDelta = Math.min(delta, 0.033);
            updateMotion(safeDelta);
            
            // 让粒子场缓慢旋转，增加动感
            particlesField.rotation.y += 0.002;
            particlesField.rotation.x += 0.001;
            stars.rotation.y += 0.0005;
            
            // 更新轨道控制
            controls.update();
            
            // 渲染
            renderer.render(scene, camera);
            labelRenderer.render(scene, camera);
            
            requestAnimationFrame(animate);
        }
        
        // 启动动画
        animate();
        
        // 窗口适配
        window.addEventListener('resize', onWindowResize, false);
        function onWindowResize() {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
            labelRenderer.setSize(window.innerWidth, window.innerHeight);
        }
        
        // 小彩蛋: 控制台输出
        console.log('3D 相互垂直简谐振动合成模型已启动 | 相位差可调，轨迹实时变化');
        
        // 添加提示指示相位差对轨迹的影响: 显示当前椭圆/线性状态的一个小提示 (在控制面板旁)
        const statusDiv = document.createElement('div');
        statusDiv.style.position = 'absolute';
        statusDiv.style.bottom = '90px';
        statusDiv.style.left = '20px';
        statusDiv.style.backgroundColor = 'rgba(0,0,0,0.5)';
        statusDiv.style.color = '#ccc';
        statusDiv.style.padding = '4px 12px';
        statusDiv.style.borderRadius = '20px';
        statusDiv.style.fontSize = '12px';
        statusDiv.style.fontFamily = 'monospace';
        statusDiv.style.pointerEvents = 'none';
        statusDiv.innerText = '✨ 同频率合成 | 轨迹: 椭圆/直线 随相位差变化';
        document.body.appendChild(statusDiv);
        
        // 动态修改状态行文字与相位差的联动 (可选简单)
        function updateStatusHint() {
            const phi = params.phiDeg;
            let shape = '';
            if (Math.abs(phi) % 180 === 0) shape = '直线段';
            else if (Math.abs(phi - 90) < 5 || Math.abs(phi - 270) < 5) shape = '正椭圆 / 圆';
            else shape = '倾斜椭圆';
            statusDiv.innerText = `✨ 相位差 ${phi}°  |  合成轨迹: ${shape}  (同频率简谐振动)`;
        }
        phaseSlider.addEventListener('input', () => { setTimeout(updateStatusHint, 10); });
        setInterval(updateStatusHint, 200);
        updateStatusHint();
    </script>
</body>
</html>
