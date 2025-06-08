<!DOCTYPE html>
<html lang="en" >
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Cat Peeking with Confetti and Cheerful Tune</title>
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Poppins:wght@600&display=swap');
    :root {
      --color-bg: #ffffff;
      --color-box-brown: #a2672a;
      --color-text-secondary: #6b7280;
      --shadow-light: 0 6px 16px rgba(0,0,0,0.12);
      --radius: 0.75rem;
      --transition: 0.4s cubic-bezier(0.4, 0, 0.2, 1);
      --confetti-colors: #2563eb, #fbbf24, #10b981, #3b82f6, #f97316, #8b5cf6;
    }
    *,
    *::before,
    *::after {
      box-sizing: border-box;
    }
    body {
      margin: 0;
      font-family: 'Poppins', sans-serif;
      background: var(--color-bg);
      color: var(--color-text-secondary);
      display: flex;
      justify-content: center;
      padding: 5rem 1rem 6rem 1rem;
      min-height: 100vh;
    }
    main.container {
      max-width: 640px;
      width: 100%;
      text-align: center;
      position: relative;
      user-select: none;
    }
    /* Box wrapper with cat peek */
    .box-wrapper {
      position: relative;
      width: 320px;
      height: 320px;
      margin: 0 auto 2.5rem auto;
      cursor: pointer;
      outline-offset: 6px;
      outline-color: transparent;
      transition: outline-color var(--transition);
    }
    .box-wrapper:focus-visible {
      outline: 3px solid var(--color-box-brown);
      outline-offset: 6px;
    }
    .box {
      width: 320px;
      height: 320px;
      background-color: var(--color-box-brown);
      border-radius: var(--radius);
      box-shadow: var(--shadow-light);
      position: relative;
      z-index: 1;
      user-select: none;
      transition: filter var(--transition);
    }
    .cat-peek {
      position: absolute;
      bottom: 100%; /* above box */
      left: 50%;
      transform: translateX(-50%) translateY(25%);
      width: 180px;
      max-width: 90vw;
      pointer-events: none;
      opacity: 0;
      filter: drop-shadow(0 6px 8px rgba(0,0,0,0.15));
      transition:
        opacity var(--transition),
        transform var(--transition);
      z-index: 2;
      user-select: none;
    }
    .box-wrapper.active .cat-peek {
      opacity: 1;
      transform: translateX(-50%) translateY(0);
      pointer-events: auto;
    }
    /* Message with typing effect */
    .message-container {
      font-size: 1.25rem;
      font-weight: 600;
      color: var(--color-text-secondary);
      padding: 1rem 1.5rem;
      border-radius: var(--radius);
      box-shadow: var(--shadow-light);
      min-height: 5rem;
      white-space: pre-wrap;
      line-height: 1.4;
      user-select: text;
      max-width: 640px;
      margin: 0 auto;
      opacity: 0;
      pointer-events: none;
      transition: opacity 0.6s ease;
    }
    .message-container.visible {
      opacity: 1;
      pointer-events: auto;
    }
    /* Confetti container */
    .confetti-container {
      position: fixed;
      top: 0; left: 0; right: 0;
      height: 100vh;
      pointer-events: none;
      overflow: visible;
      z-index: 1000;
      opacity: 0;
      transition: opacity 0.6s ease;
    }
    .confetti-container.visible {
      opacity: 1;
    }
    .confetti-piece {
      position: absolute;
      width: 8px;
      height: 8px;
      border-radius: 50%;
      opacity: 0.85;
      animation-name: confettiFall, confettiFade;
      animation-timing-function: linear, ease-in-out;
      animation-iteration-count: infinite;
      animation-fill-mode: forwards;
      will-change: transform, opacity;
    }
    @keyframes confettiFall {
      0% {
        transform: translateY(-10px) translateX(0);
      }
      100% {
        transform: translateY(110vh) translateX(40px);
      }
    }
    @keyframes confettiFade {
      0%, 90% {
        opacity: 0.85;
      }
      100% {
        opacity: 0;
      }
    }
  </style>
