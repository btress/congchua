<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hiệu ứng Pháo Hoa</title>
    <style>
        
        body {
            background: #000000;
            margin: 0;
        }
        a {
            color: #fff;
            font-size: 12px;
        }
        canvas {
            display: block;
        }
        #newYearText {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%); /* Căn giữa cả theo chiều ngang và dọc */
            font-size: 30px;
            color: #fff;
            font-weight: bold;
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.7);
        }
    </style>
</head>
<body>
    <canvas id="canvas"></canvas>
    <div id="newYearText">Năm mới vui vẻ nhó !!! chúc công chúa của tớ ngày càng dethun nhó 💜</div>


    <script>
        // Khi animating trên canvas, tốt nhất là sử dụng requestAnimationFrame thay vì setTimeout hoặc setInterval
        // Không phải tất cả các trình duyệt đều hỗ trợ, đôi khi cần thêm tiền tố, vì vậy chúng ta cần một shim
        window.requestAnimFrame = (function() {
            return window.requestAnimationFrame ||
                window.webkitRequestAnimationFrame ||
                window.mozRequestAnimationFrame ||
                function(callback) {
                    window.setTimeout(callback, 1000 / 60);
                };
        })();

        var canvas = document.getElementById('canvas'),
            ctx = canvas.getContext('2d'),
            cw = window.innerWidth,
            ch = window.innerHeight,
            fireworks = [],
            particles = [],
            hearts = [],
            hue = 120,
            limiterTotal = 20,
            limiterTick = 0,
            timerTotal = 500,
            timerTick = 0,
            mousedown = false,
            mx, my;

        canvas.width = cw;
        canvas.height = ch;

        // Hàm random lấy số ngẫu nhiên trong khoảng min và max
        function random(min, max) {
            return Math.random() * (max - min) + min;
        }

        // Tính khoảng cách giữa hai điểm
        function calculateDistance(p1x, p1y, p2x, p2y) {
            var xDistance = p1x - p2x,
                yDistance = p1y - p2y;
            return Math.sqrt(Math.pow(xDistance, 2) + Math.pow(yDistance, 2));
        }

        // Tạo pháo hoa
        function Firework(sx, sy, tx, ty) {
            this.x = sx;
            this.y = sy;
            this.sx = sx;
            this.sy = sy;
            this.tx = tx;
            this.ty = ty;
            this.distanceToTarget = calculateDistance(sx, sy, tx, ty);
            this.distanceTraveled = 0;
            this.coordinates = [];
            this.coordinateCount = 3;

            while (this.coordinateCount--) {
                this.coordinates.push([this.x, this.y]);
            }
            this.angle = Math.atan2(ty - sy, tx - sx);
            this.speed = 2;
            this.acceleration = 1.05;
            this.brightness = random(50, 70);
            this.targetRadius = 1;
        }

        // Cập nhật pháo hoa
        Firework.prototype.update = function(index) {
            this.coordinates.pop();
            this.coordinates.unshift([this.x, this.y]);

            if (this.targetRadius < 8) {
                this.targetRadius += 0.3;
            } else {
                this.targetRadius = 1;
            }

            this.speed *= this.acceleration;

            var vx = Math.cos(this.angle) * this.speed,
                vy = Math.sin(this.angle) * this.speed;

            this.distanceTraveled = calculateDistance(this.sx, this.sy, this.x + vx, this.y + vy);

            if (this.distanceTraveled >= this.distanceToTarget) {
                createParticles(this.tx, this.ty);
                fireworks.splice(index, 1);
            } else {
                this.x += vx;
                this.y += vy;
            }
        }

        // Vẽ pháo hoa
        Firework.prototype.draw = function() {
            ctx.beginPath();
            ctx.moveTo(this.coordinates[this.coordinates.length - 1][0], this.coordinates[this.coordinates.length - 1][1]);
            ctx.lineTo(this.x, this.y);
            ctx.strokeStyle = 'hsl(' + hue + ', 100%, ' + this.brightness + '%)';
            ctx.stroke();
        }

        // Tạo hạt
        function Particle(x, y) {
            this.x = x;
            this.y = y;
            this.coordinates = [];
            this.coordinateCount = 5;

            while (this.coordinateCount--) {
                this.coordinates.push([this.x, this.y]);
            }

            this.angle = random(0, Math.PI * 2);
            this.speed = random(1, 10);
            this.friction = 0.95;
            this.gravity = 0.6;
            this.hue = random(hue - 20, hue + 20);
            this.brightness = random(50, 80);
            this.alpha = 1;
            this.decay = random(0.0075, 0.009);
        }

        // Cập nhật hạt
        Particle.prototype.update = function(index) {
            this.coordinates.pop();
            this.coordinates.unshift([this.x, this.y]);
            this.speed *= this.friction;
            this.x += Math.cos(this.angle) * this.speed;
            this.y += Math.sin(this.angle) * this.speed + this.gravity;
            this.alpha -= this.decay;

            if (this.alpha <= this.decay) {
                particles.splice(index, 1);
            }
        }

        // Vẽ hạt
        Particle.prototype.draw = function() {
            ctx.beginPath();
            ctx.moveTo(this.coordinates[this.coordinates.length - 1][0], this.coordinates[this.coordinates.length - 1][1]);
            ctx.lineTo(this.x, this.y);
            ctx.strokeStyle = 'hsla(' + this.hue + ', 100%, ' + this.brightness + '%, ' + this.alpha + ')';
            ctx.stroke();
        }

        // Tạo nhóm hạt (nổ pháo hoa)
        function createParticles(x, y) {
            var particleCount = 20;
            while (particleCount--) {
                particles.push(new Particle(x, y));
            }
        }

        function Heart(x, y) {
            this.x = x;
            this.y = y;
            this.size = random(10, 20);
            this.alpha = 1;
            this.speedX = random(-2, 2);
            this.speedY = random(-1, -3);
            this.gravity = 0.02;
            this.decay = random(0.005, 0.01);
        }
        Heart.prototype.draw = function() {
            ctx.save();
            ctx.translate(this.x, this.y);
            ctx.scale(this.size / 20, this.size / 20);
            ctx.beginPath();
            ctx.moveTo(0, 0);
            ctx.bezierCurveTo(-10, -10, -20, 10, 0, 20);
            ctx.bezierCurveTo(20, 10, 10, -10, 0, 0);
            ctx.closePath();
            ctx.fillStyle = `rgba(128, 0, 128, ${this.alpha})`;
            ctx.fill();
            ctx.restore();
        }
        Heart.prototype.update = function(index) {
            this.x += this.speedX;
            this.y += this.speedY;
            this.speedY += this.gravity;
            this.alpha -= this.decay;

            if (this.alpha <= 0) {
                hearts.splice(index, 1);
            }
        }

        function createHearts() {
            if (Math.random() < 0.1) {
                hearts.push(new Heart(random(0, cw), ch));
            }
        }

        // Vòng lặp chính
        function loop() {
            requestAnimFrame(loop);

            hue += 0.5;

            ctx.globalCompositeOperation = 'destination-out';
            ctx.fillStyle = 'rgba(0, 0, 0, 0.5)';
            ctx.fillRect(0, 0, cw, ch);

            ctx.globalCompositeOperation = 'lighter';

            var i = fireworks.length;
            while (i--) {
                fireworks[i].draw();
                fireworks[i].update(i);
            }

            var i = particles.length;
            while (i--) {
                particles[i].draw();
                particles[i].update(i);
            }

            if (timerTick >= timerTotal) {
                timerTick = 0;
            } else {
                var temp = timerTick % 400;
                if (temp <= 15) {
                    fireworks.push(new Firework(100, ch, random(190, 200), random(90, 100)));
                    fireworks.push(new Firework(cw - 100, ch, random(cw - 200, cw - 190), random(90, 100)));
                }

                var temp3 = temp / 10;

                if (temp > 319) {
                    fireworks.push(new Firework(300 + (temp3 - 31) * 100, ch, 300 + (temp3 - 31) * 100, 200));
                }

                timerTick++;
            }

            if (limiterTick >= limiterTotal) {
                if (mousedown) {
                    fireworks.push(new Firework(cw / 2, ch, mx, my));
                    limiterTick = 0;
                }
            } else {
                limiterTick++;
            }

            createHearts();
            for (let i = hearts.length - 1; i >= 0; i--) {
                hearts[i].draw();
                hearts[i].update(i);
            }

            if (timerTick >= timerTotal) {
                timerTick = 0;
            } else {
                timerTick++;
            }

            if (limiterTick >= limiterTotal) {
                if (mousedown) {
                    fireworks.push(new Firework(cw / 2, ch, mx, my));
                    limiterTick = 0;
                }
            } else {
                limiterTick++;
            }
        }

        // Sự kiện chuột
        canvas.addEventListener('mousemove', function(e) {
            mx = e.pageX - canvas.offsetLeft;
            my = e.pageY - canvas.offsetTop;
        });

        canvas.addEventListener('mousedown', function(e) {
            e.preventDefault();
            mousedown = true;
        });

        canvas.addEventListener('mouseup', function(e) {
            e.preventDefault();
            mousedown = false;
        });

        window.onload = loop;
    </script>
</body>
</html>
