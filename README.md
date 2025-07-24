<!DOCTYPE html>
<html>
<head>
    <title>横屏历史时间轴</title>
    <style>
        body { 
            margin: 0; 
            overflow: hidden; 
            background: #f0f0f0; 
            height: 100vh;
            display: flex;
            flex-direction: column;
        }
        .container {
            display: flex;
            height: 100%;
            position: relative;
        }
        canvas { 
            background: white; 
            height: 100%;
            width: 100%;
        }
        .controls { 
            padding: 10px;
            background: #333;
            display: flex;
            justify-content: center;
            gap: 20px;
        }
        button { 
            padding: 10px 20px; 
            cursor: pointer;
            background: #4CAF50;
            color: white;
            border: none;
            border-radius: 4px;
            font-size: 16px;
        }
        button:hover {
            background: #45a049;
        }
        .period-info {
            position: absolute;
            top: 20px;
            left: 20px;
            background: rgba(255,255,255,0.8);
            padding: 15px;
            border-radius: 8px;
            max-width: 300px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.2);
        }
    </style>
</head>
<body>
    <div class="container">
        <canvas id="historyCanvas"></canvas>
        <div class="period-info" id="periodInfo">
            <h2 id="periodName">史前时期</h2>
            <ul id="periodEvents">
                <li>石器时代</li>
                <li>农业革命</li>
            </ul>
        </div>
    </div>
    <div class="controls">
        <button id="prevBtn">上一个时期</button>
        <button id="nextBtn">下一个时期</button>
    </div>

    <script>
        const canvas = document.getElementById('historyCanvas');
        const ctx = canvas.getContext('2d');
        const prevBtn = document.getElementById('prevBtn');
        const nextBtn = document.getElementById('nextBtn');
        const periodInfo = document.getElementById('periodInfo');
        const periodName = document.getElementById('periodName');
        const periodEvents = document.getElementById('periodEvents');
        
        // 设置画布大小
        function resizeCanvas() {
            canvas.width = window.innerWidth;
            canvas.height = window.innerHeight - 60; // 减去控制栏高度
        }
        resizeCanvas();
        
        // 历史时期数据
        const historicalPeriods = [
            { 
                name: "史前时期 (公元前300万年-公元前3000年)", 
                color: "#8B4513", 
                events: ["石器时代", "农业革命"],
                startYear: -3000000,
                endYear: -3000
            },
            { 
                name: "古代文明 (公元前3000年-公元前500年)", 
                color: "#CD5C5C", 
                events: ["埃及金字塔", "汉谟拉比法典"],
                startYear: -3000,
                endYear: -500
            },
            { 
                name: "古典时期 (公元前500年-公元500年)", 
                color: "#4682B4", 
                events: ["希腊民主", "罗马帝国"],
                startYear: -500,
                endYear: 500
            },
            { 
                name: "中世纪 (500年-1500年)", 
                color: "#556B2F", 
                events: ["封建制度", "十字军东征"],
                startYear: 500,
                endYear: 1500
            },
            { 
                name: "文艺复兴 (1500年-1700年)", 
                color: "#9932CC", 
                events: ["人文主义", "科学革命"],
                startYear: 1500,
                endYear: 1700
            },
            { 
                name: "工业革命 (1700年-1900年)", 
                color: "#D2691E", 
                events: ["蒸汽机", "工厂制度"],
                startYear: 1700,
                endYear: 1900
            },
            { 
                name: "现代世界 (1900年-至今)", 
                color: "#1E90FF", 
                events: ["世界大战", "数字化革命"],
                startYear: 1900,
                endYear: new Date().getFullYear()
            }
        ];
        
        let currentPeriod = 0;
        let animationProgress = 0;
        let timelineScroll = 0;
        const timelineScale = 0.5; // 时间轴缩放比例
        
        function updateInfoBox() {
            const period = historicalPeriods[currentPeriod];
            periodName.textContent = period.name;
            periodEvents.innerHTML = period.events.map(event => `<li>${event}</li>`).join('');
            periodInfo.style.backgroundColor = period.color.replace(")", ", 0.7)").replace("rgb", "rgba");
        }
        
        function draw() {
            // 清除画布
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            
            const period = historicalPeriods[currentPeriod];
            
            // 绘制时间轴背景
            ctx.fillStyle = "#f5f5f5";
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            
            // 绘制时间轴主线
            const timelineY = canvas.height * 0.6;
            ctx.strokeStyle = "#333";
            ctx.lineWidth = 3;
            ctx.beginPath();
            ctx.moveTo(0, timelineY);
            ctx.lineTo(canvas.width, timelineY);
            ctx.stroke();
            
            // 计算时间范围
            const minYear = historicalPeriods[0].startYear;
            const maxYear = historicalPeriods[historicalPeriods.length - 1].endYear;
            const totalYears = maxYear - minYear;
            
            // 绘制时间刻度
            ctx.font = "14px Arial";
            ctx.fillStyle = "#333";
            ctx.textAlign = "center";
            
            // 计算主要刻度间隔
            let yearStep = 1000;
            if (totalYears / yearStep > 20) yearStep = 2000;
            if (totalYears / yearStep < 10) yearStep = 500;
            
            // 绘制刻度线和年份标签
            for (let year = Math.ceil(minYear / yearStep) * yearStep; year <= maxYear; year += yearStep) {
                const x = ((year - minYear) / totalYears) * canvas.width * timelineScale + timelineScroll;
                
                if (x >= -50 && x <= canvas.width + 50) {
                    ctx.beginPath();
                    ctx.moveTo(x, timelineY - 10);
                    ctx.lineTo(x, timelineY + 10);
                    ctx.stroke();
                    
                    ctx.fillText(year >= 0 ? year + "年" : "公元前" + (-year) + "年", x, timelineY + 30);
                }
            }
            
            // 绘制所有历史时期
            historicalPeriods.forEach((p, i) => {
                const startX = ((p.startYear - minYear) / totalYears) * canvas.width * timelineScale;
                const endX = ((p.endYear - minYear) / totalYears) * canvas.width * timelineScale;
                const width = endX - startX;
                
                // 只绘制可见部分
                if (endX + timelineScroll > 0 && startX + timelineScroll < canvas.width) {
                    // 时期背景块
                    ctx.fillStyle = p.color;
                    ctx.globalAlpha = i === currentPeriod ? 0.7 : 0.4;
                    ctx.fillRect(startX + timelineScroll, timelineY - 30, width, 60);
                    ctx.globalAlpha = 1.0;
                    
                    // 时期名称
                    if (width > 100) {
                        ctx.fillStyle = "#fff";
                        ctx.font = "bold 16px Arial";
                        ctx.textAlign = "center";
                        ctx.fillText(p.name.split(" ")[0], startX + width/2 + timelineScroll, timelineY);
                    }
                }
            });
            
            // 高亮当前时期
            const current = historicalPeriods[currentPeriod];
            const currentStartX = ((current.startYear - minYear) / totalYears) * canvas.width * timelineScale;
            const currentEndX = ((current.endYear - minYear) / totalYears) * canvas.width * timelineScale;
            const currentWidth = currentEndX - currentStartX;
            
            ctx.strokeStyle = "#000";
            ctx.lineWidth = 2;
            ctx.strokeRect(currentStartX + timelineScroll, timelineY - 30, currentWidth, 60);
            
            // 绘制当前时期的事件标记
            current.events.forEach((event, index) => {
                const eventYear = current.startYear + (current.endYear - current.startYear) * (index + 1) / (current.events.length + 1);
                const x = ((eventYear - minYear) / totalYears) * canvas.width * timelineScale + timelineScroll;
                const y = timelineY - 50 - index * 20 * animationProgress;
                
                if (x >= 0 && x <= canvas.width) {
                    // 连接到时间轴的线
                    ctx.beginPath();
                    ctx.moveTo(x, timelineY - 10);
                    ctx.lineTo(x, y + 15);
                    ctx.strokeStyle = current.color;
                    ctx.lineWidth = 1;
                    ctx.stroke();
                    
                    // 事件标记
                    ctx.beginPath();
                    ctx.arc(x, y, 10, 0, Math.PI * 2);
                    ctx.fillStyle = current.color;
                    ctx.fill();
                    ctx.strokeStyle = "#000";
                    ctx.stroke();
                    
                    // 事件文本
                    ctx.font = "16px Arial";
                    ctx.fillStyle = "#000";
                    ctx.textAlign = "center";
                    ctx.fillText(event, x, y - 15);
                }
            });
            
            // 动画效果
            if (animationProgress < 1) {
                animationProgress += 0.02;
                requestAnimationFrame(draw);
            }
        }
        
        function changePeriod(direction) {
            animationProgress = 0;
            currentPeriod += direction;
            
            // 循环检查
            if (currentPeriod < 0) currentPeriod = historicalPeriods.length - 1;
            if (currentPeriod >= historicalPeriods.length) currentPeriod = 0;
            
            // 自动滚动到当前时期
            const period = historicalPeriods[currentPeriod];
            const minYear = historicalPeriods[0].startYear;
            const maxYear = historicalPeriods[historicalPeriods.length - 1].endYear;
            const totalYears = maxYear - minYear;
            
            const periodCenter = ((period.startYear + period.endYear) / 2 - minYear) / totalYears;
            timelineScroll = canvas.width / 2 - periodCenter * canvas.width * timelineScale;
            
            updateInfoBox();
            draw();
        }
        
        // 事件监听
        prevBtn.addEventListener('click', () => changePeriod(-1));
        nextBtn.addEventListener('click', () => changePeriod(1));
        
        // 初始绘制
        updateInfoBox();
        draw();
        
        // 响应式调整
        window.addEventListener('resize', () => {
            resizeCanvas();
            draw();
        });
        
        // 添加键盘导航
        document.addEventListener('keydown', (e) => {
            if (e.key === 'ArrowLeft') changePeriod(-1);
            if (e.key === 'ArrowRight') changePeriod(1);
        });
    </script>
</body>
</html>
