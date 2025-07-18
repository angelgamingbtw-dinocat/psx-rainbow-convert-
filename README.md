<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Rainbow Image Generator</title>
    <style>
        body {
            font-family: 'Arial', sans-serif;
            margin: 0;
            padding: 20px;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            display: flex;
            flex-direction: column;
            align-items: center;
            color: white;
        }

        .container {
            max-width: 800px;
            width: 100%;
            background: rgba(255, 255, 255, 0.1);
            backdrop-filter: blur(10px);
            border-radius: 20px;
            padding: 30px;
            box-shadow: 0 15px 35px rgba(0, 0, 0, 0.2);
            border: 1px solid rgba(255, 255, 255, 0.2);
        }

        h1 {
            text-align: center;
            margin-bottom: 30px;
            font-size: 2.5em;
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
        }

        .drop-zone {
            width: 100%;
            height: 300px;
            border: 3px dashed rgba(255, 255, 255, 0.5);
            border-radius: 15px;
            display: flex;
            align-items: center;
            justify-content: center;
            cursor: pointer;
            transition: all 0.3s ease;
            margin-bottom: 20px;
            position: relative;
            overflow: hidden;
        }

        .drop-zone:hover {
            border-color: rgba(255, 255, 255, 0.8);
            background: rgba(255, 255, 255, 0.1);
        }

        .drop-zone.drag-over {
            border-color: #4CAF50;
            background: rgba(76, 175, 80, 0.2);
            transform: scale(1.02);
        }

        .drop-text {
            font-size: 1.2em;
            text-align: center;
            opacity: 0.8;
        }

        .image-preview {
            max-width: 100%;
            max-height: 100%;
            border-radius: 10px;
            object-fit: contain;
            display: none;
        }

        .controls {
            display: flex;
            gap: 15px;
            justify-content: center;
            align-items: center;
            margin-bottom: 20px;
            flex-wrap: wrap;
        }

        .control-group {
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 5px;
        }

        label {
            font-size: 0.9em;
            opacity: 0.9;
        }

        input[type="range"] {
            width: 120px;
            height: 6px;
            background: rgba(255, 255, 255, 0.3);
            border-radius: 3px;
            outline: none;
            appearance: none;
        }

        input[type="range"]::-webkit-slider-thumb {
            appearance: none;
            width: 20px;
            height: 20px;
            background: white;
            border-radius: 50%;
            cursor: pointer;
            box-shadow: 0 2px 6px rgba(0, 0, 0, 0.3);
        }

        input[type="range"]::-moz-range-thumb {
            width: 20px;
            height: 20px;
            background: white;
            border-radius: 50%;
            cursor: pointer;
            border: none;
            box-shadow: 0 2px 6px rgba(0, 0, 0, 0.3);
        }

        .btn {
            padding: 12px 24px;
            background: linear-gradient(45deg, #FF6B6B, #4ECDC4);
            color: white;
            border: none;
            border-radius: 25px;
            cursor: pointer;
            font-size: 1em;
            font-weight: bold;
            transition: all 0.3s ease;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
        }

        .btn:hover {
            transform: translateY(-2px);
            box-shadow: 0 6px 20px rgba(0, 0, 0, 0.3);
        }

        .btn:disabled {
            opacity: 0.5;
            cursor: not-allowed;
            transform: none;
        }

        .btn.recording {
            background: linear-gradient(45deg, #e74c3c, #c0392b);
        }

        .canvas-container {
            text-align: center;
            margin-top: 20px;
        }

        #canvas {
            border-radius: 10px;
            box-shadow: 0 5px 15px rgba(0, 0, 0, 0.2);
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>🌈 Rainbow Image Generator</h1>
        
        <div class="drop-zone" id="dropZone">
            <div class="drop-text">
                Drop an image here or click to select
                <br><small>Supports JPG, PNG, GIF, WebP</small>
            </div>
            <img class="image-preview" id="imagePreview" alt="Preview">
            <input type="file" id="fileInput" accept="image/*" style="display: none;">
        </div>

        <div class="controls">
            <div class="control-group">
                <label for="durationSlider">GIF Duration (seconds)</label>
                <input type="range" id="durationSlider" min="1" max="10" step="0.5" value="3">
                <span id="durationValue">3s</span>
            </div>
            
            <button class="btn" id="saveGifBtn" disabled>Save as GIF</button>
        </div>

        <div class="canvas-container">
            <canvas id="canvas" style="display: none;"></canvas>
        </div>
    </div>

    <script src="https://cdnjs.cloudflare.com/ajax/libs/gif.js/0.2.0/gif.js"></script>
    <script>
        class RainbowImageGenerator {
            constructor() {
                this.canvas = document.getElementById('canvas');
                this.ctx = this.canvas.getContext('2d');
                this.image = null;
                this.animationId = null;
                this.startTime = Date.now();
                this.colors = [
                    {r: 255, g: 0, b: 0},
                    {r: 255, g: 127, b: 0},
                    {r: 255, g: 255, b: 0},
                    {r: 0, g: 255, b: 0},
                    {r: 0, g: 255, b: 255},
                    {r: 0, g: 0, b: 255},
                    {r: 255, g: 0, b: 255}
                ];
                
                this.initializeElements();
                this.setupEventListeners();
            }

            initializeElements() {
                this.dropZone = document.getElementById('dropZone');
                this.fileInput = document.getElementById('fileInput');
                this.imagePreview = document.getElementById('imagePreview');
                this.durationSlider = document.getElementById('durationSlider');
                this.durationValue = document.getElementById('durationValue');
                this.saveGifBtn = document.getElementById('saveGifBtn');
            }

            setupEventListeners() {
                // Drag and drop
                this.dropZone.addEventListener('dragover', (e) => {
                    e.preventDefault();
                    this.dropZone.classList.add('drag-over');
                });

                this.dropZone.addEventListener('dragleave', () => {
                    this.dropZone.classList.remove('drag-over');
                });

                this.dropZone.addEventListener('drop', (e) => {
                    e.preventDefault();
                    this.dropZone.classList.remove('drag-over');
                    const files = e.dataTransfer.files;
                    if (files.length > 0) {
                        this.handleFile(files[0]);
                    }
                });

                // Click to select
                this.dropZone.addEventListener('click', () => {
                    this.fileInput.click();
                });

                this.fileInput.addEventListener('change', (e) => {
                    if (e.target.files.length > 0) {
                        this.handleFile(e.target.files[0]);
                    }
                });

                // Controls
                this.durationSlider.addEventListener('input', (e) => {
                    this.durationValue.textContent = `${e.target.value}s`;
                });

                this.saveGifBtn.addEventListener('click', () => {
                    this.saveAsGif();
                });
            }

            handleFile(file) {
                if (!file.type.startsWith('image/')) {
                    alert('Please select a valid image file.');
                    return;
                }

                const reader = new FileReader();
                reader.onload = (e) => {
                    this.image = new Image();
                    this.image.onload = () => {
                        this.setupCanvas();
                        this.startAnimation();
                        this.saveGifBtn.disabled = false;
                    };
                    this.image.src = e.target.result;
                    this.imagePreview.src = e.target.result;
                    this.imagePreview.style.display = 'block';
                    this.dropZone.querySelector('.drop-text').style.display = 'none';
                };
                reader.readAsDataURL(file);
            }

            setupCanvas() {
                const maxSize = 400;
                let { width, height } = this.image;

                if (width > maxSize || height > maxSize) {
                    const ratio = Math.min(maxSize / width, maxSize / height);
                    width *= ratio;
                    height *= ratio;
                }

                this.canvas.width = width;
                this.canvas.height = height;
                this.canvas.style.display = 'block';
            }

            getColorAtTime(time) {
                const tick = (time * 0.001) % 3 / 3;
                let gradientStops = [];
                
                for (let i = 0; i < this.colors.length; i++) {
                    let stopTime = tick + i / this.colors.length;
                    if (stopTime > 1) stopTime -= 1;
                    gradientStops.push({ 
                        time: stopTime, 
                        color: this.colors[i] 
                    });
                }
                
                gradientStops.sort((a, b) => a.time - b.time);
                
                // Find the current position in the gradient
                let currentPos = 0.5; // Middle of the gradient for consistent color
                let beforeStop = gradientStops[gradientStops.length - 1];
                let afterStop = gradientStops[0];
                
                for (let i = 0; i < gradientStops.length - 1; i++) {
                    if (currentPos >= gradientStops[i].time && currentPos <= gradientStops[i + 1].time) {
                        beforeStop = gradientStops[i];
                        afterStop = gradientStops[i + 1];
                        break;
                    }
                }
                
                // Interpolate between the two colors
                const t = (currentPos - beforeStop.time) / (afterStop.time - beforeStop.time);
                const r = Math.round(beforeStop.color.r + (afterStop.color.r - beforeStop.color.r) * t);
                const g = Math.round(beforeStop.color.g + (afterStop.color.g - beforeStop.color.g) * t);
                const b = Math.round(beforeStop.color.b + (afterStop.color.b - beforeStop.color.b) * t);
                
                return { r, g, b };
            }

            applyColorEffect(ctx, width, height, r, g, b) {
                const imageData = ctx.getImageData(0, 0, width, height);
                const data = imageData.data;
                
                for (let i = 0; i < data.length; i += 4) {
                    const alpha = data[i + 3];
                    const red = data[i];
                    const green = data[i + 1];
                    const blue = data[i + 2];
                    
                    // Only apply color effect to non-transparent pixels that aren't the gray background
                    if (alpha > 0 && !(red === 150 && green === 150 && blue === 150)) {
                        data[i] = (data[i] * r) / 255;     // Red
                        data[i + 1] = (data[i + 1] * g) / 255; // Green
                        data[i + 2] = (data[i + 2] * b) / 255; // Blue
                        // Alpha channel (i + 3) remains unchanged
                    }
                }
                
                ctx.putImageData(imageData, 0, 0);
            }

            startAnimation() {
                this.startTime = Date.now();
                
                const animate = () => {
                    const currentTime = Date.now();
                    const { r, g, b } = this.getColorAtTime(currentTime);
                    
                    this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
                    
                    // Draw original image
                    this.ctx.drawImage(this.image, 0, 0, this.canvas.width, this.canvas.height);
                    
                    // Apply color effect
                    this.applyColorEffect(this.ctx, this.canvas.width, this.canvas.height, r, g, b);
                    
                    this.animationId = requestAnimationFrame(animate);
                };
                
                animate();
            }

            generateFrameCanvas(time) {
                const { r, g, b } = this.getColorAtTime(time);
                
                // Create frame canvas with gray background
                const frameCanvas = document.createElement('canvas');
                frameCanvas.width = this.canvas.width;
                frameCanvas.height = this.canvas.height;
                const frameCtx = frameCanvas.getContext('2d');
                
                // Fill with gray background (150, 150, 150)
                frameCtx.fillStyle = 'rgb(150, 150, 150)';
                frameCtx.fillRect(0, 0, frameCanvas.width, frameCanvas.height);
                
                // Draw original image on top
                frameCtx.drawImage(this.image, 0, 0, this.canvas.width, this.canvas.height);
                
                // Apply color effect while preserving transparency
                this.applyColorEffect(frameCtx, this.canvas.width, this.canvas.height, r, g, b);
                
                return frameCanvas;
            }

            async saveAsGif() {
                // Using the same proven approach from RainbowFX
                const workerResponse = await fetch('https://cdnjs.cloudflare.com/ajax/libs/gif.js/0.2.0/gif.worker.js');
                const workerScript = await workerResponse.text();
                const workerBlob = new Blob([workerScript], { type: 'application/javascript' });
                const workerUrl = URL.createObjectURL(workerBlob);

                this.saveGifBtn.textContent = '🔴 Recording...';
                this.saveGifBtn.classList.add('recording');
                this.saveGifBtn.disabled = true;
                
                if (this.animationId) {
                    cancelAnimationFrame(this.animationId);
                }

                try {
                    const gif = new GIF({
                        workers: 2,
                        quality: 20,
                        width: this.canvas.width,
                        height: this.canvas.height,
                        workerScript: workerUrl,
                        transparent: 0x969696 // Set gray (150,150,150) as transparent color
                    });
                    
                    const onFinished = new Promise(resolve => {
                        gif.on('finished', blob => resolve(blob));
                    });

                    const duration = parseFloat(this.durationSlider.value) * 1000; // Convert to milliseconds
                    const frameRate = 30;
                    const totalFrames = frameRate * (duration / 1000);
                    const delay = 1000 / frameRate;
                    const startTime = Date.now();

                    // Generate frames using the same method as RainbowFX
                    for (let i = 0; i < totalFrames; i++) {
                        const currentTime = startTime + (i * delay);
                        const frameCanvas = this.generateFrameCanvas(currentTime);
                        gif.addFrame(frameCanvas, { delay: delay });
                    }

                    gif.render();
                    this.saveGifBtn.textContent = '💾 Compiling GIF...';
                    
                    const blob = await onFinished;
                    
                    // Download the GIF
                    const link = document.createElement('a');
                    link.download = 'rainbow-image.gif';
                    link.href = URL.createObjectURL(blob);
                    link.click();
                    URL.revokeObjectURL(link.href);

                } catch (error) {
                    console.error('Failed to create GIF:', error);
                    alert('Sorry, an error occurred while recording the GIF. Please try again.');
                } finally {
                    URL.revokeObjectURL(workerUrl);
                    this.saveGifBtn.textContent = 'Save as GIF';
                    this.saveGifBtn.classList.remove('recording');
                    this.saveGifBtn.disabled = false;
                    // Restart animation
                    this.startAnimation();
                }
            }
        }

        // Initialize the application
        new RainbowImageGenerator();
    </script>
</body>
</html>
