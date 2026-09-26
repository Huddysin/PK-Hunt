// ==UserScript==
// @name         PKHunt - Auto Potion V10.1 (Box Maria apenas na Mira)
// @namespace    http://tampermonkey.net/
// @version      10.1
// @description  Widget flutuante que compra poções na NPC Maria com menu popup lateral e box de marcação visível APENAS durante a captura de coordenadas.
// @match        *://pkhunt.online/*
// @match        *://*.pkhunt.online/*
// @run-at       document-idle
// @grant        none
// ==/UserScript==

(function() {
    'use strict';

    // --- CARREGAR E SANITIZAR CONFIGURAÇÕES SALVAS ---
    function carregarConfiguracoes() {
        let configSalva = {};
        try {
            configSalva = JSON.parse(localStorage.getItem('pkhunt_potion_config')) || {};
        } catch (e) {
            configSalva = {};
        }

        const posicaoValida = configSalva.posicaoWidget && typeof configSalva.posicaoWidget.top === 'number' && typeof configSalva.posicaoWidget.left === 'number';

        return {
            ativo: configSalva.ativo !== undefined ? configSalva.ativo : true,
            coordsMaria: configSalva.coordsMaria || { x: 498, y: 315 },
            limiteMinimo: configSalva.limiteMinimo || 20,
            qtdCompra: configSalva.qtdCompra || 100,
            posicaoWidget: posicaoValida ? configSalva.posicaoWidget : { top: 20, left: window.innerWidth - 70 }
        };
    }

    let config = carregarConfiguracoes();

    const SELETOR_CIDADE = 'button[data-tutorial="city"]';
    const SELETOR_BATALHA = 'button[data-tutorial="battle"]';

    let modoCapturaAtivo = false;
    let processando = false;

    // --- 1. CRIAR WIDGET FLUTUANTE ---
    function criarWidgetFlutuante() {
        if (document.getElementById('pkhunt-widget-container')) return;

        const topPos = (config.posicaoWidget && typeof config.posicaoWidget.top === 'number') ? config.posicaoWidget.top : 20;
        const leftPos = (config.posicaoWidget && typeof config.posicaoWidget.left === 'number') ? config.posicaoWidget.left : (window.innerWidth - 70);

        const container = document.createElement('div');
        container.id = 'pkhunt-widget-container';
        container.style.position = 'fixed';
        container.style.top = topPos + 'px';
        container.style.left = leftPos + 'px';
        container.style.zIndex = '9999999';
        container.style.fontFamily = 'Segoe UI, Tahoma, Geneva, Verdana, sans-serif';

        const corBadge = config.ativo ? 'linear-gradient(135deg, #2563eb, #1d4ed8)' : 'linear-gradient(135deg, #475569, #334155)';
        const bordaBadge = config.ativo ? '#60a5fa' : '#94a3b8';

        container.innerHTML = `
            <!-- Ícone da Poção Flutuante -->
            <div id="pkhunt-badge" title="Segure e arraste para mover!" style="
                width: 44px;
                height: 44px;
                background: ${corBadge};
                border: 2px solid ${bordaBadge};
                border-radius: 50%;
                display: flex;
                align-items: center;
                justify-content: center;
                font-size: 22px;
                cursor: grab;
                box-shadow: 0 4px 14px rgba(0,0,0,0.5);
                transition: transform 0.2s;
                user-select: none;
                filter: ${config.ativo ? 'none' : 'grayscale(80%)'};
            ">🧪</div>

            <!-- Popup Menu Expansível Lateral -->
            <div id="pkhunt-popup" style="
                display: none;
                position: absolute;
                top: 0px;
                width: 230px;
                background: rgba(15, 23, 42, 0.96);
                border: 1px solid #3b82f6;
                border-radius: 10px;
                padding: 12px;
                color: #e2e8f0;
                box-shadow: 0 10px 25px rgba(0,0,0,0.6);
                backdrop-filter: blur(5px);
                flex-direction: column;
                gap: 10px;
                font-size: 12px;
            ">
                <!-- Cabeçalho com Checkbox Ligar/Desligar -->
                <div style="display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid #334155; padding-bottom: 8px;">
                    <strong style="color: #60a5fa;">🧪 AutoPotion</strong>
                    <label style="display: flex; align-items: center; gap: 6px; cursor: pointer; font-weight: bold;">
                        <input type="checkbox" id="check-script-ativo" ${config.ativo ? 'checked' : ''} style="width: 15px; height: 15px; cursor: pointer;">
                        <span id="pkhunt-status-txt" style="color: ${config.ativo ? '#4ade80' : '#ef4444'}; font-size: 11px;">
                            ${config.ativo ? 'LIGADO' : 'DESLIGADO'}
                        </span>
                    </label>
                </div>

                <!-- Configuração de Coordenadas -->
                <div style="display: flex; flex-direction: column; gap: 4px;">
                    <span style="color: #94a3b8; font-size: 11px;">Coordenada NPC Maria:</span>
                    <div style="display: flex; gap: 6px; align-items: center;">
                        <label>X:</label>
                        <input type="number" id="input-coord-x" value="${config.coordsMaria.x}" style="width: 50px; background: #1e293b; color: #fff; border: 1px solid #475569; border-radius: 4px; padding: 2px 4px; text-align: center;">
                        <label>Y:</label>
                        <input type="number" id="input-coord-y" value="${config.coordsMaria.y}" style="width: 50px; background: #1e293b; color: #fff; border: 1px solid #475569; border-radius: 4px; padding: 2px 4px; text-align: center;">
                    </div>
                </div>

                <!-- Configuração de Poções -->
                <div style="display: flex; flex-direction: column; gap: 4px;">
                    <div style="display: flex; justify-content: space-between; align-items: center;">
                        <span style="color: #94a3b8; font-size: 11px;">Comprar se < :</span>
                        <input type="number" id="input-limite" value="${config.limiteMinimo}" style="width: 50px; background: #1e293b; color: #fff; border: 1px solid #475569; border-radius: 4px; padding: 2px 4px; text-align: center;">
                    </div>
                    <div style="display: flex; justify-content: space-between; align-items: center;">
                        <span style="color: #94a3b8; font-size: 11px;">Qtd a Comprar:</span>
                        <input type="number" id="input-qtd" value="${config.qtdCompra}" style="width: 50px; background: #1e293b; color: #fff; border: 1px solid #475569; border-radius: 4px; padding: 2px 4px; text-align: center;">
                    </div>
                </div>

                <!-- Botões de Ação -->
                <div style="display: flex; gap: 6px; margin-top: 4px;">
                    <button id="btn-salvar-config" style="flex: 1; background: #2563eb; color: white; border: none; border-radius: 4px; padding: 6px; cursor: pointer; font-weight: bold;">Salvar</button>
                    <button id="btn-capturar-coords" style="flex: 1; background: #d97706; color: white; border: none; border-radius: 4px; padding: 6px; cursor: pointer; font-weight: bold;">🎯 Mira</button>
                </div>
            </div>
        `;

        document.body.appendChild(container);

        const badge = document.getElementById('pkhunt-badge');
        const popup = document.getElementById('pkhunt-popup');

        // --- FUNÇÃO PARA ABRIR O POPUP LATERALMENTE ---
        function atualizarAlinhamentoPopupLateral() {
            const currentLeft = container.offsetLeft;
            const metadeTela = window.innerWidth / 2;

            if (currentLeft > metadeTela) {
                popup.style.right = '52px';
                popup.style.left = 'auto';
            } else {
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
                    localStorage.setItem('pkhunt_potion_config', JSON.stringify(config));
                }
            };

            document.addEventListener('mousemove', onMouseMove);
            document.addEventListener('mouseup', onMouseUp);
        });

        // Hover para abrir/fechar popup
        let timerHover;
        container.addEventListener('mouseenter', () => {
            if (!isDragging) {
                clearTimeout(timerHover);
                atualizarAlinhamentoPopupLateral();
                popup.style.display = 'flex';
                badge.style.transform = 'scale(1.1)';
            }
        });

        container.addEventListener('mouseleave', () => {
            if (!modoCapturaAtivo) {
                timerHover = setTimeout(() => {
                    popup.style.display = 'none';
                    badge.style.transform = 'scale(1)';
                }, 400);
            }
        });

        // Checkbox Ligar/Desligar
        document.getElementById('check-script-ativo').addEventListener('change', function(e) {
            config.ativo = e.target.checked;
            localStorage.setItem('pkhunt_potion_config', JSON.stringify(config));

            if (config.ativo) {
                atualizarStatusText('LIGADO', '#4ade80');
                badge.style.background = 'linear-gradient(135deg, #2563eb, #1d4ed8)';
                badge.style.borderColor = '#60a5fa';
                badge.style.filter = 'none';
            } else {
                atualizarStatusText('DESLIGADO', '#ef4444');
                badge.style.background = 'linear-gradient(135deg, #475569, #334155)';
                badge.style.borderColor = '#94a3b8';
                badge.style.filter = 'grayscale(80%)';
            }
        });

        // Eventos dos botões
        document.getElementById('btn-salvar-config').addEventListener('click', salvarConfiguracoesManuais);
        document.getElementById('btn-capturar-coords').addEventListener('click', alternarModoCaptura);
    }

    function salvarConfiguracoesManuais() {
        const x = parseInt(document.getElementById('input-coord-x').value, 10);
        const y = parseInt(document.getElementById('input-coord-y').value, 10);
        const limite = parseInt(document.getElementById('input-limite').value, 10);
        const qtd = parseInt(document.getElementById('input-qtd').value, 10);

        if (!isNaN(x) && !isNaN(y) && !isNaN(limite) && !isNaN(qtd)) {
            config.coordsMaria = { x, y };
            config.limiteMinimo = limite;
            config.qtdCompra = qtd;

            localStorage.setItem('pkhunt_potion_config', JSON.stringify(config));
            atualizarStatusText(config.ativo ? 'Config Salva!' : 'DESLIGADO', config.ativo ? '#4ade80' : '#ef4444');
        }
    }

    function alternarModoCaptura() {
        modoCapturaAtivo = !modoCapturaAtivo;
        const btn = document.getElementById('btn-capturar-coords');
        if (modoCapturaAtivo) {
            btn.style.background = '#dc2626';
            btn.innerText = '🎯 Clicando...';
            atualizarStatusText('Clique na Maria...', '#f59e0b');
        } else {
            btn.style.background = '#d97706';
            btn.innerText = '🎯 Mira';
            atualizarStatusText(config.ativo ? 'LIGADO' : 'DESLIGADO', config.ativo ? '#4ade80' : '#ef4444');
        }
    }

    function atualizarStatusText(texto, cor = '#4ade80') {
        const st = document.getElementById('pkhunt-status-txt');
        if (st) {
            st.innerText = texto;
            st.style.color = cor;
        }
    }

    // --- 2. DESENHAR A BOX VERMELHA NA NPC MARIA (SOMENTE DURANTE A MIRA) ---
    function desenharBoxMariaNaTela(relX, relY) {
        if (!modoCapturaAtivo) return; // Só desenha se o botão de mira estiver ativo

        const canvas = document.querySelector('canvas.city-canvas') || document.querySelector('canvas');
        if (!canvas) return;

        const rect = canvas.getBoundingClientRect();
        const absX = rect.left + relX;
        const absY = rect.top + relY;

        let box = document.getElementById('pkhunt-maria-box');
        if (!box) {
            box = document.createElement('div');
            box.id = 'pkhunt-maria-box';
            box.style.position = 'fixed';
            box.style.border = '3px solid #ef4444';
            box.style.backgroundColor = 'rgba(239, 68, 68, 0.35)';
            box.style.boxShadow = '0 0 15px #ef4444';
            box.style.pointerEvents = 'none';
            box.style.zIndex = '9999999';
            box.style.borderRadius = '4px';
            box.style.transition = 'all 0.1s ease-in-out';
            document.body.appendChild(box);
        }

        box.style.left = (absX - 25) + 'px';
        box.style.top = (absY - 25) + 'px';
        box.style.width = '50px';
        box.style.height = '50px';
        box.style.display = 'block';

        setTimeout(() => {
            if (box) box.style.display = 'none';
        }, 3000);
    }

    // --- 3. DETECTAR CLIQUE NO CANVAS DENTRO DO MODO MIRA ---
    document.addEventListener('click', function(e) {
        if (!modoCapturaAtivo) return;

        const canvas = document.querySelector('canvas.city-canvas') || document.querySelector('canvas');
        if (canvas) {
            const rect = canvas.getBoundingClientRect();
            if (e.clientX >= rect.left && e.clientX <= rect.right &&
                e.clientY >= rect.top && e.clientY <= rect.bottom) {

                const relX = Math.round(e.clientX - rect.left);
                const relY = Math.round(e.clientY - rect.top);

                if (relX > 10 && relY > 10) {
                    config.coordsMaria = { x: relX, y: relY };
                    document.getElementById('input-coord-x').value = relX;
                    document.getElementById('input-coord-y').value = relY;

                    localStorage.setItem('pkhunt_potion_config', JSON.stringify(config));

                    desenharBoxMariaNaTela(relX, relY);
                    alternarModoCaptura();
                    atualizarStatusText(config.ativo ? 'LIGADO' : 'DESLIGADO', config.ativo ? '#4ade80' : '#ef4444');
                }
            }
        }
    }, true);

    // --- 4. EXECUÇÃO AUTOMÁTICA ---
    function clicarGarantido(el) {
        if (!el) return;
        try {
            if (typeof el.click === 'function') el.click();
            const opts = { bubbles: true, cancelable: true, view: window, pointerId: 1, isPrimary: true };
            el.dispatchEvent(new PointerEvent('pointerdown', opts));
            el.dispatchEvent(new MouseEvent('mousedown', opts));
            el.dispatchEvent(new PointerEvent('pointerup', opts));
            el.dispatchEvent(new MouseEvent('mouseup', opts));
            el.dispatchEvent(new MouseEvent('click', opts));
        } catch (e) {
            console.error(e);
        }
    }

    function clicarCanvas(canvas, relX, relY) {
        if (!canvas) return;
        const rect = canvas.getBoundingClientRect();
        const absX = rect.left + relX;
        const absY = rect.top + relY;

        // Sem chamada para desenharBoxMariaNaTela aqui durante a execução automática

        const opts = { bubbles: true, cancelable: true, view: window, clientX: absX, clientY: absY, button: 0 };
        canvas.dispatchEvent(new MouseEvent('mousedown', opts));
        canvas.dispatchEvent(new MouseEvent('mouseup', opts));
        canvas.dispatchEvent(new MouseEvent('click', opts));
    }

    function definirValorInputNativo(input, valor) {
        const nativeInputValueSetter = Object.getOwnPropertyDescriptor(window.HTMLInputElement.prototype, "value").set;
        nativeInputValueSetter.call(input, valor);
        input.dispatchEvent(new Event('focus', { bubbles: true }));
        input.dispatchEvent(new Event('input', { bubbles: true }));
        input.dispatchEvent(new Event('change', { bubbles: true }));
        input.dispatchEvent(new Event('blur', { bubbles: true }));
    }

    async function executarCompra() {
        atualizarStatusText('Indo Cidade...', '#f59e0b');

        const btnCidade = document.querySelector(SELETOR_CIDADE);
        if (btnCidade) {
            clicarGarantido(btnCidade);
            await new Promise(r => setTimeout(r, 2000));
        }

        atualizarStatusText('Clicando Maria...', '#3b82f6');
        const canvas = document.querySelector('canvas.city-canvas') || document.querySelector('canvas');
        if (canvas) {
            clicarCanvas(canvas, config.coordsMaria.x, config.coordsMaria.y);
            await new Promise(r => setTimeout(r, 2000));
        }

        atualizarStatusText('Comprando...', '#3b82f6');
        const inputPotion = document.querySelector('input[aria-label="Quantidade de Potion"]');
        if (inputPotion) {
            inputPotion.focus();
            definirValorInputNativo(inputPotion, config.qtdCompra);
            await new Promise(r => setTimeout(r, 1000));

            let btnComprar = document.querySelector('button.ui-btn.ui-btn-blue.ui-btn-md');
            if (!btnComprar) {
                const todosBotoes = Array.from(document.querySelectorAll('button'));
                btnComprar = todosBotoes.find(b => b.textContent.trim().toLowerCase().includes('comprar tudo'));
            }

            if (btnComprar) {
                clicarGarantido(btnComprar);
                await new Promise(r => setTimeout(r, 1500));
            }

            const btnFechar = document.querySelector('.ui-panel-close');
            if (btnFechar) clicarGarantido(btnFechar);
            await new Promise(r => setTimeout(r, 1000));
        }

        atualizarStatusText('Voltando Batalha...', '#f59e0b');
        const btnBatalha = document.querySelector(SELETOR_BATALHA);
        if (btnBatalha) {
            clicarGarantido(btnBatalha);
        }

        atualizarStatusText('LIGADO', '#4ade80');
    }

    async function checarEComprar() {
        if (!config.ativo || processando || modoCapturaAtivo) return;

        const elemPotion = document.querySelector('div[title*="Potion"]');
        if (!elemPotion) return;

        const match = elemPotion.textContent.match(/×\s*(\d+)/);
        if (!match) return;

        const qtdAtual = parseInt(match[1], 10);

        if (qtdAtual < config.limiteMinimo) {
            processando = true;
            await executarCompra();
            await new Promise(r => setTimeout(r, 10000));
            processando = false;
        }
    }

    // Inicialização
    setInterval(() => {
        criarWidgetFlutuante();
        checarEComprar();
    }, 2000);
})();
