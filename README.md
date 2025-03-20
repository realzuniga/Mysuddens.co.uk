<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Interactive Soundboard</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f5f5f5;
            padding: 20px;
            max-width: 800px;
            margin: 0 auto;
        }
        
        h1 {
            color: #333;
            text-align: center;
            margin-bottom: 30px;
        }
        
        .upload-container {
            background-color: white;
            border-radius: 8px;
            padding: 20px;
            box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
            margin-bottom: 30px;
        }
        
        .upload-form {
            display: flex;
            flex-direction: column;
            gap: 15px;
        }
        
        .form-group {
            display: flex;
            flex-direction: column;
            gap: 5px;
        }
        
        label {
            font-weight: bold;
            color: #555;
        }
        
        input[type="text"], input[type="file"] {
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 4px;
        }
        
        button {
            background-color: #4CAF50;
            color: white;
            border: none;
            padding: 12px;
            border-radius: 4px;
            cursor: pointer;
            font-weight: bold;
            transition: background-color 0.3s;
        }
        
        button:hover {
            background-color: #45a049;
        }
        
        .soundboard {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
            gap: 15px;
        }
        
        .sound-button {
            background-color: #3498db;
            color: white;
            border: none;
            padding: 15px 10px;
            border-radius: 8px;
            cursor: pointer;
            font-weight: bold;
            min-height: 80px;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            align-items: center;
            transition: transform 0.1s, background-color 0.3s;
            text-align: center;
            word-break: break-word;
        }
        
        .sound-button:hover {
            background-color: #2980b9;
            transform: scale(1.03);
        }
        
        .sound-button:active {
            transform: scale(0.97);
        }
        
        .sound-button .controls {
            display: flex;
            gap: 5px;
            margin-top: 5px;
        }
        
        .sound-button .controls button {
            padding: 3px 6px;
            font-size: 12px;
            background-color: rgba(0,0,0,0.2);
        }
        
        .sound-button .sound-name {
            font-size: 14px;
            margin-bottom: 5px;
        }
        
        .error-message {
            color: #e74c3c;
            margin-top: 5px;
            font-size: 14px;
        }
        
        .empty-state {
            text-align: center;
            padding: 30px;
            color: #7f8c8d;
            background-color: white;
            border-radius: 8px;
            box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
        }
        
        @media (max-width: 600px) {
            .soundboard {
                grid-template-columns: repeat(2, 1fr);
            }
        }
    </style>
</head>
<body>
    <h1>Interactive Soundboard</h1>
    
    <div class="upload-container">
        <h2>Add New Sound</h2>
        <div class="upload-form">
            <div class="form-group">
                <label for="sound-name">Sound Name:</label>
                <input type="text" id="sound-name" placeholder="Enter a name for your sound">
                <div id="name-error" class="error-message"></div>
            </div>
            
            <div class="form-group">
                <label for="sound-file">Sound File (MP3):</label>
                <input type="file" id="sound-file" accept="audio/mp3">
                <div id="file-error" class="error-message"></div>
            </div>
            
            <button id="add-sound">Add Sound</button>
        </div>
    </div>
    
    <h2>Your Sounds</h2>
    <div id="soundboard" class="soundboard">
        <div id="empty-state" class="empty-state">
            <h3>No sounds yet</h3>
            <p>Upload your first sound using the form above!</p>
        </div>
    </div>

    <script>
        document.addEventListener('DOMContentLoaded', function() {
            const soundboard = document.getElementById('soundboard');
            const emptyState = document.getElementById('empty-state');
            const addSoundButton = document.getElementById('add-sound');
            const soundNameInput = document.getElementById('sound-name');
            const soundFileInput = document.getElementById('sound-file');
            const nameError = document.getElementById('name-error');
            const fileError = document.getElementById('file-error');
            
            // Load sounds from localStorage
            const sounds = JSON.parse(localStorage.getItem('soundboardSounds')) || [];
            
            // Function to update soundboard display
            function updateSoundboard() {
                if (sounds.length === 0) {
                    emptyState.style.display = 'block';
                    return;
                }
                
                emptyState.style.display = 'none';
                
                // Clear existing buttons (except empty state)
                while (soundboard.firstChild && soundboard.firstChild !== emptyState) {
                    soundboard.removeChild(soundboard.firstChild);
                }
                
                // Add sound buttons
                sounds.forEach((sound, index) => {
                    const soundButton = document.createElement('div');
                    soundButton.className = 'sound-button';
                    
                    const soundName = document.createElement('div');
                    soundName.className = 'sound-name';
                    soundName.textContent = sound.name;
                    
                    // Create audio element
                    const audio = document.createElement('audio');
                    audio.src = sound.dataUrl;
                    
                    // Control buttons
                    const controls = document.createElement('div');
                    controls.className = 'controls';
                    
                    const deleteButton = document.createElement('button');
                    deleteButton.textContent = 'Delete';
                    deleteButton.addEventListener('click', (e) => {
                        e.stopPropagation(); // Prevent playing the sound
                        if (confirm(`Delete "${sound.name}"?`)) {
                            sounds.splice(index, 1);
                            localStorage.setItem('soundboardSounds', JSON.stringify(sounds));
                            updateSoundboard();
                        }
                    });
                    
                    controls.appendChild(deleteButton);
                    
                    // Play sound on button click
                    soundButton.addEventListener('click', () => {
                        // Stop all other audio first
                        document.querySelectorAll('audio').forEach(a => {
                            if (a !== audio) {
                                a.pause();
                                a.currentTime = 0;
                            }
                        });
                        
                        if (audio.paused) {
                            audio.play();
                        } else {
                            audio.pause();
                            audio.currentTime = 0;
                        }
                    });
                    
                    soundButton.appendChild(soundName);
                    soundButton.appendChild(controls);
                    soundboard.insertBefore(soundButton, emptyState);
                });
            }
            
            // Initialize soundboard
            updateSoundboard();
            
            // Add new sound
            addSoundButton.addEventListener('click', () => {
                // Reset error messages
                nameError.textContent = '';
                fileError.textContent = '';
                
                // Validate inputs
                if (!soundNameInput.value.trim()) {
                    nameError.textContent = 'Please enter a name for your sound';
                    return;
                }
                
                if (!soundFileInput.files || soundFileInput.files.length === 0) {
                    fileError.textContent = 'Please select an MP3 file';
                    return;
                }
                
                const file = soundFileInput.files[0];
                if (!file.type.includes('audio')) {
                    fileError.textContent = 'Please select a valid audio file';
                    return;
                }
                
                // Read file as data URL
                const reader = new FileReader();
                reader.onload = function(e) {
                    const sound = {
                        name: soundNameInput.value.trim(),
                        dataUrl: e.target.result
                    };
                    
                    sounds.push(sound);
                    localStorage.setItem('soundboardSounds', JSON.stringify(sounds));
                    
                    // Reset form
                    soundNameInput.value = '';
                    soundFileInput.value = '';
                    
                    // Update soundboard
                    updateSoundboard();
                };
                
                reader.readAsDataURL(file);
            });
        });
    </script>
</body>
</html>