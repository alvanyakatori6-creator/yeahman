    // --- ENGINE STATE ---
    let state = {
        resonance: 0,
        veilClamped: false,
        audioEnabled: true,
        autoDriveInterval: null,
        // Detailed data map for all 27 conceptual layers
        nodeDetails: {
            0: { title: "Al-Ghayb", desc: "The Absolute Unseen / Infinite Void" },
            1: { title: "Al-Ahadiyyah", desc: "The Essential Oneness / Pure Dhat" },
            2: { title: "Al-Wahdiyyah", desc: "The Oneness of Divine Attributes" },
            3: { title: "Jesus the Logos", desc: "The Triune Word Manifest / Nexus Vector" },
            4: { title: "Al-Qalam", desc: "The Cosmic Supreme Pen" },
            5: { title: "Al-Lawh", desc: "The Guarded Tablet of Destiny" },
            6: { title: "Al-Arsh", desc: "The Divine Empyrean Throne" },
            7: { title: "Al-Kursi", desc: "The Celestial Sovereign Footstool" },
            8: { title: "Al-Bayt al-Ma'mur", desc: "The Heavens' Frequented House" },
            9: { title: "Jesus the Prophet", desc: "The Abrahamic Ascetic Pivot" },
            10: { title: "Al-Malakut", desc: "The Medial Angelic Realm" },
            11: { title: "Nafas al-Rahman", desc: "The Breath of the Merciful Creator" },
            12: { title: "Al-Asma al-Husna", desc: "The Active Names of Perfection" },
            13: { title: "Al-Jabarut", desc: "The Realm of Boundless Power" },
            14: { title: "Al-Barzakh", desc: "The Medial Isthmus Horizon" },
            15: { title: "Insan al-Kamil", desc: "The Cosmic Perfect Archetype" },
            16: { title: "Al-Haqiqah", desc: "The Core Inner Reality" },
            17: { title: "Al-Shari'ah", desc: "The Structural Exoteric Law" },
            18: { title: "Al-Tariqah", desc: "The Illuminating Esoteric Path" },
            19: { title: "Cluster Gamma Prime", desc: "First Gateway of Multiplicity" },
            20: { title: "The Celestial Spheres", desc: "Cosmological Orbit Matrices" },
            21: { title: "The Elemental Horizon", desc: "Dense Material Condensation" },
            22: { title: "The Tripartite Soul", desc: "Anima Integration Phase" },
            23: { title: "The Microcosmic Core", desc: "Reflected Image Architecture" },
            24: { title: "Al-Fana", desc: "Annihilation of Self-Multiplicity" },
            25: { title: "Al-Baqa", desc: "Eternal Subsistence in Reality" },
            26: { title: "The Alpha-Omega Ring", desc: "Terminal Synthesis Matrix" },
            27: { title: "Al-Dhat Horizon", desc: "Complete Monadic Resonance Achieved" }
        }
    };

    // --- AUDIO SYSTEM (Web Audio API Synthesizer) ---
    const AudioEngine = {
        ctx: null,
        init() {
            if (!this.ctx) {
                this.ctx = new (window.AudioContext || window.webkitAudioContext)();
            }
        },
        playPing(frequency, type = 'sine', duration = 0.3) {
            if (!state.audioEnabled) return;
            this.init();
            try {
                let osc = this.ctx.createOscillator();
                let gain = this.ctx.createGain();
                
                osc.type = type;
                osc.frequency.setValueAtTime(frequency, this.ctx.currentTime);
                
                gain.gain.setValueAtTime(0.15, this.ctx.currentTime);
                gain.gain.exponentialRampToValueAtTime(0.0001, this.ctx.currentTime + duration);
                
                osc.connect(gain);
                gain.connect(this.ctx.destination);
                
                osc.start();
                osc.stop(this.ctx.currentTime + duration);
            } catch (e) {
                console.warn("Audio Context playback paused or blocked by browser policy.");
            }
        }
    };

    // --- DOM INITIALIZATION ---
    document.addEventListener("DOMContentLoaded", () => {
        renderMatrixGrids();
        updateEngineUI();
    });

    function renderMatrixGrids() {
        const clusterAlpha = document.getElementById("clusterAlpha");
        const clusterBeta = document.getElementById("clusterBeta");
        
        // Append missing layout wrapper for Cluster Gamma if it isn't hardcoded
        let clusterGamma = document.getElementById("clusterGamma");
        if (!clusterGamma) {
            const wrapper = document.createElement("div");
            wrapper.innerHTML = `
                <div class="text-[10px] text-pink-400 font-mono mb-1 mt-4">Cluster Gamma (19-27) • [Terminal Synthesis Loop]</div>
                <div id="clusterGamma" class="grid grid-cols-9 gap-1.5"></div>
            `;
            clusterBeta.parentNode.appendChild(wrapper);
            clusterGamma = document.getElementById("clusterGamma");
        }

        clusterAlpha.innerHTML = "";
        clusterBeta.innerHTML = "";
        clusterGamma.innerHTML = "";

        // Build 27 interactive matrix cells
        for (let i = 1; i <= 27; i++) {
            let cell = document.createElement("button");
            cell.id = `node-${i}`;
            cell.onclick = () => setResonanceDirect(i);
            cell.className = "bg-slate-900 border border-slate-800 rounded p-2 text-center transition-all duration-300 group hover:border-slate-600";
            cell.innerHTML = `
                <div class="text-xs font-mono font-bold text-slate-500 group-hover:text-slate-300 transition-colors">${i}</div>
                <div class="w-1.5 h-1.5 rounded-full bg-slate-800 mx-auto mt-1 transition-all duration-500 eval-dot"></div>
            `;
            
            // Assign specific highlights to target Jesus nodes
            if (i === 3) cell.classList.add("border-cyan-950", "hover:border-cyan-500");
            if (i === 9) cell.classList.add("border-purple-950", "hover:border-purple-500");

            if (i <= 9) clusterAlpha.appendChild(cell);
            else if (i <= 18) clusterBeta.appendChild(cell);
            else clusterGamma.appendChild(cell);
        }
    }

    // --- UTILITY STATE MUTATORS ---
    function alterResonance(delta) {
        if (state.veilClamped) {
            AudioEngine.playPing(150, 'sawtooth', 0.15);
            triggerStatusAlert("Action rejected: Veil Clamp is locked.");
            flashElementBorder("veilBtn", "border-rose-500");
            return;
        }

        let old = state.resonance;
        state.resonance = Math.max(0, Math.min(27, state.resonance + delta));
        
        if (old !== state.resonance) {
            if (delta > 0) AudioEngine.playPing(300 + (state.resonance * 20), 'sine', 0.25);
            else AudioEngine.playPing(250 + (state.resonance * 15), 'triangle', 0.2);
            updateEngineUI();
        }
    }

    function setResonanceDirect(targetValue) {
        if (state.veilClamped) {
            AudioEngine.playPing(150, 'sawtooth', 0.15);
            triggerStatusAlert("Action rejected: Veil Clamp is locked.");
            return;
        }
        state.resonance = targetValue;
        AudioEngine.playPing(400 + (targetValue * 10), 'sine', 0.3);
        updateEngineUI();
    }

    // --- PILLAR PIECE TRIGGERS ---
    function triggerPillar1() {
        alterResonance(3); // The Logos Word shift
    }

    function triggerPillar2() {
        alterResonance(-1); // Needle Correction calibration
    }

    function triggerPillar3() {
        state.veilClamped = !state.veilClamped;
        AudioEngine.playPing(state.veilClamped ? 180 : 350, 'square', 0.2);
        
        const btn = document.getElementById("veilBtn");
        const icon = document.getElementById("veilIcon");
        const label = document.getElementById("veilLabel");
        const badge = document.getElementById("clampBadge");

        if (state.veilClamped) {
            btn.className = "group relative bg-gradient-to-br from-rose-700 to-rose-800 text-white p-4 rounded-xl font-bold text-xs shadow-lg transition-all active:scale-95 flex flex-col items-center justify-center space-y-2 border border-rose-500/50";
            icon.className = "fa-solid fa-lock text-base group-hover:scale-125 transition-transform text-rose-100";
            label.innerText = "Veil Locked";
            badge.classList.remove("hidden");
            triggerStatusAlert("Veil Clamp established. Resonance value is frozen.");
        } else {
            btn.className = "group relative bg-gradient-to-br from-pink-600 to-pink-700 hover:from-pink-500 hover:to-pink-600 text-white p-4 rounded-xl font-bold text-xs shadow-lg transition-all active:scale-95 flex flex-col items-center justify-center space-y-2 border border-pink-400/30";
            icon.className = "fa-solid fa-shield text-base group-hover:scale-125 transition-transform text-white";
            label.innerText = "Veil Clamp";
            badge.classList.add("hidden");
            triggerStatusAlert("Veil dropped. Axis calculations available.");
        }
    }

    function quickCalibrate() {
        // Fast tracking to Jesus Logos Vector position
        setResonanceDirect(3);
        triggerStatusAlert("Calibrated to Node 3: The Logos Pivot Axis.");
    }

    function resetEngine() {
        state.resonance = 0;
        if (state.veilClamped) triggerPillar3();
        if (state.autoDriveInterval) toggleAutoDrive();
        AudioEngine.playPing(200, 'sine', 0.5);
        updateEngineUI();
        triggerStatusAlert("System reset. Current resonance: 0/27 (Al-Dhat Horizon).");
    }

    // --- UI ENGINE RE-RENDERER ---
    function updateEngineUI() {
        // Update raw pool value display text
        document.getElementById("poolValue").innerText = state.resonance;
        
        // Render descriptor text dynamically
        let currentInfo = state.nodeDetails[state.resonance];
        document.getElementById("currentNodeDesc").innerText = `${state.resonance}: ${currentInfo.title} (${currentInfo.desc})`;

        // Dynamic status bar tracker update
        let activeLabel = state.resonance === 0 ? "Al-Dhat Horizon" : currentInfo.title;
