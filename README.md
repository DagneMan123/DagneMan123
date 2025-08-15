## Hi there 👋 I'm Dagne Aydenfu
A Software Enginerr

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>3D Interactive Cube</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            background: #121212;
            overflow: hidden;
            font-family: 'Arial', sans-serif;
            perspective: 1000px;
        }

        .scene {
            width: 300px;
            height: 300px;
            position: relative;
            transform-style: preserve-3d;
            transition: transform 0.5s cubic-bezier(0.17, 0.67, 0.21, 0.99);
        }

        .cube {
            width: 100%;
            height: 100%;
            position: relative;
            transform-style: preserve-3d;
            transform: rotateX(15deg) rotateY(15deg);
        }

        .face {
            position: absolute;
            width: 100%;
            height: 100%;
            border: 2px solid rgba(255, 255, 255, 0.1);
            box-sizing: border-box;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 60px;
            color: white;
            backface-visibility: hidden;
            transition: all 0.3s ease;
            box-shadow: inset 0 0 30px rgba(0, 0, 0, 0.4);
        }

        .front  { transform: translateZ(150px); background: rgba(255, 50, 50, 0.8); }
        .back   { transform: rotateY(180deg) translateZ(150px); background: rgba(50, 255, 50, 0.8); }
        .right  { transform: rotateY(90deg) translateZ(150px); background: rgba(50, 50, 255, 0.8); }
        .left   { transform: rotateY(-90deg) translateZ(150px); background: rgba(255, 255, 50, 0.8); }
        .top    { transform: rotateX(90deg) translateZ(150px); background: rgba(255, 50, 255, 0.8); }
        .bottom { transform: rotateX(-90deg) translateZ(150px); background: rgba(50, 255, 255, 0.8); }

        .controls {
            position: absolute;
            bottom: 20px;
            color: white;
            text-align: center;
            width: 100%;
            opacity: 0.7;
            font-size: 14px;
        }

        .edge {
            position: absolute;
            background: rgba(255, 255, 255, 0.8);
            box-shadow: 0 0 10px 1px rgba(255, 255, 255, 0.8);
        }

        .particle {
            position: absolute;
            background: white;
            border-radius: 50%;
            pointer-events: none;
            opacity: 0;
        }
    </style>