</head>
<body>
  <main class="container" role="main" aria-label="Cat peeking from cardboard box with confetti and message">
    <div
      class="box-wrapper"
      role="button"
      tabindex="0"
      aria-pressed="false"
      aria-label="Cardboard box. Click or press Enter or Space to show or hide the cat peeking."
      id="boxWrapper"
    >
      <div class="cat-peek" id="catPeek" aria-hidden="true">
        <img src="https://cdn-icons-png.flaticon.com/512/616/616408.png" alt="Cute orange cat head and neck peeking out of box" width="180" height="180" decoding="async" loading="lazy" />
      </div>
      <div class="box" id="box"></div>
    </div>
    <section class="message-container" aria-live="polite" aria-atomic="true" id="message"></section>
  </main>
  <div class="confetti-container" aria-hidden="true" id="confettiContainer"></div>

  <script>
    (function(){
      // Elements
      const boxWrapper = document.getElementById('boxWrapper');
      const messageEl = document.getElementById('message');
      const confettiContainer = document.getElementById('confettiContainer');

      // State to track if effects started
      let hasStartedEffects = false;

      // Text for typing effect
      const textToType = "Hai Misa! Mungkin kamu sekarang lagi sedih banget... Tapi ingatlah bahwa setiap kesulitan akan membentukmu menjadi pribadi yang lebih kuat. So keep smiling (˶ᵔ ᵕ ᵔ˶)";
      let charIndex = 0;

      // Web Audio API setup for cheerful twinkle tune
      const AudioContext = window.AudioContext || window.webkitAudioContext;
      const audioCtx = new AudioContext();

      // Note frequencies for C major scale (middle octave)
      // Using note names for "Twinkle Twinkle Little Star" melody, tempo faster (~120bpm)
      // Notes durations in beats (quarter note = 1 beat), tempo = 120 bpm => 0.5 sec per beat

      // Melody notes with frequency & duration
      // Notes: C C G G A A G | F F E E D D C | G G F F E E D | G G F F E E D | C C G G A A G | F F E E D D C (Upbeat tempo)
      // Mapped to frequencies (Hz)
      // C4=261.63, D4=293.66, E4=329.63, F4=349.23, G4=392.00, A4=440.00

      const notes = [
        { freq: 261.63, duration: 1 }, // C
        { freq: 261.63, duration: 1 }, // C
        { freq: 392.00, duration: 1 }, // G
        { freq: 392.00, duration: 1 }, // G
        { freq: 440.00, duration: 1 }, // A
        { freq: 440.00, duration: 1 }, // A
        { freq: 392.00, duration: 2 }, // G (held 2 beats)

        { freq: 349.23, duration: 1 }, // F
        { freq: 349.23, duration: 1 }, // F
        { freq: 329.63, duration: 1 }, // E
        { freq: 329.63, duration: 1 }, // E
        { freq: 293.66, duration: 1 }, // D
        { freq: 293.66, duration: 1 }, // D
        { freq: 261.63, duration: 2 }, // C (held 2 beats)
       
        { freq: 392.00, duration: 1 }, // G
        { freq: 392.00, duration: 1 }, // G
        { freq: 349.23, duration: 1 }, // F
        { freq: 349.23, duration: 1 }, // F
        { freq: 329.63, duration: 1 }, // E
        { freq: 329.63, duration: 1 }, // E
        { freq: 293.66, duration: 2 }, // D (held 2 beats)

        { freq: 392.00, duration: 1 }, // G
        { freq: 392.00, duration: 1 }, // G
        { freq: 349.23, duration: 1 }, // F
        { freq: 349.23, duration: 1 }, // F
        { freq: 329.63, duration: 1 }, // E
        { freq: 329.63, duration: 1 }, // E
        { freq: 293.66, duration: 2 }, // D (held 2 beats)

        { freq: 261.63, duration: 1 }, // C
        { freq: 261.63, duration: 1 }, // C
        { freq: 392.00, duration: 1 }, // G
        { freq: 392.00, duration: 1 }, // G
        { freq: 440.00, duration: 1 }, // A
        { freq: 440.00, duration: 1 }, // A
        { freq: 392.00, duration: 2 }, // G (held 2 beats)

        { freq: 349.23, duration: 1 }, // F
        { freq: 349.23, duration: 1 }, // F
        { freq: 329.63, duration: 1 }, // E
        { freq: 329.63, duration: 1 }, // E
        { freq: 293.66, duration: 1 }, // D
        { freq: 293.66, duration: 1 }, // D
        { freq: 261.63, duration: 2 }, // C (held 2 beats)
      ];

      const tempo = 120; // bpm
      const beatDuration = 60 / tempo; // seconds per beat

      function playNote(freq, startTime, duration) {
        const osc = audioCtx.createOscillator();
        const gainNode = audioCtx.createGain();

        osc.type = 'triangle';
        osc.frequency.setValueAtTime(freq, startTime);

        gainNode.gain.setValueAtTime(0, startTime);
        gainNode.gain.linearRampToValueAtTime(0.15, startTime + 0.01); // attack
        gainNode.gain.linearRampToValueAtTime(0.15, startTime + duration * 0.9); // sustain
        gainNode.gain.linearRampToValueAtTime(0, startTime + duration); // release

        osc.connect(gainNode);
        gainNode.connect(audioCtx.destination);

        osc.start(startTime);
        osc.stop(startTime + duration);
      }

      function playTwinkleTune() {
        if(audioCtx.state === 'suspended') {
          audioCtx.resume();
        }
        let currentTime = audioCtx.currentTime + 0.1;
        for(let noteInfo of notes) {
          playNote(noteInfo.freq, currentTime, noteInfo.duration * beatDuration * 0.9);
          currentTime += noteInfo.duration * beatDuration;
        }
      }

      // Toggle the cat peek and start all effects on first click
      function toggleCat() {
        const wasActive = boxWrapper.classList.contains('active');
        boxWrapper.classList.toggle('active');
        boxWrapper.setAttribute('aria-pressed', !wasActive ? 'true' : 'false');

        if (!hasStartedEffects) {
          startConfettiAndTyping();
          playTwinkleTune();
          hasStartedEffects = true;
        }
      }

      boxWrapper.addEventListener('click', toggleCat);
      boxWrapper.addEventListener('keydown', e => {
        if(e.key === 'Enter' || e.key === ' ') {
          e.preventDefault();
          toggleCat();
        }
      });

      // Typing effect function
      function typeWriter() {
        if(charIndex < textToType.length) {
          messageEl.textContent += textToType.charAt(charIndex);
          charIndex++;
          setTimeout(typeWriter, 60); // slightly faster than previous
        }
      }
      // Show confetti and start typing
      function startConfettiAndTyping() {
        confettiContainer.classList.add('visible');
        messageEl.classList.add('visible');
        typeWriter();
      }

      // Initialize confetti pieces
      const confettiColors = ['#2563eb', '#fbbf24', '#10b981', '#3b82f6', '#f97316', '#8b5cf6'];
      const maxConfetti = 40;

      function randomRange(min, max) {
        return Math.random() * (max - min) + min;
      }

      function createConfettiPiece(index) {
        const confetti = document.createElement('div');
        confetti.classList.add('confetti-piece');
        confetti.style.backgroundColor = confettiColors[index % confettiColors.length];
        confetti.style.left = randomRange(0, window.innerWidth) + 'px';
        const size = randomRange(6, 10);
        confetti.style.width = confetti.style.height = size + 'px';
        confetti.style.animationDuration = randomRange(4000, 7000) + 'ms, ' + randomRange(4000, 8000) + 'ms';
        confetti.style.animationDelay = randomRange(0, 7000) + 'ms, ' + randomRange(0, 7000) + 'ms';
        confettiContainer.appendChild(confetti);
      }

      function initConfetti() {
        for(let i = 0; i < maxConfetti; i++) {
          createConfettiPiece(i);
        }
      }

      // Initialize confetti on page load but remain hidden until triggered
      initConfetti();

    })();
  </script>
</body>
</html>
