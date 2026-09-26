// ==UserScript==
// @name         PKHunt - Auto Baú V10.0 (Widget UI Lateral)
// @namespace    http://tampermonkey.net/
// @version      10.0
// @description  Widget flutuante retrátil com menu popup lateral inteligente (não sobrepõe outros ícones).
// @match        https://pkhunt.online/play*
// @match        *://pkhunt.online/*
// @run-at       document-idle
// @grant        none
// ==/UserScript==

(function() {
    'use strict';

    // --- CARREGAR E SANITIZAR CONFIGURAÇÕES SALVAS ---
    function carregarConfiguracoes() {
        let configSalva = {};
        try {
            configSalva = JSON.parse(localStorage.getItem('pkhunt_chest_config')) || {};
        } catch (e) {
            configSalva = {};
        }

        const posicaoValida = configSalva.posicaoWidget && typeof configSalva.posicaoWidget.top === 'number' && typeof configSalva.posicaoWidget.left === 'number';

        return {
            ativo: configSalva.ativo !== undefined ? configSalva.ativo : true,
            autoReloadAtivo: configSalva.autoReloadAtivo !== undefined ? configSalva.autoReloadAtivo : true,
            tempoReloadMin: configSalva.tempoReloadMin || 5,
            posicaoWidget: posicaoValida ? configSalva.posicaoWidget : { top: 74, left: window.innerWidth - 70 }
        };
    }

    let config = carregarConfiguracoes();

    let executando = false;
    let ultimoProcessado = 0;
    const SELETOR_BOTAO_MENU = 'body > main > header > nav > button:nth-child(2)';

    // --- 1. CRIAR WIDGET FLUTUANTE ---
    function criarWidgetFlutuante() {
        if (document.getElementById('pkhunt-chest-widget-container')) return;

        const topPos = (config.posicaoWidget && typeof config.posicaoWidget.top === 'number') ? config.posicaoWidget.top : 74;
        const leftPos = (config.posicaoWidget && typeof config.posicaoWidget.left === 'number') ? config.posicaoWidget.left : (window.innerWidth - 70);

        const container = document.createElement('div');
        container.id = 'pkhunt-chest-widget-container';
        container.style.position = 'fixed';
        container.style.top = topPos + 'px';
        container.style.left = leftPos + 'px';
        container.style.zIndex = '9999998';
        container.style.fontFamily = 'Segoe UI, Tahoma, Geneva, Verdana, sans-serif';

        const corBadge = config.ativo ? 'linear-gradient(135deg, #d97706, #b45309)' : 'linear-gradient(135deg, #475569, #334155)';
        const bordaBadge = config.ativo ? '#fbbf24' : '#94a3b8';

        container.innerHTML = `
            <!-- Ícone do Baú Flutuante -->
            <div id="pkhunt-chest-badge" title="Segure e arraste para mover!" style="
                width: 44px;
                height: 44px;
                background: ${corBadge};
                border: 2px solid ${bordaBadge};
                border-radius: 50%;
                display: flex;
                align-items: center;
                justify-content: center;
                cursor: grab;
                box-shadow: 0 4px 14px rgba(0,0,0,0.5);
                transition: transform 0.2s;
                user-select: none;
                filter: ${config.ativo ? 'none' : 'grayscale(80%)'};
            ">
                <img src="/ui/species-chest-elite.png" style="width: 30px; height: 30px; image-rendering: pixelated; pointer-events: none;" alt="Baú">
            </div>

            <!-- Popup Menu Expansível Lateral -->
            <div id="pkhunt-chest-popup" style="
                display: none;
                position: absolute;
                top: 0px;
                width: 230px;
                background: rgba(15, 23, 42, 0.96);
                border: 1px solid #f59e0b;
                border-radius: 10px;
                padding: 12px;
                color: #e2e8f0;
                box-shadow: 0 10px 25px rgba(0,0,0,0.6);
                backdrop-filter: blur(5px);
                flex-direction: column;
                gap: 10px;
                font-size: 12px;
            ">
                <!-- Cabeçalho -->
                <div style="display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid #334155; padding-bottom: 8px;">
                    <strong style="color: #fbbf24; display: flex; align-items: center; gap: 6px;">
                        <img src="/ui/species-chest-elite.png" style="width: 18px; height: 18px; image-rendering: pixelated;"> Auto Baú
                    </strong>
                    <label style="display: flex; align-items: center; gap: 6px; cursor: pointer; font-weight: bold;">
                        <input type="checkbox" id="check-chest-ativo" ${config.ativo ? 'checked' : ''} style="width: 15px; height: 15px; cursor: pointer;">
                        <span id="pkhunt-chest-status-txt" style="color: ${config.ativo ? '#4ade80' : '#ef4444'}; font-size: 11px;">
                            ${config.ativo ? 'LIGADO' : 'DESLIGADO'}
                        </span>
                    </label>
                </div>

                <!-- Configuração Auto-Reload -->
                <div style="display: flex; flex-direction: column; gap: 6px;">
                    <div style="display: flex; justify-content: space-between; align-items: center;">
                        <span style="color: #94a3b8; font-size: 11px;">Auto-Reload Página:</span>
                        <input type="checkbox" id="check-reload-ativo" ${config.autoReloadAtivo ? 'checked' : ''} style="width: 15px; height: 15px; cursor: pointer;">
                    </div>
                    <div style="display: flex; justify-content: space-between; align-items: center;">
                        <span style="color: #94a3b8; font-size: 11px;">Tempo Reload (min):</span>
                        <input type="number" id="input-reload-min" value="${config.tempoReloadMin}" min="1" max="60" style="width: 50px; background: #1e293b; color: #fff; border: 1px solid #475569; border-radius: 4px; padding: 2px 4px; text-align: center;">
                    </div>
                </div>

                <!-- Botão de Guardar Configurações -->
                <div style="display: flex; gap: 6px; margin-top: 4px;">
                    <button id="btn-salvar-chest-config" style="flex: 1; background: #d97706; color: white; border: none; border-radius: 4px; padding: 6px; cursor: pointer; font-weight: bold;">Salvar Configuração</button>
                </div>
            </div>
        `;

        document.body.appendChild(container);

        const badge = document.getElementById('pkhunt-chest-badge');
        const popup = document.getElementById('pkhunt-chest-popup');

        // --- ALINHAMENTO LATERAL INTELIGENTE ---
        function atualizarPosicaoPopupLateral() {
            const currentLeft = container.offsetLeft;
            const metadeTela = window.innerWidth / 2;

            if (currentLeft > metadeTela) {
                // Se está na metade direita da tela, popup abre para a ESQUERDA do ícone
                popup.style.right = '52px';
                popup.style.left = 'auto';
            } else {
                // Se está na metade esquerda da tela, popup abre para a DIREITA do ícone
                popup.style.left = '52px';
                popup.style.right = 'auto';
            }
        }

        // --- SISTEMA DE ARRASTAR ---
        let isDragging = false;
        let startX, startY, initialLeft, initialTop;

        badge.addEventListener('mousedown', (e) => {
            isDragging = false;
            startX = e.clientX;
            startY = e.clientY;
            initialLeft = container.offsetLeft;
            initialTop = container.offsetTop;
            badge.style.cursor = 'grabbing';

            const onMouseMove = (moveEvent) => {
                const dx = moveEvent.clientX - startX;
                const dy = moveEvent.clientY - startY;

                if (Math.abs(dx) > 3 || Math.abs(dy) > 3) {
                    isDragging = true;
                    popup.style.display = 'none';
                }

                if (isDragging) {
                    let newLeft = initialLeft + dx;
                    let newTop = initialTop + dy;

                    newLeft = Math.max(10, Math.min(window.innerWidth - 60, newLeft));
                    newTop = Math.max(10, Math.min(window.innerHeight - 60, newTop));

                    container.style.left = newLeft + 'px';
                    container.style.top = newTop + 'px';
                }
            };

            const onMouseUp = () => {
                document.removeEventListener('mousemove', onMouseMove);
                document.removeEventListener('mouseup', onMouseUp);
                badge.style.cursor = 'grab';

                if (isDragging) {
                    config.posicaoWidget = {
                        top: container.offsetTop,
                        left: container.offsetLeft
                    };
                    localStorage.setItem('pkhunt_chest_config', JSON.stringify(config));
                }
            };

            document.addEventListener('mousemove', onMouseMove);
            document.addEventListener('mouseup', onMouseUp);
        });

        // Hover
        let timerHover;
        container.addEventListener('mouseenter', () => {
            if (!isDragging) {
                clearTimeout(timerHover);
                atualizarPosicaoPopupLateral();
                popup.style.display = 'flex';
                badge.style.transform = 'scale(1.1)';
            }
        });

        container.addEventListener('mouseleave', () => {
            timerHover = setTimeout(() => {
                popup.style.display = 'none';
                badge.style.transform = 'scale(1)';
            }, 400);
        });

        // Checkbox Ativo
        document.getElementById('check-chest-ativo').addEventListener('change', function(e) {
            config.ativo = e.target.checked;
            localStorage.setItem('pkhunt_chest_config', JSON.stringify(config));

            if (config.ativo) {
                atualizarStatusText('LIGADO', '#4ade80');
                badge.style.background = 'linear-gradient(135deg, #d97706, #b45309)';
                badge.style.borderColor = '#fbbf24';
                badge.style.filter = 'none';
            } else {
                atualizarStatusText('DESLIGADO', '#ef4444');
                badge.style.background = 'linear-gradient(135deg, #475569, #334155)';
                badge.style.borderColor = '#94a3b8';
                badge.style.filter = 'grayscale(80%)';
            }
        });

        document.getElementById('btn-salvar-chest-config').addEventListener('click', salvarConfiguracoes);
    }

    function salvarConfiguracoes() {
        const reloadAtivo = document.getElementById('check-reload-ativo').checked;
        const tempoMin = parseInt(document.getElementById('input-reload-min').value, 10);

        if (!isNaN(tempoMin) && tempoMin > 0) {
            config.autoReloadAtivo = reloadAtivo;
            config.tempoReloadMin = tempoMin;

            localStorage.setItem('pkhunt_chest_config', JSON.stringify(config));
            atualizarStatusText(config.ativo ? 'Config Salva!' : 'DESLIGADO', config.ativo ? '#4ade80' : '#ef4444');
        }
    }

    function atualizarStatusText(texto, cor = '#4ade80') {
        const st = document.getElementById('pkhunt-chest-status-txt');
        if (st) {
            st.innerText = texto;
            st.style.color = cor;
        }
    }

    // --- 2. LÓGICA DE COLETA DE BAÚS ---
    function clicar(elemento) {
        if (!elemento) return;
        const opts = { bubbles: true, cancelable: true, view: window };
        elemento.dispatchEvent(new MouseEvent('mousedown', opts));
        elemento.dispatchEvent(new MouseEvent('mouseup', opts));
        elemento.dispatchEvent(new MouseEvent('click', opts));
    }

    async function clicarBotaoMenuPosCarregamento() {
        let tentativas = 0;
        while (tentativas < 20) {
            const btnMenu = document.querySelector(SELETOR_BOTAO_MENU);
            if (btnMenu && btnMenu.offsetParent !== null) {
                clicar(btnMenu);
                return;
            }
            tentativas++;
            await new Promise(r => setTimeout(r, 500));
        }
    }

    function obterBotoesBausPopup() {
        const imagens = Array.from(document.querySelectorAll('img[src*="chest"]'));
        return imagens
            .map(img => img.closest('button'))
            .filter(btn => btn && !btn.classList.contains('chest-invite'));
    }

    async function processarCiclo(aviso) {
        const agoraMs = Date.now();
        if (executando || (agoraMs - ultimoProcessado < 2000)) return;

        executando = true;
        ultimoProcessado = agoraMs;

        atualizarStatusText('A abrir baú...', '#f59e0b');

        clicar(aviso);
        await new Promise(r => setTimeout(r, 1000));

        let tentativasSemBau = 0;
        let totalColetados = 0;

        while (tentativasSemBau < 2 && totalColetados < 15) {
            const botoesPopup = obterBotoesBausPopup();

            if (botoesPopup.length > 0) {
                tentativasSemBau = 0;
                clicar(botoesPopup[0]);
                totalColetados++;
                await new Promise(r => setTimeout(r, 500));
            } else {
                tentativasSemBau++;
                if (tentativasSemBau < 2) {
                    await new Promise(r => setTimeout(r, 400));
                }
            }
        }

        if (totalColetados > 0) {
            await new Promise(r => setTimeout(r, 3500));
        } else {
            await new Promise(r => setTimeout(r, 1000));
        }

        atualizarStatusText(config.ativo ? 'LIGADO' : 'DESLIGADO', config.ativo ? '#4ade80' : '#ef4444');
        executando = false;
    }

    function checarEExecutar() {
        if (!config.ativo) return;

        if (executando && (Date.now() - ultimoProcessado > 15000)) {
            executando = false;
        }

        if (!executando) {
            const aviso = document.querySelector('button.chest-invite');
            if (aviso) {
                processarCiclo(aviso);
            }
        }
    }

    // --- 3. AUTO-RELOAD CONFIGURÁVEL ---
    setInterval(() => {
        if (config.autoReloadAtivo && config.tempoReloadMin > 0) {
            const tempoDecorrido = Date.now() - window.pkhunt_chest_start_time;
            if (tempoDecorrido >= config.tempoReloadMin * 60 * 1000) {
                window.location.reload();
            }
        }
    }, 10000);

    window.pkhunt_chest_start_time = Date.now();

    window.addEventListener('load', () => {
        setTimeout(clicarBotaoMenuPosCarregamento, 1500);
    });

    try {
        const workerCode = `setInterval(() => { postMessage('tick'); }, 1000);`;
        const blob = new Blob([workerCode], { type: 'application/javascript' });
        const worker = new Worker(URL.createObjectURL(blob));

        worker.onmessage = function() {
            checarEExecutar();
        };
    } catch (e) {
        setInterval(checarEExecutar, 1000);
    }

    const observer = new MutationObserver(checarEExecutar);
    observer.observe(document.body, { childList: true, subtree: true, attributes: true });

    setInterval(() => {
        criarWidgetFlutuante();
    }, 2000);

    setTimeout(clicarBotaoMenuPosCarregamento, 1500);
})();