</head>
<body>
    <div class="scene" id="scene">
        <div class="cube">
            <div class="face front">Html</div>
            <div class="face back">Css</div>
            <div class="face right">Js</div>
            <div class="face left">Node js</div>
            <div class="face top">react js</div>
            <div class="face bottom">bootsrap</div>
        </div>
    </div>
    <div class="controls">Drag to rotate • Double click to reset</div>

    <script>
        document.addEventListener('DOMContentLoaded', function() {
            const scene = document.getElementById('scene');
            let isDragging = false;
            let previousMousePosition = { x: 0, y: 0 };
            let rotation = { x: 15, y: 15 };
            let targetRotation = { x: 15, y: 15 };
            let animationId;
            let particles = [];
            let edges = [];

            // Create cube edges
            function createEdges() {
                const edgeLength = 300;
                const positions = [
                    // Vertical edges
                    { x1: -1, y1: -1, z1: -1, x2: -1, y2: 1, z2: -1 },
                    { x1: 1, y1: -1, z1: -1, x2: 1, y2: 1, z2: -1 },
                    { x1: -1, y1: -1, z1: 1, x2: -1, y2: 1, z2: 1 },
                    { x1: 1, y1: -1, z1: 1, x2: 1, y2: 1, z2: 1 },
                    
                    // Horizontal edges
                    { x1: -1, y1: -1, z1: -1, x2: 1, y2: -1, z2: -1 },
                    { x1: -1, y1: 1, z1: -1, x2: 1, y2: 1, z2: -1 },
                    { x1: -1, y1: -1, z1: 1, x2: 1, y2: -1, z2: 1 },
                    { x1: -1, y1: 1, z1: 1, x2: 1, y2: 1, z2: 1 },
                    
                    // Depth edges
                    { x1: -1, y1: -1, z1: -1, x2: -1, y2: -1, z2: 1 },
                    { x1: 1, y1: -1, z1: -1, x2: 1, y2: -1, z2: 1 },
                    { x1: -1, y1: 1, z1: -1, x2: -1, y2: 1, z2: 1 },
                    { x1: 1, y1: 1, z1: -1, x2: 1, y2: 1, z2: 1 }
                ];

                positions.forEach(pos => {
                    const edge = document.createElement('div');
                    edge.className = 'edge';
                    scene.appendChild(edge);
                    edges.push({ element: edge, ...pos });
                });
            }

            // Update edge positions based on current rotation
            function updateEdges() {
                const angleX = rotation.x * Math.PI / 180;
                const angleY = rotation.y * Math.PI / 180;
                const size = 150; // Half of cube size

                edges.forEach(edge => {
                    // Apply rotation to start point
                    const x1 = edge.x1 * size;
                    const y1 = edge.y1 * size;
                    const z1 = edge.z1 * size;
                    
                    // Apply rotation to end point
                    const x2 = edge.x2 * size;
                    const y2 = edge.y2 * size;
                    const z2 = edge.z2 * size;
                    
                    // Rotate around Y axis
                    const cosY = Math.cos(angleY);
                    const sinY = Math.sin(angleY);
                    
                    let rx1 = x1 * cosY - z1 * sinY;
                    let rz1 = x1 * sinY + z1 * cosY;
                    let ry1 = y1;
                    
                    let rx2 = x2 * cosY - z2 * sinY;
                    let rz2 = x2 * sinY + z2 * cosY;
                    let ry2 = y2;
                    
                    // Rotate around X axis
                    const cosX = Math.cos(angleX);
                    const sinX = Math.sin(angleX);
                    
                    const ry1_final = ry1 * cosX - rz1 * sinX;
                    const rz1_final = ry1 * sinX + rz1 * cosX;
                    
                    const ry2_final = ry2 * cosX - rz2 * sinX;
                    const rz2_final = ry2 * sinX + rz2 * cosX;
                    
                    // Calculate length and angle of the edge
                    const dx = rx2 - rx1;
                    const dy = ry2_final - ry1_final;
                    const dz = rz2_final - rz1_final;
                    
                    const length = Math.sqrt(dx*dx + dy*dy + dz*dz);
                    const angle = Math.atan2(dy, Math.sqrt(dx*dx + dz*dz)) * 180 / Math.PI;
                    const rotation = Math.atan2(dx, dz) * 180 / Math.PI;
                    
                    // Position the edge element
                    edge.element.style.width = `${length}px`;
                    edge.element.style.height = '2px';
                    edge.element.style.transform = `
                        translate3d(${rx1 + 150}px, ${ry1_final + 150}px, ${rz1_final}px)
                        rotateY(${rotation}deg)
                        rotateX(${-angle}deg)
                    `;
                    edge.element.style.transformOrigin = '0 0';
                });
            }

            // Create particles at mouse position
            function createParticle(x, y) {
                const particle = document.createElement('div');
                particle.className = 'particle';
                document.body.appendChild(particle);
                
                // Random size and position
                const size = Math.random() * 8 + 4;
                particle.style.width = `${size}px`;
                particle.style.height = `${size}px`;
                particle.style.left = `${x}px`;
                particle.style.top = `${y}px`;
                
                // Random color from cube faces
                const colors = ['#ff3232', '#32ff32', '#3232ff', '#ffff32', '#ff32ff', '#32ffff'];
                particle.style.background = colors[Math.floor(Math.random() * colors.length)];
                
                // Animate particle
                const angle = Math.random() * Math.PI * 2;
                const velocity = Math.random() * 3 + 1;
                const lifetime = Math.random() * 1000 + 500;
                
                particles.push({
                    element: particle,
                    x: x,
                    y: y,
                    vx: Math.cos(angle) * velocity,
                    vy: Math.sin(angle) * velocity,
                    birth: Date.now(),
                    lifetime: lifetime,
                    size: size
                });
            }

            // Update particle positions and opacity
            function updateParticles() {
                const now = Date.now();
                particles = particles.filter(p => {
                    const age = now - p.birth;
                    if (age > p.lifetime) {
                        p.element.remove();
                        return false;
                    }
                    
                    p.x += p.vx;
                    p.y += p.vy;
                    p.vy += 0.05; // gravity
                    
                    const progress = age / p.lifetime;
                    p.element.style.left = `${p.x}px`;
                    p.element.style.top = `${p.y}px`;
                    p.element.style.opacity = `${1 - progress}`;
                    p.element.style.transform = `scale(${1 + progress * 0.5})`;
                    
                    return true;
                });
                
                requestAnimationFrame(updateParticles);
            }

            // Mouse interaction handlers
            scene.addEventListener('mousedown', (e) => {
                isDragging = true;
                previousMousePosition = { x: e.clientX, y: e.clientY };
                cancelAnimationFrame(animationId);
            });

            document.addEventListener('mousemove', (e) => {
                if (isDragging) {
                    const deltaX = e.clientX - previousMousePosition.x;
                    const deltaY = e.clientY - previousMousePosition.y;
                    
                    targetRotation.y += deltaX * 0.5;
                    targetRotation.x += deltaY * 0.5;
                    
                    previousMousePosition = { x: e.clientX, y: e.clientY };
                    
                    // Create particles on drag
                    if (Math.random() > 0.7) {
                        createParticle(e.clientX, e.clientY);
                    }
                }
            });

            document.addEventListener('mouseup', () => {
                isDragging = false;
                animateRotation();
            });

            // Double click to reset
            scene.addEventListener('dblclick', () => {
                targetRotation = { x: 15, y: 15 };
            });

            // Touch interaction handlers
            scene.addEventListener('touchstart', (e) => {
                isDragging = true;
                previousMousePosition = { x: e.touches[0].clientX, y: e.touches[0].clientY };
                e.preventDefault();
                cancelAnimationFrame(animationId);
            });

            document.addEventListener('touchmove', (e) => {
                if (isDragging) {
                    const deltaX = e.touches[0].clientX - previousMousePosition.x;
                    const deltaY = e.touches[0].clientY - previousMousePosition.y;
                    
                    targetRotation.y += deltaX * 0.5;
                    targetRotation.x += deltaY * 0.5;
                    
                    previousMousePosition = { 
                        x: e.touches[0].clientX, 
                        y: e.touches[0].clientY 
                    };
                    e.preventDefault();
                }
            });

            document.addEventListener('touchend', () => {
                isDragging = false;
                animateRotation();
            });

            // Smooth rotation animation
            function animateRotation() {
                if (!isDragging) {
                    // Add some inertia
                    rotation.x += (targetRotation.x - rotation.x) * 0.1;
                    rotation.y += (targetRotation.y - rotation.y) * 0.1;
                    
                    // Apply rotation with easing
                    scene.style.transform = `
                        rotateX(${rotation.x}deg)
                        rotateY(${rotation.y}deg)
                    `;
                    
                    // Update edge positions
                    updateEdges();
                    
                    // Add slight auto-rotation when idle
                    const dx = targetRotation.x - rotation.x;
                    const dy = targetRotation.y - rotation.y;
                    const distance = Math.sqrt(dx*dx + dy*dy);
                    
                    if (distance < 2) {
                        targetRotation.y += 0.2;
                    }
                    
                    animationId = requestAnimationFrame(animateRotation);
                }
            }

            // Initialize
            createEdges();
            updateEdges();
            animateRotation();
            updateParticles();

            // Intro animation
            setTimeout(() => {
                targetRotation = { x: 25, y: 45 };
            }, 100);
        });
    </script>
