
<!DOCTYPE html>
<html lang="pt-br">
<head>
google-site-verification=qI-CRqB_eKefL4DDBdxt1AtDjHYwiiGb5FHxMi7NkKM

    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Relógio Completo</title>
    <style>
        body {
            margin: 0;
            height: 100vh;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            background: radial-gradient(circle, #000000, #0000FF);
            color: yellow;
            font-family: Arial, sans-serif;
            text-align: center;
            position: relative;
            overflow: hidden;
            border: 10px solid black;
        }
        .preload {
            position: absolute;
            width: 0;
            height: 0;
            overflow: hidden;
        }
        .title { 
            font-size: 2em; 
            text-shadow: 2px 2px 2px black; 
        }
        #location {
            font-size: 1.5em;
            text-shadow: 1px 1px 1px black;
            margin: 10px 0;
        }
        #clock { 
            font-size: 6em; 
            text-shadow: 2px 2px 2px black; 
            margin: 20px 0;
            z-index: 1;
        }
        .buttons { 
            margin-top: 20px; 
        }
        button {
            margin: 5px;
            padding: 15px;
            font-size: 1.5em;
            cursor: pointer;
            border: 2px solid black;
        }
        .clock-container {
            width: 250px;
            height: 250px;
            border: none;
            border-radius: 50%;
            position: relative;
            background: url('Sem título-1.jpg') no-repeat center;
            background-size: 100% 100%;
            display: flex;
            justify-content: center;
            align-items: center;
            overflow: hidden;
            z-index: 0;
        }
        .hand {
            position: absolute;
            bottom: 50%;
            left: 50%;
            transform-origin: bottom;
            background: gold;
            z-index: 1;
        }
        .hour { width: 6px; height: 50px; }
        .minute { width: 4px; height: 70px; }
        .second { width: 2px; height: 80px; background: red; }
        #stopwatch {
            font-size: 3em;
            font-weight: bold;
            text-shadow: 2px 2px 2px black;
            margin-top: 20px;
        }
        .notification {
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background: rgba(0, 128, 0, 0.9);
            color: white;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
            text-align: center;
            z-index: 1000;
            display: none;
        }
        .notification h2 {
            margin: 0;
            font-size: 1.5em;
        }
        .notification p {
            margin: 10px 0;
        }
        .notification button {
            background: #ffcc00;
            color: black;
            border: none;
            padding: 10px 20px;
            border-radius: 5px;
            cursor: pointer;
            font-size: 1em;
        }
        .notification button:hover {
            background: #e6b800;
        }
    </style>
