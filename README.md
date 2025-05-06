# nsfw-face-demo
<!DOCTYPE html>
<html>
<head>
  <title>Face Censor Demo</title>
  <style>
    body { margin: 0; overflow: hidden; background: black; }
    canvas, video { position: absolute; top: 0; left: 0; }
  </style>
</head>
<body>
  <video id="video" autoplay muted playsinline width="640" height="480" style="display: none;"></video>
  <canvas id="canvas" width="640" height="480"></canvas>

  <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@3.11.0"></script>
  <script src="https://cdn.jsdelivr.net/npm/@tensorflow-models/blazeface"></script>

  <script>
    const video = document.getElementById('video');
    const canvas = document.getElementById('canvas');
    const ctx = canvas.getContext('2d');

    async function setupCamera() {
      const stream = await navigator.mediaDevices.getUserMedia({ video: true });
      video.srcObject = stream;
      return new Promise((resolve) => {
        video.onloadedmetadata = () => resolve(video);
      });
    }

    async function main() {
      await setupCamera();
      video.play();

      const model = await blazeface.load();

      async function renderFrame() {
        ctx.drawImage(video, 0, 0, canvas.width, canvas.height);
        const predictions = await model.estimateFaces(video, false);

        if (predictions.length > 0) {
          predictions.forEach(pred => {
            const [x, y, width, height] = [
              pred.topLeft[0],
              pred.topLeft[1],
              pred.bottomRight[0] - pred.topLeft[0],
              pred.bottomRight[1] - pred.topLeft[1]
            ];

            // Pixelate the face region
            const faceData = ctx.getImageData(x, y, width, height);
            const pixelSize = 10;
            for (let py = 0; py < height; py += pixelSize) {
              for (let px = 0; px < width; px += pixelSize) {
                const i = ((py * width) + px) * 4;
                const r = faceData.data[i];
                const g = faceData.data[i + 1];
                const b = faceData.data[i + 2];
                for (let dy = 0; dy < pixelSize; dy++) {
                  for (let dx = 0; dx < pixelSize; dx++) {
                    const ni = (((py + dy) * width) + (px + dx)) * 4;
                    faceData.data[ni] = r;
                    faceData.data[ni + 1] = g;
                    faceData.data[ni + 2] = b;
                  }
                }
              }
            }
            ctx.putImageData(faceData, x, y);
          });
        }
        requestAnimationFrame(renderFrame);
      }

      renderFrame();
    }

    main();
  </script>
</body>
</html>