</body>
</html>


## 🌐 Socials:
[![Facebook](https://img.shields.io/badge/Facebook-%231877F2.svg?logo=Facebook&logoColor=white)](https://facebook.com/DagneMan123) [![Instagram](https://img.shields.io/badge/Instagram-%23E4405F.svg?logo=Instagram&logoColor=white)](https://instagram.com/DagneMan123) [![LinkedIn](https://img.shields.io/badge/LinkedIn-%230077B5.svg?logo=linkedin&logoColor=white)](https://linkedin.com/in/DagneMan123) [![X](https://img.shields.io/badge/X-black.svg?logo=X&logoColor=white)](https://x.com/DagneMan123) [![YouTube](https://img.shields.io/badge/YouTube-%23FF0000.svg?logo=YouTube&logoColor=white)](https://youtube.com/@DagneMan123) [![email](https://img.shields.io/badge/Email-D14836?logo=gmail&logoColor=white)](mailto:aydenfudagne@gmail.com) 

# 💻 Tech Stack:
![C](https://img.shields.io/badge/c-%2300599C.svg?style=for-the-badge&logo=c&logoColor=white) ![C#](https://img.shields.io/badge/c%23-%23239120.svg?style=for-the-badge&logo=csharp&logoColor=white) ![C++](https://img.shields.io/badge/c++-%2300599C.svg?style=for-the-badge&logo=c%2B%2B&logoColor=white) ![Apache Groovy](https://img.shields.io/badge/Apache%20Groovy-4298B8.svg?style=for-the-badge&logo=Apache+Groovy&logoColor=white) ![CSS3](https://img.shields.io/badge/css3-%231572B6.svg?style=for-the-badge&logo=css3&logoColor=white) ![HTML5](https://img.shields.io/badge/html5-%23E34F26.svg?style=for-the-badge&logo=html5&logoColor=white) ![Java](https://img.shields.io/badge/java-%23ED8B00.svg?style=for-the-badge&logo=openjdk&logoColor=white) ![JavaScript](https://img.shields.io/badge/javascript-%23323330.svg?style=for-the-badge&logo=javascript&logoColor=%23F7DF1E) ![Kotlin](https://img.shields.io/badge/kotlin-%237F52FF.svg?style=for-the-badge&logo=kotlin&logoColor=white) ![Python](https://img.shields.io/badge/python-3670A0?style=for-the-badge&logo=python&logoColor=ffdd54) ![PHP](https://img.shields.io/badge/php-%23777BB4.svg?style=for-the-badge&logo=php&logoColor=white) ![Alibaba Cloud](https://img.shields.io/badge/AlibabaCloud-%23FF6701.svg?style=for-the-badge&logo=alibabacloud&logoColor=white) ![AWS](https://img.shields.io/badge/AWS-%23FF9900.svg?style=for-the-badge&logo=amazon-aws&logoColor=white) ![Azure](https://img.shields.io/badge/azure-%230072C6.svg?style=for-the-badge&logo=microsoftazure&logoColor=white) ![Datadog](https://img.shields.io/badge/datadog-%23632CA6.svg?style=for-the-badge&logo=datadog&logoColor=white) ![Apache Hadoop](https://img.shields.io/badge/Apache%20Hadoop-66CCFF?style=for-the-badge&logo=apachehadoop&logoColor=black) ![Flutter](https://img.shields.io/badge/Flutter-%2302569B.svg?style=for-the-badge&logo=Flutter&logoColor=white) ![Grav](https://img.shields.io/badge/grav-%23FFFFFF.svg?style=for-the-badge&logo=grav&logoColor=221E1F) ![Ember](https://img.shields.io/badge/ember-1C1E24?style=for-the-badge&logo=ember.js&logoColor=#D04A37) ![Esbuild](https://img.shields.io/badge/esbuild-%23FFCF00.svg?style=for-the-badge&logo=esbuild&logoColor=black) ![Next JS](https://img.shields.io/badge/Next-black?style=for-the-badge&logo=next.js&logoColor=white) ![NodeJS](https://img.shields.io/badge/node.js-6DA55F?style=for-the-badge&logo=node.js&logoColor=white) ![React](https://img.shields.io/badge/react-%2320232a.svg?style=for-the-badge&logo=react&logoColor=%2361DAFB) ![Express.js](https://img.shields.io/badge/express.js-%23404d59.svg?style=for-the-badge&logo=express&logoColor=%2361DAFB) ![Deno JS](https://img.shields.io/badge/deno%20js-000000?style=for-the-badge&logo=deno&logoColor=white) ![DaisyUI](https://img.shields.io/badge/daisyui-5A0EF8?style=for-the-badge&logo=daisyui&logoColor=white) ![Blazor](https://img.shields.io/badge/blazor-%235C2D91.svg?style=for-the-badge&logo=blazor&logoColor=white) ![Apache](https://img.shields.io/badge/apache-%23D42029.svg?style=for-the-badge&logo=apache&logoColor=white) ![Jenkins](https://img.shields.io/badge/jenkins-%232C5263.svg?style=for-the-badge&logo=jenkins&logoColor=white) ![Nginx](https://img.shields.io/badge/nginx-%23009639.svg?style=for-the-badge&logo=nginx&logoColor=white) ![Apache Maven](https://img.shields.io/badge/Apache%20Maven-C71A36?style=for-the-badge&logo=Apache%20Maven&logoColor=white) ![Apache Tomcat](https://img.shields.io/badge/apache%20tomcat-%23F8DC75.svg?style=for-the-badge&logo=apache-tomcat&logoColor=black) ![AmazonDynamoDB](https://img.shields.io/badge/Amazon%20DynamoDB-4053D6?style=for-the-badge&logo=Amazon%20DynamoDB&logoColor=white) ![MongoDB](https://img.shields.io/badge/MongoDB-%234ea94b.svg?style=for-the-badge&logo=mongodb&logoColor=white) ![MicrosoftSQLServer](https://img.shields.io/badge/Microsoft%20SQL%20Server-CC2927?style=for-the-badge&logo=microsoft%20sql%20server&logoColor=white) ![MySQL](https://img.shields.io/badge/mysql-4479A1.svg?style=for-the-badge&logo=mysql&logoColor=white) ![SurrealDB](https://img.shields.io/badge/SurrealDB-FF00A0?style=for-the-badge&logo=surrealdb&logoColor=white) ![Adobe Acrobat Reader](https://img.shields.io/badge/Adobe%20Acrobat%20Reader-EC1C24.svg?style=for-the-badge&logo=Adobe%20Acrobat%20Reader&logoColor=white) ![Adobe InDesign](https://img.shields.io/badge/Adobe%20InDesign-49021F?style=for-the-badge&logo=adobeindesign&logoColor=FF3366) ![Adobe Illustrator](https://img.shields.io/badge/adobe%20illustrator-%23FF9A00.svg?style=for-the-badge&logo=adobe%20illustrator&logoColor=white) ![Affinity Photo](https://img.shields.io/badge/affinityphoto-%237E4DD2.svg?style=for-the-badge&logo=affinity-photo&logoColor=white) ![mlflow](https://img.shields.io/badge/mlflow-%23d9ead3.svg?style=for-the-badge&logo=numpy&logoColor=blue) ![CloudBees](https://img.shields.io/badge/CloudBees-1997B5&?logo=cloudbees&logoColor=white&style=for-the-badge) ![Gitee](https://img.shields.io/badge/Gitee-C71D23?style=for-the-badge&logo=gitee&logoColor=white) ![Forgejo](https://img.shields.io/badge/forgejo-%23FB923C.svg?style=for-the-badge&logo=forgejo&logoColor=white) ![Babel](https://img.shields.io/badge/Babel-F9DC3e?style=for-the-badge&logo=babel&logoColor=black) ![Arduino](https://img.shields.io/badge/-Arduino-00979D?style=for-the-badge&logo=Arduino&logoColor=white) ![ElasticSearch](https://img.shields.io/badge/-ElasticSearch-005571?style=for-the-badge&logo=elasticsearch) ![Meta](https://img.shields.io/badge/Meta-%230467DF.svg?style=for-the-badge&logo=Meta&logoColor=white) ![Mosquitto](https://img.shields.io/badge/mosquitto-%233C5280.svg?style=for-the-badge&logo=eclipsemosquitto&logoColor=white) ![Portfolio](https://img.shields.io/badge/Portfolio-%23000000.svg?style=for-the-badge&logo=firefox&logoColor=#FF7139) ![SonarQube](https://img.shields.io/badge/SonarQube-black?style=for-the-badge&logo=sonarqube&logoColor=4E9BCD)
# 📊 GitHub Stats:
![](https://github-readme-stats.vercel.app/api?username=DagneMan123&theme=dark&hide_border=false&include_all_commits=true&count_private=false)<br/>
![](https://nirzak-streak-stats.vercel.app/?user=DagneMan123&theme=dark&hide_border=false)<br/>
![](https://github-readme-stats.vercel.app/api/top-langs/?username=DagneMan123&theme=dark&hide_border=false&include_all_commits=true&count_private=false&layout=compact)

## 🏆 GitHub Trophies
![](https://github-profile-trophy.vercel.app/?username=DagneMan123&theme=radical&no-frame=false&no-bg=true&margin-w=4)

### ✍️ Random Dev Quote
![](https://quotes-github-readme.vercel.app/api?type=horizontal&theme=radical)

### 🔝 Top Contributed Repo
![](https://github-contributor-stats.vercel.app/api?username=DagneMan123&limit=5&theme=dark&combine_all_yearly_contributions=true)

---
[![](https://visitcount.itsvg.in/api?id=DagneMan123&icon=0&color=0)](https://visitcount.itsvg.in)

  ## 💰 You can help me by Donating
  [![PayPal](https://img.shields.io/badge/PayPal-00457C?style=for-the-badge&logo=paypal&logoColor=white)](https://paypal.me/DagneMan123) 

  
<!-- Proudly created with GPRM ( https://gprm.itsvg.in ) -->