</head>
<body>
    <div class="preload">
        <img src="https://images.unsplash.com/photo-1514999037859-b486988734f1">
        <img src="https://images.unsplash.com/photo-1502224562085-639556652f33">
        <img src="https://images.unsplash.com/photo-1448375240586-882707db888b">
        <img src="https://images.unsplash.com/photo-1600585154340-be6161a56a0c">
    </div>

    <div class="title">Relógios e Temporizadores</div>
    <div id="location">Carregando localização...</div>
    <div id="clock">Carregando hora...</div>
    <div class="clock-container">
        <div class="hand hour" id="hour"></div>
        <div class="hand minute" id="minute"></div>
        <div class="hand second" id="second"></div>
    </div>
    <div class="buttons">
        <button onclick="toggleStopwatch()">Iniciar/Pausar Cronômetro</button>
        <button onclick="setTimer()">Definir Timer</button>
        <button onclick="setAlarm()">Definir Alarme</button>
    </div>
    <div id="stopwatch">00:00</div>

    <div class="notification" id="notification">
        <h2 id="notification-title"></h2>
        <p id="notification-message"></p>
        <button onclick="closeNotification()">OK</button>
    </div>

    <script>
        // Função para obter e exibir a localização
        function getLocation() {
            if (navigator.geolocation) {
                navigator.geolocation.getCurrentPosition(
                    (position) => {
                        const lat = position.coords.latitude;
                        const lon = position.coords.longitude;

                        // Usar API Nominatim para converter coordenadas em localização
                        fetch(`https://nominatim.openstreetmap.org/reverse?lat=${lat}&lon=${lon}&format=json`)
                            .then(response => response.json())
                            .then(data => {
                                const location = data.address.city || data.address.town || data.address.village || "Localização desconhecida";
                                document.getElementById('location').textContent = `Localização: ${location}`;
                            })
                            .catch(error => {
                                console.error('Erro ao obter localização:', error);
                                document.getElementById('location').textContent = "Localização: Não disponível";
                            });
                    },
                    (error) => {
                        console.error('Erro na geolocalização:', error);
                        // Fallback para quando o usuário nega ou há erro
                        document.getElementById('location').textContent = "Localização: Não disponível";
                    }
                );
            } else {
                document.getElementById('location').textContent = "Localização: Não suportada";
            }
        }

        // Chama a função ao carregar a página
        window.onload = function() {
            getLocation();
            updateClock();
        };

        function updateClock() {
            const now = new Date();
            document.getElementById('clock').textContent = now.toLocaleTimeString();
            let hours = now.getHours() % 12;
            let minutes = now.getMinutes();
            let seconds = now.getSeconds();
            document.getElementById('hour').style.transform = `rotate(${(hours * 30) + (minutes / 2)}deg)`;
            document.getElementById('minute').style.transform = `rotate(${minutes * 6}deg)`;
            document.getElementById('second').style.transform = `rotate(${seconds * 6}deg)`;
        }
        setInterval(updateClock, 1000);
        
        let stopwatchInterval;
        let stopwatchTime = 0;
        let running = false;
        function toggleStopwatch() {
            if (running) {
                clearInterval(stopwatchInterval);
            } else {
                stopwatchInterval = setInterval(() => {
                    stopwatchTime++;
                    let minutes = String(Math.floor(stopwatchTime / 60)).padStart(2, '0');
                    let seconds = String(stopwatchTime % 60).padStart(2, '0');
                    document.getElementById('stopwatch').textContent = `${minutes}:${seconds}`;
                }, 1000);
            }
            running = !running;
        }
        
        const beepSound = new Audio("https://www.soundjay.com/button/beep-07.wav");

        function showNotification(title, message) {
            document.getElementById('notification-title').textContent = title;
            document.getElementById('notification-message').textContent = message;
            document.getElementById('notification').style.display = 'block';
            beepSound.play();
        }

        function closeNotification() {
            document.getElementById('notification').style.display = 'none';
        }

        function setTimer() {
            let timeLeft = prompt("Digite o tempo do timer em segundos:");
            timeLeft = parseInt(timeLeft);
            if (isNaN(timeLeft) || timeLeft <= 0) {
                alert("Valor inválido!");
                return;
            }
            const timer = setInterval(() => {
                if (timeLeft <= 0) {
                    clearInterval(timer);
                    showNotification("Timer Finalizado!", "O tempo do seu timer acabou! ⏰");
                } else {
                    console.log(`Timer: ${timeLeft} segundos restantes`);
                    timeLeft--;
                }
            }, 1000);
        }
        
        function setAlarm() {
            let alarmHour = prompt("Digite a hora do alarme (HH:MM):");
            if (!/^\d{2}:\d{2}$/.test(alarmHour)) {
                alert("Formato inválido! Use HH:MM");
                return;
            }
            let [hours, minutes] = alarmHour.split(":").map(Number);
            const checkAlarm = setInterval(() => {
                let now = new Date();
                if (now.getHours() === hours && now.getMinutes() === minutes) {
                    clearInterval(checkAlarm);
                    showNotification("Alarme!", "Seu alarme está tocando! 🔔");
                }
            }, 1000);
        }
    </script>
</body>
</html>
  
