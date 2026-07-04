
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Neon Analog Clock</title>
<style>
  :root{
    --bg: #05070a;
    --grid: #0d1420;
    --cyan: #4ff2e0;
    --magenta: #ff3ea5;
    --amber: #ffb84f;
    --dim: #1c2733;
  }

  *{ box-sizing: border-box; }

  html, body{
    height: 100%;
    margin: 0;
  }

  body{
    min-height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
    background:
      linear-gradient(var(--bg), var(--bg)),
      repeating-linear-gradient(0deg, var(--grid) 0px, transparent 1px, transparent 40px),
      repeating-linear-gradient(90deg, var(--grid) 0px, transparent 1px, transparent 40px);
    font-family: 'Courier New', monospace;
  }

  .panel{
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 16px;
    padding: 30px;
  }

  canvas{
    filter: drop-shadow(0 0 18px rgba(79, 242, 224, 0.15));
  }

  .readout{
    display: flex;
    gap: 10px;
    color: var(--cyan);
    font-size: 13px;
    letter-spacing: 4px;
    text-transform: uppercase;
    text-shadow: 0 0 8px rgba(79, 242, 224, 0.6);
  }

  .readout span.sep{
    color: var(--dim);
  }

  .label{
    color: var(--dim);
    font-size: 10px;
    letter-spacing: 6px;
    text-transform: uppercase;
  }
</style>
</head>
<body>

  <div class="panel">
    <canvas id="clockCanvas" width="400" height="400"></canvas>
    <div class="readout" id="readout">00 : 00 : 00</div>
    <div class="label">Local Time</div>
  </div>

<script>
  const canvas = document.getElementById('clockCanvas');
  const ctx = canvas.getContext('2d');
  const readout = document.getElementById('readout');

  const size = canvas.width;
  const center = size / 2;
  const radius = size * 0.42;

  const colors = {
    cyan: '#4ff2e0',
    magenta: '#ff3ea5',
    amber: '#ffb84f',
    dim: '#1c2733',
    faceLine: 'rgba(79, 242, 224, 0.15)'
  };

  function drawFace(){
    // outer ring
    ctx.beginPath();
    ctx.arc(center, center, radius + 12, 0, Math.PI * 2);
    ctx.strokeStyle = colors.dim;
    ctx.lineWidth = 2;
    ctx.stroke();

    ctx.beginPath();
    ctx.arc(center, center, radius, 0, Math.PI * 2);
    ctx.strokeStyle = colors.faceLine;
    ctx.lineWidth = 1;
    ctx.stroke();

    // tick marks
    for(let i = 0; i < 60; i++){
      const angle = (i * 6) * Math.PI / 180;
      const isHour = i % 5 === 0;
      const outer = radius - 4;
      const inner = isHour ? radius - 18 : radius - 10;

      const x1 = center + outer * Math.sin(angle);
      const y1 = center - outer * Math.cos(angle);
      const x2 = center + inner * Math.sin(angle);
      const y2 = center - inner * Math.cos(angle);

      ctx.beginPath();
      ctx.moveTo(x1, y1);
      ctx.lineTo(x2, y2);
      ctx.strokeStyle = isHour ? colors.cyan : colors.dim;
      ctx.lineWidth = isHour ? 2.5 : 1.5;
      ctx.stroke();
    }

    // hour numerals as small ticks with glow at 12/3/6/9
    ctx.font = '14px Courier New';
    ctx.fillStyle = colors.cyan;
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';
    const marks = [
      {n:'12', a:0}, {n:'3', a:90}, {n:'6', a:180}, {n:'9', a:270}
    ];
    marks.forEach(m => {
      const rad = m.a * Math.PI / 180;
      const x = center + (radius - 34) * Math.sin(rad);
      const y = center - (radius - 34) * Math.cos(rad);
      ctx.fillText(m.n, x, y);
    });
  }

  function drawHand(angleDeg, length, width, color, glow){
    const rad = angleDeg * Math.PI / 180;
    const x = center + length * Math.sin(rad);
    const y = center - length * Math.cos(rad);

    ctx.save();
    ctx.shadowColor = color;
    ctx.shadowBlur = glow;
    ctx.beginPath();
    ctx.moveTo(center, center);
    ctx.lineTo(x, y);
    ctx.strokeStyle = color;
    ctx.lineWidth = width;
    ctx.lineCap = 'round';
    ctx.stroke();
    ctx.restore();
  }

  function drawCenter(){
    ctx.beginPath();
    ctx.arc(center, center, 6, 0, Math.PI * 2);
    ctx.fillStyle = colors.amber;
    ctx.shadowColor = colors.amber;
    ctx.shadowBlur = 10;
    ctx.fill();
    ctx.shadowBlur = 0;
  }

  function pad(n){ return n.toString().padStart(2, '0'); }

  function render(){
    ctx.clearRect(0, 0, size, size);

    const now = new Date();
    const h = now.getHours() % 12;
    const m = now.getMinutes();
    const s = now.getSeconds();
    const ms = now.getMilliseconds();

    const secAngle = (s + ms / 1000) * 6;
    const minAngle = (m + s / 60) * 6;
    const hourAngle = (h + m / 60) * 30;

    drawFace();
    drawHand(hourAngle, radius * 0.5, 6, colors.cyan, 10);
    drawHand(minAngle, radius * 0.72, 4, colors.magenta, 8);
    drawHand(secAngle, radius * 0.82, 1.5, colors.amber, 6);
    drawCenter();

    readout.innerHTML =
      pad(now.getHours()) + ' <span class="sep">:</span> ' +
      pad(m) + ' <span class="sep">:</span> ' +
      pad(s);

    requestAnimationFrame(render);
  }

  requestAnimationFrame(render);
</script>

</body>
</html>
