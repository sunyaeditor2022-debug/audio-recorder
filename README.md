# 我的錄音室
準備好就開始吧!

<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>書中錄音室</title>
    <style>
        body { font-family: 'Microsoft JhengHei', sans-serif; padding: 20px; text-align: center; background-color: #f9f3e9; color: #333; }
        .container { max-width: 500px; margin: 0 auto; }
        h1 { color: #8B4513; }
        button { background-color: #d4a574; border: none; padding: 15px 25px; margin: 10px; border-radius: 50px; font-size: 16px; cursor: pointer; color: white; box-shadow: 0 4px 8px rgba(0,0,0,0.1); }
        button:disabled { background-color: #ccc; cursor: not-allowed; }
        button.record { background-color: #e74c3c; }
        button.stop { background-color: #34495e; }
        button.play { background-color: #27ae60; }
        ul { list-style: none; padding: 0; text-align: left; }
        li { padding: 10px; border-bottom: 1px dashed #d4a574; display: flex; justify-content: space-between; align-items: center; }
        audio { width: 100%; margin-top: 5px; }
        .delete-btn { background-color: #c0392b; padding: 5px 10px; font-size: 12px; }
    </style>
</head>
<body>
    <div class="container">
        <h1>🎤 書中錄音室</h1>
        <p>記錄您的思緒、感想或朗讀。</p>
        
        <div id="status">準備就緒</div>
        
        <button id="recordBtn" class="record">● 開始錄音</button>
        <button id="stopBtn" class="stop" disabled>■ 停止</button>
        <button id="playBtn" class="play" disabled>▶ 播放最近錄音</button>
        
        <h3>📁 錄音存檔</h3>
        <p><em>錄音僅保存在您的裝置中。</em></p>
        <ul id="recordingList"></ul>
    </div>

    <script>
        let mediaRecorder;
        let audioChunks = [];
        let recordings = JSON.parse(localStorage.getItem('bookRecordings')) || [];
        let currentAudioBlob = null;

        // 更新存檔列表
        function updateList() {
            const list = document.getElementById('recordingList');
            list.innerHTML = '';
            recordings.forEach((rec, index) => {
                const li = document.createElement('li');
                const date = new Date(rec.timestamp).toLocaleString('zh-TW');
                li.innerHTML = `
                    <div>
                        <strong>錄音 ${index + 1}</strong><br>
                        <small>${date}</small>
                        <audio controls src="${rec.url}"></audio>
                    </div>
                    <button class="delete-btn" onclick="deleteRecording(${index})">刪除</button>
                `;
                list.appendChild(li);
            });
        }

        // 刪除錄音
        window.deleteRecording = function(index) {
            if (confirm('確定刪除此錄音嗎？')) {
                URL.revokeObjectURL(recordings[index].url);
                recordings.splice(index, 1);
                localStorage.setItem('bookRecordings', JSON.stringify(recordings));
                updateList();
            }
        };

        // 開始錄音
        document.getElementById('recordBtn').onclick = async () => {
            const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
            mediaRecorder = new MediaRecorder(stream);
            mediaRecorder.start();
            audioChunks = [];
            document.getElementById('status').textContent = '錄音中...';
            document.getElementById('recordBtn').disabled = true;
            document.getElementById('stopBtn').disabled = false;
            document.getElementById('playBtn').disabled = true;

            mediaRecorder.ondataavailable = event => {
                audioChunks.push(event.data);
            };

            mediaRecorder.onstop = () => {
                currentAudioBlob = new Blob(audioChunks, { type: 'audio/wav' });
                const audioUrl = URL.createObjectURL(currentAudioBlob);
                document.getElementById('playBtn').disabled = false;
                document.getElementById('status').textContent = '錄音完成，可播放或保存。';

                // 自動存檔
                const newRecording = {
                    url: audioUrl,
                    timestamp: Date.now()
                };
                recordings.unshift(newRecording);
                localStorage.setItem('bookRecordings', JSON.stringify(recordings));
                updateList();
                stream.getTracks().forEach(track => track.stop());
            };
        };

        // 停止錄音
        document.getElementById('stopBtn').onclick = () => {
            mediaRecorder.stop();
            document.getElementById('recordBtn').disabled = false;
            document.getElementById('stopBtn').disabled = true;
        };

        // 播放最近錄音
        document.getElementById('playBtn').onclick = () => {
            if (currentAudioBlob) {
                const audioUrl = URL.createObjectURL(currentAudioBlob);
                const audio = new Audio(audioUrl);
                audio.play();
            }
        };

        // 初始化列表
        updateList();
    </script>
</body>
</html>
      
